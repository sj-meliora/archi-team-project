# Counsel: QA-07 Performance — Agent 수행 시간 (per-node, round-02)

> refs-review: [round-02/review/QA-07-performance-agent-time.md](../review/QA-07-performance-agent-time.md) · [report.md](../review/report.md)(C1·Stage2-1) · 게이트: [NQA-B](../review/NQA-B-correctness.md)
> 직전 counsel: [round-01/counsel/QA-07-performance-agent-time.md](../../round-01/counsel/QA-07-performance-agent-time.md) (채택 권장 — speedup + 품질 게이트)
> seats: 발의 **Seat 2**(수석 아키텍트) · 합의 **consensus** (Seat 1 비결정 표본편향 / Seat 3 first-pass 게이트 hook DP)
> stance: **닫힘 확인(High 해소) + NQA-B 동반 채택이 주 KPI 닫힘의 전제(OI-8) + baseline 프로토콜 명시(Med)**

## Reviewer 지적 요약

- round-02 verdict: **Sound ○ / KPI △ · Med** (R01 High 해소). "최대화" 폐기 → `노드타입별 speedup ≥3배(first-pass 성공분)` + paired 품질 게이트로 측정가능성 회복.
- **C1 핵심 잔여**: 주 KPI `speedup(first-pass 성공분)`의 분모 조건(성공 판정)이 **미채택 NQA-B golden 게이트에 전적 의존** → NQA-B 미채택·검증 [발표 서사]라 주 KPI가 독립 측정 불가. round-01 "최대화 측정불가"가 round-02 "성공 판정 게이트 부재"로 형태만 바뀐 채 잔존.
- review 권고: NQA-B 동반 채택(OI-8, 최우선), baseline 프로토콜 명시(Med), 비결정 표본편향 가드(동반 보고 의무, Low), DP-0001 first-pass 게이트 hook(OI-7).

## 개선안 (정의·KPI 기존→제안)

Council 응답: **QA-07 주 KPI 닫힘 = NQA-B 채택의 직접 효과**(동반 닫힘). QA-07 자체에서 할 일은 (a) NQA-B 의존을 명시적 전제로 고정, (b) baseline 프로토콜을 acceptance의 일부로 승격, (c) 생존자 편향 방어. Sound ○이므로 정의 유지.

**정의: 기존 → 제안 (유지 + 의존 명시)**
- 기존: 1-pass 성공 노드 한정, 노드타입별 시간 단축, 속도-품질 짝. 유지.
- 제안: "first-pass 성공 판정 = NQA-B golden 게이트"를 **명시적 외부 의존으로 본문에 고정**(현재 QAS 비고에만 — 본문 승격으로 닫힘 전제를 가시화). 양방향 cross-link 확정(QA-07→NQA-B 역방향 명문화).

**KPI: 기존 → 제안**

| # | 기존 (round-01 반영) | 제안 (round-02) | 닫힘 상태 |
|---|---|---|---|
| 주 | 노드타입별 speedup ≥3배 (first-pass 성공분) | 동일 — **NQA-B golden 게이트 채택+실측이 분모(성공 판정) 닫힘의 전제**. `baseline 출처 미정` → **baseline = 숙련도·표본 고정 조건의 수동 수행시간 중앙값, 측정 프로토콜을 슬라이드 명시**(acceptance의 일부, Med) | △→○ (**NQA-B 채택 시**) |
| 보조1 | 품질 게이트 = first-pass 성공률 (paired) | 동일 — **speedup 보고 시 표본 수·first-pass 성공률 동반 보고 의무화**(성공분만 집계하는 생존자 편향 방어, Low). 비결정 성공률 낮은 날 표본 축소 → 낙관 편향 노출 | △ (게이트=NQA-B) |
| 보조2 | 커버리지 ≥80% | 동일 — gaming(쉬운 노드만) 방지 | 측정 가능 |
| 보조3 | runner 오버헤드 비율 ≤15% | 동일 — (큐 대기+dispatch+cold start)/노드 총시간, OTel span 분해(QA-04 계측 공유) | 측정 가능 |

> ⚠️ `3배·80%·15%`는 "측정 가능 KPI의 모양" 예시값 — 노드타입별 baseline 측정으로 확정.

## 근거 (레퍼런스)

§4 라이브러리 **지연(Latency)** + **정확성(Correctness, NQA-B 게이트)** + **관측성(오버헤드 span)** 교차 적용.

- **percentile SLI(평균 금지)·throughput** — per-node 시간은 분포로 봐야 함. 단 QA-07 주 KPI는 절대시간 차 무의미 노드 믹스 때문에 **노드타입별 비율(speedup)**로 정규화(round-01 권고 정착).
  - Google SRE SLO/SLI: https://sre.google/sre-book/service-level-objectives/
  - p50/p95/p99: https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view
