# QA 리뷰 round-01 (2026-06-23) — 내비게이션

> 이 폴더는 `context/qa/` QA 10 + QAS 10에 대한 round-01 리뷰다.
> **결론 요약은 [`report.md`](report.md) 하나만 읽으면 된다.** 리뷰 방법론은 [`../Reviewer.md`](../Reviewer.md).

## 읽는 순서

1. **[`report.md`](report.md)** — 라운드 결론(판정표·교차발견 C1~C5·우선순위·신규 QA). 이것만 읽어도 전모 파악.
2. 개별 QA 상세 — 3렌즈 분석 + Stage 2 권고:

| QA | 속성 | verdict (Sound/KPI) | severity |
|---|---|---|---|
| [QA-01](QA-01-scalability.md) | Scalability | △ / ✕ | High |
| [QA-02](QA-02-availability.md) | Availability | ○ / △ | High |
| [QA-03](QA-03-controllability.md) | Controllability | ◎ / △ | Med |
| [QA-04](QA-04-observability.md) | Observability | ○ / △ | Med |
| [QA-05](QA-05-efficiency.md) | Efficiency | ○ / △ | Med |
| [QA-06](QA-06-reliability-workflow.md) | Reliability (WF 격리) | ○ / ○ | Med |
| [QA-07](QA-07-performance-agent-time.md) | Performance (Agent 시간) | △ / ✕ | High |
| [QA-08](QA-08-performance-e2e.md) | Performance (E2E) | △ / ✕ | High |
| [QA-09](QA-09-reliability-agent-consistency.md) | Reliability (일관성) | △ / △ | Med |
| [QA-10](QA-10-maintainability.md) | Maintainability | ○ / ○ | Low |

3. **[`_new-qa-candidates.md`](_new-qa-candidates.md)** — 누락된 1급 QA 권고(NQA-A Security / B Correctness / C Cost).

## 파일 구성

| 파일 | 역할 |
|---|---|
| `report.md` | 결론 리포트 (standalone) |
| `README.md` | 이 내비게이션 |
| `QA-01`~`QA-10` | QA별 3렌즈 상세 |
| `_new-qa-candidates.md` | 신규 QA 권고 |
