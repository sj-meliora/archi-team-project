---
id: NQA-C
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
    reason: "ISO/IEC 25010 Performance Efficiency(Resource Utilization) 앵커링 + 짝 QAS-C 신설"
---

# NQA-C Cost-economy — 완료 모델당 비용

> ⚠️ **신규 QA (번호 미확정).** `NQA-C`는 임시 ID다. 우선순위 Med(Security·Correctness보다 후순위이나 비즈니스 설득에 필수) — 재번호는 팀 결정 사항이라 확정 전까지 `NQA-C`로 둔다.
>
> **ISO/IEC 25010:2023 앵커**: 주 특성 **Performance Efficiency** / 하위특성 **Resource Utilization(자원 활용성)** — 비용을 "완료 모델당 소비 자원(토큰+compute+재시도)"으로 본다. ⚠️ QA-05(Efficiency)도 같은 ISO 특성이나 **altitude가 다름**: QA-05 = 요청당 토큰(per-request), NQA-C = 완료 모델당 총비용·ROI(business). 경계는 아래 정의의 altitude 줄.

## 정의 / Refinement
모델 1건 완료에 드는 **총비용(LLM 토큰 + runner compute + 재시도)**을 **수작업/기존 파이프라인 대비 절감**한다. 비즈니스 ROI의 top-line은 "완료 모델당 비용"이며, 발표에서 "**수작업 대비 N% 절감**"이 가장 강력한 설득 포인트다.

설계할 때 잡아야 할 두 가지 관점:

- **top-line은 요청당 토큰이 아니라 완료 모델당 비용이다** — 요청당 토큰(QA-05)을 줄여도 **요청 수·재시도가 폭발하면 모델당 비용은 오를 수 있다**. 그래서 altitude를 모델 1건 완료로 올려 잡고, QA-05(요청당 토큰)는 그 하위 tactic(sub-metric)으로 흡수한다.
- **비용을 분해해야 레버가 보인다** — 총비용을 **토큰비 / compute / 재시도 오버헤드**로 분해 집계한다(재시도가 비용을 잠식하므로 분리). QA-01에서 빼낸 "자원 활용률"·QA-05의 worker 가동률이 여기 compute 축으로 흡수된다.

> 이 QA는 "**완료 모델당 총비용·절감률(top-line)**"을 다룬다 — 요청당 토큰·캐시 효율은 QA-05(Efficiency)에서 sub-metric으로 본다. 비용 측정 데이터(결정당 비용/토큰)는 **QA-04(Observability)**가 공급한다.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `완료 모델당 비용($)` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **완료 모델당 비용($) ≤ $5/모델** `[주 KPI · PoC 대상]` — 토큰비+compute+재시도 분해 집계 (예시값)
  - 쉽게: 모델 한 건을 끝까지 완성하는 데 드는 총비용(LLM 토큰 + 컴퓨팅 + 재시도)을 정해둔 금액 이하로 잡는다. *예시 $5는 "비용을 이렇게 집계·관리한다"는 모양일 뿐, 실제 단가는 인프라에 따라 변동.*
- **수작업/기존 대비 비용 절감률 ≥ 70%** — 발표 ROI 핵심
  - 쉽게: 사람이 직접 하거나 기존 방식 대비 비용을 70% 이상 줄여야 한다. (발표 설득의 핵심 수치.)
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
- **ISO/IEC 25010 앵커**: Performance Efficiency / Resource Utilization(QA-05와 altitude로 분리). 짝 시나리오 `QAS-C` 신설.

**남은 일 (이 라운드에서 미반영)**
- **KPI 이동 닫기**: QA-05 top-line(`$/완료모델`)·QA-01 "자원 활용률"이 NQA-C로 흡수됨 — 두 QA 본문에 이미 "NQA-C로 이양" 명시. NQA-C 정식 채택 시 이양 확정·cross-link.
- **수작업 baseline 비용·compute 단가는 추정** — 실측 미실행([발표 서사]).
- **번호 재정렬(팀 결정)**: NQA-C는 Med라 상위 진입은 NQA-A/B보다 후순위 — 확정 전까지 임시 ID.
- 완료 모델당 비용 `$5`·절감률 `70%`는 **예시값**이며 인프라 단가·baseline으로 확정.
