---
id: QA-02
category: QA
importance: H
difficulty: H
source: pptx p.13
related-dp: [DP-0002, DP-0003]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "MTTR 단일지표 분해 → 무손실·멱등·가용률·외부장애 4축 재설계 (자세히 → ## 변경 이력)"
---

# QA-02 Availability — 운영 안정성

## 정의 / Refinement
부분 장애(노드·컴포넌트 다운) **및 외부 의존성(LLM 제공자) 장애**가 나도, 진행 중인 작업을 잃지 않고 **멱등하게(같은 일을 두 번 해도 결과가 같게) 재개**해 서비스 연속성을 유지한다.

설계할 때 잡아야 할 두 가지 관점:

- **무엇을 지키나(가용성의 본질)** — 이 시스템에서 모델 1건 처리는 수십 분~수 시간짜리 **long-running·stateful 작업**(빌드·양자화·컴파일)이다. 노드가 죽었을 때 진짜 손해는 1분의 다운타임이 아니라 **반쯤 진행된 20GB 산출물의 유실**(overview의 "Loss" pain)이다. 그래서 가용성의 1순위는 "빨리 재기동"이 아니라 **진행 작업 무손실 + 멱등 재개**다.
- **무엇이 가장 위협하나(지배적 장애원)** — 내 노드 한 대보다 **외부 LLM 제공자**가 더 큰 위협이다. 제공자 outage·rate-limit 폭주 한 번에 전 워크플로우가 동시에 멈춘다. 따라서 외부 장애를 **backoff(점증 재시도)+큐잉으로 흡수해 자동 재개**하는 것까지 가용성에 포함한다.

> 이 QA는 "장애 단위의 **복구**(죽은 걸 되살려 이어가기)"만 다룬다 — 한 장애가 다른 단위로 번지지 않게 막는 "**격리**(blast-radius 차단)"는 QA-06에서 따로 본다(서로 겹치지 않게 분리).

## 측정 (KPI)
- **agent/노드 재기동 ≤ 1분 AND in-flight 작업 손실 = 0** — durable state(영속 상태) 기준
  - 쉽게: 죽은 일꾼(agent/노드)은 1분 안에 되살아나되, 그때 **진행 중이던 작업은 한 건도 잃지 않고** 마지막 저장지점부터 이어가야 한다. *in-flight = 처리 도중(아직 안 끝난) 작업, durable state = 죽어도 안 사라지게 외부에 영속화한 작업 상태.*
- **장애에도 불구한 워크플로우 성공률 ≥ 99.5%** — 가용률 상위 지표(SLO)
  - 쉽게: 중간에 무언가 죽더라도 100건 중 99.5건 이상은 끝까지 성공해야 한다. MTTR만 보면 "자주 죽지만 빨리 복구"도 합격처럼 보이므로, 이 성공률로 그걸 막는다. *SLO = 서비스 수준 목표(달성하기로 약속한 합격선).*
- **외부 LLM 장애 시 자동 재개율 ≥ 95%** — backoff+큐잉으로 흡수, 무한 재시도 폭주 없이
  - 쉽게: LLM 제공자가 잠깐 죽거나 거부(429)해도, 작업의 95% 이상은 사람 손 없이 알아서 다시 이어져야 한다. *backoff = 실패하면 점점 간격을 늘려 다시 시도, 429 = "요청 너무 많음" 거부 응답.*
- **side-effect 멱등성 = 100%** — 재시작 시 중복 배포·중복 쓰기 0건
  - 쉽게: 복구하느라 같은 단계를 다시 실행해도 **배포가 두 번 나가거나 같은 파일이 두 번 써지는 일이 0건**이어야 한다. *side-effect = 외부에 실제로 영향을 주는 동작(배포·파일쓰기·티켓생성), 멱등 = 여러 번 해도 한 번 한 것과 결과가 같음 → exactly-once(딱 한 번) 효과.*

