# R-04 리서치 — Microkernel (Plug-in) Architecture

> category: DP-research | for: DP-0004 | updated: 2026-06-20
> 목적: 노드 단계 구현의 **변종 다양성(variety)** — NPU 타겟·모델 패밀리·세대별로 다른 Converter/Quantizer/Compiler 백엔드 — 를 흡수하는 구조 패턴 조사.

## 조사한 패턴 — Microkernel / Plug-in
- **정의**: 최소 **코어(microkernel)** + 선택적 **plugin 모듈**로 분리. 코어는 핵심 기능(실행·라우팅·관측)만 담고, plugin이 독립·standalone 기능을 추가. 코어는 **plugin registry**로 사용 가능한 plugin을 발견하고 **계약(API)** 으로만 호출한다. "Plug-in Architecture"로도 불리며, **확장성이 핵심 품질속성인 product-based 시스템**의 정석 선택.
- **장점**:
  - **Maintainability / Extensibility**: plugin이 독립 진화, 기능 동적 추가/제거, **코어 불변**.
  - **Robustness / Isolation**: plugin 버그가 코어를 (대체로) 안 죽임.
  - **Customization**: 고객/타겟별로 다른 plugin 세트 구성.
- **트레이드오프**: ⚠️ **robust plugin API 설계가 어렵고, 출시 후 변경이 어려움**(breaking change가 plugin 생태계 붕괴). cross-boundary 호출 비용, plugin 간 **버전 충돌**.
- **적합 조건**: 확장성·변종 수용이 핵심 QA인 시스템, feature set이 계속 진화하는 product.
- 출처: [Microkernel Architecture (Software System Design)](https://softwaresystemdesign.com/software-architecture-design/architectural-patterns/microkernel-architecture/), [Microkernel (csse6400 lecture)](https://csse6400.uqcloud.net/handouts/microkernel.pdf)

## DP-0004에의 함의
- 본 시스템은 **모델 140+ × 세대 × 파이프라인 단계**(overview) → 각 단계의 **구현 변종이 폭발**. 1·2안·A3~A6는 "동일 노드를 어디서/얼마나 안정적으로 실행"에 집중하지만, **"각 단계의 서로 다른 구현을 어떻게 수용·교체"** 는 다루지 않음.
- Microkernel: **코어 = 워크플로 실행 커널**, **plugin = 노드 타입 백엔드(IR Converter/Quantizer/Compiler 변종)**, plugin registry로 등록·발견 → 새 NPU 타겟/모델 패밀리를 **plugin 추가만으로** 흡수, 코어·타 plugin 불변.
- A4(Pipes-Filters, "단계를 무상태 filter로 분해")와 보완 관계: **A7은 "각 단계의 구현 변종을 plugin으로 교체·확장"** — variety 축에 초점.

→ 도출 대안: **[A7] Microkernel Plug-in Node Backends**.
