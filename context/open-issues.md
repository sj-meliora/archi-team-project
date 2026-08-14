# 정합성 / 미해결 결정 트래커 (Open Issues)

> category: meta | updated: 2026-06-26 (OI-10 신설·닫힘 — QA-06 Security/Safety → 제약 C-03 이관, ASR 축소) · 2026-06-25 (OI-9 round-04 재보정 항목 append — ★ 등급 척도 근거 보강 라운드; QA-13 하한 보정 OI-9 등록 확인 해소) · 2026-06-24 (OI-8 닫힘 — NQA-A/B/C → QA-06/07/13 정식 편입·재번호; round-02 contention 반영; OI-9 신설·닫힘 — ★ rubric 캘리브레이션 + QAS 동기화)
> 자료 변동으로 생긴 불일치 + 미확정 사항. 확정되면 해당 개념 파일에 반영하고 여기서 닫는다([x]).

## [ ] OI-1 QA 번호 불일치
- 슬라이드 11/13/25~28이 같은 QA01~04에 서로 다른 속성을 부여.
- **결정**: canonical = 팀 합의 우선순위 기준 (2026-06-23 재정렬, QA·QAS 2자리). glossary.md에 원본 매핑 보존.
- **2026-06-23 재정렬**: QA-01 Scalability / 02 Availability / 03 Controllability / 04 Observability / 05 Efficiency (이전 1위 Efficiency→5위). 06~10 불변.
- **2026-06-24 재번호(OI-8 닫힘)**: 신규 QA 정식 편입 — NQA-A→**QA-06 Security**, NQA-B→**QA-07 Correctness**(6·7위 삽입), 기존 QA-06~10 +2 시프트(→QA-08~12), NQA-C→**QA-13 Cost**. canonical = **QA-01~13**.
- **남은 작업**: 발표 슬라이드 전체에서 canonical(2자리, QA-01~13) 번호로 통일. FR/C/DP도 2자리 전환 예정.

## [x] OI-2 구 DP-0001 1안 '단점' 칸 표기 오류 — 2026-06-27 무효(파일 삭제)
- Per-Node Agent(1안) 단점에 2안의 장점이 복붙됐던 오류. **구 DP-0001 삭제(2026-06-27)로 무효화** — 배치/풀 결정 재귀속은 OI-12.

## [ ] OI-3 DP-0004 대안 라벨 오류
- 헤더가 구 DP-0002 라벨(Hierarchical/Decentralized) 복붙. 실제 내용은 "타입별 서버 풀 vs 노드당 Workflow 인스턴스".
- **결정**: 내용 기준 라벨로 정리 (DP-0004 파일에 반영 완료 시 체크).

## [ ] OI-4 FR-0005 범위 미확정
- "시스템 운영·제어"의 Agent 특화 여부·범위 미정.
- **남은 작업**: PM과 범위 확정.

## [ ] OI-5 Domain Diagram 2개 버전 공존
- 슬라이드 15·16 vs 35·36에서 컴포넌트 배치(JIRA Manager 위치 등) 상이.
- **남은 작업**: 최신본 확정 후 artifacts/domain-diagram에 단일화.

## [ ] OI-6 As-Is/To-Be 시나리오 2개 버전
- 슬라이드 9 vs 34: Compiler 시작 vs Quantizer 시작으로 서술 상이.
- **남은 작업**: 기준 시나리오 확정.

