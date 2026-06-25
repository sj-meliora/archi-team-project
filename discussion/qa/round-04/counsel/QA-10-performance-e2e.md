# Counsel: QA-10 Performance — E2E 개발 시간

> refs-review: round-04/review/QA-10-performance-e2e.md
> seats: 발의 Seat 3 (단계 분해·critical path) · 합의 consensus
> stance: **채택 권장 — ★☆☆ 사문화 교정 모범, ★★★ 2h margin 근거화 + agentic LLM 큐잉 가산 명시**

## Reviewer 지적 요약
E2E 6h→24h 재배치는 "보수적 하한(★☆☆ 사문화)" 반대 방향 문제를 정확 교정한 **세트 유일** 케이스(규칙2). 잔여(Low): (1) **★★★ 2h = "6h의 1/3"인데 왜 1/3인지 근거 없음**(권고 1·C2), (2) 배치 ML SLA에 agentic LLM 호출 큐잉·rate-limit 지연 미반영(권고 2).

## 개선안 (정의·KPI 기존→제안)
### (A) ★★★ 2h margin 근거화 (권고 1·C2)
- ★ 노트 기존: `★★★ ≤2h = 배치 표준 하한 6h의 1/3`(왜 1/3 근거 없음). → 제안: **`★★★ ≤2h = 완전자율 병렬 파이프라인 도달 대역 + Quantize/Compile compute가 critical path라 그 이하 불가 → critical path 흡수 ≈2h`**. 1/3이 임의가 아니라 "compute critical path 하한"임을 근거화.
### (B) agentic E2E 가산 명시 (권고 2)
- ★ 노트: `배치 ML SLA(6~24h)는 전통 ML 파이프라인 기준 — agentic E2E엔 LLM 호출 대기·rate-limit 큐잉이 가산됨(외부 LLM 느리면 배치 ML 관례 초과 가능). E2E = 배치 ML + LLM 큐잉`.

## 근거 (레퍼런스 + 검증 결과)
**C4 — 배치 ML SLA 출처**: 배치 ML 파이프라인 SLA 6~24h(domo·datadef류)는 본 검증에서 1차 대조 미완 → "확인 불가, nightly 배치 관례 대역 정당화용". p95 SLI는 [Google SRE](https://sre.google/sre-book/service-level-objectives/)·percentile 가이드(Council.md §4 지연) — 평균 금지·p95 측정 표준. 우리 워크로드(배치 SDK 빌드)와 도메인 정합 양호(C1 우려 낮음).

## PoC 증명법
### PoC-10: E2E p95 단계 분해 (부하시험 아키타입)
- 가설: "E2E p95 ★ 경계(2/6/24h)가 throughput SLO 부하 하 측정되고 critical path가 식별된다."
- 지표: E2E p50/p95(compute/agent 루프/큐/handoff 단계 분해), throughput ≥50/일, 전달 무결성 100%. 합격선: throughput 충족 부하에서 ★ 경계 변별.
- 셋업: claim-check(원격 A5) vs 로컬(A8) 2모드 부하시험, mock compute(Quantize/Compile 분포).
- 절차: ① throughput ≥50/일 부하 인가. ② E2E p95 단계 분해. ③ critical path 식별(compute vs LLM 큐잉).
- 합격 기준: critical path가 드러나고 ★★★ 2h가 compute 하한과 일치.
- 규모/기간: 2모드 부하시험, ~0.5일.
- 리스크/한계(silent cap): mock compute가 실제 Quantize/Compile 분포 미근사 시 왜곡. LLM 큐잉 가산은 외부 의존.

## DP·발표 영향
- DP-0004 결정 산식(5% budget → A5 vs A8)은 실측·노드 사양 의존(OI-7). QA-01 throughput 정렬.
