# Counsel: QA-03 Controllability — Agent 제어 용이성 (round-02)

> refs-review: [round-02/review/QA-03-controllability.md](../review/QA-03-controllability.md) · [report.md](../review/report.md)(C3) · 공유: [NQA-A](../review/NQA-A-security-safety.md)(red-team 하네스)
> 직전 counsel: [round-01/counsel/QA-03-controllability.md](../../round-01/counsel/QA-03-controllability.md) (채택 권장 — 위반 0건 편입 + runaway cap)
> seats: 발의 **Seat 1**(Agentic Workflow) · 합의 **consensus** (Seat 2 measurable 등급 / Seat 3 cancel 전파·runaway cap DP)
> stance: **닫힘 확인(Sound ◎ 유지) + ② 검증 실측 전환 권장 — NQA-A 채택과 한 묶음(OI-8)**

## Reviewer 지적 요약

- round-02 verdict: **Sound ◎ / KPI △ · Med.** round-01의 두 핵심(stop 의미 분리 = `ack ≤5초 AND graceful stop+롤백 ≤30초`, runaway cap KPI ③ 편입)이 반영됐고 C2(QA↔QAS 위반0 SSoT 분기)도 해소.
- 잔여: ② `허용범위 외 통과=0`의 측정수단이 **적대적 eval 세트 = [발표 서사]** — NQA-A red-team 하네스와 공유라 두 QA 닫힘이 한 묶음(C3). + runaway cap·graceful stop의 DP 귀속 부재(OI-7).
- review 권고: ②를 "발표 서사 지표"로 솔직히 강등하거나 NQA-A 채택+실측 전환, 커버리지 한계 명문화(Low), 부분 롤백 내부상태 기준 명시(Low).

## 개선안 (정의·KPI 기존→제안)

Council 응답의 핵심은 **새 KPI 발명이 아니라 ②의 검증을 [발표 서사]→실측 acceptance로 전환하는 단일 메커니즘을 NQA-A와 묶는 것**이다. Sound ◎이므로 정의 본문은 유지.

**정의: 기존 → 제안 (유지 + SSoT 명시)**
- 기존: 최소권한·allowlist + 고위험 HITL + 중단·runaway cap. `Controllability ⊂ Security`(NQA-A) 경계 명문화 — 유지.
- 제안: ④ HITL 통과율 KPI는 NQA-A와 **공유** — SSoT를 NQA-A(보안)에 두고 QA-03 ④는 제어 관점 참조임을 한 줄 명문화(중복 측정 방지, review 렌즈2 권고).

**KPI: 기존 → 제안**

| # | 기존 (round-01 반영) | 제안 (round-02) | 닫힘 상태 |
|---|---|---|---|
| ① 주 | 중단 ack ≤5초 AND graceful stop+롤백 ≤30초 | 동일 — 단 **롤백 30초가 "내부 상태 기준"임을 명문화**(외부 시스템 부작용 롤백은 silent cap, review Low). PoC-C1으로 ack/정지완료 분포 측정 | △→○ (PoC 통과 시) |
| ② 보조 | 허용범위 외 통과=0 (적대적 eval) | 동일 — **NQA-A 공유 red-team 하네스로 실측 전환**(PoC-C2≡N-A1). `절대 0` → `적대적 eval 세트 기준 0건(세트 커버리지가 신뢰 상한, OWASP LLM Top-10 매핑 log)`. 미채택 시 "발표 서사 지표"로 명시 강등 | △ (NQA-A 채택+실측 시 ○) |
| ③ 보조 | runaway cap 작동=100% (max iter·token·wall-clock) | 동일 — runner activity timeout/retry cap으로 강제. DP-0002/0003에 cap tactic 명시(OI-7) | 측정 가능 |
| ④ 보조 | HITL 통과율=100%·우회=0 | 동일 — **SSoT = NQA-A**, QA-03은 참조 | 측정 가능 |

> ⚠️ `5초·30초·0건·100%`는 "측정 가능 KPI의 모양" 예시값 — 취소 타이밍 모델·실환경으로 확정(레퍼런스 복제 아님).

## 근거 (레퍼런스)

§4 라이브러리 **제어·안전(Controllability/Safety)** 행 직접 적용.

- **zero-trust 최소권한 + 고위험 액션 HITL 승인 + 별도 가드레일 + injection 방어; 위반율은 적대적 eval로** — 자율 에이전트 제어의 필드 표준. ② `위반0`을 적대적 eval로 측정하라는 것이 곧 이 패턴.
  - Anthropic *Building Effective Agents*: https://www.anthropic.com/engineering/building-effective-agents
  - 신뢰 에이전트 프레임워크(human control): https://www.anthropic.com/news/our-framework-for-developing-safe-and-trustworthy-agents
