---
id: QA-01
category: QA
importance: H
difficulty: H
source: pptx p.13
related-dp: [DP-0001, DP-0002, DP-0004]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "KPI 측정가능 재설계 (자세히 → ## 변경 이력)"
  - date: 2026-06-24
    by: discussion/qa/round-02
    reason: "닫힘 확인(High 해소) — rate-limit 헤드룸 측정단위 명시(계정전역 TPM/RPM 대비·큐별 쿼터 배분, QA-08 cross-link) Low 보강"
  - date: 2026-06-24
    by: discussion/qa/round-03 (등급 척도 캘리브레이션)
    reason: "★ rubric(ATAM trade-off) 신설 + scaling efficiency 합격 하한 0.8 → 0.70 보정 (필드 USL 근거 + PoC margin, 헤드룸≥20% 고정 조건 하)"
  - date: 2026-06-25
    by: discussion/qa/round-04 (★ 등급 척도 근거 보강)
    reason: "margin 차등 근거화(천장 거리 비례 — ★☆☆ 0.02·★★★ 0.05) + LLM-bound USL apples silent cap(SPARCcenter CPU vs LLM worker) — 급간 수치 불변 (자세히 → ## 변경 이력)"
---

# QA-01 Scalability — 시스템 확장성

## 정의 / Refinement
처리할 일(모델·워크플로우)이 늘어나면, 자원을 그만큼 더 투입해 **처리 속도가 거의 비례해서 같이 늘도록** 유지한다.

설계할 때 잡아야 할 두 가지 관점:

- **무엇이 먼저 막히나(병목)** — 이 시스템은 서버 CPU가 아니라, **외부 LLM(예: Claude)을 호출하는 양·횟수의 한도**에 먼저 막힌다. LLM 서비스는 보통 *분당 토큰 수*(**TPM**, Tokens Per Minute = 1분에 처리해줄 수 있는 글자량)와 *분당 요청 수*(**RPM**, Requests Per Minute = 1분에 받아줄 수 있는 호출 횟수)에 상한을 둔다. 즉 확장의 진짜 천장은 이 한도다.
- **언제 자원을 더 붙이나(확장 신호)** — CPU 사용률을 보고 늘리는 게 아니라, **처리 대기열에 일이 얼마나 쌓였는지(backlog = 아직 처리 못 하고 줄 서 있는 작업의 양)와 얼마나 오래 기다리는지**를 보고 일꾼(worker)을 늘린다.

> 이 QA는 "처리량(throughput, 단위 시간당 처리한 양)"만 다룬다 — 노드 1건의 처리 속도는 QA-09, 모델 1개의 전체 소요시간은 QA-10에서 따로 본다(서로 겹치지 않게 분리).

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `scaling efficiency` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **scaling efficiency ≥ 0.70** `[주 KPI · PoC 대상]` — 부하 2배 투입 시 처리량 ≥ 1.40배 (USL 기반, rate-limit 헤드룸 ≥20% 고정 조건 하)
  - 쉽게: 일을 2배로 주면 처리량도 거의 그만큼(예: 1.4배 이상) 따라 늘어야 한다. *USL = 확장성의 정통 측정 모델(Universal Scalability Law).*
  - > 보정(2026-06-24, 등급 척도 캘리브레이션 round-03): 구 `≥0.8` → 신 `≥0.70` — 실제 USL 회귀 사례(우수 구성도 ~0.72)·가상 PoC 미완성 margin 반영. ★ 급간은 ## 등급 척도 참조.
- **rate-limit 헤드룸 ≥ 20%** — 피크 시 **계정 전역 TPM/RPM 한도** 대비 여유, throttle(429) 0건 (보장 단위: 전역 풀 헤드룸 + 큐별 token-bucket 쿼터 배분)
  - 쉽게: 가장 바쁠 때도 LLM 한도를 80%까지만 쓰고 20%는 비워둔다. 꽉 채우면 호출이 거부(429 에러)돼 줄줄이 실패. 측정 기준은 외부 LLM 제공자의 **계정 전역(account-global) 한도**(TPM/RPM은 계정 단위로 묶이므로 우리 client-side 큐가 아무리 나눠도 천장은 계정 합산)이고, 그 전역 풀을 큐별 token-bucket으로 배분해 한 큐가 다 먹지 않게 한다(WF별 쿼터 격리는 QA-08이 담당 — 같은 token-bucket 인프라 공유). *TPM = 분당 토큰 수, RPM = 분당 요청 수, account-global = 우리 큐가 아니라 LLM 계정 전체에 걸리는 한도.*
