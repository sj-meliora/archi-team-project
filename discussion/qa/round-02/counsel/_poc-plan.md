# PoC 통합 계획 — round-02 counsel

> round-02 PoC 의존그래프. 방법론: [`../../Council.md`](../../Council.md) §5 · 종합: [`counsel.md`](counsel.md) · 직전: [round-01/counsel/_poc-plan.md](../../round-01/counsel/_poc-plan.md)
> date: 2026-06-24 · 원칙: 작게 시작(앵커 50~100, 부하 수십 VU), 레퍼런스 수치 복제 금지(합격선 ◯ = PoC로 확정), 각 PoC silent cap 명시.
> ⚠️ **동작 시스템 부재 → round-02 실측 0건.** 본 계획은 round-01 PoC 목록을 승계해 **[발표 서사]로 미룬 것의 실측 전환 우선순위**를 정리한 "검증 전략 개요"(발표용)다. round-01 PoC 17개 목록·아키타입 표는 [round-01 _poc-plan](../../round-01/counsel/_poc-plan.md) 유지 — 여기선 round-02 변화분(실측 전환·동반 닫힘 의존)만 갱신.

## round-02 핵심 = 실측 전환 대상 (round-01 대비 변화)

round-01은 "KPI를 측정 가능한 형식으로" 만들었고, round-02는 그중 **[발표 서사]로 미룬 검증을 실측 acceptance로 전환**하는 게 닫힘 행동이다. 전환 대상:

| PoC | 증명 대상 | round-01 상태 | round-02 전환 | 동반 닫힘 |
|---|---|---|---|---|
| **PoC-N-B1** | golden+judge 정답률 + **judge↔인간 일치도 SLI** | [발표 서사] | **실측 전환(허브)** | → QA-07 주 KPI · QA-09 ②-2 · NQA-C 성공분 비용 |
| **PoC-C2 ≡ N-A1** | 적대적 red-team(위반0·injection·서명·secrets) | [생략]/[발표 서사] | **실측 전환(공유 하네스)** | QA-03 ② ≡ NQA-A 4축 (단일 하네스) |
| PoC-P1 | first-pass 게이트로 정규화한 speedup + baseline 프로토콜 | N-B1 후행 | N-B1 출력 수급 | QA-07 닫힘 = NQA-B 채택 |
| PoC-K1 | 캐시 우회 Δ 대조·pass^k·H_norm·②-1 | 일부 닫힘 | ②-1 독립 / ②-2만 N-B1 후행 | QA-09 부분 닫힘 |
| PoC-N-C1 | $/완료모델 분해·절감률 + baseline 가정 명시 | [발표 서사] | O1 토큰 계측 후행 | QA-01 활용률·QA-05 top-line 부유 해소 |

> 나머지 PoC(S1·S2·A1·A2·O1·R1·E2E1·E1·C1·M1)는 round-01 합격선·아키타입 유지 — 닫힘 확인 7항목(QA-01·02·04·05·06·08·10)은 measurable이 닫혀 PoC 구조 변화 없음.

## round-02 의존 그래프 (동반 닫힘 트리거 명시)

```
[토대 — 먼저 깔아야 측정됨]
PoC-O1 (OTel GenAI span + event-history)
   ├─→ PoC-N-C1 (토큰 계측 gen_ai.usage.* 재사용)
   ├─→ PoC-E1   (캐싱 $/task 계측)
   └─→ PoC-P1   (runner 오버헤드 span 분해)

[허브 1 — 정확성 게이트가 세트 닫힘 트리거 (C1)]
PoC-N-B1 (golden set + judge + judge↔인간 일치도 SLI)
   ├─→ PoC-P1  (QA-07 first-pass 성공 판정 = NQA-B 게이트)  ── 분모 통째 의존(닫힘도 최저)
   ├─→ PoC-K1  (QA-09 ②-2 정밀 유효-결정률 = NQA-B 게이트)  ── ②-1로 부분 닫힘(②-2만 의존)
   └─→ PoC-N-C1 (성공분만 비용 집계)
   ※ NQA-B 정식 채택(OI-8) 없이는 위 3개 acceptance 미닫힘 — PoC 가능해도 채택이 전제

[허브 2 — 공유 red-team 하네스 (C3)]
PoC-C2 (QA-03 위반0·runaway cap)  ≡  PoC-N-A1 (NQA-A 5축 보안)
   └─ 단일 red-team 하네스(OWASP LLM Top-10·SLSA 매핑)로 QA-03 ②·NQA-A 동반 산출

[durable 위에 — round-01 유지]
PoC-A1·A2 (chaos) ── event-history가 O1과 동일 엔진 / PoC-K1 버전 핀닝 공유

[독립 부하시험 — 병렬, round-01 유지]
PoC-S1·S2 (QA-01) ‖ PoC-E2E1 (QA-08) ‖ PoC-R1 (QA-06) ── 같은 부하 하네스(k6/Locust) 공유
PoC-E2E1 ──→ DP-0004 A5 vs A8 택일 산식 데이터 공급
```

## 핵심 교차 의존 (round-02 동반 닫힘 — 명시 요구사항)

