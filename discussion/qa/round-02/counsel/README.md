# counsel/ — round-02 blue team 권고 내비게이션

> blue team(Council) 산출물. 권고 전모는 [`counsel.md`](counsel.md) 하나로 파악. 방법론: [`../../Council.md`](../../Council.md) · red team 결론: [`../review/report.md`](../review/report.md)
> date: 2026-06-24 · scope: full · 직전: [round-01 counsel](../../round-01/counsel/counsel.md)

## 읽는 순서
1. [`counsel.md`](counsel.md) — 종합·메타(TL;DR·채택 권고표·교차권고 C1~C3·PoC 로드맵·verdict 추세·미해결·종료조건 신호).
2. [`_poc-plan.md`](_poc-plan.md) — PoC 의존그래프(허브: NQA-B golden 게이트 / 공유 red-team 하네스).
3. 개별 권고(§6 포맷):

| stance | 항목 |
|---|---|
| **정식 채택 권장** | [NQA-A](NQA-A-security-safety.md) · [NQA-B](NQA-B-correctness.md)(최우선) · [NQA-C](NQA-C-cost-economy.md) |
| **조건부(동반 닫힘)** | [QA-03](QA-03-controllability.md) · [QA-07](QA-07-performance-agent-time.md) · [QA-09](QA-09-reliability-agent-consistency.md) |
| **닫힘 확인** | [QA-01](QA-01-scalability.md) · [QA-02](QA-02-availability.md) · [QA-04](QA-04-observability.md) · [QA-05](QA-05-efficiency.md) · [QA-06](QA-06-reliability-workflow.md) · [QA-08](QA-08-performance-e2e.md) · [QA-10](QA-10-maintainability.md) |

## 한 줄 결론
수렴 라운드. 신규 KPI 0건. 잔여 = **OI-8 신규 QA 채택(사람) + OI-7 eval/검증 서브시스템 DP 신설(DP 디스커션 1순위)** 위임. NQA-B 채택이 세트 닫힘 단일 트리거.
