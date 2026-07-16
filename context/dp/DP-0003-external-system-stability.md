# DP-0003 Agent 외부 시스템 안정성 보장

> category: DP | status: 결정대기 | source: pptx p.31 | updated: 2026-07-16
> drives: QA-03(Controllability), QA-02(Availability), QA-04(Observability), QA-05(Efficiency)
> realizes: FR-0004 | constrained-by: C-02(기존 시스템 무영향)
> 재구성 note: 2026-07-16 — `reference/`에 추가한 Cheng & Cheng(2026, Decoupled HITL) · Chen et al.(2026, Securing Computer-Use Agents 제어 스택) · ISO/IEC 25059(User Controllability)를 반영해 **구 1안(사전 권한 체크)+구 2안(격리 환경)을 "사전 통제"로 통합**하고, 대안 비교축을 tactic 차이 → **HITL 구조 차이(Embedded vs Decoupled)**로 재구성. 구 3안(실시간 모니터링)은 대안이 아니라 두 대안 공통 보강책으로 재배치.

## 결정 포인트
Agent가 외부 시스템(Jira·빌드서버 등)에 영향을 주지 않도록 안정성을 보장하면서, 고위험 액션의 HITL(사람 승인) 게이트를 **어떤 구조로 둘 것인가** — 승인 로직을 게이트/에이전트에 내장(embedded)할지, 독립 컴포넌트로 분리(decoupled)할지.

## 사전 통제 (공통 기반 — 두 대안 공통)
구 1안(사전 권한 체크)과 구 2안(격리 환경 수행)은 둘 다 "사후 대응이 아닌 사전 차단"이라는 같은 계열이라 하나로 합친다: 실행 전(admission) 권한 게이트(allowlist)로 명백한 위반을 값싸게 거르고, 게이트를 통과한 액션도 격리 경계(브로커/프록시)를 거쳐서만 외부 시스템에 닿는다. 두 tactic은 대체재가 아니라 **겹 방어(defense-in-depth)** — 게이트는 "무엇을 허용하는가", 격리는 "허용된 것도 직접 경로로 못 나가게"를 맡아 旧 R-1(미정의 액션 우회)을 구조적으로 닫는다.

- **tactic/pattern**: Authorization gate, Allowlist, Policy enforcement point, Sandbox, Broker/Proxy, Bulkhead
- **근거**: Chen et al. §VII-D 제어 스택 — "Constrained action interfaces"(게이트)와 "Sandboxed and scoped authority"(격리)를 배포 시점(deploy-time) 통제로 함께 제시하며 "이 통제들은 누적적이다 — 권한 없는 액션 제약은 provenance 문제를 남기고, provenance 없는 격리는 광범위 권한을 그대로 둔다"고 명시.

이 공통 기반 위에서, 고위험 액션의 **HITL 승인 로직을 어디에 둘 것인가**가 실제 갈림길이다.

## 후보 대안

### 1안. Embedded HITL — 게이트 내장형
- **구조**: 사전 통제(게이트+격리) 통과 후 고위험 액션(배포·삭제 등) 판정 시, **승인 로직이 게이트 코드 안에 하드코딩**되어 동기적으로 사람 승인을 기다린다. 승인 조건·대상자·채널이 에이전트·워크플로마다 개별 정의된다.
- **tactic/pattern**: Authorization gate + Sandbox(공통) + **Embedded approval logic**(HITL을 애플리케이션 로직에 결합)
- **장점**: [Controllability] 별도 홉 없이 같은 프로세스 안에서 즉시 차단 — 구현 단순·저지연 / [Efficiency] 별도 인프라 불요, 토큰·네트워크 오버헤드 최소
- **단점**: [Maintainability] 승인 임계치·감사 로그 형식이 에이전트마다 중복 정의돼 거버넌스 정책 일관 적용이 어렵다("embedded HITL reduces reusability... governance policies difficult to enforce uniformly", Cheng & Cheng §III-B) / [Observability] 승인 이력이 에이전트별로 흩어져 시스템 차원 감사·패턴 분석이 어렵다 / 새 에이전트·워크플로 추가마다 승인 로직을 재구현해야 해 자율성 확대(점진적 자동화)를 체계적으로 관리하기 어렵다.

