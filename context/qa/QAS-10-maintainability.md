# QAS-10 모듈 교체 용이성 시나리오

> category: QAS | refines: QA-10 (Maintainability) | updated: 2026-06-24

| 요소 | 내용 |
|---|---|
| **자극원 (Source)** | 개발자 |
| **자극 (Stimulus)** | 한 구성요소 변경/교체 — 컴포넌트 구현 / SDK 툴체인(플래그·IR 포맷) / prompt·tool-def / **LLM 모델 v→v+1** / 신규 모델 온보딩 |
| **대상 (Artifact)** | 시스템 컴포넌트 (큐/계약으로 분리된 activity) |
| **환경 (Environment)** | 개발/운영 시 (in-flight 워크플로우 존재) |
| **응답 (Response)** | 타 요소로의 영향을 최소화(낮은 결합), 모델 교체는 워크플로우 무중단(버저닝·blue-green) |
| **응답 측정 (Measure)** | CIS **p95 ≤3개**(+컴포넌트 경계 정의), prompt/tool-def 수정 **≤2**, 모델 교체 **무중단(model-agnostic)**, 신규 모델 온보딩 **≤1일** |

## 비고
- 설계 연결: DP-0004(A5/A8 Job/단계 단위 독립 배포), DP-0005(캐시 계층), FR-0003(영향 범위 자동 분석).
- 수치(p95 3개·2·1일)는 예시값 — 합격선은 변경 시나리오 측정으로 확정. 상세는 QA-10 본문.
- agentic 유지보수의 지배적 비용은 prompt/tool-def·모델 교체·온보딩 — 컴포넌트 CIS만으론 부족.
