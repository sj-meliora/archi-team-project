# DP-06 Agent 지식 베이스 설계 (Knowledge Base) — RAG vs LLM Wiki

> category: DP | status: 초안 (디스커션 전 — 팀 검토 대기) | source: 신규 발굴 (자율 이슈처리 서사의 지식 공급 공백) | updated: 2026-08-10
> drives: QA-07(Correctness)↑, QA-05(Efficiency), QA-01(Scalability)
> realizes: FR-0003(Regression 관리의 지식 축), FR-0004(이슈처리 근거 추적) | constrained-by: C-03(지식 접근도 권한 게이트 통과)
> 신설 note: DP-01(제어평면)·DP-0004(실행구조)는 "Agent가 **어떻게 실행되나**"를 다루고, 어떤 DP도 "Agent가 **무엇을 알고 판단하나**"를 다루지 않았다. 자율 이슈처리(불량 분석·회귀)가 발표 서사의 핵심인데 그 전제인 **지식 공급 계층**이 미결정 — 본 DP가 그 공백을 닫는다. 미결정 트래킹: open-issues OI-13.

---

## 용어 (먼저 읽기 — 이 결정에서 쓰는 표현)

| 용어 | 뜻 |
|---|---|
| **지식 베이스 (Knowledge Base)** | Agent가 판단(불량 분석·회귀 선별 등)에 필요한 **조직 지식**(과거 이슈·config 변경 이력·모델별 특이사항·툴체인 제약)을 저장하고 **선별 공급**하는 계층. raw 산출물 저장소(FR-0002 Artifact Storage)와 구분 — KB는 그 **위의 검색·증류 계층** |
| **grounding / hallucination** | grounding = LLM 답을 **실제 근거 데이터에 붙들어 매는 것** / hallucination = 근거 없이 그럴듯하게 지어내는 것. 사내 SDK·NPU 도메인은 LLM 학습 데이터에 없으므로(비공개+컷오프) grounding 없이는 hallucination이 기본값 |
| **RAG** (Retrieval-Augmented Generation) | 질의 시점에 raw 데이터(로그·이슈·문서)를 **검색해서** 관련 조각을 프롬프트에 넣어주는 방식. 보통 임베딩 기반 벡터 검색 |
| **임베딩 / 벡터 검색** | 텍스트를 의미 좌표(벡터)로 바꿔(임베딩) "의미가 비슷한" 조각을 찾는 검색. 키워드 일치가 아니라 유사도 기반 |
| **chunk / top-k** | chunk = 문서를 검색 단위로 자른 조각 / top-k = 유사도 상위 k개 조각만 골라 프롬프트에 주입. **조각이라서 문맥·인과가 잘려 나갈 수 있다**(단편성) |
| **LLM Wiki** (LLM-curated wiki) | Agent(LLM)가 경험을 **구조화 문서로 증류·갱신**해 두고, 다른 Agent가 그 문서를 읽는 방식 — 이슈 close 시 postmortem 요약, config 변경 시 영향 요약을 위키 페이지로 축적. *사람 위키의 Agent판: 쓰는 쪽도 읽는 쪽도 Agent, 사람은 감사(audit) 가능* |
| **큐레이션 (curation) / 증류** | raw 데이터에서 **패턴·인과·교훈만 추려 정제 문서로 만드는 것**. Wiki의 쓰기 비용이자 품질 원천 |
| **staleness (낡음)** | 큐레이션된 지식이 갱신을 못 따라가 **현실과 어긋난 상태**. Wiki의 고유 리스크 (RAG는 인덱싱만 돌면 원본 최신성을 따라감) |
| **prompt cache 적합성** | 같은 문서를 여러 요청이 반복해서 읽으면 prompt caching(QA-05 — 캐시 read는 정상가의 ~10%) 적중률이 오른다. **안정된 위키 문서는 캐시 친화적**, 질의마다 조합이 바뀌는 top-k chunk는 캐시 비친화적 |
| **ASR / driving QA** | ASR=아키텍처 핵심 요구(목록·우선순위 SSoT: [`context/asr.md`](../asr.md)) / driving QA=이 결정이 좌우하는 주 품질속성 |
| **★ 등급 · S·T·R·N** | ★=그 QA 만족도(등급척도 앵커) · S 민감점 · T 교환점 · R 위험 · N 비위험(ATAM) |