### 2안. Decoupled HITL — 독립 컴포넌트형
- **구조**: 사전 통제(게이트+격리)는 1안과 동일하게 유지하되, 고위험 액션 판정을 **독립 HITL 컴포넌트**로 위임한다. 게이트는 액션 컨텍스트를 담아 `POST /hitl/request`로 위임하면, HITL 컴포넌트가 개입 조건(**WHEN**: 리스크·신뢰도 임계치), 승인 역할(**WHO**: 조직 롤 동적 해석), 상호작용 형태(**WHAT**: Approve/Reject/Modify/Defer), 채널(**WHERE**: Slack·이메일·대시보드)을 판정해 라우팅하고, 에이전트는 폴링(`/hitl/get-decision`) 또는 콜백으로 결과를 받아 재개한다.
- **tactic/pattern**: Authorization gate + Sandbox(공통) + **Decoupled HITL component**(독립 서비스로서의 Broker), Policy-as-code 중앙집행, Async request–resolution
- **장점**: [Maintainability] 승인 정책·감사 로그를 한 곳에서 정의·집행해 에이전트·워크플로 간 재사용("HITL... as a shared service", Cheng & Cheng §III-C) / [Observability] 승인 상호작용이 한 컴포넌트에 모여 시스템 차원 관측·분석 가능 — QA-04와 직결 / [Controllability] 반복 승인 패턴을 관측해 점진적 자율성 확대(자동 승인 임계 상향)를 체계적으로 관리할 근거 확보(Cheng & Cheng §III-E)
- **단점**: [Performance] request–resolution 왕복(네트워크 홉)이 1안 대비 추가 지연 / [Efficiency] 별도 인프라(HITL 서비스) 구축·운영 비용 / [Availability] HITL 컴포넌트 자체가 새 장애점 — callback 모드는 인바운드 연결이 필요해 C-02(온프레미스) 제약과 상충 가능(→ 폴링 모드 기본 권장).

### 3안. 실시간 모니터링 (공통 보강책 — 대안 아님)
구 3안은 더 이상 별개 대안이 아니라, 1안·2안 어느 쪽을 택하든 위에 얹는 보강 계층으로 재배치한다: Agent↔외부 직접 경로는 사전 통제로 이미 막혀 있으므로, 모니터링은 "차단"이 아니라 **관측·이상탐지·롤백**(QA-04 보강, Chen et al.의 Runtime verification+escalation 계층)을 담당한다.

- **tactic/pattern**: Runtime monitoring, Anomaly detection, Rollback
- **장점**: [Observability] 실행·판단 근거 추적 / [Availability] 빠른 탐지·복구로 MTTR 단축
- **단점**: [Efficiency] LLM 기반 탐지 시 토큰 폭증 → 규칙 기반 한정 적용 권고(기존 결정 유지)

## Trade-off 매트릭스
| 대안 | HITL 구조 | Availability | Controllability | Maintainability | Efficiency |
|---|---|:---:|:---:|:---:|:---:|
| 1안 Embedded HITL | 게이트 내장 | ★★☆ | ★★★ | ★☆☆ | ★★★ |
| 2안 Decoupled HITL | 독립 컴포넌트 | ★★☆ | ★★★ | ★★★ | ★★☆ |

> 두 대안 모두 사전 통제(게이트+격리) 공통 기반이라 Availability·Controllability 핵심 축은 동률 — **갈림은 Maintainability(거버넌스 재사용성) ↔ Efficiency(홉·인프라 비용)**.

## ATAM 분석
### 민감점 (Sensitivity Points)
- **SP-1(유지)**: 차단 시점(사전 vs 사후)이 Availability(QA-02)·C-02 보장 강도에 민감 — 사전 통제(게이트+격리) 통합으로 1안·2안 공통 해소.
- **SP-2(유지)**: 탐지 방식(규칙 vs LLM)이 Efficiency(QA-05, 토큰)에 민감 (3안, 공통 보강책).
- **SP-3(신설)**: HITL 로직 위치(내장 vs 분리)가 Maintainability·Observability에 민감 — 에이전트 수가 늘수록(DP-01 오케스트레이터 하위 노드 확대 시) 격차가 커진다.

