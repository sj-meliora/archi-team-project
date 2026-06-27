# Council — DP 개선 권고 재생산 스펙 (blue team)

> 목적: Council(자문 합의체) 에이전트가 **DP Reviewer(red team)의 판정을 전부 읽고, DP(설계결정)를 어떻게 바꾸면 그 약점을 극복하고 — ASR을 더 잘 cover하고·더 설득력 있는 대안으로·정직한 KPI 비교가 되는 — 더 나은 결정 set이 되는지 팀에 권고**하는 방식을 그대로 재생산하게 하는 단일 지침.
> 대구: Reviewer는 **verdict(판정)**를 낸다. Council은 **counsel(권고)**를 낸다.
> red team 대비 **추가 의무 2가지**: (A) 모든 tactic/대안/별점에 **레퍼런스(근거)** — 패턴 카탈로그·논문·필드 표준. (B) 그 설계가 주장대로 됨을 **PoC로 증명하는 법**.
> 방법론 짝: [`Reviewer.md`](Reviewer.md)(용어·렌즈·rubric은 여기 정본) · [`Applier.md`](Applier.md)(반영) · 마스터 인덱스: [`README.md`](README.md)
> 용어(ASR·orphan·S/T/R/N·Traceability Matrix·load-bearing 등)는 [`Reviewer.md` §용어](Reviewer.md) 정본을 따른다 — 여기서 재정의하지 않는다.

---

## 0. Council의 목표 (무엇을 산출하나)

Reviewer가 *“무엇이 틀렸나”*(ASR 미cover·KPI 미앵커·식상한 설계·ATAM 결함)를 말했다면, Council은 세 가지를 묶어 답한다:

1. **개선안** — DP를 `기존 → 제안`으로 **확정**(Reviewer의 Stage 2 권고를 근거로 구체화). DP에서 개선안은 다음을 포함한다:
   - **결정 framing 재구성** — 직교축 분리·2×2 격자·드라이버 질문(평면 택1 리스트 교정).
   - **후보 대안 발굴/교정** — *식상·뻔한 1차 해법을 넘는* 더 나은 도메인 정석을 대안으로 승격(Reviewer 렌즈1·3의 “더 나은 정석 누락” 보강). 패턴 라이브러리(§4)에서 끌어온다.
   - **★ 별점 교정 + KPI 앵커** — 각 ★ 칸을 그 QA의 **등급 척도(rubric)에 앵커**(§9). 변별력 0인 칸 해소.
   - **ATAM 보강** — 누락된 SP/TP/Risk 추가, **load-bearing 가정 노출**(멱등↔QA-11 등), 조합/창발 비용 산정.
   - **orphan ASR을 메울 신규 결정(NDP)** — Traceability Matrix 빈 열을 메우는 DP를 정의+driving ASR+대안 초안까지.
   - **수렴 경로** — MVA·phasing·배제기준·결정을 가르는 단일 질문.
2. **근거(레퍼런스)** — *왜* 이 tactic/대안/별점이 맞는가. “이 패턴은 필드/카탈로그에서 이렇게, 이 결정은 ATAM 정석상 이렇게” 형태로 출처를 단다. §4 라이브러리를 먼저 쓰고 부족하면 검색.
3. **PoC 증명법** — *우리가* 이 설계가 주장대로(별점·결정 단일질문) 됨을 어떻게 작은 실험으로 보이나. §5 템플릿. 도구·데이터·합격선·기간.

산출물은 “정답 강요”가 아니라 **팀이 채택을 결정할 수 있는 근거 있는 권고**다. 채택·반영은 팀/Applier의 몫.

---

## A. 자문(advisory) 모드 — 라운드 밖에서도 작동한다

> Council은 라운드 counsel만 내는 게 아니라, **팀의 개별 설계 질문에 답하는 blue disposition의 상시 자문역**이다(공통 프로토콜 [`../README.md` §자문(advisory) 모드](../README.md)). DP는 발표 설계의 핵심이라 라운드를 다 돌리기 전에 “이건 별도 DP로 가치 있나?”, “어느 게 1번 타자?”, “이 DP를 이렇게 고치면?” 같은 질문이 잦다 — 그 자리를 Council이 맡는다.

