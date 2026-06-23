# QA 권고 round-01 — 종합 보고서 (Council / blue team)

> 이 문서 하나로 round-01 **권고의 전모**를 파악할 수 있다(공의회 최종 문서). 방법론: [`../../Council.md`](../../Council.md) · red team 결론: [`../review/report.md`](../review/report.md) · PoC 통합: [`_poc-plan.md`](_poc-plan.md)
> date: 2026-06-24 · scope: full(QA-02~10 + NQA-A/B/C 완성) · seats: (1) Agentic Workflow · (2) 수석 아키텍트 · (3) Runner 인프라 · 직전 라운드: 없음(최초 = baseline)

## TL;DR
- red team이 깐 핵심은 **KPI ✕ 3건(QA-01·07·08)·정의↔KPI 불일치·agentic 고유 리스크 과소대표·1급 QA 누락**. Council은 이 전부에 **개선안 + 레퍼런스 + PoC**를 붙여 응답했다.
- **측정불가 KPI**(C1)는 모두 **측정가능 SLI로 환원**: QA-01→scaling efficiency, QA-07→노드타입별 speedup(+품질 게이트), QA-08→E2E latency p95·throughput.
- **agentic 고유 리스크**(C4)는 rate-limit 헤드룸(QA-01·06)·runaway cap(QA-03)·prompt caching 분리집계(QA-05)·pass^k 캐시우회(QA-09)로 KPI에 편입.
- **신규 QA 3종 모두 채택 권고** — NQA-A(Security)·NQA-B(Correctness) 강력권장(우선순위 상위), NQA-C(Cost) Med. NQA-B는 QA-07·09 KPI를 닫는 **교차 의존의 허브**.
- 17개 PoC가 §4 9개 아키타입을 전부 덮으며, **공유 자산 재사용**(부하 하네스·durable 엔진·eval 세트·golden set·OTel 계측)으로 약 5~6주 추정.

## 채택 권고표

| QA | 속성 | review verdict | stance | 핵심 권고(한 줄) | PoC |
|---|---|:---:|---|---|---|
| [QA-01](QA-01-scalability.md) | Scalability | KPI ✕ / High | 채택 권장 | N·활용률 폐기 → **scaling efficiency ≥0.8** + backlog 오토스케일 + rate-limit 헤드룸 | S1·S2 |
| [QA-02](QA-02-availability.md) | Availability | KPI △ / High | 채택 권장 | MTTR 분해 → **재기동 ≤1분 AND 무손실** + 가용률 99.5% + 외부 LLM 장애 시나리오 | A1·A2 |
| [QA-03](QA-03-controllability.md) | Controllability | KPI △ / Med | 채택 권장 | QAS의 "위반 0건" 편입 + **runaway cap** + HITL + 적대적 eval 검증 | C1·C2 |
| [QA-04](QA-04-observability.md) | Observability | KPI △ / Med | 채택 권장 | "재구성 95%" → **trace 완전성**(안전 100%/일반 95%) + event-history + MTTD | O1 |
| [QA-05](QA-05-efficiency.md) | Efficiency | KPI △ / Med | 채택 권장 | top-line → NQA-C 승격, **캐시/신규 토큰 분리집계**(8k 페널티 제거) | E1 |
| [QA-06](QA-06-reliability-workflow.md) | Reliability-WF | KPI ○ / Med | 채택 권장 | 건강 — **토큰/rate-limit 쿼터 격리 KPI 추가** + QA-02 경계 명문화 | R1 |
| [QA-07](QA-07-performance-agent-time.md) | Perf-Agent | KPI ✕ / High | 채택 권장 | "최대화" 폐기 → **노드타입별 speedup** + first-pass 품질 게이트 + 커버리지 | P1·P2 |
| [QA-08](QA-08-performance-e2e.md) | Perf-E2E | KPI ✕ / High | 채택 권장 | 정의↔KPI 정렬 → **E2E latency p95·throughput 추가**, 전달 5%는 하위로 강등 | E2E1 |
| [QA-09](QA-09-reliability-agent-consistency.md) | Reliability-일관성 | KPI △ / Med | 채택 권장 | 재프레이밍 → **캐시우회 pass^k + 유효-결정률**, NQA-B와 짝 | K1 |
| [QA-10](QA-10-maintainability.md) | Maintainability | KPI ○ / Low | 채택 권장 | 평균→**CIS p95**(tail) + prompt/model 교체·온보딩 축 추가 | M1 |
| [NQA-A](NQA-A-security-safety.md) | Security/Safety | 신규 | **신설·강력권장** | red-team eval(위반 0·injection 차단) + HITL + 공급망 서명 | N-A1 |
| [NQA-B](NQA-B-correctness.md) | Correctness | 신규 | **신설·권장** | golden set + LLM-as-judge 정답률 — QA-07/09 게이트의 전제 | N-B1 |
| [NQA-C](NQA-C-cost-economy.md) | Cost-economy | 신규 | 신설·권장(Med) | **$/완료모델** + 절감률 — QA-05 top-line·QA-01 활용률 흡수 | N-C1 |

집계: 채택 권장 10 · 신설 3(강력권장 1·권장 2). KPI ✕ 3건 전부 측정가능화.

## 핵심 근거맵 (§4 라이브러리 유형 → 적용 QA)

