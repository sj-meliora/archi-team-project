# Counsel: QA-05 Efficiency

> refs-review: round-04/review/QA-05-efficiency.md
> seats: 발의 Seat 1 (토큰 경제·caching) · 합의 consensus
> stance: **채택 권장 — tier 정식화 모범, 적중률 조건 silent cap + tier margin 명시 + 단일 출처 다양화**

## Reviewer 지적 요약
신규/캐시 분리 tier(4/6/8k)는 캐싱 역페널티 회피 모범. 잔여(Low): (1) **★ 전 경계가 Requesty 단일 출처(코딩 에이전트)** — 우리는 SDK 빌드 노드(C1), (2) **적중률 ≥85% 조건이 신규 토큰(cache-miss) 변별 폭을 스스로 좁힘**(권고 1), (3) 4k/6k/8k 2k 등간격 margin 미명시(권고 3·C2).

## 개선안 (정의·KPI 기존→제안)
### (A) 적중률 조건 silent cap (권고 1)
- ★ 노트: `조건: 적중률 ≥85% 고정`이 신규 토큰(=적중률의 직접 함수) 변별 폭을 스스로 좁힘을 명시 — `공정 비교(동일 적중률) 의도이나 ★가 잴 수 있는 건 출력 토큰·cache-miss 잔차뿐 — 공정성↔변별력 trade-off`.
### (B) tier margin 명시 (권고 3·C2)
- ★ 노트: `4k/6k/8k 2k 등간격 = 작업 난이도(단일/일반/복합) 매핑. 4k=Requesty 92% 적중 대역, 6k=일반, 8k=대입력 복합 margin`. 등간격이 자의가 아니라 난이도 tier 매핑임을 근거화.
### (C) 단일 출처 다양화 (권고 2·C1)
- `Requesty는 코딩 에이전트(86~92% 적중)이고 우리는 SDK 빌드 노드(빌드로그·config diff) — 워크로드 유사성 가정. 빌드 파이프라인 도메인 적중률은 우리 PoC로 확정` silent cap.

## 근거 (레퍼런스 + 검증 결과)
**C4 — caching 표준**: cache read ≈ base 10%(≈90%↓, 지연 85%↓)는 [Anthropic prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)(Council.md §4 비용). Requesty "Coding Agent Economy"(92% cache·86% avg)는 본 검증에서 1차 대조 미완 → "확인 불가, 코딩 도메인 데이터 — 우리 도메인 재측정"으로 명시($/task 아닌 신규 토큰 분리 측정은 표준).

## PoC 증명법
### PoC-05: 신규 토큰 tier + 적중률 sweep (비용 A/B 아키타입)
- 가설: "신규 토큰 ★ 경계(4/6/8k)가 caching A/B로 측정되고 적중률 조건이 변별을 좁히는 정도를 정량화한다."
- 지표: 작업당 신규 토큰(cache-miss+output), 캐시 적중률. 합격선: 적중률 ≥85% 고정 하 tier별 신규 토큰 변별.
- 셋업: caching On/Off A/B, 적중률 sweep(70~95%), warm pool affinity.
- 절차: ① On/Off로 신규 토큰·$/task 측정. ② 적중률 sweep로 신규 토큰 곡선. ③ 85% 고정 시 변별 잔차 측정.
- 합격 기준: 적중률 고정이 변별 폭을 얼마나 좁히는지 정량화(silent cap 입증) + tier 단조.
- 규모/기간: A/B + sweep 3점, ~0.5일.
- 리스크/한계(silent cap): mock 적중률이 실운영보다 낮으면 ★ 경계 보수적.

## DP·발표 영향
- DP-0001 2안 라우팅이 토큰 vs 비용 기준인지(OI-7), QA-13 top-line 이양 정합.
