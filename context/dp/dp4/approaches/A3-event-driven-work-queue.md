# A3 (3안) Event-Driven Work Queue — 메시지 브로커 기반 작업 분배

> category: DP-approach | for: DP-0004 | status: 발굴·평가완료 | updated: 2026-06-20
> 근거 리서치: R-01 | drives: QA-01, QA-08, QA-10, QA-12

## 구조
파이프라인 단계별로 **stage 큐**(converter-q / optimizer-q / quantizer-q / compiler-q)를 두고, Orchestrator(또는 직전 stage 소비자)가 "다음 노드 작업"을 메시지로 발행한다. 각 stage는 **소비자 풀(competing consumers)** 이 큐에서 메시지를 pull → 처리 → 다음 stage 큐로 발행한다. 중간 Artifact(20GB+)는 오브젝트 스토리지에 저장하고 메시지에는 **claim-check 토큰**만 싣는다. 큐 깊이 기반으로 stage별 소비자를 auto-scale 하고, 실패 메시지는 **DLQ**로 격리한다.

## 근거 tactic/pattern
- **Event-Driven Architecture**, **Competing Consumers**, **Queue-based Load Leveling**, **Claim-Check**, **Dead-Letter Queue**, **Idempotent Receiver**.

## 기존 1·2안과의 차별점
- 1안(타입별 풀)은 **동기 push 라우팅** → 본 안은 **비동기 pull + 버퍼링**으로 부하 폭증을 평준화하고 backpressure를 구조적으로 확보.
- 2안(노드 격리)의 장애 독립성을 **브로커 재전달 + DLQ + 멱등**으로 대체(노드 인스턴스를 통째 격리하지 않고도 메시지 단위 격리).

## QA별 장점 / 단점
- **[Scalability QA-01] ★★★**: stage별 소비자를 큐 깊이 기반 독립 auto-scale(필요 시 scale-to-zero) → 모델 폭증 시 병목 stage만 확장, 자원 활용률↑(과프로비저닝↓).
- **[Reliability-Workflow QA-08] ★★☆**: 소비자 인스턴스 장애가 타 Workflow를 막지 않음(살아있는 소비자가 재처리), 독성 메시지는 DLQ 격리. ⚠️ 단 **공유 브로커가 새 장애 전파 경로** — 브로커 다중화/파티셔닝 전제 시에만 ★★★.
- **[Performance-E2E QA-10] ★★☆**: 비동기 디커플링으로 처리량↑, claim-check로 20GB 페이로드를 메시징에서 우회. ⚠️ 단 브로커 hop + 스토리지 왕복으로 **건당 latency 추가** → 전달 오버헤드 ≤5% 목표(QA-10) 검증 필요.
- **[Maintainability QA-12] ★★★**: stage가 **메시지 계약(스키마)** 으로만 결합 → 노드 구현 교체 시 영향 범위가 계약 경계로 한정(Change Impact Scope ≤2 달성에 유리).

## Trade-off 별점
| Performance | Scalability | Reliability-WF | Maintainability |
|:---:|:---:|:---:|:---:|
| ★★☆ | ★★★ | ★★☆ | ★★★ |

## mini-ATAM
- **SP**: ① 브로커 다중화/파티셔닝 여부가 QA-08(중단≤1%)·QA-01에 민감. ② claim-check 스토리지 왕복 횟수가 QA-10(오버헤드≤5%)에 민감.
- **Risk**: 공유 브로커 장애 시 전 stage 정지(SPOF화) → 파티셔닝·다중화로 완화 필요. 메시지 순서 비보장 → 멱등 미보장 노드에서 결과 편차(QA-11 연계).
- **Non-Risk**: C-01(Docker) — 소비자/브로커 모두 컨테이너 구동 가능.

## 인접 DP 정합
- DP-0001 2안(Dynamic Agent Pool)과 자연 정합 — 소비자=Agent 동적 풀. DP-0005 공유 캐시(2안)는 claim-check 스토리지 계층과 결합 가능.
