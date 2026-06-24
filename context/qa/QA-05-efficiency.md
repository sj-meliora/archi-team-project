---
id: QA-05
category: QA
importance: H
difficulty: H
source: pptx p.13/p.25
related-dp: [DP-0001, DP-0003, DP-0005]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "raw 총토큰 8k → 신규/캐시 토큰 분리집계 + compute 효율, top-line은 NQA-C로 승격 (자세히 → ## 변경 이력)"
---

# QA-05 Efficiency — Agent 토큰·자원 효율

## 정의 / Refinement
모델 1건 완료의 **총비용(LLM 토큰 + runner compute)**을 최소화하되, **prompt caching·context 압축을 페널티 없이** 측정한다. raw 토큰만 세면 캐싱(현대 agentic 효율의 핵심 레버)을 역으로 벌주게 되므로, **신규 토큰과 캐시 토큰을 분리 집계**하는 것이 이 QA의 핵심이다.

설계할 때 잡아야 할 두 가지 관점:

- **raw 토큰이 아니라 비용·캐시 분리로 잰다** — 50k 컨텍스트를 캐시해 비용을 1/10로 줄인 전략이 raw 토큰 지표에선 "나쁨"으로 찍힌다. 빌드 로그·에러 트레이스·config diff를 받는 파이프라인 에이전트는 입력만 8k를 쉽게 넘으므로, "총 토큰 ≤ 8k" 캡은 비현실적이다. **캐시 적중 토큰은 별도 집계**하고, 신규(새로 생성한) 토큰에만 캡을 건다.
- **토큰 효율 ≠ 전부 — runner compute 효율도 비용이다** — cold start마다 컨텍스트·모델을 다시 로딩하면 낭비다. warm worker 재사용·task 친화도 스케줄링·배칭으로 compute를 아끼고, 이를 토큰비와 합쳐 **task/모델당 총비용**으로 본다. (QA-01에서 빼낸 "자원 활용률"이 사실 이 자리다.)

> 이 QA는 "**요청·task 단위의 효율(per-request)**"을 다룬다 — 비즈니스 top-line인 **`$/완료모델`·수작업 대비 절감률**은 altitude가 달라 **NQA-C(Cost-economy)로 승격**한다(NQA-C 신설 전까지 발표 ROI 수치는 잠정적으로 여기서 참조). 결정당 비용/토큰 측정 데이터는 **QA-04(Observability)**가 공급한다.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `신규 토큰 캡 + 캐시 토큰 분리집계` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **신규 토큰 ≤ 6k AND 캐시 토큰 별도 집계** `[주 KPI · PoC 대상]` — 총 토큰 = 입력+출력(명시), 캐시 적중분은 분리
  - 쉽게: 새로 생성·소비한 토큰(캐시로 재사용 안 된 부분)만 6k 이하로 잡고, 캐시에서 값싸게 가져온 토큰은 따로 센다. 그래야 큰 컨텍스트를 캐시하는 좋은 전략이 "토큰 많이 썼다"고 벌점받지 않는다. *prompt caching = 반복되는 긴 입력을 캐시해 두고 정상가의 ~10%로 재사용.*
- **(보조) 작업 난이도별 토큰 tier(★ 척도)** — ★★★ ≤4k(단일 작업) / ★★☆ 4k~6k(일반) / ★☆☆ 6k~8k(복합 추론 허용)
  - 쉽게: 작업이 어려우면 토큰을 더 써도 되게 난이도별로 등급을 둔다. 복합 추론 노드는 캡을 완화. (실용적이라 기존 척도 보존.)
- **(보조) worker 가동률 / task당 compute 비용** — runner 효율
  - 쉽게: 일꾼(worker)이 놀지 않고 얼마나 일하는지, task 1건당 컴퓨팅 비용이 얼마인지. 토큰비와 합쳐 총비용으로 본다. *가동률 = 전체 시간 중 실제 일한 시간 비율.*
- **(이양) 완료 모델당 비용($)** — **→ NQA-C(Cost-economy)로 승격**(본 QA에 남기지 않음)

