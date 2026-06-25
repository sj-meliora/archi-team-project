# Reviewer — DP 리뷰 재생산 스펙

> 목적: reviewer agent가 **이 디렉터리에서 확립할 DP 리뷰 방식을 그대로 재생산**할 수 있게 하는 단일 지침.
> 적용 대상: `context/dp/`의 DP(설계결정)·후보 대안·★ Trade-off 매트릭스·문서 내부 ATAM 분석.
> 한 번의 실행 = **1 라운드** = DP 전체 스냅샷 판정 + report.
> 짝 스펙: [`Council.md`](Council.md)(blue) · [`Applier.md`](Applier.md)(반영) · 마스터 인덱스: [`README.md`](README.md)

---

## 용어 (이 스펙에서 쓰는 표현)

> 본 스펙은 그 자체로 ATAM 기반 설계결정 리뷰 방법론 산출물이다. 아래는 본문에서 반복되는 핵심 용어 — **ATAM 표준 용어**와 **본 스펙의 약식 표현**을 구분한다.

### ATAM 표준 용어 (Architecture Tradeoff Analysis Method, SEI)
| 용어 | 뜻 |
|---|---|
| **ASR** | Architecturally Significant Requirements — 아키텍처를 좌우하는 핵심 요구. 본 과제에선 **ASR = QA-01~07**(우선순위 상위 품질속성, *DP 생성 동인*). |
| **Sensitivity Point (S, 민감점)** | 한 설계 파라미터가 특정 QA 응답을 좌우하는 지점. 그 값을 바꾸면 그 QA가 크게 움직인다. |
| **Tradeoff Point (T, 교환점)** | 둘 이상의 QA에 동시에 민감한 지점 — 한 QA를 올리면 다른 QA가 내려가는 교환이 일어나는 곳. ATAM 분석의 핵심. |
| **Risk (R, 위험)** | 그대로 두면 품질 목표(KPI) 미달로 이어질 수 있는 우려되는 결정. |
| **Non-Risk (N, 비위험)** | 분석 결과 안전하다고 판단된 — 해당 QA를 무리 없이 충족하는 결정. |
| **Traceability Matrix** | 설계결정(DP)과 ASR의 대응을 한 표로 추적하는 매트릭스. 본 스펙에선 **행=DP, 열=ASR, 셀=S/T/R/N**. 누가 무엇을 푸는지·빠진 것은 없는지를 한눈에 본다. |
| **driving QA** | 그 DP가 실현/교환하려는 주(主) 품질속성. DP 헤더의 `drives:`에 선언. |
| **realizes (FR)** | 그 DP가 구현하는 기능요구(Functional Requirement). |
| **tactic / pattern** | 품질속성을 달성하는 설계 수단 — tactic은 단위 기법(예: Redundancy, Bulkhead), pattern은 그 조합 구조(예: Pipes-and-Filters). |
| **MVA** | Minimum Viable Architecture — 최소 실행가능 아키텍처. “다 좋은” 스택 대신 **무엇을 빼고도 성립하는 최소 골격**. ATAM의 본령 = 버릴 것을 정하기. |

