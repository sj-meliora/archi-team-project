---
id: QA-08
category: QA
importance: M
difficulty: H
source: pptx p.13
related-dp: [DP-0004, DP-0005]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "건강한 KPI 유지 + 자원-쿼터 격리·캐시오염 KPI 추가 + QA-02 경계 명문화 (자세히 → ## 변경 이력)"
  - date: 2026-06-24
    by: discussion/qa/round-02
    reason: "닫힘 확인(Med→Low·세트 모범) — 외부 rate-limit 계정전역 silent cap 명문화(QA-01 헤드룸 cross-link) Low 보강"
  - date: 2026-06-24
    by: 팀 결정 (OI-8)
    reason: "재번호 QA-06 → QA-08 (NQA 정식 편입에 따른 +2 시프트, → changelog)"
  - date: 2026-06-24
    by: discussion/qa/round-03 (등급 척도 캘리브레이션)
    reason: "★ rubric 추가 + 타 WF latency 증가 합격 하한 ≤10% → ≤25%(★☆☆, 10%는 ★★☆) 보정 (중단율 ≤1%·쿼터침범 0은 조건/게이트 불변) (자세히 → ## 변경 이력)"
  - date: 2026-06-25
    by: discussion/qa/round-04 (★ 등급 척도 근거 보강)
    reason: "★★★ ≤5% margin 근거화(tail 안정화+폭주 강도 흡수) + arXiv 2604.03145 미래형 ID 검증결과·noisy-neighbor 도메인 apples silent cap + 쿼터 0건은 절대형이라 Constraint 유지(별점화 불가) — 급간 수치 불변 (자세히 → ## 변경 이력)"
---

# QA-08 Reliability — Workflow 간 독립성 보장

## 정의 / Refinement
특정 Workflow의 **장애·자원 폭주**가 타 Workflow의 **실행·지연·토큰/rate-limit 쿼터·공유 상태**에 영향을 주지 않도록 격리한다(**blast-radius 봉쇄** = 폭발 반경 차단). 자율 에이전트 환경의 진짜 공유 장애 도메인은 노드가 아니라 **공유 LLM rate-limit 풀과 공유 캐시/상태저장소**다.

설계할 때 잡아야 할 두 가지 관점:

- **noisy-neighbor의 실체는 쿼터 독점이다** — 고전적 "시끄러운 이웃"이 여기선 "**한 워크플로우가 폭주(무한 재시도)해 전체 토큰 예산·rate-limit을 빨아들여 다른 워크플로우를 굶긴다**"로 나타난다. 따라서 격리는 compute/노드 격리만으론 부족하고, **워크플로우별 토큰·rate-limit 쿼터 격리(token-bucket)**를 반드시 포함한다. (runaway 폭주 차단은 QA-03 cap과 직접 연결 — cap이 없으면 한 WF가 공유 쿼터를 독점한다.)
- **공유 상태도 격리 경계다** — 공유 캐시·상태저장소(DP-0005)에서 한 WF의 오염 데이터가 타 WF로 전파되면 안 된다. 영향 채널은 **자원(쿼터)·상태(캐시 오염)·스케줄러** 셋으로 보고 KPI가 빠짐없이 덮는지 검증한다.

> 이 QA는 "한 장애가 **다른 단위로 번지지 않게 막는 격리**(blast-radius 차단)"만 다룬다 — 죽은 단위를 되살려 이어가는 "**복구**(resumability)"는 QA-02(Availability)에서 따로 본다(서로 겹치지 않게 분리). **QA-02 = 장애 단위의 복구 / QA-08 = 타 단위로의 격리.**

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `타 WF 중단 ≤1% AND latency 증가 ≤10%` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **타 Workflow 실행 중단 ≤ 1% AND latency 증가 ≤ 25%** `[주 KPI · PoC 대상]` — 한 WF 폭주(noisy-neighbor) 주입 하에서
  - 쉽게: 한 워크플로우가 망가지거나 폭주해도, 같이 돌던 다른 워크플로우는 100건 중 1건 미만만 중단되고 지연도 25% 안쪽이어야 한다(합격 하한). *latency = 처리 지연 시간.*
  > 보정(2026-06-24, 등급 척도 캘리브레이션 round-03): 타 WF latency 증가 합격 하한 구 `≤10%` → 신 `≤25%`(★☆☆ 합격 하한, 옛 10%는 ★★☆로 상향) — ★1~3 급간 확보 + 무격리 시 p99 doubling(+100%)·CPU 56% degradation 바닥과 충분한 간격. **중단율 ≤1%·쿼터침범 0은 조건/게이트라 불변.** ★ 급간(★★★ ≤5% / ★★☆ ≤10% / ★☆☆ ≤25%)은 ## 등급 척도 참조.
