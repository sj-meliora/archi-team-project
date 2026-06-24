# QAS-09 Agent 결과 일관성 시나리오

> category: QAS | refines: QA-09 (Reliability-Agent) | updated: 2026-06-24

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 동일/유사 입력 반복 |
| **자극 (Stimulus)** | 동일 작업 재요청 (캐시 우회 k회 반복) / 의미적으로 유사한 입력 |
| **대상 (Artifact)** | Agent (순수 추론, memoization 우회) |
| **환경 (Environment)** | 동일 조건 (model/prompt/tool 버전 핀닝) |
| **응답 (Response)** | 허용 오차 내 유효(정책 허용) 결정을 안정적으로 산출 |
| **응답 측정 (Measure)** | 순수 추론 재현율(캐시 제외) **≥70% (pass^k, k=5)**, 유효-결정률 **≥95%**, 재실행 분산 **≤0.2** |

## 비고
- 설계 연결: DP-0005(2안 공유 캐시 = memoization — **측정 시 캐시 우회 모드 필수**, 캐시는 비용 지표이지 일관성 입증 아님).
- 수치(70%·95%·0.2)는 예시값 — 합격선은 캐시 우회 반복시행으로 확정. 상세는 QA-09 본문.
- 교차 의존: 유효-결정률 = NQA-B(Correctness) golden 게이트. 일관(QA-09) + 정확(NQA-B)이 짝이어야 자율 신뢰가 닫힘.