---

## 슬라이드 요약 (2장 — 16:9)

### 슬라이드 1 — 왜 지식 베이스인가 (필요성)

**결정 포인트**: Agent가 불량 분석·회귀를 수행하려면 조직 지식에 접근할 수 있어야 한다 — 그 지식을 **어떤 형태로 저장하고 Agent에 공급할 것인가.**

**필요성 3축** (사람이 하던 일을 Agent가 하려면 사람이 쓰던 지식이 필요하다):

| 축 | 문제 | 지식 베이스가 닫는 것 |
|---|---|---|
| **① 자율 이슈처리의 전제** | 불량 분석(failure triage)·회귀(regression) 선별은 **과거 유사 이슈·직전 config 변경·모델별 특이사항** 없이는 판단 자체가 불가능. 사람 엔지니어는 이 지식을 머리·위키·채팅에 갖고 있었다 — Agent에게는 공급 경로가 **없다** (Pain Point: 개인별 config 관리, Jira 대신 채팅 우회 → 지식 휘발) | 흩어진 조직 지식을 Agent가 **조회 가능한 형태**로 집약 → FR-0003(변경 추적·Regression), FR-0004(이슈처리 근거) |
| **② LLM 파라미터 지식의 한계** | 사내 SDK 파이프라인·NPU 툴체인·모델 조합은 **학습 데이터에 없다**(비공개+컷오프) → grounding 없이는 hallucination이 기본값. "일관되게 틀린" 분석은 rework 폭증(QA-07이 경고하는 바로 그 경로) | 판단을 **실제 근거(이슈 이력·로그·config diff)에 grounding** → QA-07 golden 정답률의 전제 |
| **③ 컨텍스트 윈도·토큰 한계** | 빌드 로그·20GB+ 산출물·전체 이슈 이력을 프롬프트에 다 넣을 수 없고, 욱여넣을수록 신규 토큰 캡(QA-05: ≤6k/8k) 위반 + 비용 폭증 | **필요한 지식만 선별 공급**하는 검색·증류 계층 → QA-05 신규 토큰 캡 안에서 grounding 성립 |

**동작 예시 (한 줄 서사)**: 불량 발생 → Agent가 *유사 과거 이슈 + 직전 config diff + 해당 모델 특이사항*을 지식 베이스에서 조회 → 원인 가설 수립 → 영향 범위 기반 회귀 세트 선별 실행 → 처리 결과가 다시 지식으로 축적(선순환).

> 요지: **지식 베이스는 "있으면 좋은 것"이 아니라 자율 이슈처리(발표 서사의 핵심 FR)의 성립 조건**이다. 다음 슬라이드에서 "어떤 형태로"를 결정한다.

### 슬라이드 2 — RAG vs LLM Wiki (형태 결정)

> 대안별 column = (a) 도안 + (b) 설명 + (c) 별점(행=ASR). 도안 SVG는 미작성(TODO — `diagrams/` 승격 시).

