# Review: QA-02 Availability — 운영 안정성

> source: `context/qa/QA-02-availability.md` + `QAS-02-availability.md`
> verdict: **Sound ○ / KPI △** — 품질속성은 타당하나 KPI가 불완전하고 핵심 시나리오(외부 장애)가 빠짐 · severity **High**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트

## 원문 요약
- **정의**: 부분 장애 시 서비스 연속성 보장.
- **KPI**: MTTR < 1분
- **QAS**: 자극=부분 장애(노드·컴포넌트 다운) / 환경=정상 운영 중 / 응답=연속성 유지 + 자동 복구.

## 렌즈 1 — Agentic Workflow 전문가 관점

**Agentic 파이프라인의 가용성은 “재기동 속도”가 아니라 “진행 중 작업을 잃지 않는가”다.**
- 모델 1건 처리는 수십 분~수 시간의 long-running·stateful 작업이다. 노드가 죽었을 때 진짜 손해는 1분의 다운타임이 아니라 **반쯤 진행된 compile/quantize 작업의 유실**이다(overview의 “20GB 산출물 Loss” pain과 직결). 따라서 가용성 KPI는 **checkpoint/resume·작업 유실 0**을 1순위로 담아야 한다.
- **재시작이 멱등(idempotent)하지 않으면 MTTR는 무의미하다.** 1분 만에 복구해서 비멱등 deploy를 재실행하면 중복 배포가 난다. 자동복구는 exactly-once side-effect 보장과 한 묶음.
- **가장 큰 가용성 위협은 내 노드가 아니라 외부 LLM 제공자다.** rate-limit 폭주·제공자 outage 시 전 워크플로우가 동시에 멈춘다. QAS의 자극이 “노드·컴포넌트 다운”에만 한정돼 이 지배적 시나리오가 빠졌다. → **외부 의존성 장애 시 graceful degradation**(backoff+큐잉, 폴백 모델, 일정시간 내 자동 재개)을 QAS에 추가해야 한다.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

**KPI: `MTTR < 1분` — 단독 지표로는 불완전하고, 무엇의 MTTR인지 미정의.**
- 가용성은 보통 **가용률(%)과 MTTR을 함께** 명세한다. MTTR만 있으면 “자주 죽지만 빨리 복구”도 합격처럼 보인다. `워크플로우 성공률(장애에도 불구하고) ≥ 99.x%` 같은 상위 지표 필요.
- **MTTR의 대상이 불명확**: agent 프로세스 재기동? 노드? 워크플로우 전체? stateless agent 재기동이면 1분이 가능하나, 손상된 half-done compile 복구가 1분이면 비현실적. 범위를 못 박아야 테스트 가능.
- **QA-06과 경계 중복**(둘 다 fault). 명문화 필요: **QA-02 = 장애 단위의 복구**, QA-06 = 타 단위로의 격리(C3 참조).

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

**“진행 중 작업 무손실”의 정공법은 durable execution(영속 실행)이다.**
- **이벤트 소싱형 워크플로우 엔진**(Temporal·Cadence류): 워크플로우 상태를 이벤트 히스토리로 영속화 → worker가 죽어도 다른 worker가 **마지막 체크포인트부터 재개**. MTTR은 사실상 “lease timeout + 재스케줄 시간”으로 환원된다.
- **Heartbeat + lease/visibility timeout**: 죽은 worker의 task는 lease 만료 후 자동 재큐잉. at-least-once 전달 + **멱등 activity** = exactly-once 효과(렌즈1의 멱등성 요구를 아키텍처로 보장).
- **control plane HA**: 스케줄러 다중화·leader election·영속 큐로 SPOF 제거. 외부 LLM 장애(렌즈1 지적)는 activity의 retry policy(exponential backoff + 최대시도)로 runner가 흡수.
- **runner 측 KPI**: 작업 손실 = 0(durable state 기준), task 재스케줄 시간, control-plane failover 시간 → “MTTR < 1분”을 측정 가능한 형태로 분해해 준다.

## 판정

| 항목 | 판정 | 근거 |
|---|---|---|
| QA 자체가 sound한가 | ○ | 무인 자율 운영에서 가용성은 1급 관심사. 단 06과 경계 정리 필요 |
| KPI가 측정 가능한가 | △ | MTTR은 측정 가능하나 대상 범위 미정의 → 그대로는 모호 |
| KPI가 현실적/적절한가 | △ | 1분 자동복구는 대상에 따라 비현실; 가용률·resumability 누락 |
| 정의↔KPI↔QAS 일치 | △ | 정의/QAS는 “연속성+자동복구”인데 KPI는 MTTR 1개만 대표 |

## Stage 2 권고

- **정의 보강**: “부분 장애 및 **외부 의존성 장애** 시 진행 중 작업을 잃지 않고 서비스 연속성을 유지한다.”
- **KPI 재설계 (예시)**:
  - `MTTR < 1분` → **`agent/노드 재기동 ≤ 1분 AND in-flight 작업 손실 = 0(체크포인트 재개)`** (대상·조건 명시)
  - 상위 지표 추가: **`장애에도 불구한 워크플로우 성공률 ≥ 99.5%`**
  - 신규 시나리오 KPI: **`외부 LLM 장애 시 자동 재개율 ≥ ◯% (backoff+큐잉)`**, 데이터/배포 멱등성 = 100%
- **QAS-02 보강**: 자극에 “외부 LLM 제공자 outage/rate-limit”을 추가.
- DP 연결 점검: DP-0002(Standby)·DP-0003(격리/모니터링)이 외부 의존성 degradation을 다루는지 역검토.