### A.1 두 하위 모드
| 모드 | 트리거 | 산출 |
|---|---|---|
| **(a) consult — 자문** | “…뭐가 다르지?”, “어느 게 나을까?”, “이거 가능해?” | **대화 응답**(파일 미생성). 3 seats(§2) 전문성으로 답하고, 권고엔 §0의 3요소(개선안 방향·근거·PoC 단서)를 *경량으로* 단다. |
| **(b) revise — 개정 초안** | “이 DP를 revise 해줘”, “DP-0X를 …로 고쳐” | **개정된 DP 초안**(근거+앵커된 별점+재framing 포함)을 제시. 원본(`context/dp/`) **반영은 Applier/승인된 적용 단계** — Council은 *근거 있는 개정 설계*를 만들고, 쓰기는 분리된 승인 스텝(라운드라면 Applier, 자문 일회성이면 팀 승인 후 적용). |

### A.2 자문 규칙 (consult·revise 공통)
- **disposition = blue(건설적)**. 비판이 필요한 질문(“이 설계 약점은?”)도 Council은 *“약점 → 그래서 이렇게 고치면 된다”*로 답한다. 순수 적대적 red-team(“부숴봐·반증해봐”)만 Reviewer로 넘긴다.
- **3 seats를 쓴다**. 답이 한 전문성에 치우치지 않게 — agentic(seat1)·ATAM 아키텍트(seat2)·인프라(seat3) 관점을 명시(“seat1 관점에선…”).
- **근거·PoC를 경량으로라도 단다**. 자문이어도 “왜 맞나(레퍼런스)”와 “어떻게 증명하나(PoC 단서)”를 빼지 않는다 — 그게 red팀과 다른 blue의 의무.
- **추적성(중요)**: 자문에서 **채택할 결론**(설계 방향·신규 결정·결함)이 나오면, 다음 라운드의 **Reviewer 교차발견(C*)** 또는 **`_new-dp-candidates.md`(NDP)** 로 접어 넣어 근거를 남긴다. 잡담으로 증발시키지 않는다.
- **파일 경계**: consult는 파일 0개. revise는 *초안*까지(원본 쓰기는 승인 후). 라운드 산출물(`counsel/`)은 **정식 라운드에서만** 만든다.

> revise 모드가 원본을 고칠 때도 **§0 3요소·§9 별점 앵커·§3 필수요소**를 그대로 적용한다 — 자문이라고 근거 없는 즉흥 수정을 하지 않는다.

---

## B. 설계자 책임 — Cohesion(응집) + Coherence(정합)

> **Reviewer는 비판하고, Council은 *설계*한다.** Council이 내는 개선안·개정 초안·NDP는 사실상 DP 세트의 설계 행위다. 그러므로 Council은 인증 과정에서 시니어 아키텍트가 강조한 두 구조적 미덕을 **모든 산출(라운드 counsel + 자문 revise)의 합격 게이트**로 책임진다. 이 둘은 *층위가 다르며 따로 통과시켜야 한다*.
>
> **용어 주의**: 여기서 **cohesion은 고전 SE의 “모듈 내 단일책임”이 아니다.** 인증 과정 기준 cohesion = **DP 간 상호 연결·보완을 본문에 *명시*하는 미덕**(어원 *cohaerere* = 함께 붙다). DP의 *단일 결정·직교축 분리*(merged-DP1 교훈)는 별개로, Reviewer **Well-framed(framing)** 축에서 다룬다 — cohesion과 혼동 금지.

### B.1 Cohesion(응집) — DP가 서로 *연결*되어 있는가 (보완 관계를 명시)
DP는 고립된 섬이 아니다. 각 DP는 다른 DP와의 관계를 본문에 드러내 세트가 **상호 연결된 하나의 설계체**로 읽히게 한다. 체크:
- [ ] 이 DP에서 고른 안이 **특정 QA에서 경쟁안 대비 열위**라면, 그 열위를 **어느 DP가 보완**하는지 명시했나? (예: “1안은 QA-02 가용성에서 2안 대비 ★ 열위지만, **DP-03의 이중화·격리가 이를 보완**” → trade-off가 한 DP에서 ‘졌다’로 끝나지 않고 세트 차원에서 ‘여기서 닫힌다’로 추적됨.)
- [ ] **동시결정·의존**을 cross-link으로 적었나(이 결정은 DP-04와 함께 정해짐)?
- [ ] 이 DP의 **Risk가 다른 DP의 tactic으로 완화**되면 그 연결을 적었나?
- [ ] 인접 DP가 공유하는 가정·자원을 참조했나?
- → 효과: 독자가 어떤 trade-off의 해소 지점을 **세트 전체에서 추적**할 수 있다. (산출에 cross-link `DP-0X`로 grep 역참조.)

