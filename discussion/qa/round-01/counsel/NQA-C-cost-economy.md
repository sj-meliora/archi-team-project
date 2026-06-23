# Counsel: NQA-C Cost-economy (신규 QA 권고)

> refs-review: round-01/review/_new-qa-candidates.md (NQA-C) · report.md(C5)
> seats: 발의 Seat 2(수석 아키텍트) · 합의 consensus (Seat 1 토큰 경제, Seat 3 compute 비용 흡수)
> stance: 신설 — 채택 권장 (Med; QA-05 top-line·QA-01 활용률 흡수)

## Reviewer 지적 요약
- 비즈니스 ROI top-line = `완료 모델당 비용`인데 QA-05(Efficiency)는 하위 tactic(요청당 토큰)만 봄. 요청당 토큰을 줄여도 요청 수·재시도가 폭발하면 모델당 비용은 오를 수 있음.
- 발표에서 "수작업 대비 N% 비용 절감"이 가장 강력한 설득 포인트.

## 개선안 (정의·KPI)

**정의 (신설)**: 모델 1건 완료에 드는 총비용(토큰+compute)을 수작업/기존 대비 절감한다.

**KPI**
| # | 제안 | 비고 |
|---|---|---|
| ① | **완료 모델당 비용($) ≤ ◯** | top-line. QA-05에서 승격. PoC-N-C1 |
| ② | **수작업/기존 대비 비용 절감률 ≥ ◯%** | 발표 설득 핵심 |
| ③ | **비용 분해: 토큰비 / compute / 재시도 오버헤드** | QA-01 "활용률"·QA-05 worker 가동률 흡수 |

## 근거 (레퍼런스)
- **$/task(raw token 아님) + 캐시 분리**: 비용 top-line은 $/task이며, prompt caching(cache read ≈ 정상가 10%)을 분리 집계해 절감을 정확히 반영 — Anthropic prompt caching·pricing. (§4 비용·효율) — https://platform.claude.com/docs/en/build-with-claude/prompt-caching , https://platform.claude.com/docs/en/about-claude/pricing
- **재시도 오버헤드 분리**: 재시도가 비용을 잠식하므로 분해 집계 — SRE 비용 관측 관행.
- ⚠️ $/모델·절감률 ◯%는 우리 측정으로 확정 — 레퍼런스 수치 복제 아님.

## PoC 증명법

### PoC-N-C1: 완료 모델당 비용을 분해 집계하고 baseline 대비 절감을 측정한다 (비용 A/B + 계측)
- **가설**: "모델 1건 완료의 총비용을 토큰/compute/재시도로 분해 집계하고, 수작업 baseline 대비 절감률 ◯%를 보일 수 있다."
- **지표**: $/완료모델(토큰비+compute+재시도), 절감률(vs 수작업/기존 baseline), prompt caching On/Off 절감폭, 재시도 오버헤드 비율.
- **셋업**: OTel `gen_ai.usage.*` 토큰 계측(QA-04) + compute 단가표 + 수작업 baseline 비용 추정 + prompt caching A/B.
- **절차**: ① 모델 N건 E2E 완료 시 토큰/compute/재시도 비용 분해 집계 → ② 수작업 baseline과 비교 → ③ caching On/Off로 절감 기여 분리.
- **합격(Exit)**: $/완료모델이 분해 집계되고 baseline 대비 절감률 산출, 재시도 오버헤드 가시화.
- **규모/기간**: 모델 수십 건, 약 2일(QA-04·QA-05 계측 위에 얹음).
- **리스크/한계(silent cap)**: 수작업 baseline 비용은 추정 의존(인건비·시간 가정에 민감) → 가정 명시 log. compute 단가는 실제 인프라(자가호스팅 vs 클라우드)에 따라 변동.

## DP·발표 영향
- **DP 연결**: DP-0001(작업별 최적 agent = 비용 기준 라우팅)·DP-0004(scale-to-zero로 과프로비저닝 0 = compute 비용↓)·DP-0005(캐시로 토큰비↓)가 ①③에 기여.
- **번호/서사(C5)**: 채택 시 **QA-05 top-line + QA-01 "자원 활용률" KPI를 NQA-C로 이동** → QA-01은 throughput, QA-05는 per-request·캐싱으로 정리됨. 발표: "수작업 대비 N% 절감"이 ROI 핵심 메시지.
- **우선순위**: Med(Security·Correctness보다 후순위이나 비즈니스 설득에 필수).
