# round-03 counsel — QA 등급 척도(★ rubric) 캘리브레이션

> 날짜: 2026-06-24 · 대상: QA-01~13 주 KPI(헤드라인) · 유형: **reviewer-less 변형**(red-team Reviewer 없이 **팀이 reviewer 자리**, Council 3 seats가 캘리브레이션). 방법론: [`../../Council.md` §9](../../Council.md).
> 목적: ATAM의 QA 간 trade-off(동일 조건 두 아키텍처 안 비교 → ★ 많은 안 채택)를 위해 각 주 KPI를 ★1~3(상/중/하) 급간화. KPI 합격선=★☆☆ 진입선, ★★☆/★★★는 **실제 필드 벤치마크 + PoC margin**으로 캘리브레이션.
> 산출 반영: 13개 QA 파일에 `## 등급 척도` 섹션 신설 + §측정 하한 보정(옛값 보존 트레이스). changelog 2026-06-24 항목·OI-9 참조.

## 5대 규칙 (Council.md §9)
1. KPI 하한 = ★☆☆ 진입선, 미만 불합격. ★★☆/★★★는 필드 현실 도달 범위(웹 근거).
2. 비현실 하한(★★★ 사문화)·보수 하한(★☆☆ 사문화)은 표 급간을 현실 대역으로 **재배치**(옛값 보존).
3. 이론 천장에 빡빡하게 붙이지 말고 **PoC 미완성 대비 margin**을 얹어 다소 낮게/넓게.
4. 2-index KPI는 PoC 측정 가능한 쪽을 **main 급간 축**, 나머지는 `조건/게이트`로 고정.
5. 보조지표가 또 0건/위반 절대형이면 QA 아니라 Constraint — gradable proxy 있을 때만 QA 급간.

## 13개 QA 등급 척도 요약

| QA | main 급간 축 | ★★★ | ★★☆ | ★☆☆(합격 하한) | 조건/게이트 | 하한 보정 |
|---|---|---|---|---|---|---|
| 01 Scalability | scaling efficiency↑ | ≥0.85 | 0.75~0.85 | 0.70~0.75 | 헤드룸≥20% 고정 | 0.8→0.70 |
| 02 Availability | failover 시간↓ | ≤10s | ≤1분 | ≤4분 | 게이트: 손실0·멱등100% | 1분→4분 |
| 03 Controllability | graceful stop+롤백↓ | ≤15s | ≤30s | >30s+hard-kill | 게이트: HITL100·runaway100·ack≤5s | 30s 흡수 |
| 04 Observability | 일반 trace 완전성↑ | ≥98% | 95~98% | ≥95% | 게이트: 안전100·event-hist100 | 상한 캡 |
| 05 Efficiency | 신규 토큰↓ | ≤4k | 4~6k | 6~8k | 조건: 캐시적중≥85% | 불요(정식화) |
| 06 Security | injection 차단율↑ | ≥90%+FPR≤1% | 80~90% | 70~80% | 게이트: 범위외배포=0(제약성) | 95→70% |
| 07 Correctness | golden 정답률↑ | 92~98% | 90~92% | ≥90% | 조건: judge κ≥0.8·100%=불합격 | 상한 캡+조건 |
| 08 Reliability-WF | 타WF latency 증가↓ | ≤5% | ≤10% | ≤25% | 조건: 중단율≤1%·쿼터침범0 | 10→25% |
| 09 Perf per-node | speedup 배수↑ | ≥8배 | 3~8배 | 1~3배 | 조건: first-pass게이트·커버≥80% | 3배→1× |
| 10 Perf E2E | E2E latency p95↓ | ≤2h | ≤6h | ≤24h | 조건: throughput≥50/일·전달무결성100% | 6h→24h |
| 11 Reliability-일관성 | 순수추론 pass^5↑ | ≥60% | ≥40% | ≥25% | 게이트: Δ시연·H_norm≤0.2·②-1≥95% | 70%→25/40/60 |
| 12 Maintainability | CIS p95↓ | ≤1 | =2 | =3 | 조건: 컴포넌트경계정의 | 불요 |
| 13 Cost-economy | 절감률↑ | ≥75% | 60~75% | 50~60% | 게이트: $/모델≤$5 | 70%→50% |

(↑=높을수록 상, ↓=낮을수록 상. 전 경계 예시값 — PoC로 확정. 각 QA `## 등급 척도`에 필드 근거 URL·seats·캘리브레이션 노트 상세.)

## 핵심 발견 (필드 캘리브레이션)
- **비현실적으로 높았던 하한**(★★★ 사문화 위험): QA-06 injection 95%(상용 53/91/92%·연구 94~95%) · QA-11 pass^k 70%(τ-bench frontier도 pass^5 40~60%대) · QA-13 절감률 70%(자동화 절감 30~75%) · QA-01 efficiency 0.8(USL 우수도 ~0.72).
- **오히려 보수적이던 하한**(★☆☆ 사문화): QA-10 E2E ≤6h(배치 ML 관례 6~24h).
- **방향 유지·소폭**: QA-09 ≥3배(headless 완전자율이라 유지·★☆☆만 1×로) · QA-02 1분→4분(stateful 워크플로우) · QA-08 10→25%.
- **상한 캡 신설**: QA-04 trace ≥98%(CoT 비공개로 100% 불가) · QA-07 정답률 92~98%(100%=난이도 부족 신호).

## 후속
- **OI-9**: 하한 보정 8건 ↔ 짝 QAS-* Measure·glossary 수치 정합 점검(미반영).
- 등급 경계는 전부 예시값 — 각 QA PoC(시뮬·A/B·chaos·eval 하네스)로 실측 확정.

## seats
Seat 1 Agentic Workflow · Seat 2 20년차 수석 아키텍트 · Seat 3 대규모 Workflow Runner 인프라 — 13개 QA 전 항목 consensus(초기 dissent 2건[QA-01 조건부 efficiency·QA-06 FPR]은 규칙4 main+조건 구조로 흡수돼 consensus 전환).
