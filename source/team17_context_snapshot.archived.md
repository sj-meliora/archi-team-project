# 17조 팀 과제 — 컨텍스트 스냅샷

> **이 문서는 무엇인가**
> 팀 공유 pptx(`SW_Architect_17조_-_팀_과제_.pptx`)에서 슬라이드 생성에 필요한 컨텍스트(품질 속성·요구사항·제약·설계 결정)만 추출한 **스냅샷**입니다.
> 자료가 계속 변하므로, 이후에는 이 md를 기준으로 **바뀐 부분(델타)만** 전달하면 매번 전체를 다시 설명할 필요가 없습니다.
>
> - **추출 기준 파일**: SW_Architect_17조_-_팀_과제_.pptx (45 페이지)
> - **추출 시점**: 2026-06-19
> - **발표 예정일**: 2026-07-07

---

## 1. 과제 개요

**시스템명**: Agentic AI 기반 On-device AI SDK 개발 자동화 시스템 (Agentic DevOps System)

**한 줄 정의**: NPU 구동용 On-device AI SDK(IR Converter → Graph Optimizer → Quantizer → Compiler)의 개발 파이프라인을, Agent가 사람 개입 없이 빌드·검증·배포·이슈처리까지 자동화하는 시스템.

**배경 3축**
| 축 | 내용 | 함의 |
|---|---|---|
| (수요) | On-device AI 중요성 증가, 추론이 Cloud → Edge로 이동 | NPU가 제품 경쟁력 핵심 자산 → SDK가 가치 실현의 critical path |
| (워크로드) | AI 모델 폭발적 증가 (모델 140개+, 단계별 iteration 다수) | 수작업·선형 인력으로 감당 불가 |
| (방법론) | Agentic AI 성숙 — LLM이 도구 사용·계획·실행 가능 | 개발 프로세스 자동화 가능 |

**핵심 문제 (Pain Point)**
- 대용량 산출물(20GB+) 수동 공유 과정에서 파일 Loss 등 신뢰성 이슈
- Tool Version/Config가 개발자 개인별 관리 → 추적성·Regression 확보 어려움
- Jira 상태가 개발자 수동 업데이트 의존 (속도 때문에 채팅으로 우회)
- 조합수 폭발: 모델 수 × 세대 수 × 파이프라인 단계
- 분석 도구 한계: 고정 전체 보기만 제공, 커스텀 뷰 불가

---

## 2. Stakeholder

| Stakeholder | 역할 |
|---|---|
| **모델 PM** (Project Manager) | 프로젝트 관리, 릴리즈 관리, 우선순위 조정, 품질 게이트 승인 |
| **SDK 개발자** | Tool 개발, 모델 실행, 이슈 처리 (Converter/Optimizer/Compiler 컴포넌트 단위) |
| **플랫폼 개발자** | Workflow 관리, Node-Agent 매핑 (CI/CD, Agentic DevOps 인프라 운영·자동화) |

---

## 3. 기능 요구사항 (FR)

| ID | 이름 | 상세 | Agent 특화 |
|---|---|---|---|
| **FR1** | Workflow 실행 인프라 | 노드 단위로 실행을 격리·재시도·복구할 수 있는 Workflow 실행 인프라 제공 | |
| **FR2** | Artifact 저장·전달 | 대용량 산출물을 저장·전달하는 관리 인프라 제공 | |
| **FR3** | SDK Config 변경 추적 | SDK Config(툴 버전·설정 파일) 변경 시 영향 범위 자동 분석 + Regression 관리 | |
| **FR4** | Agent 제어·관측 | Agent 동작을 제한·통제하고, 실행 이력·추론 과정 추적 (운영·관측) | ✔ |
| **FR5** | 시스템 운영·제어 | 전체 현황 조회 + Workflow·Agent·Tool 구성 변경 통합 제어 인터페이스 | (?) |

---

## 4. 제약 사항 (Constraint)

| ID | 이름 | 상세 |
|---|---|---|
| **C1** | 표준 패키징 | On-device SDK는 Python Package 형태, Ubuntu 기반 Docker 환경에서 구동 |
| **C2** | 배포 이식성 | 클라우드 외 내부 개발 서버에도 직접 구축 가능해야 함. Agent 오동작 시에도 기존 시스템(Workflow·Jira·빌드 서버 등)에 영향 없어야 함 |

