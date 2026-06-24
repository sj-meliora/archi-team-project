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

<!-- 템플릿
## YYYY-MM-DD — 한 줄 요약
- 변경: <기존> → <신규>
- 사유:
- 영향 ID: DP-0004, artifacts/domain-diagram, ...
-->