- **큐 대기 p95 ≤ 5분** — backlog 오토스케일 트리거 SLI
  - 쉽게: 대기열 작업이 5분 안에 처리되기 시작해야 한다. *p95 = 가장 오래 걸린 상위 5%를 뺀 95% 기준(드문 예외에 안 휘둘리려는 통계), backlog = 처리 대기 중인 작업량, SLI = 서비스 수준을 재는 지표.*
- **(보조·발표 앵커) 베이스라인(수작업) 대비 처리량 배수** — overview "모델 140개+" × 목표 기간으로 산출
  - 쉽게: 사람이 직접 할 때 대비 몇 배 빠른가(발표용 효과 수치).

> 위 수치(0.8·20%·5분)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 실제 환경에서 측정해 확정한다.
> 폐기: `시간당 완료 모델 수 ≥ N`(N=placeholder, 테스트 불가) · `자원 활용률 ≥ 70%`(확장성 아닌 cost 지표 + burst headroom 상충 → QA-13로 이전).

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 업계 표준 측정 방식에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **scaling efficiency (USL)** | 확장성의 정통 척도는 "부하 N배 → 처리량 N배 유지"의 효율. USL(Universal Scalability Law)은 경합·상호간섭으로 확장 한계를 수학적으로 모델링 → 곡선으로 보일 수 있음 | [WSO2 — USL로 확장성 측정](https://wso2.com/blog/research/measuring-software-scalability-using-universal-scalability-law/) |
| **backlog 신호로 오토스케일** | LLM 작업에선 CPU 사용률이 확장 신호로 무의미. 큐 깊이로 scale-out 하는 것이 이벤트·LLM-bound 워크로드의 필드 표준 | [KEDA (Kubernetes 이벤트 기반 오토스케일)](https://keda.sh/) |
| **p95로 대기시간 SLI** | 평균이 아니라 p95/p99 백분위로 지연을 재는 것이 SRE 표준 (드문 지연에 안 휘둘림) | [Google SRE Workbook — SLO 구현](https://sre.google/workbook/implementing-slos/) · [p50 vs p95 vs p99 해설](https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view) |
| **rate-limit 헤드룸** | 외부 LLM 제공자의 TPM/RPM 한도가 agentic 시스템의 실질 처리량 상한 — 이를 1차 병목으로 본 근거 | [Anthropic API rate limits](https://platform.claude.com/docs/en/api/rate-limits) |

> ⚠️ 레퍼런스의 수치(효율 0.8·헤드룸 20% 등)는 **패턴 정당화용**이며 그대로 복제하지 않는다. 우리 합격선은 위 [검증 전략](#검증-전략)의 실측·모델로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 시뮬레이션/모델**로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **scaling efficiency ≥ 0.8** `[주]` | **DP-0001 2안 동적 풀**(고정배치 ★★☆ → 풀 ★★★) · **DP-0004 A8** 병목 단계만 일꾼(worker) 확장 | **▶ 실제 제작:** ① 큐잉 시뮬: **공유 풀 vs 고정배치** 처리량·효율 비교 ② 4단계 비대칭 파이프라인에서 **병목 단계만 늘렸을 때** 전체 효율 측정 |
| rate-limit 헤드룸 ≥ 20% | **DP-0004 R-3**: 자원을 0까지 줄였다 갑자기 폭증하면 호출이 거부(throttle)됨 → **최소 대기 인스턴스(min-instance) 하한**이 필요 | 보조 모델: 버스트(순간 폭증) 도착 시뮬 — 헤드룸 0%→30% 변화 → throttle 0을 유지하는 최소 헤드룸 곡선 |
| 큐 대기 p95 ≤ 5분 | **DP-0001 R-2** 동적 할당 시 라우팅 지연 · **DP-0004 SP-3** 컨테이너 첫 기동 지연(cold-start) | 보조 모델: 위 두 시뮬에서 p95 대기 산출 (라우팅·cold-start 지연을 파라미터로) |

> 가정·한계: 서비스 시간·도착률·cold-start·LLM 한도는 **가정 파라미터**다. 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다 — 슬라이드엔 가정값을 명시한다.

## 등급 척도 (★ rubric — ATAM trade-off용)

> 동일 조건에서 설계 대안의 본 QA 만족도를 ★1~3으로 비교(별 많은 안 채택). **KPI 합격선(하한)이 ★☆☆ 진입선**, ★★☆/★★★는 **필드 기준 현실 도달 범위**로 캘리브레이션. 하한 미만은 불합격(별 없음). 수치는 예시값이며 경계는 PoC로 확정.

**조건 (2-index → main + 조건):** `측정 조건: rate-limit 헤드룸 ≥ 20% 고정`. efficiency를 main 급간 축에 두되, **계정 전역 TPM/RPM 한도가 포화되지 않은(헤드룸 ≥20%) 동일 조건에서만** efficiency를 비교한다. rate-limit 포화로 인한 등급 하락을 배제 — 순수 확장 메커니즘(공유 풀·병목단계 worker 확장)의 효율만 급간화.

| 등급 | 구간 — 주 KPI: scaling efficiency (부하 2배 시 처리량 배수 ÷ 2) | 필드 근거 (왜 이 경계인가 + 출처) |
|---|---|---|
| ★★★ (상) | **efficiency ≥ 0.85** (2배 → ≥1.70배) | 이론·HPC 최상위는 0.90~1.0(linear)지만, PoC는 가상 큐잉 시뮬(가정 파라미터)이라 이론 천장까지 못 올린다 → 이론치(0.90)에서 margin 0.05를 빼 **≥0.85로 하향 배치**. HPC "Good=75%" 위의 excellent 대역. [HemeLB/HPC Carpentries — Benchmarking & Scaling](https://hemelb-dev.github.io/HemeLB-Carpentries/03-benchmarking-and-scaling/index.html) |
| ★★☆ (중) | **0.75 ≤ efficiency < 0.85** (2배 → 1.50~1.70배) | HPC 관례 "Good" 라인 75%가 중급 진입선. 실제 USL 회귀(SPARCcenter SPEC SDM91)에서 우수 구성도 관측 efficiency 최대 **0.72**에 그친 점을 고려, 0.75~0.85를 "필드 일반 우수"로 둔다. [WSO2 — USL로 확장성 측정](https://wso2.com/blog/research/measuring-software-scalability-using-universal-scalability-law/) · [Perfdynamics — How to Quantify Scalability](https://www.perfdynamics.com/Manifesto/USLscalability.html) |
| ★☆☆ (하) | **0.70 ≤ efficiency < 0.75** (합격 최소선 = 보정된 KPI 하한) | KPI 하한을 **0.8 → 0.70으로 하향 보정**: 실제 USL 우수 구성도 0.72에 머문 필드 현실 + PoC 미완성 margin 반영. HPC "Good 75%" 바로 아래까지를 합격으로 허용. [WSO2 — USL](https://wso2.com/blog/research/measuring-software-scalability-using-universal-scalability-law/) |
| 불합격 | **efficiency < 0.70** | coherency(β)·contention(α)이 지배해 자원 추가가 처리량으로 이어지지 않는 영역. [WSO2 — USL](https://wso2.com/blog/research/measuring-software-scalability-using-universal-scalability-law/) |

> **캘리브레이션 노트**: 원 KPI 하한 0.8은 HPC 일반 기준(Good=75% 위)으론 현실적이나, **LLM rate-limited 워크로드 + 가상 PoC에는 낙관적**이다. 실제 USL 회귀 사례는 우수 구성도 **이론 근거 ~0.72**에서 멈췄고, PoC는 가정 파라미터 기반 큐잉 시뮬이라 이론 완성도 미달이 예상된다 → **이론 근거 0.72 + PoC 미완성 대비 margin ≈0.02~0.05 반영해 전 급간을 하향**([상 ≥0.85 / 중 0.75~0.85 / 하 0.70~0.75]). rate-limit 포화로 등급이 깎이는 문제는 표 밖 `측정 조건`(헤드룸 ≥20% 고정)으로 흡수. 경계 수치는 모두 예시값 — PoC 큐잉 시뮬(공유 풀 vs 고정배치)로 확정.
> **margin 차등 근거화(round-04 C2)**: ★★★ 0.05 vs ★☆☆ 0.02 margin 차이는 임의가 아니라 **"천장과의 거리에 비례"** — ★☆☆은 필드 우수(0.72) 바로 아래라 margin 최소(0.02), ★★★는 이론 천장(0.90)이라 PoC margin 크게(0.05). (구 QA-06[→2026-06-26 C-03 제약 이관]의 단일 Y 규칙과 다른 차등 근거 — 천장 종류가 다르므로.) **LLM-bound USL apples silent cap(C1)**: 인용 USL 0.72는 SPARCcenter SPEC SDM91(1990s CPU 벤치)에서 빌려옴 — 우리 병목은 외부 LLM 전역 TPM/RPM이라 contention(α)·coherency(β) 구조가 다름(0.72가 우리 천장 대표 보장 없음). `측정 조건`(헤드룸 ≥20%)으로 rate-limit 포화를 배제한 순수 큐잉 효율임을 명시. ★☆☆ [0.70,0.75) 0.05폭은 좁아 변별 빈약이나, 하한 0.68 하향은 USL 우수도와 더 벌어져 미채택(좁은 채 유지). 급간 수치는 round-03 유지(본 라운드는 근거 보강만 — OI-9 하한 보정 없음).
> **seats**: 발의 Seat 3 (인프라·USL·rate-limit) · Seat 2 동의(SLI 형식) · Seat 1 합의(dissent였던 "rate-limit 조건부 efficiency"가 `측정 조건` 줄로 흡수 → consensus).

## 변경 이력

### 2026-06-24 — 등급 척도(★ rubric) 캘리브레이션
출처: [`discussion/qa/round-03`](../../discussion/qa/round-03/) (등급 척도 캘리브레이션, 팀 승인)

- **main 급간 축**: scaling efficiency (부하 2배 시 처리량 배수 ÷ 2). 2-index 구조에서 PoC로 측정 가능한 efficiency를 main 축, rate-limit 헤드룸은 `측정 조건: 헤드룸 ≥20% 고정`으로 분리(rate-limit 포화로 등급이 깎이지 않게).
- **★ 급간**: 상 ≥0.85 / 중 0.75~0.85 / 하 0.70~0.75 / 불합격 <0.70.
- **§측정 보정(구→신)**: scaling efficiency 합격 하한 `≥0.8` → `≥0.70`. 사유: 실제 USL 회귀 사례(우수 구성도 ~0.72, SPARCcenter SPEC SDM91)·가상 PoC 미완성 margin 반영. HPC "Good=75%" 라인 기준.
- **근거 출처**: HemeLB/HPC Carpentries(Good=75%), WSO2·Perfdynamics(USL 회귀 ~0.72 실측).

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-01-scalability.md) (red team verdict: **KPI ✕ / High** — KPI 전면 재설계 필요)

**무엇이 문제였나 (review 지적)**
- KPI ① `시간당 완료 모델 수 ≥ N` — `N`이 미정의 placeholder라 acceptance test 작성 불가.
- KPI ② `자원 활용률 ≥ 70%` — 확장성이 아니라 **cost/효율 지표**의 오용. 70%를 상시 유지하면 피크 흡수용 headroom이 사라져 **가용성과 상충**.
- 정의 "비례해 확장"이 *무엇이* 비례하는지(throughput/latency/cost) 불명 → QA-09·QA-10과 경계 모호. 진짜 병목은 compute가 아니라 **LLM rate-limit**인데 KPI에 부재.

**무엇을 바꿨나 (반영)**
- **정의**: throughput 차원으로 고정 + 1차 병목=LLM rate-limit + backlog 신호 구동 명시.
- **KPI 교체**: `완료모델 ≥N`·`활용률 ≥70%` 폐기 → `scaling efficiency ≥0.8` + `rate-limit 헤드룸 ≥20%` + `큐 대기 p95 ≤5분`. 활용률은 cost 지표이므로 **QA-13(Cost)로 이전**.
- 짝 시나리오 `QAS-01`의 Response·Measure도 동일하게 동기화.

**남은 일 (이 라운드에서 미반영)**
- DP-0001(동적 풀)·DP-0004(타입별 scale-out)가 rate-limit·admission control 차원을 명시 안 함 → DP 역검토 필요 (`open-issues.md` 트래킹 대상).
- 활용률 KPI 이전은 **QA-13 신설**이 전제 — 신설 전까지는 잠정 보류.

### 2026-06-24 — round-02 디스커션 반영
출처: [`discussion/qa/round-02`](../../discussion/qa/round-02/counsel/QA-01-scalability.md) (red team verdict: **Sound ○ / KPI ○ — Low** · High 해소·닫힘 확인)

**무엇이 문제였나 (review 지적)**
- round-01 KPI 재설계로 High 해소·닫힘. 잔여는 전부 비-verdict: rate-limit 헤드룸 측정단위(전역 vs 큐별) 모호(Low), 활용률 QA-13 이양 미채택 시 부유(C2), rate-limit headroom·admission control·bounded queue DP 미명시(OI-7).

**무엇을 바꿨나 (반영)**
- **Low 보강**: rate-limit 헤드룸의 보장 단위를 명시 — **계정 전역 TPM/RPM 한도 대비** 헤드룸 + 큐별 token-bucket 쿼터 배분. WF별 쿼터 격리는 QA-08이 담당(같은 token-bucket 인프라 공유)임을 cross-link.

**남은 일 (이 라운드에서 미반영)**
- 활용률 QA-13 이양은 **QA-13 정식 채택(OI-8)** 동반 — 미채택 시 부유(C2). 사람 결정.
- rate-limit headroom·admission control·bounded queue tactic이 DP-0001/0004에 미명시(OI-7) → DP 디스커션 위임. bounded queue 없으면 `큐 p95 ≤5분`이 폭주 시 깨짐.
- 부하 단위(정규화 동시 WF 수 또는 토큰 처리량)·예시값은 실환경 측정으로 확정.

### 2026-06-25 — round-04 디스커션 반영 (★ 등급 척도 근거 보강)
출처: [`discussion/qa/round-04`](../../discussion/qa/round-04/counsel/QA-01-scalability.md) (red verdict: **Sound ◎ / KPI ○ — Low**; stance: 조건부 채택 — margin 규칙화·LLM-bound USL silent cap).

**무엇이 문제였나 (review 지적)**
- margin 0.02~0.05 흔들림(★★★ 0.05 vs ★☆☆ 0.02·C2). SPARCcenter CPU USL vs LLM-bound worker apples(C1). ★☆☆ [0.70,0.75) 좁음.

**무엇을 바꿨나 (반영 — 근거 보강만, 급간 수치 불변)**
- **margin 차등을 "천장과의 거리에 비례"로 근거화**(★☆☆ 필드우수 0.72 바로 아래라 0.02·★★★ 이론천장 0.90이라 0.05).
- **LLM-bound USL apples silent cap**: SPARCcenter 1990s CPU 벤치라 contention/coherency 구조 다름 — 측정 조건(헤드룸 ≥20%)으로 rate-limit 포화 배제한 순수 큐잉 효율 명시.

**남은 일 (이 라운드에서 미반영)**
- DP-0001/0004 rate-limit headroom·admission control·bounded queue 미명시(OI-7).
- LLM-bound worker USL 회귀는 미발견 — 우리 PoC로 직접 측정. 경계는 예시값.

> 출처: [discussion/qa/round-04](../../discussion/qa/round-04/counsel/QA-01-scalability.md) (verdict: Sound ◎ / KPI ○ — Low, 조건부 채택).
