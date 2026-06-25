# Counsel: QA-01 Scalability

> refs-review: round-04/review/QA-01-scalability.md
> seats: 발의 Seat 3 (USL·민감도) · 합의 consensus
> stance: **조건부 채택 — margin 규칙화(★☆☆ 하한 표현) + LLM-bound USL silent cap + ★☆☆ 대역 유지(좁음 명시)**

## Reviewer 지적 요약
efficiency 0.8→0.70 재배치는 USL 근거 충실. red-team(Low): (1) margin 0.02~0.05 흔들림(★★★ 0.05 vs ★☆☆ 0.02·C2), (2) SPARCcenter CPU USL vs LLM-bound worker apples(C1), (3) ★☆☆ [0.70,0.75) 대역 좁아 변별 빈약, (4) PoC가 가정 파라미터 민감.

## 개선안 (정의·KPI 기존→제안)
### (A) margin 차등 명시 (C2)
- ★ 노트: `★★★ 0.85 = 이론 0.90 − 0.05, ★☆☆ 0.70 = 필드우수 0.72 − 0.02`. → 제안: **차등을 근거화** — `★☆☆은 필드 우수(0.72) 바로 아래라 margin 최소(0.02), ★★★는 이론 천장이라 PoC margin 크게(0.05)`. margin이 "임의"가 아니라 "천장과의 거리에 비례"임을 명시(QA-06 단일 Y 규칙과 다른 차등 근거).
### (B) LLM-bound USL silent cap (C1)
- `인용 USL 0.72는 SPARCcenter SPEC SDM91(1990s CPU 벤치)에서 빌려옴. 우리 병목은 외부 LLM 전역 TPM/RPM이라 contention(α)·coherency(β) 구조가 다름 — 0.72가 우리 천장 대표라는 보장 없음. 조건(헤드룸 ≥20%)으로 rate-limit 포화를 배제한 순수 큐잉 효율임을 명시.`
### (C) ★☆☆ 대역 (권고 2)
- [0.70,0.75) 0.05 폭 유지하되 silent cap `좁아 변별 빈약 — 하한 0.68 하향은 USL 우수도와 더 벌어져 미채택, 좁은 채로 둠`.

## 근거 (레퍼런스 + 검증 결과)
**C4 — USL 출처**: WSO2 USL 사례(SPARCcenter 0.72)는 [USL](https://wso2.com/blog/research/measuring-software-scalability-using-universal-scalability-law/)(Council.md §4 확장성). LLM-bound worker USL 회귀 사례는 본 검증에서 미발견 → "확인 불가, 우리 PoC로 USL 회귀 직접 측정"으로 명시. backlog 기반 오토스케일(CPU 아님)은 [KEDA](https://keda.sh/) 표준.

## PoC 증명법
### PoC-01: scaling efficiency + USL 회귀 (부하시험 아키타입)
- 가설: "efficiency ★ 경계(0.70/0.75/0.85)가 backlog 오토스케일 시뮬로 측정되고 가정 파라미터에 robust하다."
- 지표: efficiency = (부하 2배 시 처리량 배수 ÷ 2), USL α·β 회귀. 합격선: 헤드룸 ≥20% 고정 하 ★ 경계 변별 + 민감도 ±0.05.
- 셋업: backlog 기반 오토스케일 시뮬(서비스시간·도착률·cold-start 파라미터), 부하 N배 sweep.
- 절차: ① 부하 sweep → efficiency 곡선. ② USL 회귀로 α·β 추정. ③ 파라미터 ±20% 민감도.
- 합격 기준: efficiency가 가정 파라미터에 ±0.05 robust면 ★ 경계 신뢰.
- 규모/기간: 부하 수십 VU, ~0.5일.
- 리스크/한계(silent cap): rate-limit 포화는 조건으로 배제 — 실운영 계정 한도 막히면 efficiency 무의미.

## DP·발표 영향
- DP-0001/0004의 rate-limit headroom·admission control·bounded queue 미명시(OI-7).
