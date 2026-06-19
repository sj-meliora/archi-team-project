# QAS-0003 운영 안정성 시나리오

> category: QAS | refines: QA-0003 (Availability) | updated: 2026-06-20

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 인프라 / 노드 |
| **자극 (Stimulus)** | 부분 장애 발생 (노드·컴포넌트 다운) |
| **대상 (Artifact)** | Workflow 실행 인프라 |
| **환경 (Environment)** | 정상 운영 중 |
| **응답 (Response)** | 서비스 연속성 유지 + 자동 복구 |
| **응답 측정 (Measure)** | MTTR < 1분 |

## 비고
- 설계 연결: DP-0001(1안 격리), DP-0002(3안 Standby), DP-0003(격리/모니터링).
