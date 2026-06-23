# OV-0001 시스템 개요

> category: overview | source: pptx p.1~9 | updated: 2026-06-19

## 시스템명
Agentic AI 기반 On-device AI SDK 개발 자동화 시스템 (Agentic DevOps System)

## 한 줄 정의
NPU 구동용 On-device AI SDK(IR Converter → Graph Optimizer → Quantizer → Compiler)의 개발 파이프라인을, Agent가 사람 개입 없이 빌드·검증·배포·이슈처리까지 자동화하는 시스템.

## 배경 3축
| 축 | 내용 | 함의 |
|---|---|---|
| 수요 | On-device AI 중요성 증가, 추론이 Cloud → Edge로 이동 | NPU가 제품 경쟁력 핵심 자산 → SDK가 가치 실현의 critical path |
| 워크로드 | AI 모델 폭발적 증가 (모델 140개+, 단계별 iteration 다수) | 수작업·선형 인력으로 감당 불가 |
| 방법론 | Agentic AI 성숙 — LLM이 도구 사용·계획·실행 가능 | 개발 프로세스 자동화 가능 |

## 핵심 문제 (Pain Point)
- 대용량 산출물(20GB+) 수동 공유 과정에서 파일 Loss 등 신뢰성 이슈 → 연결: FR-0002
- Tool Version/Config가 개발자 개인별 관리 → 추적성·Regression 확보 어려움 → 연결: FR-0003
- Jira 상태가 개발자 수동 업데이트 의존 (속도 때문에 채팅으로 우회)
- 조합수 폭발: 모델 수 × 세대 수 × 파이프라인 단계 → 연결: QA-01
- 분석 도구 한계: 고정 전체 보기만 제공, 커스텀 뷰 불가
