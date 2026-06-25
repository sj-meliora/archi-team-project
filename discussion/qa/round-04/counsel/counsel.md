# round-04 counsel — ★ 등급 척도 근거 보강 (blue team Council)

> 날짜: 2026-06-25 · 대상: QA-01~13 ★ 등급 척도(round-03 캘리브레이션) · 유형: **표준 Council**(red team round-04 review 입력 → blue team counsel).
> 입력: [`../review/report.md`](../review/report.md)(C1~C4·교차발견) + QA-0X review 13개 + [round-03 counsel](../../round-03/counsel/counsel.md)(★ 설계 의도). 방법론 [`../../Council.md`](../../Council.md) 특히 §9.
> 이 문서 하나로 권고 전모를 안다 — 채택 권고표·C1~C4 응답·C4 웹검증·★ 급간 변경·직전 대비.

## TL;DR
- **방향·구조는 13개 전부 통과**(review TL;DR: 규칙5 게이트/별점 분리·OI-9 정합·가짜 급간 0). Council 임무 = red-team이 깐 **근거 품질 4축(C1~C4)을 레퍼런스+PoC로 메우기**.
- **C4 웹 전수 검증 완료**: τ-bench(2406.12045)·Unit 42(53/91/92%·FPR 13.1%) **실재·정확**. 미래형 의심 arXiv ID는 현재(2026-06) 기준 **유효 과거 ID**. **단 InfEngine(2602.18985)은 ID는 실재하나 인용 수치 오류**(원문 21×·적외선 컴퓨팅 ≠ 인용 "8.6~22.7×·assistant-type") → QA-09 정정.
- **QA-11 ★★★ 최종 판정 = ≥60% → ≥45% 하향 확정.** voting이 pass^k(all-k-succeed)를 33→60%로 끌어올린다는 것은 메커니즘 오해임을 1차 출처로 반증 — voting은 pass@k(1답 정확도)를 올릴 뿐. 45%는 "frontier+결정성 레버로 시행 간 양의 상관" 근거로 댄다.
- **C3 보조 별점 축 이중화**(QA-07 rework율·QA-11 ②-1·QA-12 prompt/tool 수정 수·QA-13 재시도 오버헤드) — main 축이 [발표 서사]면 실측 가능 보조 축으로 ATAM 변별력 보강.
- stance 집계: **조건부 채택 7**(QA-01·02·06·07·09·11·13) · **채택 권장 6**(QA-03·04·05·08·10·12). ✕/보류 0.

## 채택 권고표 (13)

| QA | sev | stance | 핵심 권고(기존→제안) | C* |
|---|---|---|---|---|
| QA-01 | Low | 조건부 | margin 차등 근거화(천장 거리 비례)·LLM-bound USL silent cap | C1·C2 |
| QA-02 | Low | 조건부 | ★★★ "재기동"=lease만료→체크포인트 재개 정의 고정·외부 outage silent cap | C1 |
| QA-03 | Low | 채택 권장 | 세트 모범 — ★★★ 15초 근거 다양화(non-k8s 병기)·롤백 외부 정합성 한 줄 | C1 |
| QA-04 | Low | 채택 권장 | 경계 [95,97)/[97,98)/[98,100) 구간화·완전성=존재 AND 비-truncation | C1 |
| QA-05 | Low | 채택 권장 | 적중률 조건이 변별폭 좁힘 silent cap·tier margin 난이도 매핑 근거 | C1·C2 |
| **QA-06** | Med | 조건부 | **직접 injection apples 1차출처 명문화·★★☆ 85%로 margin 5%p 규칙화·FPR 전급간 게이트** | C1·C2·C4 |
| QA-07 | Med | 조건부 | ★★★[93,99]·★★☆[90,93)·100%만 불합격·judge κ 도메인 재측정·**rework율 보조 축** | C1·C3 |
| QA-08 | Low | 채택 권장 | ★★★ 5%=tail안정화+폭주흡수 근거화·쿼터 보조 축 검토·arXiv 확인불가 표기 | C1·C2·C3·C4 |
| **QA-09** | Med | 조건부 | **InfEngine 인용 정정(21×·도메인 불일치)·하한 표현 단일화·tool 시간 분리** | C1·C4 |
| QA-10 | Low | 채택 권장 | ★★★ 2h=compute critical path 하한 근거화·agentic LLM 큐잉 가산 | C2 |
| **QA-11** | Med | 조건부 | **★★★ 60→45% 하향(voting 반증)·②-1 보조 별점 축·k 보간 silent cap·구값 정정** | C1·C3·C4 |
| QA-12 | Low | 채택 권장 | CIS 정수→구간화(1<p95≤2)·**prompt/tool 수정 수 보조 축**·경계 종속 silent cap | C3 |
| QA-13 | Med | 조건부 | baseline 추정 silent cap·**재시도 오버헤드 보조 축**·★★★ 도메인무관 분포 재근거·OI 등록 | C1·C2·C3 |

