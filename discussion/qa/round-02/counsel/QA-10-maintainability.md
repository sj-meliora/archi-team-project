# Counsel: QA-10 Maintainability — 모듈 교체 용이성 (round-02)

> refs-review: [round-02/review/QA-10-maintainability.md](../review/QA-10-maintainability.md)
> 직전 counsel: [round-01/counsel/QA-10-maintainability.md](../../round-01/counsel/QA-10-maintainability.md) (채택 권장 — CIS p95 + model 교체축)
> seats: 발의 **Seat 2**(수석 아키텍트) · 합의 **consensus**
> stance: **닫힘 확인(건강 유지·강화) — 잔여 = DP-0004/0005 교체내성 역검토·컴포넌트 경계 정의 의존**

## Reviewer 지적 요약
- round-02 verdict: **Sound ○ / KPI ○ · Low** (건강 유지·강화). 평균 CIS(tail 은닉) → `CIS p95 ≤3 + 컴포넌트 경계 정의` + prompt/모델 교체·온보딩 축 추가. agentic 유지보수 지배 비용(prompt·tool-def·모델 교체) 반영.
- 잔여(비-verdict): DP-0004/0005 교체내성(버저닝·계약 분리·blue-green) 역검토(OI-7), 컴포넌트 경계 정의 의존(CIS 측정 전제).

## 개선안 (정의·KPI 기존→제안)
KPI 닫힘 — 새 KPI 없음. model-agnostic 무중단 교체 정의 유지. CIS p95가 의미를 가지려면 **컴포넌트 경계 정의가 선결**임을 KPI 옆에 명문화(측정 전제, Low).
> ⚠️ `CIS p95 ≤3·수정 ≤2·온보딩 ≤1일`은 "측정 가능 KPI의 모양" 예시값.

## 근거 (레퍼런스)
§4 **지연(percentile)** 패턴 차용(평균 금지 → p95로 tail 은닉 방지) + 교체내성 표준.
- **p95 CIS(평균 금지)** — 평균 CIS는 소수 고결합 컴포넌트의 tail을 숨김. https://oneuptime.com/blog/post/2025-09-15-p50-vs-p95-vs-p99-latency-percentiles/view
- **model-agnostic 교체 = 계약 분리·버저닝·blue-green** — 모델/prompt 교체 무중단의 설계 tactic(workflow 버저닝). https://docs.temporal.io/temporal

## PoC 증명법
### PoC-M1(R1 유지): CIS p95 + 모델교체 무중단 (측정)
- **가설**: "컴포넌트 경계 정의 하에 CIS p95가 측정 가능하며, 모델/prompt 교체가 무중단으로 수행된다."
- **지표+합격선**: CIS p95 ≤◯ · SDK 변경 시 prompt/tool-def 수정 ≤◯ · 모델 교체 무중단 100% · 신규 모델 온보딩 ≤◯.
- **셋업**: 컴포넌트 경계 정의 + 의존 그래프 분석 + 모델 교체 시나리오(blue-green).
- **절차**: 변경 영향 컴포넌트 수(CIS) p95 측정 → 모델 교체 무중단 검증 → 온보딩 시간.
- **합격(Exit)**: CIS p95 ≤목표 ∧ 교체 무중단 100% ∧ 경계 정의로 재현 가능.
- **규모/기간**: 약 1~2일.
- **silent cap**: CIS 측정은 컴포넌트 경계 정의 품질에 의존(경계 모호하면 CIS 무의미). 무중단 교체는 통합 환경 필요.

## DP·발표 영향
- **DP 위임 (OI-7)**: DP-0004/0005에 **workflow 버저닝·계약 분리·blue-green** tactic 역검토(교체내성). DP 디스커션 위임.
- **번호/서사**: 모델 교체 빈번한 on-device SDK 맥락에서 유지보수성 = 장기 운영 비용 메시지. importance 변동 없음(Low 건강 유지).