### 본 스펙의 약식 표현
| 용어 | 뜻 |
|---|---|
| **orphan ASR** | Traceability Matrix에서 **열 전체가 빈칸인 ASR** — 어느 DP도 풀지 않는 ASR. 설계 세트의 구멍이며 **최상위 결함**. 신규 DP(NDP)로 메운다. |
| **KPI 앵커(앵커링)** | ★ 별점을 막연히 매기지 않고 **그 QA의 실제 KPI 임계값**(예: “전달 오버헤드 ≤ E2E 5%”)에 근거를 묶는 것. 앵커 없는 별점은 비교 불가. |
| **변별력** | 대안들이 한 QA 칸에서 **서로 다른 점수**로 갈리는 정도. 모든 대안이 같은 별점 = 변별력 0(그 축으로는 아무것도 못 가린다). |
| **load-bearing 가정** | 결론을 떠받치는 **결정적 전제**(예: “모든 단계가 멱등”). 무너지면 그 위의 안전성 주장이 통째로 무너지므로 1급으로 노출·검증해야 한다. |
| **blast-radius** | 한 컴포넌트 장애가 번지는 **영향 범위**(예: 공유 브로커 다운 시 중단되는 Workflow 비율). 공유 컴포넌트가 늘수록 커진다. |
| **claim-check** | 큰 산출물(예: 20GB)을 스토리지에 두고 **참조 토큰만 전달**하는 메시징 패턴. 메시지는 가벼워지나 단계마다 스토리지 왕복(전달세금)이 생긴다. |
| **이중 제어평면(dual control plane)** | 중앙 오케스트레이션(명령)과 분산 choreography(자율)를 **동시에** 두어 제어 권한이 두 곳에 갈리는 안티패턴. |
| **null/기준선(baseline) 대안** | 비교의 바닥으로 남겨두는 **현행유지/최단순 대안**. 신규안이 이걸 정말 이기는지 대조하는 기준. |
| **NDP / NQA** | New DP / New QA 후보 — 리뷰가 발굴해 권고하는 신규 설계결정·품질속성(채택은 팀/Applier 몫). |
| **findings F-\*** | dp4의 기존 ATAM 리뷰(`context/dp/dp4/review/findings.md`)의 발견 항목(F-01~F-12) — 본 스펙의 결함 유형 선례로 인용. |

---

## 0. 리뷰의 목표 (무엇을 판정하나)

**DP는 ASR(Architecturally Significant Requirements)을 실현하려고 존재한다.** context 기준 **ASR = QA-01~07**(우선순위 상위 7개 — 곧 *DP 생성 동인*. `changelog.md`·`INDEX.md`). 그러므로 DP 리뷰가 **가장 먼저·가장 무겁게** 보는 것은 ASR cover다 — 이는 **두 층위**로 본다:

0. **세트 전체 coverage (대장 = 렌즈2 고유 책임) — 모든 DP를 합쳤을 때 ASR(QA-01~07) 전부가 어느 DP엔가 cover되는가?** 개별 DP가 각자 ASR을 잘 풀어도, **어느 DP에도 안 잡힌 ASR(orphan ASR)이 있으면 안 된다.** 세트 전체로 **Traceability Matrix(행=DP, 열=ASR, 셀=S/T/R/N)**를 그려 빈 열(미커버 ASR)을 드러내는 것이 DP 디스커션의 최상위 점검이다(→ §5). 빈 열은 신규 DP(NDP)로 메운다.

1. **개별 ASR-coverage — 이 DP가 자신이 driving하는 ASR QA를 실제로 cover하는 설계인가?**
   - 결정 포인트와 후보 대안이 그 ASR QA의 관심사를 **실제로 움직이는가**(별점이 그 QA 칸에서 갈리는가), 아니면 이름만 걸어두고 정작 그 QA를 좌우하지 못하는가?
   - driving QA로 선언된 ASR이 **맞는** ASR인가? 이 결정이 응당 cover해야 할 ASR QA가 **빠지지 않았는가**(예: 멱등 가정이 QA-07 Correctness/QA-11 일관성을 건드리는데 driving에서 누락, findings F-05/F-12)?
   - 비-ASR QA(QA-08~13)에만 강점이 쏠려 **정작 ASR은 ★★☆ 평이**한 대안을 권고하고 있지 않은가(F-07 최저중요도 과최적화의 ASR판).

2. **KPI-comparison — 각 driving QA의 주요 KPI가 대안 비교의 축으로 제대로 비교되는가?**
   - ★ Trade-off 매트릭스의 각 칸이 **그 QA의 실제 KPI**(예: QA-10 “전달 오버헤드 ≤ E2E 5%”, QA-08 “타 Workflow 중단 ≤1%”)에 **앵커**돼 있는가, 아니면 막연한 별점인가?
   - 그 KPI 축에서 대안들이 **변별**되는가(한 칸이 전부 동일 = 변별력 0, F-01)? 별점이 인용한 임계값이 **QA 원문과 일치**하는가?
   - 조합 채택 시 그 KPI가 **누적으로 깨지지** 않는가(개별 ★★☆의 합 ≠ 조합, F-02/F-04).