| | **1안 · RAG (벡터 검색)** | **2안 · LLM Wiki (증류 문서)** |
|---|---|---|
| **(a) 도안** | *(TODO: raw 저장소→임베딩 인덱스→top-k 주입)* | *(TODO: 이슈 close→증류 agent→위키 갱신→조회)* |
| **(b) 설명** | **구조**: 이슈·로그·config diff 등 raw 데이터를 그대로 임베딩 인덱싱하고, Agent가 질의 시점에 **top-k 유사 조각을 검색**해 프롬프트에 주입한다. 쓰기 경로는 자동 인덱싱 파이프라인뿐(큐레이션 없음).<br>**＋** 원본이 갱신되면 인덱스만 따라가므로 **최신성·커버리지가 자동**이고, 데이터 증가를 인덱싱이 흡수해 조합 폭발(모델 140+)에 강하다.<br>**－** 검색 품질(임베딩·chunking)에 정확성이 종속되고, **조각(chunk) 단편성** 때문에 인과·패턴이 잘려 오합성 위험이 있으며, 질의마다 top-k 조합이 바뀌어 **prompt cache 비친화적**(신규 토큰↑)이다. | **구조**: Agent가 이슈 close·config 변경 등 이벤트 시점에 경험을 **위키 문서로 증류**(postmortem·모델별 특이사항·알려진 회귀 패턴)하고, 판단 시 해당 문서를 읽는다. 사람은 위키를 감사(HITL audit)할 수 있다.<br>**＋** 인과·패턴이 정리된 **정제 지식**이라 판단 품질이 높고, 증류 요약이라 읽기 토큰이 적으며 **안정 문서 = prompt cache 친화적**(QA-05 캐시 분리집계와 정합). 사람이 검증 가능한 형태(감사 용이).<br>**－** 갱신 파이프라인이 못 따라가면 **staleness**(낡은 지식으로 틀린 판단), 문서화 안 된 영역은 **검색 자체가 불가**(커버리지 공백), 증류에 쓰기 시점 LLM 비용(QA-13)이 든다. |
| **(c) QA-07 Correctness** | ★★☆ | ★★★ ◯ |
| **(c) QA-05 Efficiency** | ★★☆ | ★★★ |
| **(c) QA-01 Scalability** | ★★★ | ★★☆ |

> ★ 앵커: Correctness = golden 정답률 [93,99]=★★★(QA-07) · Efficiency = 작업당 신규 토큰 ≤4k=★★★(QA-05, 캐시 분리집계) · Scalability = QA-01 등급척도. **2안 Correctness ★★★◯** = 갱신 파이프라인이 staleness를 억제한다는 가정(이벤트 트리거 증류 동작 PoC 확정 전 `◯`).

**권고**: 판단에 필요한 지식이 **반복 패턴(증류 가능)** 이면 2안, **비정형 롱테일(raw 검색 필요)** 이면 1안 — 실제 불량 분석은 둘 다 요구하므로 **2안(Wiki)을 1차 조회층 + 1안(RAG)을 raw fallback으로 결합한 하이브리드를 승격 후보**로 디스커션에 회부(A4). **택일 강행 금지 — 드라이버로 결정.** (상세 → Appendix.)

---

## Appendix

### A1. 결정의 핵심 — "그냥 컨텍스트에 다 넣으면 안 되나"
청중이 흔히 "요즘 LLM 컨텍스트 크지 않나"라 묻는데, 세 이유로 안 된다:
- **물리적으로 안 들어간다**: 빌드 로그 수십 MB·이슈 이력 수년치·20GB 산출물은 어떤 컨텍스트 윈도에도 안 들어간다 — 선별이 필수.
- **넣을수록 비싸고 느려진다**: 컨텍스트를 채울수록 신규 토큰 캡(QA-05 ≤6k/8k) 위반 + 지연 증가. "다 넣기"는 QA-05·QA-09를 동시에 깨는 전략.
- **넣어도 못 찾는다**: 장문 컨텍스트 중간의 정보는 회수율이 떨어진다(lost-in-the-middle) — 관련 지식만 좁혀 주입하는 것이 정확성에도 유리.
- 따라서 질문은 "지식 공급 계층이 필요한가"(자명)가 아니라 **"검색(RAG)이냐 증류(Wiki)냐"** — 저장 형태가 정확성·토큰 효율·확장성의 교환을 결정한다(= 본 DP 교환점).

**결정 드라이버 (택일 단일 질문)**
> **"불량 분석·회귀가 요구하는 지식이 *반복 패턴*(같은 유형 불량·알려진 회귀 — 증류 가능)이 지배적인가, *비정형 롱테일*(신규 유형 — raw 검색 필요)이 지배적인가?"**
> - **반복 패턴 지배 → LLM Wiki(2안)** — 증류가 정확성·토큰 효율을 동시에 올림.
> - **롱테일 지배 → RAG(1안)** — 문서화 안 된 영역은 raw 검색만이 닿는다.
> - **둘 다 실질적(예상 워크로드) → 하이브리드 승격(A4)** — Wiki 1차 + RAG fallback.

