# QAS-A 자율 에이전트 보안·안전 시나리오

> category: QAS | refines: NQA-A (Security/Safety) | source: discussion/qa/round-01 (신규) | updated: 2026-06-24
> ISO/IEC 25010:2023: Security (+ Safety)

## 6-Part Quality Attribute Scenario
| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 외부 공격자(prompt injection) / 내부 오작동 에이전트 / 운영자 |
| **자극 (Stimulus)** | injection으로 권한 외 액션·악성 config 배포 유도 / 자격증명 탈취 시도 / 변조된 artifact 주입 / 고위험 액션(deploy·delete·credential) 시도 |
| **대상 (Artifact)** | Agent / Permission Manager / 공급망(artifact 서명 검증) / secrets store / 감사 trace |
| **환경 (Environment)** | 자율 실행 중 (배포 권한·자격증명 보유) |
| **응답 (Response)** | 권한 외 액션을 실행 전 차단 / 고위험 액션은 HITL 승인 게이트 / artifact 서명·무결성 검증 / secrets 마스킹 / injection 방어(가드레일) |
| **응답 측정 (Measure)** | 권한 상승·범위 외 배포 **0건**(적대적 eval), HITL 통과율 **100%·우회 0**, artifact 서명·무결성 **100%**, prompt-injection 차단율 **≥95%**, secrets 노출(로그·trace) **0건** |

## 비고
- ISO/IEC 25010:2023 Security(기밀성·무결성·부인방지·책임추적성·인증성) + Safety. 상세 매핑은 NQA-A 본문.
- 설계 연결: DP-0003(1안 사전 권한 게이트 + 2안 격리), DP-0002(1안 HITL gate). 검증은 QA-03 PoC-C2와 공유 red-team 하네스([발표 서사]).
- 경계: Controllability(QA-03) ⊂ Security(NQA-A). 감사 trace 100%는 QA-04와 교차.
- 수치(0건·100%·95%)는 측정가능 KPI 예시값 — injection 차단율 등은 red-team 세트로 확정.
