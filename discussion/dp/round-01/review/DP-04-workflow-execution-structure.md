# Review: DP-04 Workflow 실행 구조

> source: context/dp/DP-0004-workflow-execution-structure.md (+ dp4/: INDEX·decision-axes·approaches A3~A8·evaluation·review/findings)
> verdict: **ASR △ / KPI △** — A5 vs A8 수렴은 모범적이나 driving ASR이 QA-01 1개뿐이고 그마저 변별 없는 N칸 · 풍부한 분석이 전부 비-ASR(QA-08/10/12)에 쏠림 · severity **Med** (orphan은 세트 결함으로 report C*)
> lenses: (1) Agentic Workflow 전문가 · (2) ATAM 평가 수석 아키텍트 · (3) Runner 인프라 아키텍트

## 원문 요약
- **결정 포인트**: 파이프라인 노드(IR Converter→Graph Optimizer→Quantizer→Compiler)를 어떤 실행 구조로 처리할까.
- **후보 대안**: 1안(타입별 공유 풀)·2안(노드당 인스턴스)을 기준선(null)으로 격하 → dp4/decision-axes.md가 축 A를 2개 직교 손잡이(수명: 상주↔일회용 / 배치: 원격↔로컬)로 분해, **A5(일회용·원격) vs A8(일회용·로컬)** 둘로 수렴.
- **driving ASR QA**: 헤더 `drives:` = QA-01(Scalability) · QA-08(Reliability-WF) · QA-10(Performance-E2E) · QA-12(Maintainability). **이 중 ASR(QA-01~07)은 QA-01 단 하나** — 나머지 3개는 비-ASR.
- **★매트릭스**: 1안/2안/A5/A8 × Perf/Scalability/Rel-WF/Maintain. A5·A8 모두 Scalability ★★★. 가로축(Perf↔Rel)에서만 A5/A8이 갈림.
- **ATAM**: SP-1(전달배치→Perf/QA-10)·SP-2(내구성→Rel/QA-08)·SP-3(cold-start)·SP-4(footprint) / TP-1(Perf↔Rel)·TP-2(단순성↔로컬리티) / R-1·R-2·R-3 / NR-1(C-0001·C-0002).
- **결정/근거**: `(4단계×20GB 왕복)÷대역폭`이 QA-10 5% budget 안이면 A5, 넘으면 A8. 구조를 고정하고 수치는 측정으로.

## ASR cover·KPI 비교 (1순위 판정)
- **세트 관점(렌즈2 고유)**: DP-04는 자기 driving 4개 중 **ASR을 단 1개(QA-01)만 cover**한다. SP/TP/Risk의 거의 전부가 QA-08·QA-10(비-ASR)을 가른다 — 이 DP의 분석적 풍부함(A5 vs A8 결선, 5% 산식, TP-1)은 **모두 비-ASR 축**에서 발생한다. 즉 §0-①의 안티패턴: "비-ASR(QA-08~13)에 강점이 쏠려 정작 ASR은 평이"의 **DP판**이다(F-07의 ASR판).
- **QA-01(유일 ASR) cover 품질**: A5·A8 둘 다 Scalability ★★★ — 즉 이 결정의 **유일한 ASR 칸에서 두 결선 대안이 동점**이다. ASR을 가르지 못한다(F-01형 변별력 0의 ASR판). QA-01의 주 KPI는 `scaling efficiency ≥0.70`(USL, rate-limit 헤드룸 ≥20% 조건)인데, ★매트릭스의 Scalability ★★★ 칸은 이 임계값에 **앵커되어 있지 않다** — "과프로비저닝 0"이라는 정성 서술만. 또 QA-01의 진짜 천장은 서버 CPU가 아니라 **외부 LLM TPM/RPM 한도**인데(QA-01 정의), A5/A8 어느 쪽도 admission control·rate-limit 헤드룸을 분석하지 않는다(OI-7이 "DP-0004 rate-limit·admission control 차원 미명시"로 이미 트래킹).
- **KPI 앵커 정직성(비-ASR이지만 헤드라인 산식)**: QA-10 5% budget을 `(4단계×20GB÷대역폭)`으로 환원한 것은 모범적(F-04 교환점 은폐를 정확히 해소). 단 이 KPI는 **비-ASR**(QA-10)이므로, ASR coverage 점수를 끌어올리지 못한다.
- **결론**: 이 DP의 ATAM 완성도(수렴·SP/TP/Risk·산식)는 세트 최고지만, **ASR 적합성은 가장 약하다.** driving 선언과 ASR 세트의 어긋남이 핵심 결함.

