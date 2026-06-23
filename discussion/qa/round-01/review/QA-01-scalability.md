# Review: QA-01 Scalability

> source: `context/qa/QA-01-scalability.md` + `QAS-01-scalability.md`
> verdict: **Sound △ / KPI ✕** — 품질속성은 살릴 수 있으나 **KPI 전면 재설계 필요** · severity **High**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트

## 원문 요약
- **정의**: 모델·워크플로우 수 증가에 비례해 확장.
- **KPI**: ① 시간당 완료 모델 수 ≥ **N** · ② 자원 활용률 ≥ 70%
- **QAS**: 자극=모델·워크플로우 급증 / 환경=피크 부하 / 응답=자원 비례 확장으로 처리량 유지.

## 렌즈 1 — Agentic Workflow 전문가 관점

**확장성의 진짜 병목은 compute가 아니라 LLM 추론 용량이다.**
- 이 시스템에서 “모델 1건 완료”는 IR→Optimize→Quant→Compile 각 단계에서 다수의 agent 호출을 유발한다. 즉 처리량 상한은 노드/CPU가 아니라 **LLM 엔드포인트의 TPM·RPM(rate limit)**과 동시 세션 수에 의해 결정된다. KPI에 이 차원이 전혀 없다.
- “자원을 비례 확장”(QAS Response)이 자가호스팅 모델이면 GPU 풀, 외부 API면 **rate-limit 쿼터 협상/샤딩** 문제로 바뀐다 — 둘은 확장 전략이 완전히 다르다. QA가 어느 쪽을 가정하는지 명시 필요.
- 피크 부하 시 backpressure/queue depth 관리가 핵심인데(에이전트가 무한 재시도로 rate limit을 악화시키는 패턴) 응답에 없음.

**권고 지표 추가**: `토큰 처리량(tokens/hr)`, `동시 agent 세션 수`, `rate-limit 헤드룸(%)`, `큐 대기시간 p95`.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

**KPI ①: `시간당 완료 모델 수 ≥ N` — N이 미정의 placeholder.**
- 측정 가능한 KPI의 1원칙(구체 임계값)을 위반. QAS Measure에 변수 기호가 남아 있으면 acceptance test를 쓸 수 없다. → 베이스라인(현재 수작업 처리량)을 기준으로 한 **구체값** 또는 **scaling 효율 비율**로 대체.

**KPI ②: `자원 활용률 ≥ 70%` — 확장성 지표가 아니라 비용/효율 지표.**
- 활용률은 “얼마나 빈틈없이 쓰는가”(cost)이지 “부하가 늘 때 처리량이 따라 늘는가”(scalability)가 아니다. 두 관심사를 혼동.
- 게다가 **availability/burst와 상충**: 활용률 70%를 항상 유지하면 피크 흡수용 headroom이 사라진다. 확장성 QA가 가용성을 갉아먹는 KPI를 들고 있는 셈.
- 확장성의 정통 측정은 **scaling efficiency**: 부하 2배 투입 시 자원도 ~2배로 처리량 유지(효율 ≥ 0.8), 또는 동시 워크플로우 X까지 처리량 선형 증가.

**정의의 altitude 문제**: “비례해 확장”은 방향이지 명세가 아니다. *무엇이* 비례하는지(throughput vs latency vs cost)를 못 박아야 QA-07/QA-08(performance)와 경계가 선다.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

**확장성은 runner 아키텍처가 결정한다 — control plane과 data plane을 분리하라.**
- **Pull 기반 worker pool**: 스케줄러(control plane)는 큐만 관리하고 stateless worker가 큐에서 task를 당겨 실행(data plane). worker를 자유롭게 가감 → elastic 수평확장. control plane이 병목/SPOF가 되지 않게 영속 큐 + leader election.
- **오토스케일 신호를 CPU가 아니라 backlog로**: LLM-bound 작업에서 CPU 사용률은 확장 신호로 무의미하다(KPI “활용률 70%” 오용과 같은 뿌리). **큐 깊이·대기시간·in-flight 동시성**으로 scale out.
- **이질 워크로드 분리 스케줄링**: Compile(CPU·메모리 heavy) vs agent 호출(network·IO bound)을 **worker class별로 분리**해야 무거운 compile이 가벼운 agent task를 막지 않는다 — QAS-01의 “타입별 scale out”(DP-0004)이 바로 이것. bin-packing으로 자원을 촘촘히 채우되 burst headroom은 별도 확보.
- **Admission control / bounded queue**: 무한 유입 시 backpressure로 보호(에이전트 무한 재시도가 rate-limit을 악화시키는 패턴 차단).
- **runner 측 KPI**: scale-out latency(용량 추가 시간), 큐 대기 p95, worker 가동률 대비 headroom, control-plane 처리량.

## 판정

| 항목 | 판정 | 근거 |
|---|---|---|
| QA 자체가 sound한가 | △ | 확장성은 이 시스템(조합수 폭발, 140+ 모델)의 1급 관심사로 타당. 단 정의 altitude가 모호하고 throughput 차원으로 좁혀야 함 |
| KPI가 측정 가능한가 | ✕ | `N` placeholder로 테스트 불가 |
| KPI가 현실적/적절한가 | ✕ | 활용률 70%는 확장성 지표 오용 + headroom 상충 |
| 정의↔KPI↔QAS 일치 | ○ | 셋이 같은 것을 가리키긴 함(확장) |

## Stage 2 권고

- **정의 재서술**: “피크 부하에서 자원을 비례 투입해 **처리량을 선형 유지**하며, 병목은 LLM rate-limit 용량으로 본다.”
- **KPI 재설계 (예시)**:
  - 기존 `시간당 완료 모델 수 ≥ N` → **`scaling efficiency ≥ 0.8`** (부하 2배→처리량 ≥1.8배) + 베이스라인 대비 `처리량 ≥ 수작업 대비 ◯배`
  - 기존 `자원 활용률 ≥ 70%` → 이 지표는 **QA-05/cost 계열로 이동**하고, 대신 `rate-limit 헤드룸 ≥ 20%`, `큐 대기 p95 ≤ ◯분`
- **N 확정**: overview의 “모델 140개+”와 발표 목표 처리 기간을 곱해 구체값 산출(예: 목표 = 140모델/2주 → 시간당 ≥ 0.5모델).
- DP 연결 점검: DP-0001(동적 풀)·DP-0004(타입별 scale out)가 rate-limit 차원을 다루는지 역검토.