## [ ] OI-7 DP 역검토 — round-01 QA 디스커션에서 드러난 DP 미명시 차원
- QA 디스커션 round-01 반영 중, 여러 DP가 새 KPI가 요구하는 차원을 명시하지 않음이 드러남(원본 DP는 고치지 않고 트래킹만).
- **구 DP-0001(동적 풀)·DP-0004(타입별 scale-out)**: rate-limit·admission control 차원 미명시 (QA-01 반영 시 발견). *(구 DP-0001 부분은 삭제·재귀속 보류 OI-12로 이관.)*
- **DP-01(Standby)·DP-0003(격리/모니터링)**: "외부 LLM degradation"(outage/429) 시나리오 미명시 — backoff·폴백 tactic 보강 필요 (QA-02 반영 시 발견).
- **DP-01(제어지점)·DP-0003(권한 게이트)**: **runaway cap**(max iter·token·wall-clock)과 **graceful 정지+롤백**(협조적 취소 vs hard-kill) tactic 미명시 (QA-03 반영 시 발견). DP-01 1안 HITL·DP-0003 1안 allowlist는 있으나 cap·정지 의미 분리는 후보 대안/ATAM에 없음.
- **[해소·무효 2026-06-27] 구 DP-0001 ↔ QA-02 링크 불일치**: 구 DP-0001 삭제로 `drives` 불일치 무효 — 배치/풀 결정 재귀속은 OI-12.
- **DP-0003(실시간 모니터링)**: span 수준 trace(prompt+context+tool I/O+model+token+ts) 보장 여부 미명시 — "규칙 기반 한정 적용" 시 어디까지 span을 남기는지 명시 필요 (QA-04 반영 시 발견).
- **구 DP-0001(2안 동적 풀)**: 작업별 최적 agent 선택이 **토큰 기준인지 비용 기준인지** 미명시 — 비용 기준 정렬 권고 (QA-05 반영 시 발견). *(삭제·재귀속 보류 OI-12 — 재귀속 DP에서 반영.)*
- **구 DP-0001(1안 즉시 실행)**: latency만 보고 **품질 게이트(first-pass 성공)와 무연결** — "빠르게 틀리기" 차단 tactic 미명시 (QA-09 반영 시 발견). *(삭제·재귀속 보류 OI-12.)*
- **DP-0004/0005**: E2E latency 책임 및 **결정 산식(4단계×20GB÷대역폭 5% budget → A5 vs A8)의 실측 의존**, prompt/모델 교체 내성(workflow 버저닝·계약 분리) 미명시 (QA-10·QA-12 반영 시 발견).
- **신규 QA 교차 의존(QA-07 전제)**: QA-09 first-pass 품질 게이트·QA-11 **②-2 정밀 유효-결정률**이 **QA-07(Correctness) golden 게이트에 의존** — QA-07 신설과 함께 가야 닫힘. (QA-11 contention 반영으로 ②-1 대리 유효-결정률은 golden 불요 룰 게이트로 분리·해소됨 — ②-2만 잔존 의존.)
- **교차 의존(C-03 전제) — round-02 QA-03 contention 등록**: QA-03 **②-2 적대적 위반0(acceptance)**가 **C-03(보안·안전 제약) 공유 red-team 하네스 풀커버리지(OWASP LLM Top-10 매핑)에 의존** — 하네스(eval 서브시스템 DP, 아래) 구축과 함께 가야 닫힘(QA-11 ②-2 ↔ QA-07와 동일 구조). (QA-03 contention 반영으로 ②-1 권한외 차단율은 룰 체커로 분리·**현 조건 선닫힘** — ②-2만 잔존 의존.) eval/검증 서브시스템 DP 신설(아래)이 ②-2 하네스를 받침. *(원 QA-06 채택 의존은 2026-06-26 C-03 이관(OI-10)으로 채택이 완료돼, 잔존 의존은 하네스 구축뿐.)*
- **eval/검증 서브시스템 = 신규 DP 후보**: C-03 red-team 하네스·QA-07 golden+judge 하네스가 **어떤 DP에도 없는 신규 인프라** — DP 디스커션에서 "eval/검증 서브시스템" DP 신설 검토 (C-03/QA-07/QA-03 공유 자산).
- **DP-01/0003 보안 tactic 미명시**: 공급망 artifact 서명·secrets 관리·injection 가드레일이 후보 대안/ATAM에 없음 (C-03 — 구 QA-06 — 의 핵심 게이트인데 DP에 미명시).
- **`related-dp` 추가**: QA-05에 DP-0005, QA-09에 DP-0004 추가함(반영 완료). 역방향(DP의 `drives`)과 정합 확인 필요.
- **남은 작업**: 각 DP에 해당 tactic(rate-limit headroom·admission control·외부 의존성 backoff·폴백·runaway cap·graceful 정지·span trace·품질 게이트·workflow 버저닝)을 후보 대안/ATAM에 명시할지 + 구 DP-0001 배치/풀 재귀속(OI-12) + QA-07 교차 의존을 DP 디스커션에서 역검토.