### A2. 후보 대안 (전문)
**1안. RAG (Retrieval-Augmented Generation)** — 구조: raw 데이터(이슈·로그·config diff·문서) 자동 임베딩 인덱싱, 질의 시 top-k 검색 주입. tactic: Retrieval grounding, 자동 인덱싱 파이프라인. 장점 [Scalability] 인덱싱이 데이터 증가 흡수·큐레이션 노동 0 / [Correctness] 원본 최신성 자동. 단점 [Correctness] 검색 품질 종속 + chunk 단편성(인과 절단) / [Efficiency] top-k 주입 토큰 + 캐시 비친화.

**2안. LLM Wiki (LLM-curated wiki)** — 구조: 이벤트 트리거(이슈 close·config 변경) 시 Agent가 postmortem·특이사항·회귀 패턴을 위키 문서로 증류, 판단 시 문서 조회, 사람 감사 가능. tactic: Knowledge distillation, 이벤트 트리거 갱신, HITL audit. 장점 [Correctness] 인과·패턴 정제 / [Efficiency] 요약 읽기 + 안정 문서 캐시 친화. 단점 [Correctness] staleness 리스크·미문서화 영역 공백 / [Cost·QA-13] 증류 쓰기 비용 / [Scalability] 큐레이션 파이프라인이 갱신 병목 후보.

**(승격 후보) 3안. 하이브리드 (Wiki 1차 + RAG fallback)** — 구조: 판단 시 Wiki 먼저 조회, 미문서화·신규 유형이면 RAG로 raw 검색, 처리 결과를 다시 Wiki로 증류(선순환 루프). 기대: 1안 커버리지 + 2안 정제·캐시 효율. 자기 trade-off: 두 계층 운영 복잡도·이중 인프라 비용(QA-13). **디스커션에서 정식 대안 승격 여부 결정**(현재 별점 미부여 — strawman 방지).

### A3. 대안 × ASR 통합 매트릭스 (★ + S/T/R/N)
| 대안 | QA-07 Correctness | QA-05 Efficiency | QA-01 Scalability |
|---|:---:|:---:|:---:|
| 1안 RAG (벡터 검색) | ★★☆ (S,T,R) | ★★☆ (T) | ★★★ (N) |
| 2안 LLM Wiki (증류 문서) | ★★★◯ (S,R) | ★★★ (S,T) | ★★☆ (R) |

> 하이브리드(3안)는 승격 전이라 매트릭스 미포함. 선택안 확정 시 해당 행이 세트 Traceability Matrix로 graduate. S=민감점·T=교환점·R=위험·N=비위험.

### A4. 대안 고도화 (비선택안도 tactic으로 강화)
- **1안 RAG Correctness(★★☆) 강화**: chunk 단편성 → **contextual retrieval**(chunk에 문맥 요약 전치)·re-ranking·hybrid search(BM25+벡터)로 검색 실패율 감소 — ★★★ 근접 가능. *그러나* 캐시 비친화(질의별 top-k 변동)는 구조적이라 Efficiency 열위 잔존.
- **2안 Wiki Correctness 리스크(staleness) 강화**: 갱신을 배치(주기)가 아닌 **이벤트 트리거**(이슈 close·config merge 시점 증류)로 — staleness 창을 이벤트 지연으로 축소. `◯` 해제 조건 = 이 파이프라인 동작 PoC.
- **2안 커버리지 공백 강화 = 1안 결합**: 미문서화 영역 fallback으로 RAG 부착 → **3안(하이브리드) 승격 경로** — DP-01의 "Standby 승격"(약점을 tactic으로 닫아 승격)과 동형.

