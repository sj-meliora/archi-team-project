---
id: QA-03
category: QA
importance: H
difficulty: M
source: pptx p.13
related-dp: [DP-0002, DP-0003]
updates:
  - date: 2026-06-24
    by: discussion/qa/round-01
    reason: "수용 단일지표 → ack/정지완료 분리 + 위반0·runaway cap·HITL 4축 재설계 (자세히 → ## 변경 이력)"
  - date: 2026-06-24
    by: discussion/qa/round-02
    reason: "② 위반0 → ②-1 권한외 차단율(룰 게이트·현 조건 닫힘) + ②-2 적대적 위반0(QA-06 채택 OI-8 의존 open-issue) 2단 분리 + '동반 닫힘' 조건부 정정 (contention 반영, 자세히 → ## 변경 이력)"
  - date: 2026-06-24
    by: discussion/qa/round-03 (등급 척도 캘리브레이션)
    reason: "★ rubric 추가 — main 축=graceful stop+롤백 시간(역방향), ack≤5초·HITL100·runaway100·②-1은 게이트. KPI 합격선 ≤30초 → ★★☆ 흡수·★☆☆=>30초+hard-kill 폴백 재배치(규칙2) (자세히 → ## 변경 이력)"
---

# QA-03 Controllability — Agent 제어 용이성

## 정의 / Refinement
Agent는 **최소권한·도구 allowlist(허용된 도구만 호출 가능한 목록)** 안에서만 동작하고, **고위험 액션(배포·삭제 등)은 사람 승인(HITL) 게이트**를 거치며, **중단 명령과 runaway cap(폭주 상한)에 즉시 응한다.** "사람 개입 없이"라는 이 과제의 전제를 안전하게 만드는 핵심 안전장치다.

설계할 때 잡아야 할 두 가지 관점:

- **제어의 핵심은 권한 모델이다** — "허용 범위 내에서만 동작"의 *허용 범위*는 곧 권한 모델이다. 모든 도구를 다 줄 게 아니라 **필요한 도구만(allowlist)** 주고, 배포·삭제처럼 되돌리기 어려운 **고위험 액션은 차단을 넘어 사람 승인 게이트(HITL)**를 둔다. 권한 체크는 토큰을 쓰기 전에 **실행 전(admission)** 값싸게 한다.
- **자율 에이전트의 대표 실패는 폭주(runaway)다** — 무한 재시도·무한 루프가 자율 에이전트의 전형적 사고다. **최대 반복 횟수·토큰 예산·총 실행시간(wall-clock)** 상한에 닿으면 자동으로 멈추는 cap이 제어성의 필수 축이다. 또한 "중단을 **접수했다(ack)**"와 "실제로 **안전하게 멈췄다(graceful stop)**"는 다른 사건이다 — 생성 중·배포 중 5초 ack가 곧 안전 정지는 아니므로 둘을 분리해 잰다.

> 이 QA는 "제어성(중단·권한 집행)"만 다룬다 — 비밀관리·공급망 서명·injection 방어·감사를 포괄하는 **보안 전반은 QA-06(Security)**에서 따로 본다(Controllability ⊂ Security, 제어성은 안전의 수단). 경계는 QA-06 신설 시 양방향 cross-link.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `중단 ack ≤5초 AND 안전 정지(graceful stop)+롤백 ≤30초` 1개로 유지(교체 불요 — 이미 시연 가능).** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.
> 비대칭 사유(QA-11 대비): "절대값 KPI는 대리/대조로 시연한다" 원칙은 QA-11와 동일하나, **QA-11는 ②(`pass^k`)가 헤드라인이라 헤드라인을 교체**했고 **QA-03은 ②가 처음부터 보조라 헤드라인 ①은 유지하고 ②의 표현만 통일**한다(KPI 지위 차이로 적용 지점만 갈림). 따라서 ②는 보조 KPI로서 ②-1(현 조건 닫힘)·②-2(open-issue)로 2단 분리한다.

