# Review: NQA-C Cost-economy — 완료 모델당 비용 (round-02 신설 QA 재평가)

> source: `context/qa/NQA-C-cost-economy.md` + `QAS-C-cost-economy.md` (round-01 신설 후)
> 직전 stance(round-01): **신설 — 채택 권장, Med; QA-05 top-line·QA-01 활용률 흡수**
> verdict(round-02): **Sound ○ / KPI △ — Med** — 신설 내용·ISO 앵커는 완성도 높고 정식화 권장. 단 **QA-01/05의 KPI 이양처인데 미채택이면 이양 KPI가 부유** + 주 KPI(절감률)가 수작업 baseline 추정 [발표 서사] → 닫힘 미완 · severity **Med**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 NQA-C = **[반영](신설) + 일부 [발표 서사]**. 이월: baseline 추정 [발표 서사]; KPI 이동 닫기; 번호(OI-8).

## 원문 요약 (신설 후)
- **정의**: 모델 1건 완료 총비용(토큰+compute+재시도)을 수작업/기존 대비 절감. 비즈니스 ROI top-line. QA-05(요청당 토큰)를 sub-metric으로 흡수, QA-04에서 데이터 수급.
- **ISO 앵커**: 주 특성 Performance Efficiency / 하위특성 **Resource Utilization** — QA-05와 같은 ISO 특성이나 altitude 다름(QA-05 per-request, NQA-C business top-line).
- **KPI 3축**: 주 = `완료 모델당 비용($) ≤$5(예시)`. 보조 = `수작업 대비 절감률 ≥70%(ROI 핵심)` · `비용 분해(토큰/compute/재시도)`.
- **QAS-C** 신설.

## 렌즈 1 — Agentic Workflow 전문가 관점

agentic 비용 구조의 정확한 통찰: **요청당 토큰(QA-05)을 줄여도 요청 수·재시도가 폭발하면 모델당 비용은 오를 수 있다** → top-line altitude를 모델 1건 완료로 올리고, 재시도 오버헤드를 분리 집계한 게 핵심. prompt caching(cache read≈10%) 분리·$/완료모델은 LLM 비용 관측 표준. **신설 내용 적절.**

- **잔여(silent cap, round-02)**: 주 KPI 절감률(`≥70%`)이 **수작업 baseline 비용 추정**에 의존하는데 이게 [발표 서사]다 — baseline이 인건비·시간 가정에 민감해 절감률이 가정에 좌우된다. QA-05·QA-01·QA-07·NQA-A/B 모두 "예시값" 한계가 있으나, NQA-C는 **헤드라인(절감률)이 추정 의존**이라 발표 핵심 수치(ROI)가 가장 흔들리기 쉽다 → baseline 가정을 슬라이드에 명시함이 발표 정직성에 결정적.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **정식화 가부(OI-8) = 채택 권장(Med)**: ISO Performance Efficiency / Resource Utilization 앵커 명확. **QA-05와 같은 ISO 특성이나 altitude(per-request vs business)로 분리**한 게 핵심 — taxonomy 모범. 단일 관심사(비용 top-line). **Sound ○ — 정식화 자격 충분.** 단 NQA-A/B(신뢰 두 기둥)보다 우선순위는 후순위(Med, 비즈니스 설득용).
- **KPI △ + round-02 핵심 구조 쟁점 — 이양처 미채택 시 KPI 부유(C2)**:
  - NQA-C는 **QA-05 top-line(`$/완료모델`)·QA-01 "자원 활용률"의 이양처**다. 두 QA 본문은 이미 "NQA-C로 이양/승격"이라 적었다.
  - **NQA-C가 정식 채택 안 되면** → QA-01 활용률·QA-05 top-line이 *원래 QA에선 빠졌고 NQA-C는 임시 ID*라, 두 KPI가 **어디에도 살아 있지 않은 부유 상태**. 즉 NQA-C 미채택은 자기 문제가 아니라 **QA-01·QA-05에 구멍을 남긴다.**
  - → **C2(KPI 이양 닫힘)의 닫힘 트리거 = NQA-C 채택.** NQA-B(QA-07/09 게이트)와 구조가 같다 — 신설 QA가 기존 QA의 닫힘 전제. 다만 NQA-C는 게이트가 아니라 *이양처*라 severity는 Med(부유 KPI는 발표 누락이지 측정 모순은 아님).
- consistency ○: QAS-C 동기화. 채택 시 QA-01/05 이양 cross-link 확정.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

- **비용 분해의 runner 환원**: $/완료모델을 토큰비/compute/재시도로 분해 — **재시도 오버헤드 분리**가 runner 관점 핵심(재시도가 비용 잠식). QA-04 trace(`gen_ai.usage.*`)에서 토큰, compute 단가표, 재시도 카운트로 집계. compute 축에 **QA-01 활용률·QA-05 worker 가동률이 흡수**되는 게 자연스럽다.
- **잔여(DP 위임, OI-7)**: `related-dp: [DP-0001, DP-0004, DP-0005]`로 책임 DP는 있으나 — DP-0001 라우팅이 **비용 기준인지 토큰 기준인지 미명시**(QA-05와 동일 OI-7 항목), DP-0004 scale-to-zero가 compute 비용을 책임지는지 역검토 필요.
- **runner 측 KPI**(제시): scale-to-zero 가동률(과프로비저닝 비용), 재시도 비용 비율(전체 대비), warm pool 절감 기여, 단위 모델당 compute-초·토큰 분해.

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | (신설) | **○** | 비용 top-line 단일 관심사 + QA-05와 altitude 분리(같은 ISO 특성, taxonomy 모범) |
| KPI가 측정 가능한가 | (신설) | **△** | 비용 분해는 측정 가능하나 주 KPI 절감률이 수작업 baseline 추정([발표 서사])에 의존 |
| KPI가 현실적/적절한가 | (신설) | **○** | $/완료모델·재시도 분리 현실적. baseline 추정 한계는 명시됨(슬라이드 가정 명시 필요) |
| 정의↔KPI↔QAS 일치 | (신설) | **○** | QAS-C 동기화. 단 QA-01/05 이양은 채택 시 cross-link 확정 |

**verdict(신설 → round-02): Sound ○ / KPI △ · severity Med.** 정식화 가부 = **채택 권장(Med)**. **미채택 시 QA-01 활용률·QA-05 top-line이 부유**(C2 닫힘 트리거). 주 KPI 절감률은 baseline 추정 의존이라 [발표 서사] 한계.

## Stage 2 권고 (round-02)

- **(정식 채택 — C2 닫힘 트리거, OI-8)** NQA-C 정식 채택이 **QA-01 "자원 활용률"·QA-05 `$/완료모델` 이양을 닫는다.** 미채택이면 두 KPI 부유 → QA-01/05와 이양 cross-link 확정. 우선순위는 NQA-A/B 후순위(Med).
- **(DP 디스커션 위임, OI-7)** DP-0001 라우팅을 **비용 기준 정렬** 명시(QA-05와 공유 항목), DP-0004 scale-to-zero가 compute 비용 책임지는지 역검토.
- **(baseline 가정 명시 — 발표 정직성, Med)** 주 KPI 절감률 `≥70%`의 **수작업 baseline 가정(인건비·시간)을 슬라이드에 명시** — ROI 핵심 수치가 추정 의존임을 솔직히. 가장 흔들리기 쉬운 헤드라인.
- **(예시값 확정)** `$5/모델·70%`는 인프라 단가·baseline으로 확정(레퍼런스 복제 아님).
