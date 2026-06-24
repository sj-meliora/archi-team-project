# QA Applier — 반영 방법론 (도메인 스펙)

> 한 심의 라운드(Reviewer 비평 + Council 권고)의 결과를 **원본 `context/qa/`에 실제로 반영**하는 단계의 스펙.
> discussion 사이클의 3번째 역할 — red(비평) → blue(권고) → **applier(반영)**.
> 이 스펙이 진실이다. 에이전트(`.claude/agents/discussion-applier.md`)는 얇은 실행기일 뿐.

## 이 단계가 다른 점 (중요)
- Reviewer/Council은 `discussion/`만 쓰고 원본은 안 건드린다(append-only). **Applier만 유일하게 `context/qa/` 원본을 수정**한다.
- 입력(review·counsel·filter)은 **읽기 전용**. 절대 수정하지 않는다.

## 입력 (호출자가 프롬프트로 준다)
- `concept` — 영역 (여기선 `qa`)
- `round` — 라운드 (예: `round-01`)
- `targets` — 반영할 QA ID 목록 (예: `QA-02` 또는 `QA-02..QA-10, NQA-A`). 1개씩 검토받는 게 기본.

## 읽을 입력 (반드시 다 읽는다)
1. `discussion/<concept>/<round>/review/<ID>.md` — red team 지적 (무엇이 틀렸나)
2. `discussion/<concept>/<round>/counsel/<ID>.md` — blue team 권고 (어떻게·왜 바꾸나 + 레퍼런스)
3. `discussion/<concept>/<round>/counsel/_feasibility-filter.md` — **현실성 등급**([반영]/[발표 서사]/[생략]). 이게 *무엇을 원본에 쓸지*를 결정한다.
4. 원본 `context/qa/<ID>.md` + 짝 `context/qa/QAS-<NN>.md`
5. `related-dp`에 적힌 DP들 `context/dp/DP-XXXX.md` — **검증 전략** 섹션을 쓰려면 그 설계가 뭘 주장하는지 알아야 한다.
6. (필요 시) `context/overview.md`·`glossary.md` — 배경·용어.

## 반영 원칙 (feasibility filter 등급별)
- **[반영]** — KPI/정의 교정은 원본에 **실제로 쓴다**.
- **[발표 서사]** — 검증 "방법 한 줄"만 `## 검증 전략`에 남긴다. 실행 결과(수치 증명)는 안 쓴다.
- **[생략]** — 원본에 안 쓴다. 필요하면 `## 변경 이력`의 "남은 일"에 한 줄로만.
- 숫자(0.8·20% 등)는 placeholder `◯`로 두지 말고 **구체 예시값**으로 채운다(측정가능 KPI의 모양을 보여주는 게 목적). 단 "예시값이며 실측/모델로 확정"임을 명시.
- counsel과 review가 충돌하면 counsel(최종 권고)을 따르되, **수치 자가당착**(예: efficiency 0.8인데 처리량 1.8배=0.9)을 발견하면 고치지 말고 **반환문에 플래그**해서 사람이 정하게 한다.

## QA 문서 골격 (6섹션 — 이 순서 고정)

QA-01(`context/qa/QA-01-scalability.md`)이 **확정 템플릿**이다. 막히면 그 파일을 열어 그대로 모방한다.

### 0. frontmatter (YAML)
```yaml
---
id: QA-0N
category: QA
importance: H|M|L
difficulty: H|M|L
source: pptx p.NN
related-dp: [DP-XXXX, ...]
updates:
  - date: YYYY-MM-DD
    by: discussion/qa/<round>
    reason: "한 줄 요약 (자세히 → ## 변경 이력)"
---
```
- 최상단 `updated:` 필드는 **두지 않는다**(이력으로 갈음). `updates`는 라운드마다 항목 1개 **append**.

### 1. `## 정의 / Refinement`
- 팀원이 그냥 읽고 이해되게 **풀어쓴다**. 방향이 아니라 명세 — *무엇이* 비례/보장되는지 못 박는다.
- 다른 QA와 경계가 겹치면 `> altitude:` 한 줄로 분리 명시(예: throughput=QA-01, per-node=QA-07).

