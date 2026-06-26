---
id: QA-11
category: QA
importance: L
difficulty: H
source: pptx p.13
related-dp: [DP-0005]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "결정성 목표 폐기 → '유효-결정 안정성' 재프레이밍, 캐시 우회 pass^k·유효결정률 KPI (자세히 → ## 변경 이력)"
  - date: 2026-06-24
    by: discussion/qa/round-01 contention
    reason: "contention 반영 — 헤드라인을 Δ 대조 시연으로 교체, ②2단 분리(②-1 대리/②-2 open), 분산=H_norm 정의, importance 게이트"
  - date: 2026-06-24
    by: 팀 결정 (OI-8)
    reason: "재번호 QA-09 → QA-11 (NQA 정식 편입에 따른 +2 시프트, → changelog)"
  - date: 2026-06-24
    by: discussion/qa/round-03 (등급 척도 캘리브레이션)
    reason: "★ rubric 추가 + 보조 KPI pass^k 합격선 ≥70% → 25/40/60%(★☆☆/★★☆/★★★) 전면 재배치 (헤드라인 Δ 대조는 시연 게이트 불변) (자세히 → ## 변경 이력)"
  - date: 2026-06-25
    by: discussion/qa/round-04 (★ 등급 척도 근거 보강)
    reason: "★★★ pass^5 ≥60% → ≥45% 하향(voting→pass^k 메커니즘 1차출처 반증) + ②-1 보조 별점 축 병기 + k 보간/apples silent cap + §측정 50행 구값 정정 (자세히 → ## 변경 이력)"
---

# QA-11 Reliability — Agent 결과 일관성