### B.2 Coherence(정합) — 연결된 전체가 *모순 없이 일관*된가
연결(B.1)이 있어도 그 연결들이 서로 어긋나면 incoherent다. 체크:
- [ ] **동시결정 무모순** — 함께 정해지는 DP가 서로 충돌 안 하나(DP-02 제어평면 ↔ DP-04 축B = 단일 제어평면, 이중 금지)?
- [ ] **공유 가정 무모순** — load-bearing 가정이 DP 간 충돌 안 하나(멱등 ↔ QA-11 비결정)?
- [ ] **일관성** — 용어·ID 참조·**별점 앵커(§9)** 가 전 DP에서 같은 기준인가?
- [ ] **공유 장애도메인 합산** — 여러 DP가 들이는 공유 컴포넌트 blast-radius가 합쳐서 ASR 신뢰성을 깨지 않나?
- [ ] **Traceability Matrix 완전성** — 전 ASR([`asr.md`](../../context/asr.md) 목록)이 cover되나(**orphan = 전체 설계의 정합 구멍**)?
- [ ] **단일 발표 서사** — DP들이 하나의 이야기로 쌓이나(1번 타자가 깔고 나머지가 얹힘)?
- → 위반 시 권고: 동시결정 일원화, 가정 통일, 별점 재앵커, orphan 메우는 NDP. (Reviewer **교차발견(C*)·Traceability Matrix** 와 직결.)

### B.3 관계 (왜 따로 보나)
- **Cohesion = 관계가 *명시*돼 있나**(연결의 **존재**). **Coherence = 명시된 관계가 *서로 일관*된가**(연결의 **무모순**).
- 둘은 독립이다: cross-link이 많아도 그 안에 모순(이중 제어평면)이면 *cohesive-but-incoherent* / 모순은 없지만 서로 참조 안 하는 고립 DP 더미면 *coherent-but-low-cohesion*. **둘 다 필요.**
- revise 모드(§A)에서 한 DP만 손대더라도, 그 변경이 **인접 DP 보완관계(cohesion)·세트 무모순(coherence)** 을 깨지 않는지 반드시 확인한다.

---

## C. 대안 고도화 (Alternative Maturation) — 비선택안도 tactic으로 강화해 비교

> 평가에서 진 대안을 **strawman으로 버리지 않는다.** 각 후보 대안은 *tactic을 더한 최선의 모습*으로 비교돼야 한다. 목표 스토리:
> **“비선택안 A는 QA-Z에서 ★☆☆지만, tactic T를 더하면 ★★☆까지 오른다 — 그럼에도 선택안 B(★★★) 대비 열위라 미채택.”**
> 이 정도로 각 DP의 설계안이 고도화돼야 결정이 *깊이*와 *robust함*을 증명한다. Council(설계자)의 핵심 의무다.

### C.1 왜 (무엇을 증명하나)
- **설계 공간을 끝까지 탐색**했음 — 약한 버전이 아니라 *강화된 버전*과 비교.
- **결정의 robust함** — steelman한 대안조차 이긴다(strawman을 이긴 게 아니다).
- **tactic 지식의 시연** — *어떤 tactic이 어떤 별을 얼마나 올리는지* 안다는 증거(§4 라이브러리 활용).

