# QA 심의(discussion) — 마스터 인덱스

> `context/qa/`의 QA·KPI를 **반복 심의(라운드)**하며 완성도를 끌어올리는 작업의 색인.
> 한 라운드 = red team 비평(review) + blue team 권고(counsel)의 한 묶음.
> 공통 프로토콜·다른 영역(DP 등)은 상위 [`../README.md`](../README.md). 트리거: "QA 디스커션 돌리자".
> updated: 2026-06-24 (round-03 ★ 등급 척도 캘리브레이션 — Council.md §9; round-02 완료 + OI-8 닫힘·QA 재번호)
>
> ⚠️ **2026-06-24 QA 재번호(OI-8 닫힘)**: 신규 QA 정식 편입 — `NQA-A→QA-06`(Security)·`NQA-B→QA-07`(Correctness) 6·7위 삽입, 기존 **QA-06~10 → QA-08~12**(+2), `NQA-C→QA-13`(Cost). **ASR = QA-01~07.** 아래 라운드 이력·verdict 추세표의 번호는 **각 라운드 당시(구 번호)** 기준이다 — 현재 번호 환산은 이 매핑 또는 `context/changelog.md`. (round-NN 폴더는 append-only라 미수정.)

## 관리 체계

- **두 방법론(재생산 스펙)** — 라운드 무관, 이 디렉터리 최상위:
  - [`Reviewer.md`](Reviewer.md) — **red team(비평)**. 3렌즈로 QA·KPI의 약점을 판정.
  - [`Council.md`](Council.md) — **blue team(권고)**. Reviewer 지적을 읽고 근거(레퍼런스)+PoC를 붙여 개선을 권고.
- **1 라운드 = QA 전체 스냅샷에 대한 한 번의 심의**(비평 + 권고).
- **3 렌즈 / 3 seats**: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트.
- **라운드 폴더**: `round-NN/` — **폴더명에 날짜를 넣지 않는다.** 날짜 등 메타는 그 라운드의 종합 보고서에 기록(공의회가 최종 결론을 문서화하듯).
  - `review/` — red team 산출물: `report.md`(결론)·`README.md`(내비)·`QA-0X-*.md`(3렌즈 상세)·`_new-qa-candidates.md`
  - `counsel/` — blue team 산출물: `counsel.md`(권고 결론)·`QA-0X-*.md`(개선안)·PoC 계획 등
  - `contention/` — (ASR 대상만) red↔blue 바운드 1왕복: `rebuttal.md`·`counter.md`·(선택)`referee.md`. 방법론 [`Contention.md`](Contention.md)
  - `applier/` — 반영 산출물: `report.md`(지적별 disposition 보고서 — 다음 라운드 Reviewer 입력, 루프 닫기)
- **append-only**: 지난 라운드는 수정하지 않고 보존한다(추세 비교용). 오타·링크 깨짐만 예외.
- **순환**: 심의(라운드 N) → Stage 2에서 `context/qa/` 개선 → 다음 라운드(N+1)를 새로 떠서 개선 검증 → 반복. 반영 후 Applier가 `round-NN/applier/report.md`로 **지적별 처리를 보고** → 다음 Reviewer가 받아 재검증(red↔applier 루프 닫기).
- **추적성**: 각 라운드 권고가 `context/qa/`에 반영되면 해당 QA 파일 `updated:`와 `changelog.md` 델타로 남기고, 다음 라운드 종합 보고서에서 verdict 변화를 기록.

## 라운드 이력

