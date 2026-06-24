---
id: QA-08
category: QA
importance: M
difficulty: M
source: pptx p.13
related-dp: [DP-0004, DP-0005]
related-fr: [FR-0002]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "정의(E2E)↔KPI(전달5%) 불일치 복구 → E2E latency p50/p95·throughput 추가, 전달5%는 하위로 강등 (자세히 → ## 변경 이력)"
---

# QA-08 Performance — E2E 개발 시간

## 정의 / Refinement
모델 1건의 **E2E(처음부터 끝까지) 응답 시간·처리량 목표**를 충족한다. E2E 시간은 단계 compute + agent 루프 + 큐 대기 + 단계 간 handoff(artifact 전달)의 합이며, 이 중 **artifact 전달**은 overview의 "20GB 수동 공유 Loss" pain을 정조준한 하위 목표다(전부가 아니라 한 요소).

설계할 때 잡아야 할 두 가지 관점:

- **이름이 약속한 E2E를 실제로 측정한다** — 기존 KPI는 정의(E2E 광의)와 달리 "artifact 전달 5%"(협의 하위지표) 하나뿐이라, **이름이 가리키는 것과 측정하는 것이 달랐다**. 진짜 top-line인 **모델당 E2E latency(p50/p95)·throughput**을 측정하고, 전달 5%는 그 하위 항목으로 둔다.
- **handoff는 data-plane 설계 문제다** — 20GB를 노드마다 복사하지 말고 **오브젝트 스토어/로컬 볼륨에 두고 참조(포인터)로 전달**하며, 의존 단계는 data locality로 co-location한다. 이것이 "전달 오버헤드 5%"를 실제로 달성하는 아키텍처(DP-0004 A5 claim-check / A8 로컬, DP-0005 공유 캐시와 직결).

> 이 QA는 "**모델 1건의 E2E 소요시간·전달 효율**"을 다룬다 — 노드 1건 속도는 QA-07, 단위 시간당 처리량(throughput)은 QA-01과 공유 축이다(Performance 3분할: QA-01 throughput / QA-07 per-node / QA-08 E2E). throughput KPI는 QA-01과 정렬해 중복을 피한다.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `모델당 E2E latency(p50/p95)` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **E2E latency / 모델 ≤ 6시간 (p50/p95)** `[주 KPI · PoC 대상]` — 모델 1건 처음→끝
  - 쉽게: 모델 한 건이 IR 변환부터 컴파일까지 전 과정을 끝내는 데 걸리는 시간이 (가장 오래 걸린 상위 5%를 봐도) 6시간 이하여야 한다. *p50/p95 = 중앙값 / 상위 5%를 뺀 95% 기준값.*
- **throughput ≥ 50 모델/일** — QA-01과 정렬(공유 축)
  - 쉽게: 하루에 50개 이상 모델을 끝낼 수 있어야 한다. (확장성 QA-01의 처리량과 같은 축이라 정렬해 관리.)
- **(하위) artifact 전달 오버헤드 ≤ E2E의 5%** — handoff 시간 / E2E
  - 쉽게: 전체 E2E 시간 중 단계 사이 산출물(20GB)을 옮기는 데 쓰는 시간이 5%를 넘으면 안 된다. (DP-0004 A5 vs A8 택일을 가르는 산식과 직결.)
- **artifact 전달 성공률·무결성 = 100%** — 20GB Loss pain 직결, 신뢰성 지표로 보존
  - 쉽게: 옮긴 산출물이 누락·손상 없이 100% 온전히 전달돼야 한다. (overview의 "수동 공유 중 유실" pain을 막는 신뢰성 지표.)

