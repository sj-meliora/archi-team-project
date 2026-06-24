# Counsel: NQA-B Correctness / Accuracy — 산출물 정확성 (round-02)

> refs-review: [round-02/review/NQA-B-correctness.md](../review/NQA-B-correctness.md) · [report.md](../review/report.md)(C1·C3·Stage2-1) · 동반: [QA-07](../review/QA-07-performance-agent-time.md)·[QA-09](../review/QA-09-reliability-agent-consistency.md)
> 직전 counsel: [round-01/counsel/NQA-B-correctness.md](../../round-01/counsel/NQA-B-correctness.md) (신설 — 채택 권장)
> seats: 발의 **Seat 1**(Agentic Workflow) · 합의 **consensus** (Seat 2 ISO 앵커·직교성 / Seat 3 eval 서브시스템 DP·trace 수급)
> stance: **정식 채택 강력 권장 (OI-8 최우선) — 세트 닫힘의 단일 트리거**

## Reviewer 지적 요약

- round-02 verdict: **Sound ○ / KPI △ · severity High.** 신설 내용·ISO 앵커는 완성도 높아 정식화 강력 권장. 단 **KPI 검증(golden+judge)이 [발표 서사]·`related-dp: []`**라 닫히지 않음.
- High 사유는 *단일 QA 결함이 아니라 세트 병목*: **QA-07 주 KPI(first-pass 성공 판정)·QA-09 ②-2(정밀 유효-결정률)가 NQA-B golden 게이트에 닫힘 의존** → NQA-B가 안 닫히면 두 QA도 △ 고정. review C1 = "교차 의존이 닫혔는가 → 아직 아니오". **NQA-B 정식 채택 + golden 실측 전환이 QA-07·QA-09 동반 닫힘의 단일 행동**(report Stage2-1).
- 잔여: ① 정식 채택·번호 재정렬·양방향 cross-link 확정(OI-8), ② eval/검증 서브시스템 DP 신설(OI-7, DP 디스커션 1순위), ③ judge↔인간 일치도를 게이트 신뢰 SLI로 노출, ④ `90%·10%·5%` 예시값 golden set으로 확정.

## 개선안 (정의·KPI 기존→제안)

Council의 round-02 응답은 **새 KPI 발명이 아니라 닫힘 메커니즘 확정**이다 — Reviewer가 "측정 형식은 갖췄으나 검증이 서사"라 판정했으므로, 권고의 핵심은 *어떻게 [발표 서사]를 실측 acceptance로 전환하고 동반 닫힘을 트리거하는가*다.

**정의: 기존 → 제안 (유지 + cross-link 확정)**
- 기존: "에이전트 자동 산출물이 정답·정책에 부합" + QA-09 직교 명시 — Sound ○이므로 **정의 본문은 유지**.
- 제안: 정식 채택과 동시에 **QA-07·QA-09 ②-2와 양방향 cross-link 확정**(현재 NQA-B→QA-07/09 단방향만 존재, 채택 시 역방향 명문화). `related-dp: []` → 신설 "eval/검증 서브시스템" DP로 귀속(OI-7, DP 디스커션 산출 대기).

**KPI: 기존 → 제안**

| # | 기존 (round-01 반영) | 제안 (round-02) | 닫힘 상태 |
|---|---|---|---|
| 주 | golden-set 정답률 ≥ 90% (단계별, [발표 서사]) | **동일 — 단 PoC-N-B1로 "측정 가능·judge 신뢰"를 실측 전환**(예시값은 golden으로 확정). 단계별(IR/Optimize/Quant/Compile) 정답률 분해 명문화 | △→○ (PoC 통과 시) |
| 보조1 | rework율 ≤ 10% (QA-07 공유) | 동일 — QA-07 first-pass 게이트와 **공유 라벨 공급률**로 측정 귀속 명시 | △ |
| 보조2 | 회귀 미검출률 ≤ 5% | 동일 — 알려진 회귀 케이스 주입 → 게이트 누수율 | △ |
| **신규 보조3** | (없음) | **judge↔인간 일치도 ≥ ◯%(목표 75~90%) — 게이트 신뢰 자체의 SLI**(review 신규 권고). 정답률의 신뢰 상한을 노출: judge가 틀리면 정답률이 왜곡되므로 일치도가 곧 헤드라인 KPI의 메타-신뢰도 | 측정 가능(PoC 절차 ②) |

> ⚠️ `90%·10%·5%·일치도`는 모두 "측정 가능 KPI의 모양" 예시값 — golden set 실측으로 확정(레퍼런스 복제 아님).

## 근거 (레퍼런스)

§4 라이브러리 **정확성(Correctness)** 행 직접 적용 + ISO 앵커 보강.

- **golden dataset(앵커 50~100) + LLM-as-judge(다수결, 인간 일치 75~90%)** — 정확성 측정의 필드 표준. 100% 통과면 난이도 부족 신호. (§4 정확성)
  - https://www.getmaxim.ai/articles/building-a-golden-dataset-for-ai-evaluation-a-step-by-step-guide/
  - https://www.comet.com/site/blog/llm-as-a-judge/ · https://montecarlo.ai/blog-llm-as-judge/
