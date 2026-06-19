# R-01 리서치 — Event-Driven / Message-Queue 기반 작업 분배 패턴

> category: DP-research | for: DP-0004 | updated: 2026-06-20
> 목적: 노드 작업 실행을 "동기 라우팅(1안)"이 아니라 **메시지 브로커 비동기 분배**로 재구성하는 패턴군 조사.

## 조사한 패턴

### 1) Competing Consumers (경쟁 소비자)
- **정의**: 동일 메시지 채널(큐)에서 다수 소비자 인스턴스가 메시지를 **pull**해 처리. 메시지당 정확히 1개 소비자가 처리(Pub/Sub와 구분 — Pub/Sub는 모든 구독자가 모든 메시지 수신).
- **효과**: 큐가 producer/consumer 사이 **버퍼** → 부하 평준화(Queue-based Load Leveling). 소비자 인스턴스 장애가 producer를 막지 않고, 살아있는 소비자가 메시지 재처리. **큐 깊이 기반 auto-scaling**(scale-to-zero까지) → 활용률↑, 과프로비저닝↓.
- **주의점**: 메시지 순서 비보장 → **멱등 처리** 필요. 독성 메시지(poison)는 **Dead-Letter Queue(DLQ)** 로 격리. 단일 큐가 대규모에서 병목 → **파티셔닝/큐 분할**로 완화.
- **부적합 조건**: 태스크 간 의존도가 높거나, 동기 순차 실행이 강제되는 경우.
- 출처: [Azure — Competing Consumers](https://learn.microsoft.com/en-us/azure/architecture/patterns/competing-consumers)

### 2) Queue-based Load Leveling
- 큐를 완충재로 두어 순간 부하 폭증(모델 폭증·배치 제출)을 소비자 처리율로 평탄화 → 소비자 과부하 방지, 가용성·응답성 보호.

### 3) Claim-Check (대용량 페이로드 분리)
- **정의**: 대용량 페이로드를 외부 데이터 스토어에 저장하고, 메시지에는 **참조 토큰(claim check)** 만 실어 보냄. 브로커는 페이로드를 보지도 저장하지도 않음.
- **본 과제 적합성**: SDK 산출물 **20GB+**(FR-0002)를 메시지로 직접 보낼 수 없음 → Artifact는 오브젝트 스토리지에 두고 메시지엔 참조만. 직렬화/암복호화 병목 회피, 메시징 비용↓, 페이로드 데이터 별도 이중화로 신뢰성↑.
- 출처: [Azure — Claim-Check](https://learn.microsoft.com/en-us/azure/architecture/patterns/claim-check)

### 4) Event-Driven Architecture (style)
- 컴포넌트가 직접 호출 대신 이벤트를 발행/구독 → 느슨한 결합, 독립 확장·독립 진화, 전체 감사 추적(audit trail) 확보. (Kafka 등 토픽 기반 스트리밍 플랫폼이 대표 구현.)

## DP-0004에의 함의
- 파이프라인 4단계(Converter→Optimizer→Quantizer→Compiler)를 **stage별 큐**로 두고 각 stage 소비자 풀이 경쟁 소비하면:
  - 1안(타입별 서버 풀)의 "타입별 확장"은 유지하되, **push 라우팅 → pull 기반**으로 바꿔 부하 평준화·backpressure를 구조적으로 확보.
  - 2안(노드 격리)의 격리 효과는 **DLQ + 멱등 재처리**로 부분 대체.
- 단, **공유 브로커**가 새로운 가용성 의존점이 됨 → ATAM 리스크로 추적 필요(브로커 다중화/파티셔닝 전제).

→ 도출된 대안: **[A3] Event-Driven Work Queue** (approaches/A3-event-driven-work-queue.md)
