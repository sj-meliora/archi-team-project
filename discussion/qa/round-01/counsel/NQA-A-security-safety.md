# Counsel: NQA-A Security / Safety (신규 QA 권고)

> refs-review: round-01/review/_new-qa-candidates.md (NQA-A) · report.md(C5)
> seats: 발의 Seat 1(Agentic Workflow) · 합의 consensus (Seat 2 1급 QA 정당화, Seat 3 게이트 인프라)
> stance: 신설 — 채택 강력 권장 (우선순위 상위 진입)

## Reviewer 지적 요약
- 시스템이 "사람 없이" 빌드·**배포**·이슈처리 → 자율 에이전트가 **배포 권한·자격증명·외부 도구 실행권**을 쥠 = 신종 공격면(prompt injection으로 악성 config 배포, 자격증명 유출, 공급망 변조). 현재 어떤 QA도 1급으로 안 다룸.
- QA-03(Controllability) ⊂ Security — 제어성은 "허용범위 내 동작"일 뿐 secrets·공급망·injection 방어·감사를 미포함.

## 개선안 (정의·KPI)

**정의 (신설)**: 자율 에이전트가 권한·자격증명·공급망을 안전하게 다루며, 비인가/주입 공격으로 시스템을 훼손하지 않는다.

**KPI**
| # | 제안 | 비고 |
|---|---|---|
| ① | **권한 상승·범위 외 배포 = 0** (적대적 eval 기준) | PoC-N-A1 |
| ② | **고위험 액션(deploy/delete/credential) HITL 게이트 통과율 100%, 우회 0** | QA-03 ④와 공유 |
| ③ | **Artifact 서명·무결성 검증 통과율 = 100%** (공급망) | PoC-N-A1 |
| ④ | **Prompt-injection red-team 차단율 ≥ ◯%** | ◯는 PoC로 확정 |
| ⑤ | **Secrets 노출(로그·trace 포함) = 0** | QA-04 trace와 교차 |

## 근거 (레퍼런스)
- **zero-trust 최소권한 + 고위험 HITL + 별도 가드레일 모델 + injection 방어**: 자율 에이전트 보안의 필드 표준 — Anthropic *Building Effective Agents* / 신뢰 에이전트 프레임워크. (§4 제어·안전) — https://www.anthropic.com/engineering/building-effective-agents , https://www.anthropic.com/news/our-framework-for-developing-safe-and-trustworthy-agents
- **위반율·injection 차단율은 적대적 eval로**: red-team eval 세트 통과율로만 검증. (§4 제어·안전 PoC 수단)
- **공급망 서명**: artifact 서명·무결성 검증은 SLSA/공급망 보안 표준(보강 검색). — https://slsa.dev/
- ⚠️ 차단율 ◯%는 우리 red-team으로 확정.

## PoC 증명법

### PoC-N-A1: red-team eval로 권한 위반·injection·공급망 무결성을 증명한다 (eval 하네스, 적대적)
- **가설**: "적대적 세트에서 권한상승·범위 외 배포 0, injection 차단율 ◯%, artifact 무결성 100%."
- **지표**: 권한위반 통과율(=0), injection 차단율, HITL 우회 건수(=0), artifact 서명 검증 통과율(=100%), secrets 노출 건수(=0, 로그/trace 스캔).
- **셋업**: red-team 프롬프트/시나리오(injection으로 악성 config 배포 유도·자격증명 탈취·비인가 도구 호출) + 최소권한 allowlist + HITL 게이트 + artifact 서명 검증 + secrets 스캐너.
- **절차**: ① 적대적 세트 실행 → ② 차단/통과 라벨링 → ③ 서명 변조 artifact 주입해 검증 차단 확인 → ④ 로그·trace에서 secrets 패턴 스캔.
- **합격(Exit)**: 위반·우회·노출 0, injection 차단율 ≥ ◯%, 무결성 100%.
- **규모/기간**: 적대적 앵커 50~100건 + 공급망 시나리오 수 건, 약 3~4일.
- **리스크/한계(silent cap)**: red-team 세트 커버리지 = 신뢰도 상한(zero-day·미상상 공격 미검출) → 세트 출처·OWASP LLM Top-10 등 매핑을 log. 실제 자격증명 시스템(vault) 통합은 별도 환경.

## DP·발표 영향
- **DP 연결**: DP-0003(1안 사전 권한 게이트 + 2안 격리)이 ①②③의 인프라. PoC-C2(QA-03)와 PoC-N-A1은 **하나의 적대적 eval 하네스로 통합** — 제어성·보안 공유 자산.
- **번호/서사(C5)**: NQA-A 채택 시 **우선순위 상위 진입**(QA 2자리 번호 재정렬) — Security + Correctness가 "사람 없이 믿고 맡길 수 있는가"의 두 기둥. 발표 설득력 직결.
- **경계**: QA-03(Controllability)을 NQA-A의 부분집합으로 명문화(양쪽 cross-link).
