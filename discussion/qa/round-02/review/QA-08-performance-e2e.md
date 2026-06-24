# Review: QA-08 Performance — E2E 개발 시간 (round-02 재평가)

> source: `context/qa/QA-08-performance-e2e.md` + `QAS-08-performance-e2e.md` (round-01 [반영] 후)
> 직전 verdict(round-01): **Sound △ / KPI ✕ — High**
> verdict(round-02): **Sound ○ / KPI ○ — Low** — 정의(E2E)↔KPI(전달5%) 불일치 복구로 mislabel 치명상 해소(High 해소). 잔존은 DP-0004 결정 산식 실측 의존(OI-7)·importance 상향 여지·예시값뿐 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 QA-08 = **[반영]** (정의↔KPI 불일치 복구 → E2E latency p50/p95·throughput; 전달5% 강등). 이월: DP-0004 5% 산식 실측; importance 상향 여지.

## 원문 요약 (반영 후)
- **정의**: 모델 1건 E2E 응답 시간·처리량 목표 충족. E2E = 단계 compute + agent 루프 + 큐 대기 + handoff. artifact 전달은 한 요소(20GB Loss pain 정조준). Performance 3분할(QA-01/07/08) altitude.
- **KPI**: 주 = `E2E latency/모델 ≤6시간(p50/p95)`. 보조 = `throughput ≥50모델/일(QA-01 정렬)` · `(하위) 전달 오버헤드 ≤5%` · `전달 성공률·무결성=100%`.
- **QAS-08**: 대상·Response·Measure를 E2E latency·throughput·전달 무결성으로 동기화.

## 렌즈 1 — Agentic Workflow 전문가 관점

E2E는 agent 루프·큐 대기가 지배하는데, 이것이 정의에 "E2E = 단계 compute + **agent 루프** + **큐 대기** + handoff"로 명시됐다 — round-01이 지적한 "전달은 한 요소일 뿐"이 반영됐다. agentic 관점에서 **agent 루프 시간이 E2E의 주 변동원**(비결정 재시도·HITL 대기)임을 OTel span 단계 분해로 측정하는 구조도 적절. **닫혔다.**

- **잔여(silent cap)**: E2E latency p95에 **HITL 사람 승인 대기 시간**이 포함되는지 미명시 — 고위험 액션 HITL(QA-03)이 걸리면 E2E가 사람 응답 속도에 좌우된다(에이전트 시간이 아닌 인간 시간). p95 6시간이 *HITL 대기 제외*인지 *포함*인지 정의 권고(Low). agentic 자동화 시간과 인간 게이트 시간의 분리는 발표 정직성에도 중요.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **C2(정의↔KPI 불일치, 치명적) 해소 확인 — round-02 가장 큰 회복**: round-01의 치명상은 *정의=E2E 광의인데 KPI=artifact 전달 5%(협의 하위지표) 하나뿐* = **이름이 가리키는 것과 측정하는 것이 달랐다**(mislabel). 이번에 `E2E latency/모델 ≤6시간(p50/p95)`·`throughput ≥50모델/일`을 top-line으로 올리고 전달 5%를 하위로 강등해 **이름=측정 정렬**. KPI ✕→○. consistency ○.
- **Sound △→○**: 정의가 E2E를 명세하면서 Performance 3분할(QA-01 throughput / QA-07 per-node / QA-08 E2E)을 명문화 → altitude 모호 해소, QA-01과 throughput 축 정렬(중복 회피). C3(07↔08↔01) 정리 완료.
- **남은 형식 결함(경미)**: `throughput ≥50모델/일`이 **QA-01과 공유 축**이라 정렬했는데, 같은 KPI를 두 QA가 들면 *어느 QA가 SSoT인지* 모호할 수 있다. QA-08은 "QA-01과 정렬"이라 적어 QA-01을 throughput SSoT로 두는 듯하나, 명시적으로 "throughput SSoT = QA-01, QA-08은 참조"라 박으면 더 깔끔(Low).
- **importance 상향 여지**: applier §4가 남긴 "E2E top-line은 발표 가치 지표 → M에서 상향 검토"는 **사람 결정**이라 verdict 무관. 현재 M 유지.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

- **handoff = data-plane 설계 문제**(round-01 렌즈3) → 정의에 "20GB를 참조(claim-check)로 전달, data locality로 co-location"이 명시되고, KPI `전달 오버헤드 ≤5%`가 **DP-0004 A5(claim-check 원격) vs A8(로컬) 택일을 가르는 결정 산식**(`(4단계×20GB)÷대역폭`)과 직결됐다. 이 PoC가 DP-0004 택일을 닫는 핵심 고리라는 자각이 모범적. **양호.**
- **잔여(DP 위임, OI-7)**: **DP-0004 결정 산식(5% budget → A5 vs A8)이 실측·노드 사양·20GB 대역폭에 의존**해 그 전엔 구조만 고정. 즉 KPI는 측정 가능하나 *합격선을 가르는 산식의 입력값*이 실측 의존. DP-0004/0005가 E2E latency까지 책임지는지 역검토 → DP 디스커션 위임.
- **mock compute 한계**: 검증 전략이 인정하듯 **mock compute가 실제 Quantize/Compile 시간 분포를 근사 못 하면 critical path가 왜곡**된다 → p50/p95가 mock 분포에 좌우. silent cap 명시됨.
- **runner 측 KPI**(재확인): 단계별 latency 분해(compute/agent/큐/handoff), 큐 대기 p95(QA-01 공유), handoff 전송 시간·재전송율, critical path 점유율.

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | △ | **○** | E2E 명세 + Performance 3분할 명문화로 altitude 고정, QA-01 throughput 정렬 |
| KPI가 측정 가능한가 | ✕ | **○** | 정의↔KPI 불일치 복구 — E2E latency p50/p95·throughput top-line 추가, 이름=측정 정렬 |
| KPI가 현실적/적절한가 | ✕ | **○** | percentile SLI·claim-check 현실적. 단 HITL 대기 포함 여부·DP 산식 실측은 Low 보강 |
| 정의↔KPI↔QAS 일치 | △ | **○** | QAS-08 대상·Measure가 E2E·throughput·무결성으로 동기화 확인 |

**verdict 변화: Sound △→○ / KPI ✕→○ · severity High→Low.** round-01 High의 근거(정의↔KPI mislabel)는 해소됨. High 4건 중 가장 깔끔하게 닫힌 케이스.

## Stage 2 권고 (round-02)

- **(DP 디스커션 위임, OI-7)** **DP-0004 결정 산식(5% → A5 vs A8)**의 입력값(20GB 대역폭·노드 사양) 실측 의존을 명시 — 본 PoC가 데이터 공급, 수치 확정은 실측. DP-0004/0005가 E2E latency 책임지는지 역검토.
- **(HITL 대기 분리, Low)** `E2E latency p95 ≤6시간`이 **HITL 사람 승인 대기 제외/포함** 중 무엇인지 정의 — 자동화 시간과 인간 게이트 시간 분리(발표 정직성).
- **(throughput SSoT 명시, Low)** `throughput ≥50모델/일`의 SSoT = QA-01, QA-08은 참조임을 명문화(이중 SSoT 모호 제거).
- **(importance 상향, 사람 결정)** E2E top-line의 발표 가치 → M→상향 검토(verdict 무관, OI-8 트랙).
- **(예시값 확정)** `6시간·50모델/일·5%`는 부하시험으로 확정.
