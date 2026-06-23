# QAS-01 확장성 시나리오

> category: QAS | refines: QA-01 (Scalability) | updated: 2026-06-20

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 운영 환경(모델 유입) |
| **자극 (Stimulus)** | 모델·워크플로우 수 급증 |
| **대상 (Artifact)** | Agent Pool / Workflow 실행 인프라 |
| **환경 (Environment)** | 피크 부하 |
| **응답 (Response)** | 자원을 비례 확장하여 처리량 유지 |
| **응답 측정 (Measure)** | 시간당 완료 모델 수 ≥ N, 자원 활용률 ≥ 70% |

## 비고
- 설계 연결: DP-0001(2안 동적 풀), DP-0004(1안 타입별 scale out).
