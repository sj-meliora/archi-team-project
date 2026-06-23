# QAS-03 Agent 제어 시나리오

> category: QAS | refines: QA-03 (Controllability) | source: pptx p.28 | updated: 2026-06-20

## 6-Part Quality Attribute Scenario
| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 운영자(플랫폼 개발자) 또는 정책 엔진 |
| **자극 (Stimulus)** | 중단(stop) 명령 발행, 또는 Agent가 허용 범위 밖 액션 시도 |
| **대상 (Artifact)** | Agent / Permission Manager |
| **환경 (Environment)** | Agent 실행 중 |
| **응답 (Response)** | Agent가 동작을 즉시 중단 / 허용 범위 외 액션을 차단 |
| **응답 측정 (Measure)** | 중단 명령 수용 시간 **≤ 5초**; 허용 범위 외 액션 통과 0건 |

## 비고
- 설계 연결: DP-0002(단일 제어 지점에서 정책·중단 적용), DP-0003 1안(사전 권한 체크 게이트).
