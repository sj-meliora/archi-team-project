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
| 품질 속성 | qa/ | QA | **2자리**, id는 **고정 식별자**(우선순위는 asr.md가 관리 — 재번호 안 함) |
| 품질 속성 시나리오 | qa/ | QAS | 6-part 정형화, **2자리** |
| **ASR 선정·우선순위** | (flat) **asr.md** | — | ASR(=DP 생성 동인) 목록·우선순위 **SSoT**. QA id와 분리 |
| 설계 결정 | dp/ | DP | ATAM 분석을 문서 내부에 포함 |
| 산출물(뷰/도표) | artifacts/ | (자유) | context/domain diagram 등 |

- ID 포맷: `<CAT>-<NNNN>` (4자리 zero-pad). **단 QA·QAS·C는 2자리(`QA-01`, `C-01`)**. (FR/DP도 2자리 통일 예정.) 파일명 = `<ID>-<kebab-slug>.md`.
- **모든 ID(QA·QAS 포함)는 영구 고정 식별자**다(슬러그/내용만 변경). **우선순위·ASR 선정이 바뀌어도 QA id를 재번호하지 않는다** — 우선순위는 [`asr.md`](asr.md)에서만 관리(2026-06-27 정책 전환: 종전 "번호=우선순위→재번호"가 cross-ref 출렁임을 유발해 분리). cross-link은 본문에 ID 텍스트로(`DP-01`, `QA-03`) 적어 grep 역참조 가능하게.

## 전체 ID 목록

### requirements/
- FR-0001 Workflow 실행 인프라
- FR-0002 Artifact 저장·전달
- FR-0003 SDK Config 변경 추적
- FR-0004 Agent 제어·관측
- FR-0005 시스템 운영·제어 *(범위 미확정 — open-issues #4)*
- C-01 표준 패키징
- C-02 배포 이식성
- C-03 자율 에이전트 보안·안전 게이트 *(2026-06-26 QA-06 → 제약 이관, OI-10)*

### qa/ (**ASR 선정·우선순위 SSoT = [`asr.md`](asr.md)** — 현재 QA-01~05·QA-07. QA id는 고정 식별자)
- QA-01 Scalability / QA-02 Availability / QA-03 Controllability / QA-04 Observability / QA-05 Efficiency
- ~~QA-06 Security/Safety~~ → **C-03으로 이관**(2026-06-26 제약 이관, OI-10). 번호 06은 공석(QA-07~13 현 위치 유지). / **QA-07 Correctness** (ISO Functional Correctness)
- QA-08 Reliability(Workflow) / QA-09 Performance(Agent수행시간) / QA-10 Performance(E2E) / QA-11 Reliability(Agent일관성) / QA-12 Maintainability
- **QA-13 Cost-economy** (ISO Performance Efficiency: Resource Utilization)
- QAS-01~13 (QA별 6-part 시나리오, 같은 번호. **QAS-06 제거** — QA-06 제약 이관에 따름)
- *QA-07/13은 round-01 디스커션 발굴 → 2026-06-24 정식 편입(OI-8). 기존 QA-06~10 +2 시프트. QA-06(Security)은 2026-06-26 C-03으로 이관(OI-10).*

### dp/
- DP-01 에이전트 오케스트레이션 (제어성↔가용성) — **구 DP-0001(배치)+DP-0002(위상) 대체**. 짝 DP-02(Agent 특화)는 드랍(`discussion/dp/notes/dp2-drop-rationale.md`).
- DP-0003 Agent 외부 시스템 안정성 보장
- DP-0004 Workflow 실행 구조
- DP-0005 E2E 개발시간 최적화
- _backlog.md (신규 design approach 후보)
- *2026-06-27: 구 DP-0001·DP-0002 삭제. DP-0002 → DP-01 승계, 구 DP-0001(배치/풀) 결정은 재귀속 보류(OI-12). DP는 2자리 전환 중(DP-0003/0004/0005 추후).*