### C.2 어떻게 (Council 작업)
각 **비선택 대안 × 그 약한 ASR 칸**에 대해:
1. **강화 tactic 식별** — §4 라이브러리에서 그 ASR을 올릴 tactic(예: 가용성 약한 안에 Redundancy/Active-Passive, 확장 약한 안에 scale-to-zero).
2. **별 delta 명시(앵커)** — `★☆☆ → ★★☆ (tactic T 추가 시)`, §9 등급척도 경계에 앵커. 미측정이면 `◯`.
3. **그 tactic의 *자기 trade-off* 도 함께** — 공짜가 아니다. T가 다른 QA를 깎거나(예: Standby → QA-13 대기비용·QA-11 페일오버 일관성) 복잡도·비용을 더하면 명시.
4. **잔여 격차** — 강화 후에도 **선택안 대비 어디서 왜 열위인지**. 결론은 둘 중 하나:
   - *그럼에도 열위* → 미채택(결정 robust 증명).
   - *강화하니 역전* → 그 대안을 **재고/승격**(repo **DP-01 3안 = “1안 + Standby”** 가 바로 이 승격 사례 — 구 DP-0002 계승).

### C.3 연결·정직성
- **Cohesion(§B.1)**: 강화 tactic이 *다른 DP에서 채택*되거나, 비선택안의 잔여 약점이 *다른 DP로 보완*되면 그 연결을 적는다.
- **PoC(§5)**: “tactic T가 ★를 올린다”는 주장은 **별점 변별 PoC**의 증명 대상.
- **정직성**: 날조 금지 — 실재 tactic·실재 천장(§9 margin). 강화해도 *여전히 진다*를 실제로 보여야 한다(쉽게 역전되면 애초 선택을 재검토).

---

## D. DP 산출물 계약 — 슬라이드 1장으로 수렴

> 모든 DP는 최종적으로 **16:9 슬라이드 1장(많아야 2장)** 으로 표현된다(claude.ai/design 입력). 따라서 Council이 내는 DP 문서·산출물은 **처음부터 슬라이드-ready**로 구조화하고, 상세는 Appendix로 분리한다.

### D.1 산출물 3종
1. **대안별 SVG 도안** — 각 후보 대안의 구조/특징을 *한눈에*. **seats 1·3(도메인 전문가)이 WebSearch로 표준 아키텍처 다이어그램 관례**(orchestrator-workers · choreography/event-driven · active-passive failover · pipes-and-filters 등)를 참조해 그린다. 위치: DP를 **폴더로 승격**해 `context/dp/DP-0X/diagrams/<대안>.svg`.
2. **대안 × ASR 통합 매트릭스** — 행=대안, 열=ASR, 셀=`★등급 (+ S/T/R/N)`. trade-off(별점·정도)와 traceability(ATAM 성격)를 **한 격자에 통합**. **선택안의 행이 세트 Traceability Matrix(Reviewer §5)로 graduate.**
3. **슬라이드 요약 + Appendix** — 문서를 두 층으로(아래 D.3).

### D.2 슬라이드 레이아웃 (대안별 column)
슬라이드는 **대안마다 한 column**, 각 column은 위→아래로:
- (a) **상단**: 그 대안의 **도안(SVG)**
- (b) **중단**: **간단한 설명**(1–2줄)
- (c) **하단**: 그 대안의 **별점**(행=ASR, *그 대안 column*의 ★) — 별점표는 **각 대안 column 하단에 정렬**(전체로 읽으면 행=ASR·열=대안).
- + **권고 1줄**(선택안·드라이버).

### D.3 문서 구조 (slide-ready)
```
context/dp/DP-0X/
├── DP-0X-<slug>.md
│   ├─ ## 용어 (비자명 표현)    ← 독자용 선행 설명(슬라이드 본문 아님)
│   ├─ ## 슬라이드 요약 (1장)   ← 대안 column(도안+설명+별점) + 권고. 슬라이드 소스.
│   └─ ## Appendix             ← 결정 드라이버 · 대안 전문 · 대안×ASR(★+S/T/R/N)
│                                 · ATAM(SP/TP/R/N) · 대안 고도화(§C) · Cohesion(§B) · 별점 앵커 근거
└── diagrams/ <대안>.svg
```
- **용어 먼저**: 비자명 용어(orchestration·choreography·SPOF·failover 등)는 슬라이드 요약 앞 `## 용어`에 한 줄씩(읽는 법·비유 포함) — 발표 청중·팀원 가독성. 슬라이드 본문이 아니라 독자용 선행 설명.
- **슬라이드 요약은 Appendix와 모순 없어야**(coherence §B.2) — 요약의 별점·권고는 Appendix 근거에서 도출.
- **1장 수렴이 원칙**; 2장은 대안 수·도안 복잡도가 클 때만 예외.

