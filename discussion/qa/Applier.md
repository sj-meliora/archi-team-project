# QA Applier — 반영 방법론 (도메인 스펙)

> 한 심의 라운드(Reviewer 비평 + Council 권고)의 결과를 **원본 `context/qa/`에 실제로 반영**하는 단계의 스펙.
> discussion 사이클의 3번째 역할 — red(비평) → blue(권고) → **applier(반영)**.
> 이 스펙이 진실이다. 에이전트(`.claude/agents/discussion-applier.md`)는 얇은 실행기일 뿐.

## 이 단계가 다른 점 (중요)
- Reviewer/Council은 `discussion/`만 쓰고 원본은 안 건드린다(append-only). **Applier만 유일하게 `context/qa/` 원본을 수정**한다.
- 입력(review·counsel·filter)은 **읽기 전용**. 절대 수정하지 않는다.
- 단, Applier는 **자신의 반영 보고서**를 `discussion/<concept>/<round>/applier/`에 쓴다(아래 [§라운드 반영 보고서](#라운드-반영-보고서-reviewer-앞--다음-라운드로-루프-닫기)). review·counsel·filter는 여전히 읽기 전용이고, `applier/`만 Applier의 출력 폴더다. 이게 red→blue→**applier** 사이클을 다음 라운드 Reviewer에게 **보고**로 닫는 장치다.

## 입력 (호출자가 프롬프트로 준다)
- `concept` — 영역 (여기선 `qa`)
- `round` — 라운드 (예: `round-01`)
- `targets` — 반영할 QA ID 목록 (예: `QA-02` 또는 `QA-02..QA-10, NQA-A`). 1개씩 검토받는 게 기본.

## 읽을 입력 (반드시 다 읽는다)
1. `discussion/<concept>/<round>/review/<ID>.md` — red team 지적 (무엇이 틀렸나)
2. `discussion/<concept>/<round>/counsel/<ID>.md` — blue team 권고 (어떻게·왜 바꾸나 + 레퍼런스)
2b. **(ASR 대상이면) `discussion/<concept>/<round>/contention/counter.md`(+있으면 `referee.md`)** — red↔blue 반박 후 **최종 권고 델타**. **counter의 `수용`/`부분수용` 델타가 원 counsel보다 우선**(referee 지시가 있으면 그게 최종). contention 폴더가 없으면(=ASR 아님) 생략. → 방법론 [`Contention.md`](Contention.md).
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
- **주 KPI 1개를 반드시 지정**한다(인증 과제는 하나를 골라 실제 측정·시연). 그 QA의 *정의 그 자체*를 가장 직접 재고 PoC로 보이기 좋은 것을 고른다. 나머지는 보조(가드레일).
  - 섹션 맨 앞에 안내 꼬리줄: `> 주 KPI(헤드라인·PoC 대상)는 \`<주 KPI>\` 1개. 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.`
  - 주 KPI 줄 끝에 `` `[주 KPI · PoC 대상]` `` 라벨을 붙인다(inline code). 보조 KPI엔 라벨 없음.
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
  - **책임지는 설계**: `related-dp`의 DP를 읽고, 그 DP의 trade-off 매트릭스 별점·tactic·Risk/Sensitivity 중 이 KPI에 해당하는 주장을 **인용**한다(예: "DP-01 3안 Standby ★★★", "DP-0004 R-3").
  - **검증 실험·모델**: 그 설계 주장을 **숫자로 바꿀** 간단 실험(큐잉 시뮬·버스트 시뮬·파이프라인 모델 등). 실제 시스템 없이 가능한 것만.
  - **주/보조 구분**: 주 KPI 행은 KPI 칸 끝에 `` `[주]` `` 표식 + 검증 칸을 `**▶ 실제 제작:** …`로 시작(실제로 만들 1개). 보조 행은 검증 칸을 `보조 모델: …`로 시작(만들면 보조 확인용, 안 만들어도 됨).
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
- DP 역검토 메모(예: "DP-0004가 rate-limit·admission control 미명시")는 원본 DP를 고치지 말고 `context/open-issues.md`에 트래킹 항목으로만 남긴다(또는 변경 이력의 "남은 일"에).
- 신규 QA(NQA-x)는 같은 6섹션 골격으로 `context/qa/`에 신설. 번호 재정렬은 별도 결정 사항 — 임의로 재번호하지 않고 반환문에 제안만.

## 라운드 반영 보고서 (Reviewer 앞 — 다음 라운드로 루프 닫기)

반영을 마치면, **이번에 반영한 라운드의 review/counsel을 어떻게 처리했는지**를 Reviewer가 받아볼 보고서로 남긴다. 이게 red→blue→applier 사이클의 출력이며, **다음 라운드 Reviewer의 입력**이 된다(자기 지적이 [반영]/[발표 서사]/[생략]/[거부]/[이월] 중 무엇이 됐는지 확인하고 verdict 변화를 재평가하라고).

- **출력 위치**: `discussion/<concept>/<round>/applier/report.md` (review/·counsel/와 나란한 **세 번째 폴더**). 라운드당 1개로 누적한다 — `targets`를 나눠 여러 번 반영해도 **한 파일에 append**(라운드가 닫힐 때까지 살아 있는 합본).
- **append-only**(라운드 종료 후): 일단 다음 라운드가 시작되면 수정하지 않는다(오타·링크만 예외).
- 보고서는 모든 red-team 지적에 **disposition(처리 등급)을 빠짐없이 단다** — "안 한 것"도 [생략]/[거부]/[이월]로 명시해 Reviewer가 추적 가능하게.

### disposition 등급 (지적별 처리)
- **[반영]** — 원본 `context/`에 실제 기입.
- **[발표 서사]** — 검증 "방법 한 줄"만 남기고 실측은 미실행(슬라이드용).
- **[생략]** — 원본에 안 씀(안 만들 시스템의 실행 관리물 등).
- **[거부]** — counsel이 권고했으나 미채택. **반드시 사유**를 단다(자가당착·과에포트·범위 밖 등).
- **[이월]** — 이번 라운드 미처리, 다음 라운드로 넘김(전제 미충족 등).

### 보고서 골격 (`applier/report.md`)
```
# Applier 반영 보고서 — <round> (concept=<qa>)

> 반영 대상: <round>의 review + counsel + _feasibility-filter
> 반영 일자: YYYY-MM-DD · 반영 범위: <targets> · 원본: context/<concept>/
> 다음 라운드 Reviewer는 이 보고서를 입력으로 읽고, 각 지적의 처리를 확인해 verdict 변화를 재평가한다.

## 1. 한눈에 (항목별 처리 요약)
| ID | review verdict | 처리 등급 | 무엇을 바꿨나(1줄) | 이월/재검증 |
(QA·NQA별 1행)

## 2. 지적별 처리 (closure)
교차분석 C*와 개별 QA 핵심 지적마다 disposition + (필요시)사유.
- **C2 정의↔KPI 불일치** → [반영]: QA-08 …
- … 모든 지적이 등급을 받는다([거부]는 사유 필수)

## 3. 다음 Reviewer가 다시 볼 것 (재검증 요청)
[발표 서사]로 미룬 검증 · [이월] · 사람 결정 보류 · DP 역검토(open-issues OI-*) 목록.

## 4. 사람 결정 보류
수치 자가당착 · 번호 재정렬 · NQA 신설 가부 등.
```
- §1 표의 `이월/재검증` 칸과 §3가 다음 라운드 Reviewer의 **할 일 목록**이다 — 여기에 적힌 것이 라운드 N+1의 우선 점검 대상.
- 개별 QA `## 변경 이력`의 "남은 일"과 정합을 맞춘다(중복이 아니라 **Reviewer 관점으로 재집계**: 무엇을 다시 봐달라).

## 반환 (호출자에게)
- 생성·수정한 파일 목록(원본 `context/` + **`applier/report.md`**).
- 핵심 변경 요약(어느 KPI를 무엇으로, 무엇을 [생략]/[거부]했나).
- **사람 결정 필요점**(수치 자가당착·번호 재정렬·NQA 신설 가부·DP 역검토 등)을 **간결히** 별도로.
