# Counsel: QA-08 Reliability — Workflow 간 독립성

> refs-review: round-04/review/QA-08-reliability-workflow.md
> seats: 발의 Seat 3 (bulkhead·token-bucket) · 합의 consensus
> stance: **채택 권장 — ★★★ 5% margin 근거화 + arXiv 미래형 ID 검증결과 반영 + 쿼터 보조 별점 축 검토**

## Reviewer 지적 요약
latency 증가 10→25% 재배치는 무격리 바닥(+100%)과 간격 확보 합리. 잔여(Low): (1) **★★★ 5% margin 근거 없음**(권고 1·C2), (2) **arXiv 2604.03145(noisy-neighbor) 등 미래형 ID 검증 필요**(권고 2·Med·C4), (3) agentic 진짜 공유 장애(쿼터)가 ★에서 빠지고 게이트로만(권고 3·C3), (4) 인용 "p99 doubling·CPU 56%"는 스토리지/CPU noisy-neighbor(simplyblock)지 LLM 토큰 쿼터 간섭 아님(C1).

## 개선안 (정의·KPI 기존→제안)
### (A) ★★★ 5% margin 근거화 (권고 1·C2)
- ★ 노트: `★★★ ≤5% = 무간섭 이상 + 격리 시 tail 안정화 관측 대역 하단 + 폭주 강도 가정 흡수 ≈5%`. 무격리 바닥(+100%)·★☆☆ 25% 대비 충분 간격. margin이 "임의"가 아니라 "tail 안정화 + 폭주 강도 흡수"임을 근거화.
### (B) arXiv ID 검증결과 반영 (C4)
- **2604.03145는 2026-04 ID로 현재(2026-06) 유효 과거 ID 형식**이나 본 검증에서 직접 대조 미완 → "출처 형식 유효, 인용 수치(p99 doubling·CPU 56%) 복제 아님 — 스토리지/CPU noisy-neighbor 사례라 LLM 토큰 쿼터 간섭과 apples 다름" silent cap. 인용 정확성은 "확인 불가"로 정직 표기.
### (C) 쿼터 보조 별점 축 검토 (권고 3·C3)
- agentic 진짜 공유 장애 = 토큰 쿼터 독점인데 쿼터침범은 0건 게이트로만. → **쿼터 간섭(전역 풀 마름 시 타 WF 동반 저하 정도)을 보조 별점 축 검토** — 단 쿼터침범 0건은 절대형이라 gradable proxy(예: 폭주 강도별 타 WF 쿼터 확보율)로만 별점화 가능. 0건 절대형이면 Constraint 유지.

## 근거 (레퍼런스 + 검증 결과)
**C4 — 격리 표준**: bulkhead·token-bucket·fair queueing은 [KEDA](https://keda.sh/)·SRE 표준(Council.md §4 격리). simplyblock noisy-neighbor(p99 doubling·CPU 56%)는 스토리지/CPU 사례 — 본 검증에서 1차 대조 미완·도메인 다름 → "확인 불가, LLM 쿼터 간섭은 우리 PoC로". 외부 LLM rate-limit이 계정 전역이라 token-bucket은 client-side 분배만 보장(전역 풀 마르면 전 WF 동반 저하) silent cap 유지.

## PoC 증명법
### PoC-08: 격리 latency 증가 + 쿼터 침범 (격리 주입 noisy-neighbor 아키타입)
- 가설: "타 WF latency 증가 ★ 경계(5/10/25%)가 폭주 주입으로 측정되고 격리 On/Off로 변별된다."
- 지표: 폭주 주입 하 타 WF p95 증가율, 쿼터 침범 건수, 중단율. 합격선: 중단율 ≤1%·쿼터침범 0 게이트 하 ★ 변별.
- 셋업: bulkhead·큐별 동시성·token-bucket On/Off A/B, 한 WF 폭주 주입.
- 절차: ① 폭주 강도 sweep. ② 격리 On: 타 WF p95 단조 평탄 / Off: 급증 확인. ③ 쿼터 침범 0 유지 확인.
- 합격 기준: 격리 On에서 ≤5%·쿼터침범 0이 폭주 강도 무관 유지.
- 규모/기간: 폭주 강도 3단계 × 격리 On/Off, ~0.5일.
- 리스크/한계(silent cap): 전역 토큰 풀 마름은 token-bucket로 못 막음.

## DP·발표 영향
- DP-0004/0005 격리 — DP-0005 2안 공유 캐시 채택 시 ④(오염 전파 0)가 R-1과 충돌(OI-7).
