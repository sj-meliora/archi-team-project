# 정합성 / 미해결 결정 트래커 (Open Issues)

> category: meta | updated: 2026-06-24
> 자료 변동으로 생긴 불일치 + 미확정 사항. 확정되면 해당 개념 파일에 반영하고 여기서 닫는다([x]).

## [ ] OI-1 QA 번호 불일치
- 슬라이드 11/13/25~28이 같은 QA01~04에 서로 다른 속성을 부여.
- **결정**: canonical = 팀 합의 우선순위 기준 (2026-06-23 재정렬, QA·QAS 2자리). glossary.md에 원본 매핑 보존.
- **2026-06-23 재정렬**: QA-01 Scalability / 02 Availability / 03 Controllability / 04 Observability / 05 Efficiency (이전 1위 Efficiency→5위). 06~10 불변.
- **남은 작업**: 발표 슬라이드 전체에서 canonical(2자리) 번호로 통일. FR/C/DP도 2자리 전환 예정.

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
- **DP-0001 ↔ QA-02 링크 불일치**: QA-02 반영 시 related-dp에서 DP-0001 제거(가용성 기여가 '장애 격리'라 QA-06 소관)했으나, **DP-0001 파일은 여전히 `drives: QA-02`로 선언** → 단방향 불일치. DP-0001의 `drives`를 **QA-02→QA-06으로 교정**할지 DP 디스커션에서 결정.
- **DP-0003(실시간 모니터링)**: span 수준 trace(prompt+context+tool I/O+model+token+ts) 보장 여부 미명시 — "규칙 기반 한정 적용" 시 어디까지 span을 남기는지 명시 필요 (QA-04 반영 시 발견).
- **DP-0001(2안 동적 풀)**: 작업별 최적 agent 선택이 **토큰 기준인지 비용 기준인지** 미명시 — 비용 기준 정렬 권고 (QA-05 반영 시 발견).
- **DP-0001(1안 즉시 실행)**: latency만 보고 **품질 게이트(first-pass 성공)와 무연결** — "빠르게 틀리기" 차단 tactic 미명시 (QA-07 반영 시 발견).
- **DP-0004/0005**: E2E latency 책임 및 **결정 산식(4단계×20GB÷대역폭 5% budget → A5 vs A8)의 실측 의존**, prompt/모델 교체 내성(workflow 버저닝·계약 분리) 미명시 (QA-08·QA-10 반영 시 발견).
- **신규 QA 교차 의존(NQA-B 전제)**: QA-07 first-pass 품질 게이트·QA-09 유효-결정률이 **NQA-B(Correctness) golden 게이트에 의존** — NQA-B 신설과 함께 가야 두 QA의 KPI가 닫힘.
- **eval/검증 서브시스템 = 신규 DP 후보**: NQA-A red-team 하네스·NQA-B golden+judge 하네스가 **어떤 DP에도 없는 신규 인프라** — DP 디스커션에서 "eval/검증 서브시스템" DP 신설 검토 (NQA-A/B/QA-03 공유 자산).
- **DP-0002/0003 보안 tactic 미명시**: 공급망 artifact 서명·secrets 관리·injection 가드레일이 후보 대안/ATAM에 없음 (NQA-A 신설 시 발견).
- **`related-dp` 추가**: QA-05에 DP-0005, QA-07에 DP-0004 추가함(반영 완료). 역방향(DP의 `drives`)과 정합 확인 필요.
- **남은 작업**: 각 DP에 해당 tactic(rate-limit headroom·admission control·외부 의존성 backoff·폴백·runaway cap·graceful 정지·span trace·품질 게이트·workflow 버저닝)을 후보 대안/ATAM에 명시할지 + DP-0001 drives 교정 + NQA-B 교차 의존을 DP 디스커션에서 역검토.

## [ ] OI-8 신규 QA(NQA-A/B/C) 정식 승격·번호 재정렬
- round-01 디스커션에서 신규 QA 3개 신설(임시 ID): **NQA-A Security/Safety(H)·NQA-B Correctness(H)·NQA-C Cost-economy(M)**. 파일은 `context/qa/`에 6섹션 골격으로 존재하나 **2자리 우선순위 번호 미부여**.
- **결정 필요**: ① 정식 채택 여부(팀원 논의) ② 채택 시 우선순위 번호 재정렬 — 최소 NQA-A(Security)·NQA-B(Correctness)는 "사람 없이 믿고 맡길 수 있는가"의 두 기둥이라 상위 진입이 자연스러움 ③ 재번호 시 기존 QA-01~10 전부 + INDEX·glossary·cross-ref 동기화.
- **연결 정리(채택 전제)**: QA-05 top-line·QA-01 "자원 활용률" → NQA-C 이양 확정 / QA-07·QA-09 게이트 ↔ NQA-B 양방향 cross-link / QA-03 ↔ NQA-A 양방향 cross-link.
- **검증 전략 전환**: 현재 NQA 검증은 [발표 서사](실측 미실행). **정식 QA 승격 시 PoC를 실측으로 전환**(팀 결정 — 2026-06-24).
