# DP-0004 dp4 보강작업 리뷰 — 색인 / 총평

> category: DP-review | reviewer: 수석 아키텍트 관점(20년차 가정) | for: dp4(A3~A7 + evaluation) | updated: 2026-06-20
> 대상: `context/dp/dp4/` 전체 — research(R-01~04), approaches(A3~A7), evaluation.md, INDEX.md
> 교차검증 근거: DP-0004 본문, DP-0001/0002/0005, QA-0002/0006/0008/0010(+0007/0009), FR-0001/0002, overview, open-issues

## 한 줄 총평
**대안 발굴(divergence)은 인증과정 수준을 넘어선다. 그러나 ATAM의 본령인 의사결정 수렴(convergence)과 "조합의 복합 trade-off" 검증이 비어 있어, 현 상태로 발표하면 '좋은 패턴을 다 모았다'는 과설계(over-engineering) 비판에 노출된다.**

## 잘한 점 (유지·강조할 것)
1. **직교 4축 재구성**(evaluation.md): 1안 vs 2안을 단일 택1이 아니라 실행위치/분배/분해/변종·복구의 직교 축으로 분해한 통찰은 성숙한 아키텍처 사고다. 발표의 핵심 메시지로 살릴 가치가 있다.
2. **패턴 근거의 엄밀성**: 각 안이 출처 있는 검증된 패턴(Azure, Temporal, Knative, Microkernel)에 정박. R-01~04 → A3~A7 추적이 명확.
3. **mini-ATAM 일관 구조**(SP/Risk/Non-Risk)가 기존 DP 문서 양식과 정합. ⚠️ 표기로 약점을 스스로 노출한 점도 정직하다.
4. **claim-check를 20GB+(FR-0002) 제약에 정확히 결합** — 실제 적합한 적용.
5. 인접 DP(DP-0001/0002/0005)·FR 교차링크가 대체로 존재.

## 총평 판정
- 발굴 단계로서: **합격**. 추가 대안 5개는 충분하고 근거가 탄탄하다.
- 결정 산출물로서: **미흡**. 아래 CRITICAL 5건이 해소되기 전에는 "권고 스택"을 결론으로 제시하면 안 된다.

## 심각도별 발견사항 요약 (상세: findings.md)
| ID | 심각도 | 한 줄 요약 |
|---|---|---|
| F-01 | **CRITICAL** | Performance(E2E) 별점이 신규 5안 모두 ★★☆로 동일 → 변별력 0, 조합 시 latency 합산 미분석 |
| F-02 | **CRITICAL** | 권고 스택의 **복합 QA 프로파일 미산출** — 창발적 복잡도 무시, Maintainability는 조합 시 오히려 악화 |
| F-03 | **CRITICAL** | A3(Choreography) + A6(Orchestration) 동시 채택 = **이중 제어평면** 상충 (R-03이 대비 관계로 명시) |
| F-04 | **CRITICAL** | claim-check가 곧 QA-0008의 '전달 오버헤드'인데 모든 안이 **순이익으로 오인** — 2안의 핵심 강점을 버리는 비용 미정량 |
| F-05 | **CRITICAL** | 멱등 가정이 A3/A4/A5/A6의 load-bearing인데 시스템 자체 **QA-0009 재현율≥80%(비결정성)와 모순** |
| F-06 | HIGH | QA-0002 throughput KPI(시간당 모델수) 미커버, Change Impact ≤2 **주장만 하고 미실증** |
| F-07 | HIGH | QA 중요도(H/M/M/L) **가중 미적용** — 다수 안이 최저중요도 Maintainability(L)에 과최적화 |
| F-08 | HIGH | A7의 'variety'를 Scalability(QA-0002)로 표기 = **범주 오류·이중계상** (별점 인플레) |
| F-09 | HIGH | 공유 실패도메인(broker/orchestrator/cache) **누적** vs 중단≤1% — blast-radius 분석 부재 |
| F-10 | HIGH | **의사결정 수렴 부재** — phasing/MVA/배제기준 없이 '다 좋다' 스택 (발표 리스크) |
| F-11 | MEDIUM | 비용·운영복잡도·팀 역량 축 부재 — 4~5개 신규 인프라 도입 리스크 미반영 |
| F-12 | MEDIUM | 추적성: DP-0004 본문이 A3~A7 **역참조 안 함**, driving-QA 범위 불일치, FR-0001 중복 realizes 미해소 |

## 읽는 순서
1. 이 파일(INDEX) — 총평·심각도 맵
2. `findings.md` — F-01~F-12 상세 (현상 / 왜 문제 / 보강안)
3. `action-items.md` — 우선순위 punch-list (무엇을, 어떤 순서로 보강)

## 리뷰어 권고(요지)
ATAM은 결국 **버릴 것을 정하는 활동**이다. 지금은 divergence가 끝났을 뿐 convergence가 시작되지 않았다.
다음 iteration의 목표는 "새 대안 추가"가 아니라 **(a) 권고 스택을 복합 trade-off로 검증하고, (b) 시스템의 지배 드라이버(Scalability=H, Reliability=M)에 맞춰 1~2개 패턴으로 수렴**시키는 것이어야 한다.