즉 **coverage(세트+개별) + KPI 비교**가 1순위 판정이고, 이를 통과한 뒤에 ATAM 형식 건전성(framing·SP/TP/Risk·수렴)을 본다. DP 리뷰 = **“모든 ASR이 빠짐없이 cover되고, 각 ASR을 옳게 풀며, 그 KPI로 정직하게 비교하는가”를 ATAM 절차로 검증**하는 일이다.

> **렌즈 분담**(§3): 대장 **렌즈2**가 coverage(세트 orphan 점검)·KPI·ATAM 절차를 총괄하고, 도메인 전문가 **렌즈1·3**은 *각 DP 설계가 도메인적으로 make sense하고 식상하지 않은가(난이도 있는 문제를 설득력 있게 푸는가)·trade-off 분석이 타당한가*를 본다.

산출물은 “정답”이 아니라 **Applier 단계(실제 DP 수정)의 근거**다. 권고는 `기존 → 제안` 형태로 적어 적용 가능하게 한다.

> **단일 책임**: Reviewer는 **라운드 전용 적대적 비평자**(verdict를 낸다)다. 라운드 밖의 건설적 자문("어느 게 나을까/어떻게 풀까")은 **Council disposition** 소관 — `discussion/README.md` §자문(advisory) 모드 참조. Reviewer는 명시적 red-team("부숴봐") 요청에만 비-라운드로 응한다.

> 참고 prototype: `context/dp/dp4/review/findings.md`(F-01~F-12)가 사실상 본 스펙의 선례다 — 범주오류(F-08)·이중계상·조합효과 미산정(F-02)·핵심 교환점 은폐(F-04)·load-bearing 가정 모순(F-05)·수렴 부재(F-10)가 곧 DP 리뷰의 단골 결함 유형이다.

---

## 3. 평가 렌즈 (3종 — 반드시 셋 다 적용)

리뷰어는 아래 세 전문가 페르소나를 **각각 독립 섹션**으로 적용한다. 한 렌즈가 다른 렌즈를 대신하지 않는다. (QA Reviewer 3렌즈와 동형 — 같은 전문성을 DP/ATAM에 적용.)

**역할 분담** (DP 리뷰의 핵심):
- **렌즈 2 = 리뷰어 대장(lead)** — *coverage·KPI·ATAM 절차*를 총괄한다. 특히 **세트 전체 ASR coverage**: 개별 DP가 ASR을 잘 풀어도, **모든 DP를 합쳤을 때 ASR(QA-01~07) 중 어느 DP에도 커버되지 않는 것(orphan ASR)이 있으면 안 된다.** 개별 cover(§0-①)는 각 DP에서, 집합 cover(orphan 점검)는 렌즈2가 세트 전체 **Traceability Matrix**(§5)에서 본다.
- **렌즈 1·3 = 도메인 전문가** — *설계 자체가 make sense 하는지*를 본다. 설계 approach가 도메인적으로 **말이 되는가**, 그리고 **너무 식상하거나 뻔하지 않은가** — DP는 *난이도 있는 문제를 설득력 있게* 풀어야 한다(자명한 1차 해법·교과서 패턴 단순 적용은 발표 산출물로 약하다). 더해 **각 설계의 trade-off 분석이 타당한지**(장점·단점·별점이 도메인 현실과 맞는지)를 중점적으로 본다.

