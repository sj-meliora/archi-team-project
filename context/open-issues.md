# 정합성 / 미해결 결정 트래커 (Open Issues)

> category: meta | updated: 2026-06-24 (OI-8 닫힘 — NQA-A/B/C → QA-06/07/13 정식 편입·재번호; round-02 contention 반영; OI-9 신설·닫힘 — ★ rubric 캘리브레이션 + QAS 동기화)
> 자료 변동으로 생긴 불일치 + 미확정 사항. 확정되면 해당 개념 파일에 반영하고 여기서 닫는다([x]).

## [ ] OI-1 QA 번호 불일치
- 슬라이드 11/13/25~28이 같은 QA01~04에 서로 다른 속성을 부여.
- **결정**: canonical = 팀 합의 우선순위 기준 (2026-06-23 재정렬, QA·QAS 2자리). glossary.md에 원본 매핑 보존.
- **2026-06-23 재정렬**: QA-01 Scalability / 02 Availability / 03 Controllability / 04 Observability / 05 Efficiency (이전 1위 Efficiency→5위). 06~10 불변.
- **2026-06-24 재번호(OI-8 닫힘)**: 신규 QA 정식 편입 — NQA-A→**QA-06 Security**, NQA-B→**QA-07 Correctness**(6·7위 삽입), 기존 QA-06~10 +2 시프트(→QA-08~12), NQA-C→**QA-13 Cost**. canonical = **QA-01~13**.
- **남은 작업**: 발표 슬라이드 전체에서 canonical(2자리, QA-01~13) 번호로 통일. FR/C/DP도 2자리 전환 예정.

## [ ] OI-2 DP-0001 1안 '단점' 칸 표기 오류
- Per-Node Agent(1안) 단점에 2안의 장점이 복붙됨.
- **남은 작업**: 1안의 실제 단점으로 교체 (예: [Scalability] 고정 배치라 확장 경직 / [Efficiency] 노드별 전용 Agent 유휴 비용).

## [ ] OI-3 DP-0004 대안 라벨 오류
- 헤더가 DP-0002 라벨(Hierarchical/Decentralized) 복붙. 실제 내용은 "타입별 서버 풀 vs 노드당 Workflow 인스턴스".
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
- **DP-0001(동적 풀)·DP-0004(타입별 scale-out)**: rate-limit·admission control 차원 미명시 (QA-01 반영 시 발견).
- **DP-0002(Standby)·DP-0003(격리/모니터링)**: "외부 LLM degradation"(outage/429) 시나리오 미명시 — backoff·폴백 tactic 보강 필요 (QA-02 반영 시 발견).
- **DP-0002(제어지점)·DP-0003(권한 게이트)**: **runaway cap**(max iter·token·wall-clock)과 **graceful 정지+롤백**(협조적 취소 vs hard-kill) tactic 미명시 (QA-03 반영 시 발견). DP-0002 1안 HITL·DP-0003 1안 allowlist는 있으나 cap·정지 의미 분리는 후보 대안/ATAM에 없음.
- **DP-0001 ↔ QA-02 링크 불일치**: QA-02 반영 시 related-dp에서 DP-0001 제거(가용성 기여가 '장애 격리'라 QA-08 소관)했으나, **DP-0001 파일은 여전히 `drives: QA-02`로 선언** → 단방향 불일치. DP-0001의 `drives`를 **QA-02→QA-08으로 교정**할지 DP 디스커션에서 결정.
- **DP-0003(실시간 모니터링)**: span 수준 trace(prompt+context+tool I/O+model+token+ts) 보장 여부 미명시 — "규칙 기반 한정 적용" 시 어디까지 span을 남기는지 명시 필요 (QA-04 반영 시 발견).
- **DP-0001(2안 동적 풀)**: 작업별 최적 agent 선택이 **토큰 기준인지 비용 기준인지** 미명시 — 비용 기준 정렬 권고 (QA-05 반영 시 발견).
- **DP-0001(1안 즉시 실행)**: latency만 보고 **품질 게이트(first-pass 성공)와 무연결** — "빠르게 틀리기" 차단 tactic 미명시 (QA-09 반영 시 발견).
- **DP-0004/0005**: E2E latency 책임 및 **결정 산식(4단계×20GB÷대역폭 5% budget → A5 vs A8)의 실측 의존**, prompt/모델 교체 내성(workflow 버저닝·계약 분리) 미명시 (QA-10·QA-12 반영 시 발견).
- **신규 QA 교차 의존(QA-07 전제)**: QA-09 first-pass 품질 게이트·QA-11 **②-2 정밀 유효-결정률**이 **QA-07(Correctness) golden 게이트에 의존** — QA-07 신설과 함께 가야 닫힘. (QA-11 contention 반영으로 ②-1 대리 유효-결정률은 golden 불요 룰 게이트로 분리·해소됨 — ②-2만 잔존 의존.)
- **신규 QA 교차 의존(QA-06 전제) — round-02 QA-03 contention 등록**: QA-03 **②-2 적대적 위반0(acceptance)**가 **QA-06(Security) 공유 red-team 하네스 풀커버리지(OWASP LLM Top-10 매핑)에 의존** — QA-06 정식 채택과 함께 가야 닫힘(QA-11 ②-2 ↔ QA-07와 동일 구조). (QA-03 contention 반영으로 ②-1 권한외 차단율은 룰 체커로 분리·**현 조건 선닫힘** — ②-2만 잔존 의존.) eval/검증 서브시스템 DP 신설(아래)이 ②-2 하네스를 받침.
- **eval/검증 서브시스템 = 신규 DP 후보**: QA-06 red-team 하네스·QA-07 golden+judge 하네스가 **어떤 DP에도 없는 신규 인프라** — DP 디스커션에서 "eval/검증 서브시스템" DP 신설 검토 (QA-06/07/QA-03 공유 자산).
- **DP-0002/0003 보안 tactic 미명시**: 공급망 artifact 서명·secrets 관리·injection 가드레일이 후보 대안/ATAM에 없음 (QA-06 신설 시 발견).
- **`related-dp` 추가**: QA-05에 DP-0005, QA-09에 DP-0004 추가함(반영 완료). 역방향(DP의 `drives`)과 정합 확인 필요.
- **남은 작업**: 각 DP에 해당 tactic(rate-limit headroom·admission control·외부 의존성 backoff·폴백·runaway cap·graceful 정지·span trace·품질 게이트·workflow 버저닝)을 후보 대안/ATAM에 명시할지 + DP-0001 drives 교정 + QA-07 교차 의존을 DP 디스커션에서 역검토.

