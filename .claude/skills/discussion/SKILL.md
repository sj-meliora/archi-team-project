---
name: discussion
description: >-
  Run a methodology "discussion" round — a red-team Reviewer critique followed by
  a blue-team Council recommendation — over a concept area (QA, DP, …). Trigger
  when the user asks to run/do a discussion round, e.g. "QA 디스커션 한번 돌리자",
  "QA 리뷰 라운드 돌려줘", "DP 디스커션 돌려", "디스커션 한 바퀴 돌리자". Orchestrates
  the discussion-reviewer and discussion-council subagents per discussion/README.md.
---

# discussion — 심의 라운드 오케스트레이터

사용자가 "**\<영역\> 디스커션 돌리자**"류로 부르면 이 흐름을 따른다. 너(메인 에이전트)는 **오케스트레이터**이고, 실제 생성은 서브에이전트가 한다.

## 1. concept 파악
- 사용자 발화에서 영역을 뽑는다: "QA"→`qa`, "DP"→`dp`.
- `discussion/<concept>/Reviewer.md`·`Council.md`가 없으면: 아직 스펙이 없다고 알리고 `discussion/README.md`의 "새 개념 영역 추가하는 법"을 안내한 뒤 멈춘다. (없는 영역을 임의로 만들지 않는다.)
- 모호하면 어떤 영역인지 한 번 되묻는다.

## 2. 상태 파악 → 무엇을 돌릴지 결정
`discussion/<concept>/round-*`를 스캔한다.
- **라운드 없음** → `round-01` 신규: review 단계부터.
- **최신 라운드에 `review/`만 있고 `counsel/` 없음** → 그 라운드의 **council 단계**를 돌릴지, 아니면 새 라운드(review부터)를 뜰지 사용자에게 한 줄로 확인.
- **최신 라운드 review+counsel 모두 있음** → `round-(N+1)` 신규.

먼저 `discussion/README.md`(프로토콜) + `discussion/<concept>/`의 `Reviewer.md`·`Council.md`·`README.md`를 읽어 방법론을 로드한다.

## 3. Review 단계 (red team) — 샘플 체크포인트 후 전체
1. **Agent** 도구로 `subagent_type: discussion-reviewer` 호출, 프롬프트에 `concept`·`round`·`scope: sample` 전달.
2. 서브에이전트가 만든 **1~2개 샘플 + 보고서 골격**을 사용자에게 보여주고 포맷 확인을 받는다.
3. 확인되면 다시 `discussion-reviewer`를 `scope: full`로 호출해 나머지 + 종합 보고서/인덱스/신규후보 완성.
4. `discussion/<concept>/README.md` 마스터 인덱스(라운드 이력·verdict 추세) 갱신.

## 4. Council 단계 (blue team) — 샘플 체크포인트 후 전체
> 전제: 해당 라운드 `review/`가 완성돼 있어야 한다(Council의 입력).
1. **Agent** 도구로 `subagent_type: discussion-council` 호출, `concept`·`round`·`scope: sample`.
2. 샘플 권고 1~2개(개선안+레퍼런스+PoC) + `counsel.md` 골격을 보여주고 확인.
3. 확인되면 `scope: full`로 나머지 + `counsel.md`(종합·메타) + `_poc-plan.md` 완성.
4. 마스터 인덱스의 권고 추세·verdict 변화 갱신.

## 5. 마무리
- 라운드 산출물 트리와 핵심 결론(판정 집계·주요 권고)을 요약한다.
- **커밋 여부를 묻는다**(자동 커밋하지 않음). 메시지는 Conventional Commits(`feat(discussion): …`).

## 원칙
- 각 단계는 **반드시 샘플 1~2개 먼저 → 확인 → 나머지** (프로젝트 규칙·사용자 설정).
- 서브에이전트의 반환은 사용자에게 직접 보이지 않으니, 너가 핵심을 추려 전달한다.
- 방법론 세부는 전부 `discussion/<concept>/` 스펙에 있다 — 여기서 재정의하지 않는다.
- review와 counsel은 한 라운드의 두 반쪽이다. counsel은 그 라운드 review를 입력으로 삼는다.
