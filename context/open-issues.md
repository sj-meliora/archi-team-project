# 정합성 / 미해결 결정 트래커 (Open Issues)

> category: meta | updated: 2026-06-19
> 자료 변동으로 생긴 불일치 + 미확정 사항. 확정되면 해당 개념 파일에 반영하고 여기서 닫는다([x]).

## [ ] OI-1 QA 번호 불일치
- 슬라이드 11/13/25~28이 같은 QA01~04에 서로 다른 속성을 부여.
- **결정**: canonical = 팀 합의 우선순위 기준 (2026-06-23 재정렬, QA·QAS 2자리). glossary.md에 원본 매핑 보존.
- **2026-06-23 재정렬**: QA-01 Scalability / 02 Availability / 03 Controllability / 04 Observability / 05 Efficiency (이전 1위 Efficiency→5위). 06~10 불변.
- **남은 작업**: 발표 슬라이드 전체에서 canonical(2자리) 번호로 통일. FR/C/DP도 2자리 전환 예정.

## [ ] OI-2 DP-0001 1안 '단점' 칸 표기 오류
- Per-Node Agent(1안) 단점에 2안의 장점이 복붙됨.
- **남은 작업**: 1안의 실제 단점으로 교체 (예: [Scalability] 고정 배치라 확장 경직 / [Efficiency] 노드별 전용 Agent 유휴 비용).

## [ ] OI-3 DP-0004 대안 라벨 오류
- 헤더가 DP-0002 라벨(Hierarchical/Decentralized) 복붙. 실제 내용은 "타입별 서버 풀 vs 노드당 Workflow 인스턴스".
- **결정**: 내용 기준 라벨로 정리 (DP-0004 파일에 반영 완료 시 체크).

## [ ] OI-4 FR-0005 범위 미확정
- "시스템 운영·제어"의 Agent 특화 여부·범위 미정.
- **남은 작업**: PM과 범위 확정.

## [ ] OI-5 Domain Diagram 2개 버전 공존
- 슬라이드 15·16 vs 35·36에서 컴포넌트 배치(JIRA Manager 위치 등) 상이.
- **남은 작업**: 최신본 확정 후 artifacts/domain-diagram에 단일화.

## [ ] OI-6 As-Is/To-Be 시나리오 2개 버전
- 슬라이드 9 vs 34: Compiler 시작 vs Quantizer 시작으로 서술 상이.
- **남은 작업**: 기준 시나리오 확정.