- **자원 격리: 단일 WF의 토큰/rate-limit 소비가 타 WF 쿼터 침범 = 0** — WF별 token-bucket 쿼터 (계정전역 한도의 client-side 분배 — silent cap)
  - 쉽게: 한 워크플로우가 토큰·호출 한도를 아무리 많이 써도, 다른 워크플로우 몫으로 배정된 쿼터를 단 한 번도 침범하면 안 된다. 단 token-bucket이 보장하는 건 **우리 쪽(client-side) 분배**까지다 — 외부 LLM 제공자의 TPM/RPM은 **계정 전역** 한도라(QA-01 rate-limit 헤드룸과 같은 천장) 제공자측 공유 한도 자체는 못 늘린다(silent cap). 즉 내부 공정 분배는 보장하되 전역 풀이 마르면 모든 WF가 함께 느려질 수 있다. *token-bucket = WF별로 사용량을 나눠 담는 통(한도 분배 장치), account-global = LLM 계정 전체에 걸리는 한도.*
- **공유 캐시 오염 전파 = 0** — 한 WF의 오염 데이터가 타 WF로 미전파
  - 쉽게: 공유 캐시에 한 워크플로우가 잘못된 데이터를 넣어도, 그게 다른 워크플로우 결과로 새어 나가면 안 된다(0건). *오염 전파 = 잘못된 캐시 값이 다른 작업으로 퍼지는 것.*