### A5. ATAM 분석
**민감점** — SP-1(검색 품질(임베딩·chunking)→QA-07 정답률, 1안) · SP-2(지식 신선도(갱신 지연)→QA-07, 2안) · SP-3(문서 안정성→prompt cache 적중률→QA-05 신규 토큰, 2안).
**교환점** — **TP-1 (커버리지·최신성 ↔ 정제·토큰 효율)** 본 DP 핵심: raw 검색(1안)은 넓고 신선하나 단편·토큰↑, 증류(2안)는 정제·효율적이나 좁고 낡을 수 있음 · TP-2 (읽기 효율 ↔ 쓰기 비용: 2안 증류는 읽기 토큰을 줄이는 대신 쓰기 시점 LLM 비용 발생 — QA-05↔QA-13) · TP-3 (검색 정밀도 ↔ 재현율: 1안 top-k를 좁히면 토큰↓ but 근거 누락↑).
**위험** — R-1(1안 chunk 단편성 → 인과 오합성 → 불량 오진, QA-07) · R-2(2안 staleness → 낡은 지식 기반 회귀 판단 오류, QA-07 — 이벤트 트리거 증류로 완화) · R-3(2안 미문서화 영역 검색 불가 → 신규 유형 불량 분석 실패 — RAG fallback으로 완화 = 3안 승격 동인) · R-4(2안 증류 자체가 LLM 산출물 → 위키가 오염되면 이후 판단이 연쇄 오염(오류 전파). HITL audit + QA-07 golden 게이트로 방어).
**비위험** — NR-1(두 안 모두 C-03 권한 게이트(접근 제어) 하에서 동작 가능 — pass/fail 제약이라 비교 축 아님) · NR-2(두 안 모두 FR-0002 Artifact Storage를 raw 원천으로 공유 — 저장 이중화 아님, KB는 상위 계층).

### A6. Cohesion (DP 간 보완·연결)
- **DP-0004(실행구조)·DP-01(오케스트레이션)과 역할 분리**: DP-01/0004 = Agent가 *어떻게 실행*되나(제어·실행 평면), 본 DP = *무엇을 알고 판단*하나(지식 평면). 노드 Agent·오케스트레이터 모두 KB의 소비자.
- **FR-0002(Artifact Storage)와 계층 관계**: raw 산출물 저장은 FR-0002 몫, KB는 그 위의 **검색·증류 계층** — 원천 데이터를 이중 보관하지 않는다(NR-2).
- **QA-04(Observability)가 원천 공급**: span trace·실행 이력·결정 로그가 KB의 입력(불량 분석의 1차 사료) — DP-0003 모니터링과 데이터 파이프 공유.
- **QA-07 eval 하네스와 상호 보강**: KB grounding은 golden 정답률의 전제(슬라이드 1 ②축), 역으로 QA-07 golden 게이트가 위키 오염(R-4)의 방어선 — eval/검증 서브시스템 DP(OI-7 후보)와 세트.
- **C-03(보안·안전)**: 지식 접근도 권한 게이트 통과 — Agent별 접근 범위 제한(예: 외부 공유 금지 지식). 두 안 공통 제약(NR-1).
- **module-view 반영 필요**: Repository 레이어에 `Knowledge Base` 박스(+ 2안이면 증류 Agent 경로) 신설 후보 — 선택안 확정 시 반영(OI-13).

### A7. 별점 앵커 근거 (레퍼런스)
- RAG 원형·retrieval grounding: [Lewis et al., RAG (NeurIPS 2020)](https://arxiv.org/abs/2005.11401) · chunk 단편성 완화: [Anthropic — Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval)
- 장문 컨텍스트 중간 정보 회수율 저하(A1 "다 넣기" 반박): [Lost in the Middle (TACL 2024)](https://arxiv.org/abs/2307.03172)
- 안정 문서 prompt cache 경제성(2안 Efficiency 앵커): [Anthropic — prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) (cache read = 정상가 ~10%, QA-05와 동일 출처)
- Agent 경험의 구조화 기억·회고 증류(2안 원형): [Generative Agents — memory stream & reflection (Park et al. 2023)](https://arxiv.org/abs/2304.03442)
- 별점 수치는 PoC로 확정(레퍼런스 수치 복제 금지). QA-07/05/01 등급척도 = `context/qa/QA-07·QA-05·QA-01`.
