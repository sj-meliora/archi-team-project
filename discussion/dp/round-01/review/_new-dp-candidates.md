# 신규 결정/대안 후보 (NDP-*) — round-01 review

> red team이 발굴한 신규 설계결정/대안 권고. **성격 = 권고**, 채택은 Applier/팀 결정.
> 현재 **sample 단계 — 자리(placeholder)**. 결정 포인트 초안·후보 대안·채택 영향의 본문화는 **full 단계**.
> 주 동인: Traceability Matrix의 **orphan ASR(QA-06 Security · QA-07 Correctness)** 을 메우는 것.

---

## NDP-A (placeholder) — eval/검증 + security gate 서브시스템 DP
- **메우는 orphan ASR**: **QA-06 Security/Safety(H)** + **QA-07 Correctness(H)** (둘 다 어느 DP도 cover 안 함).
- **근거(OI-7)**: "eval/검증 서브시스템 = 신규 DP 후보 — QA-06 red-team 하네스·QA-07 golden+judge 하네스가 **어떤 DP에도 없는 신규 인프라**." QA-03 ②-2·QA-11 ②-2·QA-09 first-pass 게이트가 모두 이 하네스에 의존.
- **결정 포인트 초안 (full)**: 검증/eval을 ◯ (인라인 게이트 vs 별도 서브시스템 vs 외부 judge 서비스) 중 어떻게 둘 것인가.
- **후보 대안 초안 (full)**: ① 권한 게이트 강화(DP-03 1안 확장 — secrets·공급망 서명·injection 가드) / ② golden-set+LLM-judge 하네스(QA-07 κ≥0.8) / ③ red-team 하네스(QA-06 OWASP LLM Top-10 풀커버리지).
- **채택 영향 (full)**: QA-06·QA-07 orphan 해소 + QA-03/09/11 ②-2 게이트 닫힘. 신규 인프라 도입 비용(QA-13 Cost)·운영 리스크 동반.

## NDP-B (placeholder) — Correctness 검증 하네스 DP (NDP-A에서 분리 시)
- **메우는 orphan ASR**: **QA-07 Correctness(H)** 전담.
- NDP-A와 통합/분리 여부를 full에서 판정(security gate와 correctness 하네스가 같은 서브시스템인지).

## NDP-C (placeholder) — Agent Hierarchy Federation
- **출처**: DP-02 _backlog **BL-2**(도메인별 Sub-Orchestrator) 승격 후보.
- **동인**: DP-02 단일 Orchestrator 병목(TP-2)·NR-1의 확장 한계(QA-01)를 푸는 4안. orphan은 아니나 ASR 변별 강화.

---

> full 단계 할 일: ① 위 placeholder를 §5 형식(결정 포인트 + 후보 대안 + 채택 영향)으로 본문화 · ② NDP-A/B 통합/분리 확정 · ③ NDP가 메우면 Traceability Matrix의 QA-06/07 열이 채워지는지 재검증.
