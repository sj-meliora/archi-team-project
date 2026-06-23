# Counsel: QA-08 Performance — E2E 개발 시간

> refs-review: round-01/review/QA-08-performance-e2e.md · report.md(C2·C3)
> seats: 발의 Seat 2(수석 아키텍트) · 합의 consensus (Seat 3 handoff data-plane, Seat 1 단계 병렬화)
> stance: 채택 권장 (정의↔KPI 정렬: E2E KPI 추가 + artifact 5%는 하위 항목으로 강등)

## Reviewer 지적 요약
- **정의↔KPI 불일치(C2, 치명적)**: 정의=E2E 응답·처리량(광의)인데 KPI는 artifact 전달 5%(협의 하위지표) 1개뿐 → **이름이 가리키는 것과 측정하는 것이 다름** (렌즈2).
- **실제 E2E 지표 부재**: 모델당 E2E latency(p50/p95)·throughput 같은 top-line이 없음 (렌즈2).
- E2E는 **단계 compute + agent 루프 + 큐 대기**가 지배 — artifact 전달은 한 요소일 뿐 (렌즈1).
- handoff는 **data-plane 설계 문제**(참조 전달·data locality)로 5%가 실제 달성됨 (렌즈3).
- QA-07과 Performance 패밀리로 통합 여지(C3).

## 개선안 (정의·KPI 기존→제안)

**정의**
- 기존: "응답 시간·처리량 목표 충족 (E2E)."
- 제안 (권장 = 정렬안 a): 이름·정의 유지 + **실제 E2E KPI 추가**, artifact 5%는 하위 항목으로 강등 + 무결성 지표로 보존.

**KPI**
| # | 기존 | 제안 | 비고 |
|---|---|---|---|
| ① | (없음) | **E2E latency/모델 ≤ ◯시간 (p50/p95)** | top-line. ◯는 PoC-E2E1 |
| ② | (없음) | **throughput ≥ ◯모델/일** | top-line. QA-01과 정렬(C3) |
| ③ | Artifact 전달 오버헤드 ≤ E2E의 5% | **유지(하위 항목으로 강등)** | DP-0004 SP-1의 택일 산식과 직결 |
| ④ | — | **artifact 전달 성공률·무결성 = 100%** | overview 20GB Loss pain → 신뢰성 지표로 보존 |

## 근거 (레퍼런스)
- **percentile E2E SLI + throughput SLO**: 평균 금지, p50/p95로 E2E latency, throughput은 별 SLO — Google SRE / percentile 가이드. (§4 지연) — https://sre.google/workbook/implementing-slos/ , https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view
- **참조 전달(claim-check)·data locality**: 20GB를 값 복사 대신 오브젝트 스토어/로컬 볼륨에 두고 참조 전달 → 전달 오버헤드 5% 달성의 아키텍처(DP-0004 A5 claim-check / A8 로컬). (§4 지연·DP-0004)
- ⚠️ ◯시간·◯모델/일은 부하시험으로 확정.

## PoC 증명법

### PoC-E2E1: E2E latency·throughput·전달 오버헤드를 부하시험으로 측정한다 (부하시험)
- **가설**: "모델당 E2E latency(p50/p95)·throughput·artifact 전달 오버헤드 비율이 측정 가능하고, A5/A8 택일 산식(전달 ≤5%)을 데이터로 결정한다."
- **지표**: E2E latency p50/p95, throughput(모델/일), 전달 오버헤드 비율 = handoff 시간/E2E, 큐 대기 비율, 전달 무결성(=100%).
- **셋업**: mock 4단계 파이프라인 + claim-check(원격, A5) vs 로컬 볼륨(A8) 2모드 + k6/Locust로 동시 모델 부하 + OTel span으로 단계별 분해(QA-04 연계).
- **절차**: ① 단일 모델 E2E latency 분해(단계 compute / agent / 큐 / handoff) → ② 부하 점증으로 throughput·p95 → ③ A5 vs A8 전달 오버헤드 비율 비교(5% 산식 검증).
- **합격(Exit)**: E2E latency p95 ≤ ◯, throughput ≥ ◯, 전달 오버헤드가 A5/A8 중 어느 쪽에서 5% 안에 드는지 판정.
- **규모/기간**: 모델 수십 건, 2모드 비교, 약 2~3일.
- **리스크/한계(silent cap)**: mock compute가 실제 Quantize/Compile 시간 분포를 근사 못 하면 critical path 왜곡. 실제 20GB 대역폭·노드 사양 의존(DP-0004 산식은 실측 필요 — 그 전엔 구조만 고정).

## DP·발표 영향
- **DP 연결**: PoC-E2E1이 **DP-0004의 결정 산식**(`(4단계 × 20GB) ÷ 대역폭`이 5% budget 내인가 → A5 vs A8)에 직접 데이터 공급 — 본 권고가 DP-0004 택일을 PoC로 닫는 핵심 고리. DP-0005(공유 캐시)가 ① latency 단축에 기여.
- **통합(C3)**: Performance를 QA-01(throughput)·QA-07(per-node)·QA-08(E2E) 3분할로 정렬 — ②throughput은 QA-01과 공유 축, ③전달은 QA-08 고유. QA-07/QA-01 counsel과 일관.
- **importance**: 현재 M이나, E2E top-line은 발표 가치 지표라 상향 여지.
