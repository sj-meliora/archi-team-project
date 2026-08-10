# DP 신규 Design Approach 후보 (Backlog)

> category: DP-meta | updated: 2026-06-27
> 아직 특정 DP의 정식 대안으로 확정되지 않은 아이디어 풀.
> driving QA의 tactic/pattern 카탈로그에서 발굴 → 검증되면 해당 DP 파일의 대안으로 승격.

## BL-1 → DP-01: Hierarchical + Standby Orchestrator
- **driving QA**: QA-02(Availability) — 1안의 Orchestrator SPOF(R-1) 완화 목적.
- **근거 tactic**: Redundancy(Active-Passive), Heartbeat, State resync.
- **상태**: DP-01에 3안으로 임시 반영. 페일오버 시간·일관성(QA-11) 검증 필요.

## BL-2 → DP-01: Hierarchical Federation (도메인별 Sub-Orchestrator)
- **driving QA**: QA-01(Scalability), QA-10(Performance) — 단일 Orchestrator 병목(TP-2) 완화.
- **근거 pattern**: Hierarchical control + 파이프라인 단계(Converter/Optimizer/Compiler)별 Sub-Orchestrator.
- **상태**: 미검토. 제어 일관성 유지하며 수평 확장 가능한지 평가 필요.

## BL-3 → (행선지 미정): 에이전트 배치/풀 (구 DP-0001 승계 후보)
- **맥락**: 2026-06-27 구 DP-0001(Per-Node vs Dynamic Agent Pool) 삭제 — 배치/풀 결정이 어느 DP에도 안 귀속(OI-12 재귀속 보류). DP-01(제어평면)·DP-02(특화·드랍)이 안 다룸.
- **driving QA**: QA-01(Scalability)·QA-05(Efficiency)·QA-09(Performance)·QA-13(Cost) — 동적 풀·warm pool·즉시 실행·비용 라우팅을 인용 중(`구 DP-0001` 표기).
- **근거 tactic/pattern**: Resource pooling, Dynamic routing, scale-to-zero(↔DP-0004 A5/A8 ephemeral과 정합).
- **후보 행선지**: (a) DP-0004가 scale-out으로 흡수 / (b) 신규 DP / (c) 비-ASR이면 드랍.
- **상태**: 보류(OI-12) — DP 디스커션에서 행선지 결정 후 승격/redirect.

## BL-4 → DP-06 후속: 조회 계층(retrieval layer) 상세 설계
- **driving QA**: QA-07(Correctness — 검색 품질→정답률), QA-05(Efficiency — 조회 토큰).
- **맥락**: 2026-08-10 DP-06 디스커션 — 인덱싱·조회 계층은 부트스트랩 전략 어느 안이든 공통 필수 인프라(변별 축 아님)로 확인 → DP-06은 채택 패턴(Cerebras Knowledge 레퍼런스: 단일 임베딩 스토어 + 하이브리드 검색 + RRF·rerank + age decay)만 전제로 깔고 상세 설계를 후속으로 분리.
- **근거 tactic/pattern**: 하이브리드 검색(전문검색: 에러 코드·플래그명 정확 일치 + 임베딩: 패러프레이즈), 희소어 가중치, RRF 융합, cross-encoder rerank, age decay(staleness 랭킹 완화).
- **미검증 trade-off / 리스크**: 우리 규모에서 풀 하이브리드가 과한지(초기엔 구조적 인덱스+전문검색으로 충분할 수 있음 — 시작 전략(DP-06)의 시딩 규모에 종속), 인덱스 유지 비용(QA-13).
- **상태**: 검토 대기 — DP-06(시작 전략) 확정 후 승격 여부 결정.

<!-- 템플릿
## BL-N → DP-XXXX: <아이디어 이름>
- driving QA:
- 근거 tactic/pattern:
- 의도 / 기대 효과:
- 미검증 trade-off / 리스크:
- 상태: 미검토 / 검토중 / 승격(DP-XXXX N안) / 폐기
-->
