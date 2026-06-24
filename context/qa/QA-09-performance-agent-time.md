---
id: QA-09
category: QA
importance: H
difficulty: H
source: pptx p.13
related-dp: [DP-0001, DP-0004]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "'최대화'(테스트 불가) 폐기 → 노드타입별 speedup·first-pass 품질 게이트·커버리지로 재설계 (자세히 → ## 변경 이력)"
  - date: 2026-06-24
    by: 팀 결정 (OI-8)
    reason: "재번호 QA-07 → QA-09 (NQA 정식 편입에 따른 +2 시프트, → changelog)"
---

# QA-09 Performance — Agent 수행 시간 (per-node)

## 정의 / Refinement
**1-pass(재작업 없이 한 번에) 성공한** 노드에 한해, 자동화 노드의 수행시간을 **동일 노드의 수동(개발자) baseline 대비 일정 비율 이하로 단축**한다. 측정은 **고정된 대상 노드 집합**에서 **노드타입별**로 한다("최대화" 같은 목표가 아니라 통과/실패를 가르는 임계값으로).

설계할 때 잡아야 할 두 가지 관점:

- **속도는 품질 게이트와 짝이어야 한다** — 시간만 재면 "**빠르게 틀리기**"가 최적해로 보인다. 틀린 quantize config를 빨리 내서 rework를 유발하는 에이전트는 net-negative다. 따라서 시간 지표는 반드시 **first-pass 성공한 작업에만** 집계한다(성공 판정 = 정확성 게이트, **QA-07(Correctness)** 연계).
- **절대 시간차가 아니라 비율, 그리고 gaming 방지** — 이질적 노드(Compile vs IR 변환)의 절대 초를 빼는 건 무의미하고, 쉬운 노드만 자동화해도 "차이"는 커진다(gaming). **노드타입별 speedup 비율** + **커버리지 조건**(측정 대상 노드 집합 고정)으로 막는다.

> 이 QA는 "**노드 1건의 수행 속도(per-node)**"만 다룬다 — 단위 시간당 처리량(throughput)은 QA-01, 모델 1건의 E2E 소요시간은 QA-10에서 따로 본다(Performance 3분할: QA-01 throughput / QA-09 per-node / QA-10 E2E). 단일노드 head-to-head는 시스템의 진짜 가치(병렬성·24/7 자율성)를 과소평가하므로 throughput·E2E로 위임.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `노드타입별 speedup(first-pass 성공분)` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **노드타입별 speedup ≥ 3배 (first-pass 성공 작업에만 집계)** `[주 KPI · PoC 대상]` — = 수동 baseline 중앙값 / agent 중앙값
  - 쉽게: 각 노드 종류(IR 변환·최적화·양자화·컴파일)마다, 사람이 직접 할 때보다 3배 이상 빨라야 한다. 단 **한 번에 제대로 성공한 작업만** 센다(틀리고 다시 한 건 제외). *speedup = 속도 배수, 중앙값 = 한가운데 값(드문 극단치에 안 휘둘림).*
- **품질 게이트(paired) = first-pass 성공률** — 위 speedup의 분모 조건. 성공 판정은 정확성 게이트(QA-07 golden)
  - 쉽게: "빠르게 틀리기"를 막는 짝 지표. 시간이 빨라도 first-pass 성공률이 낮으면 의미 없다. *paired metric = 한 지표가 나빠지는 걸 감시하는 짝 지표(가드레일).*
- **커버리지 ≥ 80%** — 측정 대상 노드 집합 고정(easy-node cherry-picking 방지)
  - 쉽게: 전체 노드 종류 중 80% 이상을 측정 대상에 넣어야 한다. 쉬운 노드만 골라 재면 안 된다.
- **runner 오버헤드 비율 ≤ 15%** — (큐 대기 + dispatch + cold start) / 노드 총시간
  - 쉽게: 노드 총시간 중 실제 모델 연산이 아닌 대기·기동 같은 군더더기(runner 오버헤드)가 15%를 넘으면 안 된다. 개선 레버를 드러내는 SLI. *cold start = 컨테이너 첫 기동 지연.*

