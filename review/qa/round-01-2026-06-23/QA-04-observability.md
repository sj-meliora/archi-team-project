# Review: QA-04 Observability — Agent 작업 추적 용이성

> source: `context/qa/QA-04-observability.md` + `QAS-04-observability.md`
> verdict: **Sound ○ / KPI △** — 타당한 QA이나 KPI의 운영 정의가 모호하고 trace 구체화 필요 · severity **Med**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트

## 원문 요약
- **정의**: 실행 과정·판단 근거 추적.
- **KPI**: 의사결정 재구성 가능 비율 ≥ 95%
- **QAS**: 자극=의사결정 사후 조회 요청 / 응답=실행 과정·판단 근거 재구성 제공.

## 렌즈 1 — Agentic Workflow 전문가 관점

**Agentic 시스템에서 observability는 agent loop 전구간의 trace를 의미한다.** 비결정성 때문에 “왜 이 결정을 했나”를 사후에 못 보면 디버깅·신뢰가 불가능하다. “의사결정 재구성”이라는 의도는 정확하나, 무엇을 캡처해야 재구성이 되는지를 못 박아야 한다:
- 한 결정의 재현에 필요한 최소 단위 = **prompt + 입력 context + tool 호출/결과 + model 버전 + 토큰 사용량 + timestamp**. 이 span이 빠지면 “95% 재구성”은 빈말이 된다.
- **비용·품질 신호도 같은 trace에 실어야** 한다: 결정당 토큰/비용(→ QA-05 연계), eval 점수. 관측성이 efficiency·correctness의 측정 인프라가 된다.
- OpenTelemetry **GenAI semantic convention**(agent step별 span) 같은 표준을 KPI 근거로 쓰면 “재구성 가능”이 검증 가능한 trace 완전성으로 환원된다.
- **안전 관련 액션(배포·권한 사용)은 95%가 아니라 ~100%** 추적돼야 한다(제어성·보안과 연계).

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **“재구성 가능 비율 95%”의 운영 정의 부재**: 단위가 결정 1건인가 워크플로우인가? “재구성 가능”을 누가 어떻게 판정하나(이진? 감사자 판단?)? 정의 없이는 95%가 측정 불가.
- **관측성의 폭이 좁다**: 정통 observability = traces + metrics + logs 3축 + alerting. 현재는 trace(결정 재구성) 1축만. **운영 지표 대시보드**(처리량·에러율·비용)와 **이상 탐지 시간(MTTD)**가 빠졌다 — MTTD는 QA-02(MTTR)의 전제이기도 하다.
- KPI를 **trace 완전성**(액션 중 full-span 보유 비율)으로 환원하면 측정이 명확해진다.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

**워크플로우 엔진은 “의사결정 재구성”을 기본 제공한다 — 그 위에 trace를 얹어라.**
- **이벤트 히스토리 = replay**: durable 엔진은 모든 워크플로우 전이를 이벤트로 남겨 **결정론적 재생(replay)**이 가능하다. QA-04의 “재구성 95%”를 이 **event-history 완전성**으로 환원하면 측정이 명확해진다(워크플로우 골격은 ~100% 재구성; LLM 비결정 부분만 prompt/seed 캡처로 보강).
- **분산 트레이싱 전파**: worker hop을 넘는 trace context 전파(W3C traceparent) → agent step span(렌즈1)과 워크플로우 span을 한 trace로 연결.
- **task 텔레메트리 표준화**: 큐 대기·실행시간·retry 횟수·자원 사용을 runner가 기본 수집 → metrics 축(렌즈2)을 공짜로 채운다.
- **runner 측 KPI**: event-history 완전성 100%, trace 전파 커버리지, 안전 액션 span 100%.

## 판정

| 항목 | 판정 | 근거 |
|---|---|---|
| QA 자체가 sound한가 | ○ | 자율 시스템 디버깅·신뢰의 토대. 타당 |
| KPI가 측정 가능한가 | △ | “재구성 가능”의 단위·판정 기준 미정의 |
| KPI가 현실적/적절한가 | △ | 95%는 합리적 목표이나 trace 항목·MTTD 등 누락 |
| 정의↔KPI↔QAS 일치 | ○ | 셋 다 “판단 근거 재구성”으로 일관 |

## Stage 2 권고

- **KPI 구체화 (예시)**:
  - `재구성 95%` → **`trace 완전성: 안전관련 액션 100% / 일반 액션 ≥95% (span = prompt+context+tool I/O+model ver+token+ts 포함)`**
  - 추가: **`MTTD(이상 탐지) ≤ ◯분`**, 결정당 비용/토큰 기록 100%
- **관측 3축 명시**: trace 외 metrics(처리량·에러율·비용 대시보드)·alerting을 정의에 포함.
- **연계**: QA-05(결정당 비용)·QA-03/NQA-A(안전 액션 100% 추적)·QA-02(MTTD→MTTR).
- DP 연결 점검: DP-0003(실시간 모니터링)이 span 수준 trace를 보장하는지 역검토.