### 렌즈 1 — Agentic Workflow 최상위 전문가 (Anthropic/OpenAI 급) — 도메인 전문가
**주임무: 설계가 agentic 도메인에서 make sense 하고, 식상·자명하지 않으며, trade-off 분석이 타당한가.** 점검 축:
- **설계 타당성·설득력**: 선택한 approach가 agentic 시스템에서 **말이 되는가**? 너무 **뻔한 1차 해법**(예: "그냥 중앙 오케스트레이터 두기")에 머물지 않고, 난이도 있는 문제(비결정·일관성·토큰 경제)를 **설득력 있게** 푸는가? 더 나은 agentic 정석(eval 하네스·self-consistency·HITL gate 설계)이 누락됐는가?
- **trade-off 타당성**: 각 대안의 장점/단점·별점이 agentic 현실과 맞는가 — 과장·누락된 단점은 없는가?
- 선택한 tactic/pattern이 **agentic 고유 제약**과 충돌하지 않는가 — 비결정성·환각·self-consistency가 retry/replay/멱등 가정을 깨지 않는가(F-05형 모순).
- rate limit(TPM·RPM)·토큰 경제·prompt caching·context 관리가 대안 비교에 반영됐는가 (특히 비용·HITL 경유 대안).
- 외부 제공자 의존(장애·버전)·도구 사용·runaway loop가 ATAM Risk로 잡혔는가.
- “일반 분산시스템 어휘”(큐·캐시·오케스트레이션)로 뭉개진 곳에 **agentic 고유 trade-off**가 빠졌는지.

### 렌즈 2 — 20년차 ATAM 평가 수석 아키텍트 (대장 — coverage·KPI·ATAM 총괄)
**리뷰어 대장.** 최우선(§0의 두 축)은 ASR을 옳게 **cover**하고 그 **KPI로 정직하게 비교**하는가, 그리고 **세트 전체에 orphan ASR이 없는가**다. 그 다음 ATAM 형식 완성도를 본다. 점검 축:
- **🔑 세트 전체 ASR coverage (대장 고유 책임)**: 모든 DP를 합쳤을 때 **ASR(QA-01~07) 전부가 어느 DP엔가 cover되는가?** 어느 DP에도 안 잡힌 **orphan ASR**이 있으면 그 자체가 최상위 결함(설계 세트의 구멍) → 세트 전체로 **Traceability Matrix(행=DP, 열=ASR, 셀=S/T/R/N)**를 그려 빈 열을 드러낸다(§5). 신규 DP 필요 시 NDP로 제기.
- **개별 ASR-coverage**: driving으로 선언된 ASR QA가 맞는가, 빠진 ASR이 없는가? 후보 대안이 그 ASR QA를 **실제로 갈라놓는가**(별점이 그 칸에서 변별)? 비-ASR(QA-08~13) 강점에 쏠려 ASR이 평이하지 않은가?
- **KPI-comparison**: ★ 매트릭스 각 칸이 **그 QA의 실제 KPI 임계값에 앵커**돼 있나(막연한 별점 아님)? 그 KPI 축에서 대안이 **변별**되나(한 칸 전부 동일 = 변별력 0, F-01)? 별점이 인용한 수치가 **QA 원문과 일치**하나?
- **Well-framed**: 진짜 아키텍처 결정인가? 결정축이 **직교**하게 분리됐나(여러 직교 축을 단일 택1로 뭉개지 않았나 — dp4 4축 통찰)?
- **대안 완전성·공정성**: 대안 공간이 충분한가(지배되는 기준선/null 포함)? **범주오류·이중계상**이 없나(variety를 Scalability로 표기 등, F-08)?
- **대안 고도화(strawman 아님)**: 비선택 대안이 *tactic-강화된 최선*으로 비교됐나 — "이 안은 ★☆☆지만 tactic T 추가 시 ★★☆, 그럼에도 선택안 대비 열위"식 고도화가 있나, 아니면 약한 채로 버려졌나? (강화하면 역전될 대안을 strawman으로 기각하지 않았나 — Council [§C 대안 고도화](Council.md)가 생산, Reviewer가 점검.)
- **슬라이드 수렴·도안(발표 산출물)**: DP 문서가 슬라이드 1장(많아야 2장)으로 수렴하나 — `## 슬라이드 요약`(대안 column: 도안+설명+별점)이 있고 **Appendix와 모순 없나**(coherence)? **대안별 SVG 도안**이 있나? (Council [§D 산출물 계약](Council.md).)
- **조합/창발 비용**: 여러 대안을 *함께* 채택할 때의 **부정적 상호작용**(이중 제어평면 F-03, 분산 인프라 동물원 F-02, 공유 장애도메인 누적 F-09)을 산정했나? 개별 ★점수의 합 ≠ 조합 프로파일.
- **핵심 교환점 노출**: 본 결정의 진짜 TP가 “순수 이득”으로 위장되지 않았나(claim-check 전달세금 F-04)? load-bearing 가정(멱등 등)이 한 줄로 묻히지 않았나?
- **가중**: driving QA 중요도(H/M/L)가 평가에 반영됐나, 최저중요도 QA에 과최적화되지 않았나(F-07)?
- **★ 등급 척도 경계 현실성**: ★☆☆/★★☆/★★★ 급간 경계가 필드 현실에 맞나 — 하한이 사문화되지 않는가? (경계의 캘리브레이션은 Council §등급 척도가 처리, 필드 벤치마크 근거는 렌즈 1·3이 보강.)

