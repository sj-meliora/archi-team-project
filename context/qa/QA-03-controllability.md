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
---

# QA-03 Controllability — Agent 제어 용이성

## 정의 / Refinement
Agent는 **최소권한·도구 allowlist(허용된 도구만 호출 가능한 목록)** 안에서만 동작하고, **고위험 액션(배포·삭제 등)은 사람 승인(HITL) 게이트**를 거치며, **중단 명령과 runaway cap(폭주 상한)에 즉시 응한다.** "사람 개입 없이"라는 이 과제의 전제를 안전하게 만드는 핵심 안전장치다.

설계할 때 잡아야 할 두 가지 관점:

- **제어의 핵심은 권한 모델이다** — "허용 범위 내에서만 동작"의 *허용 범위*는 곧 권한 모델이다. 모든 도구를 다 줄 게 아니라 **필요한 도구만(allowlist)** 주고, 배포·삭제처럼 되돌리기 어려운 **고위험 액션은 차단을 넘어 사람 승인 게이트(HITL)**를 둔다. 권한 체크는 토큰을 쓰기 전에 **실행 전(admission)** 값싸게 한다.
- **자율 에이전트의 대표 실패는 폭주(runaway)다** — 무한 재시도·무한 루프가 자율 에이전트의 전형적 사고다. **최대 반복 횟수·토큰 예산·총 실행시간(wall-clock)** 상한에 닿으면 자동으로 멈추는 cap이 제어성의 필수 축이다. 또한 "중단을 **접수했다(ack)**"와 "실제로 **안전하게 멈췄다(graceful stop)**"는 다른 사건이다 — 생성 중·배포 중 5초 ack가 곧 안전 정지는 아니므로 둘을 분리해 잰다.

> 이 QA는 "제어성(중단·권한 집행)"만 다룬다 — 비밀관리·공급망 서명·injection 방어·감사를 포괄하는 **보안 전반은 NQA-A(Security)**에서 따로 본다(Controllability ⊂ Security, 제어성은 안전의 수단). 경계는 NQA-A 신설 시 양방향 cross-link.

## 측정 (KPI)
> **주 KPI(헤드라인·PoC 대상)는 `중단 ack ≤5초 AND 안전 정지(graceful stop)+롤백 ≤30초` 1개.** 나머지는 보조(가드레일) — 정의엔 남기되 시연 대상이 아니다.

- **중단 ack ≤ 5초 AND 안전 정지(graceful stop)+롤백 완료 ≤ 30초** `[주 KPI · PoC 대상]` — 협조적 취소(cooperative cancellation) 기준
  - 쉽게: 중단 명령은 5초 안에 "접수했다(ack)"고 응답하되, **하던 일을 안전하게 멈추고 진행분을 되돌리는(롤백) 것까지는 30초** 안에 끝나야 한다. 접수와 실제 정지는 다른 일이다. *ack = 명령을 받았다는 즉답, graceful stop = 강제로 죽이지 않고 체크포인트에서 안전하게 멈춤, 롤백 = 진행 중이던 변경을 되돌림.*
- **허용 범위 외 액션 통과 = 0** — 적대적 eval N건 기준
  - 쉽게: 권한 밖의 행동(허용 안 된 도구 호출·범위 밖 배포)이 단 한 건도 통과하면 안 된다. 이걸 "0건"이라 말하려면 일부러 우회를 유도하는 테스트 묶음(적대적 eval)으로 재야 한다. *적대적 eval = 권한 우회·injection을 유도하는 red-team 시나리오 세트.*
- **Runaway cap 작동 = 100%** — max iteration·token budget·wall-clock 도달 시 자동 중단
  - 쉽게: 에이전트가 무한 루프·재시도에 빠지면, 정해둔 상한(반복 횟수·토큰 예산·총 실행시간)에 닿는 순간 100% 자동으로 멈춰야 한다. *runaway = 스스로 멈추지 못하고 폭주하는 상태, cap = 상한선.*
