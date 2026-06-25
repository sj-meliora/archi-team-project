# Review: QA-02 Availability

> source: context/qa/QA-02-availability.md + QAS-02-availability.md
> verdict: **Sound ◎ / KPI ○** — failover 1분→4분 재배치는 게이트/별점 분리가 모범적이나 **★★★ 10초가 "재스케줄 ~0.5초"와 "재기동 ~10초"를 섞은 단위 혼동, MTTR 단위 apples-to-apples 주의** · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> 특별 초점(round-04): ★ 등급 척도(failover 하한 1분→4분) 검증 무게중심.

## 원문 요약
- 정의: 부분 장애·외부 LLM 장애에도 무손실 멱등 재개. 가용성 본질 = 빠른 재기동이 아니라 무손실.
- 헤드라인: `재기동 ≤4분 AND in-flight 손실=0`(손실=0은 게이트 불변).
- ★ 급간: ★★★ ≤10초 / ★★☆ ≤1분 / ★☆☆ ≤4분 / 불합격 >4분 또는 게이트 위반. 게이트: 손실=0·멱등=100%.

## 렌즈 1 — Agentic Workflow 전문가 관점 (필드 근거 보강/반박)
가용성의 지배적 위협을 **외부 LLM 제공자 outage**로 정확히 짚었다(자동 재개율 ≥95%). 그러나 ★ 급간 main 축(재기동 시간)은 **내 노드/worker 재기동**만 잰다 — 외부 LLM outage는 분 단위가 아니라 **수십 분~시간** 단위일 수 있는데, 그건 자동 재개율 보조 KPI로만 다루고 ★ 급간엔 안 들어간다. 즉 ★★★(≤10초)를 받아도 **외부 LLM이 30분 죽으면 가용성은 무너진다.** apples-to-apples: 인용한 durable execution 재개(0.5~20초)·DORA elite MTTR(<1hr, 인시던트 단위)는 **서로 다른 단위**인데 표가 둘을 나란히 놓아 "DORA와 별개"라 자인은 했으나 독자 혼동 여지. → 외부 장애 길이가 ★ 급간에 반영 안 됨을 silent cap.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (★ 급간 검증 핵심)
- **규칙5 분리 모범**: `손실=0·멱등=100%`를 0건 절대형 게이트로 빼고, gradable한 재기동 시간에 별점. "가용성 본질은 무손실"을 게이트로 못박은 것 정확. ◎.
- **급간 reasonableness**: ★★★ ≤10초 / ★★☆ ≤1분 / ★☆☆ ≤4분 — **로그 스케일에 가까운 비등간격**(10s→60s→240s, 약 6배·4배). 시간 지표를 로그 배치한 건 합리적(자의적 등간격 아님). ○.
- **★★★ 10초의 단위 혼동(가장 날카로움)**: 근거가 "healthy dispatcher 잔존 시 재스케줄 ~0.5초, 컨테이너 재기동 ~10초, job manager 교체 ~10초"를 **OR로 묶어** 10초로 잡았다. 0.5초(재스케줄, 죽지도 않은 dispatcher가 다른 worker에 배정)와 10초(컨테이너 cold 재기동)는 **다른 사건**이다 — 무엇을 "재기동"으로 정의하느냐에 따라 ★★★ 경계가 0.5초일 수도 10초일 수도. main 축 정의가 모호. → 권고 1.
- **하한 보정 정합(OI-9)**: §측정(32행 ≤4분), 등급표, 변경이력, counsel(1분→4분), QAS-02(`≤4분 ★☆☆`) **5곳 일치**. glossary 없음. **OI-9 통과.**
- **변별력**: 구 1분 하한은 stateful 종단 재개에 비현실(보정 사유 정확). 신 4분은 "redundancy 라우팅 최대 3~4분" 근거로 ★☆☆ 비사문화. ★★★(10초)도 durable execution 관측 대역이라 비사문화. 양호. ○.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점
재기동 시간 = durable execution의 lease timeout + 재스케줄. main 축이 "lease timeout"에 사실상 종속(lease를 10초로 잡으면 ★★★, 4분으로 잡으면 ★☆☆) → **★ 급간이 설계 파라미터(lease) 선택의 함수**라 "설계 대안 변별"보다 "lease 값 선택"을 보상할 위험. runner KPI: `재기동 시간 분포의 분산이 게이트(손실=0) 통과율과 독립` — 빠른 lease가 손실률을 올리면 trade-off를 ★가 못 잡음.

## 판정
| 축 | 기호 | 근거 |
|---|---|---|
| Sound | ◎ | 복구 단일 관심사·QA-08(격리) 경계 명문 |
| Measurable | ○ | 재기동·성공률·재개율·멱등성 4축 구체 |
| Realistic | ○ | 4분이 stateful 현실 정합, ★★★·★☆☆ 비사문화. 단 외부 LLM outage 길이 미반영 |
| Consistent | ◎ | 5곳 ≤4분 일치(OI-9 통과) |

**verdict: Sound ◎ / KPI ○ · Low** (round-02 ○/○ Low 유지).

## Stage 2 권고
1. **[main 축 정의 명확화 · Low]** ★★★ 10초 근거의 "재스케줄 0.5초 vs 컨테이너 재기동 10초"를 분리 — `제안: "재기동"을 'lease 만료 후 다른 worker가 체크포인트부터 재개 완료까지'로 정의 고정`.
2. **[외부 outage silent cap · Low]** ★ 급간(내 노드 재기동)이 외부 LLM outage 길이를 안 잰다 — 외부 장애는 자동 재개율로만 다룸을 ## 등급 척도에 명시.
3. **[lease 파라미터 trade-off · Low]** 빠른 lease가 손실 게이트와 trade-off일 수 있음을 silent cap.
4. **[근거 단위 → Council · Low]** durable execution 0.5~20초·DORA MTTR <1hr의 단위 차이를 round-05 Council이 1차 출처로 명확화.
5. DP 역검토: DP-0002/0003의 외부 LLM degradation(backoff·폴백) 미명시(OI-7).
