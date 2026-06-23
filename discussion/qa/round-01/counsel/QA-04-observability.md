# Counsel: QA-04 Observability — Agent 작업 추적 용이성

> refs-review: round-01/review/QA-04-observability.md · report.md(렌즈3 횡단·C4)
> seats: 발의 Seat 1(Agentic Workflow) · 합의 consensus (Seat 3 event-history 환원, Seat 2 3축·MTTD)
> stance: 채택 권장 (KPI를 trace 완전성으로 환원 + 관측 3축 명시)

## Reviewer 지적 요약
- `재구성 가능 비율 ≥ 95%`의 **운영 정의 부재** — 단위(결정 1건? WF?)·판정 기준(이진? 감사자?)이 없어 측정 불가 (렌즈2).
- 재구성에 필요한 최소 span 미정의: **prompt + context + tool I/O + model ver + token + ts** (렌즈1).
- 관측성 폭이 좁음 — traces 1축뿐. metrics(처리량·에러율·비용)·alerting·**MTTD** 누락 (렌즈2).
- durable 엔진의 **event-history 완전성**으로 환원하면 측정이 명확 (렌즈3).

## 개선안 (정의·KPI 기존→제안)

**정의**
- 기존: "실행 과정·판단 근거 추적."
- 제안: "agent loop 전구간(prompt·context·tool I/O·model·token·ts)을 span으로 캡처해 의사결정을 재구성하고, traces·metrics·alerting 3축으로 운영을 관측한다."

**KPI**
| # | 기존 | 제안 | 비고 |
|---|---|---|---|
| ① | 의사결정 재구성 ≥ 95% | **trace 완전성: 안전관련 액션 100% / 일반 액션 ≥ 95%** (span = prompt+context+tool I/O+model ver+token+ts) | "재구성"을 span 완전성으로 환원. PoC-O1 |
| ② | — | **event-history 완전성 = 100%** (워크플로우 골격 재생) | durable 엔진 기준. LLM 비결정만 prompt/seed 캡처로 보강 |
| ③ | — | **MTTD(이상 탐지) ≤ ◯분** | QA-02 MTTR의 전제. ◯는 PoC-O1 |
| ④ | — | **결정당 비용/토큰 기록 = 100%** | QA-05/NQA-C 측정 인프라 제공 |

## 근거 (레퍼런스)
- **OTel GenAI semantic conventions**: `invoke_agent/chat/execute_tool` span + `gen_ai.usage.*`(토큰)·model·finish_reason로 agent 단계를 표준 계측 → "재구성 가능"을 검증 가능한 trace 완전성으로 환원. (§4 관측성) — https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/ , https://opentelemetry.io/blog/2025/ai-agent-observability/
- **event history replay**: durable 엔진은 모든 전이를 이벤트로 남겨 결정론적 재생 → 골격 ~100% 재구성 — Temporal. (§4·렌즈3) — https://docs.temporal.io/temporal
- **traces+metrics+logs 3축·SLI**: 관측 3축과 MTTD/alerting은 SRE 표준. (§4) — https://sre.google/workbook/implementing-slos/
- ⚠️ 95%·MTTD ◯분은 우리 측정으로 확정.

## PoC 증명법

### PoC-O1: span/event-history 완전성을 측정한다 (계측)
- **가설**: "OTel GenAI span + event-history로 안전 액션 100%·일반 95% 재구성이 측정 가능하다."
- **지표**: full-span 보유 비율(액션 분모별), event-history 완전성(%), MTTD(이상 주입→탐지 시간).
- **셋업**: mock 파이프라인에 OTel GenAI span(invoke_agent/execute_tool) 삽입 + durable 엔진 event-history + W3C traceparent 전파 + 간단 이상탐지 룰.
- **절차**: ① 워크플로우 1건 실행 후 한 결정을 trace만으로 재구성 시도 → ② 누락 span 카테고리 집계 → ③ 이상 주입 후 MTTD 측정.
- **합격(Exit)**: 안전 액션 span 100%, 일반 ≥95%, 한 결정이 trace만으로 재현됨, MTTD ≤ ◯분.
- **규모/기간**: 결정 수십~100건, 약 2일.
- **리스크/한계(silent cap)**: "재구성됨"의 판정을 자동화하기 어려움(감사자 판단 개입 여지) → 판정 룰을 명시 log. LLM 내부 추론(CoT 비공개)은 prompt/출력으로만 근사.

## DP·발표 영향
- **DP 연결**: DP-0003(3안 실시간 모니터링)이 span·MTTD에 직결 — "규칙 기반 한정 적용" 권고와 일관(LLM 탐지는 토큰 폭증). **span 수준 trace를 DP-0003이 보장하는지 역검토**.
- **연계**: QA-05/NQA-C(결정당 비용 trace)·QA-03/NQA-A(안전 액션 100% 추적)·QA-02(MTTD→MTTR)의 측정 인프라를 QA-04가 공급 → 관측성이 다수 QA의 토대임을 발표에서 강조.
