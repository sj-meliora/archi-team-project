# Review: QA-07 Performance — Agent 수행 시간 (per-node, round-02 재평가)

> source: `context/qa/QA-07-performance-agent-time.md` + `QAS-07-performance-agent-time.md` (round-01 [반영] 후)
> 직전 verdict(round-01): **Sound △ / KPI ✕ — High**
> verdict(round-02): **Sound ○ / KPI △ — Med** — "최대화" 폐기로 측정가능성 회복(High 해소). 단 **주 KPI가 미채택 NQA-B에 닫힘 의존** → KPI는 아직 닫히지 않음(△) · severity **Med**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 QA-07 = **[반영]** ("최대화" 폐기 → 노드타입별 speedup·first-pass 게이트·커버리지·오버헤드). 이월: NQA-B golden 게이트 전제, DP-0001 품질게이트 무연결(OI-7).

## 원문 요약 (반영 후)
- **정의**: 1-pass 성공 노드 한정, 고정 노드집합에서 노드타입별 시간 단축. 속도-품질 짝(빠르게 틀리기 차단), 비율·커버리지로 gaming 방지. Performance 3분할(QA-01/07/08) 명시.
- **KPI**: 주 = `노드타입별 speedup ≥ 3배(first-pass 성공분만)`. 보조 = `품질 게이트 = first-pass 성공률`(paired, 성공 판정 = NQA-B golden) · `커버리지 ≥ 80%` · `runner 오버헤드 비율 ≤ 15%`.
- **QAS-07**: Response·Measure가 본문과 동기화. 교차 의존(first-pass 판정 = NQA-B golden) 명시됨.

## 렌즈 1 — Agentic Workflow 전문가 관점

round-01의 세 지적이 정확히 반영됐다:
- **속도-품질 trade-off**("빠르게 틀리기") → `first-pass 성공분에만 speedup 집계` + paired 품질 게이트. agentic의 핵심 함정(빠르게 틀려 rework 폭증)을 KPI 구조로 차단 — 가장 중요한 지적이 닫혔다.
- **gaming(쉬운 노드만 자동화)** → `커버리지 ≥ 80%`. 
- **per-node altitude 과소평가** → throughput(QA-01)·E2E(QA-08)로 명시 위임, head-to-head를 적정 위치로 한정.

그러나 **반영이 새 의존을 만들었다 — 이게 round-02 핵심 쟁점:**
- "first-pass 성공"의 판정 기준이 **NQA-B(Correctness) golden 게이트에 전적으로 의존**한다. 그런데 NQA-B는 (a) 임시 ID·미채택([이월], OI-8)이고 (b) 그 golden set 구축은 [생략]/[발표 서사](실측 미실행)다. 즉 **주 KPI `speedup(first-pass 성공분)`은 분모 조건(성공 판정)이 닫히지 않아 현재로선 측정 불가**다. round-01에서 "최대화"라 측정 불가였던 것이, round-02에선 "성공 판정 게이트 부재"로 **측정 불가의 형태만 바뀐 채 잔존**한다. → KPI를 ✕에서 ○로 올릴 수 없고 **△(의존 미해소)**로 둔다.
- **agentic 고유 보완**: first-pass 성공률은 LLM 비결정성 때문에 **run마다 흔들린다**(같은 입력도 성공/실패 갈림). speedup을 "first-pass 성공분"으로만 집계하면, 비결정 성공률이 낮은 날 표본이 줄어 speedup 추정이 낙관 편향된다(생존자 편향). → speedup 옆에 **표본 수·first-pass 성공률을 함께 보고**(이미 paired 지표로 있음 — 단 "함께 보고" 의무를 명문화 권고).

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **"최대화"(optimization goal) 폐기 → 임계값(speedup ≥ 3배)**: acceptance criterion 형태 회복. 이질 노드 절대시간 차 → 노드타입별 비율. round-01 measurable 위반이 형식적으로는 교정됨. **Sound △→○** (정의가 단일 관심사 per-node로 좁혀지고 QA-01/08과 경계 명문화).

- **그러나 KPI 닫힘은 미완(△ 유지 사유 정리):**
  1. **주 KPI가 외부 미채택 QA에 닫힘 의존** — `first-pass 성공분`의 정의가 NQA-B에 있고 NQA-B 미채택. 한 QA의 acceptance가 **아직 채택되지 않은 다른 QA의 게이트**에 의존하면, 그 QA는 독립적으로 테스트 불가. round-01 applier report §3-2가 정확히 예측한 "교차 의존이 닫혔는가" 항목 — **아직 안 닫힘**. NQA-B 동반 채택이 QA-07 KPI 합격의 전제.
  2. **speedup 분모 baseline의 공정성** — 검증 전략이 스스로 인정하듯("수동 baseline은 표본·숙련도 편차가 큼 → 최대 난점"). 숙련 개발자 baseline이면 speedup이 작게, 초급이면 크게 나온다. 합격선 3배가 baseline 출처에 좌우 → **baseline 측정 프로토콜(숙련도·표본 고정)을 KPI 옆에 명시**해야 acceptance가 재현 가능.

