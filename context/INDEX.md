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
| 품질 속성 | qa/ | QA | **2자리**, 번호 = 발표 우선순위(재정렬 가능) |
| 품질 속성 시나리오 | qa/ | QAS | 6-part 정형화, **2자리** |
| 설계 결정 | dp/ | DP | ATAM 분석을 문서 내부에 포함 |
| 산출물(뷰/도표) | artifacts/ | (자유) | context/domain diagram 등 |

- ID 포맷: `<CAT>-<NNNN>` (4자리 zero-pad). **단 QA·QAS는 2자리(`QA-01`)** — 번호가 곧 발표 우선순위. (FR/C/DP도 2자리 통일 예정.) 파일명 = `<ID>-<kebab-slug>.md`.
- FR/C/DP ID는 영구 고정(슬러그/내용만 변경). **QA·QAS 번호는 우선순위가 바뀌면 재번호.** cross-link은 본문에 ID 텍스트로(`DP-0002`, `QA-03`) 적어 grep 역참조 가능하게.

## 전체 ID 목록

### requirements/
- FR-0001 Workflow 실행 인프라
- FR-0002 Artifact 저장·전달
- FR-0003 SDK Config 변경 추적
- FR-0004 Agent 제어·관측
- FR-0005 시스템 운영·제어 *(범위 미확정 — open-issues #4)*
- C-0001 표준 패키징
- C-0002 배포 이식성

### qa/ (우선순위 = 번호, 2026-06-24 재번호 / **ASR = QA-01~07**)
- QA-01 Scalability / QA-02 Availability / QA-03 Controllability / QA-04 Observability / QA-05 Efficiency
- **QA-06 Security/Safety** (ISO 25010 Security+Safety) / **QA-07 Correctness** (ISO Functional Correctness)
- QA-08 Reliability(Workflow) / QA-09 Performance(Agent수행시간) / QA-10 Performance(E2E) / QA-11 Reliability(Agent일관성) / QA-12 Maintainability
- **QA-13 Cost-economy** (ISO Performance Efficiency: Resource Utilization)
- QAS-01~13 (QA별 6-part 시나리오, 같은 번호)
- *QA-06/07/13은 round-01 디스커션 발굴 → 2026-06-24 정식 편입(OI-8). 기존 QA-06~10 +2 시프트.*

### dp/
- DP-0001 Workflow–Agent 매핑
- DP-0002 Agent Hierarchy
- DP-0003 Agent 외부 시스템 안정성 보장
- DP-0004 Workflow 실행 구조
- DP-0005 E2E 개발시간 최적화
- _backlog.md (신규 design approach 후보)
