# QAS-05 토큰·자원 효율 시나리오

> category: QAS | refines: QA-05 (Efficiency) | source: pptx p.25 | updated: 2026-06-24

## 6-Part Quality Attribute Scenario
| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | Workflow 노드가 Agent에 작업 요청 |
| **자극 (Stimulus)** | 단일 노드 작업 판단 요청 (예: 빌드 결과 검증 후 다음 행동 결정) — 빌드로그·config diff 등 대형 입력 포함 |
| **대상 (Artifact)** | Agent (LLM Endpoint) + runner(worker pool) |
| **환경 (Environment)** | 정상 운영 중 일반 Workflow 실행 |
| **응답 (Response)** | prompt caching·context 압축을 활용해 신규 토큰을 최소화하며 작업을 완료, worker는 재사용(warm pool) |
| **응답 측정 (Measure)** | **신규 토큰 ≤ 6k** AND 캐시 토큰 별도 집계(총 토큰=입력+출력); 난이도별 tier(★★★ ≤4k / ★★☆ 4k~6k / ★☆☆ 6k~8k); worker 가동률·task당 compute 비용 |

## 비고
- 측정 척도 상세·근거·검증 전략은 QA-05 본문.
- 수치(6k·tier)는 측정가능 KPI의 예시값 — 합격선은 실환경 A/B로 확정.
- 설계 연결: DP-0001(2안 작업별 최적 Agent로 토큰 절감), DP-0005(2안 공유 캐시=캐시 토큰 분리집계), DP-0003(권한 게이트는 토큰 미소모).
- top-line `$/완료모델`은 QA-13(Cost-economy)로 승격. 결정당 비용/토큰 측정은 QA-04가 공급.
