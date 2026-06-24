# QAS-07 산출물 정확성 시나리오

> category: QAS | refines: QA-07 (Correctness) | source: discussion/qa/round-01 (신규) | updated: 2026-06-24
> ISO/IEC 25010:2023: Functional Suitability / Functional Correctness

## 6-Part Quality Attribute Scenario
| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | Workflow 노드 / 정확성 검증 게이트 |
| **자극 (Stimulus)** | Agent가 자동 산출물(quantize config·빌드/검증 결정·이슈 처리)을 생성 |
| **대상 (Artifact)** | Agent 산출물 / golden 평가 게이트(eval 서브시스템) |
| **환경 (Environment)** | 정상 운영 (정확성 게이트 통과가 후속 단계 전제) |
| **응답 (Response)** | golden-set·정책 대비 정답 여부를 판정, 오답은 rework/회귀로 분류해 차단 |
| **응답 측정 (Measure)** | golden-set 대비 정답률 **≥90%**(단계별 IR/Optimize/Quant/Compile), 자동 결정 rework율 **≤10%**, 회귀 미검출률 **≤5%** |

## 비고
- ISO/IEC 25010:2023 Functional Suitability / Functional Correctness(기능 정확성). 일관성(QA-11 Reliability)과 직교 — 정확성 ≠ 일관성, 둘은 짝.
- 설계 연결: eval/검증 서브시스템(신규 — 현재 DP 없음, DP 후보). 검증은 golden + LLM-as-judge 하네스([발표 서사]).
- 교차 의존: QA-09(first-pass 성공 판정)·QA-11(유효-결정률)의 닫힘 조건. 데이터는 QA-04 trace 수급.
- 수치(90%·10%·5%)는 측정가능 KPI 예시값 — golden set으로 확정(100% 통과면 난이도 부족 신호).
