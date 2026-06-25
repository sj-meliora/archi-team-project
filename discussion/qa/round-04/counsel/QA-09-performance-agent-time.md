# Counsel: QA-09 Performance — Agent 수행 시간 (per-node)

> refs-review: round-04/review/QA-09-performance-agent-time.md
> seats: 발의 Seat 3 (runner 오버헤드·tool 시간) · 합의 consensus (Seat 1 apples·Seat 2 표현 정리)
> stance: **조건부 채택 — InfEngine 인용 수치 교체(8.6~22.7×→21× 단일+가공 정정) + 하한 표현 통일 + tool 시간 silent cap**

## Reviewer 지적 요약
speedup 3배→1× 하한 보정은 협업형(METR 0.84×)·완전자율(InfEngine) 대비를 잘 잡음. red-team: (1) **★★★ 8배가 InfEngine 단일 의존 + 미래형 arXiv ID(2602.18985)**(권고 2·Med·C4), (2) InfEngine이 우리 SDK 노드와 apples-to-apples인지 불명(C1), (3) "3배→1× 하향"과 "3배 ★★☆ 유지"가 공존해 독해 혼란(권고 1·Low), (4) first-pass 게이트가 QA-07 golden(미구축) 의존(권고 4·Med).

## 개선안 (정의·KPI 기존→제안)

### (A) InfEngine 인용 수치 교체 — **C4 핵심(가공 수치 발견)**
- **★ 노트 기존 → 제안**:
  - 기존: `InfEngine assistant-type 8.6~22.7×` 단일 의존.
  - 제안: **`InfEngine(arXiv 2602.18985) 실제 보고치 = 21× faster than manual expert, 92.7% pass rate`로 정정**(아래 §근거 — 8.6~22.7× 범위는 원문에 없음·가공 의심). 그리고 **도메인 = 적외선 복사 컴퓨팅(infrared radiation computing)**임을 apples-to-apples silent cap으로 명시(우리 SDK 빌드 노드와 작업 종류 다름). ★★★ ≥8배 경계 **방향은 유지**(21×가 8× 위라 완전자율 사례가 ★★★ 대역 지지)하되, "단일 사례·도메인 불일치"를 노트화.

### (B) 하한 표현 단일화 (권고 1)
- 기존: "3배→1× 하향 보정"과 "3배는 ★★☆ 유지"가 한 문단 공존.
- 제안: **"합격 하한 = >1배(★☆☆ 진입 — 사람보다 빠름), 구 하한 3배는 ★★☆ 경계로 상향"** 단일 표현으로 변경이력·노트 통일(정합성 결함 아니나 독해 명료화).

### (C) tool 시간 처리 명시 (권고 3)
- ## 검증 전략에 `외부 컴파일러 tool 실행 시간을 speedup 분모(수동 baseline)·분자(agent) 동일 기준으로 처리 — 별도 버킷 계측해 희석/과장 방지`.

### (D) first-pass 게이트 의존 (권고 4·C 교차)
- speedup 집계가 QA-07 golden(미구축·[발표 서사]) 의존 → QA-07 닫힘과 묶음(OI-7/OI-8 트래킹). 본 라운드 신규 조치 없음(정상 트래킹), 노트에 의존 명시 유지.

## 근거 (레퍼런스 + 검증 결과)

**C4 웹 검증 — InfEngine 실재하나 인용 수치 오류(중대)**:
- arXiv **2602.18985 실재** = ["InfEngine: A Self-Verifying and Self-Optimizing Intelligent Engine for Infrared Radiation Computing"](https://arxiv.org/abs/2602.18985). 2602=2026-02로 **현재(2026-06) 기준 유효 과거 ID**.
- **그러나 인용 수치가 틀림**: 원문 abstract = **"92.7% pass rate, 21× faster than manual expert effort"**. round-03이 쓴 `8.6~22.7×` 범위는 **원문에 없음**(가공/오기 의심). 도메인도 **적외선 복사 컴퓨팅**이지 "assistant-type"가 아님. → **인용 정정 필수**. ★★★ ≥8배 방향은 21× 단일치가 지지하나, **단일 사례·도메인 불일치**라 독립 출처 보강 권장(METR 0.84× 협업형과의 대비는 유효 — [METR RCT](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/), 19% 감속=0.84×, 2025-07 확인).
- 보강 출처: METR 2026 업데이트(self-reported 2x 향상이나 selection effect 경고 — [METR 2026](https://metr.org/blog/2026-02-24-uplift-update/))는 "협업형은 자율형보다 느림"의 대비 강화용.

## PoC 증명법

### PoC-09: speedup 배수 + tool 시간 분리 (부하시험 + 비용 A/B 아키타입)
- 가설: "노드타입별 speedup ★ 경계(1/3/8배)가 측정 가능하고, tool 시간을 분리하면 배수가 희석되지 않는다."
- 지표: speedup = 수동 baseline 중앙값 / agent 중앙값(노드타입별), runner 오버헤드 비율, 외부 tool 시간 버킷. 합격선: first-pass 성공분만·커버리지 ≥80% 하에서 ★ 경계 변별.
- 셋업: mock 파이프라인(IR/Quant/Compile 노드별 서비스시간 분포) + warm pool On/Off A/B, baseline 표본(수동 중앙값 가정).
- 절차: ① first-pass 성공 노드만 필터(QA-07 게이트 대리=룰 체커). ② speedup 산출, tool 시간 별도 버킷. ③ warm pool On/Off로 runner 오버헤드 레버 노출.
- 합격 기준: tool 시간 분리 전/후 speedup 차이를 보여 "희석/과장" 정량화 + ★ 경계 단조.
- 규모/기간: 노드 3타입 × N task, ~0.5일(mock).
- 리스크/한계(silent cap): baseline 숙련도 편차가 speedup 좌우. QA-07 golden 미구축 시 first-pass 게이트는 룰 체커 대리.

## DP·발표 영향
- DP-0001(즉시 실행)·DP-0004(cold-start) 변별을 ★가 보상. DP-0001 1안이 latency만 보고 품질 게이트 무연결(OI-7).
- 발표 영향: InfEngine 인용 정정은 **데이터 정직성** 사례로 활용(미래형 ID 검증 → 수치 오류 발견 → 정정).
