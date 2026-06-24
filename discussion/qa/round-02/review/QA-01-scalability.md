# Review: QA-01 Scalability (round-02 재평가)

> source: `context/qa/QA-01-scalability.md` + `QAS-01-scalability.md` (round-01 [반영] 후)
> 직전 verdict(round-01): **Sound △ / KPI ✕ — High**
> verdict(round-02): **Sound ○ / KPI ○ — Low** — KPI 측정가능 재설계로 High 해소. 잔존은 DP 역검토(OI-7)·예시값 확정뿐 · severity **Low**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 QA-01 = **[반영]** (완료모델≥N·활용률 폐기 → scaling efficiency≥0.8 + rate-limit 헤드룸 + 큐 p95). 이월: DP-0001/0004 rate-limit 역검토(OI-7), 활용률→NQA-C.

## 원문 요약 (반영 후)
- **정의**: throughput 차원으로 고정 — 부하 증가 시 자원 비례 투입해 처리량 선형 유지, **1차 병목 = LLM rate-limit(TPM/RPM)**, 확장 신호 = backlog(큐 깊이·대기). per-node는 QA-07, E2E는 QA-08로 분리 명시.
- **KPI**: 주 = `scaling efficiency ≥ 0.8`(USL, 부하 2배→처리량 ≥1.8배). 보조 = `rate-limit 헤드룸 ≥ 20%`(throttle 0) · `큐 대기 p95 ≤ 5분` · (발표 앵커) 수작업 대비 처리량 배수.
- **QAS-01**: Response·Measure가 본문과 동기화됨(backlog 신호·1차 병목 rate-limit·세 KPI 동일 수치).

## 렌즈 1 — Agentic Workflow 전문가 관점

round-01의 핵심 지적("진짜 병목은 compute가 아니라 LLM rate-limit")이 **정의·KPI 양쪽에 정착**됐다. `rate-limit 헤드룸 ≥ 20%`(throttle 429 = 0)는 round-01에서 비어 있던 agentic 차원을 직접 채운다. backpressure/무한재시도 악화 패턴도 "backlog 신호 오토스케일 + 큐 대기 p95"로 흡수됐다 — 이 지적은 닫혔다.

다만 **두 개의 agentic 잔여 리스크**가 새로 드러난다(신규 결함이 아니라 다음 단계 점검 지점):
- **rate-limit 헤드룸의 측정 주체가 계정 전역인지 워크로드별인지 불명** — 외부 LLM의 429는 보통 *계정/조직 전역*으로 걸린다. QA-06(WF 격리)이 "외부 rate-limit은 계정전역 silent cap"이라 인정한 것과 정합되려면, QA-01의 헤드룸 ≥20%가 **어느 단위에서 보장되는 헤드룸인지**(전역 풀 vs 큐별 쿼터 배분) 한 줄이 필요하다. 지금은 "헤드룸 20%"가 전역인데, 한 워크로드가 그걸 다 먹으면 다른 워크로드는 0%일 수 있다. → QA-06과의 cross-link 보강 권고(Low).
- **scaling efficiency의 분모(부하)가 "토큰"인지 "모델 건수"인지 미고정** — USL은 동질 단위 부하를 전제로 한다. IR/Optimize/Quant/Compile은 토큰 프로파일이 크게 다르므로(QA-07이 "이질 노드 절대시간 차 무의미"라 지적한 것과 같은 뿌리), "부하 2배"의 단위가 무엇인지 정의하지 않으면 efficiency 0.8이 노드 믹스에 따라 흔들린다. → "부하 = (정규화된) 동시 워크플로우 수 또는 토큰 처리량"으로 단위 고정 권고(Low).

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

round-01의 두 치명상이 정확히 교정됐다:
- **placeholder `N` 폐기** → `scaling efficiency ≥ 0.8`. 통과/실패를 가르는 구체 임계값으로 acceptance test 작성 가능. **measurable 합격.**
- **`활용률 70%`(확장성 지표 오용 + headroom 상충)** → NQA-C(Cost)로 이전 표기. 단 **이 이전은 NQA-C 신설을 전제**로 하고 NQA-C는 아직 [이월](미채택, OI-8)이다. 현 시점 QA-01 본문은 "NQA-C로 이전"이라 적었으나 NQA-C가 정식 채택 안 되면 활용률 KPI는 **어디에도 살아 있지 않은 상태**(QA-01엔 없고 NQA-C는 임시). → **교차 의존: NQA-C 동반 채택 확인 필요**(Low, OI-8과 동일 트랙).

