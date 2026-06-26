# DP-02는 왜 드랍됐나 — 판단 기록 (decision log)

> 작성 2026-06-26 · 대상: 17조 팀원 · 맥락: DP-02 자문 라운드(특화 → 판단 배치)의 종결.
> 용도: DP-02를 **공식 드랍**한 사유를 추적 가능하게 보존(증발 방지). 산출물(`context/dp/DP-02/`)은 삭제됨 — **이 메모가 그 결정의 유일한 기록**이다. 짝 자료: [`agent-basics-and-dp1-retrospective.md`](agent-basics-and-dp1-retrospective.md)(merged-DP1 회고).

---

## 한 줄 결론
> **DP-02는 어떤 ASR(QA-01~07)도 *positively* cover하지 못해 드랍한다. 그 탐색이 가리킨 진짜 ASR 결정은 NDP-A(eval/검증·진단 서브시스템 — orphan QA-06·QA-07)다.**

## DP-02가 뭐였나
- 최초 framing: "Worker agent를 노드별 **특화**(IRGen/Optimizer/Quantizer/Compiler 전용) vs 단일 **범용** 으로 둘 것인가" — DP-01(제어평면, 오케 有無)의 짝 결정(축 B).
- SEED: [`agent-basics-and-dp1-retrospective.md`](agent-basics-and-dp1-retrospective.md) Part 1(특화/범용 축별 비교).
- 1차 산출: 특화/범용/하이브리드 3안 + 별점 매트릭스(QA-07 Correctness·QA-06 Security ↔ QA-12 Maintainability), 2안 steelman(self-consistency·dual-LLM 등)까지 고도화.

## 붕괴 사슬 — 왜 "살아있는 ASR 결정"이 아니었나
연속 자문에서 driving ASR이 차례로 떨어져 나갔다:

1. **QA-07 Correctness(ASR) 탈락** — SDK 노드(IRGen/Opt/Quant/Compile)는 사실상 **결정적 CLI 도구**다(`IRGen.exe --model X`). happy-path 산출물의 정확성은 *도구*가 보장한다 → 에이전트 특화의 QA-07로 단 것은 **범주 오류**(에이전트는 정확성을 *만드는* 게 아니라 *호출*한다).
2. **QA-06 Security(ASR) 탈락** — 좁은 도구 표면(보안)은 **DP-03**(권한 게이트·격리·allowlist 집행) 소관이다. 노드별 allowlist는 DP-03의 *입력*일 뿐, DP-02가 별점으로 살 게 아니다.
3. **불행-경로 판단 = worker 비소유** — "다음에 뭘 할까(회귀·재계획)"는 **DP-01 오케 / DP-04 엔진**의 몫이고, "이 에러가 무슨 뜻인가·이 파라미터가 맞나"(진단)는 실행과 **분리 가능**하다(actuator ↔ evaluator: 수행은 범용 thin caller, 진단은 별도 분석 agent). → **실행 worker는 범용 thin caller로 dominated.**

재정의("도메인 판단의 배치 — 실행 worker vs 전용 분석 agent vs 오케")까지 밀어붙였으나, 그 결정의 교환점은 **컨텍스트 국소성(핸드오프) ↔ 유지보수 ↔ 오케 비대(god-object)** — **전부 비-ASR(QA-10 E2E · QA-12 Maintainability · QA-13 Cost)** 이었다.

## 결정적 기준 — ASR test
- **ASR = QA-01~07**(아키텍처 핵심 요구). **QA-10/12/13은 ASR이 아니다.**
- Council §B.2 coherence: 전 ASR이 cover돼야 하고 orphan = 정합 구멍. **그 역(逆)**: *어떤 ASR도 cover 못 하는 DP는 발표 핵심으로 설 자격이 없다.*
- QA-05 Efficiency(ASR)로 헤드라인을 갈아끼워 구제할 여지는 있었다(2안 분리 → thin caller 저가 + 진단만 right-size = 토큰 효율). 그러나 그건 **부수효과**지 결정의 본질이 아니며, 슬라이드를 정당화하려 ASR을 지어내는 **안티패턴**이라 기각.
- → **드랍.**

## 탐색이 헛되지 않은 이유 — 진짜 결정의 위치를 찾음
ASR을 떼어낼 때마다 *그 ASR이 실제로 사는 곳*이 드러났다:
- **QA-06·QA-07은 시스템 레벨의 eval/검증·진단 서브시스템 = NDP-A**가 cover한다(golden-set 게이트·LLM-as-judge·red-team 하네스). 이건 현재 **orphan ASR**(어느 DP도 안 덮음 — `discussion/dp/round-01/review/_new-dp-candidates.md`, OI-7).
- DP-02에서 *끝까지 살아남은* "전용 분석/진단 agent"는 NDP-A의 **진단 arm**이다.
- → 에이전트-레벨 특화(비-ASR)는 죽고, **서브시스템-레벨 eval/진단(ASR QA-06·QA-07)** 이 그릴 가치가 있는 결정으로 승격 후보가 됐다.

## 산출물 삭제 후 행선지 (후속 — 본 메모로 드랍은 종결, 아래는 미반영 TODO)
- **"worker = 범용 thin caller"** 결론 → DP-01(제어 scope)·DP-04(실행구조)의 **cohesion 한 줄**로 흡수 가능.
- **"분석/진단 agent"** → **NDP-A**로 이전.
- **dangling 링크**: DP-01 A6의 "DP-02(특화) value interaction" 교차참조는 DP-02 삭제로 끊긴 링크가 됨 → DP-01 갱신 시 정리 필요.

## 교훈 (처방)
1. **DP는 ASR(QA-01~07)을 *positively* 구동해야 슬라이드를 받는다.** 비-ASR(유지보수·비용·지연) 최적화는 실재해도 *핵심 결정*이 아니다.
2. **mis-scoped ASR을 떼어내는 건 실패가 아니라 기능** — 떼어낸 자리가 *진짜 결정의 위치*를 가리킨다(여기선 NDP-A).
3. **"도구 vs 에이전트" 경계를 먼저 그어라** — 결정적 CLI의 정확성/보안을 에이전트 QA로 청구하면 DP가 부풀려진다.
4. **드랍도 결정이다** — 사유를 남겨 다음 라운드가 같은 길을 다시 걷지 않게 한다.

## 한 줄 회고
> **"특화가 필요해 보였는데, 진짜 필요한 건 worker 특화가 아니라 *진단 서브시스템(NDP-A)* 이었다."** — merged-DP1의 "필연처럼 보였는데 아니다"와 같은 결의 ATAM 발견.
