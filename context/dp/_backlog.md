# DP 신규 Design Approach 후보 (Backlog)

> category: DP-meta | updated: 2026-06-20
> 아직 특정 DP의 정식 대안으로 확정되지 않은 아이디어 풀.
> driving QA의 tactic/pattern 카탈로그에서 발굴 → 검증되면 해당 DP 파일의 대안으로 승격.

## BL-1 → DP-0002: Hierarchical + Standby Orchestrator
- **driving QA**: QA-0003(Availability) — 1안의 Orchestrator SPOF(R-1) 완화 목적.
- **근거 tactic**: Redundancy(Active-Passive), Heartbeat, State resync.
- **상태**: DP-0002에 3안으로 임시 반영. 페일오버 시간·일관성(QA-0009) 검증 필요.

## BL-2 → DP-0002: Hierarchical Federation (도메인별 Sub-Orchestrator)
- **driving QA**: QA-0002(Scalability), QA-0008(Performance) — 단일 Orchestrator 병목(TP-2) 완화.
- **근거 pattern**: Hierarchical control + 파이프라인 단계(Converter/Optimizer/Compiler)별 Sub-Orchestrator.
- **상태**: 미검토. 제어 일관성 유지하며 수평 확장 가능한지 평가 필요.

<!-- 템플릿
## BL-N → DP-XXXX: <아이디어 이름>
- driving QA:
- 근거 tactic/pattern:
- 의도 / 기대 효과:
- 미검증 trade-off / 리스크:
- 상태: 미검토 / 검토중 / 승격(DP-XXXX N안) / 폐기
-->