- **first-pass 성공 판정 = golden set + LLM-as-judge** — 속도-품질 trade-off("빠르게 틀리기")를 차단하는 게이트가 NQA-B 정확성 게이트와 동일 컴포넌트. speedup을 성공분에만 집계해야 rework 폭증을 막음.
  - golden·LLM-as-judge: https://www.getmaxim.ai/articles/building-a-golden-dataset-for-ai-evaluation-a-step-by-step-guide/ · https://www.comet.com/site/blog/llm-as-a-judge/
- **OTel GenAI span으로 오버헤드 분리** — 큐 대기·dispatch·cold start를 span으로 분해해 개선 레버 노출.
  - OTel GenAI semconv: https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/
- **비결정 → 생존자 편향**: first-pass 성공률이 run마다 흔들리므로(LLM 비결정) 표본 수 동반 보고가 필드 정직성 표준(pass^k 계열 논거와 정합).
  - τ-bench pass^k(비결정 신뢰): https://arxiv.org/abs/2406.12045

> 레퍼런스 수치(3배 등)는 패턴 정당화용 — 합격선은 PoC baseline으로 확정.

## PoC 증명법

### PoC-P1(R2, NQA-B 후행): first-pass 게이트로 정규화한 speedup을 baseline 프로토콜과 함께 측정한다 (eval+측정)

- **가설(Hypothesis)**: "고정 노드집합에서 노드타입별 speedup이 first-pass 성공분에 한해 측정 가능하며, 그 성공 판정은 PoC-N-B1(NQA-B golden 게이트) 출력으로 공급된다. baseline 프로토콜을 고정하면 합격선이 재현 가능하다."
- **지표(Metric) + 합격선**:
  - 노드타입별 speedup (vs baseline 중앙값) — first-pass 성공분만
  - **표본 수 · first-pass 성공률 동반 보고**(생존자 편향 노출)
  - 커버리지 ≥ ◯% · runner 오버헤드 비율 ≤ ◯%
- **셋업(Setup)**: mock 4단계 파이프라인 + **PoC-N-B1 golden 게이트 출력(first-pass 성공/실패 라벨)** + baseline = 숙련도·표본 고정 조건의 수동 수행시간 중앙값. QA-04 OTel 계측(오버헤드 span) 공유.
- **절차(Procedure)**:
  1. baseline 측정 프로토콜 고정(숙련도·표본·노드 믹스) → 노드타입별 수동 중앙값
  2. **PoC-N-B1 게이트에서 first-pass 성공 라벨 수급** → 성공분만 speedup 집계
  3. 표본 수·성공률 동반 기록 + 오버헤드 span 분해
- **합격 기준(Exit)**: speedup이 NQA-B 게이트 출력으로 산출됨(C1 교차 의존 닫힘 시연) ∧ baseline 프로토콜 명시로 재현 가능 ∧ 표본 수·성공률 동반 보고.
- **규모/기간(Scope)**: NQA-B PoC-N-B1 후행(golden 게이트 선행) → 약 2~3일(N-B1 완료 후).
- **리스크/한계(silent cap)**: baseline 표본·숙련도 편차가 최대 난점(프로토콜로 고정해도 잔여). 비결정 성공률 낮으면 표본 축소로 speedup 낙관 편향. **NQA-B 미채택이면 주 KPI 분모가 닫히지 않음** — PoC 가능해도 acceptance는 OI-8 채택 의존.

## DP·발표 영향

- **DP 연결 (OI-7, DP 디스커션 위임)**: DP-0001 1안 즉시 실행이 latency만 보고 **품질 게이트와 무연결** — "first-pass 성공분만 집계"의 핵심 장치(게이트 hook: 성공/실패 라벨을 speedup 집계에서 분리)를 책임지는 DP가 없음. **eval/검증 서브시스템 DP**가 first-pass 게이트를 제공해야 함(NQA-B와 공유). QA 측 verdict는 DP로 위임.
- **동반 닫힘 효과 (핵심)**: **NQA-B 정식 채택 + golden 실측 = QA-07 주 KPI(△→○) 동반 닫힘의 단일 행동.** QA-07은 주 KPI 분모가 통째로 NQA-B 의존이라 QA-09(②-2만 의존)보다 닫힘도가 낮음 → **NQA-B 채택 우선순위 효과가 QA-07에서 가장 크다**(report Stage2-1).
- **번호/서사**: per-node 성능은 "사람 없이 빠르게 + 정확히"의 속도 축 — NQA-B(정확)와 paired로 발표("빠르게 틀리기 차단"). importance 변동 권고 없음.
