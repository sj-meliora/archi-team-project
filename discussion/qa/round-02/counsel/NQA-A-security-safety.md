# Counsel: NQA-A Security / Safety — 자율 에이전트 보안·안전 (round-02)

> refs-review: [round-02/review/NQA-A-security-safety.md](../review/NQA-A-security-safety.md) · [report.md](../review/report.md)(C3·Stage2-1) · 공유: [QA-03](../review/QA-03-controllability.md)(red-team 하네스)
> 직전 counsel: [round-01/counsel/NQA-A-security-safety.md](../../round-01/counsel/NQA-A-security-safety.md) (신설 — 강력권장, 우선순위 상위)
> seats: 발의 **Seat 1**(Agentic Workflow) · 합의 **consensus** (Seat 2 ISO Security 앵커·정식화 / Seat 3 admission·서명·secrets DP)
> stance: **정식 채택 권장(우선순위 상위 진입, OI-8) — QA-03과 공유 red-team 하네스 동반 닫힘**

## Reviewer 지적 요약

- round-02 verdict: **Sound ○ / KPI △ · Med.** 신설 내용·ISO/IEC 25010 Security 앵커·KPI 5축 완성도 높음 → **정식화 가부 = 채택 권장**(Sound·앵커 충분, "사람 없이 믿고 맡길 수 있는가" 두 기둥 중 하나).
- **C3 핵심 잔여**: 주 KPI(`권한 상승·범위 외 배포=0`)·injection 차단율·서명 무결성·secrets 노출 — **5축 중 4축의 측정 수단 = red-team 세트 = [발표 서사]**(미구축). QA-03 ②(위반0)와 **공유 red-team 하네스** → 두 QA 닫힘이 한 묶음.
- 잔여: 번호 재정렬(OI-8), DP 보안 tactic(서명·secrets·injection 가드레일) 부재(OI-7), HITL SSoT 명시(Low).

## 개선안 (정의·KPI 기존→제안)

Council 응답: **새 KPI 발명이 아니라 (a) 정식 채택으로 4축 검증을 [발표 서사]→실측 전환, (b) QA-03과 공유 하네스로 동반 닫힘, (c) HITL SSoT 확정**. Sound ○·정의 유지.

**정의: 기존 → 제안 (유지 + SSoT)**
- 기존: 권한·자격증명·공급망 안전 + injection 내성. `Controllability(QA-03) ⊂ Security` 양방향 명문화 — 유지.
- 제안: HITL 통과율 KPI는 QA-03 ④와 공유 — **SSoT = NQA-A(보안)**, QA-03 ④는 제어 관점 참조임을 명문화(중복 측정 방지, review 렌즈2).

**KPI: 기존 → 제안**

| # | 기존 (round-01 신설) | 제안 (round-02) | 닫힘 상태 |
|---|---|---|---|
| 주 | 권한 상승·범위 외 배포=0 (적대적 red-team eval) | 동일 — **공유 red-team 하네스로 실측 전환**(PoC-N-A1≡C2). `절대 0` → `적대적 세트 기준 0건(커버리지=신뢰 상한, OWASP LLM Top-10 매핑 log)` | △ (채택+실측 시 ○) |
| 보조1 | HITL 통과율=100%·우회=0 (QA-03 공유) | 동일 — **SSoT=NQA-A** 명시 | 측정 가능 |
| 보조2 | artifact 서명·무결성=100% | 동일 — DP 공급망 서명 tactic 명시 선결(OI-7) | △ |
| 보조3 | injection 차단율 ≥95% | 동일 — red-team 세트 실측(예시값 confirm) | △ |
| 보조4 | secrets 노출=0 | 동일 — secrets 스캐너·trace 마스킹 DP tactic 선결(OI-7) | △ |

> ⚠️ `0건·100%·95%`는 "측정 가능 KPI의 모양" 예시값 — red-team 세트로 확정(0건·100%는 절대 기준).

## 근거 (레퍼런스)

§4 라이브러리 **제어·안전(Controllability/Safety)** 행 + ISO Security 앵커 + 공급망 표준.

- **zero-trust 최소권한 + HITL + 가드레일 + injection 방어; 위반율은 적대적 eval로** — 자율 에이전트(배포권·자격증명·외부 도구 실행권 보유)의 신종 공격면 방어 필드 표준.
  - Anthropic *Building Effective Agents*: https://www.anthropic.com/engineering/building-effective-agents
  - 신뢰 에이전트 프레임워크: https://www.anthropic.com/news/our-framework-for-developing-safe-and-trustworthy-agents