### 교환점 (Tradeoff Points)
- **TP-1(유지, Availability ↔ Performance)**: 격리 경유는 구조적 안전성↑·정상경로 오버헤드↑ (공통 기반).
- **TP-2(신설, Maintainability ↔ Efficiency)**: 2안(Decoupled)은 거버넌스 재사용성↑·네트워크 홉·인프라 비용↑; 1안(Embedded)은 그 반대.
- 旧 TP-2(예방력 ↔ Observability)는 3안 재배치로 소멸 — 모니터링이 더 이상 "예방 대안"이 아니므로.

### 위험 (Risks)
- **R-1(해소)**: 旧 "1안 단독은 미정의 액션 우회로 C-02 위반 위험" — 사전 통제 통합(게이트+격리 겹 방어)으로 구조적으로 닫힘.
- **R-2(유지, 재배치)**: 3안(모니터링) 단독 채택 시 사후 탐지라 이미 발생한 외부 영향을 차단할 수 없다 — 그래서 대안이 아닌 보강책으로 재배치.
- **R-3(신설)**: 2안의 HITL 컴포넌트가 신규 장애점(SPOF) — callback 모드는 인바운드 연결이 필요해 C-02(온프레미스) 제약과 상충 가능. 폴링 모드 기본 + HITL 컴포넌트 자체 가용성 설계(DP-01 Standby 패턴 재사용 검토) 필요.
- **R-4(신설)**: 1안의 승인 로직 중복이 에이전트 수 증가 시 정책 드리프트(에이전트마다 다른 승인 임계치) 위험(Cheng & Cheng §III-B).

### 비위험 (Non-Risks)
- **NR-1(유지)**: 게이트의 저오버헤드는 QA-05(토큰) 관점에서 안전 — 사전 통제 공통 기반이므로 두 대안 동일.

## 결정 / 근거
- (미정.) 권고: 사전 통제(게이트+격리)는 C-02가 강제 제약이므로 **두 대안 공통 필수**로 확정. 그 위에서, **DP-01이 이미 오케스트레이터 하위 다중 노드(파이프라인 단계별 에이전트)를 전제**하므로 HITL 승인 정책이 여러 에이전트에 걸쳐 반복될 가능성이 높다 — **2안(Decoupled HITL)을 우선 권고**, R-3(신규 SPOF)은 폴링 모드 기본 채택 + 필요시 DP-01 Standby 패턴 재사용으로 완화한다. 단일 에이전트·저지연이 우선인 시나리오만 남는다면 1안으로 축소해도 무방하다. 3안(모니터링)은 두 대안 공통으로 규칙 기반 한정 적용한다.

## 근거 / 레퍼런스
| 결정 요소 | 왜 이렇게 했나 | 출처 |
|---|---|---|
| 사전 통제(게이트+격리) 통합 | 배포 시점 통제(권한 바인딩)는 게이트+격리가 누적적으로 작동 — 하나만으로는 불충분 | Chen, Sood et al., *Securing Computer-Use Agents: A Unified Architecture & Lifecycle Framework for Deployment-Grounded Reliability*, §VII-D 통제 스택 |
| HITL 구조 비교축 (Embedded vs Decoupled) | HITL을 애플리케이션 로직에 내장하면 재사용성·거버넌스 일관성이 떨어지고, 독립 컴포넌트로 분리하면 여러 워크플로가 공유 가능 | Cheng & Cheng, *A Decoupled Human-in-the-Loop System for Controlled Autonomy in Agentic Workflows*, §III |
| WHEN/WHO/WHAT/WHERE 4축 | 2안 HITL 컴포넌트의 개입조건·역할·상호작용·채널 설계 모델 | Cheng & Cheng §IV |
| User Controllability 정의 정합 | "사람 또는 외부 에이전트가 시의적절하게 개입 가능해야 함"이 본 DP의 QA-03 지향과 정합 | ISO/IEC 25059:2023 §5.2 |

> 전문 발췌는 `reference/` 참고 — `a-decoupled-human-in-the-loop-system-for-controlled-autonomy-in-agentic-workflows.md`, `securing-computer-use-agents-a-unified-architecture-lifecycle-framework-for-deployment-grounded-reliability.md`, `iso-iec-25059-2023-quality-model-for-ai-systems-sample.md`.
