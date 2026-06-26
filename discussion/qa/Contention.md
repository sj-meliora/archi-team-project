# Contention — 라운드 내 적대적 반박 단계 (ASR selective)

> 목적: counsel(blue) 직후, applier 반영 **전**에 red↔blue가 **바운드된 1 왕복**으로 쟁점을 다툰다.
> red→blue 일방 파이프라인의 빈틈(약한 반박·회피된 지적·가짜 합의)을 *반영 전에* 잡는다.
> **상태: selective 채택** — **ASR(architecturally significant)로 선정된 QA에만** 적용. 현재 선정: **QA-01~05, QA-07**(2026-06-24 Security·Correctness 편입으로 QA-01~07 확장, OI-8 → 2026-06-26 Security가 제약 C-03으로 이관돼 ASR에서 빠짐, OI-10). (round-01 QA-09[現 QA-11] 트라이얼로 가치 검증 완료 — 헤드라인 KPI 모순을 해소.)
> 공통 프로토콜·다른 역할: [`../README.md`](../README.md) · [`Reviewer.md`](Reviewer.md) · [`Council.md`](Council.md) · [`Applier.md`](Applier.md).

## 적용 대상 (ASR selective — 전수 아님)
- contention은 비싸다(트라이얼 기준 QA 1개당 ~78k 토큰·2단계). 그래서 **전 QA에 매 라운드 돌리지 않는다.**
- **대상 = ASR(architecturally significant)로 선정된 QA뿐.** ASR 선정은 **팀 입력**이다(우선순위 = 발표 비중 상위, DP 생성 동인). **현재 선정: QA-01~05, QA-07**(2026-06-26 Security → 제약 C-03 이관으로 QA-06 제외, OI-10).
- ASR 아닌 QA(건강한·저우선)는 `counsel → applier`로 직행(contention 생략). 건강한 QA에 억지로 돌리면 공허한 트집만 늘어난다.
- ASR 선정이 바뀌면 이 줄과 skill의 대상 목록을 갱신한다.

## 왜 (파이프라인의 빈틈)
- Reviewer는 한 번 까고, Council은 한 번 답하고 끝 → **Council이 지적을 회피·약하게 답해도 검증할 기회가 없다.** Applier가 뒤에서 일부 거르지만, blue의 권고를 red 관점으로 되치는 단계가 없다.
- 단, **자유 토론(free-form chat)은 금지** — LLM 에이전트는 수렴/아첨(sycophancy)·가짜 합의·추적성 손실로 빠진다. 그래서 **바운드된 구조적 반박**으로 대체한다.

## 위치 (플로우)
```
review/ → counsel/ → [contention/] → applier/
```
- 출력 폴더: `discussion/<concept>/<round>/contention/`
  - `rebuttal.md` — red 되치기 (Reviewer 페르소나)
  - `counter.md` — blue 응답 (Council 페르소나)
  - `referee.md` — (선택) 중립 판결 — counter 후에도 충돌이 남을 때만
- append-only(라운드 종료 후). review·counsel은 불가침(읽기 전용).

## 바운드 규칙 (수렴 폭주·아첨 방지 — 핵심)
- **정확히 1 왕복**: rebuttal(red) → counter(blue). 그 이상 핑퐁 금지.
- 미해결 쟁점만 referee(중립) **1회** 판결, 또는 Applier/사람에게 에스컬레이트.
- **안티-수렴 가드(필수)**: 각 단계는 **최소 1개 진짜 이견**을 내거나, "남은 이견 없음 + 사유"를 **명시 인증**해야 한다. "좋은 지적입니다/동의합니다"만으론 단계 종료 불가 — 동의라면 *무엇을 왜* 동의하는지 근거를 단다.
- **독립성 보존**: rebuttal은 원 review를 쓴 red 관점을 유지하되 counsel을 처음 정독한다(앵커링은 이 시점부터 허용 — 그 전 review 독립성은 이미 확보됨).

## 역할 1 — Rebuttal (red 되치기)
- 입력: `review/<ID>.md`(자기 원 지적) + `counsel/<ID>.md`(blue 응답) + 원본 `context/qa/<ID>.md`(현 상태).
- **원 지적마다** Council 처리를 분류:
  - `해소` — 충분히 답함(증거·PoC가 지적을 실제로 닫음). **근거 1줄 필수.**
  - `회피` — 다른 걸 답하거나 핵심을 비켜감.
  - `부분` — 일부만 답하고 빈틈 남음.
  - `반박` — red가 여전히 동의 못 함(에스컬레이트).
