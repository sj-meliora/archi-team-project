---
id: QA-07
category: QA
importance: H
difficulty: H
source: discussion/qa/round-01 (신규 — pptx 외 발굴, C5)
iso-25010: "Functional Suitability / Functional Correctness (기능 정확성)"
related-dp: []  # eval/검증 서브시스템은 신규 DP 후보 (현재 DP 없음 — ## 검증 전략 참조)
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "신설 — '맞는 결정'(정확성)을 1급 QA로 발굴, QA-09/11 KPI의 닫힘 조건 (자세히 → ## 변경 이력)"
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "ISO/IEC 25010 Functional Correctness 앵커링 + 짝 QAS-07 신설"
  - date: 2026-06-24
    by: 팀 결정 (OI-8)
    reason: "정식 QA 편입 — NQA-B → QA-07(우선순위 7위), ASR 선정(QA-01~07) (자세히 → ## 변경 이력)"
---

# QA-07 Correctness / Accuracy — 산출물 정확성

> 📌 **정식 QA (2026-06-24 팀 결정, OI-8).** Security(QA-06)와 함께 "사람 없이 믿고 맡길 수 있는가"의 두 기둥. 우선순위 **7위**로 편입(NQA-B → QA-07). **ASR 선정(QA-01~07)** — DP 생성 동인. 추가 상향은 팀 논의 후 별도.
>
> **ISO/IEC 25010:2023 앵커**: 주 특성 **Functional Suitability** / 하위특성 **Functional Correctness(기능 정확성)** — "Correctness"가 ISO 하위특성명 그 자체. (golden 커버리지는 보조적으로 **Functional Completeness**.) 일관성(QA-11 Reliability)과 **직교** — 정확성 ≠ 일관성, 둘은 짝.

## 정의 / Refinement
에이전트의 자동 산출물(**quantize config, 빌드/검증 결정, 이슈 처리**)이 **정답·정책에 부합**한다. 자율성 신뢰의 #1 차단요인은 "산출물이 **맞는가**"이며, 어떤 기존 QA도 이를 측정하지 않았다.

설계할 때 잡아야 할 두 가지 관점:

- **일관성 ≠ 정확성** — QA-11(일관성)는 "**같은** 결정"만 보고 "**맞는** 결정"은 보지 않는다 — 일관되게 틀릴 수 있다. 정확성은 **golden-set(정답 데이터) 대비 정답률**로 따로 재야 하며, QA-11와 짝(일관 + 정확)이어야 자율 신뢰가 성립한다.
- **틀리면 시간 이득이 사라진다** — 잘못된 quantize config·빌드 결정은 rework(사람 재작업)를 폭증시켜 QA-09/10의 속도 이득을 모두 까먹는다. 그래서 정확성은 **QA-09(first-pass 성공 판정)·QA-11(유효-결정률)의 KPI를 닫는 전제**다(이 게이트 없이는 두 QA가 측정 불가).

> 이 QA는 "산출물이 **맞는가(정확성)**"를 다룬다 — "**같은 일을 다시 시켜도 흔들리지 않는가(일관성)**"는 QA-11에서 본다. 정확성 게이트는 QA-09(빠르게 틀리기 차단)·QA-11(유효-결정 판정)에 측정 기준을 공급한다.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `golden-set 대비 산출물 정답률` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **golden-set 대비 산출물 정답률 ≥ 90%** `[주 KPI · PoC 대상]` — 단계별(IR/Optimize/Quant/Compile)
  - 쉽게: 정답이 라벨된 표준 문제집(golden set)을 에이전트에 풀려, 단계별로 90% 이상 맞혀야 한다. *golden set = 사람이 정답을 매긴 평가용 데이터셋. 단, 100% 통과면 문제집이 너무 쉽다는 신호(적정 난이도 필요).*
- **자동 결정 rework율 ≤ 10%** — 사람 재작업 유발 비율(QA-09 first-pass 게이트와 공유)
  - 쉽게: 에이전트 결정 100건 중 사람이 다시 손봐야 하는 게 10건 이하여야 한다. *rework = 잘못돼서 다시 하는 작업.*
- **회귀(regression) 미검출률 ≤ 5%** — 정확성 게이트 누수
  - 쉽게: 예전엔 잘 되던 게 새로 망가졌는데(회귀) 게이트가 못 잡고 통과시키는 비율이 5% 이하여야 한다. *회귀 = 기존 정상 동작이 변경으로 깨지는 것.*