### 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 (구현 현실성) — 도메인 전문가
**주임무: 설계가 인프라 도메인에서 make sense·구현 가능하고, 식상하지 않으며, trade-off 분석이 타당한가.** “이 결정을 **runner 아키텍처로 실제로 만들고·운영하고·측정할 수 있나**?”를 묻는다. 점검 축:
- **설계 타당성·설득력**: approach가 인프라적으로 **말이 되는가**? **뻔한 해법**(단일 공유 풀·동기 호출)에 머물지 않고 난이도 있는 확장·격리·내구 문제를 **설득력 있게** 푸는가? 더 나은 인프라 정석(durable execution·backpressure·data locality)이 누락됐는가? 각 대안의 장점/단점·별점이 인프라 현실과 맞는가(**trade-off 타당성**)?
- **구현 환원**: 추상 tactic을 구체 메커니즘으로 환원(durable execution=Temporal류, 큐=competing consumers/claim-check, 격리=bulkhead/쿼터, 페일오버=Active-Passive+heartbeat). 그 메커니즘이 별점 주장을 실제로 떠받치나?
- **blast-radius / FMEA**: 공유 컴포넌트(브로커·오케스트레이터·공유 캐시)가 죽으면 영향 범위(% Workflow 중단)는? QA 신뢰성 KPI(중단 ≤1% 등)를 충족하나(F-09)?
- **데이터 평면**: artifact 전달 비용(claim-check 20GB 왕복·data locality·co-location)이 Performance KPI 예산(E2E 5% 등) 안에 드나(F-04)?
- **확장·격리·제어**: backlog 오토스케일, warm pool/cold-start, 쿼터·동시성 제한, circuit breaker, 협조적 취소+hard-kill이 대안 실현에 필요한가.
- **채택/운영 리스크**: 신규 인프라 수·요구 역량·비용 모델이 ATAM Risk에 있나(F-11)? — 런타임 QA만큼 결정에 중요.
- 각 렌즈3 말미에 **구현·측정 메커니즘**(검증 가능한 형태: PoC 산식·계측 지표)을 제시한다.

> 렌즈3는 추상 비판이 아니라 **“어떻게 만들/측정할 것인가”의 구현적 답**을 주는 자리다. 가능하면 구체 메커니즘과 **정량 산식 1개**(예: `(단계수 × 20GB 왕복) ÷ 대역폭` vs 5% budget)로 환원한다.

---

## 1. 입력 (리뷰 전 반드시 읽을 것)