## [x] OI-8 신규 QA 정식 승격·번호 재정렬 — 2026-06-24 닫힘
- **결정(2026-06-24 팀)**: 신규 QA 3종 **정식 채택**. 번호 — NQA-A→**QA-06 Security/Safety(H)**, NQA-B→**QA-07 Correctness(H)**(6·7위 삽입), NQA-C→**QA-13 Cost-economy(M)**(말미 유지). 기존 QA-06~10은 +2 시프트(→QA-08~12). 우선순위 추가 상향(Security를 더 위로)은 팀 논의 후 별도(번호와 분리).
- **반영 완료**: 파일 리네임(QA/QAS-06·07·13) + frontmatter id·heading + 전 QA·QAS·INDEX·glossary·open-issues·DP cross-ref 동기화 + 각 신규 QA 변경이력. 전체 매핑은 `changelog.md` 2026-06-24 항목.
- **ASR 재선정(팀 판단, DP 생성 동인)**: **QA-01~07**(기존 5 + Security·Correctness). 기존 Reliability-WF(→QA-08)·Performance-Agent(→QA-09)는 ASR에서 빠짐. discussion 스킬·Contention 방법론 갱신.
- **연결 확정**: QA-05 top-line·QA-01 활용률 → QA-13 이양 / QA-09·QA-11 ②-2 게이트 ↔ QA-07 양방향 / QA-03 ②-2 ↔ QA-06 red-team 하네스(round-02 contention; ②-1 선닫힘, ②-2만 QA-06 의존).
- **잔여(OI-7로 이관)**: 검증 PoC 실측 전환·eval/검증 서브시스템 DP 신설은 DP 디스커션(OI-7). 발표 우선순위 추가 상향은 팀 논의.

## [x] OI-9 ★ 등급 척도 캘리브레이션의 하한 보정 ↔ QAS·glossary 정합 — 2026-06-24 닫힘
- round-03 ★ rubric 캘리브레이션(2026-06-24)으로 8개 QA의 §측정 합격 하한을 재배치(QA-01 0.8→0.70 / QA-02 1분→4분 / QA-06 95→70% / QA-08 10→25% / QA-09 3배→1× / QA-10 6h→24h / QA-11 pass^k 70→25·40·60% / QA-13 70→50%·$/모델→게이트) + QA-03(graceful stop 재배치)·QA-04(★★★ 상한 캡)·QA-07(만점 캡+judge κ 조건) 상한/조건 추가. QA 본문엔 옛값 보존 트레이스로 반영.
- **2026-06-24 닫힘**: 짝 QAS 9개(QAS-01·02·03·06·08·09·10·11·13) Measure 수치를 보정 합격 하한으로 동기화(게이트 불변)+footnote·`updated:`·★ 급간 포인터. **glossary는 수치 미포함이라 동기화 불요.** QAS-04·07은 합격 하한 불변이라 제외. 등급 경계는 전부 예시값 — PoC 실측 시 최종 확정. (`changelog.md` 2026-06-24 OI-9 항목.)
