---
id: QA-13
category: QA
importance: M
difficulty: M
source: discussion/qa/round-01 (신규 — pptx 외 발굴, C5)
iso-25010: "Performance Efficiency / Resource Utilization (자원 활용성)"
related-dp: [DP-0001, DP-0004, DP-0005]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "신설 — 완료 모델당 비용(ROI top-line) 발굴, QA-05 top-line·QA-01 활용률 흡수 (자세히 → ## 변경 이력)"
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "ISO/IEC 25010 Performance Efficiency(Resource Utilization) 앵커링 + 짝 QAS-13 신설"
  - date: 2026-06-24
    by: 팀 결정 (OI-8)
    reason: "정식 QA 편입 — NQA-C → QA-13(우선순위 13위 유지) (자세히 → ## 변경 이력)"
  - date: 2026-06-24
    by: discussion/qa/round-03 (등급 척도 캘리브레이션)
    reason: "★ rubric 추가 + 절감률 합격 하한 70%→50% 재배치(70%는 ★★☆로), $/모델≤$5는 게이트 (자세히 → ## 변경 이력)"
  - date: 2026-06-25
    by: discussion/qa/round-04 (★ 등급 척도 근거 보강)
    reason: "baseline 추정 silent cap + 재시도 오버헤드 보조 별점 축 + ★★★ 75% 도메인무관 분포 재근거(법무 단일사례 강등) + margin 차등 분포 근거화 + OI-9 등록 확인 (자세히 → ## 변경 이력)"
---

# QA-13 Cost-economy — 완료 모델당 비용

> 📌 **정식 QA (2026-06-24 팀 결정, OI-8).** round-01 발굴 → round-02 정식 채택. 우선순위 **13위**(말미 유지, NQA-C → QA-13) — Med, Correctness·보안(C-03)보다 후순위이나 비즈니스 설득에 필수. ASR 비대상(QA-01~05, QA-07만).
>
> **ISO/IEC 25010:2023 앵커**: 주 특성 **Performance Efficiency** / 하위특성 **Resource Utilization(자원 활용성)** — 비용을 "완료 모델당 소비 자원(토큰+compute+재시도)"으로 본다. ⚠️ QA-05(Efficiency)도 같은 ISO 특성이나 **altitude가 다름**: QA-05 = 요청당 토큰(per-request), QA-13 = 완료 모델당 총비용·ROI(business). 경계는 아래 정의의 altitude 줄.

## 정의 / Refinement
모델 1건 완료에 드는 **총비용(LLM 토큰 + runner compute + 재시도)**을 **수작업/기존 파이프라인 대비 절감**한다. 비즈니스 ROI의 top-line은 "완료 모델당 비용"이며, 발표에서 "**수작업 대비 N% 절감**"이 가장 강력한 설득 포인트다.

설계할 때 잡아야 할 두 가지 관점:

- **top-line은 요청당 토큰이 아니라 완료 모델당 비용이다** — 요청당 토큰(QA-05)을 줄여도 **요청 수·재시도가 폭발하면 모델당 비용은 오를 수 있다**. 그래서 altitude를 모델 1건 완료로 올려 잡고, QA-05(요청당 토큰)는 그 하위 tactic(sub-metric)으로 흡수한다.
- **비용을 분해해야 레버가 보인다** — 총비용을 **토큰비 / compute / 재시도 오버헤드**로 분해 집계한다(재시도가 비용을 잠식하므로 분리). QA-01에서 빼낸 "자원 활용률"·QA-05의 worker 가동률이 여기 compute 축으로 흡수된다.

> 이 QA는 "**완료 모델당 총비용·절감률(top-line)**"을 다룬다 — 요청당 토큰·캐시 효율은 QA-05(Efficiency)에서 sub-metric으로 본다. 비용 측정 데이터(결정당 비용/토큰)는 **QA-04(Observability)**가 공급한다.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `완료 모델당 비용($)` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **완료 모델당 비용($) ≤ $5/모델** `[게이트 · PoC 대상]` — 토큰비+compute+재시도 분해 집계 (예시값)
  - 쉽게: 모델 한 건을 끝까지 완성하는 데 드는 총비용(LLM 토큰 + 컴퓨팅 + 재시도)을 정해둔 금액 이하로 잡는다. *예시 $5는 "비용을 이렇게 집계·관리한다"는 모양일 뿐, 실제 단가는 인프라에 따라 변동.*
  > 보정(2026-06-24, 등급 척도 캘리브레이션 round-03): 구 `주 KPI` → 신 `게이트(pass/fail)` — $/모델은 인프라 단가 의존이라 별점 main축 부적합 → pass/fail 게이트로 분리. ★ 급간은 ## 등급 척도 참조.