> 위 수치(6k·tier 4~8k)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 실제 환경에서 측정해 확정한다.
> 폐기: 旧 `단일 요청 총 토큰 ≤ 8k`(raw 총토큰 단독) — 입력+출력 여부 미명시 + **prompt caching을 역페널티**(캐시로 비용 1/10인 전략이 "나쁨"으로 찍힘) + 빌드로그·config diff에 비현실. → 신규/캐시 분리집계로 교정.

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 LLM 비용·효율의 표준 측정 방식에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **신규/캐시 토큰 분리 집계** | raw token이 아닌 $/task가 옳은 효율 척도. prompt caching은 cache read가 정상가의 ~10%(지연도 대폭↓)라, 신규/캐시 토큰을 분리해야 캐싱 전략이 페널티 받지 않음 | [Anthropic — prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) · [Anthropic — pricing](https://platform.claude.com/docs/en/about-claude/pricing) |
| **compute 효율(worker 가동률)** | 토큰비와 별개로 worker 가동률·배칭은 SRE/오토스케일 효율 영역 — task/모델당 총비용으로 통합 | [Google SRE Workbook — SLO 구현](https://sre.google/workbook/implementing-slos/) |
| **top-line $/완료모델은 별 altitude** | 요청당 토큰을 줄여도 요청 수가 폭발하면 무의미 — 비즈니스 효율 top-line은 완료 모델당 비용 → NQA-C로 분리 | round-01 counsel(C5) — NQA-C 신설 |

> ⚠️ 레퍼런스의 수치(캐시 ~10%·8k 등)는 **패턴 정당화용**이며 그대로 복제하지 않는다. 우리 합격선은 위 [검증 전략](#검증-전략)의 실측(A/B)으로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 A/B·계측**으로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **신규 토큰 캡 + 캐시 분리집계** `[주]` | **DP-0005 2안 공유 캐시**(memoization으로 유사 작업 재사용, [Performance]★★★) · **DP-0001 2안 동적 풀**([Efficiency] 작업별 최적 agent 선택 ★★★) | **▶ 실제 제작:** prompt caching On/Off **A/B** — 대표 노드 작업(빌드로그·config diff 포함) N건 × caching On/Off → **$/task·신규 vs 캐시 토큰 비율·TTFT** 측정, 복합 노드의 신규 토큰 분포로 캡 후보 산출 |
| 작업 난이도별 tier | **DP-0001 2안** 작업별 최적 agent(난이도에 맞는 모델 선택) | 보조 모델: 위 A/B를 노드타입별로 분리해 tier(★) 경계가 현실적인지 확인 |
| worker 가동률 / compute 비용 | **DP-0001 2안 동적 풀**(warm pool 재사용으로 cold-start 절감) — routing 지연(SP-1)은 캐싱/사전 워밍으로 완화 | 보조 모델: warm pool On/Off로 가동률·cold-start 비용 비교(QA-01 시뮬과 공유) |

> 가정·한계: 캐시 적중률·작업 다양성·인프라 단가는 **가정 파라미터**다(캐시 적중률은 워크로드 유사성에 의존 — 다양성 높은 실운영에선 절감폭이 다를 수 있음). 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다. compute 비용(보조)은 실제 인프라 단가가 있어야 정밀 — silent cap.

## 변경 이력

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-05-efficiency.md) (red team verdict: **Sound ○ / KPI △ — Med** — altitude 낮음 + 캐싱 역페널티 + top-line 부재)

**무엇이 문제였나 (review 지적)**
- **`총 토큰 ≤ 8k` 비현실 + raw 토큰이 prompt caching을 역페널티** — 캐시로 비용 1/10인 전략이 raw 지표에선 "나쁨"으로 찍힘. "총 토큰" 입력+출력 여부 미명시.
- **Altitude 낮음** — 비즈니스 효율 top-line은 `완료 모델당 비용`인데 KPI는 요청당 토큰(하위 tactic)뿐.
- runner compute 효율(worker 가동률 등)을 토큰비와 합쳐 task/모델당 총비용으로 통합 필요.

**무엇을 바꿨나 (반영)**
- **정의**: 모델 1건 총비용(토큰+compute) 최소화 + 캐싱 페널티 없는 측정으로 재정의. per-request·캐싱 효율 축으로 좁히고 top-line은 NQA-C로 승격 명시.
- **KPI 교정**: 旧 `총 토큰 ≤8k` 폐기 → ① `신규 토큰 ≤6k + 캐시 토큰 별도 집계(입력+출력 명시)` ② tier(★) 표 유지(복합 노드 캡 완화) ③ `worker 가동률/compute 비용`(QA-01에서 빼낸 "활용률"의 올바른 자리). top-line `$/완료모델`은 **NQA-C로 이양**.
- `related-dp`에 **DP-0005 추가**(공유 캐시=캐시 토큰 분리집계와 직결).
- 짝 시나리오 `QAS-05`의 Response·Measure를 신규/캐시 분리·compute 효율로 동기화.

**남은 일 (이 라운드에서 미반영)**
- **DP-0001(2안 작업별 최적 agent)이 토큰 기준인지 비용 기준인지 역검토** — 비용 기준 정렬 권고 (`open-issues.md` 트래킹 대상).
- top-line `$/완료모델` 이양은 **NQA-C 신설이 전제** — 신설 전까지 발표 ROI 수치는 잠정 보유.
- 신규 토큰 `6k`·tier 경계·compute 단가는 **예시값**이며 실환경 A/B로 확정.
