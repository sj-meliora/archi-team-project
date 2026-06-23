# Review: QA-07 Performance — Agent 수행 시간

> source: `context/qa/QA-07-performance-agent-time.md` + `QAS-07-performance-agent-time.md`
> verdict: **Sound △ / KPI ✕** — KPI가 테스트 불가능하고 **altitude(관심사 위치)가 의심됨** · severity **High**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트

## 원문 요약
- **정의**: 개발자 직접 수행 대비 단축 최대화.
- **KPI**: `{개발자 노드 수행 시간 − Agent 자동 노드 수행 시간}` **최대화**
- **QAS**: 자극=노드 작업 요청 / 응답=개발자 수동 대비 시간 단축.

## 렌즈 1 — Agentic Workflow 전문가 관점

**진짜 가치는 per-node latency가 아니라 throughput·자율성이다.**
- 단일 노드 1건만 보면 에이전트가 사람보다 **느릴 수도** 있다(LLM 왕복 지연 × 반복 루프 + 도구 실행). 그런데도 시스템이 이기는 이유는 **사람이 못 하는 병렬성**(140모델 × 단계 동시 처리)과 **24/7 무인 진행**이다. 이 QA의 head-to-head 단일노드 프레이밍은 시스템의 실제 가치를 **과소평가**하고, QA-01(throughput)과 충돌한다.
- **속도-품질 trade-off가 빠져 있다.** 빠르지만 틀린 quantize config를 내서 rework를 유발하는 에이전트는 net-negative다. 시간만 측정하면 “빠르게 틀리기”가 최적해로 보인다. 반드시 **성공/품질 게이트와 paired metric**이어야 한다(예: 1-pass 성공한 노드에 한해 시간 비교).
- agent 수행시간의 구성요소(LLM latency × iteration 수 + tool 시간)를 분해해야 개선 레버가 보인다 — “반복 횟수”가 보통 지배적.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

**KPI: `{사람 − 에이전트} 최대화` — 이건 KPI가 아니다.**
- “최대화”는 **optimization goal**이지 **acceptance criterion**이 아니다. QAS Measure는 통과/실패를 가르는 **구체 임계값**이어야 한다. 현 상태로는 어떤 값이 합격인지 누구도 말할 수 없다.
- **이질적 노드의 절대시간 차를 빼는 것**은 의미가 약하다(Compile 노드와 IR 변환 노드의 초 단위를 더해 비교?). 노드별 **speedup 비율**(또는 노드타입별 임계)이 정상.
- **gaming 취약**: 쉬운 노드만 자동화해도 “차이”는 커진다. 커버리지 조건이 없으면 지표가 왜곡된다.

**정의 altitude**: 이 QA는 QA-08(E2E)·QA-01(throughput)과 한 묶음(Performance)인데 셋이 분산돼 경계가 흐리다. → Performance를 **(a) per-node speedup, (b) E2E throughput** 2 sub-metric으로 재편하면 07/08이 자연스럽게 합쳐진다.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

**노드 수행시간 ≠ agent 연산시간 — runner 오버헤드를 분리 계측하라.**
- **지연 분해**: 노드 latency = 큐 대기 + dispatch + worker cold start + **agent 연산** + tool 실행. “단축”을 논하려면 runner 오버헤드(큐·콜드스타트)와 모델 연산을 분리해야 개선 레버가 보인다. warm pool·prefetch로 큐 대기·콜드스타트 절감.
- **병렬성이 진짜 이득**: runner는 모델·노드를 fan-out 병렬 실행해 사람이 못 하는 throughput을 낸다(렌즈1·2의 “throughput이 본질” 주장을 실현하는 메커니즘) — per-node 비교보다 동시 실행도(parallelism factor)가 가치 지표.
- **runner 측 KPI**: 스케줄링 오버헤드 / activity 시간 비율, 큐 대기 p95, 평균 동시 실행도.

## 판정

| 항목 | 판정 | 근거 |
|---|---|---|
| QA 자체가 sound한가 | △ | “자동화로 빨라진다”는 핵심 가치이나, per-node 단일비교는 잘못된 altitude. throughput·자율성으로 재프레이밍 필요 |
| KPI가 측정 가능한가 | ✕ | “최대화”는 임계값 없음 → 테스트 불가 |
| KPI가 현실적/적절한가 | ✕ | 이질 노드 절대시간 차 + 품질 게이트 부재 + gaming 취약 |
| 정의↔KPI↔QAS 일치 | △ | 셋 다 “시간 단축”을 말하나 모두 비측정 표현 |

## Stage 2 권고

- **정의 재서술**: “**1-pass 성공 기준**으로, 자동화 노드의 수행시간을 동일 노드 수동 baseline 대비 일정 비율 이하로 단축한다.”
- **KPI 재설계 (예시)**:
  - 기존 `{사람−에이전트} 최대화` → **`노드타입별 speedup ≥ ◯배`** 또는 **`Agent 노드시간 ≤ 수동 baseline의 30%`**
  - **품질 게이트 동반**: 위 시간지표는 **first-pass 성공한 작업에 한해** 집계(재작업분 제외) → “빠르게 틀리기” 차단
  - 커버리지 명시: 측정 대상 노드 집합을 고정(easy-node cherry-picking 방지)
- **QA-08과 통합 검토**: Performance QA를 하나로 묶고 07=per-node, 08=E2E throughput의 sub-metric으로. README C3 참조.
- DP 연결 점검: DP-0001(즉시 실행)이 latency만 보는지, 품질 게이트와 연결되는지 역검토.