- 회피/부분/반박엔 **무엇이 왜 부족한지** 1~2줄 + Council에 던지는 **구체 질문 1개**.
- 추가로 counsel이 **새로 끌어들인 주장/수치**(원 review에 없던 것)도 점검 — 근거 충분한가?
- 출력 `rebuttal.md`: 지적별 분류표 + **에스컬레이트 쟁점 목록**(번호 R1, R2…).

## 역할 2 — Counter (blue 응답)
- 입력: 위 + `rebuttal.md`.
- **에스컬레이트 쟁점(R*)마다**:
  - `수용` — red 맞음 → **권고 수정안 제시**(기존 → 새 제안, Applier가 반영할 델타).
  - `방어` — 추가 증거/레퍼런스로 재반박(왜 원 권고가 옳은지).
  - `부분수용` — 일부 수정 + 일부 방어.
- 출력 `counter.md`: 쟁점별 수용/방어 + **최종 권고 델타**(원 counsel 대비 무엇이 바뀌나) + 남은 충돌(referee 회부 여부).

## 역할 3 (선택) — Referee (중립 판결)
- counter 후에도 남은 충돌만. review+counsel+rebuttal+counter를 읽고 쟁점별 1줄 판결: **Council 유지 / Reviewer 인용 / 합성안** + Applier 지시 1줄.
- 중립 스탠스 — rebuttal/counter를 쓴 에이전트와 **다른 호출**로 독립 실행.

## Applier 연계 (반영 시)
- Applier는 review+counsel **+ contention(`counter.md`의 최종 델타, 있으면 `referee.md` 지시)** + `_feasibility-filter`를 읽고 반영한다.
- **우선순위**: counter의 `수용` 델타 > 원 counsel(최신 권고 우선). referee 지시가 있으면 그게 최종.
- Applier 반영 보고서(`applier/report.md`)의 disposition에 **contention 결과를 반영**한다(예: "Council 회피 → counter에서 수용 → [반영]").

## 산출물 포맷

### `rebuttal.md`
```
# Rebuttal — <ID> (round-NN, red 되치기)
> 입력: review/<ID>.md + counsel/<ID>.md · 스탠스: red(원 지적 유지)

## 지적별 Council 처리 분류
| 원 지적(요약) | Council 처리 | 판정 | 부족한 점·질문 |
(해소/회피/부분/반박)

## 에스컬레이트 쟁점
- **R1**: <쟁점> — <왜 안 닫혔나> — Council에 질문: <…>
- (최소 1개. 없으면 "이견 없음" + 사유 명시)
```

### `counter.md`
```
# Counter — <ID> (round-NN, blue 응답)
> 입력: rebuttal.md · 스탠스: blue(권고 방어/수정)

## 쟁점별 응답
- **R1** [수용|방어|부분수용]: <응답> → 권고 델타: `기존 → 새 제안`(또는 방어 근거)

## 최종 권고 델타 (Applier 반영용)
- <원 counsel 대비 바뀐 KPI/정의 — Applier가 이걸 우선 반영>

## 남은 충돌 (referee 회부?)
- <없음 | R# referee 회부>
```

## 트라이얼 결과 (round-01 QA-09 backtest — 채택 근거)
- **결과: 가치 입증 → selective 채택.** rebuttal이 진짜 빈틈 3개(공허한 트집 아님)를 찾고, counter가 권고를 실제로 개선했다:
  - **R1**: 헤드라인 KPI(`pass^k 절대값`)가 우리 조건상 실측 불가([발표 서사]) → 헤드라인을 **시연 가능한 `캐시 On/Off 갭 Δ` 대조**로 교체 + 분산을 `H_norm` 엔트로피로 정의.
  - **R2**: 유효-결정률 앵커(NQA-B)가 [생략] → ②를 **②-1 대리 게이트(golden 불요·[반영]) + ②-2 정밀(open-issue 강등)**로 분리(blue가 red의 "강등 단독"을 과소 처방이라 되받아 합성 — 한쪽만으론 안 나올 결과).
  - **R3**: 측정 불가인데 importance 상향 → 상향을 R1·R2 해소에 **게이트**.
- applier(단독)는 이 모순을 `사람 결정`으로 떠넘겼었다 → contention이 **구체 설계로 해소**(QA-09 `## 변경 이력` 2026-06-24 contention 블록에 반영 완료).
- 산출물: [`round-01/contention/rebuttal.md`](round-01/contention/rebuttal.md) · [`counter.md`](round-01/contention/counter.md).
