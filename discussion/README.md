# discussion — 심의 프레임워크 (도메인 무관 프로토콜)

> 방법론 개념 영역(QA, DP, …)을 **red team 비평 + blue team 권고**로 반복 심의(라운드)해 완성도를 끌어올리는 공통 틀.
> 이 파일은 **도메인 무관 프로토콜**만 담는다. 각 영역의 렌즈·rubric·레퍼런스는 `discussion/<concept>/`의 스펙에 둔다.

## 두 팀 (역할)

| 팀 | 에이전트 | 산출 | 한 줄 |
|---|---|---|---|
| 🔴 red | **Reviewer** | verdict(판정) | 현재 산출물의 약점을 렌즈별로 깐다 |
| 🔵 blue | **Council** | counsel(권고) | Reviewer 지적을 읽고 **근거(레퍼런스)+PoC**를 붙여 개선을 권고한다 |

red가 *무엇이 틀렸나*를 말하면, blue가 *어떻게 바꾸고·왜 맞고·어떻게 증명하나*를 답한다. 한 라운드 = 이 한 쌍의 심의.

## 폴더 규약

```
discussion/
├── README.md                ← (이 파일) 공통 프로토콜
└── <concept>/               ← 개념 영역별 인스턴스 (qa, dp, …)
    ├── Reviewer.md           red team 방법론 — 도메인별 렌즈·rubric
    ├── Council.md            blue team 방법론 — 도메인별 레퍼런스·PoC
    ├── README.md             그 영역의 마스터 인덱스 (라운드 이력·verdict 추세)
    └── round-NN/             한 라운드 (날짜 없음 — 메타는 보고서에 기록)
        ├── review/           red team 산출물: report.md(결론)·README.md·항목별 상세·신규후보
        └── counsel/          blue team 산출물: counsel.md(권고)·항목별 개선안·_poc-plan.md
```

## 라운드 규칙 (모든 영역 공통)

- **1 라운드 = 대상 전체 스냅샷에 대한 한 번의 심의**(비평 + 권고).
- **`round-NN/` 폴더명에 날짜를 넣지 않는다.** 날짜 등 메타는 그 라운드의 종합 보고서(`review/report.md`, `counsel/counsel.md`)에 기록 — *공의회가 최종 결론을 문서화하듯*.
- **append-only**: 지난 라운드는 수정하지 않고 보존(추세 비교용). 오타·링크만 예외.
- **순환**: 심의(라운드 N) → Stage 2에서 원본(`context/<concept>/`) 개선 → 라운드 N+1로 검증 → 반복.
- **추적성**: 권고가 원본에 반영되면 `changelog.md` 델타 + 다음 라운드 종합 보고서에 verdict 변화 기록.

## 새 개념 영역 추가하는 법 (예: DP)

1. `discussion/dp/Reviewer.md` — DP용 렌즈·rubric 작성(QA의 측정가능성 rubric 대신 ATAM tradeoff/risk/sensitivity 등).
2. `discussion/dp/Council.md` — DP용 레퍼런스 라이브러리·PoC 수단.
3. `discussion/dp/README.md` — 그 영역 마스터 인덱스.
4. 끝. **`.claude/`의 에이전트·트리거는 도메인 무관**이라 그대로 동작한다.

## 트리거

자연어로 발동한다 (Skill `discussion`):
- "**QA 디스커션 한번 돌리자**" → concept=qa 라운드 실행
- "**DP 디스커션 돌려줘**" → concept=dp (스펙 존재 시)

발동 시 Reviewer→Council 순으로 한 라운드를 돌린다. 각 단계는 **1~2개 샘플 먼저 보여주고 확인**받은 뒤 나머지를 생성한다(프로젝트 규칙). 구현: [`.claude/skills/discussion/SKILL.md`](../.claude/skills/discussion/SKILL.md), 워커: [`.claude/agents/discussion-reviewer.md`](../.claude/agents/discussion-reviewer.md)·[`discussion-council.md`](../.claude/agents/discussion-council.md).

## 개념 영역 레지스트리

| concept | 상태 | 대상 원본 | 인덱스 |
|---|---|---|---|
| **qa** | active (round-01 review 완료) | `context/qa/` | [qa/README.md](qa/README.md) |
| **dp** | planned | `context/dp/` | — |