- **C1 / QA-07 ↔ NQA-B**: PoC-P1 first-pass 성공 판정 = PoC-N-B1 출력 → **NQA-B 선행 + 정식 채택이 QA-07 주 KPI 닫힘 전제**(닫힘도 최저 — 분모 통째 의존).
- **C1 / QA-09 ②-2 ↔ NQA-B**: PoC-K1 ②-2 = PoC-N-B1 의존. 단 ②-1(룰 게이트)·Δ·H_norm은 **N-B1 없이 독립 닫힘** → QA-09 부분 닫힘(QA-07보다 닫힘도 높음).
- **C2 / QA-01·QA-05 ↔ NQA-C**: PoC-N-C1이 살아남아야 이양 KPI(활용률·top-line)가 부유 안 함 — **NQA-C 채택이 닫힘 트리거**(게이트 아닌 이양처).
- **C3 / QA-03 ≡ NQA-A**: PoC-C2 = PoC-N-A1 — **단일 red-team 하네스**로 두 QA 동반 닫힘.
- **토대 / QA-04 → 비용·성능**: PoC-O1 토큰 계측(`gen_ai.usage.*`)을 N-C1·E1·P1이 재사용 → **QA-04 계측 선행**.

## 우선순위 (round-02 — 실측 전환 중심)

**Group 1 (토대 + 허브 — 세트 닫힘 전제)**
1. PoC-O1 (관측 계측) — 토큰·span을 비용·성능 PoC가 재사용.
2. **PoC-N-B1 (정확성 게이트 + judge 일치도 SLI)** — QA-07·QA-09 ②-2·NQA-C를 닫는 허브. **OI-8 채택 동반이 acceptance 전제.**
3. **PoC-C2 ≡ N-A1 (공유 red-team 하네스)** — QA-03 ②·NQA-A 동반. OWASP LLM Top-10·SLSA 매핑.

**Group 2 (허브 후행 — 동반 닫힘 실현)**
4. PoC-P1 (QA-07 speedup, N-B1 후행 + baseline 프로토콜).
5. PoC-K1 (QA-09, ②-1 독립 / ②-2 N-B1 후행).
6. PoC-N-C1 (NQA-C 비용 분해, O1 후행 + baseline 가정 명시).

**Group 3 (닫힘 확인 항목 — round-01 합격선 유지)**
7. PoC-S1·S2(QA-01)·A1·A2(QA-02)·R1(QA-06)·E2E1(QA-08)·E1(QA-05)·C1(QA-03 latency)·M1(QA-10).

## 공유 자산 (한 번 만들어 여러 PoC가 사용)
- **golden set + LLM-as-judge(+ 인간 일치도 검증)**: N-B1 → P1·K1(②-2)·N-C1. **judge↔인간 일치도 = 게이트 신뢰 SLI(round-02 신규)**.
- **적대적 red-team 세트(OWASP LLM Top-10·SLSA)**: C2 ≡ N-A1.
- **OTel GenAI 계측**: O1 → E1·N-C1·P1.
- **부하 하네스(k6/Locust + mock 4단계)**: S1·S2·E2E1·R1.
- **durable 엔진(event-history·버전 핀닝)**: A1·A2·O1·K1.
- **결정 판정 게이트(룰 체커)**: K1 ②-1 = H_norm 라벨 매핑 동일 컴포넌트(인프라 절약).

## 총 기간 (러프 추정 — round-01 정합)
- Group 1: 약 1.5~2주(golden set 4단계 라벨 + red-team 세트 구축이 최장 critical path).
- Group 2: 약 1주(허브 후행, 공유 자산 재사용으로 단축).
- Group 3: 약 1.5주(닫힘 확인 — round-01 합격선 그대로).
- **합계 약 4~5주**(round-01 5~6주 대비 단축 — 신규 PoC 없이 실측 전환 중심, 공유 자산 재사용).

## PoC가 못 보는 것 (round-02 전역 silent cap)
- **동작 시스템 부재 → 실측 0건**: 본 계획은 검증 전략 개요. 합격선 ◯는 PoC 실행 시 확정.
- **채택 ≠ PoC**: OI-8 정식 채택은 사람 결정 — PoC가 측정 가능을 보여도 채택을 대체 못함(NQA-A/B/C 공통).
- **eval/golden/red-team 커버리지 = 신뢰 상한**: 미상상 케이스 미검출(QA-03 ②·NQA-A zero-day·NQA-B golden 미포함·QA-09 유사입력).
- **judge 자체 편향**: judge↔인간 일치도 검증이 전제(미달이면 정답률 무효).
- **baseline 추정 의존**: QA-07 speedup·NQA-C 절감률이 수작업 baseline 가정에 좌우 — 슬라이드 가정 명시 필수.
- **mock 파이프라인 현실성·외부 시스템 통합·self-host vs 외부 API**: round-01 silent cap 승계(실환경 의존).
- **eval/검증 서브시스템 DP 부재**: PoC 가능해도 운영 귀속이 비어 있음(OI-7 — DP 디스커션 선행 필요).
