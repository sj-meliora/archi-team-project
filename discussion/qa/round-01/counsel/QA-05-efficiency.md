# Counsel: QA-05 Efficiency — Agent 토큰 사용량

> refs-review: round-01/review/QA-05-efficiency.md · report.md(C4·C5)
> seats: 발의 Seat 2(수석 아키텍트) · 합의 consensus (Seat 1 캐싱 분리, Seat 3 compute 효율 통합)
> stance: 채택 권장 (top-line은 NQA-C로 승격, per-request는 캐싱 반영 보완)

## Reviewer 지적 요약
- **Altitude 낮음**: 비즈니스 효율 top-line은 `완료 모델당 비용`인데 KPI는 요청당 토큰(하위 tactic)뿐 (렌즈2).
- **`총 토큰 ≤ 8k` 비현실**: 빌드로그·에러트레이스·config diff가 쉽게 8k 초과. "총 토큰" 입력+출력 여부 미명시 (렌즈1·2).
- **raw 토큰이 prompt caching을 역페널티**: 캐시로 비용 1/10인 전략이 raw 토큰 지표에선 "나쁨" → 캐시/신규 토큰 분리 필요 (렌즈1).
- runner compute 효율(worker 가동률 등)을 토큰비와 합쳐 task/모델당 총비용으로 통합 (렌즈3).

## 개선안 (정의·KPI 기존→제안)

**정의**
- 기존: "Agent 토큰 사용량 최소화 — 단일 요청 총 토큰."
- 제안: "모델 1건 완료의 총비용(토큰+compute)을 최소화하되, prompt caching·context 압축을 페널티 없이 측정한다." (top-line은 NQA-C로 승격, 본 QA는 per-request·캐싱 효율 축으로 재정의)

**KPI**
| # | 기존 | 제안 | 비고 |
|---|---|---|---|
| top-line | (없음) | **→ NQA-C로 승격**: 완료 모델당 비용($) | C5. 본 QA에 남기지 않고 이전 |
| ① | 단일 요청 총 토큰 ≤ 8k | **신규 토큰 ≤ ◯ + 캐시 토큰 별도 집계** (총 토큰 = 입력+출력 명시) | 캐싱 페널티 제거. ◯는 PoC-E1 |
| ② | tier 표(★ 척도) | **유지** — 작업 난이도별 차등(복합 추론 노드 캡 완화/별도 tier) | 실용적이라 보존 |
| ③ | — | **worker 가동률 / task당 compute 비용** (runner 효율) | QA-01에서 빼낸 "활용률"의 올바른 자리 |

## 근거 (레퍼런스)
- **$/task + 캐시 분리 집계**: raw token이 아닌 $/task가 옳은 효율 척도이며, prompt caching은 cache read가 정상가의 ~10%(지연도 대폭↓)라 **신규/캐시 토큰을 분리 집계**해야 캐싱 전략이 페널티 받지 않음 — Anthropic prompt caching·pricing. (§4 비용·효율) — https://platform.claude.com/docs/en/build-with-claude/prompt-caching , https://platform.claude.com/docs/en/about-claude/pricing
- **compute 효율(가동률)은 별 지표**: worker 가동률·배칭은 SRE/오토스케일 효율 영역. (§4 격리·확장)
- ⚠️ 8k·10% 등 수치는 우리 A/B로 확정 — 레퍼런스 복제 아님.

## PoC 증명법

### PoC-E1: prompt caching On/Off로 $/task·TTFT 절감을 실측한다 (비용 A/B)
- **가설**: "캐시 분리 집계 시 동일 작업의 $/task·TTFT가 유의하게 줄고, 신규 토큰 캡 ◯가 현실적이다."
- **지표**: $/task(캐시 On vs Off), 신규 토큰 vs 캐시 토큰 비율, TTFT, 복합 추론 노드의 신규 토큰 분포(캡 ◯ 산출).
- **셋업**: 대표 노드 작업(빌드로그·config diff 포함) N건 × prompt caching On/Off A/B + 토큰/비용 계측(OTel `gen_ai.usage.*`, QA-04 연계).
- **절차**: ① Off로 baseline $/task·토큰 분포 측정 → ② On으로 동일 작업 → ③ 절감폭·신규 토큰 캡 후보 산출.
- **합격(Exit)**: 캐시 On에서 $/task 유의 감소, 신규 토큰 기준 캡 ◯가 복합 노드도 수용.
- **규모/기간**: 노드타입별 10~20건(앵커 50~100), 약 1~2일.
- **리스크/한계(silent cap)**: 캐시 적중률은 워크로드 유사성에 의존 — 다양성 높은 실운영에선 절감폭이 다를 수 있음(log). compute 비용(③)은 실제 인프라 단가가 있어야 정밀.

## DP·발표 영향
- **DP 연결**: DP-0001(2안 작업별 최적 agent 선택)이 **토큰 기준인지 비용 기준인지 역검토** — 비용 기준으로 정렬 권고. DP-0005(공유 캐시 = 결과 메모이제이션)는 캐시 토큰 집계와 직결.
- **번호/서사(C5)**: top-line `$/완료모델`은 **NQA-C로 승격** → QA-05는 per-request·캐싱 효율로 좁혀짐. NQA-C 미채택 시 top-line을 QA-05에 임시 보유. 발표: "수작업 대비 N% 비용 절감"은 NQA-C로 전달.
