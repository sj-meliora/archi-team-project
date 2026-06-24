# 신규 QA 권고 (round-02) — 누락된 1급 품질속성

> 성격: **권고(recommendation)**. round-01에서 발굴한 NQA-A/B/C는 이미 `context/qa/`에 신설(임시 ID)됐고, round-02는 그 **정식화(OI-8)**를 재평가했다(상세는 각 [`NQA-*.md`](NQA-A-security-safety.md) 리뷰).
> 근거 렌즈: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트

## round-02 신규 발굴 = 0건

세트 전체를 3렌즈로 재전수했으나 **round-02에서 새로 발굴된 1급 QA 후보는 없다.** round-01 C5의 NQA-A/B/C가 자율 agentic 시스템의 빠진 속성(보안·정확성·비용 top-line)을 이미 덮었고, round-02 잔여는 **신규 속성 누락이 아니라 (a) 기존 신설 QA의 정식 채택 미결(OI-8) (b) KPI-DP 귀속(OI-7)** 문제다. 즉 *발굴*보다 *채택·연결*이 다음 행동.

## 기존 신설 QA(NQA-A/B/C) 정식화 재평가 (OI-8)

| 후보 | round-02 verdict | 정식화 stance | 닫힘 영향 |
|---|---|---|---|
| **NQA-A** Security/Safety | Sound ○ / KPI △ · Med | **채택 권장**(우선순위 상위) | QA-03 ②(위반0)와 공유 red-team 하네스 — 동반 닫힘 |
| **NQA-B** Correctness | Sound ○ / KPI △ · **High** | **채택 강력권장** | **세트 닫힘 병목 — QA-07 주 KPI·QA-09 ②-2를 동반으로 닫는 단일 트리거** |
| **NQA-C** Cost-economy | Sound ○ / KPI △ · Med | 채택 권장(Med) | **미채택 시 QA-01 활용률·QA-05 top-line 부유** — 채택이 이양 닫힘 트리거 |

- **세 QA 모두 Sound ○**(ISO/IEC 25010 앵커·단일 관심사·경계 명문화 충족) → **정식 1급 QA 자격 충분.** KPI △는 측정 수단([발표 서사]·이양처)이 *채택 동반*으로 닫히는 구조라, 정식화 자체가 닫힘 행동.

## 채택 시 영향 (Stage 2 메모 — round-01 대비 갱신)

- **채택이 닫힘 트리거**: NQA-B 채택 → QA-07·QA-09 ②-2 동반 닫힘. NQA-C 채택 → QA-01·QA-05 부유 해소. NQA-A 채택 → QA-03 공유 하네스 실측 전환.
- **번호 재정렬**(OI-8): 최소 NQA-A(Security)·NQA-B(Correctness)는 "사람 없이 믿고 맡길 수 있는가" 두 기둥이라 상위 진입 자연스러움 → 기존 QA-01~10 전부 + INDEX·glossary·cross-ref 동기화.
- **양방향 cross-link 확정**: QA-07/09↔NQA-B, QA-03↔NQA-A(이미 양방향), QA-01/05↔NQA-C 이양.
- **검증 전환**: 정식 승격 시 [발표 서사]→실측 전환(red-team 세트·golden set 구축) — **eval/검증 서브시스템 DP 신설**(OI-7)이 선행 인프라.
