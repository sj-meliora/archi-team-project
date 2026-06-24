# QAS-08 Workflow 독립성 시나리오

> category: QAS | refines: QA-08 (Reliability-Workflow) | updated: 2026-06-24

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 특정 Workflow |
| **자극 (Stimulus)** | 해당 Workflow에서 장애 발생 또는 **자원 폭주**(무한 재시도·대량 토큰 소비) |
| **대상 (Artifact)** | 동시 실행 중인 타 Workflow + 공유 자원(LLM rate-limit 풀·공유 캐시) |
| **환경 (Environment)** | 다수 Workflow 동시 실행 |
| **응답 (Response)** | 장애·폭주가 격리되어 타 Workflow의 실행·지연·쿼터·공유 상태에 무영향(blast-radius 봉쇄) |
| **응답 측정 (Measure)** | 타 WF 실행 중단 **≤1%**, latency 증가 **≤10%**, 단일 WF의 토큰/rate-limit 쿼터 침범 **0건**, 공유 캐시 오염 전파 **0건** |

## 비고
- 설계 연결: DP-0004(A5/A8 invocation·단계 bulkhead), DP-0005(1안 로컬 캐시=오염 격리 / 2안 공유 캐시=무효화·읽기전용 계층화 필요).
- 수치(1%·10%·0건)는 합격선 — ①②는 기존 건강 KPI 유지, ③④ 임계는 실환경 측정으로 확정. 상세는 QA-08 본문.
- 경계: QA-02(복구)↔QA-08(격리) 분리. 쿼터 격리는 QA-01(rate-limit 헤드룸)과 공유 인프라.