- **수작업/기존 대비 비용 절감률 ≥ 50%** `[주 KPI(★ 급간 축)]` — 발표 ROI 핵심 (목표선 70%는 ★★☆)
  - 쉽게: 사람이 직접 하거나 기존 방식 대비 비용을 줄인 정도. 합격 진입선은 50%, 발표 목표선 70%는 ★★☆ 우수 구간. (발표 설득의 핵심 수치.)
  > 보정(2026-06-24, 등급 척도 캘리브레이션 round-03): 구 `≥70%` → 신 `≥50%(★☆☆ 합격 하한, 70%는 ★★☆)` — 자동화 절감 필드 30~75%에서 70% 하한은 상단이라 ★★★가 죽어 규칙2로 재배치. ★ 급간은 ## 등급 척도 참조.
- **비용 분해: 토큰비 / compute / 재시도 오버헤드** — 분해 가시화
  - 쉽게: 총비용이 어디서 나오는지(토큰값·컴퓨팅·재시도 낭비) 쪼개서 보여, 어디를 줄일지 드러낸다. *재시도 오버헤드 = 실패해서 다시 하느라 더 든 비용.*

> 위 수치($5/모델·70%)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 우리 측정·인프라 단가로 확정한다(레퍼런스 수치 복제 아님).

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 LLM 비용 관측의 표준에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **$/완료모델 (raw token 아님) + 캐시 분리** | 비용 top-line은 $/task·$/모델이며, prompt caching(cache read ≈ 정상가 10%)을 분리 집계해 절감을 정확히 반영 | [Anthropic — prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) · [Anthropic — pricing](https://platform.claude.com/docs/en/about-claude/pricing) |
| **재시도 오버헤드 분리** | 재시도가 비용을 잠식하므로 분해 집계 — SRE 비용 관측 관행 | round-01 counsel §4 비용·효율 |
| **수작업 baseline 대비 절감률** | "수작업 대비 N% 절감"이 ROI 설득의 핵심 — baseline 비용 추정과 비교 | round-01 counsel §4 (발표 서사) |

> ⚠️ 레퍼런스의 수치(캐시 ~10% 등)는 **패턴 정당화용**이며 그대로 복제하지 않는다. $/모델·절감률은 위 [검증 전략](#검증-전략)의 측정으로 확정한다.

## 검증 전략

> ⚠️ **이 QA의 검증은 [발표 서사]다(QA-05 PoC-E1과 공유).** prompt caching A/B는 QA-05에서 설계한 것을 공유하고, 여기에 compute 단가·**수작업 baseline 비용 추정**을 얹어 절감률을 산출하는 "방법"까지만 남긴다(실측 미실행).

| KPI | 책임지는 설계 (DP 주장) | 검증 방법 (설계) |
|---|---|---|
| **완료 모델당 비용($)** `[주]` | **DP-0001 2안**(작업별 최적 agent = 비용 기준 라우팅) · **DP-0004 A5/A8**(scale-to-zero로 과프로비저닝 0 = compute↓) · **DP-0005 2안**(캐시로 토큰비↓) | **▶ 발표 서사(실측 미실행):** 비용 분해 집계 — OTel `gen_ai.usage.*` 토큰 계측(QA-04) + compute 단가표 + 재시도 카운트로 $/완료모델을 토큰/compute/재시도로 분해. QA-05 prompt caching A/B 위에 얹어 절감 기여 분리. **수작업 baseline은 추정값** |
| 수작업 대비 절감률 ≥ 70% | 위 DP들의 비용 절감 효과 합산 vs 수작업 baseline | 보조(발표 서사): $/완료모델 vs 수작업 baseline 비교 → 절감률 |
| 비용 분해(토큰/compute/재시도) | QA-04 trace(결정당 비용) + 재시도 계측 | 보조(발표 서사): 세 축으로 분해 집계해 재시도 오버헤드 가시화 |

> 가정·한계: **수작업 baseline 비용은 추정 의존**(인건비·시간 가정에 민감) → 가정 명시. **compute 단가는 실제 인프라(자가호스팅 vs 클라우드)에 따라 변동**. 본 라운드는 KPI 정의와 "이렇게 측정하도록 설계했다"는 서사까지만(실행 0건).

## 등급 척도 (★ rubric — ATAM trade-off용)

> 동일 조건 설계 대안의 본 QA 만족도를 ★1~3 비교(별 많은 안 채택). KPI 합격선(하한)=★☆☆ 진입선, ★★☆/★★★는 필드 현실 도달 범위+PoC margin. 하한 미만 불합격. 예시값이며 경계는 PoC로 확정.

KPI가 `$/모델 ≤$5` + `절감률 ≥70%` **2-index** → 규칙4. **절감률(%)을 main축**으로(인프라 단가 비의존, baseline 대비로 PoC 측정 가능), `$/모델`은 게이트(pass/fail)로 분리. 헤드라인 `완료 모델당 비용($)`은 단가 의존이라 별점 main 부적합 → 별점은 gradable한 **수작업/기존 대비 절감률(%)**에 매긴다 (gradable이라 QA 성립). 단 절감률 절대값은 수작업 baseline 추정([발표 서사])에 좌우되므로(silent cap), **재시도 오버헤드 비율을 보조 별점 축**으로 병기해 실측 변별을 보강한다(round-04 C3).

**조건 (2-index):** `조건: 동일 수작업/기존 baseline · 동일 compute 단가표 · 캐시 On 고정` — 절감률은 baseline 정의(인건비·시간 가정)에 민감하므로, 동일 baseline에서만 두 설계 대안의 절감률을 비교해야 공정. `$/완료모델 ≤ $5`는 별도 pass/fail 게이트(단가 변동 흡수, 별점 무관).

| 등급 | 구간 — 주 KPI(main 축): 수작업/기존 대비 총비용 절감률 | 필드 근거 (경계 이유 + URL) |
|---|---|---|
| ★★★ (상) | ≥ 75% | **(round-04 재근거: 도메인 무관 분포)** 자동화 절감 분포(Forrester ~30%·일반 자동화 30~75%)의 **희소 상단(75%+)** — caching 7x↓(≈85%↓)가 토큰축을 끌어내려 도달. 법무 75%는 도메인 다른 **상위 사례 1점**으로 강등(단일 의존 탈피). ([Requesty](https://www.requesty.ai/coding-agent-economy) · [Medium 70% 사례](https://medium.com/@techdigesthq/llm-cost-optimization-the-real-patterns-behind-70-savings-in-august-2025-7967704c121b)) |
| ★★☆ (중) | 60% ~ 75% (margin 15%p) | "수작업 대비 N% 절감" 설득 성립 대역 — 70% 발표 목표선이 이 안에 안착. **margin 15%p > ★☆☆ 10%p 차등 근거: 자동화 절감 분포가 50~75% 구간에 두텁고 75%+가 희소 → 상단 폭을 넓힘**(round-04 C2) |
| ★☆☆ (하) | 50% ~ 60% (margin 10%p) — 합격 최소선=(보정된) KPI 하한 | Forrester ~30%·일반 자동화 상단을 넘어 ROI 설득이 서는 진입선(50% = ROI 진입선). 70%→50%로 하한 보정 |
| 불합격 | 절감률 < 50%, 또는 $/완료모델 > $5(게이트 위반) / baseline·단가 미고정 | baseline 없이 절감률 비교 불가 |

**보조 별점 축 (실측 가능 — round-04 C3):** main 축 절감률은 수작업 baseline 추정([발표 서사])에 좌우 → `$/모델을 토큰비/compute/재시도 3축 분해한 뒤 재시도 오버헤드 비율`을 보조 별점 축으로 병기(재시도율은 baseline 추정 없이 파이프라인 로그에서 산출). 역방향(낮을수록 ★ 높음). ATAM 비교 시 절감률이 추정이면 재시도 오버헤드로 변별(QA-11 ②-1·QA-07 rework와 동형).

| 보조 등급 | 구간 — 재시도 오버헤드 비율 (낮을수록 좋음) | 근거 |
|---|---|---|
| ★★★ (상) | 재시도 오버헤드 ≤ 10% | 재시도로 인한 추가 비용이 거의 없음 — golden·baseline 추정 없이 로그에서 실측 |
| ★★☆ (중) | 10% < 재시도 오버헤드 ≤ 25% | 일반 우수 — 재시도가 비용을 일부 잠식 |
| ★☆☆ (하) | 25% < 재시도 오버헤드 ≤ 40% — 합격 진입선 | 재시도 폭증 직전 — 그 이상은 절감률 붕괴 |

> **캘리브레이션 노트**: 하한 70%는 필드 상단(자동화 절감 30~75%, 대부분 30~50%대)이라 ★★★가 죽음 → **규칙2 재배치**: 합격 하한 70% → 50%(★☆☆ 50~60 / ★★☆ 60~75 / ★★★ ≥75). **이론 근거: 자동화 절감 필드 30~75% · caching effective input 7x↓(Forrester·법무 사례·Requesty) + PoC margin: 수작업 baseline이 추정값이라 절감률 분산이 커 대역을 넓게**. 신뢰도 상한(silent cap): 수작업 baseline 인건비·시간 가정에 절대치가 좌우(발표 서사·실측 미실행). KPI 정의 §측정의 `≥70%`는 위에서 `≥50%`로 보정 트레이스 남김(70%는 ★★☆ 목표선) — open-issues 정합 점검 대상.
> **재캘리브레이션(round-04)**: **★★★ 75% 재근거**(C1·법무 단일 사례 탈피) — 도메인 무관 자동화 절감 분포(Forrester ~30%·일반 자동화 30~75%)의 희소 상단으로 ★★★ 위치를 재확인하고, 법무 75%는 도메인 다른 "상위 사례 1점"으로 강등. **margin 차등 근거화(C2)**: ★★☆ 15%p > ★☆☆ 10%p는 임의가 아니라 `자동화 절감 분포가 50~75% 구간에 두텁고 75%+가 희소 → 상단 폭을 넓힘`(분포 근거). **baseline 추정 silent cap(C3)**: ★ 전 경계는 수작업 baseline([발표 서사] 추정값) 정의에 좌우 — 보수적으로 잡으면 절감률↑(★↑), 절대 경계가 아니라 동일 baseline 고정 하 두 설계 비교용. **재시도 오버헤드 보조 별점 축(C3)**: $/모델을 토큰비/compute/재시도 3축 분해하고 재시도율(로그 산출·baseline 추정 불요)을 보조 별점 축 병기(≤10/25/40%). **OI 등록 확인(권고 4)**: 하한 보정(70→50%)이 OI-9에 등록됐는지 확인 — round-03 변경이력 "범위 외" 표현을 round-04에서 OI-9 round-04 항목에 명시 등록(QA-08~10은 등록 명시였으나 QA-13만 미등록 의심이었음, 본 라운드에서 해소).
> **seats**: 발의 Seat 2(수석 아키텍트 — KPI 측정가능성) · consensus (Seat 1 절감률 main의 PoC 측정가능 동의 / Seat 3 $/모델 단가 변동을 게이트로 분리 동의)

## 변경 이력

### 2026-06-24 — round-01 디스커션 신설
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/NQA-C-cost-economy.md) · [신규 QA 후보](../../discussion/qa/round-01/review/_new-qa-candidates.md) (stance: **신설 — 채택 권장, Med; QA-05 top-line·QA-01 활용률 흡수**)

**왜 신설했나 (review 지적)**
- 비즈니스 ROI top-line = `완료 모델당 비용`인데 QA-05(Efficiency)는 하위 tactic(요청당 토큰)만 봄. 요청당 토큰을 줄여도 요청 수·재시도가 폭발하면 모델당 비용은 오를 수 있음.
- 발표에서 "수작업 대비 N% 비용 절감"이 가장 강력한 설득 포인트.

**무엇을 담았나 (신설 내용)**
- **정의**: 모델 1건 완료 총비용(토큰+compute+재시도)을 수작업 대비 절감. QA-05를 sub-metric으로 흡수, QA-04에서 데이터 수급.
- **KPI 3축**: ① 완료 모델당 비용 ≤$5(주, 예시) ② 수작업 대비 절감률 ≥70% ③ 비용 분해(토큰/compute/재시도).
- 검증은 QA-05 caching A/B 공유 + 수작업 baseline 추정(실행은 [발표 서사]).
- **ISO/IEC 25010 앵커**: Performance Efficiency / Resource Utilization(QA-05와 altitude로 분리). 짝 시나리오 `QAS-13` 신설.

**남은 일 (이 라운드에서 미반영)**
- **KPI 이동 닫기**: QA-05 top-line(`$/완료모델`)·QA-01 "자원 활용률"이 QA-13로 흡수됨 — 두 QA 본문에 이미 "QA-13로 이양" 명시. QA-13 정식 채택 시 이양 확정·cross-link.
- **수작업 baseline 비용·compute 단가는 추정** — 실측 미실행([발표 서사]).
- **번호 재정렬(팀 결정)**: QA-13는 Med라 상위 진입은 QA-06/07보다 후순위 — 확정 전까지 임시 ID.
- 완료 모델당 비용 `$5`·절감률 `70%`는 **예시값**이며 인프라 단가·baseline으로 확정.

### 2026-06-24 — 정식 QA 편입 (팀 결정, OI-8)
출처: 팀 결정 — NQA-C 정식 편입. OI-8 닫음.

**무엇을 바꿨나**
- **ID 확정**: `NQA-C`(임시) → **`QA-13`**(우선순위 13위 유지, 말미). 짝 `QAS-C` → `QAS-13`.
- 위 "남은 일"의 **번호 재정렬(팀 결정)** 항목을 닫음. QA-05 top-line·QA-01 활용률의 QA-13 이양 cross-link 확정.
- **ASR 비대상**(ASR = QA-01~07만) — Cost-economy는 DP 생성 동인이 아님(비즈니스 설득 지표).

**남은 일**
- 수작업 baseline·단가 추정 [발표 서사]·예시값 확정은 종전대로.

### 2026-06-24 — 등급 척도(★ rubric) 캘리브레이션
출처: [`discussion/qa/round-03`](../../discussion/qa/round-03/) (Council 3 seats — reviewer-less ★ rubric 변형, 팀 승인)

**무엇을 했나**
- **★ rubric 추가**: main 급간 축 = `수작업/기존 대비 총비용 절감률(%)`. ★★★ ≥75% / ★★☆ 60~75% / ★☆☆ 50~60%(합격 하한) / 불합격 <50%.
- **규칙4(2-index → main+게이트)**: `절감률(%)`을 별점 main축(인프라 단가 비의존·PoC 측정 가능), `$/완료모델 ≤$5`는 pass/fail 게이트로 분리. 조건: 동일 수작업 baseline·동일 compute 단가표·캐시 On 고정.
- **규칙2(비현실 하한 재배치) — 이번 그룹 핵심 보정**: 절감률 합격 하한 구 `≥70%` → 신 `≥50%`(70%는 ★★☆ 목표선으로 흡수). 사유: 자동화 절감 필드 30~75%에서 70% 하한은 상단이라 ★★★가 죽음. §측정에 구→신 보정 트레이스 남김.
- 근거 출처: [Requesty Coding Agent Economy](https://www.requesty.ai/coding-agent-economy)(caching 7x↓), [Anthropic prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)(cache read=base 10%), 자동화 절감 30~75% 사례.

**남은 일**
- §측정 하한 보정(70%→50%)과 KPI 정의 정합·발표 목표선(70%=★★☆) 표기를 `open-issues.md`와 정합 점검(본 작업 범위 외 — 다른 파일 미수정).

### 2026-06-25 — round-04 디스커션 반영 (★ 등급 척도 근거 보강)
출처: [`discussion/qa/round-04`](../../discussion/qa/round-04/counsel/QA-13-cost-economy.md) (red verdict: **Sound ◎ / KPI ○ — Med**; stance: 조건부 채택 — baseline silent cap·재시도 보조축·★★★ 도메인무관 재근거·OI 등록 확인).

**무엇이 문제였나 (review 지적)**
- ★ 전 경계가 수작업 baseline 추정([발표 서사])에 좌우(C3).
- ★★★ 75%가 법무 단일 사례(Medium 블로그) 의존 + SDK 빌드와 도메인 다름(C1).
- ★★☆ 폭(15%p)>★☆☆(10%p) margin 근거 약함(C2).
- 변경이력 "OI 등록 범위 외" — QA-08~10은 등록 명시인데 **QA-13만 미등록 의심**(권고 4).

**무엇을 바꿨나 (반영)**
- **baseline 추정 silent cap** 명문화(절대 경계 아님·동일 baseline 고정 하 비교용).
- **재시도 오버헤드 보조 별점 축 병기**(≤10/25/40%) — baseline 추정 없이 로그 산출.
- **★★★ 75% 도메인 무관 분포로 재근거**(Forrester ~30%·일반 자동화 30~75% 희소 상단), 법무는 상위 사례 1점으로 강등(단일 의존 탈피). margin 차등(15%p>10%p)을 분포 두께로 근거화.
- **OI-9에 QA-13 하한 보정(70→50%) 등록 확인** — `open-issues.md` OI-9 round-04 항목에 명시(미등록 의심 해소).

**남은 일 (이 라운드에서 미반영)**
- 수작업 baseline·compute 단가 추정은 [발표 서사](실측 미실행).
- DP-0001(비용 라우팅)·DP-0004(scale-to-zero)·DP-0005(캐시) 귀속(OI-7).
- 절감률(50/60/75%)·재시도 오버헤드(10/25/40%) 예시값 — 우리 측정·단가로 확정.

> 출처: [discussion/qa/round-04](../../discussion/qa/round-04/counsel/QA-13-cost-economy.md) (verdict: Sound ◎ / KPI ○ — Med, 조건부 채택).
