# Review: DP-02 Agent Hierarchy

> source: context/dp/DP-0002-agent-hierarchy.md (+ _backlog.md BL-1·BL-2)
> verdict: **ASR ○ / KPI △** — 핵심 ASR 3개(QA-01/02/03)를 cover하고 TP-1(Controllability↔Availability)이 실재 교환점 · 단 ★칸이 QA 실제 KPI에 미앵커·3안 별점이 미검증 추정 · severity **Med** (제어평면이 DP-04와 동시 결정 미명시 = High 후보, report C*)
> lenses: (1) Agentic Workflow 전문가 · (2) ATAM 평가 수석 아키텍트 · (3) Runner 인프라 아키텍트

## 원문 요약
- **결정 포인트**: Agent들을 어떤 위상(topology)으로 — 중앙 제어성 vs 가용성 교환.
- **후보 대안**: 1안 Hierarchical(Orchestrator 중심) · 2안 Fully Decentralized(P2P/Choreography) · 3안 Hierarchical + Standby Orchestrator(Active-Passive 이중화, 상태 외부화 — BL-1).
- **driving ASR QA**: 헤더 `drives:` = QA-03(Controllability)↑ · QA-02(Availability) · QA-10(Performance-E2E) · QA-01(Scalability). **ASR(QA-01~07) = QA-01·02·03 셋 + 비-ASR QA-10.** → DP-04와 달리 ASR 다수 cover.
- **★매트릭스**: 1안/2안/3안 × Controllability/Scalability/Availability/Perf. 3안은 "(추정)" 명기.
- **ATAM**: SP-1(Orchestrator 가용성→QA-02)·SP-2(정책 집중도→QA-03) / TP-1(Controllability↔Availability, 핵심)·TP-2(Perf↔Controllability) / R-1(1안 SPOF→QA-02)·R-2(2안 분산정책→QA-03+토큰QA-05) / NR-1(Scalability 1·2안 동급).
- **결정/근거**: Controllability H + C-0002 강제 → 1안 기반, R-1은 3안(Standby)으로 완화 검토.

## ASR cover·KPI 비교 (1순위 판정)
- **세트 관점**: DP-02는 **ASR을 3개(QA-01·02·03) cover** — 세트에서 ASR coverage가 가장 두터운 DP. QA-02·QA-03을 TP-1로 **직접 교환**(둘 다 H 중요도)하므로, 이 DP는 ASR 핵심 교환의 중심축이다. QA-01은 NR-1로 "안전 충족(N)" 처리 — 변별 없음(1·2안 동급 ★★☆).
- **개별 cover 품질 (QA-03 Controllability)**: SP-2가 "정책 적용 지점 집중도 → QA-03(중단 ≤5초)"로 cover. 단 ⚠️ QA-03의 **현 주 KPI는 `중단 ack ≤5초 AND graceful stop+롤백 ≤30초`**(round-04 ★ rubric: main축=graceful stop 시간)인데, ATAM은 옛 "중단 ≤5초"만 인용 — **graceful stop·롤백·runaway cap 차원 미명시**(OI-7: "DP-0002 제어지점 runaway cap·graceful 정지 tactic 미명시"가 이미 트래킹). 1안 HITL gate는 있으나 cap/정지 분리는 후보 대안·ATAM에 없음.
- **개별 cover 품질 (QA-02 Availability)**: SP-1·R-1이 Orchestrator SPOF→QA-02로 cover. 단 ⚠️ QA-02의 **현 주 KPI는 `재기동 ≤4분 AND in-flight 손실=0`**(durable state 기준)인데, ATAM은 옛 "MTTR<1분"만 인용. 3안 Standby의 페일오버가 **무손실 재개(손실=0 게이트)**를 보장하는지가 핵심인데 "(추정)"으로 닫힘. 또 **외부 LLM outage/429 시나리오 미명시**(OI-7) — QA-02 정의가 "외부 LLM 제공자가 더 큰 위협"이라 명시하는데 ATAM R에 없음.
- **KPI 앵커·변별**: ★매트릭스 칸이 QA 실제 KPI 임계값에 **미앵커**(Controllability ★★★ ↔ ack≤5초/graceful≤30초 연결 없음, Availability ★★★ ↔ 재기동≤4분/손실=0 연결 없음). **3안 별점은 전부 "추정"** — 1안 대비 Availability만 ★★☆→★★★로 올렸으나 페일오버 시간(MTTR)·일관성(QA-11)·대기 자원 비용이 미검증(BL-1 자인). 추정 별점은 비교축으로 약하다.
- **결론**: ASR coverage 폭은 세트 최고지만, KPI 앵커링·3안 검증이 약해 KPI 축은 △.