## [x] OI-8 신규 QA 정식 승격·번호 재정렬 — 2026-06-24 닫힘
- **결정(2026-06-24 팀)**: 신규 QA 3종 **정식 채택**. 번호 — NQA-A→**QA-06 Security/Safety(H)**, NQA-B→**QA-07 Correctness(H)**(6·7위 삽입), NQA-C→**QA-13 Cost-economy(M)**(말미 유지). 기존 QA-06~10은 +2 시프트(→QA-08~12). 우선순위 추가 상향(Security를 더 위로)은 팀 논의 후 별도(번호와 분리).
- **반영 완료**: 파일 리네임(QA/QAS-06·07·13) + frontmatter id·heading + 전 QA·QAS·INDEX·glossary·open-issues·DP cross-ref 동기화 + 각 신규 QA 변경이력. 전체 매핑은 `changelog.md` 2026-06-24 항목.
- **ASR 재선정(팀 판단, DP 생성 동인)**: **QA-01~07**(기존 5 + Security·Correctness). 기존 Reliability-WF(→QA-08)·Performance-Agent(→QA-09)는 ASR에서 빠짐. discussion 스킬·Contention 방법론 갱신.
- **연결 확정**: QA-05 top-line·QA-01 활용률 → QA-13 이양 / QA-09·QA-11 ②-2 게이트 ↔ QA-07 양방향 / QA-03 ②-2 ↔ QA-06 red-team 하네스(round-02 contention; ②-1 선닫힘, ②-2만 QA-06 의존).
- **잔여(OI-7로 이관)**: 검증 PoC 실측 전환·eval/검증 서브시스템 DP 신설은 DP 디스커션(OI-7). 발표 우선순위 추가 상향은 팀 논의.

