# A7 (7안) Microkernel Plug-in Node Backends — 노드 변종 plugin 수용

> category: DP-approach | for: DP-0004 | status: 발굴·평가완료 | updated: 2026-06-20
> 근거 리서치: R-04 | drives: QA-12(주), QA-01(variety), QA-08, QA-10

## 구조
**최소 코어(워크플로 실행·라우팅·관측 커널)** + **노드 타입 백엔드를 plugin으로 분리**한다. IR Converter / Graph Optimizer / Quantizer / Compiler의 **NPU 타겟·모델 패밀리·세대별 변종**을 각각 plugin으로 구현하고, **plugin registry**로 등록·발견한다. 코어는 plugin **계약(표준 API)** 만 알고 호출한다. 새 타겟이 생기면 plugin만 추가 → 코어·타 plugin 불변.

## 근거 tactic/pattern
- **Microkernel / Plug-in architecture**, **Plugin registry**, **Stable plugin contract(API)**, **Configuration-driven extension**.

## 기존 1·2안과의 차별점
- 1·2안·A3·A5는 "동일 노드를 **어디서/얼마나 안정적으로** 실행"을 결정, A6는 "실행 상태 내구성". A7은 **"각 단계의 서로 다른 구현 변종을 어떻게 수용·교체하나"** 라는 **variety 축**을 다룸(직교적 보강). A4(무상태 filter 분해)와 보완: A4=단계 분해, A7=단계 구현 교체.

## QA별 장점 / 단점
- **[Maintainability QA-12] ★★★ (주 강점)**: plugin 독립 진화·동적 추가/제거, 코어 불변 → 한 백엔드 교체가 그 plugin에 국소화(Change Impact Scope ≤2 직접 달성). FR-0003(Config 추적)과도 정합.
- **[Scalability QA-01 — variety] ★★★**: 모델 140+ × 세대 × 단계 변종을 **plugin 카탈로그로 수용**(기능 다양성 확장). ※ 인스턴스 수평 확장은 별도 축 → A5와 결합 필요.
- **[Reliability-Workflow QA-08] ★★☆**: plugin 버그가 코어를 (대체로) 안 죽임 → 격리. ⚠️ 단 in-process plugin이면 격리 한계 → 프로세스/컨테이너 경계 격리 권장(2안·A5와 결합).
- **[Performance-E2E QA-10] ★★☆**: 코어 경량. ⚠️ 단 cross-boundary 호출·plugin 로딩 비용.

## Trade-off 별점
| Performance | Scalability(variety) | Reliability-WF | Maintainability |
|:---:|:---:|:---:|:---:|
| ★★☆ | ★★★ | ★★☆ | ★★★ |

## mini-ATAM
- **SP**: **plugin 계약(API) 안정성**이 QA-12(변종 교체 효과)에 강하게 민감. plugin 격리 경계(in-process vs 컨테이너)가 QA-08에 민감.
- **Risk**: ⚠️ **plugin API가 출시 후 변경 어려움** — breaking change가 plugin 생태계 붕괴 → 초기 계약 설계가 critical. plugin 간 **버전 충돌**, cross-boundary 비용 누적.
- **Non-Risk**: C-01(Docker) — plugin을 컨테이너로 격리·배포 가능. C-02(이식성)도 계약 표준화로 유리.

## 인접 DP 정합
- **A7(변종 수용) + A5(ephemeral 실행) + A6(내구 실행)** = 다양성·확장·복구를 각 축에서 분담하는 강력한 조합. DP-0003(외부 시스템 안정성)의 어댑터도 plugin으로 통합 가능.
