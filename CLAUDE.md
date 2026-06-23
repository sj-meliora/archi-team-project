# CLAUDE.md

## 프로젝트 목적

소프트웨어 아키텍트 인증 과정의 **17조 팀 과제**를 수행하는 저장소다.
최종 목표는 **발표 슬라이드**(발표 예정일 2026-07-07)와 그에 필요한 문서·다이어그램·도표를 생성하는 것.

과제 주제: *Agentic AI 기반 On-device AI SDK 개발 자동화 시스템* — NPU 구동용 On-device AI SDK 파이프라인(IR Converter → Graph Optimizer → Quantizer → Compiler)을 Agent가 사람 개입 없이 빌드·검증·배포·이슈처리까지 자동화하는 시스템의 아키텍처 설계.

## 프로젝트 구조

```
archi-team-project/
├── source/        팀 공유 pptx 최신본을 주기적으로 다운로드해 보관 (원본 자료)
├── context/       과제 컨텍스트의 단일 진실 공급원(SSoT) — 방법론 개념 단위로 관리
│   ├── INDEX.md         색인·읽기순서·전체 ID 목록
│   ├── overview.md      시스템 정의·배경·Pain Point (OV)
│   ├── stakeholders.md  이해관계자 (ST)
│   ├── glossary.md      용어·약어·QA번호 슬라이드별 매핑
│   ├── open-issues.md   정합성/미결정 트래커 (OI-1~6)
│   ├── changelog.md     자료 변경 델타 로그
│   ├── requirements/    FR / NFR / C (기능요구·비기능·제약)
│   ├── qa/              QA(품질속성) + QAS(6-part 시나리오)
│   ├── dp/              DP(설계결정) — ATAM 분석을 각 문서 내부에서 수행, _backlog.md
│   └── artifacts/       context-diagram, domain-diagram 등 산출물 (Mermaid)
└── slides/        발표 슬라이드 초안 (최종 산출물)
```

### context 관리 규칙
- **방법론 개념 단위**로 관리한다 (FR / NFR / Constraint / QA / QAS / DP / ATAM). DP 보강 시 driving QA의 tactic·pattern에서 새 design approach를 발굴하기 위함이다.
- **ID 포맷**: `<CAT>-<NNNN>` (4자리 zero-pad). **단 QA·QAS는 2자리(`QA-01`, `QAS-01`)** — 번호가 곧 발표 우선순위. (FR/C/DP도 추후 2자리로 통일 예정.) 파일명 = `<ID>-<kebab-slug>.md`. FR/C/DP의 ID는 영구 고정하고 슬러그/내용만 바꾼다. **QA·QAS 번호는 우선순위가 바뀌면 재번호한다.**
- **cross-link**은 본문에 ID 텍스트(`DP-0002`, `QA-03`)로 적어 grep 역참조가 되게 한다.
- **항목은 파일로 분리**한다. 커지면 폴더로 승격하되 ID는 유지한다.
- 자료가 바뀌면 최신 pptx를 `source/`에 넣고 **델타만 `changelog.md`에 append**한다. 정합성·미결정은 `open-issues.md`로 관리한다.
- **QA canonical 번호 = 팀 합의 우선순위 기준**으로 통일한다 (과거 슬라이드 13 순서에서 2026-06-23 재정렬). 원본 슬라이드 대조는 `glossary.md`.
- 대량 파일 생성 전에는 신규 포맷 1~2개를 먼저 보여주고 확인받은 뒤 나머지를 생성한다.

## 커밋 규칙

[Conventional Commits](https://www.conventionalcommits.org/)를 지킨다.

형식: `<type>(<scope>): <subject>`

| type | 용도 |
|---|---|
| `feat` | 새 산출물·기능 추가 (새 DP, 슬라이드, 다이어그램 등) |
| `fix` | 오류 수정 (정합성 오류 교정, 잘못된 수치 등) |
| `docs` | 문서 내용 보강·정리 |
| `refactor` | 구조 변경 (파일 분리·이동, 리네이밍 등 내용 변화 없음) |
| `chore` | 잡무 (설정, 보관 이동 등) |

- scope는 선택. 가능하면 ID나 폴더를 쓴다 (예: `feat(dp): DP-0002 Standby 대안 추가`, `fix(qa): QA 번호 슬라이드13 기준 통일`).
- subject는 한국어로 간결하게, 명령형/요약형.

## 언어
사용자와의 소통 및 문서 작성은 **한국어**로 한다.
