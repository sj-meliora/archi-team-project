# round-04 counsel — 내비게이션 (blue team Council)

> ★ 등급 척도 근거 보강 라운드 — round-04 review(C1~C4) 지적을 레퍼런스+PoC로 응답.
> 날짜 2026-06-25 · 방법론 [`../../Council.md`](../../Council.md)(특히 §9 ★ rubric). review 입력: [`../review/report.md`](../review/report.md).

## 읽기 순서
1. [`counsel.md`](counsel.md) — **이것만 읽으면 권고 전모**(채택 권고표·C1~C4 응답·★ 급간 변경·C4 웹검증·직전 대비).
2. [`_poc-plan.md`](_poc-plan.md) — C1~C4 + QA-11 ★★★ 재판정 PoC 통합 계획·C4 웹검증 표.
3. QA별 개선 권고 (§6 포맷):

| QA | 파일 | stance | 핵심 |
|---|---|---|---|
| QA-01 | [counsel](QA-01-scalability.md) | 조건부 | margin 차등 근거화·LLM-bound USL silent cap |
| QA-02 | [counsel](QA-02-availability.md) | 조건부 | ★★★ main 축 정의 고정·외부 outage silent cap |
| QA-03 | [counsel](QA-03-controllability.md) | 채택 권장 | 세트 모범·★★★ 15초 근거 다양화 |
| QA-04 | [counsel](QA-04-observability.md) | 채택 권장 | 경계 표기 명확화·완전성 정의 강화 |
| QA-05 | [counsel](QA-05-efficiency.md) | 채택 권장 | 적중률 조건 silent cap·tier margin 명시 |
| **QA-06** | [counsel](QA-06-security-safety.md) | 조건부 | **Unit 42 직접 injection apples 명문화·margin 5%p 규칙화·FPR 전급간 게이트** |
| QA-07 | [counsel](QA-07-correctness.md) | 조건부 | ★★☆ 확장·judge κ 도메인 재측정·rework율 보조 축 |
| QA-08 | [counsel](QA-08-reliability-workflow.md) | 채택 권장 | ★★★ 5% margin 근거화·쿼터 보조 축 |
| QA-09 | [counsel](QA-09-performance-agent-time.md) | 조건부 | **InfEngine 인용 정정(8.6~22.7×→21×·도메인 불일치)·하한 표현 통일** |
| QA-10 | [counsel](QA-10-performance-e2e.md) | 채택 권장 | ★★★ 2h margin 근거화·agentic LLM 큐잉 가산 |
| **QA-11** | [counsel](QA-11-reliability-agent-consistency.md) | 조건부 | **★★★ 60→45% 하향(voting 메커니즘 반증)·②-1 보조 별점 축** |
| QA-12 | [counsel](QA-12-maintainability.md) | 채택 권장 | 정수 입도 구간화·agentic 축 보조 별점 축 |
| QA-13 | [counsel](QA-13-cost-economy.md) | 조건부 | baseline silent cap·재시도 오버헤드 보조 축·OI 등록 확인 |

## 한 줄 결론
방향·구조는 13개 전부 통과(review TL;DR). Council은 **근거 4축**을 메웠다: C1 apples 명문화(특히 QA-06 직접 injection 1차 출처 확인) · C2 margin 규칙화 · C3 보조 별점 축 이중화(QA-07·11·12·13) · C4 웹 전수 검증(τ-bench·Unit 42 정확, **InfEngine 수치 오류 정정**, voting→pass^k 메커니즘 반증). **QA-11 ★★★ = 45%로 하향 확정.**