| 라운드 | 날짜 | 대상 | High | Med | Low | 한 줄 요약 |
|---|---|---|:---:|:---:|:---:|---|
| round-01 [review](round-01/review/report.md) · [counsel](round-01/counsel/counsel.md) · [applier](round-01/applier/report.md) | review 2026-06-23 · counsel·applier 2026-06-24 | QA-01~10 (+NQA-A/B/C 신설) | 4 | 5 | 1 | red team 최초 리뷰(KPI ✕ 3·정의↔KPI 불일치·agentic 리스크 과소대표) → **blue team counsel**: KPI ✕ 3건 전부 측정가능화, C1~C5 응답, NQA-A/B/C 권고 → **applier 반영 완료**: QA-01~10 교정 + NQA-A/B/C 신설(임시 ID). 지적별 처리는 [applier/report.md](round-01/applier/report.md). |
| round-02 [review](round-02/review/report.md) · [counsel](round-02/counsel/counsel.md) | review·counsel 2026-06-24 | QA-01~10 + NQA-A/B/C(13항목 재평가) | 1 | 5 | 7 | round-01 반영 검증: **KPI ✕ 3건 전부 소멸**(QA-01·07·08), **High 4→0**(기존 QA). 잔여 = **신설 QA 미채택(OI-8)으로 인한 교차의존 미닫힘**(QA-07/09↔NQA-B·QA-01/05↔NQA-C) + **KPI-DP 귀속 placeholder(OI-7)**. NQA-B가 세트 닫힘 병목(High). 신규 결함 0건 → **blue team counsel**: 닫힘 확인 7·채택 권장 3·조건부 동반닫힘 3, 신규 KPI 0 — **C3 단일 eval/검증 DP 수렴** 권고. 다음은 OI-8 채택 + DP 디스커션. |
| round-03 [counsel](round-03/counsel/counsel.md) ★rubric | 2026-06-24 | QA-01~13 주 KPI | — | — | — | **등급 척도 캘리브레이션(reviewer-less 변형)**: red-team 없이 팀이 reviewer, Council 3 seats가 각 주 KPI를 ★1~3으로 필드 벤치마크+PoC margin 캘리브레이션 → 13개 QA에 `## 등급 척도` 신설. §측정 하한 8건 재배치(옛값 보존)+상한 캡/조건 추가. 방법론 [`Council.md §9`](Council.md). 후속 OI-9(QAS·glossary 정합). |

## QA별 verdict 추세

표기: ◎ 우수 · ○ 타당 · △ 부분결함 · ✕ 재설계 (형식: `Sound / KPI`). counsel = blue team 권고 stance.

> round-01 counsel은 전 항목 `context/qa/`에 **반영 완료**(2026-06-24). 지적별 처리(반영/발표서사/생략)·다음 라운드 재검증 목록: [round-01/applier/report.md](round-01/applier/report.md).
> round-02 review는 round-01 반영을 검증한 재평가다 — disposition 추적·근거는 [round-02/review/report.md](round-02/review/report.md).

