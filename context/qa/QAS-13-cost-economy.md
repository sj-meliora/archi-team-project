# QAS-13 완료 모델당 비용 시나리오

> category: QAS | refines: QA-13 (Cost-economy) | source: discussion/qa/round-01 (신규) | updated: 2026-06-24
> ISO/IEC 25010:2023: Performance Efficiency / Resource Utilization

## 6-Part Quality Attribute Scenario
| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 개발 파이프라인 / 경영(ROI 평가) |
| **자극 (Stimulus)** | 모델 1건의 E2E 완료 (IR 변환 → 최적화 → 양자화 → 컴파일) |
| **대상 (Artifact)** | 비용 집계 계층 (토큰 + compute + 재시도) |
| **환경 (Environment)** | 정상 운영 |
| **응답 (Response)** | 완료 모델당 총비용을 토큰/compute/재시도로 분해 집계, 수작업 baseline 대비 절감 산출 |
| **응답 측정 (Measure)** | 완료 모델당 비용 **≤ $5/모델**(예시), 수작업/기존 대비 절감률 **≥70%**, 비용 분해(토큰비/compute/재시도 오버헤드) 가시화 |

## 비고
- ISO/IEC 25010:2023 Performance Efficiency / Resource Utilization(자원 활용성). QA-05(Efficiency)와 같은 ISO 특성이나 altitude 분리: QA-05=요청당 토큰, QA-13=완료 모델당 총비용·ROI.
- 설계 연결: DP-0001(비용 기준 라우팅)·DP-0004(scale-to-zero)·DP-0005(캐시). 검증은 QA-05 caching A/B 공유 + 수작업 baseline 추정([발표 서사]).
- 흡수: QA-05 top-line(`$/완료모델`)·QA-01 "자원 활용률"을 QA-13로 이양. 데이터는 QA-04 결정당 비용 trace 수급.
- 수치($5·70%)는 측정가능 KPI 예시값 — 인프라 단가·baseline 추정으로 확정.