> 위 수치(3배·80%·15%)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 **노드타입별 baseline 측정**으로 확정한다.
> 폐기: 旧 `{개발자 수행시간 − Agent 수행시간} 최대화` — "최대화"는 optimization goal이지 acceptance criterion이 아님(합격선 없음→테스트 불가) + 이질 노드 절대시간 차 무의미 + 품질 게이트·커버리지 부재(gaming 취약). → 비율·게이트·커버리지로 교정.

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 SLO·실험 설계의 표준에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **임계값 기반 acceptance ("최대화" 금지)** | SLO/SLI는 통과/실패를 가르는 구체 임계값으로 정의 — "최대화"는 측정 불가 | [Google SRE Workbook — SLO 구현](https://sre.google/workbook/implementing-slos/) |
| **paired/guardrail metric (속도-품질)** | 단일 지표 최적화의 gaming을 막기 위해 핵심지표에 guardrail을 짝지움(실험·eval 표준). 시간엔 정확성 게이트를 paired로 — golden + LLM-as-judge로 성공 판정 | [LLM-as-a-judge 해설](https://www.comet.com/site/blog/llm-as-a-judge/) |
| **runner 오버헤드 span 분해 / percentile** | OTel GenAI span으로 큐·tool·agent 연산을 분리 계측, 평균 대신 p95 | [OTel — GenAI agent spans](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/) · [p50/p95/p99 해설](https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view) |

> ⚠️ 레퍼런스의 수치(speedup·오버헤드 %)는 **패턴 정당화용**이며 그대로 복제하지 않는다. 우리 합격선은 위 [검증 전략](#검증-전략)의 baseline 측정으로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 baseline 비교·계측**으로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **노드타입별 speedup (first-pass)** `[주]` | **DP-0001 1안 즉시 실행**([Performance] 선택 없이 즉시 실행 ★★★) — 단, **latency만 보고 품질 게이트와 무연결** → first-pass 게이트 연계 보강 필요 | **▶ 실제 제작:** speedup×품질게이트 모델 — 고정 노드 집합(IR/Optimize/Quant/Compile 각 N건) + 수동 baseline 시간표 + 성공판정용 mini golden(QA-07) → agent 수행 후 **first-pass 성공분만으로 speedup·커버리지** 산출, easy-node만 골랐을 때 커버리지 하락(gaming) 탐지 |
| 품질 게이트 = first-pass 성공률 | **QA-07(Correctness) golden 게이트**(교차 의존 — QA-07와 함께 가야 KPI가 닫힘) | 보조 모델: 위 모델의 성공/실패 라벨링이 곧 게이트 — golden mini set으로 판정 |
| 커버리지 ≥ 80% | 고정 노드 집합 정의(easy-node cherry-picking 방지) | 보조 모델: 측정 노드 / 전체 노드타입 비율 집계 |
| runner 오버헤드 비율 ≤ 15% | **DP-0004 SP-3 cold-start**(이미지 크기·pre-warm/min-instance가 오버헤드에 직결) + warm pool | 보조 모델: OTel span으로 (큐+콜드스타트)/총시간 분해, warm pool On/Off A/B — PoC-P2가 DP-0004 A5 vs A8 콜드스타트 budget에도 데이터 공급 |

> 가정·한계: 수동 baseline은 **표본·숙련도 편차가 큼**(공정 baseline 확보가 최대 난점) → baseline 출처·측정조건을 명시. golden 게이트가 작으면 first-pass 성공 판정이 낙관 편향될 수 있음. tool 실행 시간(외부 컴파일러)은 우리 최적화 대상 밖(분리는 되나 단축 레버 아님) — silent cap.

## 변경 이력

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-07-performance-agent-time.md) (red team verdict: **Sound △ / KPI ✕ — High** — "최대화"는 테스트 불가, altitude 의심)

**무엇이 문제였나 (review 지적)**
- KPI `{사람 − 에이전트} 최대화`는 **optimization goal이지 acceptance criterion이 아님** → 합격선 없음, 테스트 불가.
- **품질 게이트 부재** — 시간만 재면 "빠르게 틀리기"가 최적해(rework 유발 net-negative).
- **gaming 취약** — 쉬운 노드만 자동화해도 차이가 커짐. 이질 노드 절대시간 차는 무의미 → 노드타입별 speedup 비율이 정상.
- runner 오버헤드(큐·콜드스타트)와 agent 연산을 분리 계측해야 개선 레버가 보임.

**무엇을 바꿨나 (반영)**
- **정의**: "1-pass 성공한 노드 한정, 고정 노드집합, 노드타입별 비율 단축"으로 재서술("최대화" 제거, 품질·커버리지 조건 명시). Performance 3분할(QA-01/09/10) altitude 명시.
- **KPI 재설계**: 旧 `{사람−에이전트} 최대화` → ① `노드타입별 speedup ≥3배(first-pass 성공분)` ② `품질 게이트=first-pass 성공률`(paired) ③ `커버리지 ≥80%` ④ `runner 오버헤드 비율 ≤15%`.
- `related-dp`에 **DP-0004 추가**(cold-start SP-3가 ④ 오버헤드에 직결).
- 짝 시나리오 `QAS-09`의 Response·Measure를 speedup·품질게이트·커버리지로 동기화.

**남은 일 (이 라운드에서 미반영)**
- **first-pass 성공 판정은 QA-07(Correctness) golden 게이트에 의존** → **QA-07 채택과 함께 가야 QA-09 KPI가 닫힌다**(교차 의존). golden set **구축**은 [생략]/[발표 서사](실측 미실행) — 슬라이드 한 줄("speedup을 first-pass 성공분에만 집계해 '빠르게 틀리기' 차단").
- **DP-0001(1안 즉시 실행)이 latency만 보고 품질 게이트와 무연결** → 품질 게이트 연계 보강 역검토 (`open-issues.md` 트래킹 대상).
- speedup `3배`·커버리지 `80%`·오버헤드 `15%`는 **예시값**이며 노드타입별 baseline 측정으로 확정.
