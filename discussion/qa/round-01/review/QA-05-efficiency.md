# Review: QA-05 Efficiency — Agent 토큰 사용량

> source: `context/qa/QA-05-efficiency.md` + `QAS-05-efficiency-token-budget.md`
> verdict: **Sound ○ / KPI △** — 측정 가능한 KPI이나 altitude가 낮고(per-request) 캐싱·현실성 미반영 · severity **Med**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트

## 원문 요약
- **정의**: Agent 토큰 사용량 최소화 — 단일 요청 총 토큰.
- **KPI**: 단일 요청 총 토큰 ≤ 8k (★★★ ≤4k / ★★☆ 4k~6k / ★☆☆ 6k~8k)
- **QAS**: 자극=단일 노드 작업 판단 요청 / 응답=토큰 예산 내 완료.

## 렌즈 1 — Agentic Workflow 전문가 관점

**`총 토큰 ≤ 8k`는 빌드/디버깅 컨텍스트에서 비현실적일 수 있다.** 파이프라인 에이전트는 빌드 로그·에러 트레이스·config diff를 입력으로 받는데, 이것만 8k를 쉽게 넘긴다. “총 토큰”이 입력+출력이면 복합 추론 노드에서 캡이 너무 빡빡하다 — tier 표의 “복합 추론 6~8k”라는 단서 자체가 “어려운 작업엔 8k도 부족”을 시사한다.
- **raw 토큰 측정은 prompt caching을 역으로 페널티한다.** 50k 컨텍스트를 캐시해 비용은 1/10인 전략이, raw 토큰 지표에선 “나쁨”으로 찍힌다. 현대 agentic 효율의 핵심 레버(prompt caching, context compaction, retrieval)를 측정이 죽이는 셈.
- 따라서 효율은 **비용($) 또는 캐시/신규 토큰 분리 측정**이 옳다. 캐시 적중 토큰은 별도 집계.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **Altitude 문제**: 비즈니스 효율 = **완료 모델당 비용**(E2E)이지 요청당 토큰이 아니다. 요청당 토큰을 줄여도 요청 수가 폭발하면 의미 없다. top-line KPI는 `$/완료모델`(또는 토큰/모델), `≤8k/request`는 그 하위 tactic.
- **모호성**: “총 토큰”이 입력+출력인지 명시 안 됨. 측정 단위를 못 박아야.
- 긍정: tier(★) 척도는 작업 난이도별 차등이라 실용적 — 살릴 가치 있음.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

**토큰 효율과 별개로, runner는 자원(compute) 효율을 책임진다 — 둘을 한 비용지표로 합쳐라.**
- **worker 가동률·warm pool**: cold start마다 컨텍스트·모델 로딩을 반복하면 낭비 → warm worker 재사용, task 친화도(affinity) 스케줄링. (QA-01에서 빼낸 “자원 활용률”은 사실 이 runner 효율 지표의 자리다.)
- **배칭·co-location**: 동종 LLM 요청 배칭, 의존 단계 co-location으로 왕복·전송 절감.
- **spot/preemptible + 우선순위 큐**: 비긴급 task는 저비용 자원으로, durable state가 있으니 선점돼도 재개 가능.
- **runner 측 KPI**: worker 가동률 vs idle, task당 compute 비용, 배칭 비율 → 토큰비와 합쳐 **task/모델당 총비용**(C5 NQA-C)으로 통합.

## 판정

| 항목 | 판정 | 근거 |
|---|---|---|
| QA 자체가 sound한가 | ○ | 토큰=직접 비용이라 효율은 타당한 QA. 단 altitude 낮음 |
| KPI가 측정 가능한가 | ○ | 토큰 수는 명확히 측정 가능(입력+출력 정의만 보완) |
| KPI가 현실적/적절한가 | △ | 8k 캡이 복합 노드엔 빡빡; 캐싱 미반영; top-line 부재 |
| 정의↔KPI↔QAS 일치 | ○ | 셋 다 “요청당 토큰”으로 일관(단 협소) |

## Stage 2 권고

- **top-line KPI 신설**: **`완료 모델당 비용($) ≤ ◯`**(또는 토큰/모델), 수작업 대비 절감률(→ C5 NQA-C와 통합 가능).
- **per-request KPI 보완**:
  - “총 토큰” = **입력+출력** 명시
  - **캐시 토큰 vs 신규 토큰 분리 집계**(캐싱 전략이 페널티 받지 않게)
  - 복합 추론 노드는 캡 완화/별도 tier 인정
- **연계**: QA-04(결정당 비용 trace)에서 측정 데이터 수급.
- DP 연결 점검: DP-0001(작업별 최적 agent 선택)이 비용 기준인지 토큰 기준인지 역검토.
