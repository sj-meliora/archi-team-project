# Counsel: QA-04 Observability — Agent 작업 추적 용이성 (round-02)

> refs-review: [round-02/review/QA-04-observability.md](../review/QA-04-observability.md)
> 직전 counsel: [round-01/counsel/QA-04-observability.md](../../round-01/counsel/QA-04-observability.md) (채택 권장 — trace 완전성 + event-history)
> seats: 발의 **Seat 3**(Runner 인프라) · 합의 **consensus**
> stance: **닫힘 확인(Med 해소) — 측정 인프라 토대(다수 QA 수급) · 잔여 = DP-0003 span 보장 위임**

## Reviewer 지적 요약
- round-02 verdict: **Sound ○ / KPI ○ · Low** (Med 해소). 모호한 "재구성 95%" → 4축(`trace 완전성 안전100%/일반≥95%` · event-history 100% · MTTD≤5분 · 결정당 비용/토큰 100%). span 환원으로 측정가능 닫힘.
- 잔여(비-verdict): DP-0003 "규칙 기반 한정" 시 span 범위 역검토(OI-7). 이 QA는 **비용·정확성·안전 PoC의 계측 토대** — 세트 수급 허브.

## 개선안 (정의·KPI 기존→제안)
KPI 닫힘 — 새 KPI 없음. 정의·KPI 유지. agent loop 전구간(prompt·context·tool I/O·model ver·token·ts) span 캡처 + traces·metrics·alerting 3축 유지. **NQA-B golden·NQA-C 비용·QA-07 오버헤드가 이 trace에서 수급**됨을 cross-link 강조.
> ⚠️ `100%·95%·5분`은 "측정 가능 KPI의 모양" 예시값.

## 근거 (레퍼런스)
§4 **관측성(Observability)** 행.
- **OTel GenAI semconv**: `invoke_agent/chat/execute_tool` span + `gen_ai.usage.*` 토큰·model·finish_reason → trace 완전성. https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/ · https://opentelemetry.io/blog/2025/ai-agent-observability/
- event-history 완전성 = durable execution과 동일 엔진(QA-02 공유). https://docs.temporal.io/temporal

## PoC 증명법
### PoC-O1(R1 유지, 토대 — 먼저): span/event-history 완전성 (계측)
- **가설**: "OTel GenAI span 삽입 후 event-history/span 완전성이 측정 가능하며, 토큰 계측을 비용·정확성 PoC가 재사용한다."
- **지표+합격선**: trace 완전성(안전 100%/일반 ≥◯%) · event-history 100% · MTTD ≤◯ · `gen_ai.usage.*` 토큰 기록 100%.
- **셋업**: 워크플로우에 OTel GenAI 계측 삽입(+durable 엔진).
- **절차**: agent loop 전구간 span 삽입 → 완전성 % 측정 → 토큰 계측을 N-C1·P2가 재사용.
- **합격(Exit)**: 완전성 ≥목표 ∧ 토큰 계측이 PoC-N-C1·P1 오버헤드에 공급(허브 검증).
- **규모/기간**: Group 1 토대(먼저 깔아야 비용·성능 PoC 측정) — 약 3~4일.
- **silent cap**: span 커버리지 = 재구성 신뢰 상한(미계측 경로 사각). DP-0003 규칙 기반 한정 시 span 범위 의존.

## DP·발표 영향
- **DP 위임 (OI-7)**: DP-0003이 "규칙 기반 한정"일 때 **span 수준 trace 보장 범위** 역검토. DP 디스커션 위임.
- **번호/서사**: 다수 QA의 측정 인프라 토대 — 발표에서 "관측 가능성이 다른 KPI를 측정 가능케 한다"는 횡단 메시지. importance 변동 없음.
