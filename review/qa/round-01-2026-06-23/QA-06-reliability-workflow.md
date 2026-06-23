# Review: QA-06 Reliability — Workflow 간 독립성 보장

> source: `context/qa/QA-06-reliability-workflow.md` + `QAS-06-reliability-workflow.md`
> verdict: **Sound ○ / KPI ○** — 잘 형성된 QA. 단 QA-02와 경계 중복 + 자원 쿼터 격리 차원 누락 · severity **Med**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트

## 원문 요약
- **정의**: 특정 Workflow 장애가 타 Workflow에 무영향.
- **KPI**: 타 Workflow 실행 중단 ≤ 1% · latency 증가 ≤ 10%
- **QAS**: 자극=특정 WF 장애 / 환경=다수 WF 동시 실행 / 응답=격리되어 무영향.

## 렌즈 1 — Agentic Workflow 전문가 관점

**Agentic 시스템의 진짜 공유 장애 도메인은 노드가 아니라 LLM 엔드포인트(rate-limit 풀)와 공유 상태저장소다.** 고전적 noisy-neighbor가 여기선 “**한 워크플로우가 폭주해 전체 토큰 예산/rate-limit을 빨아들여 다른 워크플로우를 굶긴다**”로 나타난다. 즉 WF 독립성은 compute/노드 격리만으로 부족하고 **워크플로우별 토큰·rate-limit 쿼터 격리**를 반드시 포함해야 한다. 현재 KPI엔 이 자원-격리 차원이 없다.
- runaway 워크플로우(→ QA-03 cap)와 직접 연결: cap이 없으면 한 WF가 공유 쿼터를 독점한다.
- 공유 캐시·상태저장소(DP-0005)도 격리 경계여야 한다(한 WF의 오염 데이터가 타 WF로 전파 금지).

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **잘 형성된 fault-isolation QA**: blast-radius 봉쇄라는 단일 관심사가 명확하고, KPI(`타 WF 중단 ≤1%`, `latency ≤10%`)는 구체적·측정 가능. 이 세트에서 드물게 KPI가 건강하다.
- **QA-02와 경계 중복(C3)**: 둘 다 fault를 다룬다. 명문화 필요 — **QA-02 = 장애 단위의 복구(가용성)**, **QA-06 = 타 단위로의 격리(blast radius)**. 교차 참조를 본문에 박아 grep 추적되게.
- 보완: “무영향”의 영향 채널을 열거(자원·상태·스케줄러)하면 KPI가 빠짐없이 커버하는지 검증 가능.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

**워크플로우 격리는 runner의 핵심 역량이다 — 이 QA는 runner 프리미티브로 거의 그대로 매핑된다.**
- **task queue·worker pool 분리**: 워크플로우 타입/테넌트별 전용 큐와 worker pool로 장애·부하 격리(bulkhead).
- **자원 쿼터·동시성 제한**: 큐별 max concurrency, cgroup/네임스페이스/pod limit으로 한 WF가 자원을 독점하지 못하게. **공유 LLM rate-limit 풀은 WF별 token-bucket rate limiter**로 분배(렌즈1의 noisy-neighbor 해법).
- **fair scheduling + circuit breaker**: weighted fair queueing으로 공평 분배, 장애 WF는 circuit breaker로 격리해 재시도 폭주 차단.
- **runner 측 KPI**: 큐 간 간섭률, WF별 쿼터 준수율, 공유 풀에서의 fairness 지수.

## 판정

| 항목 | 판정 | 근거 |
|---|---|---|
| QA 자체가 sound한가 | ○ | 단일 관심사(격리) 명확. 단 QA-02와 경계 정리 필요 |
| KPI가 측정 가능한가 | ○ | `≤1%`, `≤10%` 모두 구체·측정 가능 |
| KPI가 현실적/적절한가 | ○ | 합리적. 단 자원-쿼터 격리 KPI 추가 권장 |
| 정의↔KPI↔QAS 일치 | ○ | 일관 |

## Stage 2 권고

- **KPI 추가**: **`자원 격리: 단일 WF의 토큰/rate-limit 소비가 타 WF 쿼터를 침범 0`**(또는 WF별 쿼터 보장률).
- **경계 명문화(C3)**: QA-02↔QA-06 역할 분리를 양쪽 본문에 cross-link.
- **영향 채널 열거**: 자원/상태(공유 캐시 오염)/스케줄러 격리를 QAS 응답에 추가.
- DP 연결 점검: DP-0004(노드 격리)·DP-0005(로컬 캐시)가 쿼터·상태 격리를 보장하는지 역검토.
