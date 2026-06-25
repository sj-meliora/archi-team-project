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
  - date: 2026-06-24
    by: discussion/qa/round-03 (등급 척도 캘리브레이션)
    reason: "★ rubric 추가 — main 축=golden 정답률(정방향). 100% 통과=난이도 부족 상한 캡(★★★ 92~98%) + judge↔인간 일치도(κ≥0.8) 조건 신설 — 구 ≥90% → 신 90~98%(만점 캡)+judge κ 조건 (자세히 → ## 변경 이력)"
  - date: 2026-06-25
    by: discussion/qa/round-04 (★ 등급 척도 근거 보강)
    reason: "★★★ 92~99% / ★★☆ 90~93% / 100%만 불합격((98,100) 공백 제거·★★☆ 확장) + 경계 구간화([90,91)/[91,93)/[93,99]) + judge κ 도메인 재측정 단서 + rework율 보조 별점 축 (자세히 → ## 변경 이력)"
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
  - > 보정(2026-06-24, 등급 척도 캘리브레이션 round-03): 합격 하한 구 `≥90%`(상한·채점기 검증 무제약) → 신 `≥90% 유지 + 상한 캡 90~98%(100% 통과는 난이도 부족 → 불합격) + judge↔인간 일치도 조건`. 사유: 100% 통과 golden은 난이도 부족 신호이므로 ★★★를 만점 직전(92~98%)으로 캡하고, judge가 틀리면 정답률이 왜곡되므로 **LLM-as-judge ↔ 인간 일치도(Cohen's κ ≥ 0.8, acceptable κ ≥ 0.6)를 선검증**해야 정답률이 유효(미검증 시 측정 무효). 새 제약이라 보정 트레이스로 보존. ★ 급간은 ## 등급 척도 참조.
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

## 등급 척도 (★ rubric — ATAM trade-off용)

> 동일 조건 설계 대안의 본 QA 만족도를 ★1~3 비교(별 많은 안 채택). KPI 합격선(하한)=★☆☆ 진입선, ★★☆/★★★는 필드 현실 도달 범위+PoC margin. 하한 미만 불합격. 예시값이며 경계는 PoC로 확정.

> 헤드라인 `golden-set 정답률 ≥90%`. **⚠️ 100% 통과 = golden 난이도 부족 신호** → ★★★를 **[93,99]**(만점 아닌 고득점, 100%만 불합격)로 두고 **적정 난이도 + judge↔인간 일치도 조건**으로 처리(규칙4). main = 단계별 golden 정답률(정방향). 경계는 ≥/> 모호 제거 위해 구간([90,91)/[91,93)/[93,99])으로 표기(QA-04와 동형). main 축이 [발표 서사]라 **rework율을 보조 별점 축**으로 병기(round-04 C3).

**조건 (규칙4, 신뢰 전제):** `조건: LLM-as-judge ↔ 인간 일치도 Cohen's κ ≥ 0.8 (또는 동의율 ≥80%) 선검증` — judge가 틀리면 정답률 자체가 왜곡되므로, 별점을 매기기 전에 채점기(judge)의 인간 일치도를 먼저 검증한다. 미검증 시 정답률은 신뢰 불가(불합격 아닌 "측정 무효"). **(round-04 C1) κ는 우리 단계별 quantize golden으로 재측정 — 일반 벤치 κ 복제 아님**(인용 κ는 일반 QA frontier judge라 도메인 특화 정답성 판정에서 더 낮을 수 있음·silent cap). 추가 조건: `golden 난이도 적정(과쉬움 페널티)` — 100% 통과 세트는 난이도 부족으로 ★ 부여 보류.

| 등급 | 구간 — 주 KPI(main 축): 단계별 golden 정답률 % (정방향, 단 100% 만점 아님) | 필드 근거 (경계 이유 + URL) |
|---|---|---|
| ★★★ (상) | **[93, 99] (만점 아닌 충분 고득점) + judge κ ≥ 0.8** | **(round-04: 상한 98→99로 넓혀 (98,100) 공백 제거)** 적정 난이도 golden(앵커 50~100, contamination 차단)에서 93-99%는 "어려운 문제를 거의 다 맞힘". 100%만 난이도 부족 신호라 불합격(상한 캡=99%). frontier judge가 인간과 κ 0.81-0.87(o4-mini 0.873/GPT-5-mini 0.870/GPT-4.1 0.811) 달성하므로 κ≥0.8 동반 조건이 현실적. [LLM-as-judge κ 0.8+ 사례](https://futureagi.com/blog/llm-as-judge-best-practices-2026) · [MMLU 90%+는 변별력 상실(난이도 부족)](https://medium.com/@federicomoreno613/golden-datasets-the-foundation-of-reliable-ai-evaluation-486ce97ce89d) |
| ★★☆ (중) | **[91, 93) + judge κ ≥ 0.6** | **(round-04: 90~92 2%p → [91,93) 확장, ≥/> 모호 제거)** 단계별 91~93% = 일반 우수. judge 일치도는 production 허용선 κ≥0.6(acceptable)로 완화. 인간 전문가 task success도 ~90%(DigiData 90.1%) 대역이라 90%대 초반이 강한 자동화 신뢰선. [judge κ≥0.6 acceptable](https://futureagi.com/blog/llm-as-judge-best-practices-2026) · [인간 전문가 ~90%](https://arxiv.org/pdf/2511.07413) |
| ★☆☆ (하) | **[90, 91) (= KPI 하한·합격 최소선) + judge 일치도 검증 완료** | KPI 정의 하한 `≥90%`가 ★☆☆ 진입선. golden 90%는 frontier 모델 MMLU 수준이자 자율 신뢰의 현실 진입선이라 비현실적으로 높지 않음 → 규칙2 재배치 불요. 단 judge 일치도 미검증이면 측정 무효. [golden 90% 진입선](https://www.getmaxim.ai/articles/building-a-golden-dataset-for-ai-evaluation-a-step-by-step-guide/) |
| 불합격 | 정답률 < 90% / **= 100%(난이도 부족→재설계)** / judge 인간 일치도 미검증 또는 κ < 0.6 | (98,100) 공백 제거 — 100%만 불합격, 99%까지 ★★★ |

**보조 별점 축 (실측 가능 — round-04 C3):** main 축 golden 정답률이 [발표 서사](golden·judge 미구축)라 모델 추정 → `rework율(자동 산출물이 후속 단계/재시도로 되돌아오는 비율, 파이프라인 로그에서 golden 없이 산출)`을 보조 별점 축으로 병기. 역방향(낮을수록 ★ 높음). ATAM 비교 시 정답률이 미실측이면 rework율로 변별(QA-11 ②-1과 동형).

| 보조 등급 | 구간 — rework율 (낮을수록 좋음) | 근거 |
|---|---|---|
| ★★★ (상) | rework율 ≤ 5% | 자동 산출물이 거의 되돌아오지 않음 — golden 없이 파이프라인 로그에서 실측 |
| ★★☆ (중) | 5% < rework율 ≤ 15% | 일반 우수 — 가끔 재작업 |
| ★☆☆ (하) | 15% < rework율 ≤ 30% — 합격 진입선 | §측정 rework율 보조 KPI(≤10%)와 정합 대역(상한 30%까지 합격) |

> **캘리브레이션 노트**: 하한 `90%`는 필드 진입선으로 적정 → 규칙2 재배치 불요. **상한 처리가 핵심** — golden 100% 통과는 난이도 부족 신호이므로 ★★★를 만점 직전으로 캡하고 "과쉬움 페널티"를 명시(100% = 불합격 처리). `이론 근거 = frontier judge κ 0.81-0.87 도달` / `PoC margin = 우리 golden은 가상 설계·실행 미수행([발표 서사])이라 judge 일치도·난이도 적정을 PoC로 확정해야 함 → κ 조건을 ★ 등급에 동반`. silent cap: **golden 커버리지 = 정확성 신뢰 상한**(미포함 케이스 미검출) + **judge 자체 편향**(judge 틀리면 정답률 왜곡, 그래서 일치도 선검증이 전제) + **quantize "정답" 정의**(정확도/지연 trade-off) 합의가 선결. eval/검증 서브시스템은 모듈 박스로만 존치(실측 0건, OI-7 DP 신설 의존).
> **재캘리브레이션(round-04)**: ★★★ 상한 `98→99`로 넓혀 **(98,100) 무등급 공백 제거**(100%만 불합격 유지), ★★☆ `90~92(2%p) → [91,93)`로 확장, ★☆☆ `[90,91)`. 경계 ≥/> 모호를 **구간 표기**([90,91)/[91,93)/[93,99])로 제거(QA-04와 동형). **judge κ 도메인 재측정 단서(C1)**: 인용 κ(o4-mini 0.873 등)는 일반 벤치 LLM-as-judge라 quantize 도메인과 apples 아님 → κ는 우리 단계별 quantize golden으로 재측정(복제 아님), 도메인 특화 판정에서 더 낮을 수 있음(silent cap). **rework율 보조 별점 축(C3)**: main 정답률이 [발표 서사]라 모델 추정 → rework율(파이프라인 로그·golden 불요)을 보조 별점 축 병기(≤5/15/30%). golden·judge 미구축으로 정답률 main 축·κ는 순환(정답률←golden←judge κ←인간라벨←미수행). 본 보정은 OI-9 등급표↔변경이력↔짝 QAS 동기화 대상.
> **seats**: 발의 Seat 1(eval 하네스·LLM-as-judge·100% 난이도 신호) · consensus (Seat 2: ★★★ 만점 캡·과쉬움 페널티 형식화 동의 / Seat 3: judge 일치도 조건을 ★ 등급에 동반시키는 구조 동의)

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

### 2026-06-24 — 등급 척도(★ rubric) 캘리브레이션
출처: discussion/qa/round-03 (등급 척도 캘리브레이션 — Council 3 seats, 팀 승인). 용도 = ATAM trade-off에서 동일 조건 설계 대안 비교(★ 많은 안 채택).

**무엇을 했나**
- **main 급간 축**: 단계별 **golden-set 정답률 %**(정방향). single-axis라 규칙5 분기 불요.
- **★ 급간**: ★★★ 92~98%(만점 아닌 고득점) + judge κ≥0.8 / ★★☆ 90% 초과~92% + judge κ≥0.6 / ★☆☆ ≥90% + judge 일치도 검증 완료. 근거 = frontier judge κ 0.81-0.87·인간 전문가 task success ~90%·MMLU 90%+ 변별력 상실(난이도 부족).
- **조건(규칙4)**: `LLM-as-judge ↔ 인간 일치도 κ≥0.8(또는 동의율 ≥80%) 선검증` — 미검증 시 정답률 측정 무효. + golden 난이도 적정(과쉬움 페널티).
- **§측정 보정(구→신)**: 구 `≥90%`(상한·채점기 검증 무제약) → 신 `≥90% 유지 + 상한 캡 90~98%(100% 통과 = 난이도 부족 → 불합격) + judge κ 조건`. 새 제약이라 보정 트레이스를 §측정 해당 KPI 줄 아래 보존.
- silent cap: golden 커버리지 = 정확성 신뢰 상한 / judge 자체 편향(일치도 선검증 전제) / quantize "정답" 정의 합의 선결. eval/검증 서브시스템은 모듈 박스 존치(실측 0건, OI-7 의존).

### 2026-06-25 — round-04 디스커션 반영 (★ 등급 척도 근거 보강)
출처: [`discussion/qa/round-04`](../../discussion/qa/round-04/counsel/QA-07-correctness.md) (red verdict: **Sound ◎ / KPI ○ — Med**; stance: 조건부 채택 — 대역+공백 정리·judge κ 도메인·rework율 보조축).

**무엇이 문제였나 (review 지적)**
- **★★☆ (90,92) 2%p 좁음 + (98,100) 무등급 공백.**
- **judge κ는 도메인 의존** — 인용 κ(o4-mini 0.873 등)는 일반 벤치라 quantize 도메인과 apples 아님(C1).
- main 축(golden 정답률)이 [발표 서사]라 실측 불가 → 모델 추정 별점·순환(C3).

**무엇을 바꿨나 (반영)**
- **★ 급간: ★★★ [93,99] / ★★☆ [91,93) / ★☆☆ [90,91) / 100%만 불합격** — (98,100) 공백 제거(상한 98→99)·★★☆ 2%p→2%p 명확화. 경계 ≥/> 모호를 구간 표기로 제거(QA-04와 동형).
- **judge κ 도메인 재측정 단서**: 우리 단계별 quantize golden으로 κ 재측정(일반 벤치 κ 복제 아님), 도메인 특화 판정에서 더 낮을 수 있음(silent cap).
- **rework율 보조 별점 축 병기**(≤5/15/30%) — golden 없이 파이프라인 로그 산출, main이 미실측이면 변별.

**남은 일 (이 라운드에서 미반영)**
- golden·judge 구축 [생략]([발표 서사]) — 정답률 main 축·κ는 순환(미수행).
- eval/검증 서브시스템 신규 DP(related-dp 빈 상태, OI-7 1순위) — QA-03/06 red-team 하네스와 공유.
- QA-09 first-pass·QA-11 ②-2가 QA-07 golden 의존 → QA-07 닫힘이 교차 의존 해소 키.
- 경계(93/99·91/93·rework 5/15/30%)는 예시값 — golden set으로 확정.

> 출처: [discussion/qa/round-04](../../discussion/qa/round-04/counsel/QA-07-correctness.md) (verdict: Sound ◎ / KPI ○ — Med, 조건부 채택).