## [x] OI-9 ★ 등급 척도 캘리브레이션의 하한 보정 ↔ QAS·glossary 정합 — 2026-06-24 닫힘
- round-03 ★ rubric 캘리브레이션(2026-06-24)으로 8개 QA의 §측정 합격 하한을 재배치(QA-01 0.8→0.70 / QA-02 1분→4분 / QA-06 95→70% / QA-08 10→25% / QA-09 3배→1× / QA-10 6h→24h / QA-11 pass^k 70→25·40·60% / QA-13 70→50%·$/모델→게이트) + QA-03(graceful stop 재배치)·QA-04(★★★ 상한 캡)·QA-07(만점 캡+judge κ 조건) 상한/조건 추가. QA 본문엔 옛값 보존 트레이스로 반영.
- **2026-06-24 닫힘**: 짝 QAS 9개(QAS-01·02·03·06·08·09·10·11·13) Measure 수치를 보정 합격 하한으로 동기화(게이트 불변)+footnote·`updated:`·★ 급간 포인터. **glossary는 수치 미포함이라 동기화 불요.** QAS-04·07은 합격 하한 불변이라 제외. 등급 경계는 전부 예시값 — PoC 실측 시 최종 확정. (`changelog.md` 2026-06-24 OI-9 항목.)
- **2026-06-25 round-04 재보정(★ 등급 척도 근거 보강 라운드 — 5곳 정합 재점검)**: round-04 디스커션이 ★ 급간을 추가 보정 → §측정·등급표·변경이력·짝 QAS Measure·glossary 5곳 재동기화 완료.
  - **★ 급간 수치 보정 3건**(QAS Measure 동기화 대상): **QA-11 ★★★ pass^5 `60→45%`**(★★☆ `40~60→35~45`; voting→pass^k 메커니즘 반증) → QAS-11 Measure `25/35/45%` + ②-1 보조축 / **QA-06 ★★☆ `80~90→85~90%`·FPR 전급간 게이트** → QAS-06 Measure FPR 게이트 범위·★ 급간 참조 / **QA-07 ★★★ `92~98→[93,99]`·100%만 불합격** → QAS-07 Measure ★ 급간 참조(하한 ≥90% 불변).
  - **표기 구간화·보조 별점 축·근거 보강 10건**(하한/급간 수치 불변 — QAS는 노트만): QA-04 경계 `[95,97)/[97,98)/[98,100)`+비-truncation / QA-12 CIS 구간화·prompt 보조축 / QA-09 InfEngine 인용 정정 / QA-13 재시도 보조축·★★★ 75% 도메인무관 재근거 / QA-01·02·03·05·08·10 margin·apples·정의 silent cap.
  - **QA-13 하한 보정(70→50%) OI-9 등록 확인(round-04 권고 4)**: round-03 변경이력의 "open-issues 정합 점검(범위 외)" 표현으로 QA-13만 미등록 의심이었으나(QA-08~10은 등록 명시), **본 항목으로 명시 등록 — 해소.**
  - **glossary**: round-04도 수치 미포함이라 동기화 불요(★ 급간·KPL 수치는 glossary에 없음 — QA번호↔슬라이드 매핑만). **glossary 충돌 0 재확인.**
  - 등급 경계는 전부 예시값 — PoC 실측 시 최종 확정. (`changelog.md` 2026-06-25 round-04 항목 + 각 QA `## 변경 이력` 2026-06-25 블록.)

## [x] OI-10 QA-06 Security/Safety → 제약 C-03 이관 — 2026-06-26 닫힘
- **결정(2026-06-26 팀)**: `QA-06 Security/Safety`를 **제약 `C-03`으로 이관**. 사유 = 헤드라인 `권한 상승·범위 외 배포 = 0`이 1·2건을 허용 못 하는 **0건 절대형(pass/fail 게이트)** 이라 ★ 급간화 불가 → ATAM 대안 변별용 QA가 아니라 **모든 대안이 무조건 통과할 제약(Constraint)** 이 적정. (round-03 ★ rubric이 이미 "헤드라인 = Constraint 성격"을 지적했던 것을 팀이 전체 이관으로 확정.)
- **gradable 보존**: `injection 차단율 ≥70%·FPR ≤1%`는 C-03 **측정 임계**로 보존. 종전 ★ rubric(★1~3 등급표)은 폐기(제약엔 ATAM ★ 비교 미적용).
- **번호 정책**: **QA-07~13 현 위치 유지**(QA-07을 06으로 당기지 않음 — QA-06 번호 공석/은퇴). 짝 **QAS-06 제거**(제약엔 QAS 없음).
- **ASR 축소(DP 생성 동인)**: ASR = **QA-01~05, QA-07**(종전 QA-01~07에서 Security 제외). C-03은 제약으로서 DP-0002/0003을 구동하나 ASR 목록엔 미포함.
- **반영 완료**: `context/requirements/C-03-agent-security-safety.md` 신설(QA-06 6섹션 콘텐츠 제약 형식 이관) + QA-06·QAS-06 파일 삭제 + INDEX·glossary·open-issues(OI-7)·changelog + QA-03·QAS-03·QA-04·QAS-04·QA-07·QA-11·QA-13 라이브 cross-ref C-03 redirect + discussion 방법론 스펙(Contention.md·README.md) ASR 목록 갱신. (round-NN append-only 스냅샷·각 QA 변경이력의 과거 서술은 당시 번호 보존.)
- **잔여**: 발표 우선순위 추가 상향(Security를 더 위로)은 무의미해짐(제약은 우선순위 번호 밖). DP-0002/0003 보안 tactic 미명시·eval 서브시스템 DP 신설은 OI-7로 계속 트래킹. **C 2자리 통일 완료(2026-06-26)**: C-0001/0002 → C-01/C-02 리네임 — 이제 C 전부 2자리(C-01·02·03). (FR/DP 2자리 전환은 추후 별도.)

