# Review: QA-04 Observability — Agent 작업 추적 용이성 (round-02 재평가)

> source: `context/qa/QA-04-observability.md` + `QAS-04-observability.md` (round-01 [반영] 후)
> 직전 verdict(round-01): **Sound ○ / KPI △ — Med**
> verdict(round-02): **Sound ○ / KPI ○ — Low** — 모호한 "재구성 95%" → span 완전성·event-history·MTTD·결정당비용 4축으로 환원해 측정가능성 회복(Med 해소). 잔존은 DP-0003 span 보장 역검토(OI-7)뿐 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 QA-04 = **[반영]** (재구성95% → trace 완전성(span 정의)·event-history·MTTD·결정당 비용). 이월: DP-0003 span 보장 역검토(OI-7).

## 원문 요약 (반영 후)
- **정의**: agent loop 전구간(prompt·context·tool I/O·model ver·token·ts)을 span으로 캡처 + traces·metrics·alerting 3축. 다수 QA(비용·정확성·안전)의 측정 인프라 토대.
- **KPI 4축**: 주 = `trace 완전성(안전 100%/일반 ≥95%)`. 보조 = `event-history 완전성=100%` · `MTTD ≤5분` · `결정당 비용/토큰 기록=100%`.
- **QAS-04**: Response·Measure를 span 완전성·event-history·MTTD로 동기화.

## 렌즈 1 — Agentic Workflow 전문가 관점

round-01 핵심 지적("재구성 가능"이 빈말 → 재구성에 필요한 **최소 span을 명시하라")이 정확히 반영됐다. span = `prompt + 입력 context + tool I/O + model 버전 + token + timestamp`로 구성요소가 명시돼, "재구성 가능"이 검증 가능한 trace 완전성으로 환원됐다. OTel GenAI span 표준에 앵커한 것도 적절. **닫혔다.**

agentic 관점에서 이 QA가 **세트 전체의 측정 토대**라는 자각이 정의에 들어온 게 큰 진전이다 — QA-05/NQA-C(결정당 비용)·QA-03/NQA-A(안전 액션 100%)·QA-02(MTTD→MTTR)가 모두 QA-04 데이터를 소비한다. 즉 QA-04가 닫히지 않으면 여러 QA의 KPI가 데이터 없이 뜬다. 이 의존을 정의가 명시한 것은 모범적.

- **잔여(silent cap)**: LLM 내부 추론(CoT)은 비공개라 prompt/출력으로만 근사 — "왜 이 결정을 했나"의 *진짜 이유*는 trace로 못 본다. KPI는 "재구성 가능한 외형"만 보장. 이건 LLM 본질적 한계라 신규 결함 아님 — 정의에 이미 명시됨.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **measurable 회복**: round-01의 치명상(`재구성 ≥95%`의 단위(결정 1건? WF?)·판정기준(이진? 감사자?) 미정의 → 측정 불가)이 **span 완전성 + event-history 완전성**으로 환원됐다. 분모(액션 종류·전이 수)가 정의돼 acceptance 산출 가능. **measurable 합격.**
- **Sound ○ 유지**: 관측 인프라 단일 관심사. 단 정의가 "측정 토대"로 넓어지며 *다른 QA의 측정 데이터 공급*까지 포함했는데, 이는 altitude를 흐리지 않고 오히려 경계를 명확히 했다(QA-04는 데이터 공급, 합격 임계는 각 QA가 정함). consistency ○.
- **남은 형식 결함(경미)**: `"재구성됨"의 판정 자동화 한계`(감사자 판단 개입 여지)가 검증 전략 silent cap에만 있다. 주 KPI `trace 완전성`은 *span 누락률*로 자동 산출되므로 판정 주관성이 낮지만, "재구성 성공"을 *사람이 보고* 판정하면 다시 주관 개입 → KPI를 **span 누락률 기준(자동)**으로 한정함을 명문화 권고(Low). 현재 KPI는 이미 그렇게 쓰여 있어 위험 낮음.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

이 QA도 durable execution이 깔끔하게 환원된 사례다:
- **event-history 완전성=100%**는 durable 엔진(event sourcing)의 정의적 성질 — 모든 전이를 이벤트로 남겨 골격을 결정론적 replay. LLM 비결정 부분만 prompt/seed로 분리 보강. runner 메커니즘으로 직접 측정 가능. **양호.**
- **잔여(DP 위임, OI-7 — 이번 라운드 진짜 잔여)**: 주 KPI `trace 완전성`을 **DP-0003 3안 실시간 모니터링**에 귀속시키나, OI-7이 명시하듯 **DP-0003이 span 수준 trace(6요소)를 보장하는지 미명시**다. 특히 DP-0003을 "규칙 기반 한정 적용"으로 권고했는데, 그 경우 *어디까지 span을 남기는지* 명시가 없으면 trace 완전성 95%/100%의 책임 설계가 비어 있다 → DP 디스커션 위임.
- **runner 측 KPI**(재확인): span 드롭률(샘플링 손실), W3C traceparent 전파 성공률, event-history replay 성공률, 계측 오버헤드(span 부착이 latency에 주는 비용 — 관측이 성능을 잡아먹지 않는지).

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | ○ | **○** | 관측 인프라 단일 관심사 + 측정 토대로서의 공급 경계 명확화 |
| KPI가 측정 가능한가 | △ | **○** | `재구성 95%` 모호 폐기 → span 완전성·event-history로 분모 정의, 자동 산출 가능 |
| KPI가 현실적/적절한가 | △ | **○** | OTel GenAI 표준 앵커, 안전 액션 100%/일반 95% 차등 현실적. CoT 한계는 명시됨 |
| 정의↔KPI↔QAS 일치 | ○ | **○** | QAS-04 Measure가 span·event-history·MTTD로 동기화 확인 |

**verdict 변화: Sound ○→○ / KPI △→○ · severity Med→Low.** round-01 Med의 근거(재구성 운영정의 부재)는 해소됨.

## Stage 2 권고 (round-02)

대부분 닫혔으므로 **DP 위임·잔여 보강(Low)만** 남긴다:

- **(DP 디스커션 위임, OI-7 — 최우선 잔여)** DP-0003에 **span 수준 trace 보장**(6요소: prompt+context+tool I/O+model ver+token+ts)을 명시 — "규칙 기반 한정 적용" 시 어디까지 span을 남기는지. trace 완전성 KPI의 책임 설계가 현재 비어 있음.
- **(자동 판정 한정, Low)** `trace 완전성` acceptance를 **span 누락률 기준(자동 산출)**으로 한정 명문화 — "재구성 성공"을 사람이 판정하는 주관 개입 차단.
- **(계측 오버헤드 가드, Low)** 관측이 성능을 잡아먹지 않도록 *계측 오버헤드 ≤ N%* 가드 KPI 추가 검토(관측의 self-cost).
- **(예시값 확정)** `MTTD 5분·일반 trace 95%`는 실환경 측정으로 확정.