- **고위험 액션 HITL 게이트 통과율 = 100%, 우회 = 0건** — deploy/delete 등
  - 쉽게: 배포·삭제 같은 위험한 행동은 100% 사람 승인을 거쳐야 하고, 승인 없이 빠져나가는 우회 경로가 0건이어야 한다. *HITL = Human-in-the-loop, 자동 흐름 중간에 사람 승인을 끼우는 것.*

> 위 수치(5초·30초·0건·100%)는 **"측정 가능한 KPI는 이런 모양이다"를 보여주는 예시값**이며, 실제 합격 기준은 실제 환경에서 측정해 확정한다.
> 폐기: 旧 `중단 명령 수용 시간 ≤ 5초` 단독 — "수용"이 ack(접수)인지 정지완료인지 미정의 + 권한집행(위반0)·runaway cap·HITL 축 부재. → ack/정지완료 분리 + 4축으로 확장.

## 근거 / 레퍼런스

왜 KPI를 이렇게 잡았는지 — 각 선택은 자율 에이전트 제어·안전의 표준 패턴에 근거한다 (round-01 counsel에서 확보).

| KPI 선택 | 왜 이렇게 정의했나 | 출처 |
|---|---|---|
| **HITL 게이트 + 최소권한 allowlist** | 자율 에이전트는 도구 allowlist로 권한을 최소화하고 고위험 액션은 사람 승인(HITL)·별도 가드레일을 두는 것이 정석 — "허용 범위"의 정의 자체가 권한 모델이어야 함 | [Anthropic — Building Effective Agents](https://www.anthropic.com/engineering/building-effective-agents) · [Anthropic — 안전·신뢰 에이전트 프레임워크(human control)](https://www.anthropic.com/news/our-framework-for-developing-safe-and-trustworthy-agents) |
| **위반 0건 = 적대적 eval 통과율** | "허용범위 외 0건"은 구호가 아니라 red-team eval 세트 통과율로만 검증 가능 — 측정 수단을 KPI 정의에 묶음 (NQA-A red-team PoC와 공유) | counsel §4 제어·안전 (별도 URL 없음 — eval 세트는 우리가 구축) |
| **협조적 취소 + timeout/retry cap** | cancel token 폴링 + 불응 시 hard-kill 폴백, activity timeout·retry cap으로 runaway를 runner(control plane)가 강제 차단 | [Temporal — 문서(cancellation·timeout/retry policy)](https://docs.temporal.io/temporal) |

> ⚠️ 레퍼런스의 수치·임계(5초·차단율 등)는 **패턴 정당화용**이며 그대로 복제하지 않는다. 우리 합격선은 위 [검증 전략](#검증-전략)의 실측·모델로 확정한다.

## 검증 전략

각 KPI를 **실제로 달성하는 건 특정 설계 결정(DP)** 이다. 그 설계가 KPI를 만족하는지는 **간단한 시뮬레이션/모델**로 (실제 시스템 없이) 보일 수 있다 — 설계 주장(별점)을 근거 있는 그래프로 바꾸는 것이 목표.

| KPI | 책임지는 설계 (DP 주장) | 검증 실험·모델 |
|---|---|---|
| **중단 ack ≤5초 AND 안전 정지(graceful stop)+롤백 ≤30초** `[주]` | **DP-0002 1안 단일 제어지점**(중앙 정책·HITL로 Controllability ★★★) · **SP-2** 정책 적용 지점의 집중도가 중단 latency를 좌우 + runner의 협조적 취소(cancel token 폴링→불응 시 hard-kill) | **▶ 실제 제작:** 협조적 취소 타이밍 모델 — 장기 activity에 cancel 폴링 지점을 **밀도별로** 삽입한 mock 파이프라인에서 cancel 발행 → **ack 시간·정지완료 시간**을 폴링 밀도별 분포로 산출, 불응 시 hard-kill로 상한 보장 확인 |
| 허용범위 외 통과 = 0 | **DP-0003 1안 사전 권한 게이트**(allowlist·실행 전 admission, Controllability ★★★) | 보조 모델(발표 서사): 적대적 eval 세트(권한 외 도구 호출·injection)로 통과/차단 라벨 — 실제 세트 구축은 NQA-A와 공유, 본 라운드 미실행 |
| Runaway cap 작동 = 100% | **DP-0002**(중앙 오케스트레이터가 cap 강제) + runner activity timeout/retry cap — 단, **DP-0002/0003이 runaway cap을 명시 안 함**(아래 "남은 일") | 보조 모델: cap 시나리오(무한루프 유도) 주입 → max iter/token/wall-clock 도달 시 자동 중단율 집계 |
| 고위험 HITL 통과율 100%·우회 0 | **DP-0002 1안 HITL gate** + **DP-0003 1안 권한 게이트**(허용 액션만 통과) | 보조 모델: 고위험 액션 경로에 HITL 게이트 삽입 → 승인 없는 우회 경로 탐색(=0 목표) |

> 가정·한계: 폴링 지점 밀도·취소 전파 지연·activity 길이는 **가정 파라미터**다. 이 실험이 증명하는 것은 "이 설계가 *이런 메커니즘으로* KPI를 달성하고, KPI가 *이 방법으로 측정 가능*하다"이지 가상 시스템의 실측치가 아니다 — 슬라이드엔 가정값을 명시한다. deploy 중 취소의 **부분 롤백 정합성**(외부 시스템 상태)과 적대적 세트의 **커버리지 상한**(미상상 공격 미검출)은 통합 환경·실제 eval 구축이 필요해 미검증(silent cap).

## 변경 이력

### 2026-06-24 — round-01 디스커션 반영
출처: [`discussion/qa/round-01`](../../discussion/qa/round-01/counsel/QA-03-controllability.md) (red team verdict: **Sound ◎ / KPI △ — Med** — KPI 통일·확장 + stop 의미 분리 필요)

**무엇이 문제였나 (review 지적)**
- **QA↔QAS 불일치(C2)**: QAS-03엔 "허용범위 외 액션 통과 0건"이 있으나 QA-03 파일 KPI엔 "≤5초"만 → SSoT 분기.
- **누락 3축**: 최소권한·도구 allowlist·고위험 HITL 게이트 / **runaway cap**(max iter·token·wall-clock) / "0건"의 **검증수단(적대적 eval)**.
- **stop 의미 모호**: "수용(ack) ≤5초" ≠ "실제 정지" — 안전 정지(graceful stop, 롤백) vs 강제 종료(hard-kill) 미구분.

**무엇을 바꿨나 (반영)**
- **정의**: 최소권한·도구 allowlist + 고위험 액션 HITL 승인 + 중단·runaway cap 즉응으로 보강. NQA-A(Security)와 경계를 `> altitude` 한 줄로 분리(Controllability ⊂ Security).
- **KPI 분해·확장**: 旧 `수용 ≤5초` → ① `ack ≤5초 AND 안전 정지(graceful stop)+롤백 ≤30초`(ack/정지완료 분리) ② `허용범위 외 통과=0`(QAS에만 있던 것 QA 파일에 정식 편입 = C2 해소) ③ `runaway cap 100%` ④ `HITL 통과율 100%·우회 0`.
- 짝 시나리오 `QAS-03`의 자극(runaway·고위험 액션 추가)·Response·Measure를 동일하게 동기화.

**남은 일 (이 라운드에서 미반영)**
- **검증수단(적대적 eval로 위반0 증명)은 [발표 서사]로 분기** — 슬라이드 한 줄("권한 외 액션·injection을 red-team eval로 통과 0건 검증")로만, 본 라운드 실측 미실행. PoC-C2는 NQA-A(Security) PoC와 공유.
- **DP-0002/0003이 runaway cap·안전 정지(graceful stop, 롤백)를 명시 안 함** → tactic 보강 역검토 필요 (`open-issues.md` 트래킹 대상).
- NQA-A(Security) 신설 시 Controllability ⊂ Security 경계 cross-link을 **양방향**으로 박는다(현재 QA-03 정의에 단방향 명시).
- 안전 정지(graceful stop)+롤백 `30초`·HITL/위반 임계는 **예시값**이며 실환경 측정으로 확정.
</content>
</invoke>