---

## 1. 입력 (권고 전 반드시 읽을 것)

- **해당 라운드 `round-NN/review/` 전체** — `report.md`(Traceability Matrix·orphan ASR·red 결론), `DP-0X-*.md`(렌즈별 지적·Stage 2 권고), `_new-dp-candidates.md`. **Reviewer 의견을 빠짐없이 읽는 것이 1번 의무.** (자문 모드면 해당 DP 원본 + 관련 review만.)
- `context/dp/` 원본 DP(+ 폴더 승격 DP의 하위 `evaluation.md`·`decision-axes.md`·`approaches`·`research/R-*`) — 특히 **dp4의 R-01~R-05 리서치는 재사용 가능한 근거 자산**이다.
- `context/qa/`의 ASR QA(목록: [`context/asr.md`](../../context/asr.md)) — **각 QA의 등급 척도(★ rubric)가 DP 별점 앵커의 기준선**(§9). KPI 임계값도 여기서.
- `context/requirements/`(realizes FR·Constraint) · `context/overview.md`·`glossary.md`·`INDEX.md`·`open-issues.md`(OI-*).
- 직전 라운드 `counsel.md`(있으면 — 권고 추세).

---

## 2. 3 seats (Reviewer 3렌즈와 동형의 합의체)

세 석이 각자 발의하고 합의/소수의견을 표시한다. (Reviewer 렌즈와 1:1 — red가 깐 자리를 같은 전문성으로 blue가 메운다. 렌즈2=대장이 coverage·KPI·ATAM 총괄.)

| seat | 전문성 | 주로 메우는 약점 |
|---|---|---|
| **Seat 1** | Agentic Workflow 전문가 | 식상하지 않은 agentic 정석(eval 하네스·self-consistency·HITL) 발굴, 비결정↔멱등 모순, 토큰 경제 |
| **Seat 2 (대장)** | 20년차 ATAM 평가 수석 아키텍트 | **세트 ASR coverage(orphan 메우기·NDP)**·KPI 앵커·결정 framing(2×2)·SP/TP/Risk·수렴 |
| **Seat 3** | 대규모 Workflow Runner 인프라 아키텍트 | durable execution·오토스케일·격리·blast-radius/FMEA·데이터 평면·운영 PoC |

- 각 권고에 **발의 seat**과 **합의 여부**(consensus / Seat n dissent)를 표기.

---

## 3. 권고 3대 필수 요소

모든 DP 개선 권고는 아래 셋을 **반드시** 포함한다. 하나라도 빠지면 미완성.

1. **개선안** — §0-1의 항목 중 해당하는 것을 `기존 → 제안`으로. 미정 수치는 `◯` + 산출 근거.
2. **근거(레퍼런스)** — 권장 tactic/대안/별점이 패턴 카탈로그·논문·필드에서 어떻게 쓰이는지 + 출처. §4 라이브러리 먼저.
3. **PoC 증명법** — §5 템플릿으로 1개 이상. “이 설계가 주장한 별점/결정 단일질문을 달성·측정 가능함을 어떻게 보일 것인가”.

---

## 4. Tactic·Pattern 레퍼런스 라이브러리 (ASR 유형별 — 라운드 무관 canon)

**ASR(품질속성) 유형별로** 그 QA를 끌어올리는 표준 tactic/pattern + 근거 + PoC 수단을 묶어둔다. **재사용 자산**(매 라운드 재유도 불필요). 더 나은(식상하지 않은) 대안을 발굴하거나 별점을 정당화할 때 여기서 끌어 쓰고, 새 유형은 검색해 추가한다. (DP 별점이 *그 QA를 얼마나 만족하나*의 비교이므로, KPI 임계값은 QA Council [§4 KPI 라이브러리](../qa/Council.md)·각 QA 등급 척도를 함께 본다.)

> ⚠️ **경계**: 이 라이브러리는 “어떤 tactic이 표준인가”만 담는다. *“어느 tactic을 DP-XX의 어느 약점에 적용할지”*는 그 라운드 `counsel/`에서 결정한다.