- **Sound 개선**: 정의 altitude가 "비례 확장"(방향)에서 "throughput 선형 유지"(명세)로 좁혀졌고, QA-07(per-node)·QA-08(E2E)와 경계를 본문에서 명문화했다(Performance 3분할). round-01의 "altitude 모호" 지적 해소 → **Sound ○로 상향**.

- **남은 형식 결함(경미)**: 보조 KPI "수작업 대비 처리량 배수"가 `overview "모델 140개+" × 목표 기간`으로 산출한다고 했으나 **목표 기간이 아직 미정**(overview에 처리 기간 명시 없음). 발표 앵커 수치라 PoC 대상은 아니지만, 발표 슬라이드에 배수를 쓰려면 목표 기간 합의가 선결.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

round-01 렌즈3 권고(control/data plane 분리, backlog 오토스케일, worker class 분리, admission control)가 **KPI 레벨에서는 반영**됐다(`backlog 신호`·`큐 대기 p95`·rate-limit 헤드룸). 그러나 이를 **실현하는 DP가 아직 그 차원을 명시하지 않는다** — 이게 round-02의 진짜 잔여물이다:

- 검증 전략 표가 `scaling efficiency`를 **DP-0001 2안 동적 풀 + DP-0004 A8 병목 단계 확장**에 귀속시키지만, OI-7이 명시하듯 **DP-0001·DP-0004 어디에도 rate-limit headroom·admission control tactic이 없다.** 즉 KPI는 측정 가능해졌으나 **그 KPI를 책임지는 설계가 비어 있다**(KPI-DP 연결의 한쪽 끝이 placeholder). 이건 QA 리뷰의 verdict를 내리진 못하고(QA 자체는 sound·measurable), **DP 디스커션으로 넘겨야 할 항목**임을 확인한다 → applier report §3-3과 일치, **다음은 DP 디스커션 차례**.
- admission control / bounded queue가 없으면 `큐 대기 p95 ≤ 5분`은 폭주 시 무한히 깨진다(p95가 정의되지 않음). p95 SLI가 의미를 가지려면 bounded queue가 DP에 있어야 한다.

- **runner 측 KPI**(재확인): scale-out latency(용량 추가 시간), 큐 대기 p95(반영됨), **admission 거부율/backpressure 발동률**(미반영 — bounded queue DP 신설 시 추가), worker class별 가동률 대비 headroom.

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | △ | **○** | throughput으로 altitude 고정 + QA-07/08 경계 명문화. 확장성은 이 시스템 1급 관심사로 타당 |
| KPI가 측정 가능한가 | ✕ | **○** | `N` placeholder 폐기 → scaling efficiency 0.8 등 구체 임계값. acceptance test 가능 |
| KPI가 현실적/적절한가 | ✕ | **○** | 활용률 오용 제거, rate-limit 헤드룸으로 agentic 병목 반영. 단 헤드룸 측정단위·USL 부하단위는 Low 보강 |
| 정의↔KPI↔QAS 일치 | ○ | **○** | QAS-01 Response·Measure가 본문과 동기화 확인 |

**verdict 변화: Sound △→○ / KPI ✕→○ · severity High→Low.** round-01 High의 근거(측정불가 KPI)는 해소됨.

## Stage 2 권고 (round-02)

대부분 닫혔으므로 **잔여 보강(Low)만** 남긴다 — 이번 라운드 신규 수정 대상이 아니라 DP 디스커션·팀 결정으로 위임되는 항목이다:

- **(DP 디스커션 위임, OI-7)** scaling efficiency·rate-limit 헤드룸·큐 p95를 책임지는 DP-0001/DP-0004에 **rate-limit headroom·admission control·bounded queue tactic**을 명시. KPI는 닫혔으나 설계 귀속이 placeholder.
- **(NQA-C 동반, OI-8)** `자원 활용률`의 이전처(NQA-C) 정식 채택 확인 — 미채택이면 활용률 KPI가 부유(浮遊). QA-01 본문은 그대로 두되 채택 결정과 동기화.
- **(헤드룸 단위 명시, Low)** `rate-limit 헤드룸 ≥ 20%` `기존(단위 불명)` → `제안: 계정 전역 TPM/RPM 대비 헤드룸 ≥ 20%, 큐별 쿼터로 워크로드 간 배분(QA-06 격리와 cross-link)`.
- **(USL 부하단위 고정, Low)** `scaling efficiency ≥ 0.8` 부하 정의를 `정규화 동시 워크플로우 수 또는 토큰 처리량`으로 못 박아 노드 믹스 의존성 제거.
- **(발표 앵커)** "수작업 대비 처리량 배수"는 목표 처리 기간 합의 후 산출(현재 overview에 기간 없음).
