# Counsel: QA-08 Performance — E2E 개발 시간 (round-02)

> refs-review: [round-02/review/QA-08-performance-e2e.md](../review/QA-08-performance-e2e.md)
> 직전 counsel: [round-01/counsel/QA-08-performance-e2e.md](../../round-01/counsel/QA-08-performance-e2e.md) (채택 권장 — E2E latency·throughput 추가)
> seats: 발의 **Seat 2**(수석 아키텍트) · 합의 **consensus**
> stance: **닫힘 확인(High 해소·가장 깔끔) — 잔여 = DP-0004 5% 산식 실측·importance 상향 여지(사람 결정)**

## Reviewer 지적 요약
- round-02 verdict: **Sound ○ / KPI ○ · Low** (R01 High 해소, 가장 깔끔). 정의(E2E)↔KPI(전달5%) mislabel 복구 → `E2E latency/모델 ≤6시간 p50/p95` 주, `throughput ≥50모델/일` · `전달 오버헤드 ≤5%` 강등 보조.
- 잔여(비-verdict): DP-0004 5% 결정 산식(A5 vs A8 택일) 실측 의존(OI-7), importance 상향 여지(E2E top-line이라 사람 결정), 예시값.

## 개선안 (정의·KPI 기존→제안)
KPI 닫힘 — 새 KPI 없음. E2E = 단계 compute + agent 루프 + 큐 대기 + handoff 정의 유지. artifact 전달(20GB Loss pain)은 한 요소로 강등 유지. Performance 3분할(QA-01/07/08) altitude 유지.
> ⚠️ `6시간·50모델/일·5%`는 "측정 가능 KPI의 모양" 예시값 — mock 파이프라인 실측·DP-0004 택일 데이터로 확정.

## 근거 (레퍼런스)
§4 **지연(Latency)** + **확장성·처리량(Scalability)** 행.
- **percentile SLI(p50/p95, 평균 금지)** + throughput SLO. https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view · https://sre.google/sre-book/service-level-objectives/
- E2E 부하시험으로 A5 vs A8(전달 5%) 택일 산식 데이터 공급(DP-0004).

## PoC 증명법
### PoC-E2E1(R1 유지): E2E latency·throughput·전달 5% (부하시험)
- **가설**: "mock 4단계 파이프라인 E2E로 latency p50/p95·throughput이 측정 가능하며, 전달 오버헤드 5% 산식으로 DP-0004 A5 vs A8 택일 데이터를 공급한다."
- **지표+합격선**: E2E latency p50/p95 ≤◯ · throughput ≥◯모델/일 · 전달 오버헤드 ≤◯%(A5 vs A8 산식 판정).
- **셋업**: 부하 하네스(k6/Locust — S1·R1 공유) + mock 4단계 파이프라인.
- **절차**: E2E 부하 주입 → latency 분포·throughput 측정 + 전달 오버헤드 비율 산출 → DP-0004 택일 데이터.
- **합격(Exit)**: latency/throughput 측정 가능 ∧ 전달 5% 산식이 A5/A8 택일에 데이터 공급.
- **규모/기간**: 약 2일(부하 하네스 공유).
- **silent cap**: mock 파이프라인 compute 분포·20GB 대역폭·노드 사양은 실환경 의존(산식 구조만 PoC로 고정).

## DP·발표 영향
- **DP 위임 (OI-7)**: DP-0004 A5 vs A8 택일 산식(전달 5% 결정)·DP-0005 E2E latency 책임 — 실측 데이터 필요. DP 디스커션 위임.
- **importance (사람 결정)**: E2E top-line이라 **상향 여지** — pain point(20GB Loss·개발 시간) 직결이라 발표 우선순위 상위 후보(OI-8 트랙, verdict 무관, 사람 결정).
- **번호/서사**: "사람 없이 E2E 개발 시간 단축"이 가장 직관적 비즈니스 메시지 — 발표 헤드라인 후보.
