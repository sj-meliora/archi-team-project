---
id: QA-04
category: QA
importance: M
difficulty: M
source: pptx p.13
related-dp: [DP-0003]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "모호한 '재구성 95%' → span 완전성·event-history·MTTD·결정당 비용 4축으로 환원 (자세히 → ## 변경 이력)"
  - date: 2026-06-24
    by: discussion/qa/round-03 (등급 척도 캘리브레이션)
    reason: "★ rubric 추가 — main 축=일반 액션 trace 완전성(정방향), 안전100·event-history100은 게이트. ★★★ 상한 100% 미만(≥98%) 캡(CoT 비공개 한계) (자세히 → ## 변경 이력)"
  - date: 2026-06-25
    by: discussion/qa/round-04 (★ 등급 척도 근거 보강)
    reason: "경계 표기 구간화([95,97)/[97,98)/[98,100), ≥/> 모호 제거) + 완전성 정의 강화(span 존재 AND 비-truncation) (자세히 → ## 변경 이력)"
---

# QA-04 Observability — Agent 작업 추적 용이성

## 정의 / Refinement
Agent loop 전구간(**prompt·입력 context·tool 호출/결과·model 버전·token·timestamp**)을 **span(추적 단위)**으로 캡처해 한 의사결정을 사후 재구성하고, **traces·metrics·alerting 3축**으로 운영을 관측한다. 비결정적 LLM 시스템에서 "왜 이 결정을 했나"를 사후에 못 보면 디버깅·신뢰가 불가능하므로, 이 QA는 다른 여러 QA(비용·정확성·안전)의 **측정 인프라 토대**가 된다.

설계할 때 잡아야 할 두 가지 관점:

- **무엇을 캡처해야 재구성되나(trace 완전성)** — 한 결정의 재현에 필요한 최소 단위 span = **prompt + 입력 context + tool I/O + model 버전 + token 사용량 + timestamp**. 이 중 하나라도 빠지면 "재구성 가능"은 빈말이 된다. 배포·권한 사용 같은 **안전 관련 액션은 95%가 아니라 ~100%** 추적돼야 한다(제어성·보안과 연계).
- **관측은 3축이다(traces·metrics·alerting)** — trace(결정 재구성) 1축만으론 좁다. 운영 지표 대시보드(처리량·에러율·**결정당 비용**)와 **이상 탐지 시간(MTTD)**까지 갖춰야 정통 observability다. durable 엔진의 **event-history(모든 워크플로우 전이를 이벤트로 기록)**로 골격은 ~100% 재생되고, LLM 비결정 부분만 prompt/seed 캡처로 보강한다.

> 이 QA는 "관측 인프라를 갖춘다"까지만 다룬다 — 그 trace로 재는 **결정당 비용**은 QA-05/QA-13, **안전 액션 100% 추적**은 QA-03/QA-06, **MTTD→MTTR**는 QA-02에서 각각 활용한다. 즉 QA-04는 측정 데이터를 *공급*하고, 합격 임계는 각 QA가 정한다.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `trace 완전성` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **trace 완전성: 안전관련 액션 100% / 일반 액션 ≥ 95%** `[주 KPI · PoC 대상]` — span = prompt+context+tool I/O+model ver+token+ts. **완전성 = span 존재 AND 비-truncation(prompt/context 잘림 없음)** (round-04)
  - 쉽게: 한 결정을 되짚는 데 필요한 정보(어떤 프롬프트로·무슨 입력에·어떤 도구를 써서·어떤 모델이·토큰 얼마로·언제)가 다 남아 있어야 한다. 배포·권한 같은 위험한 행동은 100%, 일반 행동은 95% 이상. **단순히 span이 "존재"하는 것만이 아니라 prompt/context가 잘리지 않고(non-truncation) 온전히 남아야 완전한 것으로 센다** — 존재하나 잘린 span을 과대 카운트하지 않기 위함. *span = 한 동작의 추적 기록 단위.*
  - > 보정(2026-06-24, 등급 척도 캘리브레이션 round-03): 일반 액션 합격 하한 `≥95%` 유지(재배치 불요). 단 **★★★ 상한을 100% 미만(≥98%)으로 캡** — LLM 내부 CoT 비공개·비결정 구간은 prompt/seed 근사로만 메우므로 일반 액션 trace 완전성 100% 단정은 과대주장. 안전관련 액션 100%·event-history 100%는 게이트(불변). ★ 급간은 ## 등급 척도 참조.