---

## 5. 품질 속성 (QA) — 핵심 컨텍스트

> ⚠️ QA 번호는 슬라이드마다 달라집니다(정합성 메모 참고). 아래는 **상세 표(슬라이드 13)** 기준으로 정리한 canonical 버전입니다.

| ID | 품질 속성 | Refinement | 시나리오 / KPI | 중요도 | 난이도 |
|---|---|---|---|:---:|:---:|
| QA01 | Efficiency | Agent 토큰 사용량 | 토큰 사용량 최소화 / **단일 요청 총 토큰 ≤ 8k** | H | H |
| QA02 | Scalability | 시스템 확장성 | 모델·워크플로우 수 증가에 비례 확장 / **시간당 완료 모델 수 ≥ N, 자원 활용률 ≥ 70%** | H | H |
| QA03 | Availability | 운영 안정성 | 부분 장애 시 서비스 연속성 보장 / **MTTR < 1분** | H | H |
| QA04 | Controllability | Agent 제어 용이성 | 허용 범위 내에서만 동작 / **중단 명령 수용 시간 ≤ 5초** | H | M |
| QA05 | Observability | Agent 작업 추적 용이성 | 실행 과정·판단 근거 추적 / **의사결정 재구성 가능 비율 ≥ 95%** | M | M |
| QA06 | Reliability | Workflow 간 독립성 보장 | 특정 Workflow 장애가 타 Workflow에 무영향 / **타 Workflow 실행 중단 ≤ 1%, latency 증가 10% 이내** | M | H |
| QA07 | Performance | Agent 수행 시간 | 개발자 직접 수행 대비 단축 최대화 / **{개발자 노드 수행 시간 − Agent 자동 노드 수행 시간} 최대화** | H | H |
| QA08 | Performance | E2E 개발 시간 | 응답 시간·처리량 목표 충족 / **Artifact 전달 오버헤드 ≤ 전체 E2E의 5%** | M | M |
| QA09 | Reliability | Agent 결과 일관성 | 동일 입력에 일관된 판단 / **추론 재현 성공률 ≥ 80%** | L | H |
| QA10 | Maintainability | 모듈 교체 용이성 | 구성요소 변경 시 타 요소 영향 최소화 / **평균 Change Impact Scope ≤ 2개 컴포넌트** | L | M |

**QA01~04 척도 근거(슬라이드 25~28에 상세)**
- QA01 Efficiency: ★★★ 토큰 ≤4k (단일 작업 최적화) / ★★☆ 4k~6k (일반 Workflow) / ★☆☆ 6k~8k (복합 추론 허용)

---

## 6. Context Diagram (시스템 경계)

**시스템 경계**: `SDK 개발 자동화 시스템` (내부 = `< Agentic DevOps 시스템 >` + Monitoring/Logging + Server + NFS Storage)

| 외부 엔티티 | 관계(방향) |
|---|---|
| Platform Developer | System Development (→ 시스템) |
| SDK Developer | Request / Approval (↔) |
| Project Manager | Monitoring Data (← 시스템) |
| JIRA | API Call (→) |
| LLM Service | Request / Response (↔) |
| NPU | Run (←) |

내부 흐름: Agentic DevOps → Metrics → Monitoring/Logging / Agentic DevOps → SDK Execution → Server / Agentic DevOps ↔ Read·Write ↔ NFS Storage

---

## 7. Domain Diagram (컴포넌트 레이어)

`< Agentic DevOps 시스템 >` 5계층:

| 레이어 | 컴포넌트 |
|---|---|
| **Presentation** | Dashboard GUI, Metric |
| **Agent Capability** | Agent Manager, Permission Manager, LLM Endpoint, Skills Manager |
| **Workflow** | AI Model Manager, Workflow Manager, Node Manager |
| **Execution** | SDK Manager, Config Manager, Artifact Manager |
| **Resource** | Resource Manager, Storage Manager, NPU Manager, JIRA Manager |

외부: `< On-device AI SDK >` (IR Converter / Graph Optimizer / Quantizer / Compiler), `< NPU >` (Device Driver), NFS Storage

