# round-01 / counsel — blue team 권고 (내비게이션)

> Council(자문 합의체)이 red team review를 읽고 낸 **개선 권고**. 방법론: [`../../Council.md`](../../Council.md) · red team: [`../review/`](../review/)
> 상태: **full 스코프 완료** — QA-01~10 + NQA-A/B/C 전부 + 종합 보고서 + PoC 통합 계획.

## 먼저 읽을 것
- [`counsel.md`](counsel.md) — **이것만 읽으면 권고 전모**. 채택 권고표·근거맵·교차권고(C1~C5)·PoC 로드맵·메타(date 2026-06-24).
- [`_poc-plan.md`](_poc-plan.md) — 17개 PoC 우선순위·의존 그래프·공유 자산·총 기간.

## 개별 QA 권고 (§6 포맷: 지적요약→개선안→근거→PoC→DP영향)
- [`QA-01-scalability.md`](QA-01-scalability.md) — scaling efficiency + backlog 오토스케일 + rate-limit 헤드룸 (PoC-S1·S2)
- [`QA-02-availability.md`](QA-02-availability.md) — MTTR 분해(무손실 재개) + 외부 LLM 장애 (PoC-A1·A2)
- [`QA-03-controllability.md`](QA-03-controllability.md) — 위반 0건 편입 + runaway cap + HITL (PoC-C1·C2)
- [`QA-04-observability.md`](QA-04-observability.md) — trace 완전성 + event-history + MTTD (PoC-O1)
- [`QA-05-efficiency.md`](QA-05-efficiency.md) — 캐시/신규 토큰 분리집계, top-line→NQA-C (PoC-E1)
- [`QA-06-reliability-workflow.md`](QA-06-reliability-workflow.md) — 토큰/rate-limit 쿼터 격리 (PoC-R1)
- [`QA-07-performance-agent-time.md`](QA-07-performance-agent-time.md) — 노드타입별 speedup + first-pass 품질 게이트 (PoC-P1·P2)
- [`QA-08-performance-e2e.md`](QA-08-performance-e2e.md) — E2E latency p95·throughput 추가, 전달 5% 강등 (PoC-E2E1)
- [`QA-09-reliability-agent-consistency.md`](QA-09-reliability-agent-consistency.md) — 캐시우회 pass^k + 유효-결정률 (PoC-K1)
- [`QA-10-maintainability.md`](QA-10-maintainability.md) — CIS p95 + prompt/model 교체축 (PoC-M1)

## 신규 QA 권고 (채택 권고 + KPI 근거 + PoC)
- [`NQA-A-security-safety.md`](NQA-A-security-safety.md) — 신설·강력권장. red-team eval + HITL + 공급망 서명 (PoC-N-A1)
- [`NQA-B-correctness.md`](NQA-B-correctness.md) — 신설·권장. golden set + LLM-as-judge. **QA-07/09 게이트 전제** (PoC-N-B1)
- [`NQA-C-cost-economy.md`](NQA-C-cost-economy.md) — 신설·권장(Med). $/완료모델 + 절감률 (PoC-N-C1)

## 권고 포맷 (§6)
각 QA 파일 = Reviewer 지적 요약 → 개선안(정의·KPI 기존→제안) → 근거(레퍼런스) → PoC 증명법 → DP·발표 영향. 3 seats 발의·합의 표기.