- `context/dp/DP-*.md` (대상 전체) + 폴더 승격된 DP의 하위(`dp4/approaches/`·`evaluation.md`·`decision-axes.md`·`research/`·`review/`)
- `context/dp/_backlog.md` (후보 대안·미검증 trade-off 풀 — BL-*)
- `context/qa/QA-*.md` (driving QA의 **KPI가 곧 ★별점·Risk·SP의 기준선** — DP가 인용한 임계값이 QA 원문과 일치하는지, 현실적인지 대조)
- `context/requirements/` (realizes FR·Constraint — DP가 무엇을 실현/준수하는지)
- `context/overview.md` (시스템 정의·Pain Point — 현실성 기준선) · `context/glossary.md` · `context/INDEX.md` (ID 체계·DP 목록)
- `context/open-issues.md` (DP 관련 미결정·정합성 트래커 — OI-*)
- 직전 라운드 `round-NN/review/report.md` (있으면 — 추세 비교용)
- **직전 라운드 `round-NN/applier/report.md` (있으면 — 내 지적이 어떻게 처리됐는지)**. Applier가 이전 review/counsel을 [반영]/[발표 서사]/[생략]/[거부]/[이월] 중 무엇으로 처리했는지의 보고서다. 이번 라운드는 **이걸 받아 검증한다** — §8 절차 0 참조.

## 2. 판정 rubric

| 축 | 질문 | 합격 조건 |
|---|---|---|
| **ASR-coverage** ★1순위 | (세트) 모든 ASR이 어느 DP엔가 cover되나 + (개별) 이 DP가 driving ASR을 실제 cover하나? | **orphan ASR 0** · driving ASR 정확·누락 없음 · 대안이 그 ASR 칸에서 변별 · 비-ASR 강점에 과편향 없음 |
| **KPI-comparison** ★1순위 | 각 driving QA 주요 KPI가 비교 축으로 정직히 비교되나? | ★칸이 QA 실제 KPI 임계값에 앵커 · 칸별 변별력 존재 · 인용 수치가 QA 원문과 일치 · 조합 누적 시 KPI 비파탄 |
| **Well-framed** | 진짜 아키텍처 결정인가? 결정축이 직교한가? | 단일/직교 결정축 · realizes FR 연결 · 결정 포인트가 trade-off를 품음 |
| **ATAM-sound** | SP/TP/Risk/Non-Risk가 실재·load-bearing인가? | 장식적 아님 · 핵심 교환점 비은폐 · 범주오류·이중계상 없음 · 조합/창발 비용 산정 |
| **Convergent** | 무엇을 버릴지 정했나? | ‘다 좋다’ 스택 아님 · MVA/phasing/배제기준·트리거 명시 · QA 중요도 가중 적용 |
| **Traceable** | DP↔QA KPI↔FR↔cross-link 일관한가? | 본문↔하위 산출물 역참조 · 중복 realizes 없음 · open-issues 연계 |

**판정 표기**: ◎ 우수 · ○ 타당 · △ 부분 결함 · ✕ 재설계 필요
**verdict 형식**: `ASR <기호> / KPI <기호>` + severity(High/Med/Low) — 두 1순위 축을 헤드라인으로(QA의 `Sound / KPI` 대구). 나머지 4축은 판정 표에서 본다.
**severity 기준**: **ASR 미cover·누락**·핵심 KPI 미비교(앵커 없는 별점)·load-bearing 가정 모순·이중 제어평면 = **High** / 변별력 0·범주오류·이중계상·조합비용 미산정·수렴 부재 = **Med** / 추적성·미세 정합·문서 역참조 누락 = **Low**

---

## 4. 개별 DP 리뷰 파일 포맷 (`DP-0X-<slug>.md`)

