# QA 심의(discussion) — 마스터 인덱스

> `context/qa/`의 QA·KPI를 **반복 심의(라운드)**하며 완성도를 끌어올리는 작업의 색인.
> 한 라운드 = red team 비평(review) + blue team 권고(counsel)의 한 묶음.
> 공통 프로토콜·다른 영역(DP 등)은 상위 [`../README.md`](../README.md). 트리거: "QA 디스커션 돌리자".
> updated: 2026-06-23

## 관리 체계

- **두 방법론(재생산 스펙)** — 라운드 무관, 이 디렉터리 최상위:
  - [`Reviewer.md`](Reviewer.md) — **red team(비평)**. 3렌즈로 QA·KPI의 약점을 판정.
  - [`Council.md`](Council.md) — **blue team(권고)**. Reviewer 지적을 읽고 근거(레퍼런스)+PoC를 붙여 개선을 권고.
- **1 라운드 = QA 전체 스냅샷에 대한 한 번의 심의**(비평 + 권고).
- **3 렌즈 / 3 seats**: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트.
- **라운드 폴더**: `round-NN/` — **폴더명에 날짜를 넣지 않는다.** 날짜 등 메타는 그 라운드의 종합 보고서에 기록(공의회가 최종 결론을 문서화하듯).
  - `review/` — red team 산출물: `report.md`(결론)·`README.md`(내비)·`QA-0X-*.md`(3렌즈 상세)·`_new-qa-candidates.md`
  - `counsel/` — blue team 산출물: `counsel.md`(권고 결론)·`QA-0X-*.md`(개선안)·PoC 계획 등
- **append-only**: 지난 라운드는 수정하지 않고 보존한다(추세 비교용). 오타·링크 깨짐만 예외.
- **순환**: 심의(라운드 N) → Stage 2에서 `context/qa/` 개선 → 다음 라운드(N+1)를 새로 떠서 개선 검증 → 반복.
- **추적성**: 각 라운드 권고가 `context/qa/`에 반영되면 해당 QA 파일 `updated:`와 `changelog.md` 델타로 남기고, 다음 라운드 종합 보고서에서 verdict 변화를 기록.

## 라운드 이력

| 라운드 | 날짜 | 대상 | High | Med | Low | 한 줄 요약 |
|---|---|---|:---:|:---:|:---:|---|
| [round-01](round-01/review/report.md) | 2026-06-23 | QA-01~10 | 4 | 5 | 1 | red team 최초 리뷰. KPI 측정불가(placeholder·“최대화”)·정의↔KPI 불일치 다수, agentic 고유 리스크 과소대표. (blue team counsel 예정) |

## QA별 verdict 추세

표기: ◎ 우수 · ○ 타당 · △ 부분결함 · ✕ 재설계 (형식: `Sound / KPI`)

| QA | 속성 | round-01 |
|---|---|---|
| QA-01 | Scalability | △ / ✕ |
| QA-02 | Availability | ○ / △ |
| QA-03 | Controllability | ◎ / △ |
| QA-04 | Observability | ○ / △ |
| QA-05 | Efficiency | ○ / △ |
| QA-06 | Reliability (WF 격리) | ○ / ○ |
| QA-07 | Performance (Agent 시간) | △ / ✕ |
| QA-08 | Performance (E2E) | △ / ✕ |
| QA-09 | Reliability (일관성) | △ / △ |
| QA-10 | Maintainability | ○ / ○ |

## 신규 QA 권고 추세

| 후보 | 제안 라운드 | 상태 |
|---|---|---|
| NQA-A Security/Safety | round-01 | 권고 (미채택) |
| NQA-B Correctness/Accuracy | round-01 | 권고 (미채택) |
| NQA-C Cost-economy | round-01 | 권고 (미채택) |
