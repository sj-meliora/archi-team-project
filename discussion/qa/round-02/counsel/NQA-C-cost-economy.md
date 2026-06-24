# Counsel: NQA-C Cost-economy — 완료 모델당 비용 (round-02)

> refs-review: [round-02/review/NQA-C-cost-economy.md](../review/NQA-C-cost-economy.md) · [report.md](../review/report.md)(C2) · 이양원: [QA-01](../review/QA-01-scalability.md)·[QA-05](../review/QA-05-efficiency.md)
> 직전 counsel: [round-01/counsel/NQA-C-cost-economy.md](../../round-01/counsel/NQA-C-cost-economy.md) (신설 — 권장 Med, QA-05/01 KPI 흡수)
> seats: 발의 **Seat 2**(수석 아키텍트) · 합의 **consensus** (Seat 1 baseline 추정 정직성 / Seat 3 재시도 비용 분리·scale-to-zero)
> stance: **정식 채택 권장(Med, OI-8) — 채택이 QA-01 활용률·QA-05 top-line 부유 해소의 트리거(C2)**

## Reviewer 지적 요약

- round-02 verdict: **Sound ○ / KPI △ · Med.** ISO Performance Efficiency / Resource Utilization 앵커 명확, QA-05와 같은 ISO 특성이나 **altitude 분리**(per-request vs business top-line)가 taxonomy 모범 → 정식화 권장.
- **C2 핵심 잔여(이양처 미채택 부유)**: NQA-C는 **QA-01 "자원 활용률"·QA-05 `$/완료모델` top-line의 이양처**. **미채택이면 두 KPI가 원 QA에도 NQA-C에도 안 살아 있는 부유 상태** → NQA-C 채택이 닫힘 트리거(게이트 아닌 이양처라 severity Med).
- 잔여: 주 KPI 절감률(`≥70%`)이 수작업 baseline 추정 [발표 서사] — ROI 헤드라인이 가장 흔들리기 쉬움(슬라이드 가정 명시 결정적), DP-0001 비용 기준 라우팅 미명시(OI-7).

## 개선안 (정의·KPI 기존→제안)

Council 응답: **새 KPI 발명이 아니라 (a) 정식 채택으로 QA-01/05 이양 부유 해소, (b) baseline 가정을 슬라이드에 명시해 ROI 정직성 확보, (c) 비용 기준 라우팅 DP 위임**. Sound ○·정의 유지.

**정의: 기존 → 제안 (유지 + 이양 cross-link)**
- 기존: 모델 1건 완료 총비용(토큰+compute+재시도) 절감, 비즈니스 ROI top-line. QA-05를 sub-metric 흡수, QA-04에서 데이터 수급 — 유지.
- 제안: 채택 시 **QA-01 활용률·QA-05 `$/완료모델` 이양 cross-link 확정**(양방향). compute 축에 QA-01 활용률·QA-05 worker 가동률 흡수 명문화.

**KPI: 기존 → 제안**

| # | 기존 (round-01 신설) | 제안 (round-02) | 닫힘 상태 |
|---|---|---|---|
| 주 | 완료 모델당 비용($) ≤$5(예시) | 동일 — QA-04 trace(`gen_ai.usage.*`)+compute 단가+재시도 카운트로 집계 | 측정 가능 |
| 보조1 | 수작업 대비 절감률 ≥70% (ROI 핵심) | 동일 — **수작업 baseline 가정(인건비·시간)을 슬라이드 명시**(추정 의존 솔직히, Med). 가장 흔들리기 쉬운 헤드라인 | △(추정 의존) |
| 보조2 | 비용 분해(토큰/compute/재시도) | 동일 — **재시도 오버헤드 분리**가 runner 관점 핵심(재시도가 비용 잠식) | 측정 가능 |

> ⚠️ `$5/모델·70%`는 "측정 가능 KPI의 모양" 예시값 — 인프라 단가·baseline으로 확정(레퍼런스 복제 아님).

## 근거 (레퍼런스)

§4 라이브러리 **비용·토큰 효율(Cost/Efficiency)** 행 직접 적용 + ISO 앵커.