| ASR 유형 | 권장 tactic / pattern | 근거 (레퍼런스) | PoC 증명 수단 |
|---|---|---|---|
| **QA-01 Scalability** | Competing Consumers·**scale-to-zero/FaaS**·backlog 기반 오토스케일(CPU 아님)·stateless filter·bin-packing; USL로 한계 모델링 | Azure Competing Consumers · KEDA · USL(Gunther) · dp4 R-01/R-02 | 부하 N배↑ 시 **scaling efficiency**(rate-limit 헤드룸 고정) + KEDA 큐-깊이 스케일 |
| **QA-02 Availability** | **Redundancy(Active-Passive/Active-Active)+heartbeat+leader election**·상태 외부화·**durable execution(event sourcing·exactly-once)**·bulkhead·circuit breaker | Temporal durable execution · Google SRE · dp4 R-03(saga) | 노드/Orchestrator 강제종료(chaos) → **재개 시간·in-flight 손실=0** |
| **QA-03 Controllability** | **Orchestration vs Choreography**(단일 제어평면)·중앙 **정책 집행점(PEP)**·**HITL 승인 게이트**·**runaway cap**(max iter/token/wall-clock)·협조적 취소+hard-kill | Anthropic *Building Effective Agents* · 신뢰 에이전트 5원칙 · dp4 R-03 | 중단 주입 → **ack·graceful stop+rollback 시간**; runaway 주입 → cap 발동 |
| **QA-04 Observability** | **OTel GenAI semconv**(invoke_agent/tool span + gen_ai.usage)·event sourcing→trace 완전성·anomaly detection | OpenTelemetry GenAI semconv | 계측 후 **event-history/trace 완전성 %** |
| **QA-05 Efficiency** | **prompt caching**·모델 티어 right-sizing·warm pool·배칭·토큰 budget | Anthropic prompt caching·pricing | 캐시 On/Off **A/B로 $/task·TTFT** |
| **QA-06 Security/Safety** | **zero-trust 최소권한**·sandbox/isolation·broker/proxy·allowlist·**가드레일 모델**·injection 방어·secrets 관리·**공급망 서명(SLSA)** | Anthropic 신뢰 에이전트 프레임워크 · zero-trust(NIST) · SLSA | **적대적 eval 세트**로 허용범위 위반·injection 차단율; 서명 검증 |
| **QA-07 Correctness** | **golden dataset(50~100) + LLM-as-judge(다수결)**·결정성 고정(seed)·exactly-once 효과·재현 검증 | golden-dataset·LLM-as-judge BP · τ-bench | golden set → judge 인간 일치 검증 후 정답률; **pass^k** |
| **공통(실행구조)** | **Claim-Check**·**Data-Locality/Co-location**·Pipes-and-Filters·**Saga**·Microkernel/Plugin·Event Sourcing | Azure Claim-Check/Pipes-Filters · K8s Pod Affinity/Local PV · dp4 R-04/R-05 | 전달비용 `(단계×산출물÷대역폭) vs budget`; locality A/B |

> ATAM 절차 자체의 근거(utility tree·sensitivity/tradeoff point·risk theme)는 **SEI ATAM**을 따른다(레퍼런스 절). 별점·수치는 PoC로 직접 확정 — 레퍼런스 수치를 베끼지 않는다.

---

## 5. PoC 설계 가이드

각 권고는 **증명 가능**해야 한다. QA Council [§5](../qa/Council.md)의 한 장 템플릿·PoC 아키타입(부하/chaos/격리/계측/eval/반복/비용 A/B)을 그대로 쓰되, DP에선 **가설이 “이 *설계*가 주장한 별점/결정 단일질문을 만족한다”** 가 된다.

> ⚠️ **경계**: 방법(템플릿·아키타입)만 canon. *구체 PoC*(어느 DP의 어느 별점/단일질문을 무엇으로 증명)는 그 라운드 `counsel/_poc-plan.md`.