## 렌즈 1 — Agentic Workflow 전문가 관점
- **설득력·비식상성**: ⚠️ 1안(중앙 Orchestrator)·2안(완전 분산)은 **agentic 토폴로지의 교과서 1차 해법**에 가깝다(다소 식상). 3안 Standby가 난이도를 더하지만 본질은 "SPOF에 Active-Passive 붙이기"라는 정석. **더 나은 agentic 정석 누락**: (a) **HITL gate 설계의 입도**(어느 액션이 gate를 거치는가 — QA-06 고위험 액션 매핑) 미상세, (b) **Hierarchical Federation(BL-2, 도메인별 Sub-Orchestrator)**이 단일 Orchestrator 병목(TP-2)을 푸는 더 흥미로운 대안인데 _backlog에만 있고 정식 대안으로 안 올라옴.
- **trade-off 타당성**: TP-1(제어 집중↔가용성)은 agentic 현실과 정확히 맞다(중앙 정책=제어성↑·SPOF↑). 단 2안 단점 "[Controllability] 정책 분산 → 추가 비용(HITL·토큰)"은 **토큰 경제**를 건드리는데 QA-05(Efficiency, ASR)와 연결만 하고 정량 없음.
- **agentic 고유 제약**: 3안 Standby의 **상태 외부화 + 페일오버 중 일관성**은 QA-11(재현율) 비결정성과 충돌 가능 — 페일오버 시점에 in-flight Agent가 LLM 호출 중이면 재개 시 다른 산출물(F-05형). BL-1이 "페일오버 중 일관성(QA-11)"을 미검증으로 자인하나 ATAM SP/Risk로 승격 안 됨.
- **runaway·HITL**: QA-03 정의의 핵심인 **runaway cap(max iter·token·wall-clock)**이 1안 HITL과 별개 축인데 ATAM에 없음(OI-7). 자율 에이전트 대표 실패(폭주)를 제어 토폴로지 결정이 안 다루는 것은 공백.

## 렌즈 2 — ATAM 평가 수석 아키텍트 관점 (ASR cover·KPI 비교·분석 건전성)
- **Well-framed / 직교성**: ○ "중앙 제어성 vs 가용성" 단일 교환축으로 잘 framed. realizes FR-0004 연결. 단 ⚠️ **제어평면(Orchestration vs Choreography)이 DP-04의 축 B(A3 vs A6)와 동일 결정**인데(F-03 이중 제어평면), DP-02 본문은 이 동시 결정성을 **명시하지 않는다** — evaluation.md(dp4)는 "DP-0002와 동시 결정"이라 못박았으나 DP-02 쪽은 일방. 이는 결정 간 의존 누락(report C*, severity High 후보).
- **대안 완전성·공정성**: △ 1·2·3안은 충분하나 **BL-2(Federation)가 빠진 대안 공간**. 또 3안만 "(추정)"이고 1·2안은 확정 별점 — 공정 비교 위해 1·2안도 KPI 앵커 필요.
- **조합/창발 비용**: N/A(단일 택1 결정) — 단 3안 Standby는 "1안 + 이중화"라 조합 성격, 대기 인스턴스 자원·페일오버 일관성 비용 미산정.
- **핵심 교환점 노출**: ◎ TP-1을 "본 DP의 핵심 교환점"으로 정직 노출 — 순이익 위장 없음.
- **가중**: ○ "Controllability가 H 중요도"를 결정 근거에 반영(1안 기반 권고). 단 QA-02도 H인데 1안 기반은 QA-02를 R-1로 깎으므로, **H↔H 교환**을 3안으로 봉합 — 이 봉합의 KPI 검증이 핵심.
- **★ 등급 척도 경계**: 3안 별점 "추정"은 ★ rubric 캘리브레이션 대상이 아니라 **미측정**. PoC 전까지 비교축 불가.

## 렌즈 3 — Runner 인프라 아키텍트 관점 (구현 현실성)
- **구현 환원**: ○ 3안 = Active-Passive + Heartbeat + State resync(BL-1)로 환원 — 인프라 정석(leader election·외부 상태저장). 단 "상태 외부화"의 구체 메커니즘(어디에·무엇을·어떤 일관성으로 외부화)이 미상세.
- **blast-radius / FMEA**: ⚠️ 1안 Orchestrator = **단일 공유 장애도메인**. Orchestrator 다운 시 blast-radius = 전 Workflow. QA-02 재기동≤4분/손실=0을 충족하려면 Standby 페일오버 시간 + 상태 동기화 지연을 합산해야 하는데 정량 없음. **DP-04 A8 로컬 디스크 유실 + DP-02 Orchestrator + DP-05 공유 캐시 = 공유 장애도메인 누적**(F-09, report C*).
- **데이터 평면**: TP-2(Orchestrator 경유 병목→QA-10)는 잡았으나 정량(Orchestrator hop이 E2E 5% budget 잠식하는지) 없음.
- **확장·격리·제어**: NR-1("Scalability 1·2안 동급")은 단일 Orchestrator가 WF 수 폭증 시 병목이 될 수 있음을 과소평가 — BL-2 Federation이 바로 이 한계를 푸는 대안인데 NR로 닫아버려 **확장 위험을 N으로 숨김**.
- **채택/운영 리스크**: Standby 이중화의 대기 인스턴스 상시 비용(QA-13 Cost)·운영 복잡도가 ATAM Risk에 없음(F-11).