---

## 8. 설계 결정 (Design Points)

> 각 DP는 대안 비교 + 장단점 + Trade-off 별점(★★★ = 우수) 구조. 별점은 슬라이드 표기 그대로.

### DP1. Workflow – Agent 매핑
| | 1안: Per-Node Agent | 2안: Dynamic Agent Pool |
|---|---|---|
| 구조 | 노드별 전용 Agent 고정 배치 | Agent Router가 Pool에서 동적 할당 |
| 장점 | [Performance] 선택 없이 즉시 실행 / [Availability] 고정 구조로 장애 격리·운영 안정 | [Scalability] 동적 할당으로 확장 용이 / [Efficiency] 작업별 최적 Agent 선택 |
| 단점 | *(슬라이드 표기 오류 — 정합성 메모 참고)* | [Performance] Routing 지연 / [Availability] 동적 경로로 운영 복잡성 증가 |
| Trade-off | Efficiency ★★☆ · Scalability ★★☆ · Performance ★★★ · Availability ★★★ | Efficiency ★★★ · Scalability ★★★ · Performance ★★☆ · Availability ★★☆ |

### DP2. Agent Hierarchy
| | 1안: Hierarchical Multi-Agent | 2안: Fully Decentralized Multi-Agent |
|---|---|---|
| 구조 | Orchestrator 중심 (Workflow Engine 하위) | Agent 간 직접 통신 (Orchestrator 없음) |
| 장점 | [Controllability] 단일 지점 정책·HITL 승인 / [Scalability] Workflow별 인스턴스 생성 | [Availability] 단일 장애점 없음, 장애 무전파 / [Performance] 직접 통신, 오버헤드 적음 |
| 단점 | [Availability] Orchestrator 장애 = 전체 장애 / [Performance] Orchestrator 경유 병목 | [Controllability] 정책 분산 → 추가 비용(HITL·토큰) / [Scalability] 결합도 높아 구성 변경 시 호출관계 수정 |
| Trade-off | Controllability ★★★ · Scalability ★★☆ · Availability ★★☆ · Performance(E2E) ★★☆ | Controllability ★★☆ · Scalability ★★☆ · Availability ★★★ · Performance(E2E) ★★★ |

### DP3. Agent 외부 시스템 안정성 보장
| | 1안: 사전 권한 체크 | 2안: 격리 환경 수행 | 3안: 실시간 모니터링 |
|---|---|---|---|
| 구조 | 권한 체크 게이트(정책·allowlist), 허용만 통과 | 격리 경계 + 브로커/프록시 경유, 직접 경로 차단 | Agent↔외부 직접 + 실시간 모니터가 관측·차단·롤백 |
| 장점 | [Controllability] 허용 범위 직접 제어 / [Efficiency] 토큰 미소모·저오버헤드 | [Availability] 외부 직접 접근 차단 구조적 보장 / [Efficiency] 격리는 토큰 무관 | [Observability] 실행·판단 근거 추적 / [Availability] 빠른 탐지·복구로 MTTR 단축 |
| 단점 | [Availability] 미정의 액션·우회 경로 차단 불가 | [Performance] 정상 작업도 브로커 경유 / [Scalability] 격리 환경 수백 개 → 자원 점유 | [Availability] 사후 탐지(예방 불가) / [Efficiency] LLM 탐지 시 토큰 폭증 |
| Trade-off | Availability ★★☆ · Performance ★★★ · Efficiency ★★★ · Controllability ★★★ | Availability ★★★ · Performance ★★☆ · Efficiency ★★★ · Controllability ★★☆ | Availability ★☆☆ · Performance ★★☆ · Efficiency ★☆☆ · Controllability ★★★ |

