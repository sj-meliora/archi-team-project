# Module View — To-Be

> category: artifact | updated: 2026-06-22
> cross-link: DP-0002, DP-0003, FR-0004

---

## 구성 개요

두 시스템이 좌우로 배치된다. 왼쪽의 **SDK 개발 자동화 시스템**이 기존 시스템이고,
오른쪽의 **Agent Management 시스템**이 사이드카로 붙는다.
두 시스템 사이의 연결은 Agent Management 시스템 내부의 **Agentic Interface**를 통해 이루어진다.
Agentic Interface는 두 시스템이 맞닿는 경계면에 세로로 길게 배치된다.

---

## 1. SDK 개발 자동화 시스템 (As-Is 기반)

3개 레이어가 상→하로 쌓인다.

### UI 레이어

터미널 모듈 4개가 가로로 나열된다.

| 모듈 |
|---|
| Dashboard |
| Verification |
| Workflow |
| Result |

### Module 레이어

두 개의 모듈 컨테이너가 가로로 나열된다.

**Main Module**

| 모듈 |
|---|
| Workflow Manager |
| SDK Manager |
| Config Manager |
| Task Manager |

**Workflow Node Module**

| 모듈 |
|---|
| Node Executor |
| Verification Execution Manager |

### Repository 레이어

두 개의 스토리지 컨테이너가 가로로 나열된다.
컨테이너 자체에는 색을 칠하지 않고, 하위 항목은 DB 카드(상단 색 띠 있는 사각형)로 표현한다.

**System DB**

| 항목 |
|---|
| Config / State |
| Workflow Metadata |

**Artifact Storage**

| 항목 |
|---|
| Build Artifacts |
| Model Configs |

---

## 2. Agent Management 시스템 (사이드카)

### Agentic Interface

두 시스템의 경계면에 세로 띠 형태로 배치된다.
내부 구현이 미확정이므로 이름만 표기하고 세부 모듈을 두지 않는다.

### Agent Module

| 모듈 |
|---|
| Orchestrator |
| LLM Endpoint |
| Skills Manager |
| Permission Manager |
| Tool Adapters (JIRA · Build · Config) |

### Verification Module

| 모듈 |
|---|
| Test Runner |
| Regression Analyzer |
| Report Generator |
| Issue Tracker |
| Result Store |

---

## 3. 시스템 간 인터페이스

- 두 시스템 사이에 양방향 굵은 화살표(flow)를 배치한다.
- 화살표는 SDK 개발 자동화 시스템의 **Module 레이어 높이**에 위치한다.
- 구체적인 호출 프로토콜·API는 미확정 (Agentic Interface 설계 시 확정 예정).

---

## 4. 시각 표현 규칙

| 요소 | 표현 방식 |
|---|---|
| 시스템 외곽 | 직각 사각형 테두리, 흰 바탕 |
| 레이어 경계 | 직각 사각형 테두리 (점선 없음) |
| 모듈 컨테이너 (Main Module 등) | 직각 사각형, 색 없음 |
| 터미널 모듈 — SDK 시스템 | 하늘색 채움 (`#C8E6FA`, 테두리 `#5AAEE0`) |
| 터미널 모듈 — Agent 시스템 | 청록색 채움 (`#B2E8D8`, 테두리 `#3DB898`) |
| Repository 하위 항목 | 흰 바탕 + 상단 하늘색 띠 DB 카드 형태 (`rx=4`) |
| Agentic Interface | 청록색 세로 띠, 텍스트 90° 회전 |
| 시스템 간 화살표 | 양방향, stroke-width 6, 회색 (`#888`) |

---

## 5. 설계 근거 cross-link

- **DP-0002** (Agent Hierarchy): Orchestrator → Workflow Manager 단일 제어 진입점.
- **DP-0003** (외부 시스템 안정성): Permission Manager가 Tool Adapters 사용 전 사전 권한 체크.
- **FR-0004** (Agent 제어·관측): Report Generator / Issue Tracker가 Agent 실행 관측 지원.