## 렌즈 1 — Agentic Workflow 전문가 관점
- **설득력·비식상성**: 1안·2안 → A5 vs A8 수렴은 "그냥 중앙 풀 두기"라는 1차 해법을 넘어 **수명×배치 2x2**로 직교 분해한 점에서 설득력이 높다(식상하지 않음). data-affinity(A8) vs claim-check(A5)의 대결은 agentic 파이프라인의 실제 난제(20GB 중간 산출물)를 정면으로 다룬다.
- **agentic 고유 제약 충돌(load-bearing 가정)**: ⚠️ **멱등 가정이 QA-11(재현율 기반)과 모순**(findings **F-05**). A5의 "invocation별 자동 retry + 스토리지 보존"과 A8의 "단계부터 재실행"은 둘 다 **재시도/재실행이 동일 산출물을 낳음(멱등)**을 암묵 전제한다. 그러나 시스템은 QA-11에서 비결정성을 공식 인정(`pass^k`) — Quantizer/Compiler 출력이 run-to-run 변동하면 재실행 경로마다 다른 산출물. 이 load-bearing 가정이 ATAM 4절 어디에도 SP/Risk로 노출되지 않았다. **QA-11은 DP-04 driving에도 없다**(F-12 범위 불일치). 단 QA-11은 비-ASR이라 severity는 Med.
- **rate limit·토큰 경제**: A5의 "near-infinite auto-scale"은 외부 LLM TPM/RPM 천장을 무시한 표현이다(렌즈2의 QA-01 앵커 문제와 연동). 호출 폭증 시 진짜 막히는 곳은 컨테이너가 아니라 LLM 쿼터 — auto-scale ★★★ 주장이 과장될 소지.
- **외부 제공자 의존**: cold-start(R-1·R-3)는 잡았으나, **외부 LLM outage/429 시나리오**(OI-7: "DP-0004 외부 의존성 backoff·폴백 미명시"는 DP-0002/0003 쪽이지만 실행구조도 무관치 않음)는 R로 없음.

## 렌즈 2 — ATAM 평가 수석 아키텍트 관점 (ASR cover·KPI 비교·분석 건전성)
- **Well-framed / 직교성**: ◎ 우수. dp4의 4축(A=실행위치 / B=작업분배 / C=구조분해 / D=변종·복구) 분리와 축 A의 2손잡이 재분해는 "여러 직교 축을 단일 택1로 뭉개지 않는다"는 모범. evaluation.md가 A3↔A6를 **이중 제어평면 금지**(F-03)로 분기하고 DP-02와 동시 결정으로 못박은 것도 우수.
- **대안 완전성·공정성**: ○ 1안·2안을 null 기준선으로 명시 유지 → 지배관계가 투명. F-08(범주오류)도 evaluation에서 A7 variety를 Scalability에서 분리해 해소.
- **조합/창발 비용**: ○ 종전 "다 쌓기 스택"을 F-02/F-10 반영해 폐기하고 "조건부 얹기"로 수렴. 단 **A8 + DP-0005 공유 캐시 결합**(본문 "A8은 DP-0005 캐시를 노드 로컬 볼륨 계층과 결합")의 합성 신뢰성은 미산정(F-09 잔존 — 공유 장애도메인 누적은 report C*로).
- **핵심 교환점 노출**: ◎ TP-1(Perf↔Rel)을 가로축=교환점으로 정면 노출, claim-check 전달세금을 "순이익"으로 위장하지 않음(F-04 해소).
- **가중(중요도)**: △ driving 4개 중요도(QA-01=H, QA-08=M, QA-10=M, QA-12=L)가 매트릭스에 가중으로 반영 안 됨(F-07 잔존). 특히 **유일 ASR QA-01(H)이 최강 변별 칸이 아니다** — 결정을 가르는 산식은 QA-10(M, 비-ASR) 축. ASR·중요도 가중을 적용하면 "이 결정의 헤드라인은 QA-01인데 정작 QA-01로는 안 갈린다"는 모순이 드러난다.
- **★ 등급 척도 경계**: A5·A8 Scalability ★★★ 동률 + QA-01 ★ rubric(scaling efficiency ≥0.70 하한) 미앵커 → 이 칸은 변별·앵커 둘 다 결여.

## 렌즈 3 — Runner 인프라 아키텍트 관점 (구현 현실성)
- **구현 환원**: ◎ A5=K8s Job/Knative/FaaS, A8=Pod Affinity+Local PersistentVolume(R-05 근거)으로 구체화 — 추상 tactic이 실제 메커니즘으로 환원됨. data-locality 스케줄링·co-location은 인프라 정석(HDFS data locality 선례)으로 식상하지 않음.
- **blast-radius / FMEA**: △ A8 로컬 디스크 유실(R-2)은 잡았으나 **% Workflow 중단 정량 없음**. QA-08 KPI(타 WF 중단 ≤◯%)에 대조한 FMEA 부재(F-09). 단 QA-08은 비-ASR.
- **데이터 평면**: ◎ `(4단계×20GB÷대역폭) vs 5% budget` 산식이 정확히 데이터평면 비용을 계량 — 렌즈3가 요구하는 "정량 산식 1개"를 본문이 이미 제시.
- **확장·격리·제어**: △ scale-to-zero 후 동시폭증 throttling(R-3, min-instance 하한)은 잡음. 단 **이 throttling이 곧 QA-01 scaling efficiency를 깎는 지점**인데 QA-01 KPI와 연결 안 됨(렌즈2 앵커 문제와 같은 뿌리).
- **채택/운영 리스크**: △ 신규 인프라(Knative/Local PV/data-affinity 스케줄러)의 운영 부담·요구 역량이 ATAM Risk에 없음(F-11 잔존).