```
# Review: DP-0X <결정명>

> source: context/dp/DP-0X-*.md (+ 폴더 승격 시 하위 산출물)
> verdict: **ASR <기호> / KPI <기호>** — <한 줄 판정> · severity **<High/Med/Low>**
> lenses: (1) Agentic Workflow 전문가 · (2) ATAM 평가 수석 아키텍트 · (3) Runner 인프라 아키텍트

## 원문 요약          — 결정 포인트 · 후보 대안 · driving ASR QA · ★매트릭스 · ATAM(SP/TP/Risk) · 결정/근거 핵심을 3~4줄로
## ASR cover·KPI 비교  — driving ASR(QA-01~07) cover 여부 + 각 QA 주요 KPI가 ★칸에 앵커·변별되는지 (이 DP의 1순위 판정)
## 렌즈 1 — Agentic Workflow 전문가 관점
## 렌즈 2 — ATAM 평가 수석 아키텍트 관점 (ASR cover·KPI 비교·분석 건전성)
## 렌즈 3 — Runner 인프라 아키텍트 관점 (구현 현실성)   (말미에 구현·측정 메커니즘)
## 판정               — 6축(ASR/KPI/Framing/ATAM/Convergent/Traceable) 표 + 근거
## Stage 2 권고       — ASR cover 보강·driving QA 교정 · 대안/별점 `기존 → 제안`(KPI 앵커) · ATAM 보강(SP/TP/Risk) · 수렴 경로(배제기준)
```

- cross-link은 ID 텍스트(`DP-02`, `QA-03`, `QAS-05`, `FR-0004`, `OI-7`)로 적어 grep 역참조되게.
- ★별점 교정은 **근거와 함께** `기존 ★★☆ → 제안 ★☆☆ (사유)` 형태로. 미검증 수치는 `◯`로 두고 산출 근거(예: QA-10의 5% budget, overview의 140모델)를 메모.
- 보강 제안은 **현상(파일/근거) → 왜 문제인가 → 보강안**의 3단으로 (findings.md F-* 스타일 계승).

---

## 5. 교차(cross-cutting) 분석

개별 DP를 넘어 **결정 세트 전체의 구조적 문제**를 `C1, C2, …`로 번호 매겨 정리한다. **Traceability Matrix는 매 라운드 필수**(대장 렌즈2 산출):
- **🔑 Traceability Matrix — DP × ASR (필수)**: **행 = DP, 열 = ASR(QA-01~07)**. 각 셀에 그 DP가 해당 ASR에 대해 갖는 ATAM 성격을 **S/T/R/N** 으로 표기(DP 문서의 ATAM 4절과 1:1):
  - **S** 민감점(Sensitivity) — 이 DP가 그 ASR 달성을 좌우 · **T** 교환점(Tradeoff) — 그 ASR을 다른 ASR과 교환 · **R** 위험(Risk) — 그 ASR 미달 위험 · **N** 비위험(Non-Risk) — 안전 충족 · **빈칸** — 무관(이 DP는 그 ASR 안 다룸).
  - 한 셀에 복수 표기 가능(예: `S,T`). 근거는 해당 DP 문서의 SP/TP/Risk/Non-Risk 항목으로 grep 역참조.
  - **열 전체가 빈칸인 ASR = orphan ASR**(어느 DP도 안 푸는 ASR) → 최상위 우선순위 결함으로 report에 올리고, 메울 신규 DP를 `_new-dp-candidates.md`에 NDP로 제안. (한 ASR에 DP·R이 과집중된 불균형, T가 몰린 핵심 교환 ASR도 함께 읽힌다.)
- **결정 간 의존**: 동시 결정돼야 할 DP가 따로 노는가(예: DP-02 Agent Hierarchy ↔ DP-04 제어평면 = 동시 결정, F-03).
- **공유 장애도메인 누적**: 여러 DP가 각자 공유 컴포넌트(브로커·오케스트레이터·공유 캐시)를 들여 합산 blast-radius가 QA 신뢰성 KPI를 깨는가(F-09).
- **driving QA 범위 불일치 / 중복 realizes**: DP가 인용한 QA·FR이 헤더 선언과 어긋나거나 중복되는가(F-12).
- **누락된 결정 포인트·대안 축**(→ `_new-dp-candidates.md`): 발굴 안 된 직교 결정축이나 미검토 대안(패턴 기반).

신규 결정/대안 권고는 `NDP-A, B, …`로 매기고 `_new-dp-candidates.md`에 결정 포인트 초안 + 후보 대안 + 채택 영향까지 적는다. (성격은 **권고**이며 채택은 Applier/팀 결정.)

