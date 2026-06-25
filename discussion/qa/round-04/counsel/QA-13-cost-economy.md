# Counsel: QA-13 Cost-economy — 완료 모델당 비용

> refs-review: round-04/review/QA-13-cost-economy.md
> seats: 발의 Seat 2 (규칙2·baseline) · 합의 consensus (Seat 1 도메인 다양화·Seat 3 재시도 분해)
> stance: **조건부 채택 — baseline 추정 silent cap + $/모델 분해 보조 별점 축 + ★★★ 75% 도메인 무관 분포로 재근거 + OI 등록 확인**

## Reviewer 지적 요약
절감률 70→50% 재배치(규칙2 핵심)+$/모델 게이트 분리 정확. red-team: (1) **★ 전 경계가 수작업 baseline 추정([발표 서사])에 좌우**(권고 1·Med·C3), (2) **★★★ 75%가 법무 단일 사례(Medium 블로그) 의존 + SDK 빌드와 도메인 다름**(권고 2·Med·C1), (3) ★★☆ 폭(15%p)>★☆☆(10%p) margin 근거 약함(권고 3·Low·C2), (4) 변경이력 "OI 등록 범위 외" — QA-13만 미등록 의심(권고 4·Low).

## 개선안 (정의·KPI 기존→제안)

### (A) baseline 추정 silent cap (C3 — main 축 추정 의존)
- **★ 노트 기존 → 제안**: 절감률 절대값이 수작업 baseline 인건비·시간 가정의 함수임을 명시 — `★ 전 경계는 수작업 baseline([발표 서사] 추정값) 정의에 좌우. baseline을 보수적으로 잡으면 절감률↑(★↑). 절대 경계가 아니라 동일 baseline 고정 하 두 설계 비교용.`

### (B) $/모델 3축 분해 보조 별점 축 (C3 보강)
- **기존**: 절감률이 유일 main 별점 축(추정 baseline 의존).
- **제안**: **$/모델을 토큰비 / compute / 재시도 3축 분해하고, 재시도 오버헤드 비율을 보조 별점 축 병기** — 재시도율은 baseline 추정 없이 파이프라인 로그에서 산출 가능. 보조 ★: `재시도 오버헤드 ≤10% = ★★★ / 10~25% = ★★☆ / 25~40% = ★☆☆`. ATAM 비교 시 절감률이 추정이면 재시도 오버헤드로 변별(QA-11 ②-1·QA-07 rework와 동형 이중 축).

### (C) ★★★ 75% 재근거 + margin 차등 명시 (권고 2·3)
- 기존: 법무 75% 단일 사례. → 제안: **도메인 무관 자동화 절감 분포(Forrester ~30%·일반 자동화 30~75%)에서 ★★★ 위치를 재확인**하고 법무는 "상위 사례 1점"으로 강등(단일 의존 탈피). ★★☆ 15%p>★☆☆ 10%p 차등은 `자동화 절감 분포가 50~75% 구간에 두텁고 75%+가 희소 → 상단 폭을 넓힘`으로 근거화(margin 차등의 분포 근거).

### (D) OI 등록 확인 (권고 4)
- 변경이력 138행 "open-issues.md 정합 점검(범위 외)" → 하한 보정(70→50%)이 **OI-9에 실제 등록됐는지 Applier가 확인**(QA-08~10은 "등록 대상" 명시, QA-13만 미등록 의심). 미등록이면 등록.

## 근거 (레퍼런스 + 검증 결과)

**C4 — 절감률 출처**: 법무 75% Medium 블로그·Forrester ~30%·Requesty는 본 검증에서 1차 대조 미완 → "확인 불가, 분포 정당화용"으로 명시(수치 복제 아님). caching(cache read ≈ base 10%, ≈90%↓)은 [Anthropic prompt caching](https://platform.claude.com/docs/en/build-with-claude/prompt-caching) 표준(Council.md §4 비용). 자동화 절감 30~75% 대역은 50% 하한(ROI 진입선)·75% 상한(상위 사례)을 지지.

## PoC 증명법

### PoC-13: $/모델 분해 + 절감률 민감도 (비용 A/B 아키타입)
- 가설: "절감률은 baseline 추정 의존이나 $/모델 3축 분해·재시도 오버헤드는 실측되어 보조 축이 된다."
- 지표: $/모델(토큰/compute/재시도 분해), 절감률(baseline sweep), 재시도 오버헤드 비율. 합격선: ★ 경계 변별 + 재시도 폭증 시 절감률 붕괴 포착.
- 셋업: caching On/Off A/B(QA-05 공유) + compute 단가 + baseline 추정 sweep(보수/중립/낙관).
- 절차: ① $/모델 3축 분해. ② baseline sweep으로 절감률 민감도 곡선. ③ 재시도율↑ 주입 시 절감률 변화.
- 합격 기준: 재시도 오버헤드가 두 설계에서 변별되고, 절감률이 baseline 가정에 ±X%p 민감함을 정량화(silent cap 입증).
- 규모/기간: caching A/B + sweep 3점, ~0.5일.
- 리스크/한계(silent cap): 절감률 절대값은 baseline 추정([발표 서사]) 좌우. 법무 사례는 도메인 불일치.

## DP·발표 영향
- DP-0001(비용 기준 라우팅)·DP-0004(scale-to-zero)·DP-0005(캐시) 귀속(OI-7). QA-05 per-request altitude와 정렬.
- 발표: "절감률 절대값 대신 재시도 오버헤드 실측 보조 축으로 ATAM 변별력 확보" = C3 정면 응답.
