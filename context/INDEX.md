# context INDEX

> 17조 팀 과제 컨텍스트의 단일 진실 공급원(SSoT) 색인.
> 폴더는 알파벳순으로 정렬되므로, **방법론 읽기 순서**는 이 파일을 따른다.
> 원본 자료는 `../source/`의 최신 pptx, 변경 이력은 `changelog.md`.

## 읽기 순서 (ATAM 추적 사슬)

```
overview → stakeholders → requirements(FR/NFR/C) → qa(QA → QAS) → dp(설계결정 + ATAM) → artifacts(뷰/도표)
```

## ID 체계

| 카테고리 | 폴더 | ID prefix | 비고 |
|---|---|---|---|
| 개요 | (flat) overview.md | OV | 시스템 정의·배경·Pain Point |
| 이해관계자 | (flat) stakeholders.md | ST | |
| 기능 요구사항 | requirements/ | FR | |
| 비기능 요구사항 | requirements/ | NFR | (현재 없음, 예약) |
| 제약 | requirements/ | C | |
| 품질 속성 | qa/ | QA | canonical = 슬라이드 13 기준 |
| 품질 속성 시나리오 | qa/ | QAS | 6-part 정형화 |
| 설계 결정 | dp/ | DP | ATAM 분석을 문서 내부에 포함 |
| 산출물(뷰/도표) | artifacts/ | (자유) | context/domain diagram 등 |

- ID 포맷: `<CAT>-<NNNN>` (4자리 zero-pad). 파일명 = `<ID>-<kebab-slug>.md`.
- ID는 영구 고정 — 슬러그/내용만 변경. cross-link은 본문에 ID 텍스트로(`DP-0002`, `QA-0004`) 적어 grep 역참조 가능하게.

## 전체 ID 목록

### requirements/
- FR-0001 Workflow 실행 인프라
- FR-0002 Artifact 저장·전달
- FR-0003 SDK Config 변경 추적
- FR-0004 Agent 제어·관측
- FR-0005 시스템 운영·제어 *(범위 미확정 — open-issues #4)*
- C-0001 표준 패키징
- C-0002 배포 이식성

### qa/
- QA-0001 Efficiency / QA-0002 Scalability / QA-0003 Availability / QA-0004 Controllability
- QA-0005 Observability / QA-0006 Reliability(Workflow) / QA-0007 Performance(Agent수행시간)
- QA-0008 Performance(E2E) / QA-0009 Reliability(Agent일관성) / QA-0010 Maintainability
- QAS-0001.. (QA별 6-part 시나리오)

### dp/
- DP-0001 Workflow–Agent 매핑
- DP-0002 Agent Hierarchy
- DP-0003 Agent 외부 시스템 안정성 보장
- DP-0004 Workflow 실행 구조
- DP-0005 E2E 개발시간 최적화
- _backlog.md (신규 design approach 후보)