> 위 수치(1분·99.5%·95%·100%)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 실제 환경에서 측정해 확정한다.
> 폐기: `MTTR < 1분`(단독 지표 — *무엇의* MTTR인지 미정의 + half-done compile 복구를 1분에 묶으면 비현실 → 위 4축으로 분해).

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 업계 표준 가용성 설계에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **무손실·멱등 재개 (durable execution)** | 워크플로우 상태를 event history로 영속화하면 worker가 죽어도 다른 worker가 마지막 체크포인트부터 재개. MTTR이 "lease timeout + 재스케줄"로 환원되고 at-least-once 전달 + 멱등 activity = **exactly-once 효과**로 중복 side-effect 0 | [Temporal — Durable Execution이란](https://temporal.io/blog/what-is-durable-execution) · [Temporal 문서](https://docs.temporal.io/temporal) |
| **가용률 ≥ 99.5% (SLO·error budget)** | MTTR 단독이 아니라 가용률(%)·error budget로 명세하는 것이 SRE 표준 — "자주 죽지만 빨리 복구"를 합격으로 통과시키지 않음 | [Google SRE Book — Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) |
| **외부 장애 자동 재개 (retry backoff)** | activity retry policy(exponential backoff + 최대시도)로 외부 제공자 outage를 runner가 흡수 — 무한 재시도 폭주 없이 자동 재개 | [Temporal — Retry policy](https://docs.temporal.io/encyclopedia/retry-policies) |

> ⚠️ 레퍼런스의 수치(가용률 99.5%·재기동 1분 등)는 **패턴 정당화용**이며 그대로 복제하지 않는다. 우리 합격선은 위 [검증 전략](#검증-전략)의 실측·모델로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 시뮬레이션/chaos 모델**로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **재기동 ≤ 1분 AND 손실 = 0** | **DP-0002 3안 H+Standby**(Active-Passive 이중화·상태 외부화로 Availability ★★☆ → ★★★) · **DP-0002 SP-1** Orchestrator 가용성이 전체 MTTR을 좌우 | chaos 모델: durable 워크플로우 엔진(Temporal류) + mock 4단계 파이프라인에서 **worker를 무작위 강제 종료** → lease 만료·재스케줄로 다른 worker가 체크포인트부터 재개 → **손실 건수·재개 시간**을 분포로 산출 |
| **워크플로우 성공률 ≥ 99.5%** | **DP-0002 R-1** Orchestrator SPOF 완화(3안 Standby) · **DP-0003 3안 모니터링**(빠른 탐지·복구로 MTTR 단축) | 위 chaos 모델을 N회 반복해 **장애 주입 하에서의 종단 성공률**을 집계 (failover 시간을 파라미터로) |
| **외부 LLM 자동 재개율 ≥ 95%** | **DP-0003 2안 격리(브로커/프록시)** + **3안 모니터링**으로 외부 장애를 흡수 — 단, **외부 LLM degradation을 명시 안 함**(아래 "남은 일") | fault-injection 모델: LLM 호출 프록시에 timeout/429/5xx 주입 → retry backoff + bounded queue → **outage 종료 후 자동 재개율**과 backpressure 작동(무한 재시도 폭주 없음)을 곡선으로 |
| **side-effect 멱등성 = 100%** | **DP-0003 1안 사전 권한 게이트**(허용 액션만 통과) + 멱등 키 부여한 activity | 위 chaos 모델에서 deploy/쓰기 activity에 멱등 키 부여 → 강제 재실행 후 **중복 배포·중복 쓰기 건수(=0 목표)** 집계 |

> 가정·한계: 서비스 시간·도착률·failover 시간·lease timeout·외부 장애 길이는 **가정 파라미터**다. 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다 — 슬라이드엔 가정값을 명시한다. control-plane 자체의 split-brain·외부 시스템(Jira·빌드서버)이 멱등 키를 실제 존중하는지는 통합 환경이 필요해 미검증.

## 변경 이력

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-02-availability.md) (red team verdict: **Sound ○ / KPI △ — High** — MTTR 단독 지표 분해 + 외부 장애 시나리오 신설 필요)

**무엇이 문제였나 (review 지적)**
- KPI `MTTR < 1분` — *무엇의* MTTR인지 미정의(agent? 노드? 워크플로우 전체?). stateless agent면 1분 가능하나 half-done compile 복구를 1분에 묶으면 비현실.
- 가용성을 MTTR 단독으로만 명세 → "자주 죽지만 빨리 복구"도 합격처럼 보임. **가용률·무손실(resumability)·멱등성**이 빠짐.
- 지배적 시나리오 누락: **외부 LLM 제공자 outage/rate-limit 폭주** 시 전 워크플로우 동시 정지. QAS 자극이 "노드 다운"에만 한정.
- QA-06과 경계 중복(둘 다 fault) — QA-02=복구 / QA-06=격리 명문화 필요.

**무엇을 바꿨나 (반영)**
- **정의**: "외부 의존성(LLM) 장애 포함 + 진행 작업 무손실 + 멱등 재개"로 보강. QA-06(격리)과 경계를 `> altitude` 한 줄로 분리.
- **KPI 분해**: `MTTR<1분` 폐기 → ① `재기동 ≤1분 AND in-flight 손실=0`(대상·조건 명시) ② `워크플로우 성공률 ≥99.5%`(가용률 상위지표) ③ `외부 LLM 장애 자동 재개율 ≥95%`(예시값) ④ `side-effect 멱등성=100%`.
- 짝 시나리오 `QAS-02`의 자극(외부 LLM outage 추가)·Response·Measure를 동일하게 동기화.
- `related-dp`를 `DP-0001, DP-0002, DP-0003` → `DP-0002, DP-0003`으로 조정(DP-0001은 Per-Node 격리 결정으로 QA-02 복구 책임과 직접 연결 약함 — QAS 비고와 검증 전략은 DP-0002 Standby·DP-0003 격리/모니터링 기준으로 재배치).

**남은 일 (이 라운드에서 미반영)**
- **DP-0002(Standby)·DP-0003(격리/모니터링) 모두 "외부 LLM degradation" 시나리오를 명시 안 함** → backoff·폴백 tactic 보강 역검토 필요 (`open-issues.md` OI-7 트래킹 대상). 검증 전략 ③의 "책임지는 설계"가 현재 약한 이유.
- 외부 LLM 장애 자동 재개율의 `95%`·가용률 `99.5%`는 **예시값**이며 실환경 측정으로 확정.
- 폴백 모델 전환 시 결정 일관성(다른 모델로 재개)은 QA-09/NQA-B 영역 — 여기선 미검증(검증 전략의 silent cap).
- QA-06↔QA-02 cross-link은 QA-06 반영 시 양방향으로 박는다(현재 QA-02 정의에 단방향 명시).
