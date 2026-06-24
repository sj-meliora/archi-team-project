# QAS-10 E2E 개발 시간 시나리오

> category: QAS | refines: QA-10 (Performance-E2E) | updated: 2026-06-24 (round-03 ★ 급간 동기화)

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 개발 파이프라인 |
| **자극 (Stimulus)** | 모델 1건의 E2E 처리 (IR 변환 → 최적화 → 양자화 → 컴파일) |
| **대상 (Artifact)** | 전체 파이프라인 (단계 compute·agent 루프·큐·artifact 전달 경로) |
| **환경 (Environment)** | 정상 운영 / 부하 |
| **응답 (Response)** | 목표 E2E latency·throughput 내 처리, artifact는 참조 전달로 오버헤드·유실 최소화 |
| **응답 측정 (Measure)** | E2E latency/모델 ≤24시간(p50/p95; 6h=★★☆), throughput **≥50모델/일**, artifact 전달 오버헤드 **≤E2E의 5%**(하위), 전달 성공률·무결성 **100%** |

## 비고
- 설계 연결: DP-0004(A5 claim-check / A8 로컬 — 5% 산식으로 택일), DP-0005(2안 공유 캐시로 E2E 단축).
- 수치(24시간·50모델/일·5%)는 예시값 — 합격선은 부하시험으로 확정. 상세는 QA-10 본문. (★ 급간은 QA-10 등급 척도)
- Performance 3분할: throughput은 QA-01과 공유 축, per-node는 QA-09. 전달 무결성은 overview 20GB Loss pain 직결.
