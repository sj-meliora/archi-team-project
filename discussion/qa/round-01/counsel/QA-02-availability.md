# Counsel: QA-02 Availability — 운영 안정성

> refs-review: round-01/review/QA-02-availability.md · report.md(렌즈3 횡단·C3·C4)
> seats: 발의 Seat 3(Runner 인프라) · 합의 consensus (Seat 1 무손실·멱등, Seat 2 가용률 보강)
> stance: 채택 권장 (KPI 분해 + 외부 LLM 장애 시나리오 신설)

## Reviewer 지적 요약
- `MTTR < 1분` 단독은 불완전 — **무엇의 MTTR인지 미정의**, 가용률·무손실(resumability)이 빠짐 (렌즈2).
- agentic 가용성의 본질은 "재기동 속도"가 아니라 **진행 중 long-running 작업 무손실**(20GB compile/quantize 유실 방지) + **멱등 재시작** (렌즈1).
- **지배적 시나리오 누락**: 외부 LLM 제공자 outage/rate-limit 폭주 시 전 워크플로우 동시 정지. QAS 자극이 "노드 다운"에만 한정 (렌즈1).
- durable execution(event sourcing·lease timeout)으로 MTTR을 측정 가능한 형태로 분해 가능 (렌즈3).

## 개선안 (정의·KPI 기존→제안)

**정의**
- 기존: "부분 장애 시 서비스 연속성 보장."
- 제안: "부분 장애 **및 외부 의존성(LLM 제공자) 장애** 시 진행 중 작업을 잃지 않고 멱등 재개로 서비스 연속성을 유지한다."

**KPI**
| # | 기존 | 제안 | 비고 |
|---|---|---|---|
| ① | MTTR < 1분 | **agent/노드 재기동 ≤ 1분 AND in-flight 작업 손실 = 0** (체크포인트 재개) | 대상·조건 명시. durable state 기준 |
| ② | — | **장애에도 불구한 워크플로우 성공률 ≥ 99.5%** | 상위 가용률 지표("자주 죽지만 빨리 복구" 차단) |
| ③ | — | **외부 LLM 장애 시 자동 재개율 ≥ ◯%** (backoff+큐잉) | 신규 시나리오. ◯는 PoC-A2 |
| ④ | — | **side-effect 멱등성 = 100%** (재시작 시 중복 배포 0) | exactly-once 보장. PoC-A1에서 검증 |

## 근거 (레퍼런스)
- **durable execution / event sourcing**: 워크플로우 상태를 event history로 영속화해 worker 사망 시 마지막 체크포인트부터 재개. MTTR을 "lease timeout + 재스케줄"로 환원하고 exactly-once를 보장 — Temporal. (§4 가용성·복구) — https://temporal.io/blog/what-is-durable-execution , https://docs.temporal.io/temporal
- **SLO 가용률 + error budget**: MTTR 단독이 아닌 가용률(%)·error budget로 명세 — Google SRE. (§4) — https://sre.google/sre-book/service-level-objectives/
- **재시도 backoff(외부 의존성)**: activity retry policy(exponential backoff + max attempts)로 외부 outage를 runner가 흡수 — Temporal retry. (§4 가용성)
- ⚠️ 99.5%·1분 등은 우리 시스템 측정으로 확정 — 레퍼런스 수치 복제 아님.

## PoC 증명법

### PoC-A1: 노드 강제종료 후 무손실·멱등 재개를 증명한다 (chaos)
- **가설**: "노드를 강제 종료해도 in-flight 작업 손실 0 + 중복 side-effect 0으로 재개된다."
- **지표**: 작업 손실 건수(=0 목표), 재개 시간(목표 ≤ 1분), 중복 배포/쓰기 건수(=0).
- **셋업**: durable 워크플로우 엔진(Temporal류) + mock 4단계 파이프라인 + chaos(worker pod 강제 kill) + 멱등 키 부여한 deploy activity.
- **절차**: ① long-running WF 실행 중 worker kill → ② lease 만료·재스케줄로 다른 worker가 체크포인트부터 재개 → ③ 손실·중복·재개시간 집계.
- **합격(Exit)**: 손실 0·중복 0·재개 ≤ 1분 재현.
- **규모/기간**: WF 수십 건, kill 시점 무작위 N회, 약 1~2일.
- **리스크/한계(silent cap)**: control-plane(스케줄러 자체) 동시 다중 장애·split-brain은 미검증(leader election 별도 PoC). 멱등 키가 외부 시스템(Jira·빌드서버)에서 실제로 존중되는지는 통합 환경 필요.

### PoC-A2: 외부 LLM 장애 시 graceful degradation·자동 재개를 증명한다 (chaos)
- **가설**: "외부 LLM 제공자 outage/429를 주입해도 backoff+큐잉으로 자동 재개율 ◯%를 달성한다."
- **지표**: 외부 장애 주입 구간의 자동 재개율(%), 데이터 손실 0, 폴백 모델 전환 성공률(있으면).
- **셋업**: LLM 호출에 fault-injection proxy(타임아웃/429/5xx 주입) + retry policy + bounded queue.
- **절차**: ① 정상 처리량 측정 → ② 제공자 outage N초 주입 → ③ 복구 후 자동 재개율·손실 측정.
- **합격(Exit)**: 무한재시도 폭주 없이(backpressure 작동) outage 종료 후 자동 재개율 ◯% 달성.
- **규모/기간**: 단일 엔드포인트, outage 시나리오 2~3종, 약 1일.
- **리스크/한계**: 폴백 모델의 품질 동등성(다른 모델로 재개 시 결정 일관성)은 QA-09/NQA-B 영역 — 여기선 미검증.

## DP·발표 영향
- **DP 연결**: DP-0002(3안 Standby Orchestrator)가 control-plane HA(②③)에, DP-0003(2안 격리·3안 모니터링)이 외부 장애 흡수에 직결. **DP-0002/0003 모두 "외부 LLM degradation" 시나리오를 명시 안 함** → backoff·폴백 tactic 보강 권고(역검토).
- **경계(C3)**: QA-02 = 장애 단위의 **복구**, QA-06 = 타 단위로의 **격리** — 양쪽 본문에 cross-link 박기(QA-06 counsel과 짝).
- **발표**: "1분 복구"가 아니라 "**무손실 재개**"로 메시지 전환 — overview 20GB Loss pain을 정조준하는 서사.
