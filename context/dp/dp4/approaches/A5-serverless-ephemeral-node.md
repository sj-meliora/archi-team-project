# A5 (5안) Serverless / Ephemeral Compute-per-Node — 호출 단위 일시 실행

> category: DP-approach | for: DP-0004 | status: 발굴·평가완료 | updated: 2026-06-20
> 근거 리서치: R-02 | drives: QA-01(주), QA-08, QA-10, QA-12

## 구조
노드 작업 **호출마다** ephemeral 컨테이너(Kubernetes Job / Knative / FaaS)를 띄워 처리하고 종료한다. 호출당 독립 sandbox로 격리되며, idle 시 **scale-to-zero**, 부하 급증 시 near-infinite auto-scale. 중간 Artifact(20GB+)는 오브젝트 스토리지(claim-check)로 전달하고, 내장 retry로 노드 단위 복구한다.

## 근거 tactic/pattern
- **Function-as-a-Service**, **Ephemeral instance**, **Scale-to-zero**, **Bulkhead(invocation 단위 격리)**, **Built-in retry**.

## 기존 1·2안과의 차별점
- 1안(상시 타입 풀)·2안(상시 노드 인스턴스)은 **상시 자원 점유** → A5는 **상시 자원 0, 호출 시에만 점유**. 2안의 핵심 단점인 "Workflow 단위 복제로 자원낭비(R-2)"를 정면 해소하면서, 격리는 2안만큼(혹은 더 강하게) 유지.

## QA별 장점 / 단점
- **[Scalability QA-01] ★★★ (주 강점)**: 호출 단위 near-infinite auto-scale + scale-to-zero → 과프로비저닝 0에 수렴, 자원 활용률 극대화. 모델 폭증(배경 워크로드 축) 대응 최적.
- **[Reliability-Workflow QA-08] ★★★**: 노드 호출마다 **독립 sandbox** → 장애가 그 invocation에만 격리(2안보다 미세한 bulkhead), 자동 retry·정리. 타 Workflow 무영향(중단≤1%에 유리).
- **[Performance-E2E QA-10] ★★☆**: ⚠️ **cold start** 지연이 핵심 약점 — 대형 컴파일러 이미지/GPU 초기화 시 건당 latency 급증 → pre-warming/min-instance 필요. claim-check로 20GB 전달은 우회.
- **[Maintainability QA-12] ★★☆**: 함수/Job 단위 독립 배포로 교체 용이. ⚠️ 단 FaaS 제어 플레인·트리거 바인딩·관측 등 운영 복잡성 추가.

## Trade-off 별점
| Performance | Scalability | Reliability-WF | Maintainability |
|:---:|:---:|:---:|:---:|
| ★★☆ | ★★★ | ★★★ | ★★☆ |

## mini-ATAM
- **SP**: **cold start 시간**이 QA-10(E2E)·QA-09(Agent 수행시간)에 강하게 민감. min-instance/pre-warm 정책이 비용↔latency 교환점.
- **Risk**: 대형 이미지 cold start로 E2E 목표 잠식 / scale-to-zero 후 동시 폭증 시 throttling. → pre-warm pool, 이미지 슬림화로 완화.
- **Non-Risk**: C-0001(Docker)·C-0002(이식성) — K8s Job/Knative는 컨테이너 기반·이식성 우수.

## 인접 DP 정합
- DP-0001 2안(Dynamic Agent Pool)과 정합 — Agent를 ephemeral invocation으로. A3(큐) 트리거로 호출하면 Knative 이벤트 구동과 자연 결합. 2안 대비 활용률↑이라 DP-0001 R-1(활용률 미달) 완화에도 기여.