> 위 수치(90%·10%·5%)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 우리 golden set으로 측정해 확정한다(레퍼런스 수치 복제 아님).

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 LLM 정확성 평가의 필드 표준에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **golden dataset 대비 정답률** | 단계별 golden(앵커 50~100, 사람 라벨) 대비 정답률이 정확성 측정의 표준. 100% 통과면 난이도 부족 신호 | [golden dataset 구축 가이드](https://www.getmaxim.ai/articles/building-a-golden-dataset-for-ai-evaluation-a-step-by-step-guide/) |
| **LLM-as-judge (다수결·인간 일치 검증)** | 자동 채점은 LLM-as-judge 다수결로, 단 **judge↔인간 일치도(75~90%)를 먼저 검증**해야 신뢰 가능 | [LLM-as-a-judge 해설](https://www.comet.com/site/blog/llm-as-a-judge/) · [Monte Carlo — LLM-as-judge](https://montecarlo.ai/blog-llm-as-judge/) |
| **rework·회귀 분리 집계** | 정답률만이 아니라 사람 재작업률·회귀 누수를 분리해야 게이트 품질이 보임 | round-01 counsel §4 정확성 |

> ⚠️ 레퍼런스의 수치(정답률·일치도)는 **패턴 정당화용**이며 그대로 복제하지 않는다. 우리 합격선은 위 [검증 전략](#검증-전략)의 golden set으로 확정한다.

## 검증 전략

> ⚠️ **이 QA의 검증(golden set·LLM-as-judge 구축)은 [발표 서사]다.** feasibility-filter상, **eval/검증 서브시스템은 "아키텍처 요소(모듈 다이어그램의 박스)"로 남기되**(설계 산출물 = OK), **단계별 golden set(사람 라벨)·judge 인간 일치도 검증 구축은 [생략]**(실행 노동 = drop)한다. 아래는 "이렇게 검증하도록 *설계*했다"는 방법이지 실측이 아니다.

| KPI | 책임지는 설계 | 검증 방법 (설계) |
|---|---|---|
| **golden-set 정답률** `[주]` | **eval/검증 서브시스템(신규 — 현재 DP 없음)** + QA-04 trace가 산출물·결정 데이터 공급. QA-03/QA-06 red-team 하네스와 인프라 공유 가능 | **▶ 발표 서사(실측 미실행):** golden set + LLM-as-judge 하네스 — 단계별 golden(IR/Optimize/Quant/Compile) 대비 정답률 측정, **judge↔인간 일치도를 먼저 검증**한 뒤 채점. 하네스는 모듈 다이어그램에 박스로 존치, golden 구축·실행은 미수행 |
| rework율 ≤ 10% | QA-09 first-pass 게이트와 공유(성공 판정 = 정확성 게이트) | 보조(발표 서사): 자동 결정 중 사람 재작업 유발 비율 집계 |
| 회귀 미검출률 ≤ 5% | 회귀 골든·게이트 | 보조(발표 서사): 알려진 회귀 케이스 주입 → 게이트 누수율 측정 |

> 가정·한계: **golden set 커버리지가 곧 정확성 신뢰 상한**(미포함 케이스 미검출). **judge 자체 편향**(judge가 틀리면 정답률 왜곡)이라 인간 일치도 검증이 전제. quantize "정답"의 정의(정확도/지연 trade-off) 합의가 선결 사항. 본 라운드는 KPI 정의와 "검증하도록 설계했다"는 서사까지만(실행 0건).

## 변경 이력

### 2026-06-24 — round-01 디스커션 신설
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/NQA-B-correctness.md) · [신규 QA 후보](../../discussion/qa/round-01/review/_new-qa-candidates.md) (stance: **신설 — 채택 권장, 자율 신뢰의 #1 기둥**)

**왜 신설했나 (review 지적)**
- 신뢰의 #1 차단요인 = "산출물이 **맞는가**". QA-11(일관성)는 "같은 결정"만 보고 "맞는 결정"은 어떤 QA도 측정 안 함 — 일관되게 틀릴 수 있음.
- 잘못된 quantize config·빌드 결정 → rework 폭증 → QA-09/10 시간 이득 소멸.

**무엇을 담았나 (신설 내용)**
- **정의**: 자동 산출물이 정답·정책에 부합. 일관성(QA-11)과 짝, QA-09/10 게이트의 전제임을 명시.
- **KPI 3축**: ① golden-set 정답률 ≥90%(주, 단계별) ② rework율 ≤10%(QA-09 공유) ③ 회귀 미검출률 ≤5%.
- 검증은 golden + LLM-as-judge 하네스로 설계(실행은 [발표 서사]).
- **ISO/IEC 25010 앵커**: Functional Suitability / Functional Correctness. 짝 시나리오 `QAS-07` 신설.

**남은 일 (이 라운드에서 미반영)**
- **교차 의존 닫기**: QA-09(first-pass 성공 판정)·QA-11(유효-결정률)가 본 QA의 정확성 게이트에 의존 — 두 QA "남은 일"에 QA-07 전제로 기록됨. QA-07 정식 채택 시 양방향 cross-link 확정.
- **golden set·judge 구축은 [생략]** — eval/검증 서브시스템은 모듈 다이어그램 박스로만 존치(설계 산출물).
- **eval/검증 서브시스템은 신규 DP 후보** — 현재 어떤 DP도 명시 안 함 → DP 디스커션에서 신설 검토(`open-issues.md` 트래킹 대상).
- **번호 재정렬(팀 결정)**: QA-07 상위 진입 여부 — 확정 전까지 임시 ID.
- 정답률 `90%`·rework `10%`는 **예시값**이며 golden set으로 확정.

### 2026-06-24 — 정식 QA 편입 (팀 결정, OI-8)
출처: 팀 결정 — round-02 디스커션이 "세트 닫힘 단일 트리거(최우선)"로 올린 NQA-B를 정식 편입. OI-8 닫음.

**무엇을 바꿨나**
- **ID 확정**: `NQA-B`(임시) → **`QA-07`**(우선순위 7위, Security 다음). 짝 `QAS-B` → `QAS-07`.
- 위 "남은 일"의 **번호 재정렬(팀 결정)** 항목을 닫음. QA-09(first-pass)·QA-11(②-2 유효-결정률)와 **양방향 cross-link** 확정 — 정식 채택으로 두 QA의 golden 게이트 의존이 닫힘 가능 상태가 됨(실측 전환은 PoC, 팀 결정).
- **ASR 선정(QA-01~07)** — DP 생성 동인. eval/검증 서브시스템 DP 신설은 OI-7(DP 디스커션).

**남은 일**
- golden set·judge 구축 [생략]·예시값 확정은 종전대로. PoC 실측 전환은 팀 결정.