### 2. `## 측정 (KPI)`
- 각 KPI는 **기술 표기(측정 기준) 그대로** 한 줄 + 바로 아래 `- 쉽게: …` 하위줄로 풀이.
- 처음 나오는 전문용어는 풀이줄에서 한글 뜻을 붙인다(TPM/RPM/p95/backlog/SLI/cold-start 등).
- 폐기한 KPI는 `> 폐기: …(사유)` 로 남겨 추적성 유지.
- 예시값임을 알리는 `> 위 수치(…)는 …예시값이며 실제 합격 기준은 실제 환경에서 측정해 확정한다.` 꼬리줄.

### 3. `## 근거 / 레퍼런스`
- counsel의 **근거(레퍼런스)** 섹션에서 가져온다. 표 3열: `KPI 선택 | 왜 이렇게 정의했나 | 출처(링크)`.
- 출처는 클릭 가능한 markdown 링크. counsel에 URL이 있으면 그대로, 부족하면 그 사실만 적고 발명하지 않는다.
- 꼬리줄: `> ⚠️ 레퍼런스 수치는 패턴 정당화용이며 복제하지 않는다 — 합격선은 [검증 전략](#검증-전략)으로 확정.`

### 4. `## 검증 전략`
- 한 줄 도입: "각 KPI를 실제로 달성하는 건 특정 설계 결정(DP)이다. 간단한 시뮬레이션/모델로 (실제 시스템 없이) 보인다 — 설계 주장(별점)을 근거 있는 그래프로."
- 표 3열: `KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델`.
  - **책임지는 설계**: `related-dp`의 DP를 읽고, 그 DP의 trade-off 매트릭스 별점·tactic·Risk/Sensitivity 중 이 KPI에 해당하는 주장을 **인용**한다(예: "DP-0001 2안 동적 풀 ★★★", "DP-0004 R-3").
  - **검증 실험·모델**: 그 설계 주장을 **숫자로 바꿀** 간단 실험(큐잉 시뮬·버스트 시뮬·파이프라인 모델 등). 실제 시스템 없이 가능한 것만.
- 꼬리줄: `> 가정·한계: …는 가정 파라미터다. 이 실험이 증명하는 건 "설계가 이런 메커니즘으로 KPI를 달성하고 KPI가 이 방법으로 측정 가능"이지 가상 시스템 실측치가 아니다.`

### 5. `## 변경 이력`
- 라운드마다 `### YYYY-MM-DD — <round> 디스커션 반영` 블록을 **append**(과거 블록 보존).
- 블록 구성 3개:
  - `**무엇이 문제였나 (review 지적)**` — red team 핵심 지적 bullet.
  - `**무엇을 바꿨나 (반영)**` — 정의/KPI 실제 변경 + QAS 동기화 사실.
  - `**남은 일 (이 라운드에서 미반영)**` — [생략]/[발표 서사]로 분기한 것, DP 역검토, 신규 QA 전제 등 follow-up.
- 출처 링크: `출처: [discussion/qa/<round>](…/counsel/<ID>.md) (verdict: …)`.

## 짝 QAS 동기화 (필수)
- `context/qa/QAS-<NN>.md`의 **Response·Measure** 행을 새 정의·KPI와 일치시킨다(SSoT 통일). 다른 행은 필요 시에만.

## 부수 작업 (거의 0비용 — 빠뜨리지 말 것)
- DP 역검토 메모(예: "DP-0001이 rate-limit·admission control 미명시")는 원본 DP를 고치지 말고 `context/open-issues.md`에 트래킹 항목으로만 남긴다(또는 변경 이력의 "남은 일"에).
- 신규 QA(NQA-x)는 같은 6섹션 골격으로 `context/qa/`에 신설. 번호 재정렬은 별도 결정 사항 — 임의로 재번호하지 않고 반환문에 제안만.

## 반환 (호출자에게)
- 생성·수정한 파일 목록.
- 핵심 변경 요약(어느 KPI를 무엇으로, 무엇을 [생략]했나).
- **사람 결정 필요점**(수치 자가당착·번호 재정렬·NQA 신설 가부·DP 역검토 등)을 **간결히** 별도로.