## C1~C4 응답 요지

### C1 — apples-to-apples 미검증 (전 세트)
- **대표 응답 = QA-06**: Unit 42가 **단발·직접 injection(JailbreakBench, single-turn only)**임을 1차 출처로 **확인**하고, 우리 QA-06는 **agentic 간접 injection**을 잰다는 차이를 silent cap으로 명문화. 이는 추정이 아니라 검증된 사실.
- 나머지(QA-01 SPARCcenter CPU USL / QA-05 Requesty 코딩 / QA-07 일반 judge κ / QA-08 스토리지 noisy-neighbor / QA-09 적외선 컴퓨팅 / QA-13 법무)는 각 ★ 노트에 "인용 = 다른 셋업·도메인, 경계는 차이 감안 예시값, 우리 PoC로 확정" silent cap. QA-03·04·10은 인용이 우리 도메인과 정합(apples 우려 낮음 — 유지).

### C2 — PoC margin 미규칙화
- **margin에 근거 부여**(임의값 탈출): QA-06 **단일 Y=5%p 규칙**(★★☆ 85%로 통일) / QA-01 "천장과의 거리에 비례하는 차등(이론 0.05·필드 0.02)" / QA-08 "tail 안정화+폭주 강도 흡수 ≈5%" / QA-10 "compute critical path 하한 2h" / QA-13 "절감 분포 두께 차등(50~75% 두텁고 75%+ 희소)" / QA-05 "tier=난이도 매핑". 각 margin이 **왜 그 값인지** 답한다.

### C3 — main 별점 축 실측불가 군집 (QA-07·11·13 + QA-12)
- **이중 축 설계**(모델 추정 main + 실측 가능 보조): QA-11 **②-1(룰 게이트)** · QA-07 **rework율** · QA-13 **재시도 오버헤드** · QA-12 **prompt/tool 수정 수**. 모두 golden·baseline 추정 없이 우리 조건에서 산출 가능. ATAM 비교 시 main 축이 미실측/동률이면 보조 축으로 변별 → ★ 급간 실효성 회복.

### C4 — 근거 출처 신뢰성
- **웹 전수 검증**(§ 아래 표). 핵심: τ-bench·Unit 42 정확, **InfEngine 수치 오류 발견·정정**, voting→pass^k 메커니즘 반증, 미래형 의심 ID는 유효 과거 ID. 미대조 인용(TRYLOCK·Const.Classifiers·배치 ML SLA·법무·Requesty·일부 arXiv)은 "확인 불가"로 정직 표기 — 수치 복제 금지(§4 경계) 준수.

## C4 웹 검증 결과 (1차 출처)

| 인용 | 실재 | 정확 | 조치 |
|---|---|---|---|
| τ-bench 2406.12045 (QA-11) | ✅ | ✅ GPT-4o pass^1 61.2·pass^4 ~37·pass^8 ~25·frontier ~80% | 유지 |
| **voting→pass^k 향상** (QA-11 ★★★) | — | ❌ 메커니즘 오해(voting=pass@k 1답 정확도, pass^k 아님) | **★★★ 60→45% 하향** |
| Unit 42 53/91/92·FPR 13.1% (QA-06) | ✅ | ✅ 정확·**단발 직접 injection** | apples 명문화 |
| **InfEngine 2602.18985 (QA-09)** | ✅ | ❌ 원문 21×·92.7% pass·적외선 컴퓨팅 ≠ "8.6~22.7×·assistant" | **인용 정정** |
| 2511.07413 (QA-07) | ✅ DigiData | 맥락 대조 미완 | 출처 유효 표기 |
| 2511.15759·2504.11168 (QA-06) | 형식 유효(2025) | 미대조 | 형식 유효·복제 아님 |
| 2604.03145 (QA-08) | 형식 유효(2026-04) | 미대조·스토리지 도메인 | 확인 불가+apples |
| METR 0.84×(19% 감속) (QA-09) | ✅ | ✅ | 유지 |

> 미래형 ID 결론: 의심된 26xx는 **현재(2026-06) 유효 과거 ID**. 실재가 문제가 아니라 **ID 실재 + 수치/도메인 오류**(InfEngine)가 진짜 결함.

## ★ 급간 변경 요약 (round-03 → round-04 권고)

