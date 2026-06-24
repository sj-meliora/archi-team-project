# QAS-09 Agent 수행 시간 시나리오 (per-node)

> category: QAS | refines: QA-09 (Performance-Agent) | updated: 2026-06-24

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | Workflow 노드 |
| **자극 (Stimulus)** | 노드 작업 수행 요청 (고정 대상 노드 집합) |
| **대상 (Artifact)** | Agent (자동 수행) + runner |
| **환경 (Environment)** | 정상 운영 |
| **응답 (Response)** | 1-pass 성공 시 동일 노드 수동 baseline 대비 노드타입별로 시간 단축 (first-pass 실패분은 speedup 집계 제외) |
| **응답 측정 (Measure)** | 노드타입별 speedup **≥3배**(first-pass 성공분만) · first-pass 성공률(품질 게이트) · 커버리지 **≥80%** · runner 오버헤드 비율 **≤15%** |

## 비고
- 설계 연결: DP-0001(1안 즉시 실행), DP-0004(SP-3 cold-start → 오버헤드 비율).
- 수치(3배·80%·15%)는 예시값 — 합격선은 노드타입별 baseline 측정으로 확정. 상세는 QA-09 본문.
- 교차 의존: first-pass 성공 판정 = QA-07(Correctness) golden 게이트. throughput·E2E는 QA-01·QA-10로 위임.
