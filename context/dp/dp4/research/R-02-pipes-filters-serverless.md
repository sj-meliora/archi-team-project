# R-02 리서치 — Pipes-and-Filters 구조 스타일 + Serverless/Ephemeral 실행 기반

> category: DP-research | for: DP-0004 | updated: 2026-06-20
> 목적: (a) 파이프라인을 **구조적으로 어떻게 분해**할지(Pipes-and-Filters), (b) 노드를 **어떤 실행 기반**에서 돌릴지(상시 풀/노드 vs ephemeral)를 분리해 패턴 조사.

## 조사한 패턴

### 1) Pipes and Filters (파이프-필터)
- **정의**: 복잡한 처리를 **독립·자기완결·대개 무상태(stateless)** 인 filter들로 분해하고, pipe로 연결. filter는 inbound pipe에서 메시지를 받아 outbound pipe로 발행만 하며 **서로를 모름**(입·출력 스키마만 인지).
- **핵심 효과**:
  - **표준 스키마**를 쓰면 filter **재정렬·재사용·삽입/제거**가 쉬움 → compositional reuse. (본 과제의 140+ 모델 × 세대 × 단계 조합 폭발 대응에 직접 유효)
  - filter를 **서로 다른 하드웨어에서 독립 배포·확장**(연산 집약 단계만 고성능/병렬), 가장 느린 filter가 병목 → 그 filter만 병렬 인스턴스로 확장.
  - 스트림 I/O면 단계 **중첩 실행**(filter1이 끝나기 전에 결과를 filter2로) → 처리량↑.
- **주의점**: ⚠️ 대개 **monolithic pipeline** 으로 구현됨 — filter/pipe 하나 실패가 전체 파이프라인 실패로 이어질 수 있어 fault-tolerance·**pipe 데이터 유실 방지 인프라** 필요. filter는 **멱등** 이어야(재처리 시 중복). 외부 상태 load/persist 오버헤드 주의. 큐로 pipe 구현 시 중복 메시지 자동 제거 가능.
- 출처: [Azure — Pipes and Filters](https://learn.microsoft.com/en-us/azure/architecture/patterns/pipes-and-filters)

### 2) Serverless / FaaS + Ephemeral Compute (scale-to-zero)
- **정의**: 노드 작업 호출마다 **ephemeral·stateless 컨테이너 sandbox**(Docker/경량 VM)를 띄워 처리하고 끝나면 정리. pay-per-execution, idle 시 **scale-to-zero**, 트래픽 급증 시 near-infinite auto-scale.
- **구현 옵션**: AWS Fargate/Lambda, **Kubernetes Job**(`RunTask`/Job 객체 — 완료 후 자동 정리), **Knative**(K8s 위 scale-to-zero·이벤트 구동).
- **효과**: 변동 큰 배치 워크로드(밀리초~수시간)에 적합, 내장 retry, 호출 단위 격리. 상시 자원 0 → 활용률·비용 최적.
- **주의점**: ⚠️ **cold start** 지연(대형 컴파일러 이미지/GPU 초기화), 상태/대용량은 외부 스토어(claim-check)로.
- 출처: [Serverless vs Containers (DEV, 2026)](https://dev.to/ripenapps-technologies/serverless-vs-containers-whats-winning-in-2026-556e), [Knative on Kubernetes (CloudRaft)](https://www.cloudraft.io/blog/building-serverless-functions-on-kubernetes-using-knative)

## DP-0004에의 함의 — 결정 축이 사실 2개임을 분리
- 기존 1안·2안·A3는 "노드를 **어디서** 실행하나"(풀/노드/큐 소비자)에 집중.
- **Pipes-and-Filters는 "실행 단위를 어떻게 구조적으로 분해하나"** — 직교적 보강 → 1·2안·A3 위에 얹을 수 있는 **구조 스타일** (A4).
- **Ephemeral 실행 기반**은 상시 점유(1·2안)와 대비되는 **세 번째 실행 기반** — 2안의 자원낭비(R-2)를 정면 해소 (A5).

→ 도출 대안: **[A4] Pipes-and-Filters Stateless Stage**, **[A5] Serverless/Ephemeral Compute-per-Node**.
