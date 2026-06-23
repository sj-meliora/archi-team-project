# QA 리뷰 round-01 — 결론 리포트

> **이 문서 하나로 round-01의 전모를 파악할 수 있다.** 개별 근거는 `QA-0X-*.md`, 방법론은 [`../../Reviewer.md`](../../Reviewer.md).
> date: 2026-06-23 · scope: `context/qa/` QA 10 + QAS 10
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 직전 라운드: 없음(최초)

## TL;DR

- **QA 자체는 대체로 타당**하나(✕ 없음), **KPI가 절반은 깨져 있다** — 측정 불가(placeholder·“최대화”) 또는 정의↔KPI 불일치.
- **가장 시급(High 4)**: QA-01·QA-07(측정 불가 KPI), QA-08(정의=E2E ↔ KPI=artifact 불일치), QA-02(MTTR 불완전 + 외부 LLM 장애 시나리오 부재).
- **구조적 문제**: QA 경계 중복(02↔06, 07↔08), **agentic 고유 리스크 과소대표**(rate-limit·캐싱·runaway·eval), **1급 QA 누락**(Security·Correctness·Cost).
- **렌즈3 결론**: 다수 KPI가 **runner 아키텍처로 측정 가능하게 환원된다**(durable execution→MTTR/무손실, 큐 쿼터→WF 격리, event-history→재구성). 즉 “고치는 법”이 이미 보인다.

## 종합 판정표

| QA | 속성 | Sound | KPI | 핵심 결함 | severity |
|---|---|:---:|:---:|---|:---:|
| [QA-01](QA-01-scalability.md) | Scalability | △ | ✕ | KPI `N` 미정의; “활용률 70%”는 확장성 아닌 비용 지표(headroom과 상충); 오토스케일 신호로도 부적절 | **High** |
| [QA-02](QA-02-availability.md) | Availability | ○ | △ | `MTTR<1분` 불완전(가용률·무손실 누락)·비현실; 외부 LLM 장애 부재. → durable execution으로 해소 | **High** |
| [QA-03](QA-03-controllability.md) | Controllability | ◎ | △ | QAS의 “허용범위 외 0건” 누락(불일치); eval·runaway cap 부재 | Med |
| [QA-04](QA-04-observability.md) | Observability | ○ | △ | “재구성 95%” 모호 → event-history 완전성 + agent span으로 구체화 | Med |
| [QA-05](QA-05-efficiency.md) | Efficiency | ○ | △ | `≤8k` 비현실(빌드로그)·캐싱 페널티; top-line은 `$/완료모델` | Med |
| [QA-06](QA-06-reliability-workflow.md) | Reliability (WF 격리) | ○ | ○ | QA-02와 경계 중복; 토큰/rate-limit 쿼터 격리 누락(runner 큐 쿼터로 해소) | Med |
| [QA-07](QA-07-performance-agent-time.md) | Performance (Agent 시간) | △ | ✕ | “최대화”=비측정; 품질 게이트 없음; 진짜 가치는 throughput·병렬성 | **High** |
| [QA-08](QA-08-performance-e2e.md) | Performance (E2E) | △ | ✕ | 정의(E2E) ↔ KPI(artifact 5%) 불일치; 실제 E2E 지표 부재 | **High** |
| [QA-09](QA-09-reliability-agent-consistency.md) | Reliability (일관성) | △ | △ | 일관성 ≠ 정확성; 캐시 재현율 100%는 치트; 결정성 목표 재고 | Med |
| [QA-10](QA-10-maintainability.md) | Maintainability | ○ | ○ | `CIS≤2 평균`이 tail 은닉; prompt/tool/model 유지보수축 누락 | Low |

집계: **High 4 · Med 5 · Low 1** · Sound ✕ 0 · KPI ✕ 3.

## 교차(cross-cutting) 발견

- **C1. 측정 불가능한 KPI** — QA-01(`N`), QA-07(“최대화”). QAS Measure는 반드시 구체 임계값. → 최우선 교정.
- **C2. 정의↔KPI↔QAS 불일치** — QA-08(정의 E2E vs KPI artifact), QA-03(QAS의 0건이 QA 파일에 없음).
- **C3. QA 경계 중복(taxonomy)** — 02↔06(fault: 복구 vs 격리), 07↔08↔01(performance: latency/E2E/throughput). → Performance 통합(per-node + E2E throughput) 권고.
- **C4. Agentic 고유 리스크 과소대표** — rate-limit, 외부 제공자 장애, 토큰 경제·prompt caching, runaway cap, eval/red-team 하네스가 거의 부재. 현재 QA는 “일반 분산시스템” 어휘에 머묾.
- **C5. 누락된 1급 QA**(권고, [`_new-qa-candidates.md`](_new-qa-candidates.md)) — **NQA-A Security/Safety**, **NQA-B Correctness/Accuracy**, **NQA-C Cost-economy**.

## 렌즈3(Runner 인프라) 횡단 결론

세 번째 렌즈가 드러낸 핵심: **상당수 결함은 runner 아키텍처 선택으로 측정 가능·달성 가능해진다.**
- **durable execution(이벤트 소싱·체크포인트·replay)** → QA-02(무손실·MTTR 분해), QA-04(재구성=event-history 완전성), QA-09(제어흐름 결정성).
- **큐별 쿼터·token-bucket·fair queueing** → QA-06(WF 격리)·QA-01(noisy-neighbor) 동시 해소.
- **backlog 기반 오토스케일·worker class 분리·bin-packing** → QA-01(확장)·QA-05(가동률) — “활용률 70%”의 올바른 자리는 runner 효율 지표.
- **artifact 참조 전달·data locality** → QA-08(전달 오버헤드 5%)의 실제 달성 수단.
- **activity timeout/retry cap·협조적 취소+hard-kill** → QA-03(runaway·중단)의 인프라 보장.

## Stage 2 착수 우선순위

1. **High 4 먼저**: QA-01·07(측정불가 KPI 교체), QA-08(정의↔KPI 정렬), QA-02(MTTR 분해 + 외부장애 시나리오).
2. **분류 정리(C3)**: 02↔06, 07↔08 경계/통합 결정 후 일괄 재서술.
3. **신규 QA 채택(C5)**: 최소 **NQA-A(Security)·NQA-B(Correctness)** 채택 권장 — “사람 없이 믿고 맡길 수 있는가”의 두 기둥, 발표 설득력 직결. 채택 시 번호 재정렬·KPI 이동(활용률→NQA-C 등) 동반.

## 이번 라운드에서 한 일 (요약)

- QA 10개를 3렌즈로 전수 리뷰, 4축 rubric으로 판정.
- 세트 전체 교차분석 C1~C5 도출, 신규 QA 3종 권고.
- KPI별 `기존 → 제안`을 개별 파일 Stage 2 권고에 기재(적용 가능 형태).
