# 변경 이력 (Changelog)

> category: meta
> 최신 pptx를 `../source/`에 넣은 뒤, "무엇이 → 무엇으로, 왜, 영향받는 ID"만 append.
> 매번 전체를 다시 설명하지 않고 델타만 기록한다.

## 2026-06-20 — context 구조화
- `team17_context_snapshot.md`(단일 스냅샷)를 방법론 개념 단위(requirements/qa/dp/artifacts + flat)로 분해.
- 추출 기준: SW_Architect_17조_-_팀_과제_.pptx (45p), 추출 시점 2026-06-19.
- 정합성 메모 6건 → open-issues.md(OI-1~6)로 이관.

## 2026-06-23 — QA 우선순위 재정렬 + QA·QAS 2자리 번호 전환
- 변경: QA 우선순위 재정렬 — QA-01 Scalability / 02 Availability / 03 Controllability / 04 Observability / 05 Efficiency (이전 1위 Efficiency가 5위로). 06~10은 속성 불변, 자릿수만 2자리.
- 변경: QA·QAS ID 포맷 4자리→**2자리**(`QA-0002`→`QA-01`). FR/C/DP는 4자리 유지(추후 전환).
- 사유: 발표 슬라이드·팀 산출물이 참조할 canonical 번호를 팀 우선순위와 일치시키기 위함. 자릿수 혼용 방지로 QA·QAS 10개 일괄 2자리.
- 영향 ID: qa/ 전체(QA·QAS 20파일 리네임), DP-0001~0005, dp4/*, FR-0001~0005, C-0002, overview, INDEX, glossary, open-issues(OI-1).

## 2026-06-24 — QA 디스커션 round-02 반영 (수렴 라운드)
- 변경(QA-03 contention): ② `허용 범위 외 액션 통과 = 0` → **②-1 권한외 액션 차단율**(룰 체커·admission 게이트·"주입→차단/안전정지 latency 분포"·OI-8 무관 현 조건 닫힘) + **②-2 적대적 위반0 acceptance**(OWASP LLM Top-10 매핑 풀세트·NQA-A 채택 OI-8 의존 open-issue로 강등) 2단 분리. "동반 닫힘" 무조건 표현 → 조건부("②-2·NQA-A는 OI-8 채택 시 동반 닫힘; ②-1은 선닫힘") 정정. 헤드라인 ① 유지(교체 불요) + QA-09 비대칭 사유 명시. QAS-03 Response·Measure 동기화. (QA-03 파일 끝 `</content></invoke>` 잔여 아티팩트 제거.)
- 변경(Low cross-link 4건): QA-01 rate-limit 헤드룸 측정단위(계정전역 TPM/RPM·큐별 token-bucket 배분, QA-06 cross-link) / QA-05 캐시 적중률 보조 KPI 노출 / QA-06 외부 rate-limit 계정전역 silent cap 명문화(QA-01 cross-link) / QA-10 CIS p95 컴포넌트 경계 정의 선결 명문화.
- 변경(open-issues): OI-8에 "QA-03 ②-2 ↔ NQA-A 공유 red-team 하네스 채택" 교차 의존 등록 + OI-7에 동일 교차 의존·eval/검증 서브시스템 DP 귀속 갱신.
- 사유: round-02 QA 디스커션(수렴 단계) — actionable [반영]은 QA-03 contention 델타 + Low cross-link뿐. 닫힘 7건(QA-01·02·04·05·06·08·10)은 round-01 반영분 재확인(신규 본문 대수정 없음). NQA-A/B/C 정식 채택·번호 재정렬·예시값 확정은 [이월](OI-8 사람 결정), DP 귀속은 [이월](OI-7 DP 디스커션).
- 영향 ID: QA-03, QAS-03, QA-01, QA-05, QA-06, QA-10, open-issues(OI-7·OI-8).

## 2026-06-24 — 신규 QA 정식 편입 + 번호 재정렬 (OI-8 닫힘)
- 변경(정식 채택): round-01 발굴 신규 QA 3종(NQA-A/B/C, 임시 ID)을 **정식 QA로 편입**(팀 결정). 6섹션 본문·짝 QAS는 기존 작성분 유지, ID만 확정.
- 변경(번호 재정렬) — **우선순위 6·7위 삽입, 기존 06~10 +2 시프트, C는 말미 유지**:

  | 구 ID | → 새 ID | 속성 |
  |---|---|---|
  | NQA-A | **QA-06** | Security/Safety |
  | NQA-B | **QA-07** | Correctness |
  | QA-06 | QA-08 | Reliability(Workflow) |
  | QA-07 | QA-09 | Performance(Agent수행시간) |
  | QA-08 | QA-10 | Performance(E2E) |
  | QA-09 | QA-11 | Reliability(Agent일관성) |
  | QA-10 | QA-12 | Maintainability |
  | NQA-C | **QA-13** | Cost-economy |

  (QA-01~05 불변. 짝 QAS-NN·QAS-A/B/C도 동일 매핑.)
- 변경(ASR 재선정, DP 생성 동인·팀 판단): ASR = **QA-01~07**(기존 5 + Security·Correctness). 기존 Reliability-WF(→QA-08)·Performance-Agent(→QA-09)는 ASR 제외. `discussion/qa/Contention.md`·discussion 스킬 갱신.
- 반영: context/qa/ 16파일 리네임 + 전 context(qa·dp·requirements·INDEX·glossary·open-issues) cross-ref 일괄 동기화(경로 링크·frozen discussion 스냅샷은 보존). 각 신규 QA 변경이력 + frontmatter updates. OI-8 [x] 닫힘.
- 사유: round-02 counsel이 NQA-A/B/C 정식 채택 권장(NQA-B 최우선)했고, 팀이 6·7위 삽입으로 확정. 발표 우선순위 추가 상향(Security를 더 위로)은 팀 논의 후 별도.
- ⚠️ 위 2026-06-24 round-02 반영 항목 및 `discussion/qa/round-01·02/`(append-only 스냅샷)는 **재번호 이전 번호**로 서술됨 — 현재 번호는 이 매핑표로 환산.
- 영향 ID: qa/ 전체(QA·QAS 16파일 리네임 + cross-ref), dp/(DP-0001/0002/0004/0005·dp4/*), requirements(FR-0001~0003), INDEX, glossary, open-issues(OI-1·OI-7·OI-8), discussion(qa/README·SKILL·Contention).

## 2026-06-24 — QA 등급 척도(★ rubric) 캘리브레이션 (round-03)
- 변경: 13개 QA 전체에 `## 등급 척도 (★ rubric — ATAM trade-off용)` 섹션 신설. ATAM의 QA 간 trade-off(동일 조건 두 아키텍처 안 비교 → ★ 많은 안 채택)를 위해 각 주 KPI를 ★1~3(상/중/하)으로 급간화. KPI 합격선=★☆☆ 진입선, ★★☆/★★★는 Council 3 seats가 **실제 필드 벤치마크 + PoC margin**으로 캘리브레이션(reviewer-less 변형 — 팀이 reviewer 자리). 방법론은 `discussion/qa/Council.md §9`에 codify.
- 변경(§측정 하한 재배치 — 규칙2, 옛값 보존 트레이스 부착): 필드 기준 비현실적으로 높거나(★★★ 사문화) 보수적인(★☆☆ 사문화) 하한을 현실 대역으로 재배치 —
  - QA-01 scaling efficiency `≥0.8 → ≥0.70` (USL 회귀 우수도 ~0.72)
  - QA-02 재기동 `≤1분 → ≤4분` (손실=0·멱등100%는 게이트 불변)
  - QA-06 injection 차단율 `≥95% → ≥70%` (+FPR≤1% 조건; 헤드라인 권한상승·범위외배포=0은 Constraint성 게이트 불변)
  - QA-08 타WF latency 증가 `≤10% → ≤25%` (10%는 ★★☆로)
  - QA-09 speedup `≥3배 → 1×~3배` (3배는 ★★☆로; headless 완전자율이라 협업형 3배 상회 가능)
  - QA-10 E2E latency `≤6h → p95 ≤24h` (6h는 ★★☆로; 배치 ML 관례 6~24h 대비 6h는 보수적)
  - QA-11 보조 pass^k `≥70% → 25/40/60%(★☆☆/★★☆/★★★)` (τ-bench 대비 70% 비현실)
  - QA-13 절감률 `≥70% → ≥50%` (70%는 ★★☆로) + `$/완료모델 ≤$5` 주KPI→게이트(pass/fail)
- 변경(§측정 상한 캡·조건 추가): QA-03 graceful stop+롤백 `≤30초`를 ★★☆ 상한으로 흡수(★☆☆=>30초+hard-kill 폴백) / QA-04 ★★★ trace 완전성 100% 미만(≥98%) 캡(CoT 비공개) / QA-07 정답률 `≥90%`에 만점 캡(★★★ 92~98%, 100%=난이도부족 불합격)+judge↔인간 일치도 κ 조건 / QA-05 신규토큰 tier 정식화(≤6k 일반·≤8k 복합)+캐시적중≥85% 조건. QA-12 보정 없음(CIS p95≤3 유지).
- 변경(2-index → main+조건, 규칙4): QA-01 헤드룸≥20% / QA-03 ack≤5초 / QA-04 안전액션100% / QA-08 중단율≤1%·쿼터침범0 / QA-10 throughput≥50·전달무결성100% / QA-11 H_norm·②-1·Δ시연 / QA-13 baseline·단가·캐시 고정 등을 게이트/조건으로 분리.
- 사유: ATAM trade-off의 ★ 비교를 필드 현실에 정착(임의 경계 방지). council 근거는 실제 웹 출처(USL/HPC, Unit42·arXiv injection, τ-bench, 배치 ML SLA, prompt caching 등)만 사용·각 표에 URL 명시. 경계는 전부 예시값 — PoC 실측으로 확정.
- 후속(OI-9): §측정 하한 보정 8건이 짝 QAS-* Measure·glossary 수치 표기와 정합하는지 점검(이번 미반영).
- 영향 ID: QA-01~13 전체, discussion/qa/Council.md(§9 신설), open-issues(OI-9 신설).

## 2026-06-24 — OI-9 닫힘 (★ 등급 척도 ↔ QAS 수치 동기화)
- 변경: round-03 §측정 하한 보정을 짝 QAS 9개(QAS-01·02·03·06·08·09·10·11·13)의 `응답 측정(Measure)` 행에 동기화 — scaling efficiency 0.8→0.70 / 재기동 1분→4분 / injection 95→70% / 타WF latency 10→25% / speedup 3배→1×~3배 / E2E 6h→24h / pass^k 70→25·40·60% / 절감률 70→50%·$/모델→게이트 / graceful stop+롤백 ★급간화. 각 QAS footnote 예시값 목록·`updated:`·`★ 급간 QA-XX` 포인터 갱신.
- 불변: 게이트(0건/100% 절대형 — in-flight 손실=0·권한외배포=0·HITL 100%·서명 100% 등)는 그대로. QAS-04·07은 합격 하한 불변이라 제외. glossary는 수치 미포함이라 동기화 불요.
- 사유: OI-9(QA 본문 보정 ↔ QAS 정합) 닫음. 경계는 예시값 — PoC 실측 시 확정.
- 영향 ID: QAS-01·02·03·06·08·09·10·11·13, open-issues(OI-9 [x]).

## 2026-06-25 — QA 등급 척도(★ rubric) 근거 보강 (round-04)
- 변경(★ 급간 수치 보정 — 3건): **QA-11 ★★★ pass^5 `≥60% → ≥45%` 하향**(★★☆ `40~60 → 35~45`; voting→pass^k 메커니즘 1차출처 반증 — voting=pass@k 1답 정확도, pass^k=전 시행 성공으로 별개) + **QA-06 ★★☆ `80~90 → 85~90%`**(margin 단일 Y=5%p 규칙화; ★☆☆ 상한 동반 `80→85%`)·**FPR ≤1% 전 급간 게이트화** + **QA-07 ★★★ `92~98 → [93,99]`·★★☆ `90~92 → [91,93)`·100%만 불합격**((98,100) 공백 제거).
- 변경(경계 표기 구간화 — ≥/> 모호 제거): **QA-04 `[95,97)/[97,98)/[98,100)`** + 완전성=존재 AND 비-truncation / **QA-12 CIS `=1/=2/=3 → ≤1 / 1<p95≤2 / 2<p95≤3`**(분수 p95·동률 변별).
- 변경(보조 별점 축 신설 — C3 이중 축, main=모델추정·보조=실측 가능): **QA-11 ②-1 룰 게이트 통과율(95/97/99%)** · **QA-07 rework율(≤5/15/30%)** · **QA-12 prompt/tool-def 수정 수(≤1/=2/=3)** · **QA-13 재시도 오버헤드 비율(≤10/25/40%)**.
- 변경(인용 정정 — C4 웹검증): **QA-09 InfEngine(arXiv 2602.18985)** — round-03 인용 `8.6~22.7×·assistant-type`은 원문에 없는 가공 수치 → 원문 `21× faster·92.7% pass·적외선 복사 컴퓨팅`으로 정정 + 도메인 불일치 apples silent cap. **★★★ ≥8배 방향은 유지**(21×가 지지).
- 변경(근거 보강 — 급간 수치 불변): margin 규칙화(QA-01 천장거리 비례·QA-08 tail+폭주 흡수 5%·QA-10 compute critical path 하한 2h·QA-13 margin 분포 두께 차등) + apples silent cap(QA-01·05·08·09·13 = 인용 다른 셋업/도메인·우리 PoC로 확정; QA-06 직접/간접 injection 1차출처 확인) + 정의 고정(QA-02 "재기동"=lease 만료→체크포인트 재개) + QA-03 ★★★ 15초 근거 다양화·롤백 외부 정합성 / QA-05 적중률 조건·tier margin·단일출처 / QA-07 judge κ 도메인 재측정.
- 변경(짝 QAS 동기화): 수치 보정 QAS 3개(QAS-06·07·11)는 Measure 동기화, 나머지 10개는 ★ 급간 참조·보조 별점 축·근거 보강 노트만(급간 수치 불변). QAS-07엔 "★ 급간=QA-07 등급 척도" 참조 신규 추가.
- 변경(open-issues): OI-9에 round-04 재보정 5곳 정합 항목 append(특히 QA-13 하한 보정 70→50%의 OI-9 등록 확인 — round-03 "범위 외" 표현 해소).
- 사유: round-04(★ 등급 척도 red-team 검증 → Council 근거 보강). review 4축(C1 apples / C2 margin / C3 main축 실측불가 / C4 출처 신뢰성)에 레퍼런스+PoC로 응답. `_feasibility-filter.md` 없는 수렴/캘리브레이션 라운드 — counsel 채택 권고표를 등급 기준으로 적용. 경계는 전부 예시값 — PoC 실측으로 확정.
- 영향 ID: QA-01~13 전체 + QAS-06·07·11(Measure) + QAS-01·02·03·04·05·08·09·10·12·13(노트), open-issues(OI-9 round-04 항목), changelog.

## 2026-06-26 — QA-06 Security/Safety → 제약 C-03 이관 (OI-10 닫힘)
- 변경(이관): `QA-06 Security/Safety` → **제약 `C-03`**(`context/requirements/C-03-agent-security-safety.md` 신설, QA-06 6섹션 콘텐츠를 제약 형식으로 이관). QA-06·QAS-06 파일 삭제.
- 사유: 헤드라인 `권한 상승·범위 외 배포 = 0`이 1·2건을 허용 못 하는 **0건 절대형(pass/fail 게이트)** 이라 ★ 급간화 불가 → ATAM 대안 변별용 QA가 아니라 제약(Constraint)이 적정(팀 의논). round-03 ★ rubric이 이미 "헤드라인 = Constraint 성격"을 지적했던 것을 전체 이관으로 확정.
- 변경(gradable 보존): `injection 차단율 ≥70%·FPR ≤1%`는 C-03 **측정 임계**로 승계. 종전 ★ rubric(★1~3 등급표)은 폐기.
- 변경(번호): **QA-07~13 현 위치 유지**(QA-07을 06으로 당기지 않음 — QA-06 번호 공석). 짝 **QAS-06 제거**.
- 변경(ASR 축소): ASR = **QA-01~05, QA-07**(종전 QA-01~07에서 Security 제외). discussion 방법론 스펙(`Contention.md`·`README.md`) ASR 목록 갱신.
- 변경(cross-ref): INDEX·glossary·open-issues(OI-7 갱신·OI-10 신설) + QA-03·QAS-03·QA-04·QAS-04·QA-07·QA-11·QA-13 라이브 참조를 "QA-06" → "C-03"으로 redirect. (round-NN append-only 스냅샷·각 QA 변경이력의 과거 서술은 당시 번호 보존.)
- 비고: 같은 날 **C-0001/0002 → C-01/C-02 2자리 통일**도 수행(아래 별도 항목) — 이제 C 전부 2자리.
- 영향 ID: C-03(신규), QA-06·QAS-06(삭제), QA-03, QAS-03, QA-04, QAS-04, QA-07, QA-11, QA-13, INDEX, glossary, open-issues(OI-7·OI-10), discussion(qa/Contention·README).

## 2026-06-26 — Constraint ID 2자리 통일 (C-0001/0002 → C-01/C-02)
- 변경: `C-0001 표준 패키징` → **`C-01`**, `C-0002 배포 이식성` → **`C-02`**(파일 리네임 + heading + 라이브 cross-ref). C-03 신규와 자릿수 정합.
- 사유: CLAUDE.md "FR/C/DP도 2자리 통일 예정"의 C 부분 이행 — C-03 신설로 생긴 4자리/2자리 혼용 해소.
- 반영: requirements/ 2파일 리네임 + 라이브 참조(INDEX·dp/DP-0002·DP-0003·DP-0004·DP-01·dp4/approaches A3~A8) 일괄 치환. **round-NN append-only 스냅샷(discussion/dp/round-01/*)·과거 changelog 영향 ID(2026-06-23 항목)는 당시 표기 보존.**
- 비고: ID는 영구 고정 원칙이나 자릿수 포맷 전환은 예외(내용·의미 불변, zero-pad만). FR/DP 2자리 전환은 추후 별도.
- 영향 ID: C-01, C-02(리네임), INDEX, DP-0002, DP-0003, DP-0004, DP-01, dp4/approaches(A3·A4·A5·A6·A7·A8), changelog, open-issues(OI-10 잔여).

## 2026-06-27 — ASR 선정·우선순위 SSoT 분리 (context/asr.md 신설, OI-11)
- 변경: ASR(=DP 생성 동인) 목록·우선순위를 **`context/asr.md`로 분리(SSoT)**. 현재 ASR = QA-01~05, QA-07(우선순위순).
- 사유: 종전 `QA 번호 = 발표 우선순위`라 우선순위 변경 시 **QA id 재번호 → 전 cross-ref 출렁임**. QA id를 **고정 식별자**로 두고 우선순위는 asr.md에서만 관리.
- 반영: asr.md 신설 + **하드코딩된 ASR 목록을 asr.md 참조로 전환** — INDEX(ID 체계·qa 헤더)·CLAUDE.md(번호 정책)·discussion/dp(Reviewer·Council)·discussion/qa(Contention·README)·`.claude/skills/discussion/SKILL.md`·context/dp/DP-01. **우선순위가 바뀌어도 QA 재번호 안 함.**
- 비고: round-NN append-only 스냅샷·각 변경이력·`discussion/dp/notes/dp2-drop-rationale.md`(드랍 결정 기록)의 과거 'QA-01~07' 서술은 당시 기준으로 보존.
- 영향 ID: asr.md(신규), INDEX, CLAUDE.md, discussion(dp/Reviewer·Council, qa/Contention·README), SKILL.md, DP-01, changelog, open-issues(OI-11).

<!-- 템플릿
## YYYY-MM-DD — 한 줄 요약
- 변경: <기존> → <신규>
- 사유:
- 영향 ID: DP-0004, artifacts/domain-diagram, ...
-->