- **중단 ack ≤ 5초 AND 안전 정지(graceful stop)+롤백 완료 ≤ 30초** `[주 KPI · PoC 대상]` — 협조적 취소(cooperative cancellation) 기준
  - 쉽게: 중단 명령은 5초 안에 "접수했다(ack)"고 응답하되, **하던 일을 안전하게 멈추고 진행분을 되돌리는(롤백) 것까지는 30초** 안에 끝나야 한다. 접수와 실제 정지는 다른 일이다. *ack = 명령을 받았다는 즉답, graceful stop = 강제로 죽이지 않고 체크포인트에서 안전하게 멈춤, 롤백 = 진행 중이던 변경을 되돌림.*
  - > 보정(2026-06-24, 등급 척도 캘리브레이션 round-03): graceful stop+롤백 합격선 구 `≤30초 = ★☆☆ 진입선` → 신 `≤30초를 ★★☆ 상한으로 흡수, ★☆☆ = 30초 초과 + hard-kill 폴백으로 유한시간 정지 보장`. 사유: 30초(k8s grace 기본값)를 ★☆☆ 게이트로 두면 ★★☆가 사문화 → 규칙2 재배치. ack ≤5초·HITL100·runaway100·②-1은 게이트라 불변. ★ 급간은 ## 등급 척도 참조.
- **②-1 권한외 액션 차단율 — 주입한 hard-invalid 시도 → 실행 전 차단/안전정지 latency 분포** (룰 체커·admission 게이트, QA-06/하네스 풀세트 불요)
  - 쉽게: 명백히 권한 밖인 행동(허용 안 된 도구 호출·범위 밖 배포·스키마 위반 = **hard-invalid**, 룰로 즉시 판정 가능)을 일부러 주입했을 때, 실행 전(admission) 게이트가 이를 차단하는 비율과 **차단/안전정지까지 걸린 시간 분포**를 잰다. "위반이 없음"(시연 불가)이 아니라 "주입한 위반을 막는 메커니즘이 작동함"(시연 가능)을 보이는 게 핵심 — ①의 `취소→ack/정지 분포`와 같은 "주입→차단/정지 분포" 아키타입으로 통일한다. *hard-invalid = allowlist·스키마·범위로 룰 판정되는 명백한 위반, admission = 토큰 쓰기 전 실행 전 차단.* **QA-06 채택(OI-8)과 무관하게 현 조건에서 산출되는 1차 게이트.**
- **②-2 적대적 위반 통과 = 0 (acceptance)** — 적대적 eval 풀세트(OWASP LLM Top-10 매핑) 기준, 세트 커버리지가 신뢰 상한 `(open-issue — QA-06 채택 OI-8 의존)`
  - 쉽게: injection·권한상승처럼 **룰로 미리 표현 안 되는 미상상·우회 공격**까지 포함한 적대적 풀세트에서 통과가 단 한 건도 없어야 한다. 이 "0건"은 *세트가 상상한 공격에 대해서만* 0건이라 커버리지(OWASP LLM Top-10 매핑)를 함께 로그해 과대주장을 막는다. 이 풀커버리지 실측은 **공유 red-team 하네스(QA-06)** 구축에 묶이므로, **QA-06 정식 채택(OI-8·사람 결정) 시 닫히는 추적되는 open-issue**다 — 무조건 "동반 닫힘"이 아니라 *채택되면* 닫힘. *적대적 eval = 권한 우회·injection을 유도하는 red-team 시나리오 세트.*
- **Runaway cap 작동 = 100%** — max iteration·token budget·wall-clock 도달 시 자동 중단
  - 쉽게: 에이전트가 무한 루프·재시도에 빠지면, 정해둔 상한(반복 횟수·토큰 예산·총 실행시간)에 닿는 순간 100% 자동으로 멈춰야 한다. *runaway = 스스로 멈추지 못하고 폭주하는 상태, cap = 상한선.*