### DP4. Workflow 실행 구조
| | 1안: 타입별 전용 서버 풀 | 2안: 노드당 Workflow 인스턴스 |
|---|---|---|
| 구조 | 노드 작업을 타입별 서버 풀로 라우팅 + 오브젝트 스토리지로 중간 Artifact 전달, 풀별 독립 scale out | 노드당 다중 Workflow 인스턴스 + 로컬 스토리지, 노드 단위 격리·scale out |
| 장점 | [Scalability] 노드 타입별 독립 scale out, 자원 utilization 유리 / [Performance] 사전 Staging으로 오버헤드 은닉 | [Reliability] 노드 격리로 장애 독립성 구조적 보장 / [Performance] 네트워크 전달 없음 → 오버헤드 최소 / [Maintainability] 인프라 단순 |
| 단점 | [Reliability] 노드 타입 장애가 해당 타입 쓰는 모든 Workflow로 전파 / [Maintainability] 풀·스토리지·Staging 관리 복잡 | [Scalability] Scale 단위가 Workflow 전체로 고정 → 병목 시 전체 복제로 자원 낭비 |
| Trade-off | Performance ★★☆ · Scalability ★★★ · Reliability ★★☆ · Maintainability ★☆☆ | Performance ★★★ · Scalability ★★☆ · Reliability ★★★ · Maintainability ★★★ |

### DP5. E2E 개발시간 최적화
| | 1안: Workflow별 로컬 캐시 | 2안: Workflow 간 공유 캐시 |
|---|---|---|
| 구조 | Workflow마다 독립 Cache | 중앙 Shared Cache 공유 |
| 장점 | [Reliability-Workflow] 한 Workflow 오류가 타에 무영향 / [Maintainability] 공유 인프라 없이 단순 | [Performance] 유사 Workflow 간 캐시 공유로 E2E 단축 / [Reliability-Agent] Workflow 간 Agent 판단 일관성 보장 |
| 단점 | [Performance] 유사 설정 타 Workflow 결과 재사용 불가 / [Reliability-Agent] Workflow 간 결과 편차 | [Reliability-Workflow] 공유 캐시 오류가 타 Workflow로 전파 / [Maintainability] 중앙 캐시 추가, 모듈 교체 범위 판단 필요 |
| Trade-off | Performance ★★☆ · Reliability-Workflow ★★★ · Reliability-Agent ★★☆ · Maintainability ★★★ | Performance ★★★ · Reliability-Workflow ★★☆ · Reliability-Agent ★★★ · Maintainability ★★☆ |

---

## 9. 정합성 메모 (자료 변동 흔적 — 슬라이드 생성 전 확인 필요)

이 md를 단일 기준으로 쓰려면 아래 불일치를 먼저 정리하는 게 좋습니다.

1. **QA 번호 불일치**
   - 슬라이드 11(Driver): QA01~04 = 토큰 / 확장성 / **Agent 수행 시간** / 안정적 운영
   - 슬라이드 13(상세 표): QA01~10, QA03 = **운영 안정성**, Agent 수행 시간은 QA07
   - 슬라이드 25~28(QA 상세): QA03 = **Agent 수행 시간**, QA04 = 운영 안정성
   → 같은 QA01~04가 슬라이드마다 다른 속성을 가리킴. **canonical 번호를 하나로 고정**해야 함 (본 문서는 슬라이드 13 기준).

2. **DP1 1안 '단점' 칸 표기 오류**
   - 1안(Per-Node)의 단점에 `[Scalability] 동적 할당으로 확장 용이 / [Efficiency] 작업별 최적 Agent 선택`이 적혀 있는데, 이는 **2안의 장점**임 (복붙 흔적). 1안의 실제 단점으로 교체 필요.

3. **DP4 대안 라벨 오류**
   - 헤더가 `Hierarchical / Fully Decentralized Multi-Agent`로 되어 있으나 (DP2 라벨 복붙), 실제 내용은 **타입별 서버 풀 vs 노드당 Workflow 인스턴스**임. 본 문서는 내용 기준으로 라벨 수정해 정리함.

4. **FR5 미확정**
   - `(?) 시스템 운영·제어`로 (?) 표기. Agent 특화 여부·범위 확정 필요.

5. **Domain Diagram 2개 버전 공존**
   - 슬라이드 15·16 vs 35·36에서 컴포넌트 배치(JIRA Manager 위치 등)가 다름. 어느 쪽이 최신인지 확정 필요.

6. **As-Is/To-Be 시나리오 2개 버전** (슬라이드 9 vs 34): Compiler 시작 vs Quantizer 시작으로 서술이 다름.