> **구현·측정 메커니즘**: ① 페일오버 PoC — Orchestrator kill → Standby 승격까지 `failover_time = heartbeat_timeout + state_resync + leader_election` 측정, **QA-02 재기동≤4분 AND in-flight 손실=0** 충족 검증(특히 손실=0은 게이트). ② graceful stop — 중단 명령 주입 → `ack_time`(≤5초)·`graceful_stop+rollback`(≤30초)을 1안 HITL 경로에서 계측(QA-03 main축). ③ 페일오버 중 일관성 — 승격 직후 in-flight Agent 재개 산출물을 `pass^k`(QA-11)로 측정해 비결정 영향 정량화.

## 판정
| 축 | 판정 | 근거 |
|---|:---:|---|
| **ASR-coverage** | ○ | ASR 3개(QA-01·02·03) cover — 세트 최다 · TP-1으로 QA-02↔QA-03(둘 다 H) 직접 교환 · 단 QA-01은 N으로 변별 0, BL-2 대안 누락 |
| **KPI-comparison** | △ | ★칸이 QA 실제 KPI에 미앵커(Controllability↔ack/graceful, Availability↔재기동/손실=0 연결 없음) · 3안 별점 전부 "추정"(미검증) · runaway cap·외부 LLM outage 차원 미명시(OI-7) |
| **Well-framed** | ○ | 단일 교환축으로 framed · FR-0004 연결 · 단 제어평면이 DP-04 축 B와 동시 결정임을 본문 미명시(F-03) |
| **ATAM-sound** | ○ | TP-1 핵심 교환점 정직 노출 · 단 페일오버 일관성(QA-11)·runaway cap 미승격, NR-1이 확장 위험을 N으로 숨김 |
| **Convergent** | △ | "1안 기반 + 3안 완화 검토"로 방향은 있으나 **택일 트리거·배제기준 부재**(언제 2안? 언제 3안 확정?) · 3안 미검증으로 결정 미수렴 |
| **Traceable** | ○ | _backlog BL-1 역참조 · realizes FR-0004 · 단 QA-05 토큰 연결만 하고 정량 없음, BL-2 미승격 |

## Stage 2 권고
1. **★칸 KPI 앵커 (핵심)**: `Controllability ★★★ → ★★★ (ack≤5초·graceful stop+롤백≤30초 충족 시)` / `Availability ★★☆/★★★ → 칸 옆 정량: 재기동 ◯분·in-flight 손실=0 여부`. 막연 별점을 QA-02/03 현 KPI에 묶는다.
2. **3안 별점 "추정" 해소**: BL-1의 미검증 trade-off(페일오버 시간 vs 재기동≤4분 / 일관성 QA-11 / 대기 자원 QA-13)를 PoC로 측정해 추정→실측 전환. 미측정이면 ★ 대신 `◯`로 표기해 "추정 별점이 비교축처럼 보이는 것"을 막는다.
3. **runaway cap·graceful stop 명시 (OI-7)**: ATAM에 "SP-3(runaway cap → QA-03)" + 후보 대안에 협조적 취소(graceful) vs hard-kill 폴백 분리. 1안 HITL과 별개 안전축으로.
4. **외부 LLM degradation 시나리오 (OI-7)**: R-3 신설 — 외부 LLM outage/429 시 backoff+큐잉 자동 재개(QA-02 정의의 지배적 장애원). 1·2·3안이 이 시나리오를 어떻게 흡수하는지.
5. **제어평면 동시 결정 명시 (F-03, report C*)**: DP-02 본문에 "이 결정은 DP-04 축 B(A3 Choreography vs A6 Orchestration)와 **동시 단일화**(이중 제어평면 금지)" 한 줄 + cross-link.
6. **BL-2 Federation 정식 대안 승격 검토**: 단일 Orchestrator 병목(TP-2)·NR-1의 확장 한계를 푸는 4안으로 _backlog→대안 승격. NR-1을 "확장 안전(N)"에서 "조건부 위험(R)"으로 재판정.