> 위 수치(6시간·50모델/일·5%)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 부하시험으로 확정한다.
> 폐기: 旧 `Artifact 전달 오버헤드 ≤ 5%` **단독** — 정의(E2E 광의)와 KPI(전달 협의)가 불일치(이름≠측정). → E2E latency·throughput을 top-line으로 추가하고 전달 5%는 하위 항목으로 강등 + 무결성 지표 보존.

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 지연·데이터 전달의 업계 표준에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **percentile E2E SLI + throughput SLO** | 평균 금지, p50/p95로 E2E latency를 재고 throughput은 별도 SLO로 — SRE 표준 | [Google SRE Workbook — SLO 구현](https://sre.google/workbook/implementing-slos/) · [p50/p95/p99 해설](https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view) |
| **참조 전달(claim-check)·data locality** | 20GB를 값 복사 대신 오브젝트 스토어/로컬 볼륨에 두고 참조 전달 → 전달 오버헤드 5% 달성의 아키텍처(DP-0004 A5/A8) | DP-0004 SP-1 결정 산식 (round-01 counsel §4) |
| **전달 무결성 = 신뢰성 지표** | overview 20GB Loss pain → 전달 성공률·무결성 100%를 성능이 아닌 신뢰성 지표로 보존 | overview Pain Point (수동 공유 유실) |

> ⚠️ 레퍼런스의 수치·패턴은 **정당화용**이며 그대로 복제하지 않는다. E2E latency(예시 6시간)·throughput(예시 50모델/일)은 위 [검증 전략](#검증-전략)의 부하시험으로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 부하시험·파이프라인 모델**로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **E2E latency p50/p95** `[주]` | **DP-0005 2안 공유 캐시**(유사 WF 재사용으로 E2E 단축 [Performance]★★★) · **DP-0004 A8**(로컬 전달로 E2E ★★★) | **▶ 실제 제작:** mock 4단계 파이프라인 부하시험 — claim-check(원격, A5) vs 로컬 볼륨(A8) 2모드 + k6/Locust 동시 모델 부하 + OTel span 단계 분해 → **E2E latency p50/p95**를 단계(compute/agent/큐/handoff)별로 분해 산출 |
| throughput ≥ 50모델/일 | DP-0004 A5/A8 일회용 near-infinite auto-scale (QA-01 공유 축) | 보조 모델: 위 부하시험을 부하 점증으로 돌려 throughput·p95 곡선 산출 (QA-01 시뮬과 정렬) |
| 전달 오버헤드 ≤ 5% | **DP-0004 SP-1/TP-1 전달 배치**(A5 claim-check vs A8 로컬) — `(4단계×20GB)÷대역폭`이 5% budget 내인가가 A5/A8을 가름 | 보조 모델: A5 vs A8 전달 오버헤드 비율 비교 → **DP-0004 결정 산식(5%)을 데이터로 판정**(본 PoC가 DP-0004 택일을 닫는 핵심 고리) |
| 전달 성공률·무결성 = 100% | DP-0004 claim-check 무결성(체크섬)·DP-0005 캐시 검증 | 보조 모델: 전달 중 손상/누락 주입 → 검출·재전송으로 무결성 100% 유지 확인 |

> 가정·한계: **mock compute가 실제 Quantize/Compile 시간 분포를 근사 못 하면 critical path가 왜곡**된다. 실제 20GB 대역폭·노드 사양에 의존(DP-0004 산식은 실측 필요 — 그 전엔 구조만 고정). 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다 — silent cap으로 명시.

## 변경 이력

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-08-performance-e2e.md) (red team verdict: **Sound △ / KPI ✕ — High** — 정의↔KPI 불일치(mislabel))

**무엇이 문제였나 (review 지적)**
- **정의↔KPI 불일치(C2, 치명적)**: 정의=E2E 응답·처리량(광의)인데 KPI는 artifact 전달 5%(협의 하위지표) 1개뿐 → 이름이 가리키는 것과 측정하는 것이 다름.
- **실제 E2E 지표 부재**: 모델당 E2E latency(p50/p95)·throughput 같은 top-line이 없음.
- E2E는 단계 compute + agent 루프 + 큐 대기가 지배 — artifact 전달은 한 요소일 뿐.

**무엇을 바꿨나 (반영)**
- **정의**: 이름·정의(E2E 광의) 유지 + handoff가 data-plane 설계 문제임을 명시. Performance 3분할 altitude로 QA-01(throughput)·QA-07(per-node)과 경계.
- **KPI 정렬**: 旧 `전달 5%` 단독 → ① `E2E latency/모델 ≤6시간(p50/p95)`(top-line) ② `throughput ≥50모델/일`(QA-01 정렬) ③ 전달 5%는 **하위 항목으로 강등** ④ `전달 성공률·무결성 100%`(신뢰성 보존).
- 짝 시나리오 `QAS-08`의 대상·Response·Measure를 E2E latency·throughput·전달 무결성으로 동기화.

**남은 일 (이 라운드에서 미반영)**
- **importance 상향 여지** — E2E top-line은 발표 가치 지표라 현재 M에서 상향 검토 가능(사람 결정).
- **DP-0004 결정 산식(5% budget → A5 vs A8)은 실측·노드 사양 의존** → 본 PoC가 데이터를 공급하되 수치 확정은 실측 필요. DP-0004/0005가 E2E latency까지 책임지는지 역검토 (`open-issues.md` 트래킹 대상).
- E2E latency `6시간`·throughput `50모델/일`은 **예시값**이며 부하시험으로 확정.
