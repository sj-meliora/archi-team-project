# Review: NQA-B Correctness / Accuracy — 산출물 정확성 (round-02 신설 QA 재평가)

> source: `context/qa/NQA-B-correctness.md` + `QAS-B-correctness.md` (round-01 신설 후)
> 직전 stance(round-01): **신설 — 채택 권장, 자율 신뢰의 #1 기둥**
> verdict(round-02): **Sound ○ / KPI △ — High** — 신설 내용·ISO 앵커는 완성도 높고 정식화 권장. 그러나 **NQA-B는 QA-07·QA-09(②-2)의 닫힘 게이트인데 자신의 검증(golden+judge)이 [발표 서사]·DP 부재** → 두 QA의 미닫힘을 좌우하는 *세트 전체의 병목* → severity **High**(닫힘 의존의 진원지)
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 NQA-B = **[반영](신설) + eval [생략]**. 이월: golden 구축 [생략]; eval 서브시스템 신규 DP(OI-7); 번호(OI-8).

## 원문 요약 (신설 후)
- **정의**: 에이전트 자동 산출물(quantize config·빌드/검증 결정·이슈 처리)이 정답·정책에 부합. 일관성(QA-09)과 직교 — "맞는 결정". QA-07/09 KPI의 닫힘 전제.
- **ISO 앵커**: 주 특성 Functional Suitability / 하위특성 **Functional Correctness**(이름 그 자체) + Completeness(golden 커버리지).
- **KPI 3축**: 주 = `golden-set 정답률 ≥90%(단계별)`. 보조 = `rework율 ≤10%(QA-07 공유)` · `회귀 미검출률 ≤5%`.
- **QAS-B** 신설.

## 렌즈 1 — Agentic Workflow 전문가 관점

자율 신뢰의 #1 차단요인("산출물이 **맞는가**")을 1급으로 발굴한 게 정확하다. QA-09(일관성)가 "같은 결정"만 보고 "맞는 결정"은 어떤 QA도 안 봤다는 진단, "일관되게 틀릴 수 있다"는 통찰이 이 QA의 존재 이유. golden-set 대비 정답률 + LLM-as-judge(judge↔인간 일치도 선검증)라는 측정 설계도 필드 표준. **신설 내용 강건.**

**round-02 핵심 — NQA-B는 세트의 닫힘 병목(진원지):**
- applier §3-2가 묻는 "교차 의존이 닫혔는가"의 답이 NQA-B에 달려 있다. **QA-07 주 KPI(first-pass 성공 판정)·QA-09 ②-2(정밀 유효-결정률)가 NQA-B golden 게이트에 닫힘 의존.** 그런데 NQA-B 자신의 검증(단계별 golden·judge 구축)이 [발표 서사]/[생략]이다 → **NQA-B가 안 닫히면 QA-07·QA-09도 안 닫힌다.** 이게 NQA-B를 다른 신설 QA보다 severity High로 올리는 이유 — 자기 KPI만의 문제가 아니라 *세트 전체의 미닫힘을 좌우*한다.
- **재판정**: golden 하네스를 모듈 박스로 존치(설계 OK)하나, 정답률 90%·rework 10%·회귀 5%의 acceptance가 실측으로 닫히지 않음 → KPI △. **NQA-B 정식 채택 + golden 실측 전환이 QA-07·QA-09 동반 닫힘의 트리거.**

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **정식화 가부(OI-8) = 채택 권장(NQA-A와 함께 강한 권장)**: Functional Correctness가 ISO 하위특성명 그 자체라 앵커가 가장 명확. QA-09(일관성)와 **직교** 명시로 중복 없음(일관 ≠ 정확, 둘은 짝). 단일 관심사(정답성). **Sound ○ — 1급 정식화 자격 충분.** "사람 없이 믿고 맡길 수 있는가" 두 기둥 중 하나.
- **KPI △ + severity High 사유**: measurable 형식은 갖췄으나 측정 수단(golden+judge)이 [발표 서사]. 게다가 **이 미닫힘이 QA-07·QA-09로 전파**돼 단일 QA 결함을 넘는다 → High. (다른 신설 NQA-A/C는 자기 KPI에 국한돼 Med.)
- **judge 편향 리스크**: judge가 틀리면 정답률이 왜곡 → 인간 일치도(75~90%) 선검증이 전제로 명시됨(양호). quantize "정답"의 정의(정확도/지연 trade-off) 합의가 선결이라는 silent cap도 정직.
- **양방향 cross-link 미완**: NQA-B 정의는 QA-07/09를 명시하나, "NQA-B 정식 채택 시 양방향 cross-link 확정"이 아직 [이월]. consistency는 ○(QAS-B 동기화)이나 채택 시 cross-link 확정 필요.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

