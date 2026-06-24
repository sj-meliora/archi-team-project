# QAS-03 Agent 제어 시나리오

> category: QAS | refines: QA-03 (Controllability) | source: pptx p.28 | updated: 2026-06-24 (round-02: ② 위반0 → ②-1 차단율/②-2 acceptance 2단 분리)

## 6-Part Quality Attribute Scenario
| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 운영자(플랫폼 개발자) 또는 정책 엔진 |
| **자극 (Stimulus)** | 중단(stop) 명령 발행 / Agent가 허용 범위 밖 액션 시도 / runaway(무한 루프·cap 초과) 발생 / 고위험 액션(배포·삭제) 시도 |
| **대상 (Artifact)** | Agent / Permission Manager / Orchestrator(제어 지점) |
| **환경 (Environment)** | Agent 실행 중 (장기 activity 진행 포함) |
| **응답 (Response)** | 중단 명령을 즉시 접수(ack) 후 안전 정지(graceful stop)·롤백 / 주입한 hard-invalid(권한외·스키마 위반) 액션을 **실행 전** admission 게이트가 차단 / 적대적 풀세트 위반 통과 0 / runaway cap 도달 시 자동 중단 / 고위험 액션은 HITL 승인 게이트를 거침 |
| **응답 측정 (Measure)** | 중단 ack **≤ 5초** AND 안전 정지(graceful stop)+롤백 완료 **≤ 30초**; **②-1 권한외 액션 차단율** — 주입 hard-invalid → 차단/안전정지 latency 분포(룰 체커·현 조건 산출); **②-2 적대적 위반 통과 0건**(OWASP LLM Top-10 매핑 풀세트·세트 커버리지가 신뢰 상한 — QA-06 채택 OI-8 의존 open-issue); runaway cap 작동 **100%**; 고위험 액션 HITL 통과율 **100%·우회 0건** |

## 비고
- 설계 연결: DP-0002(1안 단일 제어 지점에서 정책·HITL·cap 적용), DP-0003 1안(실행 전 사전 권한 체크 게이트).
- 수치(5초·30초·0건·100%)는 측정가능 KPI의 예시값 — 합격선은 실환경 측정으로 확정. 상세·근거·검증 전략은 QA-03 본문.
- 경계: 보안 전반(비밀관리·공급망·injection·감사)은 QA-06(Security)에서 — Controllability ⊂ Security.