- **협조적 취소(cooperative cancellation) + hard-kill 폴백 + activity timeout** — durable execution이 runaway cap·graceful stop을 강제하는 정석. ①ack≠정지 분리와 ③cap을 runner가 받친다.
  - Temporal durable execution(activity timeout·cancellation): https://temporal.io/blog/what-is-durable-execution · https://docs.temporal.io/temporal
- **커버리지 = 신뢰 상한**: 적대적 eval "0건"은 *세트가 상상한 공격*에 대해 0건 — OWASP LLM Top-10 매핑으로 커버리지를 노출해 과대주장 방지.
  - OWASP LLM Top-10: https://owasp.org/www-project-top-10-for-large-language-model-applications/

> 레퍼런스 수치(5초·0건 등)는 패턴 정당화용 — 합격선은 PoC로 확정.

## PoC 증명법

### PoC-C2(≡N-A1, R2 실측 전환): 적대적 eval 세트로 위반 통과 0과 runaway cap 작동을 실측한다 (eval 하네스 — NQA-A 공유)

- **가설(Hypothesis)**: "공유 red-team eval 세트(OWASP LLM Top-10 매핑)로 허용범위 외 통과율·runaway cap 작동률이 측정 가능하며, NQA-A 보안 위반0과 **동일 하네스 1개로 동반 산출**된다."
- **지표(Metric) + 합격선**:
  - 허용범위 외 통과 = 0 (적대적 세트 기준; 세트 커버리지를 log)
  - runaway cap 작동률 (iter/token/wall-clock 3종 cap별 차단 정확도)
  - HITL 게이트 우회 탐색 커버리지 (NQA-A ④ 공유)
- **셋업(Setup)**: 적대적 eval 세트 50~100건(OWASP LLM Top-10 매핑) + runaway 주입(무한루프·토큰 폭주 케이스) + cancel 폴링 밀도별 측정. **NQA-A PoC-N-A1과 동일 red-team 하네스**(C3 단일 인프라).
- **절차(Procedure)**:
  1. 적대적 세트 구축(허용범위 외 액션·injection·권한상승 시도)
  2. 각 케이스 통과/차단 측정 + runaway 주입 → cap별 차단 정확도
  3. **NQA-A 보안 위반0 집계로 동일 하네스 출력 공유 확인**(C3 동반 닫힘 시연)
- **합격 기준(Exit)**: 적대적 세트에 대해 통과 0 ∧ cap 100% 작동 ∧ **동일 하네스가 NQA-A 주 KPI 출력을 공급**(공유 인프라 1개로 2 QA 닫힘).
- **별도 PoC-C1(중단 latency)**: cancel 폴링 밀도별 ack ≤5초·graceful stop+롤백 ≤30초 분포 측정(측정 아키타입). 내부 상태 기준임을 log.
- **규모/기간(Scope)**: 적대적 세트 구축이 critical path(NQA-A와 공유라 1회 비용) → 약 3~4일.
- **리스크/한계(silent cap)**: 세트 커버리지 = 위반0의 신뢰 상한(미상상 우회 미검출). 부분 롤백은 내부 상태 기준 — 외부 시스템(Jira·빌드서버) 부작용 롤백은 통합 환경 필요. **PoC는 NQA-A 채택(OI-8)을 대체하지 못함** — ② 닫힘은 채택 동반.

## DP·발표 영향

- **DP 연결 (OI-7, DP 디스커션 위임)**: DP-0002/0003에 **runaway cap·graceful stop+롤백(협조적 취소 vs hard-kill) tactic** 미명시 — KPI ①·③의 설계 귀속이 비어 있음. **eval/검증 서브시스템 DP**(NQA-A red-team 하네스와 공유)가 ②를 받침. QA 측 verdict는 여기로 내려가지 않고 DP로 위임.
- **동반 닫힘 효과**: ② 위반0은 **NQA-A와 공유 red-team 하네스** — NQA-A 정식 채택 + 하네스 실측 전환이 QA-03 ②·NQA-A 주 KPI를 동반 닫는다(C3 수렴). NQA-B(golden)와는 다른 게이트(red-team)지만 **같은 eval/검증 서브시스템 DP**로 묶임.
- **번호/서사**: QA-03(◎)은 "사람 개입 없이"를 안전화하는 핵심 — 발표에서 NQA-A와 "제어(QA-03) ⊂ 보안(NQA-A)" 한 쌍으로 제시. importance 변동 권고 없음.