> **구현·측정 메커니즘**: ① QA-01 앵커링 — `scaling efficiency = (2배 부하 시 throughput) / (1배 throughput)`을 A5(클러스터 폭증)·A8(노드 내 단계 입도) 각각에 대해 PoC 측정, **rate-limit 헤드룸 ≥20% 고정 조건**에서 ≥0.70(=1.40배) 충족 여부로 ★ 변별. ② 멱등 검증 — Quantizer/Compiler 단계를 결정적/비결정적으로 분류하고 비결정 단계에 seed 고정 or de-dup+결과 영속(exactly-once 효과)을 적용해 재실행 산출물 동일성을 `pass^k`(QA-11)로 계측.

## 판정
| 축 | 판정 | 근거 |
|---|:---:|---|
| **ASR-coverage** | △ | driving 4개 중 ASR 1개(QA-01)뿐 + 그마저 A5·A8 동점(★★★)으로 변별 0 · 분석 풍부함이 전부 비-ASR(QA-08/10/12)에 쏠림(F-07 ASR판) |
| **KPI-comparison** | △ | QA-10 5% 산식은 모범 앵커(F-04 해소)이나 **비-ASR** · 유일 ASR인 QA-01은 `scaling efficiency ≥0.70`에 미앵커·미변별 |
| **Well-framed** | ◎ | 4축 직교 분해 + 2x2 수명·배치 재분해 · realizes FR-0001/0002 연결 · 결정 포인트가 TP-1을 품음 |
| **ATAM-sound** | ○ | TP-1 교환점 정면 노출·claim-check 비은폐(F-04 해소) · 단 멱등 load-bearing 가정 미노출(F-05)·합성 신뢰성 미산정(F-09) |
| **Convergent** | ◎ | "다 쌓기"→A5 vs A8 단일 산식 택일 + 배제기준(cold-start·footprint 트리거) · MVA=2안 기준선 유지 (F-10 해소) |
| **Traceable** | ○ | 본문이 dp4/ 역참조·OI-3 라벨 교정 명시 · 단 driving에 QA-11 누락·QA-01 silent(F-12 잔존), drives↔ASR 어긋남 |

## Stage 2 권고
1. **driving QA ↔ ASR 정렬 (핵심)**: `drives: QA-01(Scalability), QA-08, QA-10, QA-12` → ASR이 QA-01뿐임을 헤더에 명시하거나, **QA-08을 QA-01과 동급 헤드라인으로 올릴 수 없다면** 이 DP는 "비-ASR 실행품질 결정"으로 자리매김하고 ASR coverage는 QA-01에 집중. (대안: QA-08을 ASR 재선정 논의에 — 단 OI-8에서 QA-08은 이미 ASR 제외 확정이므로 비-ASR 유지.)
2. **QA-01 KPI 앵커**: `Scalability ★★★ → 칸 옆 정량 보조열 추가` — A5: `scaling efficiency ◯`(클러스터 auto-scale, rate-limit 헤드룸 ≥20% 조건) / A8: `scaling efficiency ◯`(노드 내 단계 입도). 두 값이 ≥0.70을 넘되 **서로 갈리는지** PoC로 변별. (현재 ★★★ 동률 → 변별 0.)
3. **rate-limit·admission control 차원 명시 (OI-7)**: A5의 "near-infinite auto-scale"에 **외부 LLM TPM/RPM 천장 + admission control** 단서 추가 — 진짜 천장은 컨테이너가 아니라 LLM 쿼터.
4. **멱등 load-bearing 가정 1급 노출 (F-05)**: ATAM Risk에 "SP-5(멱등성 성립 조건 → QA-11)" 신설 — 비결정 단계 분류 + seed 고정/exactly-once 설계. driving에 QA-11 추가 검토(F-12).
5. **합성 신뢰성 (F-09)**: A8+DP-0005 공유 캐시 결합 시 공유 장애도메인 누적을 간이 FMEA로 — % WF 중단을 QA-08 KPI에 대조(report C*로 세트 차원 처리).
6. **채택/운영 리스크 (F-11)**: Knative/Local PV/data-affinity 스케줄러 도입 부담을 ATAM Risk "채택/운영" 범주로 추가.