## 정의 / Refinement
동일/**유사** 입력에서 **유효(정책상 허용) 결정의 안정성**을 유지한다. LLM에서 완전한 결정성(같은 출력)은 비현실적·고비용이므로, 추구 대상은 "같은 출력"이 아니라 **허용 오차 내에서 흔들리지 않는 유효한 결정**이다.

설계할 때 잡아야 할 두 가지 관점:

- **일관성 ≠ 정확성 — 정확성 앵커와 짝지어야 한다** — 일관되게 **틀릴** 수도 있다(정확성 없는 일관성은 약한 QA). 재현율 80%가 "5번 중 1번은 갈린다"인데, 갈린 결과가 다 유효하면 괜찮고 일부가 오답이면 심각하다. 그래서 **유효-결정률**(재실행 중 정책상 유효한 비율)을 함께 재고, **QA-07(Correctness)**와 짝지어야 자율성 신뢰가 성립한다.
- **캐시는 일관성을 입증하지 못한다(치트 주의)** — DP-0005 공유 캐시로 동일 입력을 캐시하면 재현율은 자명하게 ~100%가 되지만, 저장된 답을 되돌려줄 뿐 **순수 추론의 일관성은 입증하지 못하고** 유사 입력의 흔들림도 못 잡는다. 따라서 측정은 **캐시를 우회(memoization 우회 모드)**한 순수 추론 기준으로 한다. 캐시는 비용 절감책이지 일관성 입증이 아니다.

> 이 QA는 "**같은 일을 다시 시켰을 때 결정이 흔들리지 않는가(일관성)**"를 다룬다 — 그 결정이 "**맞는가(정확성)**"는 QA-07(Correctness)에서 본다. 둘은 짝(일관 + 정확)이어야 신뢰가 닫힌다.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `캐시 On/Off 일관성 갭 Δ` 1개.** 우리 조건(가상 설계·실측 불가)에서 시연 가능한 건 *절대값*이 아니라 **명제의 대조**다 — 나머지는 보조(가드레일).

- **캐시 On/Off 일관성 갭 Δ = pass^k(캐시 On) − pass^k(캐시 우회)** `[주 KPI · PoC 대상]` — 명제: "캐시는 일관성을 입증하지 못한다"
  - 쉽게: 같은 입력을 반복하면 **캐시를 켜면** 저장된 답을 돌려줘 ~100% 일관처럼 보이지만, **캐시를 끄면(순수 추론)** 그보다 낮게 나온다. 이 **갭 Δ가 크다는 것 자체**가 "캐시가 일관성을 가린다(치트)"의 증거다. 방법론 모델에서 이 대조를 **직접 시연**하는 게 헤드라인 — 시연 0건이 아니라 모델 자체가 시연이다.
- **(보조) 순수 추론 pass^k (k=5)** — 합격 하한 ≥ 25%(★☆☆), 절대값은 실환경 측정으로 확정
  - 쉽게: 캐시 끈 상태에서 5번 모두 일관한 비율. 헤드라인을 받치는 가드레일 값이며, 절대값 실측은 실제 에이전트가 있어야 가능([발표 서사]). *pass^k = k회 전부 통과(일관)한 비율 — pass@1 90%여도 k=8이면 57%로 급락하는 엄격한 척도.*
  > 보정(2026-06-24, 등급 척도 캘리브레이션 round-03): pass^k(k=5) 합격선 구 `≥70%` → 신 `25/40/60%`(★☆☆/★★☆/★★★ 전면 재배치) — τ-bench 대비 70%는 비현실(frontier도 retail pass^1 ~80%, GPT-4o pass^4 ~37%·pass^8 ~25%)이라 ★★★조차 죽는 등급. **헤드라인 Δ 대조는 시연 게이트라 불변**, pass^k는 캐시 우회(순수 추론) main 축. ★ 급간은 ## 등급 척도 참조.
  > **재보정(2026-06-25, round-04)**: pass^5(k=5) 급간 `25/40/60%` → **`25/35/45%`**(★☆☆/★★☆/★★★) — voting이 pass^k(전 시행 성공)를 60%로 끌어올린다는 round-03 가정이 **메커니즘 오해**임을 1차 출처로 확정(voting은 pass@k=1답 정확도만 올림). ★★★를 0.8^5≈33%(독립 상한 근방) 약간 위인 45%로 둬 "frontier + 결정성 레버(temp=0·fixed seed)로 시행 간 약한 양의 상관" 도달 대역으로 재정의(사문화 회피). ②-1(룰 게이트 통과율)을 **보조 별점 축**으로 병기. ★ 급간은 ## 등급 척도 참조.
- **(보조) ②-1 대리 유효-결정률 ≥ 95%** — 정책 룰셋/제약 위반 검사 통과율 (golden 불요)
  - 쉽게: golden 정답이 없어도 **명백히 무효인 결정**(제약 C 위반·allowlist 위반·스키마/구조 위반 = hard-invalid)은 룰로 걸러낼 수 있다. 재실행 결과가 이 "필요조건 게이트"를 통과하는 비율 ≥ 95%. 우리 조건에서 룰 체커로 산출 가능. *대리(proxy) = golden 없이도 무효를 거르는 1차 게이트.*
- **(보조·open-issue) ②-2 정밀 유효-결정률** — golden 대비 정답성까지 본 유효 비율 (QA-07 의존)
  - 쉽게: "무효 아님"을 넘어 "정답인가"까지 보려면 QA-07 golden이 있어야 한다 → 미구축이라 **추적되는 open-issue로 강등**(닫힌 KPI 아님). *placeholder(숨은 빈틈)가 아니라 선언된 open-issue(추적되는 빈틈).*
- **(보조) 재실행 분산 H_norm ≤ 0.2** — 정규화 Shannon 엔트로피
  - 쉽게: 반복 결과를 유한 라벨 집합 C(예: {승인, 반려, 보류})로 매핑한 뒤 흩어짐을 0~1로 잰다. `H_norm = −Σ p·log p / log|C|` (0 = 전부 한 라벨=완전 일관, 1 = 균등 분산=최대 흔들림). 0.2 이하 = 한 라벨에 충분히 몰림. *자유형 텍스트 출력이면 라벨 매핑(②-1 룰 게이트와 같은 컴포넌트) 선행 필요 — silent cap.*

> 위 수치(Δ 대조·pass^5 25/35/45%·②-1 95%·H_norm 0.2)는 **측정가능 KPI의 모양 예시**이며, 절대 합격선은 실환경 측정으로 확정한다.
> 폐기: 旧 `추론 재현 성공률 ≥ 80%`(단독). **강등(contention R1)**: `pass^k 절대값`을 헤드라인에서 보조로 — 우리 조건상 실측 불가([발표 서사])라, 헤드라인은 **시연 가능한 Δ 대조**로 교체.

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 LLM 일관성·정확성 평가의 표준에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **pass^k (일관성 척도)** | k회 i.i.d. 시행의 일관성. pass@1이 90%여도 k=8엔 57%로 급락 — 캐시 우회·반복시행으로 측정 | [τ-bench (arXiv 2406.12045)](https://arxiv.org/abs/2406.12045) · [Sierra — τ-bench 해설](https://sierra.ai/blog/tau-bench-shaping-development-evaluation-agents) |
| **유효성 판정 = golden/policy** | 일관 + 정확이 함께여야 자율 신뢰 — 유효-결정률은 정확성 게이트(QA-07 golden)에 의존 | [LLM-as-a-judge 해설](https://www.comet.com/site/blog/llm-as-a-judge/) |
| **결정 안정성 정공법(캐시는 비용책)** | temperature=0·fixed seed·structured output·self-consistency voting이 안정성 레버 — 캐시는 비용 절감책이지 일관성 입증 아님 | round-01 counsel §4 (렌즈1) |

> ⚠️ 레퍼런스의 수치(pass^k·k값)는 **패턴 정당화용**이며 그대로 복제하지 않는다. 우리 합격선은 위 [검증 전략](#검증-전략)의 반복시행으로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 반복시행 모델**로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **캐시 On/Off 일관성 갭 Δ** `[주]` | **DP-0005 2안 공유 캐시 = memoization**(캐시 On이 ~100%로 부풀림 — 측정을 오염) + runner 버전 핀닝(model/prompt/tool 고정) | **▶ 실제 제작:** 캐시 On/Off 대조 모델 — memoization On vs 우회 모드에서 동일 입력 k회 반복 → `pass^k(On)≈100%` vs `pass^k(우회)<100%`의 **갭 Δ를 그래프로 대조 시연**(명제 "캐시는 일관성 입증 아님"). 이 **대조 자체가 시연**이라 [발표 서사]가 아님 |
| (보조) 순수 추론 pass^k 절대값 | 위 우회 모드 | 보조 모델(발표 서사): 우회 pass^k *절대값* 실측은 실제 에이전트 필요 — 미실행 |
| ②-1 대리 유효-결정률 ≥ 95% | 정책 룰셋/제약 위반 검사기(C-제약·allowlist·스키마 위반 = hard-invalid) — golden 불요 | 보조 모델: 재실행 결과를 룰 체커에 통과시켜 hard-invalid 차단율 산출(**우리 조건 산출 가능 = [반영]**) |
| ②-2 정밀 유효-결정률 | **QA-07(Correctness) golden 게이트**(교차 의존, open-issue 강등) | (open-issue) QA-07 구축 전 미산출 — 닫힌 KPI 아님 |
| 재실행 분산 H_norm ≤ 0.2 | 버전 핀닝 + 라벨 매핑(②-1 룰 게이트 컴포넌트 재사용) | 보조 모델: 반복 결과를 라벨 집합 C로 매핑해 `−Σp·log p/log\|C\|` 정규화 엔트로피 산출 |

> 가정·한계: **"유사 입력"의 흔들림은 동일 입력 반복으로 못 봄**(semantic-equivalent 입력 세트가 별도로 필요 — 본 라운드 범위 밖). 유효성 판정 게이트가 작으면 유효-결정률이 낙관 편향. 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다 — silent cap으로 명시.

## 등급 척도 (★ rubric — ATAM trade-off용)

> 동일 조건 설계 대안의 본 QA 만족도를 ★1~3 비교(별 많은 안 채택). KPI 합격선(하한)=★☆☆ 진입선, ★★☆/★★★는 필드 현실 도달 범위+PoC margin. 하한 미만 불합격. 예시값이며 경계는 PoC로 확정.
>
> 헤드라인 `캐시 On/Off 일관성 갭 Δ`는 대조/명제형 시연(0건 절대형·Constraint 성격)이라 별점 급간이 무의미 → Δ는 시연 게이트로 두고, 별점은 보조지표 `순수 추론 pass^k (k=5)`로 급간화(gradable이라 QA 성립). pass^k는 정방향(높을수록 ★ 높음). H_norm은 동반 게이트로 병기.
>
> **이중 축(round-04, C3): main = pass^5(모델 추정) + 보조 = ②-1 룰 게이트 통과율(실측 가능).** main 축 pass^5 *절대값*은 우리 조건상 실측 불가([발표 서사])라 ★가 모델 추정에 그친다 → ②-1(룰 체커 차단율, golden 불요·우리 조건 산출 가능)을 **보조 별점 축**으로 병기해, ATAM 두 설계 비교 시 pass^5가 동률/미실측이면 ②-1로 변별한다. 두 축 모두 정방향.

**조건 (시연 게이트 + 측정 조건 — 표 밖):**
- `캐시 On/Off 갭 Δ > 0이 그래프로 시연됨` — 헤드라인 명제("캐시는 일관성 입증 아님"). Δ 절대값은 등급화 대상 아님(대조 시연 자체가 pass/fail). pass^k는 반드시 캐시 우회(memoization 우회) 순수 추론 모드에서 측정(캐시 On은 ~100%로 자명히 부풀어 측정 오염).
- `k = 5 고정` · `H_norm ≤ 0.2 AND ②-1 대리 유효-결정률 ≥ 95% 동반 통과` — pass^k가 "일관되게 틀린" 결과를 통과시키지 않도록 유효성 게이트 동반(일관 ≠ 정확, QA-07 짝). ②-2 정밀 유효-결정률은 QA-07 golden 의존이라 open-issue(OI-7)로 제외.

| 등급 | 구간 — 주 KPI(main 축): 순수 추론 pass^k (k=5, 캐시 우회) | 필드 근거 (경계 이유 + URL) |
|---|---|---|
| ★★★ (상) | pass^5 ≥ 45% | **(round-04 하향: 60→45%)** frontier(p≈80~85%) + 결정성 레버(temp=0·fixed seed)로 시행 간 약한 양의 상관이 붙으면 pass^5가 독립 가정 0.8^5≈33%보다 위로 끌리는 대역. voting은 pass^k(전 시행 성공)를 직접 못 올리므로(=pass@k 1답 정확도만 올림) 60%는 사문화 → 45%가 근거 있는 상한. GPT-4o pass^4 ~37%보다 위라 변별력 유지. [τ-bench arXiv](https://arxiv.org/pdf/2406.12045) · [pass@k vs voting](https://leehanchung.github.io/blogs/2025/09/08/pass-at-k/) · [Certified Self-Consistency 2510.17472](https://arxiv.org/pdf/2510.17472) |
| ★★☆ (중) | 35% ≤ pass^5 < 45% | **(round-04 압축: 40~60→35~45%, 상한 동반 하향)** GPT-4o retail pass^4 ~37% 관측 대역 — 안정성 레버(temp=0·fixed seed) 적용한 일반 우수 [Sierra τ-bench](https://sierra.ai/blog/tau-bench-shaping-development-evaluation-agents) · [Alan agent benchmarking](https://medium.com/alan/benchmarking-ai-agents-stop-trusting-headline-scores-start-measuring-trade-offs-0fdae3a418cf) |
| ★☆☆ (하) | 25% ≤ pass^5 < 35% — 합격 최소선 (KPI 원본 `pass^k ≥70%`를 현실 재배치) | GPT-4o pass^8 ~25%가 필드 바닥 — pass^5 합격 진입을 그 수준으로. **k 보간 silent cap: 25%는 GPT-4o pass^8 값의 보수적 차용 — k=5 환산 시 GPT-4o pass^5≈33%라 하한에 여유. pass^5 직접 보고치 확보 시 교체.** 단 H_norm ≤0.2·②-1 ≥95% 동반 통과 필수 [τ-bench arXiv](https://arxiv.org/pdf/2406.12045) |
| 불합격 | pass^5 < 25% / 또는 Δ 시연 실패 / H_norm > 0.2 / ②-1 < 95% | pass^k가 필드 worst 이하거나 유효성 게이트 위반 — |

**보조 별점 축 (실측 가능 — round-04 C3):** main 축 pass^5가 동률/미실측일 때 `②-1 대리 유효-결정률(룰 체커 차단율, golden 불요·우리 조건 산출 가능)`로 변별. 정방향(높을수록 ★ 높음).

| 보조 등급 | 구간 — ②-1 룰 게이트 통과율 | 근거 |
|---|---|---|
| ★★★ (상) | ②-1 ≥ 99% | hard-invalid(C-제약·allowlist·스키마 위반) 거의 0 — 결정론적 룰 체커라 우리 조건에서 실측 가능(C-03 차단율·H_norm과 공유 컴포넌트) |
| ★★☆ (중) | 97% ≤ ②-1 < 99% | 룰 게이트 일반 우수 — 드물게 무효 결정 잔존 |
| ★☆☆ (하) | 95% ≤ ②-1 < 97% — 합격 진입선 | §측정 ②-1 하한(≥95%)이 보조 ★☆☆ 진입선 |

> **캘리브레이션 노트**: KPI 원본 보조선 `pass^k ≥70%`는 필드 기준 비현실적으로 높음 — frontier도 retail pass^1 ~80%, GPT-4o는 pass^4 ~37%·pass^8 ~25%라 pass^5 70%는 ★★★조차 죽는 등급. round-03이 표 급간을 현실 대역으로 재배치(25/40/60%). PoC margin = pass^5 절대값은 우리 조건상 실측 불가(실제 에이전트 필요·[발표 서사])라 ★★★를 frontier보다 보수적으로. silent cap: ① pass^5 *절대값*은 미실측(헤드라인 Δ 대조만 시연 — main 축은 "이렇게 등급화한다"는 형식 제시) ② "유사 입력" semantic-equivalent 흔들림은 동일 입력 반복으로 못 봄(범위 밖) ③ ②-2 정밀 유효-결정률은 QA-07 golden 미구축으로 게이트에서 제외(OI-7). importance L 유지(상향은 헤드라인 Δ + ②-1 게이트 선결 — 본 캘리브레이션이 충족 보강). ②-1·H_norm이 우리 조건에서 산출 가능한 gradable proxy라 Constraint flag 불요(QA로 성립).
> **재캘리브레이션(round-04)**: ★★★를 `≥60% → ≥45%`로 하향(★★☆ 동반 압축 `40~60 → 35~45`, ★☆☆ `25~35` 유지). **사유 = voting→pass^k 메커니즘 1차출처 반증**: self-consistency/majority voting은 k샘플을 다수결로 합쳐 *1답의* 정확도(pass@k)를 올릴 뿐 *k회 시행 전부 성공*(pass^k=all-k-succeed)을 직접 못 올린다 — 둘은 다른 지표([Self-Consistency](https://www.emergentmind.com/topics/self-consistency-sampling)·[pass@k vs voting](https://leehanchung.github.io/blogs/2025/09/08/pass-at-k/)·[Certified Self-Consistency 2510.17472](https://arxiv.org/pdf/2510.17472)). 따라서 "voting이 0.8^5≈33%를 60%로 끌어올린다"는 round-03 정당화는 오해 → ★★★를 0.8^5(독립 상한) 약간 위인 45%로 둬 "frontier + 결정성 레버로 시행 간 약한 양의 상관"을 **margin 있게** 반영(사문화·물러짐 동시 회피). **②-1을 보조 별점 축으로 병기**(main=모델 추정·보조=실측 가능 이중 축, C3). **k 보간 silent cap**: ★☆☆ 25%는 GPT-4o pass^8 차용 — k=5 환산 시 pass^5≈33%라 여유, 직접치 확보 시 교체. **apples silent cap(C1)**: 인용 τ-bench는 **retail/airline 고객응대 멀티턴 도구사용**이고 우리는 **SDK 빌드 파이프라인(IR→Optimizer→Quantizer→Compiler)** — "유효 결정" 난이도 프로파일이 달라 대역 차용은 가정, 우리 PoC(캐시 우회 반복시행)로 확정. 본 보정은 OI-9(§측정↔등급표↔변경이력↔짝 QAS Measure↔glossary 5곳 정합) 재점검 대상.
> **seats**: 발의 Seat 1(비결정·일관성·eval 하네스) · consensus (Seat 2가 Δ=시연 게이트·pass^k=유일 gradable main이라 규칙5 적용 타당성 확인 — Seat 3은 보조지표 전원 미실측 시 Constraint flag 우려했으나 ②-1·H_norm이 산출 가능 gradable이라 QA 성립으로 합의)

## 변경 이력

### 2026-06-25 — round-04 디스커션 반영 (★ 등급 척도 근거 보강)
출처: [`discussion/qa/round-04`](../../discussion/qa/round-04/counsel/QA-11-reliability-agent-consistency.md) (red verdict: **Sound ◎ / KPI ○ — Med**; stance: 조건부 채택 — ★★★ 하향·②-1 보조축·apples/k 보간 silent cap). round-04 review의 **가장 날카로운 단일 지적**(★★★ 60% 사문화)에 1차출처로 정면 응답.

**무엇이 문제였나 (review 지적)**
- **★★★ pass^5 ≥60% 사문화 의심**: frontier pass^1 ~80% → 독립시행 0.8^5≈33%인데 voting이 33→60%를 끌어올린다는 근거가 인용에 없음.
- ★☆☆ 25% 하한이 GPT-4o **pass^8** 값이라 k=5 main 축과 k 혼동.
- main 별점 축(pass^5)이 [발표 서사]라 실측 불가 → ②-1/H_norm을 보조 별점 축 병기 권고(C3).
- τ-bench retail vs SDK 빌드 apples-to-apples 미명시(C1). §측정 50행 구값 `pass^k 70%` 잔존.

**무엇을 바꿨나 (반영)**
- **★★★ pass^5 ≥60% → ≥45% 하향**(★★☆ `40~60 → 35~45` 동반 압축, ★☆☆ `25~35` 유지) — voting→pass^k 메커니즘 1차출처 반증(voting=pass@k 1답 정확도, pass^k≠). 45% = frontier+결정성 레버로 시행 간 약한 양의 상관 도달 대역(margin 있는 상한).
- **②-1 룰 게이트 통과율을 보조 별점 축으로 병기**(★★★ ≥99% / ★★☆ 97~99% / ★☆☆ 95~97%) — main=모델 추정·보조=실측 가능 이중 축.
- **k 보간 silent cap**(★☆☆ 25%=pass^8 차용·환산 여유) + **apples silent cap**(τ-bench retail 고객응대 vs SDK 빌드, 우리 PoC로 확정) 명문화.
- **§측정 보정**: 캘리브레이션 노트 줄 `25/40/60%` → `25/35/45%` 재보정 트레이스 추가 + 예시값 꼬리줄 `pass^k 70%` → `pass^5 25/35/45%` 정정.
- 짝 `QAS-11` Measure를 `25/35/45%` + ②-1 보조축으로 동기화.

**남은 일 (이 라운드에서 미반영)**
- pass^5 *절대값* 실측은 여전히 [발표 서사](실제 에이전트 필요) — ②-1 보조축이 실측 변별 대신함.
- **②-2 정밀 유효-결정률 ↔ QA-07 golden** 교차 의존(OI-7) 잔존.
- voting 내장(에이전트가 매 시행 내부 k-샘플 투표)은 신규 tactic 후보 — DP 디스커션 위임(OI-7).
- 경계(25/35/45%·②-1 95/97/99%)는 예시값 — PoC 캐시 우회 반복시행으로 확정.

> 출처: [discussion/qa/round-04](../../discussion/qa/round-04/counsel/QA-11-reliability-agent-consistency.md) (verdict: Sound ◎ / KPI ○ — Med, 조건부 채택).

### 2026-06-24 — 등급 척도(★ rubric) 캘리브레이션
출처: discussion/qa/round-03 (등급 척도 캘리브레이션 — Council blue team, 팀 검토·승인). main 급간 축 = **순수 추론 pass^k (k=5, 캐시 우회)**(정방향, 높을수록 ★ 높음).

- **★ 급간**: ★★★ pass^5 ≥60% / ★★☆ ≥40% / ★☆☆ ≥25%. 하한 미만 불합격.
- **조건/게이트(표 밖)**: 헤드라인 `캐시 On/Off 갭 Δ > 0 시연`은 대조/명제형이라 pass/fail 시연 게이트(규칙5) — 별점은 gradable한 pass^k로 급간화. 측정 조건 `k=5 고정 · 캐시 우회 순수 추론 · H_norm ≤0.2 AND ②-1 ≥95% 동반 통과`(일관 ≠ 정확 유효성 게이트). ②-2는 QA-07 golden 의존이라 제외(OI-7).
- **구→신(§측정 보정)**: 보조 KPI pass^k(k=5) `≥70% → 25/40/60%`(★☆☆/★★☆/★★★ 전면 재배치). τ-bench 대비 70%는 ★★★조차 죽는 비현실(규칙2). 헤드라인 Δ 대조는 시연 게이트라 불변.
- **근거 출처**: τ-bench frontier retail pass^1 ~80% / GPT-4o pass^1 61%→pass^4 ~37%→pass^8 ~25%([Sierra τ-bench](https://sierra.ai/blog/tau-bench-shaping-development-evaluation-agents), [τ-bench arXiv 2406.12045](https://arxiv.org/pdf/2406.12045), [Alan agent benchmarking](https://medium.com/alan/benchmarking-ai-agents-stop-trusting-headline-scores-start-measuring-trade-offs-0fdae3a418cf)).
- pass^k 합격선 재배치(70%→25/40/60%)는 `open-issues.md`(OI-7) 등록 대상(KPI 정의 정합 재확인). ②-1·H_norm이 gradable proxy라 Constraint flag 불요(QA 성립).

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-09-reliability-agent-consistency.md) (red team verdict: **Sound △ / KPI △ — Med** — 일관성/정확성 혼동 + 결정성 목표 재고 + 캐시 치트)

**무엇이 문제였나 (review 지적)**
- **일관성 ≠ 정확성(핵심 결함)**: 일관되게 틀릴 수 있음 — 정확성 앵커 없는 일관성은 약함.
- **캐시 치트**: DP-0005 공유 캐시로 동일 입력 재현율은 자명하게 ~100% → 일관성 입증 아님. 순수 추론(캐시 제외) 재현율을 별도 측정해야.
- **결정성 목표 재고**: LLM 완전 결정성은 비현실 → "같은 출력"이 아니라 "허용 오차 내 유효 결정 안정성"으로 재프레이밍. "성공"의 정의 부재.

**무엇을 바꿨나 (반영)**
- **정의**: "동일/유사 입력에서 유효(정책 허용) 결정의 안정성"으로 재프레이밍(결정성 목표 폐기). 일관성↔정확성(QA-07) 경계를 altitude로 명시.
- **KPI 재설계**: 旧 `재현율 ≥80%` → ① `순수 추론 재현율(캐시 제외) ≥70% pass^k` ② `유효-결정률 ≥95%`(QA-07 golden 연계) ③ `재실행 분산 ≤0.2`.
- 짝 시나리오 `QAS-11`의 자극(유사 입력 추가)·Response·Measure를 캐시 우회 pass^k·유효결정률로 동기화.

**남은 일 (이 라운드에서 미반영)**
- **pass^k 실측은 [발표 서사]로 분기** — 슬라이드 한 줄("DP-0005 캐시를 우회해 동일 입력 k회 반복(pass^k)으로 순수 추론 일관성 측정 — 캐시 100%는 치트임을 대조로 입증"), 본 라운드 실측 미실행.
- **유효-결정률은 QA-07(Correctness) golden 게이트에 의존** → QA-07와 짝지어야 KPI가 닫힘(교차 의존, QA-09 first-pass 게이트와 동일 구조).
- **importance 상향 여지** — 현재 L이나 자율 신뢰 토대라 QA-07와 묶이면 상향 검토(사람 결정).
- "유사 입력" 흔들림 측정(semantic-equivalent 세트)은 범위 밖 — silent cap. pass^k `70%`·k값은 예시값.

> ⚠️ 아래 블록이 위 "남은 일"의 R1·R2·R3을 contention에서 해소함(헤드라인·② 분리·importance 게이트).

### 2026-06-24 — round-01 contention 반영 (QA-11)
출처: [`discussion/qa/round-01/contention`](../../discussion/qa/round-01/contention/counter.md) (rebuttal R1·R2·R3 → counter 수용/부분수용; referee 불요). contention 단계 첫 적용(ASR 트라이얼).

**무엇이 문제였나 (rebuttal 지적)**
- **R1**: 주 KPI(pass^k 절대값)가 우리 조건에선 실측 불가([발표 서사]) — 헤드라인이 "못 만드는 시스템에서만 나오는 값"이라 정합성 깨짐. 분산 ≤0.2의 정규화 척도도 미정의.
- **R2**: 유효-결정률 ②의 앵커(QA-07 golden)가 [생략] → ②는 값 낼 경로 없는 placeholder("일관+정확 닫힘"이 서류상뿐).
- **R3**: R1·R2 미해결인데 importance 상향은 모순.

**무엇을 바꿨나 (counter 반영)**
- **헤드라인 교체(R1 수용)**: 주 KPI = `pass^k 절대값` → **`캐시 On/Off 일관성 갭 Δ` 대조 시연**(명제 "캐시는 일관성 입증 아님"). pass^k 70%는 보조 강등 → 헤드라인이 우리 조건에서 **실제 시연 가능**([발표 서사] 탈출).
- **분산 척도 정의(R1)**: `≤0.2` = **정규화 Shannon 엔트로피 H_norm = −Σp·log p/log|C|** ∈[0,1]. 자유형 출력은 라벨 매핑 선행(silent cap).
- **② 2단 분리(R2 부분수용)**: ②-1 대리 유효-결정률(정책 룰셋/제약 위반 검사, golden 불요 → **[반영]**) + ②-2 정밀(QA-07 의존 → **open-issue 명시 강등**). placeholder를 추적되는 open-issue + 측정 가능한 게이트로 전환.
- **importance 게이트(R3 수용)**: 상향을 R1·R2 해소 **선결 조건**으로(미충족 시 L 유지).
- 짝 `QAS-11` Measure 동기화.

**남은 일**
- **②-2 ↔ QA-07 golden 교차 의존** — `open-issues.md` OI-7 트래킹. QA-07 구축 시 닫힘.
- pass^k 절대값 실측·"유사 입력" 흔들림은 여전히 [발표 서사]/범위 밖(silent cap).
- importance 상향은 선결 2조건(헤드라인 Δ 재정의 + ②-1 게이트 정의) 충족 후 사람 결정.
