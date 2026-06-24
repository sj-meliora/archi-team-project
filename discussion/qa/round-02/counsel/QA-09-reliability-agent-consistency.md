# Counsel: QA-09 Reliability — Agent 결과 일관성 (round-02)

> refs-review: [round-02/review/QA-09-reliability-agent-consistency.md](../review/QA-09-reliability-agent-consistency.md) · [report.md](../review/report.md)(C1) · contention: [round-01/contention/counter.md](../../round-01/contention/counter.md)
> 직전 counsel: [round-01/counsel/QA-09-reliability-agent-consistency.md](../../round-01/counsel/QA-09-reliability-agent-consistency.md) (채택 권장 — 캐시우회 pass^k + 유효-결정률)
> seats: 발의 **Seat 1**(Agentic Workflow) · 합의 **consensus** (Seat 2 H_norm·② 2단 / Seat 3 결정 판정 게이트 재사용)
> stance: **닫힘 확인(contention으로 R1~R3 해소·부분 닫힘) + ②-2만 NQA-B 동반 채택 전제(OI-8)**

## Reviewer 지적 요약

- round-02 verdict: **Sound ○ / KPI △ · Med.** round-01 review→반영→**contention 1왕복**까지 거친 유일 항목. 결정성 폐기·유효-결정 안정성 재프레이밍으로 Sound 회복 + **contention이 헤드라인(캐시 On/Off 갭 Δ 대조)·② 2단 분리·H_norm 정의로 R1~R3 해소**.
- **C1 잔여(비대칭)**: ②-2 정밀 유효-결정률만 미채택 NQA-B golden에 닫힘 의존 → open-issue 강등(정직, placeholder 아님). ②-1 대리 게이트(룰 체커, golden 불요)는 이미 닫힘 → QA-09는 **부분 닫힘**(QA-07보다 닫힘도 높음).
- review 권고: NQA-B 채택 시 ②-2 닫힘(OI-8), eval/검증 서브시스템 DP(OI-7), importance 상향은 선결 2조건 충족 후 사람 결정(contention 게이트).

## 개선안 (정의·KPI 기존→제안)

Council 응답: **QA-09는 contention으로 헤드라인이 이미 [발표 서사]를 탈출했다**(Δ 대조는 방법론 자체가 시연). 남은 일은 ②-2를 NQA-B 채택으로 닫는 것뿐 — QA-09 자체에서 새로 발명할 KPI 없음. Sound ○·정의 유지.

**정의: 기존 → 제안 (유지)**
- 기존: 동일/유사 입력에서 유효(정책 허용) 결정의 안정성. 캐시 우회 측정. NQA-B(정확성)와 짝. 유지.
- 제안: ②-2↔NQA-B 양방향 cross-link 확정(채택 시). 변경 없음.

**KPI: 기존 → 제안 (contention 결과 유지)**

| # | 기존 (contention 후) | 제안 (round-02) | 닫힘 상태 |
|---|---|---|---|
| 주 | 캐시 On/Off 일관성 갭 Δ 대조 시연 | 동일 — **헤드라인은 값이 아니라 방법론(대조)**이라 시연 자체가 acceptance. [발표 서사] 탈출 확정 | ○ (closed) |
| 보조 pass^k | 순수 추론 pass^k(k=5, 예시 ≥70%) | 동일 — *절대값*만 [발표 서사] 보조(헤드라인 아닌 가드레일이라 정합성 유지) | △(가드레일) |
| 보조 ②-1 | 대리 유효-결정률 ≥95% (룰 게이트, golden 불요) | 동일 — 룰 체커로 우리 조건에서 산출 가능 | ○ (closed) |
| 보조 ②-2 | 정밀 유효-결정률 (NQA-B 의존, open-issue) | 동일 — **NQA-B golden 게이트 채택+실측이 닫힘 전제**. "무효 아님"(②-1) 넘어 "정답인가"는 NQA-B 없이 불가 | △→○ (**NQA-B 채택 시**) |
| 보조 H_norm | 재실행 분산 H_norm ≤0.2 | 동일 — 정규화 Shannon 엔트로피 H_norm=−Σp·log p/log\|C\| ∈[0,1], 라벨 매핑은 ②-1 게이트와 동일 컴포넌트 재사용 | ○ (closed) |

> ⚠️ `Δ 대조·pass^k 70%·②-1 95%·H_norm 0.2`는 "측정 가능 KPI의 모양" 예시값 — 캐시 우회 반복시행으로 확정.

## 근거 (레퍼런스)

§4 라이브러리 **일관성(Consistency)** + **정확성(②-2 게이트)** 교차 적용.

