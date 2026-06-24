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
---

# QA-01 Scalability — 시스템 확장성

## 정의 / Refinement
처리할 일(모델·워크플로우)이 늘어나면, 자원을 그만큼 더 투입해 **처리 속도가 거의 비례해서 같이 늘도록** 유지한다.

설계할 때 잡아야 할 두 가지 관점:

- **무엇이 먼저 막히나(병목)** — 이 시스템은 서버 CPU가 아니라, **외부 LLM(예: Claude)을 호출하는 양·횟수의 한도**에 먼저 막힌다. LLM 서비스는 보통 *분당 토큰 수*(**TPM**, Tokens Per Minute = 1분에 처리해줄 수 있는 글자량)와 *분당 요청 수*(**RPM**, Requests Per Minute = 1분에 받아줄 수 있는 호출 횟수)에 상한을 둔다. 즉 확장의 진짜 천장은 이 한도다.
- **언제 자원을 더 붙이나(확장 신호)** — CPU 사용률을 보고 늘리는 게 아니라, **처리 대기열에 일이 얼마나 쌓였는지(backlog = 아직 처리 못 하고 줄 서 있는 작업의 양)와 얼마나 오래 기다리는지**를 보고 일꾼(worker)을 늘린다.

> 이 QA는 "처리량(throughput, 단위 시간당 처리한 양)"만 다룬다 — 노드 1건의 처리 속도는 QA-07, 모델 1개의 전체 소요시간은 QA-08에서 따로 본다(서로 겹치지 않게 분리).

## 측정 (KPI)
- **scaling efficiency ≥ 0.8** — 부하 2배 투입 시 처리량 ≥ 1.8배 (USL 기반)
  - 쉽게: 일을 2배로 주면 처리량도 거의 그만큼(예: 1.8배 이상) 따라 늘어야 한다. *USL = 확장성의 정통 측정 모델(Universal Scalability Law).*
- **rate-limit 헤드룸 ≥ 20%** — 피크 시 TPM/RPM 한도 대비 여유, throttle(429) 0건
  - 쉽게: 가장 바쁠 때도 LLM 한도를 80%까지만 쓰고 20%는 비워둔다. 꽉 채우면 호출이 거부(429 에러)돼 줄줄이 실패. *TPM = 분당 토큰 수, RPM = 분당 요청 수.*
- **큐 대기 p95 ≤ 5분** — backlog 오토스케일 트리거 SLI
  - 쉽게: 대기열 작업이 5분 안에 처리되기 시작해야 한다. *p95 = 가장 오래 걸린 상위 5%를 뺀 95% 기준(드문 예외에 안 휘둘리려는 통계), backlog = 처리 대기 중인 작업량, SLI = 서비스 수준을 재는 지표.*
- **(보조·발표 앵커) 베이스라인(수작업) 대비 처리량 배수** — overview "모델 140개+" × 목표 기간으로 산출
  - 쉽게: 사람이 직접 할 때 대비 몇 배 빠른가(발표용 효과 수치).

> 위 수치(0.8·20%·5분)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 실제 환경에서 측정해 확정한다.
> 폐기: `시간당 완료 모델 수 ≥ N`(N=placeholder, 테스트 불가) · `자원 활용률 ≥ 70%`(확장성 아닌 cost 지표 + burst headroom 상충 → NQA-C로 이전).

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
| **scaling efficiency ≥ 0.8** | **DP-0001 2안 동적 풀**(고정배치 ★★☆ → 풀 ★★★) · **DP-0004 A8** 병목 단계만 일꾼(worker) 확장 | ① 큐잉 시뮬: **공유 풀 vs 고정배치** 처리량·효율 비교 ② 4단계 비대칭 파이프라인에서 **병목 단계만 늘렸을 때** 전체 효율 측정 |
| **rate-limit 헤드룸 ≥ 20%** | **DP-0004 R-3**: 자원을 0까지 줄였다 갑자기 폭증하면 호출이 거부(throttle)됨 → **최소 대기 인스턴스(min-instance) 하한**이 필요 | 버스트(순간 폭증) 도착 시뮬: 헤드룸 0%→30% 변화 → **throttle 0을 유지하는 최소 헤드룸**을 곡선으로 도출 |
| **큐 대기 p95 ≤ 5분** | **DP-0001 R-2** 동적 할당 시 라우팅 지연 · **DP-0004 SP-3** 컨테이너 첫 기동 지연(cold-start) | 위 두 시뮬에서 p95 대기 산출 (라우팅·cold-start 지연을 파라미터로 반영) |

> 가정·한계: 서비스 시간·도착률·cold-start·LLM 한도는 **가정 파라미터**다. 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다 — 슬라이드엔 가정값을 명시한다.

## 변경 이력

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-01-scalability.md) (red team verdict: **KPI ✕ / High** — KPI 전면 재설계 필요)

**무엇이 문제였나 (review 지적)**
- KPI ① `시간당 완료 모델 수 ≥ N` — `N`이 미정의 placeholder라 acceptance test 작성 불가.
- KPI ② `자원 활용률 ≥ 70%` — 확장성이 아니라 **cost/효율 지표**의 오용. 70%를 상시 유지하면 피크 흡수용 headroom이 사라져 **가용성과 상충**.
- 정의 "비례해 확장"이 *무엇이* 비례하는지(throughput/latency/cost) 불명 → QA-07·QA-08과 경계 모호. 진짜 병목은 compute가 아니라 **LLM rate-limit**인데 KPI에 부재.

**무엇을 바꿨나 (반영)**
- **정의**: throughput 차원으로 고정 + 1차 병목=LLM rate-limit + backlog 신호 구동 명시.
- **KPI 교체**: `완료모델 ≥N`·`활용률 ≥70%` 폐기 → `scaling efficiency ≥0.8` + `rate-limit 헤드룸 ≥20%` + `큐 대기 p95 ≤5분`. 활용률은 cost 지표이므로 **NQA-C(Cost)로 이전**.
- 짝 시나리오 `QAS-01`의 Response·Measure도 동일하게 동기화.

**남은 일 (이 라운드에서 미반영)**
- DP-0001(동적 풀)·DP-0004(타입별 scale-out)가 rate-limit·admission control 차원을 명시 안 함 → DP 역검토 필요 (`open-issues.md` 트래킹 대상).
- 활용률 KPI 이전은 **NQA-C 신설**이 전제 — 신설 전까지는 잠정 보류.