## [x] OI-11 ASR 선정·우선순위 SSoT 분리(asr.md) — 2026-06-27 닫힘
- **결정(2026-06-27)**: ASR(=DP 생성 동인) 목록·우선순위를 **`context/asr.md`로 분리**. **QA id는 고정 식별자**(우선순위가 바뀌어도 재번호하지 않는다).
- **사유**: 종전 `QA 번호 = 발표 우선순위` 결합이 우선순위 변경 시 QA 재번호 → 전 cross-ref·문서 출렁임을 유발(OI-8·OI-10이 그 사례). id와 우선순위를 분리해 churn 제거.
- **반영**: `context/asr.md` 신설(현재 ASR = QA-01~05·QA-07, 우선순위순) + INDEX(ID 체계·qa 헤더)·CLAUDE.md(번호 정책) 갱신 + QA/DP 디스커션 스펙·skill이 ASR 목록을 **하드코딩 대신 asr.md 참조**(discussion/dp Reviewer·Council, discussion/qa Contention·README, SKILL.md, context/dp/DP-01).
- **유지보수**: 우선순위·ASR 변경은 **asr.md만** 수정(+ 근거는 여기 open-issues, 델타는 changelog). QA 본문·스펙의 목록을 일일이 고칠 필요 없음.
- **후속**: FR/DP 2자리 통일은 추후 별도(OI-10 잔여와 동일).

## [ ] OI-12 구 DP-0001(에이전트 배치/풀) 결정 재귀속 보류
- **맥락(2026-06-27)**: 구 DP-0001(Per-Node vs Dynamic Agent Pool = 에이전트 배치/풀)·구 DP-0002 삭제. DP-0002(위상·Standby)는 **DP-01로 승계**됐으나, **구 DP-0001의 "동적 풀·즉시 실행" 결정은 DP-01(제어평면)이 다루지 않아** 어느 DP에도 귀속되지 않음(orphan). 팀 결정으로 **재귀속을 보류**하고 추적만 한다.
- **인용 중인 곳(보류 포인터)**: QA-01(scaling efficiency·큐 p95)·QA-05(작업별 최적 agent·warm pool)·QA-09(즉시 실행 speedup)·QA-13(비용 라우팅)의 검증 전략, 짝 QAS-01·05·09·13, FR-0001 `realized-by`, dp4(A3/A5/A8·evaluation·INDEX·review)의 "정합" 언급. 본문엔 모두 `구 DP-0001`로 표기(삭제 표식 — grep 추적 가능).
- **후보 행선지(미결정)**: (a) **DP-0004**(Workflow 실행 구조)가 scale-out(A5/A8 ephemeral)으로 동적 풀을 흡수 / (b) 신규 DP로 분리 / (c) 비-ASR이면 드랍. **DP 디스커션에서 결정.**
- **남은 작업**: 행선지 확정 시 위 인용처의 `구 DP-0001` 참조를 새 owner로 redirect + `_backlog.md` BL-3 갱신.