- **pass^k**(k회 i.i.d. 시행의 일관성; pass@1 90%도 k=8엔 57%로 급락) — 캐시 우회 측정. 비결정 일관성의 엄격 척도. 헤드라인을 절대값이 아닌 캐시 On/Off **대조**로 옮긴 것이 contention의 핵심(가상 설계라 절대값 실측 불가).
  - τ-bench pass^k: https://arxiv.org/abs/2406.12045 · https://sierra.ai/blog/tau-bench-shaping-development-evaluation-agents
- **정규화 Shannon 엔트로피 H_norm** — 재실행 분산을 척도로 정의("0.2가 무엇의 0.2인지" 해소). 출력 클래스 분포의 불확실성을 [0,1]로 정규화.
  - Shannon entropy(정보이론 표준): https://en.wikipedia.org/wiki/Entropy_(information_theory)
- **버전 핀닝 + 캐시 우회 모드(durable)** — model/prompt/tool 버전 핀닝으로 동일 조건 보장, memoization On vs 우회 대조가 "캐시는 일관성 입증 아님" 명제를 시연.
  - Temporal durable execution(버전 핀닝): https://docs.temporal.io/temporal
- **②-2 정밀 유효-결정 = golden 게이트** — "무효 아님"(②-1 룰) 넘어 "정답"은 golden+judge 필요(NQA-B 공유).
  - golden·LLM-as-judge: https://montecarlo.ai/blog-llm-as-judge/

> 레퍼런스 수치(70%·0.2 등)는 패턴 정당화용 — 합격선은 PoC로 확정.

## PoC 증명법

### PoC-K1(R2, ②-2는 NQA-B 후행): 캐시 우회 반복시행으로 Δ 대조·pass^k·H_norm·②-1을 측정한다 (반복시행)

- **가설(Hypothesis)**: "캐시 우회 k회 반복으로 캐시 On/Off 일관성 갭 Δ가 시연 가능하고, pass^k·H_norm·②-1 대리 유효-결정률이 산출된다. ②-2 정밀 유효-결정률만 PoC-N-B1 golden 게이트에 의존한다."
- **지표(Metric) + 합격선**:
  - **캐시 On/Off pass^k 갭 Δ**(헤드라인 — 대조 시연 자체)
  - 순수 추론 pass^k(k=5) · H_norm ≤ ◯ · ②-1 대리 유효-결정률 ≥ ◯%
  - (②-2는 PoC-N-B1 게이트 출력 의존 — 동반 닫힘 표시)
- **셋업(Setup)**: 고정 작업셋 + model/prompt/tool 버전 핀닝(durable 엔진) + **캐시 우회 모드**(memoization On vs 우회). ②-1 룰 게이트 = H_norm 라벨 매핑과 동일 컴포넌트 재사용. ②-2는 PoC-N-B1 golden 게이트.
- **절차(Procedure)**:
  1. 버전 핀닝 후 캐시 On / 우회 각각 k회 반복 → pass^k 갭 Δ 대조
  2. 출력 분포로 H_norm 산출 + 룰 게이트로 ②-1 대리 유효-결정률
  3. **②-2는 PoC-N-B1 게이트 출력으로 산출**(NQA-B 채택 시 동반 닫힘)
- **합격 기준(Exit)**: Δ 대조 시연(캐시 치트 입증) ∧ H_norm·②-1 측정 가능 ∧ **②-2가 NQA-B 게이트 출력으로 닫힘**(C1 동반 닫힘).
- **규모/기간(Scope)**: 버전 핀닝·캐시 우회 ②-1까지 독립 가능(약 2일), ②-2만 PoC-N-B1 후행.
- **리스크/한계(silent cap)**: "유사 입력" 흔들림은 범위 밖(같은 입력 일관성만). pass^k 절대값은 가상 설계라 시연 불가([발표 서사] 가드레일). **②-2 닫힘은 NQA-B 채택(OI-8) 의존** — PoC가 채택을 대체 못함.

## DP·발표 영향

- **DP 연결 (OI-7, DP 디스커션 위임)**: **eval/검증 서브시스템(NQA-B golden + 결정 판정 게이트) DP 신설** 검토 — ②-2·H_norm·②-1 모두 이 게이트 컴포넌트 의존(현재 어떤 DP에도 없음). ②-1 룰 게이트와 H_norm 라벨 매핑이 단일 컴포넌트라 인프라 절약 — DP에 "결정 판정 게이트" 1개로 명시 권고.
- **동반 닫힘 효과**: NQA-B 채택 시 ②-2(△→○) 닫힘. 단 **②-1로 이미 부분 닫힘**이라 QA-07(주 KPI 통째 의존)보다 NQA-B 의존도 낮음 — contention의 비대칭 효과.
- **번호/서사 (importance)**: contention 게이트 선결 2조건(헤드라인 Δ 재정의 + ②-1 게이트 정의) 충족 → **importance 상향(L→M)은 이제 사람 결정 가능**. NQA-B와 묶으면 "일관(QA-09) + 정확(NQA-B)" 자율 신뢰 토대라 상향 여지(OI-8 트랙, verdict 무관).
