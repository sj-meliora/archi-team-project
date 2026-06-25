# Counsel: QA-04 Observability

> refs-review: round-04/review/QA-04-observability.md
> seats: 발의 Seat 3 (OTel span·누락 분해) · 합의 consensus
> stance: **채택 권장 — ★★★ 캡 모범, 경계 표기 명확화 + 완전성 정의(존재 AND 비-truncation) 보강**

## Reviewer 지적 요약
★★★ 100% 미만(≥98%) 캡은 CoT 비공개 정직 반영 모범(OTel GenAI semconv apples 정합 — C1 우려 최저). 잔여(Low): (1) **★★☆ (95,98) 3%p 대역 좁음 + ★☆☆ ≥95% vs ★★☆ >95% 경계 모호**(권고 1), (2) "완전성 %"가 span 존재율이지 충실도(truncation) 미반영(권고 2).

## 개선안 (정의·KPI 기존→제안)
### (A) 경계 표기 명확화 (권고 1)
- ★ 급간 기존: `★☆☆ ≥95% / ★★☆ 95~98% / ★★★ ≥98%`(95.0%가 어디인지 모호). → 제안: **`★☆☆ [95,97) / ★★☆ [97,98) / ★★★ [98,100)`**으로 구간화(≥/> 일관, ★★☆ 1%p로 더 좁아지나 명확). 또는 ★★☆ [96,98)로 ★☆☆ [95,96)와 분리해 변별 여유. → Council 권장: **[95,97)/[97,98)/[98,100)** (auto-instrumentation이 95~98%에 몰리는 현실을 ★★☆ 좁게 두고 명확화).
### (B) 완전성 정의 강화 (권고 2)
- ## 측정: `완전성 = span 존재 AND 비-truncation(prompt/context 잘림 없음)` — 존재하나 잘린 span 과대 카운트 방지.

## 근거 (레퍼런스 + 검증 결과)
**C4 — OTel GenAI semconv 실재**: [OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/)(invoke_agent/chat/execute_tool span + gen_ai.usage.*)는 실재 표준(Council.md §4 관측성) — 우리 측정 대상(agent 단계 계측)과 apples-to-apples 정합. event-history 100% 게이트는 durable 엔진 결정론적 replay로 구조적 보장(가정 아님).

## PoC 증명법
### PoC-04: trace 완전성 + 누락 카테고리 분해 (계측 instrumentation 아키타입)
- 가설: "trace 완전성 ★ 경계(95/97/98%)가 OTel span 자동계측으로 측정되고, 누락 카테고리가 분해된다."
- 지표: span 완전성 %(존재 AND 비-truncation), 누락 카테고리(표준/커스텀 tool/외부 호출), event-history 100%. 합격선: 안전 액션 100%·event-history 100% 게이트 하 일반 완전성 ★ 변별.
- 셋업: OTel GenAI span 삽입한 mock 워크플로우, 커스텀 tool·외부 호출 포함.
- 절차: ① span 자동계측 커버리지 측정. ② 누락 2% 카테고리 분해. ③ truncation 검사.
- 합격 기준: 누락 위치가 카테고리별로 드러나 ★★★ 98% 도달 가능성 판정.
- 규모/기간: N개 액션 종류, ~0.3일.
- 리스크/한계(silent cap): 분모(액션 종류)가 작으면 98% 변별 표본 부족.

## DP·발표 영향
- DP-0003이 span 수준 trace 보장하는지(규칙 기반 한정 시 어디까지)(OI-7). 다수 QA 측정 인프라 토대.