- **consistency(QA↔QAS↔정의)**: QAS-07 Measure가 본문 4 KPI와 일치, 교차 의존도 QAS 비고에 기록됨 → ○.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

- **runner 오버헤드 분리**(round-01 권고) → `runner 오버헤드 비율 ≤ 15% = (큐 대기+dispatch+cold start)/노드 총시간`으로 반영. OTel GenAI span 분해로 측정 — 개선 레버를 드러내는 좋은 SLI. **반영 양호.**
- **병렬성이 진짜 가치**(round-01 렌즈3) → throughput(QA-01)·parallelism으로 위임 명시. per-node QA가 병렬성 가치를 과소평가하지 않게 분리됐다.
- **잔여(DP 위임, OI-7)**: 검증 전략이 주 KPI를 **DP-0001 1안 즉시 실행**에 귀속시키나, OI-7이 명시하듯 **DP-0001은 latency만 보고 품질 게이트와 무연결**이다. 즉 "first-pass 성공분만 집계"라는 KPI의 핵심 장치를 **책임지는 DP가 없다.** runner 측에서 first-pass 성공/실패 라벨을 어디서 받아 speedup 집계에서 제외하는지(게이트 hook)가 DP에 미명시 → DP 디스커션 위임.
- **runner 측 KPI**(재확인): 스케줄링 오버헤드/activity 시간 비율(반영됨=오버헤드 15%), 큐 대기 p95, 평균 동시 실행도, **first-pass 게이트 처리율**(성공 판정 hook의 throughput — 미반영, 게이트 DP 신설 시 추가).

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | △ | **○** | per-node로 altitude 고정 + 품질·커버리지 조건 명시 + QA-01/08 경계 명문화 |
| KPI가 측정 가능한가 | ✕ | **△** | "최대화" 폐기로 임계값화는 됐으나, 주 KPI `first-pass 성공분`이 **미채택 NQA-B 게이트에 닫힘 의존** → 독립 측정 불가 잔존 |
| KPI가 현실적/적절한가 | ✕ | **○** | 품질 게이트·커버리지로 gaming/빠르게-틀리기 차단. 단 baseline 공정성·비결정 표본편향은 Med 보강 |
| 정의↔KPI↔QAS 일치 | △ | **○** | QAS-07 Measure·교차 의존이 본문과 동기화 확인 |

**verdict 변화: Sound △→○ / KPI ✕→△ · severity High→Med.** High(완전 측정 불가)는 해소됐으나, **KPI 닫힘이 NQA-B 채택에 걸려 있어** ○까지는 못 올린다 — applier report §3-2 "교차 의존이 닫혔는가"의 답은 **아직 아니오**.

## Stage 2 권고 (round-02)

- **(교차 의존 닫기 — 최우선, OI-8)** **NQA-B 동반 채택 없이는 QA-07 주 KPI가 측정 불가.** "first-pass 성공 판정 = NQA-B golden"이 살아 있는 한, NQA-B를 정식 QA로 승격하고 양방향 cross-link을 확정해야 QA-07 KPI가 ○로 닫힌다. 미채택 유지 시 QA-07 KPI는 △ 고정.
- **(DP 디스커션 위임, OI-7)** DP-0001 1안에 **first-pass 품질 게이트 hook**(성공/실패 라벨을 speedup 집계에서 분리하는 tactic) 명시. 현재 KPI 핵심 장치의 설계 귀속이 비어 있음.
- **(baseline 프로토콜 명시, Med)** `노드타입별 speedup ≥ 3배` `기존(baseline 출처 미정)` → `제안: baseline = 숙련도·표본 고정 조건의 수동 수행시간 중앙값, 측정조건을 슬라이드에 명시`. 합격선 3배가 baseline에 좌우되므로 프로토콜이 acceptance의 일부.
- **(비결정 표본편향 가드, Low)** speedup 보고 시 **표본 수·first-pass 성공률 동반 보고를 의무화** — 성공분만 집계하는 생존자 편향을 드러냄(paired 지표는 이미 있으니 "동반 보고" 규칙만 추가).
- **(예시값 확정)** `3배·80%·15%`는 노드타입별 baseline 측정으로 확정(round-01부터의 미결 — 발표 전 팀 합의).