- **raw token 아닌 $/task; cache read=정상가 10%(≈90%↓) 분리 집계** — top-line altitude를 모델 1건 완료로 올리고 재시도 오버헤드 분리. 요청당 토큰(QA-05) 줄여도 요청 수·재시도 폭발하면 모델당 비용 상승 가능.
  - Anthropic prompt caching·pricing: https://platform.claude.com/docs/en/build-with-claude/prompt-caching · https://platform.claude.com/docs/en/about-claude/pricing
- **ISO/IEC 25010 Performance Efficiency / Resource Utilization** — QA-05와 같은 ISO 특성이나 altitude(per-request vs business)로 분리한 taxonomy 모범. 정식화 앵커.
  - ISO 25010: https://iso25000.com/index.php/en/iso-25000-standards/iso-25010
- **토큰 계측 = OTel GenAI semconv** — `gen_ai.usage.*`(input/output 토큰)을 QA-04 trace에서 수급해 비용 분해. compute 단가표·재시도 카운트와 합산.
  - OTel GenAI semconv: https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/

> 레퍼런스 수치(90%↓·$5 등)는 패턴 정당화용 — 합격선은 PoC 비용 A/B로 확정.

## PoC 증명법

### PoC-N-C1(R2, QA-04 계측 후행): $/완료모델을 토큰/compute/재시도로 분해하고 절감률을 baseline 대비 측정한다 (비용 A/B+계측)

- **가설(Hypothesis)**: "완료 모델당 비용이 토큰·compute·재시도로 분해 측정 가능하며, prompt caching On/Off A/B로 절감을 실측한다. 절감률은 명시된 수작업 baseline 가정 하에서만 의미를 가진다."
- **지표(Metric) + 합격선**:
  - $/완료모델 분해(토큰비/compute/재시도) — 재시도 비용 비율 분리
  - prompt caching On/Off A/B로 $/task·TTFT 절감
  - 수작업 대비 절감률(**baseline 가정 명시 — 인건비·시간**)
- **셋업(Setup)**: QA-04 OTel 계측(`gen_ai.usage.*`) 토큰 수급 + compute 단가표 + 재시도 카운트 + prompt caching On/Off A/B. scale-to-zero 가동률 측정.
- **절차(Procedure)**:
  1. QA-04 trace에서 토큰 집계 + compute-초·재시도 카운트 → $/완료모델 분해
  2. caching On/Off A/B로 절감 실측
  3. 수작업 baseline 가정(인건비·시간) 명시 → 절감률 산출(가정 의존 log)
- **합격 기준(Exit)**: $/완료모델이 3축 분해로 산출됨 ∧ caching 절감 유의 ∧ **baseline 가정이 슬라이드에 명시**(ROI 정직성). QA-01/05 이양 KPI가 이 PoC로 살아남(C2 부유 해소 시연).
- **규모/기간(Scope)**: QA-04 계측(PoC-O1) 후행 — 토큰 계측 재사용 → 약 2~3일.
- **리스크/한계(silent cap)**: 절감률은 수작업 baseline 추정에 좌우(인건비·시간 가정에 민감) — ROI 헤드라인이 가장 흔들리기 쉬움. compute 단가가 self-host vs 외부 API로 갈림. **PoC는 정식 채택(OI-8)을 대체 못함** — QA-01/05 부유 해소는 채택 동반.

## DP·발표 영향

- **DP 연결 (OI-7, DP 디스커션 위임)**: `related-dp: [DP-0001, DP-0004, DP-0005]`로 책임 DP는 있으나 — **DP-0001 라우팅이 비용 기준인지 토큰 기준인지 미명시**(QA-05와 동일 OI-7 항목). 비용 효율 KPI인데 라우팅이 토큰 기준이면 정렬 어긋남 → "비용 기준 정렬" DP 위임. DP-0004 scale-to-zero가 compute 비용 책임지는지 역검토.
- **동반 닫힘 효과 (C2 트리거)**: NQA-C 정식 채택이 **QA-01 활용률·QA-05 `$/완료모델` 이양을 닫는다**(미채택이면 두 KPI 부유). NQA-B(게이트)와 구조가 같으나 NQA-C는 *이양처*라 severity Med(부유 KPI는 발표 누락이지 측정 모순 아님).
- **번호/서사 (OI-8)**: NQA-A/B(신뢰 두 기둥) 후순위(Med, 비즈니스 ROI 설득용). 채택 시 QA-01/05 이양 cross-link 확정. 발표에서 "수작업 대비 비용 절감 = ROI" top-line으로 사용(baseline 가정 명시 전제).
