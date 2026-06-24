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
    ├── Applier.md            반영 방법론 — 원본 문서 골격·등급별 반영 규칙
    ├── README.md             그 영역의 마스터 인덱스 (라운드 이력·verdict 추세)
    └── round-NN/             한 라운드 (날짜 없음 — 메타는 보고서에 기록)
        ├── review/           red team 산출물: report.md(결론)·README.md·항목별 상세·신규후보
        ├── counsel/          blue team 산출물: counsel.md(권고)·항목별 개선안·_poc-plan.md
        ├── contention/       (ASR 대상만) red↔blue 바운드 1왕복: rebuttal.md·counter.md·(선택)referee.md
        └── applier/          applier 산출물: report.md — 반영 보고서(지적별 disposition, 다음 Reviewer 입력)
```

## 라운드 규칙 (모든 영역 공통)

- **1 라운드 = 대상 전체 스냅샷에 대한 한 번의 심의**(비평 + 권고).
- **`round-NN/` 폴더명에 날짜를 넣지 않는다.** 날짜 등 메타는 그 라운드의 종합 보고서(`review/report.md`, `counsel/counsel.md`)에 기록 — *공의회가 최종 결론을 문서화하듯*.
- **append-only**: 지난 라운드는 수정하지 않고 보존(추세 비교용). 오타·링크만 예외.
- **순환**: 심의(라운드 N) → **원본(`context/<concept>/`) 반영** → 라운드 N+1로 검증 → 반복. 반영은 **Applier**(red→blue→applier의 3번째 역할)가 수행 — review·counsel·filter를 읽어 등급 [반영]만 원본에 쓴다. 방법론은 `discussion/<concept>/Applier.md`, 워커는 [`.claude/agents/discussion-applier.md`](../.claude/agents/discussion-applier.md).
- **보고-루프(red↔applier 닫기)**: 반영을 마치면 Applier가 **반영 보고서**(`round-NN/applier/report.md`)를 남겨, 모든 지적의 처리([반영]/[발표 서사]/[생략]/[거부]+사유/[이월])를 명시한다. **다음 라운드 Reviewer는 이 보고서를 입력으로 받아**(Reviewer §8 절차 0), [반영]은 verdict 변화로 재평가하고 [발표 서사]/[이월]/[거부]는 다시 판정한다 — 지적이 처리됐는지 추적 가능해진다.
- **contention(선택·ASR 대상만)**: counsel 직후 applier 전에, red↔blue가 **바운드된 1 왕복**(rebuttal→counter, +선택 referee)으로 약한 반박·회피된 지적·헤드라인 KPI 모순을 *반영 전에* 잡는다. **자유 토론 아님**(수렴/아첨 방지 — 1왕복 상한 + 안티-수렴 가드). 비싸서 **ASR 선정 QA에만**. 방법론: [`<concept>/Contention.md`](qa/Contention.md), 워커: [`.claude/agents/discussion-contention.md`](../.claude/agents/discussion-contention.md).
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