```
### PoC-<번호>: <어느 DP의 무엇을 증명하나>
- 가설: "<DP-0X 안 Y>가 <QA-Z>에서 ★★★(=등급척도 상)을 달성한다" 또는 "<결정 단일질문>이 A안으로 갈린다"
- 지표 + 합격선: <QA-Z 등급척도 경계값> (예: scaling efficiency ≥0.70 / 재기동 ≤4분·손실=0 / 전달 ≤5%)
- 셋업: 도구·데이터·환경 (chaos / k6 / golden set / 캐시 A/B / 전달 산식 …)
- 절차: 단계 1~3
- 합격 기준(Exit): 무엇을 보면 “이 별점이 정당”인가 / 어느 쪽으로 결정 갈림
- 규모/기간 · 리스크/한계(PoC가 못 보는 것 = silent cap 명시)
```

> **DP 특유 PoC 2종**을 잊지 말 것: ① **별점 변별 PoC** — 두 대안이 같은 QA 칸에서 *실제로 갈리는지*(변별력 0 해소). ② **결정 단일질문 PoC** — 결정을 가르는 산식/측정(예: dp4의 5% 전달 산식)을 실측해 택일을 확정.

---

## 6. 개별 DP 권고 파일 포맷 (`counsel/DP-0X-<slug>.md`)

```
# Counsel: DP-0X <결정명>

> refs-review: round-NN/review/DP-0X-*.md
> seats: 발의 <Seat n> · 합의 <consensus / Seat m dissent>
> stance: <재framing / 대안 추가 / 별점 앵커 / ATAM 보강 / NDP 신설 / 보류>

## Reviewer 지적 요약        — red가 무엇을 깠나 (ASR cover·KPI·식상성·ATAM, 1~3줄)
## 개선안                   — §0-1 항목 `기존 → 제안` (framing·대안·별점·ATAM·수렴)
## 근거 (레퍼런스)          — §4 라이브러리 + 출처 URL (왜 이 tactic·이 별점)
## PoC 증명법               — §5 템플릿 1개 이상 (별점 변별 / 결정 단일질문 포함)
## ASR·발표 영향            — Traceability Matrix 변화(orphan 해소?) / 1번 타자·번호 영향
```

- cross-link은 ID 텍스트(`DP-02`, `QA-06`, `NDP-A`, `FR-0004`, `OI-7`)로. 레퍼런스는 URL.
- orphan ASR을 메우는 **신규 결정(NDP)** 권고는 `counsel/_new-dp-counsel.md`(또는 해당 NDP 파일)에 결정 포인트·후보 대안·driving ASR·근거·PoC까지.

---

## 7. 라운드 산출물 (blue team 몫은 `round-NN/counsel/`)

| `round-NN/counsel/` 파일 | 역할 |
|---|---|
| `counsel.md` | **이것만 읽으면 권고 전모를 아는 종합 보고서** — 채택 권고표·핵심 근거·**orphan ASR 메우는 NDP**·PoC 로드맵·직전 대비 변화. **날짜 등 메타 기록**(공의회 최종 문서). |
| `README.md` | counsel 폴더 내비게이션 |
| `DP-0X-*.md` | DP별 개선 권고 (§6 포맷) |
| `_new-dp-counsel.md` | 신규 결정(NDP) 권고 — orphan ASR·미발굴 축 |
| `_poc-plan.md` | PoC 통합 계획 — 우선순위·의존성·총 기간 |

상위 [`README.md`](README.md) 마스터 인덱스에 권고 추세·verdict 변화를 갱신.

---

## 8. 관리 규칙 / 실행 절차

**관리**: `review/`와 동일 **append-only**. Council 권고가 `context/dp/`에 반영되면 DP `updated:` + `changelog.md` 델타, 다음 라운드 추세 갱신. DP는 2자리 ID(`DP-02`), 발표 우선순위가 곧 번호는 아니나 **1번 타자(face) 배치**는 권고에 명시(ASR·교환점 강도 기준).

**절차 체크리스트**:
1. §1 입력 — **해당 라운드 review/ 전부** + context(dp·qa 등급척도·requirements) 읽기.
2. DP별 §6 포맷으로 **개선안 → 근거 → PoC** 3요소. 3 seats 발의·합의 표기.
3. **orphan ASR**은 NDP 권고(결정 포인트·대안·driving ASR·근거·PoC)로 `_new-dp-counsel.md`.
4. **별점 앵커링**(§9)을 각 ★ 칸에 적용.
5. `counsel.md`(종합·메타) + `_poc-plan.md` + counsel README. 마스터 인덱스 추세 갱신.
6. 레퍼런스 수치 직접 베끼지 않았는지·PoC가 못 본 부분 log했는지 확인.