| QA | 속성 | round-01 review | round-01 counsel | **round-02 review** | **round-02 counsel** | sev R01→R02 |
|---|---|---|---|---|---|---|
| QA-01 | Scalability | △ / ✕ | 채택 권장 (scaling efficiency + rate-limit 헤드룸) | **○ / ○** (High 해소·placeholder 소멸; 잔여 DP·NQA-C) | 닫힘 확인 (부하단위·헤드룸단위 Low + NQA-C 동반·rate-limit DP 위임) | High→**Low** |
| QA-02 | Availability | ○ / △ | 채택 권장 (MTTR 분해·무손실 + 외부 LLM 장애) | **○ / ○** (4축 분해 정착; 잔여 외부 degradation DP) | 닫힘 확인 (외부 LLM degradation DP·`drives` 교정 위임) | High→**Low** |
| QA-03 | Controllability | ◎ / △ | 채택 권장 (위반 0건 편입 + runaway cap) | **◎ / △** (C2 해소; ②위반0 측정수단 [발표 서사] 잔존) | 조건부 (②를 NQA-A 공유 red-team 하네스로 실측 전환·동반 닫힘) | Med |
| QA-04 | Observability | ○ / △ | 채택 권장 (trace 완전성 + event-history) | **○ / ○** (span 환원; 잔여 DP-0003 span 보장) | 닫힘 확인 (측정 토대·다수 QA 수급 + DP-0003 span 위임) | Med→**Low** |
| QA-05 | Efficiency | ○ / △ | 채택 권장 (캐시 분리집계, top-line→NQA-C) | **○ / ○** (캐싱 역페널티 제거; top-line NQA-C 부유 의존) | 닫힘 확인 (캐시 적중률 Low + NQA-C 동반·비용 라우팅 DP 위임) | Med→**Low** |
| QA-06 | Reliability (WF 격리) | ○ / ○ | 채택 권장 (쿼터 격리 KPI 추가) | **○ / ○** (C3 닫힘·세트 모범; 잔여 DP-0005 오염격리) | 닫힘 확인 (세트 모범 + QA-01 헤드룸 cross-link·격리 DP 역검토) | Med→**Low** |
| QA-07 | Performance (Agent 시간) | △ / ✕ | 채택 권장 (speedup + 품질 게이트) | **○ / △** (High 해소; **주 KPI가 미채택 NQA-B 게이트 의존**) | 조건부 (**NQA-B 동반 채택이 주 KPI 닫힘 전제** + baseline 프로토콜 Med) | High→**Med** |
| QA-08 | Performance (E2E) | △ / ✕ | 채택 권장 (E2E latency·throughput 추가) | **○ / ○** (mislabel 복구·가장 깔끔; 잔여 DP-0004 산식) | 닫힘 확인 (DP-0004 5% 산식 실측·importance 상향 여지 사람 결정) | High→**Low** |
| QA-09 | Reliability (일관성) | △ / △ | 채택 권장 (캐시우회 pass^k + 유효-결정률) | **○ / △** (contention으로 헤드라인 Δ·②2단·H_norm 해소; ②-2만 NQA-B 의존) | 조건부 (**부분 닫힘** — Δ·②-1·H_norm ○ / ②-2만 NQA-B 의존) | Med |
| QA-10 | Maintainability | ○ / ○ | 채택 권장 (CIS p95 + model 교체축) | **○ / ○** (건강 유지; 잔여 DP-0004/0005 교체내성) | 닫힘 확인 (컴포넌트 경계 정의 전제 명시 + 교체내성 DP 역검토) | Low |

## 신규 QA 권고 추세

| 후보 | 제안 라운드 | round-01 counsel stance | round-02 review verdict | **round-02 counsel stance** | 상태 |
|---|---|---|---|---|---|
| NQA-A Security/Safety | round-01 | 신설·**강력권장** (우선순위 상위) | **○ / △ · Med** — 정식화 권장(ISO Security 앵커); KPI 4/5축 [발표 서사] | **정식 채택 권장(상위 진입)** — red-team 4축 실측 전환 + **QA-03 ② 공유 하네스 동반 닫힘**(C3); 보안 tactic DP 위임 | **정식 채택 → QA-06** (2026-06-24, OI-8 닫힘) |
| NQA-B Correctness/Accuracy | round-01 | 신설·**권장** (QA-07/09 게이트 전제) | **○ / △ · High** — **세트 닫힘 병목**(QA-07·QA-09 ②-2 닫힘이 NQA-B 채택에 달림); eval/검증 DP 부재(OI-7) | **정식 채택 강력 권장(최우선)** — golden+judge 실측 전환 + judge↔인간 일치도 SLI 신설 → **QA-07·QA-09 ②-2 동반 닫힘 단일 트리거** | **정식 채택 → QA-07** (2026-06-24, OI-8 닫힘) |
| NQA-C Cost-economy | round-01 | 신설·권장 (Med, QA-05/01 KPI 흡수) | **○ / △ · Med** — 정식화 권장; **미채택 시 QA-01/05 KPI 부유**(채택이 이양 닫힘 트리거) | **정식 채택 권장(Med)** — $/완료모델 분해 + baseline 가정 슬라이드 명시 → **QA-01 활용률·QA-05 top-line 부유 해소**(C2) | **정식 채택 → QA-13** (2026-06-24, OI-8 닫힘) |
