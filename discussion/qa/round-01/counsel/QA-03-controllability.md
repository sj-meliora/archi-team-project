# Counsel: QA-03 Controllability — Agent 제어 용이성

> refs-review: round-01/review/QA-03-controllability.md · report.md(C2·C4·C5)
> seats: 발의 Seat 1(Agentic Workflow) · 합의 consensus (Seat 3 협조적 취소 구현, Seat 2 QA↔QAS 통일)
> stance: 채택 권장 (KPI 통일·확장; NQA-A Security와 경계 정리)

## Reviewer 지적 요약
- **QA↔QAS 불일치(C2)**: QAS-03엔 "허용범위 외 액션 통과 0건"이 있는데 QA-03 파일 KPI엔 "≤5초"만 → SSoT 분기.
- **누락 3축**: 최소권한·도구 allowlist·고위험 액션 HITL 게이트 / **runaway cap**(max iter·token·wall-clock) / "0건"의 **검증수단(적대적 eval)** (렌즈1).
- **stop 의미 모호**: "수용(ack) ≤5초" ≠ "실제 정지" — graceful stop(롤백) vs hard-kill 구분 필요 (렌즈1·3).

## 개선안 (정의·KPI 기존→제안)

**정의**
- 기존: "Agent가 허용 범위 내에서만 동작."
- 제안: "Agent는 **최소권한·도구 allowlist** 내에서만 동작하고, 고위험 액션은 HITL 승인을 거치며, 중단 명령과 runaway cap에 즉시 응한다."

**KPI**
| # | 기존 | 제안 | 비고 |
|---|---|---|---|
| ① | 중단 수용 ≤ 5초 | **중단 ack ≤ 5초 AND graceful 정지+롤백 완료 ≤ ◯초** | "수용=ack" 정의, 정지완료 별도. PoC-C1 |
| ② | (QAS에만 있던) | **허용범위 외 액션 통과 = 0** (적대적 eval N건 기준) | QA 파일에 정식 편입(C2 해소). PoC-C2 |
| ③ | — | **Runaway cap 작동 100%**: max iteration/token/wall-clock 도달 시 자동 중단 | agentic 필수 축(C4). PoC-C2 |
| ④ | — | **고위험 액션 HITL 게이트 통과율 100%, 우회 0건** | NQA-A와 공유. 권한 모델 명문화 |

## 근거 (레퍼런스)
- **zero-trust 최소권한 + 고위험 HITL + 가드레일**: 자율 에이전트는 도구 allowlist·고위험 액션 사람 승인·별도 가드레일이 정석 — Anthropic *Building Effective Agents* / 신뢰 에이전트 프레임워크(human control 원칙). (§4 제어·안전) — https://www.anthropic.com/engineering/building-effective-agents , https://www.anthropic.com/news/our-framework-for-developing-safe-and-trustworthy-agents
- **위반율은 적대적 eval로**: "0건"은 red-team eval 세트 통과율로만 검증 가능. (§4 제어·안전 PoC 수단)
- **협조적 취소 + timeout/retry cap**: cancel token 폴링 + 불응 시 hard-kill, activity timeout으로 runaway 차단 — Temporal cancellation/timeout. (§4·렌즈3) — https://docs.temporal.io/temporal
- ⚠️ 차단율·cap 임계는 우리 eval로 확정.

## PoC 증명법

### PoC-C1: 중단 latency와 graceful 정지를 분리 측정한다
- **가설**: "중단 ack ≤ 5초, graceful 정지+롤백 ≤ ◯초가 측정 가능하다."
- **지표**: cancel 신호→ack 시간 p95(≤5초), ack→실제 정지·롤백 완료 시간 p95(≤◯초), hard-kill 폴백 발동률.
- **셋업**: 장기 activity에 cancel token 폴링 지점 삽입 + 협조적 취소 + 불응 시 pod hard-kill 폴백.
- **절차**: ① 실행 중 cancel 발행 → ② ack·정지완료 timestamp 수집 → ③ 폴링 지점 밀도별 정지시간 비교.
- **합격(Exit)**: ack ≤5초 p95, 정지완료가 임계 내, 불응 시 hard-kill로 상한 보장.
- **규모/기간**: 단계별 long-running activity 각 N건, 약 1일.
- **리스크/한계(silent cap)**: deploy 중 취소의 부분 롤백 정합성(외부 시스템 상태)은 통합 환경 필요 — 여기선 내부 상태만.

### PoC-C2: 허용범위 위반 통과 0 + runaway cap을 적대적 eval로 증명한다 (eval 하네스)
- **가설**: "적대적 세트에서 허용범위 외 액션 통과 0, runaway cap 100% 작동."
- **지표**: 위반 통과율(=0 목표), injection으로 권한상승 유도 차단율, cap 도달 시 자동 중단율(=100%).
- **셋업**: red-team 프롬프트/시나리오 세트(권한 외 도구 호출·무한루프 유도·injection) + admission gate(실행 전 권한 체크) + cap 설정.
- **절차**: ① 적대적 세트 실행 → ② 차단/통과 라벨 → ③ cap 시나리오로 자동 중단 확인.
- **합격(Exit)**: 위반 통과 0, cap 100%, injection 차단율 ≥ 목표.
- **규모/기간**: 적대적 앵커 50~100건, 약 2~3일.
- **리스크/한계**: 적대적 세트 커버리지가 곧 신뢰도 상한(미상상 공격 미검출) — 세트 출처·범위를 log. 이 PoC는 NQA-A(Security) PoC와 공유·확장됨.

## DP·발표 영향
- **DP 연결**: DP-0002(1안 단일 제어지점 HITL)·DP-0003(1안 사전 권한 게이트)가 ④에, runner의 협조적 취소·timeout cap이 ①③에 실현. **DP-0002/0003이 runaway cap·graceful 정지를 명시 안 함** → 보강 권고.
- **NQA-A 경계(C5)**: Controllability ⊂ Security. 제어성(중단·권한 게이트)은 안전의 수단, NQA-A는 비밀관리·공급망·injection·감사를 포괄 → 양쪽 본문에 경계 cross-link. PoC-C2는 NQA-A PoC로 확장.
- **발표**: "사람 없이"의 안전장치 = steerability + runaway cap → 자율 신뢰 핵심 서사.