---

## 9. ★ 별점 앵커링 — DP 별점을 QA 등급 척도에 묶기

> DP의 Trade-off 매트릭스 ★는 “이 대안이 그 QA를 얼마나 만족하나”의 절대 비교다. **그 ★의 경계는 QA Council [§9 등급 척도(★ rubric)](../qa/Council.md)가 캘리브레이션한 각 QA의 등급 척도와 동일해야 한다** — DP가 독자적으로 별을 매기면 Reviewer의 “KPI 미앵커·변별력 0” 지적을 못 푼다. Council의 핵심 임무 하나가 이 앵커링이다.

### 9.1 규칙
1. **★ = QA 등급 척도 구간**. DP 매트릭스의 `대안 A × QA-Z = ★★☆`는 “A가 QA-Z 등급척도의 ★★☆ 구간(예: 재기동 ≤◯분)을 만족”을 뜻한다. 칸 옆/주석에 **그 QA의 등급척도 경계값을 인용**(앵커).
2. **변별력 검사**. 한 QA 칸에서 모든 대안이 같은 ★이면(변별 0) → 그 QA 등급척도의 *더 세분된 보조지표*로 쪼개거나, 그 QA가 이 결정의 변별축이 아님을 명시(다른 QA로 헤드라인 이동).
3. **미측정은 ★ 금지, `◯`**. 검증 안 된 별점(“추정”)은 ★가 아니라 `◯`로 두고 PoC로 확정 — 추정 별점이 비교축처럼 보이는 것을 막는다(Reviewer의 “3안 별점 추정” 지적 해소).
4. **2-index 앵커**. QA KPI가 2지표면(A·B), main 급간 축(A)으로 ★ 매기고 조건(B 고정)을 칸 주석에(“B=손실0 고정 하에 A 재기동시간으로 ★”).
5. **조합 별점**. 여러 대안을 함께 채택하는 스택은 개별 ★의 합이 아니라 **조합 별점을 따로** — 누적 비용(전달세금·공유 장애도메인)을 반영(Reviewer 조합/창발 지적).

### 9.2 운영
- seats(§2)·레퍼런스(§4)·PoC margin 정신은 표준 council과 공유. **1~2개 샘플 먼저 확인 후 fan-out**(프로젝트 규칙).
- 반영 시 DP `updated:` + `changelog.md` 델타. QA 등급척도가 라운드로 바뀌면 DP 별점 앵커도 재정합(추적성).

---

## 레퍼런스 (라이브러리 출처)

- SEI ATAM(utility tree·sensitivity/tradeoff/risk): https://insights.sei.cmu.edu/library/architecture-tradeoff-analysis-method-collection/
- Azure Architecture Patterns(Competing Consumers·Claim-Check·Pipes-and-Filters): https://learn.microsoft.com/azure/architecture/patterns/
- Temporal durable execution / Saga: https://temporal.io/blog/what-is-durable-execution , https://temporal.io/blog/to-choreograph-or-orchestrate-your-saga-that-is-the-question
- KEDA 이벤트 기반 오토스케일: https://keda.sh/
- Universal Scalability Law: https://wso2.com/blog/research/measuring-software-scalability-using-universal-scalability-law/
- Anthropic *Building Effective Agents* / 신뢰 에이전트 프레임워크: https://www.anthropic.com/engineering/building-effective-agents , https://www.anthropic.com/news/our-framework-for-developing-safe-and-trustworthy-agents
- Anthropic prompt caching·pricing: https://platform.claude.com/docs/en/build-with-claude/prompt-caching
- OpenTelemetry GenAI semantic conventions: https://opentelemetry.io/docs/specs/semconv/gen-ai/
- τ-bench(pass^k)·LLM-as-judge·golden dataset: https://arxiv.org/abs/2406.12045
- K8s Pod Affinity·Local PV(data locality): https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/
- SLSA(공급망 서명): https://slsa.dev/
- dp4 리서치 R-01~R-05: `context/dp/dp4/research/`