- **ISO/IEC 25010:2023 — Functional Suitability / Functional Correctness**: "Correctness"가 ISO 하위특성명 그 자체라 정식화 앵커가 가장 명확(golden 커버리지는 보조적으로 Functional Completeness). review 렌즈2 = "1급 정식화 자격 충분". 출처: https://iso25000.com/index.php/en/iso-25000-standards/iso-25010
- **judge 신뢰 = 게이트 신뢰의 SLI**: LLM-as-judge는 judge↔인간 일치도를 *먼저* 검증해야 정답률이 신뢰 가능 — 일치도를 보조 KPI로 노출하라는 review 신규 권고는 필드 표준과 정합(judge 편향이 정답률을 왜곡하는 silent cap 방어). (위 comet·montecarlo)

> 레퍼런스 수치(정답률 90%·일치도 75~90%)는 **패턴 정당화용** — 우리 합격선은 PoC-N-B1 golden set으로 확정.

## PoC 증명법

### PoC-N-B1(R2 확장): golden set + LLM-as-judge로 정확성을 측정하고 동반 닫힘을 트리거한다 (eval 하네스)

- **가설(Hypothesis)**: "단계별 golden set과 (인간 일치 검증된) LLM-as-judge로 정답률·rework·회귀가 측정 가능하며, **그 게이트가 QA-07 first-pass 라벨·QA-09 ②-2 유효-결정 판정을 동반 공급**한다."
- **지표(Metric) + 합격선**:
  - 단계별 정답률 (측정 가능 ∧ 100% 아님 = 적정 난이도)
  - **judge↔인간 일치도 ≥ ◯%**(목표 75~90% — 이게 충족돼야 정답률이 신뢰 가능; 게이트 신뢰 SLI)
  - rework율 · 회귀 미검출률 (분리 집계)
  - **동반 닫힘 검증: 동일 게이트가 QA-07 first-pass 성공/실패 라벨과 QA-09 ②-2 정밀 유효-결정 판정을 출력**(공유 컴포넌트 1개로 3 QA에 공급)
- **셋업(Setup)**: 단계별 golden set(IR/Optimize/Quant/Compile 각 앵커 50~100, 사람 라벨) + LLM-as-judge(다수결) + 인간 일치도 검증 샘플. 산출물·결정 데이터는 **QA-04 trace에서 공급**(세트 의존 닫힘). QA-03/NQA-A red-team 하네스와 인프라 공유.
- **절차(Procedure)**:
  1. golden set 구축·사람 라벨 (단계 4종, IR/Optimize/Quant/Compile 난이도 상이 반영)
  2. judge로 자동 채점 후 **judge↔인간 일치도 먼저 검증**(일치도 미달이면 채점 무효 — 게이트 신뢰 선결)
  3. agent 산출물 정답률·rework·회귀 측정 + **first-pass/유효-결정 라벨을 QA-07·QA-09 집계로 흘려보내 동반 닫힘 확인**
- **합격 기준(Exit)**: judge 일치도 ≥ 목표(신뢰 확보) ∧ 정답률 측정 가능·100% 아님(적정 난이도) ∧ **QA-07 speedup 분모(first-pass 성공분)·QA-09 ②-2가 이 게이트 출력으로 산출됨**(교차 의존 닫힘 시연).
- **규모/기간(Scope)**: 단계 4종 × 앵커 50~100 + judge 일치도 검증 → 약 4~5일. golden set 4단계 라벨이 critical path(round-01 _poc-plan 일치).
- **리스크/한계(silent cap)**: golden set 커버리지 = 정확성 신뢰 상한(미포함 케이스 미검출). judge 자체 편향(일치도 검증이 전제). quantize "정답"의 정의(정확도/지연 trade-off) 합의가 선결. **PoC는 채택 결정(OI-8)을 대체하지 못함** — 채택은 사람 몫. eval/검증 서브시스템 DP가 없으면 PoC는 가능해도 운영 귀속이 비어 있음(OI-7).

## DP·발표 영향

- **DP 연결 (OI-7, DP 디스커션 1순위)**: `related-dp: []` — NQA-B를 책임지는 DP가 전무. **eval/검증 서브시스템 DP 신설**이 NQA-B golden+judge+회귀 게이트를 받치며, **NQA-A red-team·QA-03 위반0·QA-07 first-pass·QA-09 결정 판정 게이트와 단일 인프라로 묶임**(review 렌즈3 = DP 디스커션 핵심 의제). NQA-B 미닫힘 = 이 DP 부재의 직접 결과 — QA 측 verdict는 여기로 내려가지 않고 DP로 위임.
- **동반 닫힘 효과 (핵심)**: NQA-B 정식 채택 + golden 실측 전환 = **QA-07 주 KPI(△→○)·QA-09 ②-2(△→○) 동반 닫힘의 단일 행동.** 비대칭 주의 — QA-09는 contention 덕에 ②-1(룰 게이트, golden 불요)로 *부분 닫힘*이라 NQA-B 의존은 ②-2뿐이지만, QA-07은 주 KPI 분모가 통째로 NQA-B 의존이라 닫힘도가 더 낮다. 따라서 **NQA-B 채택의 우선순위 효과가 QA-07에서 가장 크다.**
- **번호/서사 (OI-8)**: NQA-A와 함께 "사람 없이 믿고 맡길 수 있는가" 두 기둥 → 우선순위 상위 진입 후보. 채택 시 QA 2자리 번호(=우선순위) 재정렬·INDEX/glossary 동기화·양방향 cross-link 확정. QA-09와 묶어 "일관(QA-09) + 정확(NQA-B)" 발표 서사. **importance H 유지** 권고(세트 병목 진원지).