- **ISO/IEC 25010:2023 Security**(기밀성·무결성·부인방지·책임추적성·인증성) + Safety — KPI↔하위특성 앵커링(기밀성↔secrets·무결성↔서명·책임추적성↔HITL)이 정식화 자격의 근거. 정식 1급 QA 승격 모범.
  - ISO 25010: https://iso25000.com/index.php/en/iso-25000-standards/iso-25010
- **공급망 서명·무결성 = SLSA** — artifact 서명·provenance 검증이 공급망 변조 방어 표준.
  - SLSA: https://slsa.dev/
- **injection·권한상승 적대적 세트 = OWASP LLM Top-10** — red-team 세트 커버리지 매핑으로 신뢰 상한 노출.
  - OWASP LLM Top-10: https://owasp.org/www-project-top-10-for-large-language-model-applications/

> 레퍼런스 수치(95% 등)는 패턴 정당화용 — 합격선은 PoC red-team 세트로 확정.

## PoC 증명법

### PoC-N-A1(≡C2, R2 실측 전환): red-team eval 세트로 보안 5축을 측정하고 QA-03과 동반 닫는다 (eval 적대적 — 공유 하네스)

- **가설(Hypothesis)**: "OWASP LLM Top-10 매핑 red-team 세트로 권한상승·범위외배포·injection·서명·secrets 5축이 측정 가능하며, **QA-03 ②(위반0)와 동일 하네스 1개로 동반 산출**된다."
- **지표(Metric) + 합격선**:
  - 권한 상승·범위 외 배포 = 0 (적대적 세트 기준; 커버리지 log)
  - injection 차단율 ≥ ◯% · 서명 검증 통과/차단율 · secrets 스캐너 탐지율
  - HITL 게이트 우회 탐색 커버리지(QA-03 ④ 공유)
- **셋업(Setup)**: red-team 세트 50~100건(OWASP 매핑) + 서명 검증(SLSA류) 주입 + secrets 스캐너 + injection 페이로드. **QA-03 PoC-C2와 동일 하네스**(C3 단일 인프라). admission control(권한 게이트)로 차단.
- **절차(Procedure)**:
  1. red-team 세트 구축(권한상승·injection·공급망 변조·secrets 유출 시도)
  2. admission/서명/스캐너 게이트로 차단 측정 + injection 차단율
  3. **QA-03 ② 위반0 집계로 동일 하네스 출력 공유 확인**(C3 동반 닫힘 시연)
- **합격 기준(Exit)**: 적대적 세트에 대해 위반 0 ∧ 차단율 ≥ 목표 ∧ 서명 100% ∧ **동일 하네스가 QA-03 ② 출력을 공급**(공유 인프라 1개로 2 QA 닫힘).
- **규모/기간(Scope)**: red-team 세트 구축이 critical path(QA-03 공유라 1회 비용) → 약 3~4일.
- **리스크/한계(silent cap)**: 세트 커버리지 = 위반0의 신뢰 상한(zero-day·미상상 우회 미검출). 서명·secrets·injection은 통합 환경(vault·빌드서버) 필요. **PoC는 정식 채택(OI-8)을 대체하지 못함** — 4축 닫힘은 채택 동반.

## DP·발표 영향

- **DP 연결 (OI-7, DP 디스커션 위임)**: DP-0002/0003에 **공급망 서명·secrets 관리·injection 가드레일** tactic 미명시 — 5축 중 서명·secrets·injection 3축의 책임 설계가 비어 있음. 주 KPI·HITL은 admission control(DP-0003)+HITL(DP-0002)에 환원되나 나머지는 부재. **eval/검증 서브시스템 DP**(red-team 하네스)가 NQA-B golden·QA-03·QA-07·QA-09와 공유 — OI-7 큰 줄기.
- **동반 닫힘 효과 (C3 수렴)**: NQA-A 정식 채택 + 공유 red-team 하네스 실측 = **QA-03 ②·NQA-A 주 KPI 동반 닫힘**. NQA-B(golden)와는 다른 게이트(red-team)지만 **같은 eval/검증 서브시스템 DP**로 묶임 → C3 단일 DP 수렴의 한 축.
- **번호/서사 (OI-8)**: NQA-B(Correctness)와 함께 "사람 없이 믿고 맡길 수 있는가" 두 기둥 → **우선순위 상위 진입**(Security를 QA-01급 후보). 채택 시 QA 2자리 번호 재정렬·INDEX/glossary 동기화·전 QA cross-ref. importance 상위 권고.