> 위 수치(1%·10%·0건)는 **합리적 합격선 예시**다 — ①②는 기존부터 건강한 측정 가능 KPI라 유지하고, ③④의 임계는 실제 환경에서 측정해 확정한다.

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 멀티테넌시 격리의 업계 표준에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **bulkhead·큐별 동시성·token-bucket** | WF 타입/테넌트별 전용 큐·worker pool + 큐별 max concurrency + 공유 LLM 풀의 WF별 token-bucket rate limiter로 noisy-neighbor 차단 | [KEDA (이벤트 기반 오토스케일·큐별 스케일)](https://keda.sh/) · [Google SRE Workbook — SLO 구현](https://sre.google/workbook/implementing-slos/) |
| **circuit breaker로 장애 WF 격리** | 장애 WF는 차단기로 격리해 재시도 폭주 전파 차단 (SRE 패턴) | [Google SRE Workbook — SLO 구현](https://sre.google/workbook/implementing-slos/) |
| **공유 캐시 무효화·읽기전용 계층화** | 공유 캐시(DP-0005 2안)는 오염 전파 위험 → 무효화·검증·읽기전용 계층으로 격리 경계 유지 | DP-0005 R-1 완화 권고 (round-01 counsel) |

> ⚠️ 레퍼런스의 수치·패턴은 **정당화용**이며 그대로 복제하지 않는다. ③④ 임계는 위 [검증 전략](#검증-전략)의 실측으로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 격리 주입 시뮬레이션**으로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **타 WF 중단 ≤1% AND latency ≤10%** `[주]` | **DP-0004 A5/A8 Bulkhead**(invocation/단계 격리로 Reliability-WF ★★★/★★☆) · WF별 전용 큐·worker pool | **▶ 실제 제작:** noisy-neighbor 격리 주입 — 다수 WF 동시 실행 + WF별 전용 큐·token-bucket rate limiter 모델에서 **한 WF에 폭주(무한루프·대량 토큰) 주입** → 타 WF의 중단율·latency(p95)·쿼터 침범을 rate limiter On/Off로 비교 |
| 자원 격리: 쿼터 침범 = 0 | WF별 token-bucket rate limiter(공유 LLM 풀 분배) — QA-01 헤드룸과 공유 인프라 | 보조 모델: 위 주입 모델에서 폭주 WF의 토큰/rate-limit 소비가 타 WF 쿼터를 침범하는 건수(=0 목표) 집계 |
| 공유 캐시 오염 전파 = 0 | **DP-0005 1안 로컬 캐시**(오염 격리 ★★★) vs **2안 공유 캐시**(전파 위험 R-1) — 2안 채택 시 무효화·읽기전용 계층화 필수 | 보조 모델: 공유 캐시에 오염 데이터 주입 → 타 WF로 전파되는지(무효화 계층 On/Off) 대조 |

> 가정·한계: 폭주 강도·WF 수·token-bucket 파라미터는 **가정 파라미터**다. 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다. **외부 LLM 제공자 측 rate-limit이 계정 전역이면 token-bucket은 client-side 분배만 보장**(제공자측 공유 한도 자체는 못 늘림) — silent cap으로 명시.

## 등급 척도 (★ rubric — ATAM trade-off용)

> 동일 조건 설계 대안의 본 QA 만족도를 ★1~3 비교(별 많은 안 채택). KPI 합격선(하한)=★☆☆ 진입선, ★★☆/★★★는 필드 현실 도달 범위+PoC margin. 하한 미만 불합격. 예시값이며 경계는 PoC로 확정.
>
> 헤드라인 `타 WF 중단 ≤1% AND latency 증가 ≤10%`는 2-index(둘 다 연속·gradable) → main + 조건. PoC 측정 쉬운 `타 WF latency 증가율(p95)`을 main 축에, `중단율`을 표 밖 조건으로 고정. 둘 다 역방향(낮을수록 ★ 높음).

**조건 (2-index — 표 밖):** `타 WF 실행 중단율 ≤ 1% 고정 AND 폭주 WF의 쿼터 침범 = 0건` — noisy-neighbor 폭주(무한 재시도·대량 토큰) 주입 하에서. 중단율을 고정해야 latency 급간이 "같은 격리 강도"를 비교하는 의미를 가짐. 쿼터 침범 0건은 0건 절대형 게이트(규칙5)로 동반.

| 등급 | 구간 — 주 KPI(main 축): 폭주 주입 하 타 WF latency(p95) 증가율 (중단율 ≤1% 고정) | 필드 근거 (경계 이유 + URL) |
|---|---|---|
| ★★★ (상) | latency 증가 ≤ 5% | **(round-04 margin 근거화)** ≤5% = 무간섭 이상 + 격리 시 tail 안정화 관측 대역 하단 + **폭주 강도 가정 흡수 ≈5%**(임의값 아님 — "tail 안정화 + 폭주 강도 흡수"가 근거). 적절한 격리 시 "unrelated workload가 더 이상 간섭 못 해 tail latency 안정화"가 필드 최상위 — 무격리 바닥(+100%)·★☆☆ 25% 대비 충분 간격. [HorizonIQ noisy neighbor](https://www.horizoniq.com/blog/what-are-noisy-neighbors-in-cloud-computing/) · [TiDB noisy-neighbor playbook](https://www.pingcap.com/playbook-noisy-neighbor-multi-tenant-mysql/) |
| ★★☆ (중) | 5% < latency 증가 ≤ 10% | KPI 원본 하한(10%)이 이 대역. bulkhead·큐별 동시성으로 간섭을 한 자릿수%로 묶는 일반 우수 격리 [KEDA](https://keda.sh/) · [Google SRE Workbook](https://sre.google/workbook/implementing-slos/) |
| ★☆☆ (하) | 10% < latency 증가 ≤ 25% — 합격 최소선 (옛 하한 10%를 ★★☆로 올리고 합격 하한 재배치) | 격리는 작동하나 여유가 빠듯한 대역. 무격리 시 p99 doubling(+100%)·CPU 56% degradation이 필드 관측 바닥이므로 그 한참 위인 25%를 합격 하한으로 [noisy-neighbor causal inference](https://arxiv.org/html/2604.03145v1) · [simplyblock p99](https://simplyblock.io/glossary/p99-storage-latency/) |
| 불합격 | latency 증가 > 25% / 또는 중단율 > 1% / 쿼터 침범 > 0건 | 무격리(p99 2배·56% 저하)에 근접 — |

> **캘리브레이션 노트**: KPI 원본 하한 `≤10%`는 필드 기준 건강한 합격선(이 세트에서 드물게 건강 — round-01 verdict)이라 비현실 재배치는 불요. 다만 ★1~3을 띄우려고 하한(10%)을 ★★☆로 올리고 ★☆☆ 합격 하한을 25%로 한 칸 내려 급간 확보(무격리 +100% 바닥과 충분한 간격). 이론 근거 = 무격리 56% degradation/p99 doubling 관측, PoC margin = noisy-neighbor 주입 모델의 폭주 강도·WF 수·token-bucket 파라미터가 가정값이라 ★★★(≤5%)를 무간섭 이상보다 다소 넓게. silent cap: 외부 LLM 제공자 rate-limit은 **계정 전역**이라 token-bucket은 client-side 분배만 보장(전역 풀 마르면 전 WF 동반 저하 — 쿼터 침범 0 게이트는 내부 분배 기준; QA-01 헤드룸과 같은 천장).
> **재캘리브레이션(round-04)**: ★★★ ≤5% margin을 `무간섭 이상 + 격리 시 tail 안정화 관측 대역 하단 + 폭주 강도 흡수 ≈5%`로 **근거화**(C2 — 임의값 탈출). **arXiv 미래형 ID 검증결과(C4)**: 2604.03145는 2026-04 ID로 현재(2026-06) 유효 과거 ID 형식이나 본 검증에서 직접 대조 미완 → "출처 형식 유효, 인용 수치(p99 doubling·CPU 56%) 복제 아님 — **스토리지/CPU noisy-neighbor 사례라 LLM 토큰 쿼터 간섭과 apples 다름**(simplyblock)" silent cap, 인용 정확성은 "확인 불가"로 정직 표기. **쿼터 보조축 검토(C3)**: agentic 진짜 공유 장애 = 토큰 쿼터 독점이나 쿼터침범 0건은 **0건 절대형**이라 gradable proxy 부재 → 별점화 불가, Constraint(게이트) 유지(폭주 강도별 타 WF 쿼터 확보율 같은 gradable proxy를 만들면 별점화 가능하나 본 라운드 범위 밖). 급간 수치는 round-03 유지(본 라운드 근거 보강만 — OI-9 하한 보정 없음).
> **seats**: 발의 Seat 3(격리·멀티테넌시) · consensus (Seat 1이 runaway 폭주 차단=QA-03 cap 연동을 폭주 주입 조건의 전제로 확인 — 2-index를 main=latency/조건=중단율+쿼터로 흡수)

## 변경 이력

### 2026-06-24 — 등급 척도(★ rubric) 캘리브레이션
출처: discussion/qa/round-03 (등급 척도 캘리브레이션 — Council blue team, 팀 검토·승인). main 급간 축 = **폭주 주입 하 타 WF latency(p95) 증가율**(역방향, 낮을수록 ★ 높음).

- **★ 급간**: ★★★ ≤5% / ★★☆ ≤10% / ★☆☆ ≤25%. 하한 미만 불합격.
- **조건/게이트(표 밖)**: `타 WF 중단율 ≤1% 고정 AND 쿼터 침범 = 0건`(0건 절대형 게이트, 규칙5·규칙4 2-index). 중단율을 고정해야 latency 급간이 같은 격리 강도를 비교.
- **구→신(§측정 보정)**: 타 WF latency 증가 합격 하한 `≤10% → ≤25%`(★☆☆). 옛 10%는 ★★☆로 상향 — ★1~3 급간 확보(무격리 p99 doubling·56% degradation 바닥과 간격). 중단율 ≤1%·쿼터침범 0은 조건/게이트라 불변.
- **근거 출처**: 무격리 noisy-neighbor 56% degradation·p99 doubling([causal inference](https://arxiv.org/html/2604.03145v1), [simplyblock p99](https://simplyblock.io/glossary/p99-storage-latency/)) · 격리 시 tail latency 안정화([HorizonIQ](https://www.horizoniq.com/blog/what-are-noisy-neighbors-in-cloud-computing/), [TiDB playbook](https://www.pingcap.com/playbook-noisy-neighbor-multi-tenant-mysql/)) · bulkhead/큐별 동시성([KEDA](https://keda.sh/), [SRE Workbook](https://sre.google/workbook/implementing-slos/)).
- 합격 하한 보정(10%→25%, 10%는 ★★☆)은 `open-issues.md` 등록 대상(KPI 정의 정합 재확인).

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-06-reliability-workflow.md) (red team verdict: **Sound ○ / KPI ○ — Med** — KPI 건강, 자원-쿼터 격리 추가 + QA-02 경계 명문화)

**무엇이 문제였나 (review 지적)**
- 이 세트에서 드물게 **KPI가 건강**(`타 WF 중단 ≤1%`·`latency ≤10%` 구체·측정 가능).
- **QA-02와 경계 중복(C3)**: 둘 다 fault를 다룸 → 명문화 필요(QA-02=복구, QA-08=격리/blast radius).
- **자원-쿼터 격리 누락**: agentic 진짜 공유 장애 도메인은 노드가 아니라 **LLM rate-limit 풀·공유 캐시** — 한 WF 폭주가 전체 토큰 예산을 빨아들임.

**무엇을 바꿨나 (반영)**
- **정의**: "장애·자원 폭주가 실행·지연·토큰/rate-limit 쿼터·공유 상태에 무영향"으로 보강. 영향 채널(자원·상태·스케줄러) 명시. **QA-02(복구)↔QA-08(격리) 경계를 `> altitude`로 양방향 명문화**.
- **KPI 추가**: 기존 ①② 유지(건강) + ③ `자원 격리: 쿼터 침범=0` + ④ `공유 캐시 오염 전파=0`(DP-0005 격리 경계).
- 짝 시나리오 `QAS-08`의 자극(자원 폭주 추가)·Response·Measure를 동기화.

**남은 일 (이 라운드에서 미반영)**
- **DP-0004(bulkhead)·DP-0005(캐시 격리)가 쿼터·상태 격리를 보장하는지 역검토** — 특히 **DP-0005 2안 채택 시 ④(오염 전파 0)가 R-1 위험과 직접 충돌** → 무효화·읽기전용 계층화 명시 필요 (`open-issues.md` 트래킹 대상).
- QA-02 본문의 단방향 cross-link을 이번에 **양방향**으로 맞춤(QA-08 정의에 명시) — QA-02 정의의 altitude 줄과 짝.
- 쿼터 침범 `0`·캐시오염 `0` 임계는 측정으로 확정(①② 1%·10%는 기존 건강 값 유지).

### 2026-06-24 — round-02 디스커션 반영
출처: [`discussion/qa/round-02`](../../discussion/qa/round-02/counsel/QA-06-reliability-workflow.md) (red team verdict: **Sound ○ / KPI ○ — Low** · Med→Low·세트 모범·C3 닫힘 확인)

**무엇이 문제였나 (review 지적)**
- round-01 쿼터 격리·캐시오염 KPI + QA-02 경계 양방향 명문화로 C3 닫힘·세트 모범. 잔여는 비-verdict: DP-0004/0005 격리 역검토(OI-7), 외부 rate-limit 계정전역 silent cap을 KPI 옆에 명문화 누락(Low·QA-01 헤드룸 단위와 연결).

**무엇을 바꿨나 (반영)**
- **Low 보강**: ③ 쿼터 침범 KPI 옆에 **외부 LLM rate-limit이 계정 전역**이라 token-bucket은 client-side 분배만 보장(제공자측 공유 한도는 못 늘림)임을 silent cap으로 명문화 + QA-01 rate-limit 헤드룸과 같은 천장임을 cross-link.

**남은 일 (이 라운드에서 미반영)**
- DP-0004/0005 격리 역검토(OI-7) — 특히 **DP-0005 2안 공유 캐시 채택 시 ④(오염 전파 0)가 R-1 위험과 충돌** → 무효화·읽기전용 계층화 명시 필요. DP 디스커션 위임.

### 2026-06-25 — round-04 디스커션 반영 (★ 등급 척도 근거 보강)
출처: [`discussion/qa/round-04`](../../discussion/qa/round-04/counsel/QA-08-reliability-workflow.md) (red verdict: **Sound ◎ / KPI ○ — Low**; stance: 채택 권장 — ★★★ 5% margin 근거화·arXiv 검증·쿼터 보조축 검토).

**무엇이 문제였나 (review 지적)**
- ★★★ 5% margin 근거 없음(C2). arXiv 2604.03145 미래형 ID 검증 필요(C4). 쿼터(진짜 공유 장애)가 ★에서 빠지고 게이트로만(C3). 인용 p99 doubling·CPU 56%는 스토리지/CPU사례지 LLM 토큰 쿼터 간섭 아님(C1).

**무엇을 바꿨나 (반영 — 근거 보강만, 급간 수치 불변)**
- **★★★ ≤5% margin을 "tail 안정화 + 폭주 강도 흡수 ≈5%"로 근거화**.
- **arXiv 2604.03145 검증결과**: 유효 과거 ID 형식이나 직접 대조 미완 → "확인 불가, 스토리지/CPU noisy-neighbor라 LLM 토큰 쿼터와 apples 다름" silent cap.
- **쿼터 보조축 검토 → Constraint 유지**: 쿼터침범 0건은 0건 절대형이라 별점화 불가(gradable proxy 부재).

**남은 일 (이 라운드에서 미반영)**
- DP-0004/0005 격리 역검토·DP-0005 2안 ④↔R-1 충돌(OI-7).
- 폭주 강도별 타 WF 쿼터 확보율 같은 gradable proxy 별점화는 범위 밖. 경계 예시값.

> 출처: [discussion/qa/round-04](../../discussion/qa/round-04/counsel/QA-08-reliability-workflow.md) (verdict: Sound ◎ / KPI ○ — Low, 채택 권장).
