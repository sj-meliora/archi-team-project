# QA 리뷰 — 마스터 인덱스

> `context/qa/`의 QA·KPI를 **반복 검토(라운드)**하며 완성도를 끌어올리는 작업의 색인.
> updated: 2026-06-23

## 관리 체계

- **리뷰 방법론(재생산 스펙)**: [`Reviewer.md`](Reviewer.md) — 3렌즈·rubric·포맷·절차의 단일 지침. reviewer agent는 이걸 따른다.
- **1 리뷰 = 1 라운드 = QA 10개 전체 스냅샷 판정.** (개별 QA만 고치는 patch 리뷰도 라운드로 친다.)
- **3 렌즈**: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트.
- **라운드 폴더**: `round-NN-YYYY-MMDD/`
  - `report.md` — **그 라운드 결론 요약(이것만 읽으면 됨)**: 판정표·교차발견·우선순위·신규 QA
  - `README.md` — 폴더 내비게이션(링크·읽기순서)
  - `QA-0X-*.md` — QA별 3렌즈 상세
  - `_new-qa-candidates.md` — 누락 QA 신설 권고(NQA-*)
- **append-only**: 지난 라운드는 수정하지 않고 보존한다(추세 비교용). 오타·링크 깨짐만 예외.
- **순환**: 리뷰(라운드 N) → Stage 2에서 `context/qa/` 개선 → 다음 라운드(N+1)를 새로 떠서 개선 검증 → 반복.
- **추적성**: 각 라운드 권고가 `context/qa/`에 반영되면 해당 QA 파일 `updated:`와 `changelog.md` 델타로 남기고, 다음 라운드 report에서 verdict 변화를 기록.

## 라운드 이력

| 라운드 | 날짜 | 대상 | High | Med | Low | 한 줄 요약 |
|---|---|---|:---:|:---:|:---:|---|
| [round-01](round-01-2026-06-23/report.md) | 2026-06-23 | QA-01~10 | 4 | 5 | 1 | 최초 리뷰. KPI 측정불가(placeholder·“최대화”)·정의↔KPI 불일치 다수, agentic 고유 리스크 과소대표 |

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
