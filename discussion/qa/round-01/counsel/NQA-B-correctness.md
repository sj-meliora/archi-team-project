# Counsel: NQA-B Correctness / Accuracy (신규 QA 권고)

> refs-review: round-01/review/_new-qa-candidates.md (NQA-B) · report.md(C5)
> seats: 발의 Seat 1(Agentic Workflow) · 합의 consensus (Seat 2 일관성과의 분리, Seat 3 eval 데이터 수급)
> stance: 신설 — 채택 권장 (자율 신뢰의 #1 기둥; QA-07/09 KPI의 닫힘 조건)

## Reviewer 지적 요약
- 신뢰의 #1 차단요인 = "에이전트 산출물이 **맞는가**". QA-09(일관성)는 "**같은** 결정"만 보고 "**맞는** 결정"은 어떤 QA도 측정 안 함 — 일관되게 틀릴 수 있음.
- 잘못된 quantize config·빌드 결정 → rework 폭증 → QA-07/08 시간 이득 소멸.

## 개선안 (정의·KPI)

**정의 (신설)**: 에이전트의 자동 산출물(quantize config, 빌드/검증 결정, 이슈 처리)이 정답·정책에 부합한다.

**KPI**
| # | 제안 | 비고 |
|---|---|---|
| ① | **golden-set 대비 산출물 정답률 ≥ ◯%** (단계별 IR/Optimize/Quant/Compile) | PoC-N-B1 |
| ② | **자동 결정 rework율 ≤ ◯%** (사람 재작업 유발 비율) | QA-07 first-pass 게이트와 공유 |
| ③ | **회귀(regression) 미검출률 ≤ ◯%** (정확성 게이트 누수) | |

## 근거 (레퍼런스)
- **golden dataset(앵커 50~100) + LLM-as-judge(다수결, 인간 일치 75~90%)**: 정확성 측정의 필드 표준. 100% 통과면 난이도 부족 신호 — golden-dataset·LLM-as-judge 베스트프랙티스. (§4 정확성) — https://www.comet.com/site/blog/llm-as-a-judge/ , https://www.getmaxim.ai/articles/building-a-golden-dataset-for-ai-evaluation-a-step-by-step-guide/ , https://montecarlo.ai/blog-llm-as-judge/
- ⚠️ 정답률·rework ◯%는 우리 golden set으로 확정 — 레퍼런스 수치 복제 아님.

## PoC 증명법

### PoC-N-B1: golden set + LLM-as-judge로 정확성을 측정한다 (eval 하네스)
- **가설**: "단계별 golden set과 (인간 일치 검증된) LLM-as-judge로 정답률·rework율이 측정 가능하다."
- **지표**: 단계별 정답률, judge의 인간 일치도(75~90% 목표), rework율, 회귀 미검출률. 100% 통과 시 난이도 부족 신호로 간주.
- **셋업**: 단계별 golden set(IR/Optimize/Quant/Compile 각 앵커 50~100, 사람 라벨) + LLM-as-judge(다수결) + 인간 일치도 검증 샘플.
- **절차**: ① golden set 구축·사람 라벨 → ② judge로 자동 채점 후 **judge↔인간 일치도 먼저 검증** → ③ agent 산출물 정답률·rework·회귀 측정.
- **합격(Exit)**: judge 인간 일치도 ≥ 목표(judge 신뢰 확보), 정답률 측정 가능·100% 아님(적정 난이도).
- **규모/기간**: 단계 4종 × 앵커 50~100, judge 검증 포함 약 4~5일.
- **리스크/한계(silent cap)**: golden set 커버리지 = 정확성 신뢰 상한(미포함 케이스 미검출). judge 자체 편향(judge가 틀리면 정답률 왜곡) → 인간 일치도 검증이 전제. quantize "정답"의 정의(정확도/지연 trade-off) 합의가 선결 — log.

## DP·발표 영향
- **DP 연결**: eval 하네스(C4)가 신규 인프라 — DP에 명시 안 됨 → eval/검증 서브시스템 신설 검토(QA-03/NQA-A red-team 하네스와 공유 가능).
- **교차 의존(핵심)**: NQA-B는 **QA-07(first-pass 성공 판정)·QA-09(유효-결정률)의 KPI를 닫는 전제** — 정확성 게이트 없이는 두 QA가 측정 불가. PoC 의존 그래프에서 NQA-B 선행(_poc-plan.md).
- **번호/서사(C5)**: NQA-A(Security)와 함께 "믿고 맡길 수 있는가" 두 기둥 → 우선순위 상위. QA-09와 묶어 "일관 + 정확" 서사.