| §4 유형 | 적용 QA | 핵심 출처 |
|---|---|---|
| 확장·처리량 (USL·KEDA) | QA-01 | wso2 USL · keda.sh |
| 지연 (percentile SLI) | QA-07·08·10 | Google SRE · oneuptime p50/p95/p99 |
| 가용·복구 (durable execution·SLO) | QA-02 | Temporal · Google SRE |
| 제어·안전 (zero-trust·HITL·injection) | QA-03·NQA-A | Anthropic Building Effective Agents / 신뢰 프레임워크 · SLSA |
| 관측성 (OTel GenAI semconv) | QA-04 | OpenTelemetry GenAI spans |
| 비용·효율 (prompt caching·$/task) | QA-05·NQA-C | Anthropic prompt caching·pricing |
| 일관성 (pass^k) | QA-09 | τ-bench arXiv 2406.12045 |
| 정확성 (golden set·LLM-as-judge) | NQA-B | comet/maxim/montecarlo |
| 격리·멀티테넌시 (bulkhead·token-bucket) | QA-06 | KEDA · SRE |

> 보강 검색: 공급망 서명 SLSA(https://slsa.dev/), LLM rate-limit(Anthropic API rate-limits). 모든 레퍼런스 수치는 **패턴 정당화용** — 우리 합격선은 PoC로 확정(복제 금지 준수).

## 교차권고 (red team C1~C5에 대한 응답)

- **C1 측정 불가 KPI** → QA-01(`N`)·QA-07("최대화")을 각각 scaling efficiency·노드타입별 speedup으로 환원. QAS Measure의 변수기호·"최대화" 전면 제거. **응답 완료**.
- **C2 정의↔KPI↔QAS 불일치** → QA-08(정의 E2E vs KPI artifact)은 E2E KPI 추가로 정렬(권장안 a). QA-03은 QAS의 "위반 0건"을 QA 파일에 정식 편입. **응답 완료**.
- **C3 QA 경계 중복(taxonomy)** → (a) QA-02(복구)↔QA-06(격리) 경계를 양쪽 본문 cross-link로 명문화. (b) **Performance 3분할**: QA-01=throughput, QA-07=per-node speedup, QA-08=E2E — 세 counsel이 이 정렬로 작성됨.
- **C4 agentic 고유 리스크 과소대표** → rate-limit 헤드룸(QA-01·06)·runaway cap(QA-03)·prompt caching 분리집계(QA-05)·pass^k 캐시우회(QA-09)·OTel agent span(QA-04)·red-team eval(QA-03·NQA-A) 편입. **응답 완료**.
- **C5 누락 1급 QA** → NQA-A·B·C 모두 채택 권고(정의·KPI·PoC 완비). 채택 시 번호 재정렬·KPI 이동(QA-01 활용률·QA-05 top-line → NQA-C) 명시. **응답 완료**.

## PoC 로드맵 요약 (상세: [`_poc-plan.md`](_poc-plan.md))

- **17개 PoC**가 §4의 9개 아키타입(부하·chaos·격리주입·계측·eval·반복시행·비용 A/B)을 전부 덮음.
- **3그룹 우선순위**: Group 1 토대(O1 계측·N-B1 정확성 게이트·A1 durable) → Group 2 High verdict(S1/S2·P1/P2·E2E1·C2/N-A1) → Group 3 보강.
- **핵심 교차 의존**: **NQA-B(PoC-N-B1)가 허브** — QA-07 first-pass·QA-09 유효-결정 판정의 전제로 선행 필요. QA-03≡NQA-A는 적대적 eval 하네스 공유. QA-04 계측이 비용 PoC 전부의 토대.
- **총 기간 약 5~6주**(공유 자산 재사용 가정). golden set 4단계 라벨이 critical path.

## 직전 라운드 대비 변화
- **최초 라운드 — baseline.** 다음 라운드는 본 counsel의 채택분이 `context/qa/`에 반영된 뒤 verdict 변화(KPI ✕ 3 → ?)를 추세로 비교한다.

## 미해결 / 추가조사 (팀 결정·silent cap)
- **확장 전략 가정**: QA-01은 self-host GPU 풀 vs 외부 API rate-limit 중 어느 쪽인가에 따라 KPI·PoC가 갈림 — **팀 선결**.
- **공정 baseline**: QA-07 수동 baseline·NQA-C 수작업 비용은 추정 의존(표본·숙련도·인건비 가정) — 측정조건 명시 필요.
- **DP 보강 역검토**: DP-0001/0002/0003 모두 **rate-limit·admission control·runaway cap·외부 LLM degradation·model-agnostic**을 명시 안 함 → 해당 DP에 tactic 보강 권고(각 QA counsel의 "DP 영향"에 개별 기재).
- **eval 인프라 신설**: NQA-A/B·QA-03/09의 red-team·golden 하네스는 현 DP에 없는 신규 서브시스템 — Stage 2에서 도입 결정 필요.
- **번호 재정렬**: NQA-A/B 상위 진입 + 활용률·top-line 이전은 QA 2자리 번호(=우선순위) 재정렬을 유발 — Stage 2 일괄 적용.
- **PoC silent cap**: mock 파이프라인 현실성·외부 시스템 통합·eval 커버리지 상한은 PoC가 못 봄(각 PoC 파일·_poc-plan.md에 log).