- **event-history 완전성 = 100%** — 워크플로우 골격 재생(durable 엔진 기준)
  - 쉽게: 워크플로우가 거쳐 간 모든 단계 전이가 이벤트로 빠짐없이 남아, 골격은 100% 그대로 재생(replay)할 수 있어야 한다. LLM의 비결정적 부분만 prompt/seed로 따로 보강. *event-history = 모든 상태 변화를 순서대로 적은 기록, replay = 그 기록으로 실행을 그대로 되돌려 재생.*
- **MTTD(이상 탐지 시간) ≤ 5분** — 이상 발생→탐지까지
  - 쉽게: 뭔가 잘못됐을 때 5분 안에 알아채야 한다. 이건 QA-02의 복구시간(MTTR)이 시작되기 위한 전제다. *MTTD = Mean Time To Detect(평균 탐지 시간).*
- **결정당 비용/토큰 기록 = 100%** — QA-05/QA-13 측정 인프라
  - 쉽게: 모든 의사결정에 든 토큰·비용이 trace에 100% 기록돼야, 효율·비용 QA가 그 데이터를 가져다 쓸 수 있다.

> 위 수치(100%·95%·5분)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 실제 환경에서 측정해 확정한다.
> 폐기: 旧 `의사결정 재구성 가능 비율 ≥ 95%` 단독 — "재구성 가능"의 단위(결정 1건? WF?)·판정 기준(이진? 감사자?)이 미정의라 측정 불가. → span 완전성·event-history로 환원.

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 관측성·계측의 업계 표준에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **span 완전성 (OTel GenAI)** | `invoke_agent/chat/execute_tool` span + `gen_ai.usage.*`(토큰)·model·finish_reason로 agent 단계를 표준 계측 → "재구성 가능"이 검증 가능한 trace 완전성으로 환원 | [OTel — GenAI agent spans](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/) · [OTel — AI agent observability](https://opentelemetry.io/blog/2025/ai-agent-observability/) |
| **event-history replay** | durable 엔진은 모든 전이를 이벤트로 남겨 결정론적 재생 → 워크플로우 골격 ~100% 재구성 (LLM 비결정만 보강) | [Temporal — 문서(event history)](https://docs.temporal.io/temporal) |
| **3축(traces+metrics+logs)·MTTD** | 관측 3축과 MTTD/alerting은 SRE 표준 — trace 1축만으론 운영 관측 불충분 | [Google SRE Workbook — SLO 구현](https://sre.google/workbook/implementing-slos/) |

> ⚠️ 레퍼런스의 수치(95%·MTTD 분 등)는 **패턴 정당화용**이며 그대로 복제하지 않는다. 우리 합격선은 위 [검증 전략](#검증-전략)의 실측·모델로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 계측/시뮬레이션**으로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **trace 완전성** `[주]` | **DP-0003 3안 실시간 모니터링**([Observability] 실행·판단 근거 추적) — 단, **규칙 기반 한정 적용** 권고(LLM 탐지는 토큰 폭증). **span 수준 trace를 DP-0003이 보장하는지 역검토 필요** | **▶ 실제 제작:** OTel GenAI span 계측 모델 — mock 파이프라인에 `invoke_agent/execute_tool` span 삽입 + W3C traceparent 전파 → 워크플로우 1건 실행 후 한 결정을 **trace만으로 재구성** 시도 → 누락 span 카테고리(분모: 액션 종류)를 집계해 완전성 산출 |
| event-history 완전성 = 100% | durable 엔진(Temporal류)의 event-history — 워크플로우 골격 결정론적 재생 | 보조 모델: 위 모델에 durable event-history 붙여 골격 replay 성공률 집계 (LLM 호출만 비결정으로 분리 표시) |
| MTTD ≤ 5분 | **DP-0003 3안 모니터링**(빠른 탐지·복구로 MTTR 단축) + 간단 이상탐지 룰 | 보조 모델: 이상(에러·이탈) 주입 → 탐지까지 시간 분포 산출 |
| 결정당 비용/토큰 기록 = 100% | OTel `gen_ai.usage.*` 토큰 계측(QA-05/QA-13 공급) | 보조 모델: span에 토큰/비용 필드 부착 후 결정 N건 기록 누락률(=0 목표) 집계 |

> 가정·한계: 결정 수·이상 주입 빈도·이상탐지 룰은 **가정 파라미터**다. 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다 — 슬라이드엔 가정값을 명시한다. **"재구성됨"의 판정 자동화 한계**(감사자 판단 개입 여지)와 LLM 내부 추론(CoT 비공개)은 prompt/출력으로만 근사하는 점은 silent cap으로 명시.

## 등급 척도 (★ rubric — ATAM trade-off용)

> 동일 조건 설계 대안의 본 QA 만족도를 ★1~3 비교(별 많은 안 채택). KPI 합격선(하한)=★☆☆ 진입선, ★★☆/★★★는 필드 현실 도달 범위+PoC margin. 하한 미만 불합격. 예시값이며 경계는 PoC로 확정.

> 헤드라인 `trace 완전성: 안전 100% / 일반 ≥95%`. **안전관련 액션 100%는 게이트/조건(규칙5)** — 0건 누락 절대형이라 별점 축으로 부적합. 별점 main = **일반 액션 trace 완전성(gradable %)**, 높을수록 ★ 높음(정방향).

**조건 (2-index, 규칙4):** `조건: 안전관련 액션(배포·권한사용·삭제) span 완전성 = 100% 고정` — 안전 액션은 제어성·보안(QA-03/QA-06)과 연계되어 누락이 허용 안 되는 게이트. 이 게이트를 통과한 전제에서 **일반 액션 완전성 %** 로 설계 대안을 변별한다.

| 등급 | 구간 — 주 KPI(main 축): 일반 액션 trace 완전성 % (정방향, 높을수록 상) | 필드 근거 (경계 이유 + URL) |
|---|---|---|
| ★★★ (상) | **[98, 100) (100% 미만 허용)** | **(round-04 구간화: ≥98% → [98,100), ≥/> 모호 제거)** OTel GenAI semconv(1.37+)는 `invoke_agent`/`chat`/`execute_tool` span 자동계측으로 reasoning chain 전체를 child span으로 남겨 완전성 상단이 매우 높음. 다만 LLM 내부 CoT 비공개·일부 비결정 구간은 prompt/seed 근사로만 메우므로 100% 단정은 과대주장 → ★★★ 상한을 100% 직전으로 두고 margin. [OTel GenAI semconv — full span tree](https://opentelemetry.io/blog/2026/genai-observability/) · [Uptrace — AI agent OTel](https://uptrace.dev/blog/opentelemetry-ai-systems) |
| ★★☆ (중) | **[97, 98)** | **(round-04 구간화: 95~98 → [97,98), auto-instrumentation 몰림 대역을 좁고 명확하게)** auto-instrumentation(OpenAI/Anthropic/LangChain)로 표준 span은 거의 다 잡히나 커스텀 tool·외부 호출 누락이 남는 일반 우수 대역. [Datadog OTel GenAI semconv 지원](https://www.datadoghq.com/blog/llm-otel-semantic-convention/) |
| ★☆☆ (하) | **[95, 97) (= KPI 하한·합격 최소선)** | KPI 정의 하한 `일반 ≥95%`가 ★☆☆ 진입선. OTel GenAI semconv가 output 평가·content 품질은 안 덮으므로(span 완전성 ≠ 의미 완전성) 95%가 "재구성 가능"의 현실 진입선으로 타당 — 비현실적으로 높지 않아 재배치 불요. [Fiddler — OTel가 덮지 못하는 것](https://www.fiddler.ai/blog/opentelemetry-ai-observability-guide) |
| 불합격 | 일반 완전성 < 95% / 안전 액션 < 100%(게이트 위반) / event-history 완전성 < 100% | — |

**게이트 (별점과 AND, 규칙5):**
- **안전관련 액션 span 완전성 = 100%** (조건축, 위 명시).
- **event-history 완전성 = 100%** (durable 엔진 골격 replay) — 결정론적 재생이라 100% 절대형 게이트.
- (보조 metrics: MTTD ≤5분, 결정당 비용/토큰 기록 100% — 별점 미사용.)

> **캘리브레이션 노트**: 하한 `95%`는 필드 기준 비현실적으로 높지 않음(span 완전성 ≠ 결정 의미 재현) → 규칙2 재배치 불요. `이론 근거 = OTel GenAI 자동계측 full span tree(천장 ~100%)` / `PoC margin = CoT 비공개·비결정 구간 prompt/seed 근사 한계를 반영해 ★★★를 ≥98%로(100%에 붙이지 않음)`. silent cap: **"재구성됨" 판정 자동화 한계**(감사자 판단 개입 여지) + **LLM 내부 추론(CoT) 비공개**는 prompt/출력으로만 근사 → 완전성 %가 "span 존재율"이지 "의미 재현율"을 100% 보장하지 않음.
> **재캘리브레이션(round-04)**: 경계를 **구간 표기 `[95,97)/[97,98)/[98,100)`** 로 명확화(round-03 `≥95 / 95~98 / ≥98`의 ≥/> 모호 — 95.0%·98.0%가 어디인지 — 제거; auto-instrumentation이 95~98%에 몰리는 현실 반영해 ★★☆를 [97,98) 1%p로 좁게). **완전성 정의 강화**: `완전성 = span 존재 AND 비-truncation(prompt/context 잘림 없음)` — 존재하나 잘린 span 과대 카운트 방지(C 보강). 수치 하한(95%)·천장 캡(<100%)은 불변이라 OI-9 §측정 하한 보정은 없음(표기·정의만 정밀화) — 등급표↔변경이력 동기화만.
> **seats**: 발의 Seat 2(KPI 측정가능성·SLI 형식 — 안전/일반 2-index 분리) · consensus (Seat 1: CoT 비공개로 ★★★ 100% 미만 캡 동의 / Seat 3: event-history 100%는 durable 엔진 결정론적 replay라 게이트로 동의)

## 변경 이력

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-04-observability.md) (red team verdict: **Sound ○ / KPI △ — Med** — "재구성" 운영 정의 부재 + trace span·MTTD 누락)

**무엇이 문제였나 (review 지적)**
- `재구성 가능 비율 ≥ 95%`의 **운영 정의 부재** — 단위(결정 1건? WF?)·판정 기준(이진? 감사자?)이 없어 측정 불가.
- 재구성에 필요한 **최소 span 미정의**: prompt + context + tool I/O + model ver + token + ts.
- 관측성 폭이 좁음 — traces 1축뿐. metrics(처리량·에러율·비용)·alerting·**MTTD** 누락.

**무엇을 바꿨나 (반영)**
- **정의**: agent loop 전구간 span 캡처 + traces·metrics·alerting 3축 + 다수 QA의 측정 인프라 토대임을 명시. altitude로 QA-05/QA-13·QA-03/QA-06·QA-02와의 공급 관계 분리.
- **KPI 환원·확장**: 旧 `재구성 ≥95%` → ① `trace 완전성(안전 100%/일반 ≥95%, span 구성요소 명시)` ② `event-history 완전성 100%` ③ `MTTD ≤5분` ④ `결정당 비용/토큰 기록 100%`.
- 짝 시나리오 `QAS-04`의 Response·Measure를 span 완전성·event-history·MTTD로 동기화.

**남은 일 (이 라운드에서 미반영)**
- **DP-0003이 span 수준 trace를 보장하는지 역검토** — 3안 모니터링이 "규칙 기반 한정"일 때 어디까지 span을 남기는지 명시 필요 (`open-issues.md` 트래킹 대상).
- "재구성됨" 판정 자동화 한계·CoT 비공개는 검증 전략의 silent cap으로만 명시.
- MTTD `5분`·일반 trace `95%`는 **예시값**이며 실환경 측정으로 확정.

### 2026-06-24 — 등급 척도(★ rubric) 캘리브레이션
출처: discussion/qa/round-03 (등급 척도 캘리브레이션 — Council 3 seats, 팀 승인). 용도 = ATAM trade-off에서 동일 조건 설계 대안 비교(★ 많은 안 채택).

**무엇을 했나**
- **main 급간 축**: 헤드라인 2-index(안전 100% / 일반 ≥95%) 중 **일반 액션 trace 완전성 %**를 main 축으로(규칙4) — **정방향**(높을수록 ★ 높음). 안전관련 액션 100%는 표 밖 `조건:`으로 고정.
- **★ 급간**: ★★★ ≥98%(100% 미만 캡) / ★★☆ 95% 초과~98% 미만 / ★☆☆ ≥95%. 근거 = OTel GenAI semconv 자동계측 full span tree·auto-instrumentation 커버리지.
- **게이트(규칙5, 100% 절대형)**: 안전관련 액션 span 완전성 100% / event-history 완전성 100%(durable 결정론적 replay). 별점 축 아님.
- **§측정 보정**: 일반 액션 합격 하한 `≥95%`는 유지(재배치 불요). 단 **★★★ 상한을 100% 미만(≥98%)으로 캡** — LLM CoT 비공개·비결정 구간 prompt/seed 근사 한계로 100% 단정은 과대주장. 캡 사실을 §측정 해당 KPI 줄 아래 1줄 반영.
- silent cap: "재구성됨" 판정 자동화 한계(감사자 개입 여지) / CoT 비공개 — 완전성 %는 "span 존재율"이지 "의미 재현율" 100% 보장 아님.

### 2026-06-25 — round-04 디스커션 반영 (★ 등급 척도 근거 보강)
출처: [`discussion/qa/round-04`](../../discussion/qa/round-04/counsel/QA-04-observability.md) (red verdict: **Sound ◎ / KPI ○ — Low**; stance: 채택 권장 — 경계 표기 명확화·완전성 정의 강화).

**무엇이 문제였나 (review 지적)**
- ★★☆ (95,98) 3%p 좁음 + ★☆☆ ≥95% vs ★★☆ >95% 경계 모호.
- "완전성 %"가 span 존재율이지 충실도(truncation) 미반영.

**무엇을 바꿨나 (반영)**
- **경계 구간화 `[95,97)/[97,98)/[98,100)`** (≥/> 모호 제거, QA-07과 동형). 하한 95%·천장 캡 <100% 불변.
- **완전성 = span 존재 AND 비-truncation** 정의 강화(§측정·검증 전략) — 잘린 span 과대 카운트 방지.

**남은 일 (이 라운드에서 미반영)**
- DP-0003이 span 수준 trace 보장하는지 역검토(OI-7).
- MTTD 5분·일반 trace 95% 예시값 — 실환경 측정으로 확정.

> 출처: [discussion/qa/round-04](../../discussion/qa/round-04/counsel/QA-04-observability.md) (verdict: Sound ◎ / KPI ○ — Low, 채택 권장).
