# QAS-0001 토큰 예산 시나리오

> category: QAS | refines: QA-0001 (Efficiency) | source: pptx p.25 | updated: 2026-06-20

## 6-Part Quality Attribute Scenario
| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | Workflow 노드가 Agent에 작업 요청 |
| **자극 (Stimulus)** | 단일 노드 작업 판단 요청 (예: 빌드 결과 검증 후 다음 행동 결정) |
| **대상 (Artifact)** | Agent (LLM Endpoint) |
| **환경 (Environment)** | 정상 운영 중 일반 Workflow 실행 |
| **응답 (Response)** | 토큰 예산 내에서 작업을 완료 |
| **응답 측정 (Measure)** | 단일 요청 총 토큰 **≤ 8k** (★★★ ≤4k / ★★☆ 4k~6k / ★☆☆ 6k~8k) |

## 비고
- 측정 척도 상세는 QA-0001 참고.
- 설계 연결: DP-0001(작업별 최적 Agent 선택으로 토큰 절감), DP-0003(권한 게이트는 토큰 미소모).
