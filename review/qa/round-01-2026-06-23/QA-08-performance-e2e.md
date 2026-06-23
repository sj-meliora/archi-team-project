# Review: QA-08 Performance — E2E 개발 시간

> source: `context/qa/QA-08-performance-e2e.md` + `QAS-08-performance-e2e.md`
> verdict: **Sound △ / KPI ✕** — 정의와 KPI가 서로 다른 것을 가리키는 **불일치(mislabel)** · severity **High**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트

## 원문 요약
- **정의**: 응답 시간·처리량 목표 충족 (E2E).
- **KPI**: Artifact 전달 오버헤드 ≤ 전체 E2E의 5%
- **QAS**: 자극=모델 1건 E2E 처리 / 응답=Artifact 전달 오버헤드 최소화.

## 렌즈 1 — Agentic Workflow 전문가 관점

**E2E 시간은 artifact 전달이 아니라 단계 compute + agent 루프 + 큐 대기가 지배한다.** Quantize·Compile은 그 자체로 오래 걸리고, 각 단계에서 agent가 반복 추론하며, 피크엔 큐 대기가 붙는다. Artifact 전달 5%는 overview의 “20GB 수동 공유 Loss” pain을 정조준한 **타당한 하위 목표**지만, 이걸 **E2E 성능의 유일 지표로 삼으면 본질을 놓친다**.
- 실제 개선 레버는 단계 병렬화·agent 반복 횟수 절감·큐 backpressure — 이것들이 E2E KPI에 보여야 한다.
- QA-07(노드 latency)·QA-01(throughput)과 같은 Performance 패밀리 → 분산돼 경계가 흐림.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **정의↔KPI 불일치(C2, 치명적)**: 정의·제목은 “E2E 응답·처리량”(광의)인데 KPI는 “artifact 전달 5%”(협의 하위지표) 1개뿐. **QA가 측정하는 것과 이름이 가리키는 것이 다르다.** 둘 중 하나로 정렬해야 한다:
  - (a) QA를 “**Artifact 전달 효율**”로 개명하고 5%를 유지하거나,
  - (b) 이름을 살리고 **진짜 E2E KPI**(모델당 E2E latency, 처리량)를 추가.
- **실제 E2E 지표 부재**: `E2E latency/모델 ≤ ◯시간(p50/p95)`, `throughput ≥ ◯모델/일` 같은 top-line이 없다.
- **QA-07과 통합 여지(C3)**: Performance를 하나로 묶고 07=per-node speedup, 08=E2E throughput을 sub-metric으로 두면 중복이 해소된다.

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

**E2E는 단계 실행 + 단계 간 handoff + 큐 대기의 합 — handoff는 data-plane 설계 문제다.**
- **artifact는 값이 아니라 참조로 전달**: 20GB를 노드마다 복사하지 말고 **공유 볼륨/오브젝트 스토어에 두고 참조(포인터) 전달**, 의존 단계는 data locality로 co-location. 이것이 “artifact 오버헤드 5%”를 실제로 달성하는 아키텍처(DP-0004 Staging·DP-0005 공유 캐시와 직결).
- **파이프라인 병렬화·critical path**: barrier를 줄이고 임계경로 외 단계를 겹쳐 E2E latency 단축. 큐 대기가 E2E에서 차지하는 비율을 별도 계측(피크에 지배적).
- **runner 측 KPI**: 단계 간 handoff 오버헤드, E2E 중 큐 대기 비율, 임계경로 자원 가동률 → 정의가 약속한 “E2E 응답·처리량”의 실측 지표를 채운다.

## 판정

| 항목 | 판정 | 근거 |
|---|---|---|
| QA 자체가 sound한가 | △ | E2E 성능은 타당한 관심사이나, 현재 내용은 사실상 artifact 전달 QA로 축소됨 |
| KPI가 측정 가능한가 | ✕ | 5%는 측정 가능하나 **정의가 약속한 E2E를 측정하지 않음** |
| KPI가 현실적/적절한가 | △ | 5% 자체는 합리적 하위목표이나 E2E 대표 지표로는 부적절 |
| 정의↔KPI↔QAS 일치 | ✕ | 정의=E2E 광의, KPI=artifact 협의 → 불일치 |

## Stage 2 권고

- **택1로 정렬**:
  - **권장**: 이름·정의 유지 + **E2E KPI 추가** — `E2E latency/모델 ≤ ◯시간(p50/p95)`, `throughput ≥ ◯모델/일`. artifact 5%는 그 **하위 항목**으로 강등.
  - 또는 QA를 “Artifact 전달 효율”로 개명하고 E2E는 QA-07과 통합.
- **Performance 통합 검토(C3)**: QA-07+QA-08을 한 Performance QA로, per-node/E2E 2 sub-metric.
- artifact 5%는 overview pain(20GB Loss)과 직결되니 **신뢰성 지표로도 보존**(전달 성공률·무결성 100%).
- DP 연결 점검: DP-0004(Staging 은닉)·DP-0005(공유 캐시)·FR-0002가 E2E latency까지 책임지는지 역검토.
