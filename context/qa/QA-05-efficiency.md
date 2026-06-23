# QA-05 Efficiency — Agent 토큰 사용량

> category: QA | importance: H | difficulty: H | source: pptx p.13/p.25 | updated: 2026-06-19
> refines-scenario: QAS-05 | related-dp: DP-0001, DP-0003

## 정의 / Refinement
Agent 토큰 사용량 최소화 — 단일 요청 총 토큰.

## 측정 (KPI)
단일 요청 총 토큰 ≤ 8k
- ★★★ ≤ 4k (단일 작업 최적화)
- ★★☆ 4k~6k (일반 Workflow)
- ★☆☆ 6k~8k (복합 추론 허용)