- **golden 게이트의 runner 환원**: golden set + LLM-as-judge 하네스가 산출물·결정 데이터를 **QA-04 trace에서 공급**받아 채점하는 구조 — QA-04(Observability)가 측정 토대라는 세트 의존이 여기서 닫힌다. first-pass 게이트(QA-07)·결정 판정 게이트(QA-09 ②-1)와 **인프라 공유** 가능.
- **잔여(DP 위임, OI-7 — 가장 큰 줄기)**: `related-dp: []` — **NQA-B는 자신을 책임지는 DP가 아예 없다.** eval/검증 서브시스템(golden+judge+회귀 게이트)이 어떤 DP에도 없는 신규 인프라 → **"eval/검증 서브시스템" DP 신설이 DP 디스커션의 1순위 후보.** NQA-A red-team·QA-03·QA-07·QA-09와 공유하므로 한 DP로 묶임. NQA-B의 미닫힘 = 이 DP 부재의 직접 결과.
- **runner 측 KPI**(제시): golden 채점 throughput, judge↔인간 일치도(게이트 신뢰 자체의 SLI), 회귀 게이트 누수율, 단계별(IR/Optimize/Quant/Compile) 정답률 분해, first-pass/rework 라벨 공급률(QA-07 공유).

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | (신설) | **○** | "맞는 결정" 단일 관심사 + QA-09 직교 명시 + ISO Functional Correctness 앵커(이름 그 자체) |
| KPI가 측정 가능한가 | (신설) | **△** | golden·judge 측정 수단이 [발표 서사]·DP 부재. **미닫힘이 QA-07·QA-09로 전파** |
| KPI가 현실적/적절한가 | (신설) | **○** | golden+judge(일치도 선검증)·rework·회귀 분리 현실적. 정답 정의 합의는 선결(명시됨) |
| 정의↔KPI↔QAS 일치 | (신설) | **○** | QAS-B 동기화. 단 QA-07/09 양방향 cross-link은 채택 시 확정 |

**verdict(신설 → round-02): Sound ○ / KPI △ · severity High.** 정식화 가부 = **채택 강력 권장**(NQA-A와 함께 두 기둥). **severity High는 단일 QA 결함이 아니라 세트 닫힘 병목** — NQA-B 채택·실측이 QA-07·QA-09 동반 닫힘의 단일 트리거이기 때문. round-02의 최우선 구조 이슈.

## Stage 2 권고 (round-02)

- **(정식 채택 — 최우선, OI-8)** NQA-B를 정식 QA로 승격 + 우선순위 상위. **채택 자체가 QA-07 주 KPI·QA-09 ②-2를 동반으로 닫는 단일 행동.** QA-07·QA-09와 양방향 cross-link 확정. 채택 시 golden·judge를 실측으로 전환.
- **(DP 디스커션 1순위, OI-7)** **"eval/검증 서브시스템" DP 신설** — NQA-B golden+judge+회귀 게이트가 `related-dp: []`로 책임 설계 전무. NQA-A red-team·QA-03·QA-07·QA-09와 공유하는 단일 인프라 DP로 묶어 신설. **이게 다음 DP 디스커션의 핵심 의제.**
- **(judge 신뢰 SLI, Med)** judge↔인간 일치도(75~90%)를 *게이트 신뢰 자체의 보조 KPI*로 노출 — golden 정답률의 신뢰 상한.
- **(예시값 확정)** `90%·10%·5%`는 golden set으로 확정. 단계별 정답률 분해 권장(IR/Optimize/Quant/Compile 난이도 상이).
