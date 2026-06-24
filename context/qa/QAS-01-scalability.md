# QAS-01 확장성 시나리오

> category: QAS | refines: QA-01 (Scalability) | updated: 2026-06-20

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 운영 환경(모델 유입) |
| **자극 (Stimulus)** | 모델·워크플로우 수 급증 |
| **대상 (Artifact)** | Agent Pool / Workflow 실행 인프라 |
| **환경 (Environment)** | 피크 부하 |
| **응답 (Response)** | **backlog(큐 깊이) 신호**로 자원을 비례 투입해 처리량을 선형에 가깝게 유지 (1차 병목=LLM rate-limit) |
| **응답 측정 (Measure)** | scaling efficiency ≥ 0.8 (부하 2배→처리량 ≥1.8배), rate-limit 헤드룸 ≥ 20%(throttle 0), 큐 대기 p95 ≤ 5분 |

## 비고
- 설계 연결: DP-0001(2안 동적 풀), DP-0004(1안 타입별 scale out).
