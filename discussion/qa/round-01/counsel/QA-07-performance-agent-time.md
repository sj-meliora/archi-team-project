# Counsel: QA-07 Performance — Agent 수행 시간

> refs-review: round-01/review/QA-07-performance-agent-time.md · report.md(C1·C3)
> seats: 발의 Seat 2(수석 아키텍트) · 합의 consensus (Seat 1 품질게이트·Seat 3 분해 보강)
> stance: 채택 권장 (KPI 측정가능화 + 품질 게이트 paired metric)

## Reviewer 지적 요약
- KPI `{사람 − 에이전트} 최대화` 는 **optimization goal이지 acceptance criterion이 아님** → 합격선 없음, 테스트 불가 (렌즈2, C1).
- **품질 게이트 부재** — 시간만 재면 "빠르게 틀리기"가 최적해로 보임(rework 유발 net-negative) (렌즈1).
- **gaming 취약** — 쉬운 노드만 자동화해도 차이가 커짐. 이질 노드 절대시간 차는 무의미 → **노드타입별 speedup 비율**이 정상 (렌즈2).
- runner 오버헤드(큐·콜드스타트)와 agent 연산을 **분리 계측**해야 개선 레버가 보임 (렌즈3).

## 개선안 (정의·KPI 기존→제안)

**정의**
- 기존: "개발자 직접 수행 대비 단축 최대화."
- 제안: "**1-pass(재작업 없이) 성공한** 노드에 한해, 자동화 노드 수행시간을 동일 노드 **수동 baseline 대비 일정 비율 이하로 단축**한다. 측정은 **고정된 대상 노드 집합**에서 노드타입별로 한다." — "최대화" 제거, 품질·커버리지 조건 명시.

**KPI**
| # | 기존 | 제안 | 비고 |
|---|---|---|---|
| ① | {사람−에이전트} 최대화 | **노드타입별 speedup ≥ ◯배** (또는 Agent 노드시간 ≤ 수동 baseline의 ◯%) | 절대차→비율. ◯는 노드타입별 baseline 측정으로 확정(PoC-P1) |
| ② | — | **품질 게이트(paired)**: ①은 **first-pass 성공 작업에만** 집계 | "빠르게 틀리기" 차단. 성공판정=정확성 게이트(NQA-B 연계) |
| ③ | — | **커버리지 ≥ ◯%** (측정 대상 노드 집합 고정) | easy-node cherry-picking 방지 |
| ④(분해) | — | **runner 오버헤드 비율 = (큐 대기+콜드스타트)/노드 총시간 ≤ ◯%** | 개선 레버 노출용 SLI. ◯는 PoC-P2 |

> per-node 시간은 sub-metric으로 두고, throughput·병렬성 차원은 **QA-01/QA-08로 위임**(C3 Performance 3분할).

## 근거 (레퍼런스)
- **임계값 기반 acceptance(평균·"최대화" 금지)**: SLO/SLI는 통과/실패를 가르는 구체 임계값으로 정의해야 한다 — Google SRE. (§4 지연·아키텍트 원칙) — https://sre.google/workbook/implementing-slos/
- **paired/guardrail metric (속도-품질 동시)**: 단일 지표 최적화의 gaming을 막기 위해 핵심지표에 **guardrail metric**을 짝지우는 것은 실험·eval 설계 표준. 시간 지표엔 정확성 게이트를 paired로. (§4 정확성과 연계) — golden-set + LLM-as-judge로 성공판정: https://www.comet.com/site/blog/llm-as-a-judge/
- **runner 오버헤드 분리 계측(span)**: OTel GenAI semconv의 `invoke_agent/execute_tool` span으로 큐 대기·tool·agent 연산을 분해 계측하는 것이 관측 표준. (§4 관측성) — https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/
- **percentile로 노드시간 SLI**: 평균 대신 p95. (§4 지연) — https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view
- ⚠️ speedup ◯배·오버헤드 ◯% 등은 우리 baseline 측정으로 확정 — 레퍼런스 수치 복제 금지.

## PoC 증명법

### PoC-P1: speedup가 품질 게이트와 함께 측정 가능함을 보인다
- **가설**: "노드타입별 speedup ≥ ◯배가, first-pass 성공 작업에 한해 측정 가능하고 gaming되지 않는다."
- **지표**: 노드타입별 speedup = 수동 baseline 중앙값 / agent 중앙값 (first-pass 성공분만). + 커버리지(측정 노드 / 전체 노드타입).
- **셋업**: 고정 노드 집합(IR/Optimize/Quant/Compile 각 N건) + 수동 baseline 시간표(사람 측정 또는 기존 로그) + 성공판정용 golden/검증 게이트(NQA-B mini eval).
- **절차**: ① 노드타입별 수동 baseline 측정 → ② agent 동일 작업 수행, first-pass 성공/실패 라벨 → ③ 성공분만으로 speedup·커버리지 산출. easy-node만 골랐을 때 커버리지가 떨어지는지 확인(gaming 탐지).
- **합격(Exit)**: 성공분 speedup이 전 노드타입에서 임계 이상 + 커버리지 ◯% 이상이면 "측정 가능·게임 불가" 증명.
- **규모/기간**: 노드타입 4종 × 각 10~20건(앵커 50~100), 약 2~3일.
- **리스크/한계(silent cap)**: 수동 baseline은 표본·숙련도 편차가 큼(공정 baseline 확보가 최대 난점) → baseline 출처/측정조건을 명시 log. golden 게이트가 작으면 first-pass 성공 판정이 낙관 편향될 수 있음.

### PoC-P2: runner 오버헤드 분리 계측이 개선 레버를 드러낸다
- **가설**: "노드 총시간 중 runner 오버헤드(큐+콜드스타트) 비율을 span으로 분리 계측할 수 있다."
- **지표**: 오버헤드 비율 = (큐 대기 + dispatch + cold start) / 노드 총시간. 합격선 ≤ ◯%.
- **셋업**: OTel GenAI span 계측(invoke_agent/execute_tool/queue) + warm pool On/Off A/B.
- **절차**: ① span 삽입 후 노드 1건 trace 완전성 확인 → ② cold vs warm 비교 → ③ 오버헤드 비율·절감폭 산출.
- **합격(Exit)**: 분해 trace 완전성 ~100%, warm pool로 오버헤드 비율 유의 감소 관측.
- **규모/기간**: 수십 trace, 약 1일.
- **리스크/한계**: tool 실행 시간(외부 컴파일러)은 우리 최적화 대상 밖 — 분리는 되나 단축 레버 아님(log).

## DP·발표 영향
- **DP 연결**: DP-0001(1안 즉시 실행)이 **latency만 보고 품질 게이트와 무연결** → 품질 게이트(first-pass 성공) 연계 보강 권고. DP-0004 A5/A8의 cold-start(SP-3)가 ④ 오버헤드 비율에 직결 → PoC-P2가 A5 vs A8 택일(콜드스타트 budget)에도 데이터 공급.
- **품질 게이트 = NQA-B(Correctness) 의존**: first-pass 성공 판정은 정확성 eval이 있어야 성립 → **NQA-B 채택과 함께 가야 QA-07 KPI가 닫힌다**(교차 의존 명시).
- **번호/서사**: report.md C3에 따라 **Performance를 QA-01(throughput)·QA-07(per-node)·QA-08(E2E) 3분할**로 재편 시 QA-07은 "per-node speedup" sub-metric으로 명확화. 발표: "빠르게 틀리기를 KPI 설계로 차단" → 자율 신뢰 서사 강화.