---

## 6. 라운드 산출물 (red team 몫은 `round-NN/review/`)

라운드 폴더는 `round-NN/` (날짜 없음 — 메타는 보고서에 기록). red team 산출물은 그 아래 `review/`에 둔다(`counsel/`은 blue team [`Council.md`](Council.md) 몫).

| `round-NN/review/` 파일 | 역할 |
|---|---|
| `report.md` | **이것만 읽으면 라운드 전체를 아는 결론 요약** — **Traceability Matrix(DP×ASR, 셀 S/T/R/N)·orphan ASR**·판정표·교차발견(C*)·우선순위·신규결정(NDP)·직전 라운드 대비 변화. **날짜 등 메타를 여기 기록.** |
| `README.md` | 폴더 내비게이션(파일 목록·읽기순서·링크). 분석 내용은 두지 않음 |
| `DP-0X-*.md` | DP별 3렌즈 상세 (위 §4 포맷) |
| `_new-dp-candidates.md` | 신규 결정/대안 권고(NDP-*) |

상위 `discussion/dp/README.md`(마스터 인덱스)에 라운드 1줄 + verdict 추세표를 갱신한다.

---

## 7. 관리 규칙 (discussion/dp/)

- **append-only**: 지난 라운드는 수정 금지(오타·링크만 예외). 개선은 새 라운드로.
- **순환**: 리뷰(라운드 N) → Applier가 `context/dp/` 개선 → 라운드 N+1로 검증.
- **추적성**: 권고가 `context/dp/`에 반영되면 해당 DP `updated:` + `changelog.md` 델타로 남기고, 다음 라운드 report에 verdict 변화를 기록.
- **번호 주의**: DP는 **2자리** ID(`DP-02`)로 통일한다(QA·QAS와 동일 포맷). DP 번호는 발표 우선순위가 아니므로 ID는 **고정**하고 슬러그/내용만 바꾼다(재번호 안 함). 신규 결정 채택 시 다음 빈 번호 부여. (기존 4자리 `DP-0002` 파일의 리네이밍은 별도 마이그레이션.)
- **dp4류 폴더 승격 DP**: 본문(`DP-0X-*.md`)과 하위 산출물(`evaluation.md` 등)의 **정합**을 항상 함께 본다 — 본문이 하위 대안을 역참조 안 하면 그 자체가 Traceable 결함(F-12).

---

## 8. 리뷰 실행 절차 (체크리스트)

0. **직전 라운드 반영 보고서 받기**(`round-NN/applier/report.md`가 있으면 — 1라운드면 생략하되, `dp4/review/findings.md`를 prior art로 참조). 각 지적의 disposition을 확인하고 이번 라운드에 반영한다:
   - **[반영]** 항목 → 원본이 실제로 그 지적을 해소했는지 **재평가**(verdict가 개선됐나? 새 결함이 생겼나?). report에 verdict 변화 기록.
   - **[발표 서사]/[이월]** 항목 → 아직 안 닫혔으니 **이번 라운드에서 다시 판정**.
   - **[거부]** 항목 → 사유가 타당한지 점검, 필요하면 재반박 또는 수용.
   - 보고서 §“다음 Reviewer가 다시 볼 것”이 곧 **이번 라운드의 우선 점검 목록**이다.
1. §1 입력 전부 읽기 (overview·QA KPI·_backlog 포함 — 현실성·별점 기준선).
2. DP별로 §4 포맷에 따라 **3렌즈 모두** 작성 → 5축 판정 → Stage 2 권고.
3. 세트 전체로 §5 교차분석(C*) + 신규 결정/대안(NDP-*) 도출.
4. `report.md` 작성(§6) — 판정표·교차발견·우선순위·신규결정·직전 대비 변화.
5. `README.md`(내비) + 마스터 인덱스 추세표 갱신.
6. append-only·추적성(§7) 준수 확인.
