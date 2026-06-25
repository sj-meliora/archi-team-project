# round-04 review — 내비게이션

> red team(Reviewer) 산출물. round-03 ★ 등급 척도(KPI 급간) 캘리브레이션의 **red-team 검증** 라운드.
> 방법론: [`../../Reviewer.md`](../../Reviewer.md) · 공통 프로토콜 [`../../../README.md`](../../../README.md).

## 읽는 순서

1. **[report.md](report.md)** — 이것만 읽으면 라운드 전모(판정표 13행·교차발견 C*·★ 급간 종합·직전 대비 변화). **메타·날짜 여기.**
2. 개별 QA 3렌즈 상세 (각 4축 판정 + Stage 2 권고, ★ 급간 검증 무게중심):
   - [QA-01](QA-01-scalability.md) Scalability · [QA-02](QA-02-availability.md) Availability · [QA-03](QA-03-controllability.md) Controllability
   - [QA-04](QA-04-observability.md) Observability · [QA-05](QA-05-efficiency.md) Efficiency · [QA-06](QA-06-security-safety.md) Security
   - [QA-07](QA-07-correctness.md) Correctness · [QA-08](QA-08-reliability-workflow.md) Reliability-WF · [QA-09](QA-09-performance-agent-time.md) Perf per-node
   - [QA-10](QA-10-performance-e2e.md) Perf E2E · [QA-11](QA-11-reliability-agent-consistency.md) Reliability-일관성 · [QA-12](QA-12-maintainability.md) Maintainability · [QA-13](QA-13-cost-economy.md) Cost-economy
3. [_new-qa-candidates.md](_new-qa-candidates.md) — 신규 QA 후보(**없음** — 본 라운드는 ★ 급간 보정이 주 산출).

## 입력 (이 라운드가 받은 것)
- round-03 [counsel.md](../../round-03/counsel/counsel.md) — ★ 급간 설계 의도(reviewer-less 캘리브레이션, round-03엔 applier 없음).
- round-02 [applier/report.md](../../round-02/applier/report.md) §3 — 직전 disposition.
- `context/qa/QA-*.md` + `QAS-*.md` 13쌍 · overview · glossary.

## 샘플 → full
- 샘플(팀 확인): QA-06·QA-11(가장 공격적 재배치).
- full(이 폴더): 나머지 11개 + C* 전체 + NQA + 인덱스 갱신.
