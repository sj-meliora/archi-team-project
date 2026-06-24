# Counsel: QA-02 Availability — 운영 안정성 (round-02)

> refs-review: [round-02/review/QA-02-availability.md](../review/QA-02-availability.md)
> 직전 counsel: [round-01/counsel/QA-02-availability.md](../../round-01/counsel/QA-02-availability.md) (채택 권장 — MTTR 분해·무손실 + 외부 LLM 장애)
> seats: 발의 **Seat 3**(Runner 인프라) · 합의 **consensus**
> stance: **닫힘 확인(High 해소) — 잔여 = 외부 LLM degradation DP 위임·예시값**

## Reviewer 지적 요약
- round-02 verdict: **Sound ○ / KPI ○ · Low** (R01 High 해소). MTTR 단독 폐기 → 4축(`재기동≤1분 AND in-flight 손실0` · 가용률≥99.5% · 외부 LLM 자동 재개율≥95% · side-effect 멱등100%). durable state 정착.
- 잔여(비-verdict): 외부 LLM degradation backoff·폴백이 DP-0002/0003에 미명시(OI-7) + DP-0001 `drives: QA-02`→`QA-06` 교정. 예시값 확정.

## 개선안 (정의·KPI 기존→제안)
KPI 닫힘 — 새 KPI 없음. 정의·KPI 유지. 가용성 1순위 = "빨리 재기동"이 아니라 **무손실** 강조 유지. QA-06(격리)과 `> altitude` 경계 유지.
> ⚠️ `1분·99.5%·95%·100%`는 "측정 가능 KPI의 모양" 예시값 — 실측/SLO 합의로 확정.

## 근거 (레퍼런스)
§4 **가용성·복구(Availability)** 행.
- **SLO(목표%) + error budget**; MTTR을 durable execution으로 분해(lease timeout+재스케줄), event sourcing 무손실. https://sre.google/sre-book/service-level-objectives/
- **Temporal durable execution**(event history·exactly-once) — in-flight 손실0·멱등 재개. https://temporal.io/blog/what-is-durable-execution
- 외부 LLM 장애 graceful degradation·backoff = 재시도·폴백 표준(SRE error budget 소비 관리).

## PoC 증명법
### PoC-A1·A2(R1 유지): 무손실·멱등 재개 + 외부 LLM degradation (장애주입 chaos)
- **가설**: "노드/제공자 강제종료에도 작업 손실 0·멱등 재개, 외부 LLM 장애 시 자동 재개가 측정 가능."
- **지표+합격선**: 손실 0·중복 0·재개 ≤1분 · 외부 LLM 자동 재개율 ≥◯% · backpressure 작동.
- **셋업**: durable 엔진(event-history — O1·K1과 공유) + 노드 강제종료 chaos + 외부 LLM mock outage/429.
- **절차**: 진행 중 작업에 chaos 주입 → 손실·재개 시간 측정 + LLM outage 주입 → 자동 재개율.
- **합격(Exit)**: 손실 0 ∧ 멱등 100% ∧ 재개 ≤목표.
- **규모/기간**: 약 2~3일(O1과 엔진 공유).
- **silent cap**: 외부 시스템(Jira·빌드서버·vault) 멱등성은 통합 환경에서만 검증.

## DP·발표 영향
- **DP 위임 (OI-7)**: DP-0002/0003에 **외부 LLM degradation backoff·폴백** tactic 미명시. + DP-0001 `drives: QA-02`→`QA-06` 교정(격리는 QA-06). DP 디스커션 위임.
- **번호/서사**: "무손실 연속성"이 자율 운영 신뢰의 토대 — 발표 안정성 축. importance 변동 없음.
