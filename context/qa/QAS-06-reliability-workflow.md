# QAS-06 Workflow 독립성 시나리오

> category: QAS | refines: QA-06 (Reliability-Workflow) | updated: 2026-06-20

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 특정 Workflow |
| **자극 (Stimulus)** | 해당 Workflow에서 장애 발생 |
| **대상 (Artifact)** | 동시 실행 중인 타 Workflow |
| **환경 (Environment)** | 다수 Workflow 동시 실행 |
| **응답 (Response)** | 장애가 격리되어 타 Workflow에 무영향 |
| **응답 측정 (Measure)** | 타 Workflow 실행 중단 ≤ 1%, latency 증가 ≤ 10% |

## 비고
- 설계 연결: DP-0004(2안 노드 격리), DP-0005(1안 로컬 캐시).