| QA | round-03 ★ | round-04 권고 | 사유 |
|---|---|---|---|
| **QA-11** | ★★★ ≥60% / ★★☆ ≥40 / ★☆☆ ≥25 | **★★★ ≥45 / ★★☆ 35~45 / ★☆☆ 25~35** + ②-1 보조축 | voting 메커니즘 반증 — 60% 사문화 |
| **QA-06** | ★★☆ 80~90 (margin 11%p) | **★★☆ 85~90 (margin 5%p 통일)** + FPR 전급간 | margin 규칙화·직접 injection apples |
| QA-07 | ★★★ 92~98 / ★★☆ 90~92 / (98,100)공백 | **★★★ 93~99 / ★★☆ 90~93 / 100%만 불합격** + rework율 보조축 | 공백 제거·★★☆ 확장 |
| QA-04 | ★☆☆ ≥95 / ★★☆ 95~98(모호) | **[95,97)/[97,98)/[98,100)** | ≥/> 모호 제거 |
| QA-12 | ★★★ ≤1 / ★★☆ =2 / ★☆☆ =3(정수) | **≤1 / 1<p95≤2 / 2<p95≤3** + prompt/tool 보조축 | 분수 p95·동률 변별 |
| QA-09 | ★★★ ≥8배(InfEngine 8.6~22.7×) | ≥8배 유지·**인용 21×로 정정+도메인 명시** | 수치 오류 정정 |
| QA-13 | ★★★ ≥75(법무 단일) | ≥75 유지·**도메인무관 분포 재근거**+재시도 보조축 | 단일 사례 탈피 |
| 그 외(01·02·03·05·08·10) | — | 급간 수치 유지·**margin/apples 근거 보강**만 | 방향 통과 |

## 핵심 근거 (레퍼런스)
- **pass@k vs pass^k vs voting**: [Self-Consistency](https://www.emergentmind.com/topics/self-consistency-sampling) · [pass@k 통계](https://leehanchung.github.io/blogs/2025/09/08/pass-at-k/) · [Certified Self-Consistency 2510.17472](https://arxiv.org/pdf/2510.17472) — voting은 다수결로 1답 정확도↑(pass@k), pass^k(전 시행 성공)와 별개.
- **τ-bench**: [arXiv 2406.12045](https://arxiv.org/abs/2406.12045) · [Sierra](https://sierra.ai/blog/benchmarking-ai-agents).
- **Unit 42 가드레일**: [Unit 42](https://unit42.paloaltonetworks.com/comparing-llm-guardrails-across-genai-platforms/) — Table 1·2 직접 확인.
- **InfEngine**: [arXiv 2602.18985](https://arxiv.org/abs/2602.18985) — 21×·적외선 컴퓨팅(인용 정정 근거).
- **METR**: [2025 RCT](https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/)·[2026 업데이트](https://metr.org/blog/2026-02-24-uplift-update/).
- 라이브러리(§4): caching([Anthropic](https://platform.claude.com/docs/en/build-with-claude/prompt-caching)) · USL([WSO2](https://wso2.com/blog/research/measuring-software-scalability-using-universal-scalability-law/)) · OTel([semconv](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/)) · SRE p95([Google SRE](https://sre.google/sre-book/service-level-objectives/)) · durable execution([Temporal](https://temporal.io/blog/what-is-durable-execution)) · KEDA([KEDA](https://keda.sh/)).

## 직전 대비 (round-03 → round-04)
- round-03(reviewer-less ★ 캘리브레이션)이 ★ 급간을 **수립**, round-04 review가 **red-team 검증**(방향 통과·근거 4축 지적), 이 counsel이 **근거를 레퍼런스+PoC로 메움**.
- 가장 큰 변화: **QA-11 ★★★ 60→45% 하향**(review의 가장 날카로운 지적에 1차 출처로 정면 응답) + **InfEngine 인용 정정**(웹 검증이 잡은 수치 오류) + **C3 이중 별점 축 4개 QA 신설**.
- 잔존 트래킹: OI-7(eval/검증 DP — ②-2·golden·red-team 하네스 단일 수렴 후보) · QA-13 OI 등록 확인(Applier) · QAS-07 ★ 참조 추가.

## 후속 (Applier 위임)
- ★ 급간 반영 시 QA `updated:` + `changelog.md` 델타. QA-11 ★★★·QA-06 ★★☆·QA-07 경계·QA-04 표기·QA-12 입도 변경은 **§측정·등급표·변경이력·짝 QAS·counsel 5곳 동기화**(OI-9 정합 — 새 보정이라 재점검 필수).
- QA-09 InfEngine 인용 정정은 ★ 노트 + 변경이력 동기화. QA-13 OI-9 등록 확인.
- 보조 별점 축(②-1·rework·재시도·prompt수정)은 ## 등급 척도에 "보조 축" 행 추가(main 축과 병기).

## seats
Seat 1 Agentic Workflow · Seat 2 20년차 수석 아키텍트 · Seat 3 대규모 Workflow Runner 인프라 — 13개 전 항목 consensus. QA-11 ★★★ 하향은 Seat 1 발의(voting 메커니즘)·Seat 2 합의(이중 축으로 변별력 보강)·Seat 3 동의(②-1 산출 가능). dissent 0.