- **고위험 액션 HITL 게이트 통과율 = 100%, 우회 = 0건** — deploy/delete 등
  - 쉽게: 배포·삭제 같은 위험한 행동은 100% 사람 승인을 거쳐야 하고, 승인 없이 빠져나가는 우회 경로가 0건이어야 한다. *HITL = Human-in-the-loop, 자동 흐름 중간에 사람 승인을 끼우는 것.*

> 위 수치(5초·30초·차단율·0건·100%)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 실제 환경에서 측정해 확정한다.
> 폐기: 旧 `허용 범위 외 액션 통과 = 0` 단독 — "없음"을 시연할 수 없는 절대값 + 풀세트 구축 전엔 산출 불가. → ②-1 차단율(현 조건 닫힘)/②-2 acceptance(QA-06 채택 의존 open-issue)로 2단 분리(round-02 contention).
> 폐기: 旧 `중단 명령 수용 시간 ≤ 5초` 단독 — "수용"이 ack(접수)인지 정지완료인지 미정의 + 권한집행(위반0)·runaway cap·HITL 축 부재. → ack/정지완료 분리 + 4축으로 확장.

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 자율 에이전트 제어·안전의 표준 패턴에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **HITL 게이트 + 최소권한 allowlist** | 자율 에이전트는 도구 allowlist로 권한을 최소화하고 고위험 액션은 사람 승인(HITL)·별도 가드레일을 두는 것이 정석 — "허용 범위"의 정의 자체가 권한 모델이어야 함 | [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) · [Anthropic — 안전·신뢰 에이전트 프레임워크(human control)](https://www.anthropic.com/news/our-framework-for-developing-safe-and-trustworthy-agents) |
| **위반 0건 = 적대적 eval 통과율** | "허용범위 외 0건"은 구호가 아니라 red-team eval 세트 통과율로만 검증 가능 — 측정 수단을 KPI 정의에 묶음 (QA-06 red-team PoC와 공유) | counsel §4 제어·안전 (별도 URL 없음 — eval 세트는 우리가 구축) |
| **협조적 취소 + timeout/retry cap** | cancel token 폴링 + 불응 시 hard-kill 폴백, activity timeout·retry cap으로 runaway를 runner(control plane)가 강제 차단 | [Temporal — 문서(cancellation·timeout/retry policy)](https://docs.temporal.io/temporal) |

> ⚠️ 레퍼런스의 수치·임계(5초·차단율 등)는 **패턴 정당화용**이며 그대로 복제하지 않는다. 우리 합격선은 위 [검증 전략](#검증-전략)의 실측·모델로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 시뮬레이션/모델**로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **중단 ack ≤5초 AND 안전 정지(graceful stop)+롤백 ≤30초** `[주]` | **DP-0002 1안 단일 제어지점**(중앙 정책·HITL로 Controllability ★★★) · **SP-2** 정책 적용 지점의 집중도가 중단 latency를 좌우 + runner의 협조적 취소(cancel token 폴링→불응 시 hard-kill) | **▶ 실제 제작:** 협조적 취소 타이밍 모델 — 장기 activity에 cancel 폴링 지점을 **밀도별로** 삽입한 mock 파이프라인에서 cancel 발행 → **ack 시간·정지완료 시간**을 폴링 밀도별 분포로 산출, 불응 시 hard-kill로 상한 보장 확인 |
| **②-1 권한외 차단율 + 차단/안전정지 latency 분포** | **DP-0003 1안 사전 권한 게이트**(allowlist·실행 전 admission, Controllability ★★★) — hard-invalid(allowlist/스키마/범위 위반)를 실행 전 차단 | 보조 모델: 주입한 hard-invalid 시도 N건(룰 판정 가능) → admission 게이트 통과/차단 라벨 + 차단/안전정지 latency 분포. **golden·적대적 풀세트 불요 — 룰 체커로 현 조건 산출**(①의 cancel→ack/정지 분포 모델과 합치거나 별도 마이크로-PoC, PoC 밀도는 재량). **OI-8 무관 선(先)닫힘.** |
| ②-2 적대적 위반0 (acceptance) | **DP-0003 1안** + **eval/검증 서브시스템(QA-06 공유 red-team 하네스, OI-7)** | 보조 모델(발표 서사): 적대적 eval 풀세트(OWASP LLM Top-10 매핑·injection·권한상승)로 통과/차단 — **실제 세트 구축은 QA-06와 공유**, 본 라운드 미실행. **②-2·QA-06 주 KPI는 QA-06 채택(OI-8) 시 동일 하네스로 동반 닫힘**(무조건 동반 닫힘 아님 — 채택 의존 open-issue). 세트 커버리지 = 위반0의 신뢰 상한(미상상 우회 미검출, OWASP 매핑 log). |
| Runaway cap 작동 = 100% | **DP-0002**(중앙 오케스트레이터가 cap 강제) + runner activity timeout/retry cap — 단, **DP-0002/0003이 runaway cap을 명시 안 함**(아래 "남은 일") | 보조 모델: cap 시나리오(무한루프 유도) 주입 → max iter/token/wall-clock 도달 시 자동 중단율 집계 |
| 고위험 HITL 통과율 100%·우회 0 | **DP-0002 1안 HITL gate** + **DP-0003 1안 권한 게이트**(허용 액션만 통과) | 보조 모델: 고위험 액션 경로에 HITL 게이트 삽입 → 승인 없는 우회 경로 탐색(=0 목표) |

> 가정·한계: 폴링 지점 밀도·취소 전파 지연·activity 길이는 **가정 파라미터**다. 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다 — 슬라이드엔 가정값을 명시한다. deploy 중 취소의 **부분 롤백 정합성**(외부 시스템 상태)과 적대적 세트의 **커버리지 상한**(미상상 공격 미검출)은 통합 환경·실제 eval 구축이 필요해 미검증(silent cap).

## 등급 척도 (★ rubric — ATAM trade-off용)

> 동일 조건 설계 대안의 본 QA 만족도를 ★1~3 비교(별 많은 안 채택). KPI 합격선(하한)=★☆☆ 진입선, ★★☆/★★★는 필드 현실 도달 범위+PoC margin. 하한 미만 불합격. 예시값이며 경계는 PoC로 확정.

> 헤드라인 `중단 ack ≤5초 AND graceful stop+롤백 ≤30초`는 **2-index(둘 다 시간·gradable)** → 규칙4. main 축 = **graceful stop+롤백 완료 시간(가장 무겁고 PoC 측정 가능)**, ack는 표 밖 `조건:`으로 고정. 시간 지표는 **역방향(낮을수록 ★ 높음)**. ②-1 권한외 차단율·HITL 100%·runaway cap 100%는 **0건/100% 절대형** → 규칙5상 별점 축이 아니라 **게이트(pass/fail)**.

**조건 (2-index, 규칙4):** `조건: 중단 ack ≤ 5초 고정` — ack(접수 즉답)는 cancel-token 폴링/heartbeat 한 사이클이면 충분히 달성되는 1차 응답이라 변별력이 낮다. 따라서 ack는 진입 게이트로 고정하고, 설계 대안의 변별은 **무거운 쪽(graceful stop + 롤백 완료)** 으로 잰다. 아래 게이트도 함께 통과해야 ★ 부여.

| 등급 | 구간 — 주 KPI(main 축): graceful stop + 롤백 완료 시간 (역방향, 낮을수록 상) | 필드 근거 (경계 이유 + URL) |
|---|---|---|
| ★★★ (상) | **≤ 15초** | k8s `terminationGracePeriodSeconds` 기본 30초 안에서 LB 드레인(5초)·in-flight 마무리를 빼면 실제 cleanup window는 ~15-25초 — 15초 내 안전 정지+롤백이면 우수 상단. Temporal heartbeat 전파도 aggressive 설정(5초 hb)에서 cancel이 15-20초에 도착하므로 그 하단인 ≤15초는 폴링 밀도를 촘촘히 둔 잘 설계된 control-plane의 도달선. PoC margin 고려해 이론 천장(즉시 전파)에 붙이지 않음. [k8s graceful](https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-terminating-with-grace) · [Temporal cancel 전파](https://medium.com/@asakisakamoto02/how-to-cancel-on-heartbeat-timeout-in-temporal-framework-11afb4f9c708) |
| ★★☆ (중) | **15초 초과 ~ 30초 이하** | 30초 = k8s 기본 grace period(SIGTERM→SIGKILL) 표준 상한. heartbeat 전파 지연(15-20초)+롤백 여유를 합치면 일반 우수 설계의 현실 대역. [k8s 30초 기본·SIGTERM 윈도](https://web-alert.io/blog/graceful-shutdown-sigterm-zero-downtime-deploys-guide) |
| ★☆☆ (하) | **30초 초과 (= 보정된 하한선·합격 최소). 단 hard-kill 폴백으로 상한 보장 필수** | KPI 정의 하한 `≤30초`를 ★☆☆ 진입선으로 두면 30초가 게이트가 되어 ★★☆가 죽으므로, 규칙2로 30초를 ★★☆/★★★ 변별 안에 흡수하고 ★☆☆는 "30초 초과지만 cancel→ack→hard-kill 폴백으로 유한 시간 내 정지 보장됨"으로 재배치. heartbeat 미설정 activity는 cancel을 못 받으므로(전파 0%) hard-kill 폴백 존재가 합격 최소 조건. [Temporal — hb 없으면 cancel 수신 불가](https://docs.temporal.io/activity-execution) |
| 불합격 | ack > 5초(접수 자체 지연) / hard-kill 폴백 부재로 정지 시간 무한 / 아래 게이트 위반 | — |

**게이트 (별점과 AND, 규칙5 — 0건/100% 절대형):**
- **HITL 통과율 = 100% · 우회 = 0건** (deploy/delete 등 고위험 액션) — pass/fail.
- **runaway cap 작동 = 100%** (max iter·token·wall-clock) — pass/fail.
- **②-1 권한외 hard-invalid 차단율** — admission 게이트가 룰 판정 가능 위반을 차단. 별점 축 아닌 게이트(차단/안전정지 latency 분포는 ① PoC에 합산해 보조 그래프로만; gradable proxy이나 헤드라인 변별엔 미사용).

> **캘리브레이션 노트**: 하한 `30초`는 k8s grace 기본값과 정합하나 그대로 ★☆☆ 게이트로 쓰면 ★★☆가 사실상 사문화 → 규칙2 재배치(30초를 ★★☆ 상한으로, ★★★는 ≤15초). `이론 근거 = 즉시 전파(heartbeat 폴링 밀도 무한)` / `PoC margin = heartbeat 전파 지연 15-20초 실측 관례를 흡수해 ★★★를 ≤15초로(0초에 붙이지 않음)`. silent cap: **deploy 중 부분 롤백의 외부 시스템 정합성**(외부 상태 되돌림)은 통합 환경 필요 → 미검증. 헤드라인 0건/100% 게이트의 신뢰 상한은 ②-2 적대적 풀세트 커버리지(OWASP LLM Top-10 매핑, OI-8 의존)이며 미상상 우회는 미검출.
> **seats**: 발의 Seat 3(Workflow Runner 인프라 — durable cancel 전파) · consensus (Seat 1: ack 고정·게이트 분리 동의 / Seat 2: 30초 재배치로 ★★☆ 부활 동의)

## 변경 이력

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-03-controllability.md) (red team verdict: **Sound ◎ / KPI △ — Med** — KPI 통일·확장 + stop 의미 분리 필요)

**무엇이 문제였나 (review 지적)**
- **QA↔QAS 불일치(C2)**: QAS-03엔 "허용범위 외 액션 통과 0건"이 있으나 QA-03 파일 KPI엔 "≤5초"만 → SSoT 분기.
- **누락 3축**: 최소권한·도구 allowlist·고위험 HITL 게이트 / **runaway cap**(max iter·token·wall-clock) / "0건"의 **검증수단(적대적 eval)**.
- **stop 의미 모호**: "수용(ack) ≤5초" ≠ "실제 정지" — 안전 정지(graceful stop, 롤백) vs 강제 종료(hard-kill) 미구분.

**무엇을 바꿨나 (반영)**
- **정의**: 최소권한·도구 allowlist + 고위험 액션 HITL 승인 + 중단·runaway cap 즉응으로 보강. QA-06(Security)와 경계를 `> altitude` 한 줄로 분리(Controllability ⊂ Security).
- **KPI 분해·확장**: 旧 `수용 ≤5초` → ① `ack ≤5초 AND 안전 정지(graceful stop)+롤백 ≤30초`(ack/정지완료 분리) ② `허용범위 외 통과=0`(QAS에만 있던 것 QA 파일에 정식 편입 = C2 해소) ③ `runaway cap 100%` ④ `HITL 통과율 100%·우회 0`.
- 짝 시나리오 `QAS-03`의 자극(runaway·고위험 액션 추가)·Response·Measure를 동일하게 동기화.

**남은 일 (이 라운드에서 미반영)**
- **검증수단(적대적 eval로 위반0 증명)은 [발표 서사]로 분기** — 슬라이드 한 줄("권한 외 액션·injection을 red-team eval로 통과 0건 검증")로만, 본 라운드 실측 미실행. PoC-C2는 QA-06(Security) PoC와 공유.
- **DP-0002/0003이 runaway cap·안전 정지(graceful stop, 롤백)를 명시 안 함** → tactic 보강 역검토 필요 (`open-issues.md` 트래킹 대상).
- QA-06(Security) 신설 시 Controllability ⊂ Security 경계 cross-link을 **양방향**으로 박는다(현재 QA-03 정의에 단방향 명시).
- 안전 정지(graceful stop)+롤백 `30초`·HITL/위반 임계는 **예시값**이며 실환경 측정으로 확정.

### 2026-06-24 — round-02 디스커션 반영 (contention)
출처: [`discussion/qa/round-02`](../../discussion/qa/round-02/contention/counter.md) (red team verdict: **Sound ◎ / KPI △ — Med** · contention R1[수용]·R2[부분수용])

**무엇이 문제였나 (review·contention 지적)**
- **R1 — "동반 닫힘"의 조건부성 은폐**: ② `위반0`의 실측 전환이 QA-06 정식 채택(OI-8·사람 결정)에 통째 의존하는데, counsel이 이를 "동반 닫힘"이라는 무조건처럼 읽히는 표현으로 낙관(round-02 counsel line 69).
- **R2 — 절대값 KPI 시연 불가 + QA-11 무브 미적용 비대칭**: `허용범위 외 통과=0`은 QA-11 `pass^k 절대값`과 동형의 "없음을 시연할 수 없는" 절대값인데, 같은 ASR 세트에서 QA-11엔 "절대값→대리/대조" 무브를 쓰고 QA-03 ②엔 안 쓴 비대칭.

**무엇을 바꿨나 (반영)**
- **② 2단 분리**(QA-11 ②-1/②-2 아키타입 이식): ② `허용범위 외 통과=0` →
  - **②-1 권한외 액션 차단율** = 주입한 hard-invalid(allowlist 외 도구 호출·범위 밖 배포·스키마 위반)를 DP-0003 1안 admission 게이트가 차단하는 비율. 측정은 **"주입 위반 → 차단/안전정지 latency 분포"**로(①의 cancel→ack/정지 분포와 동일 아키타입 통일). **룰 체커로 산출 — golden/하네스 풀세트 불요 → OI-8 무관 현 조건 닫힘 1차 게이트.**
  - **②-2 적대적 위반0 acceptance** = OWASP LLM Top-10 매핑 적대적 풀세트 통과=0. **QA-06 공유 하네스 풀커버리지 의존 → OI-8 채택 시 닫히는 추적 open-issue로 명시 강등**(커버리지 silent cap·OWASP 매핑 log 유지).
- **"동반 닫힘" 조건부 정정**: 무조건 "동반 닫힘" → "②-2·QA-06는 OI-8 채택 시 동일 하네스로 동반 닫힘; ②-1은 OI-8 무관 선닫힘"으로 검증 전략·KPI에 조건 명시.
- **헤드라인 불변 명시**: 주 KPI(헤드라인·PoC 대상)는 ① 유지(시연 가능 — 교체 불요). QA-11와의 비대칭 사유(② 지위=헤드라인 vs 보조)를 KPI 섹션에 1줄 명시.
- 짝 시나리오 `QAS-03`의 Response·Measure를 ②-1/②-2 2단으로 동기화.

**남은 일 (이 라운드에서 미반영)**
- **②-2 ↔ QA-06 하네스 채택(OI-8)**: 적대적 풀커버리지 위반0 실측은 QA-06 정식 채택(사람 결정) 동반 — `open-issues.md` OI-8에 교차 의존 등록(QA-11 ②-2 ↔ QA-07와 동일 구조).
- **DP 귀속(OI-7)**: runaway cap·graceful stop+롤백 tactic이 DP-0002/0003에 미명시 + eval/검증 서브시스템 DP 신설(②-2 하네스) → DP 디스커션 위임.
- ②-1 latency 분포를 ① PoC에 합칠지 별도 마이크로-PoC로 둘지는 PoC 밀도 재량(측정가능성·정합성 무관).

### 2026-06-24 — 등급 척도(★ rubric) 캘리브레이션
출처: discussion/qa/round-03 (등급 척도 캘리브레이션 — Council 3 seats, 팀 승인). 용도 = ATAM trade-off에서 동일 조건 설계 대안 비교(★ 많은 안 채택).

**무엇을 했나**
- **main 급간 축**: 헤드라인 2-index(ack·graceful stop+롤백) 중 **graceful stop+롤백 완료 시간**을 main 축으로(규칙4) — 시간 **역방향**(낮을수록 ★ 높음). ack ≤5초는 표 밖 `조건:`으로 고정(변별력 낮은 1차 응답).
- **★ 급간**: ★★★ ≤15초 / ★★☆ 15초 초과~30초 / ★☆☆ 30초 초과 + hard-kill 폴백. 근거 = k8s grace 기본 30초·LB 드레인·Temporal heartbeat cancel 전파 15-20초.
- **게이트(규칙5, 0건/100% 절대형)**: HITL 통과율 100%·우회 0건 / runaway cap 100% / ②-1 권한외 hard-invalid 차단(admission). 별점 축 아님 — pass/fail.
- **§측정 보정(구→신)**: graceful stop+롤백 합격선 구 `≤30초 = ★☆☆ 진입선` → 신 `≤30초를 ★★☆ 상한으로 흡수, ★☆☆ = 30초 초과 + hard-kill 폴백`(규칙2 재배치 — 30초를 ★☆☆ 게이트로 두면 ★★☆ 사문화). 보정 트레이스를 §측정 해당 KPI 줄 아래 보존.
- silent cap: deploy 중 부분 롤백의 외부 시스템 정합성 미검증 / 0건·100% 게이트 신뢰 상한은 ②-2 풀세트 커버리지(OI-8 의존).
