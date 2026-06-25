# QAS-04 관측성 시나리오

> category: QAS | refines: QA-04 (Observability) | updated: 2026-06-25 (round-04 완전성=존재 AND 비-truncation·경계 구간화 — 하한 불변)

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 운영자 / PM / 이상탐지 룰 |
| **자극 (Stimulus)** | Agent 의사결정 사후 조회 요청 / 운영 중 이상(에러·이탈) 발생 |
| **대상 (Artifact)** | Monitoring/Logging (traces·metrics·alerting) |
| **환경 (Environment)** | 운영 중 / 사후 분석 |
| **응답 (Response)** | agent loop 전구간 span(prompt·context·tool I/O·model·token·ts)으로 한 결정을 재구성하여 제공 / event-history로 워크플로우 골격 재생 / 이상을 탐지·알림 |
| **응답 측정 (Measure)** | trace 완전성(= span 존재 AND 비-truncation) 안전 액션 **100%**·일반 **≥95%**(★ 급간 [95,97)/[97,98)/[98,100) — QA-04 등급 척도); event-history 완전성 **100%**; MTTD **≤5분**; 결정당 비용/토큰 기록 **100%** |

## 비고
- 설계 연결: DP-0003(3안 실시간 모니터링 — 규칙 기반 한정 적용 권고).
- 수치(100%·95%·5분)는 측정가능 KPI의 예시값 — 합격선은 실환경 측정으로 확정. 상세·근거·검증 전략은 QA-04 본문.
- 측정 인프라 공급: 결정당 비용→QA-05/QA-13, 안전 액션 추적→QA-03/QA-06, MTTD→QA-02(MTTR 전제).
