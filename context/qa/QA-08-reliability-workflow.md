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

- **타 Workflow 실행 중단 ≤ 1% AND latency 증가 ≤ 10%** `[주 KPI · PoC 대상]` — 한 WF 폭주(noisy-neighbor) 주입 하에서
  - 쉽게: 한 워크플로우가 망가지거나 폭주해도, 같이 돌던 다른 워크플로우는 100건 중 1건 미만만 중단되고 지연도 10% 안쪽이어야 한다. *latency = 처리 지연 시간.*
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

## 변경 이력

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