## [ ] OI-13 DP-06 지식 베이스 — 위키 거버넌스(차등 게이팅 vs 정본+런타임 공유) 택일 미확정
- **맥락(2026-08-10)**: DP-06(Agent 지식 베이스) 신설 — 자율 이슈처리(불량 분석·회귀)의 전제인 지식 공급 계층이 어떤 DP에도 없던 공백을 닫음. 슬라이드 본편 4장(필요성 / 공통 구조 / 대안 구조도 / 비교·결정) + Appendix 1장(Cerebras) 구성.
- **경위(v1→v4)**: v1 "RAG vs LLM Wiki" 형태 비교 → v2 디스커션: 인덱싱·조회 계층은 공통 전제(채택 패턴 = Cerebras Knowledge), 결정 축을 부트스트랩 전략으로 전환 → v3(+.1·.2) 발표 용어 정비(1안 RAG / 2안 LLM 위키 + 강화 tactic)·KB-DQ 신설·설계명 확정 → **v4(2026-08-13) 거버넌스 재프레이밍**: 팀 방향 전환으로 **LLM 위키 기반 채택**(A4 수렴 노트 채택 — 형태 택일 종료, 기존 소스(IMS·Confluence·Slack) 증류 md로 구축·콜드스타트 닫음). **결정 축 = 위키 갱신 거버넌스** — 정본(설계·정책)의 사람(시니어·아키텍트) PR 게이트는 양안 공통, 대안은 비정본 지식의 흐름: **1안 지식 등급별 차등 게이팅**(운영 티어 = LLM-as-a-judge auto-merge, 영속 등재) vs **2안 엄격 정본 + 런타임 공유 계층**(위키 밖 무게이트 즉시 공유, 승격은 전량 사람 게이트). KB-DQ-4(Consistency) 신설. 조회 계층 상세는 `_backlog` BL-4.
- **미확정**: ① **1안(차등 게이팅 — 권고) vs 2안(정본+런타임 공유) 택일** — 드라이버 Q1(시니어·아키텍트 리뷰 대역폭이 위키 갱신 전량을 감당하는가)·Q2(미검증 최신 정보의 실시간 공유가 업무 흐름에 필수인가)를 팀/PM이 판정, DP 디스커션 회부. ② `◯` 해제 — 1안 정본 ★★★◯(오분류 표본 감사 규율 수립), 1안 운영 ★★★◯(judge κ≥0.8 선검증, QA-07 조건 공유). ③ 도안 SVG — v4 신규 2장 작성 완료(2026-08-13, `1an-tiered-gating.svg`·`2an-canonical-runtime.svg` — 구 RAG/위키 도안 삭제); 선택안 확정 시 선택안 강조 버전으로 갱신만 남음. ④ BL-4(조회 계층 상세 설계) 승격 여부. ⑤ A4 수렴 노트(1안 + 런타임 계층 = 3계층 게이트 스펙트럼)의 정식 대안 승격 여부. ⑥ **KB-DQ의 QA 승격 여부** — v4에서 KB-DQ-4 정합성(25012 Consistency) 추가(커버리지·신선도·축적 속도·정합성), 전사 QA(비-ASR)로 올릴지 팀 디스커션 + 예시 KPI(context recall·stale-hit율·time-to-knowledge·모순 검출률) PoC 확정. ⑦ **(v4 신설) 티어 경계 룰 초안** — 1안 채택 시 정본/운영 분류 기준(경로·frontmatter 스키마·judge 에스컬레이션 룰) 작성 필요. ⑧ **(v4 신설) 2안 채택 시 런타임 공유 계층의 브로커 귀속** — Orchestrator(DP-01)가 게시·구독을 겸하는지 인터페이스 정합 확인.
- **남은 작업**: 선택안 확정 시 — module-view Repository 레이어에 `Knowledge Base`(위키 + 조회 계층 + 게이팅 파이프라인, 2안 시 런타임 공유 계층 추가) 반영, QA-07/05/01 `related-dp`에 DP-06 역방향 링크 추가, eval/검증 서브시스템 DP(OI-7)와 judge 하네스 공유(1안 auto-merge·2안 승격 필터, R-3 방어) 정합 확인, C-03에 provenance "미검증 근거 비가역 액션 금지" 규칙 반영 여부(2안).
