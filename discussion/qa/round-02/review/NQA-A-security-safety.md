# Review: NQA-A Security / Safety — 자율 에이전트 보안·안전 (round-02 신설 QA 재평가)

> source: `context/qa/NQA-A-security-safety.md` + `QAS-A-security-safety.md` (round-01 신설 후)
> 직전 stance(round-01): **신설 — 채택 강력권장, 우선순위 상위 진입**
> verdict(round-02): **Sound ○ / KPI △ — Med** — 신설 내용·ISO 앵커·KPI 5축은 형식적으로 완성도 높음(정식화 가부 = 채택 권장). 단 **주·다수 KPI의 검증이 [발표 서사](red-team 세트 미구축)** + 번호 미확정·DP 보안 tactic 부재 → KPI 닫힘 미완 · severity **Med**
> lenses: (1) Agentic Workflow 전문가 · (2) 20년차 수석 아키텍트 · (3) 대규모 Workflow Runner 인프라 아키텍트
> disposition 확인: applier report §1 NQA-A = **[반영](신설) + eval [생략]**. 이월: red-team 세트 구축 [생략]; 번호 재정렬(OI-8); DP 보안 tactic(OI-7).

## 원문 요약 (신설 후)
- **정의**: 자율 에이전트가 권한·자격증명·공급망을 안전하게 다루며 비인가·injection 공격에 시스템 훼손 없음. `Controllability(QA-03) ⊂ Security` 경계 명문화.
- **ISO 앵커**: 주 특성 Security(기밀성·무결성·부인방지·책임추적성·인증성) + Safety. KPI↔하위특성 매핑 명시.
- **KPI 5축**: 주 = `권한 상승·범위 외 배포=0(적대적 red-team eval)`. 보조 = `HITL 통과율=100%·우회=0(QA-03 공유)` · `artifact 서명·무결성=100%` · `injection 차단율 ≥95%` · `secrets 노출=0`.
- **QAS-A** 신설.

## 렌즈 1 — Agentic Workflow 전문가 관점

이 QA는 **agentic 시스템에서만 존재하는 신종 공격면**(배포 권한·자격증명·외부 도구 실행권을 쥔 자율 에이전트)을 1급으로 끌어올렸다 — round-01 C5의 가장 강한 발굴. injection→악성 배포·자격증명 유출·공급망 변조라는 위협 모델이 정확하고, 방어가 zero-trust 최소권한 + HITL + 가드레일 + injection 내성 + 공급망 서명의 다층으로 설계된 것도 필드 표준(Anthropic·SLSA 앵커)에 부합. **신설 내용 자체는 강건하다.**

**round-02 핵심 쟁점 — 검증이 통째로 [발표 서사]:**
- 주 KPI(`권한 상승·범위 외 배포=0`)·injection 차단율·서명 무결성·secrets 노출이 모두 **red-team eval 세트 통과율**로만 측정되는데, 그 세트(50~100건) 구축·실행은 [생략]/[발표 서사]다. 즉 **5개 KPI 중 4개의 측정 수단이 살아 있지 않다**. round-01 applier §3-1 "발표 서사로 미룬 검증이 유효한 설계인가"의 핵심 대상.
- **재판정**: red-team 하네스를 *모듈 다이어그램의 박스*로 존치하는 건 설계 산출물로서 유효(OK). 그러나 "0건·95%"라는 **acceptance가 실측으로 닫히지 않는다** → KPI는 △. QA-03 ②(위반0)와 정확히 같은 구조이고, QA-03·NQA-A가 **공유 red-team 하네스**를 쓰므로 두 QA의 닫힘은 한 묶음. NQA-A 정식 채택 + 하네스 실측 전환이 동반돼야 둘 다 닫힌다.

## 렌즈 2 — 20년차 수석 아키텍트 관점 (QA 완성도)

- **정식화 가부(OI-8 핵심 질문) = 채택 권장**: 단일 관심사(보안) 명확, QA-03과 `⊂` 경계 명문화로 중복 없음, ISO/IEC 25010 Security 하위특성에 KPI를 각각 앵커링(기밀성↔secrets·무결성↔서명·책임추적성↔HITL)한 것은 방법론적으로 모범. **Sound ○ — 1급 QA로 정식 승격 자격 충분.** "사람 없이 믿고 맡길 수 있는가"의 두 기둥(Security·Correctness) 중 하나라 우선순위 상위 진입이 자연스럽다(번호 재정렬은 사람 결정, OI-8).
- **KPI △ 사유**: 측정 수단([발표 서사])이 4/5 미실현. measurable 형식(0건·100%·95%·구체)은 갖췄으나 산출 경로가 닫히지 않음. High는 아님(정의↔KPI 불일치가 아니라 측정 실행 미수행) — Med 적정.
- **양방향 cross-link 확인**: QA-03 정의가 `Controllability ⊂ Security`로 NQA-A를 단방향 명시했고, NQA-A 정의도 QA-03을 명시 → **양방향 완료**. consistency ○. HITL KPI를 QA-03 ④와 *공유*하는데, 이 경우 **SSoT를 어디에 두는지** 명시 권고(NQA-A를 보안 SSoT, QA-03은 제어 관점 참조).

## 렌즈 3 — 대규모 Workflow Runner 인프라 아키텍트 관점

- **권한 게이트·서명 검증의 runner 환원**: 주 KPI를 DP-0003 1안 사전 권한 게이트(allowlist·실행 전 admission) + DP-0002 1안 HITL에 귀속 — admission control은 runner control plane의 정석. 서명 검증(SLSA류)을 게이트에 추가하는 것도 적절.
- **잔여(DP 위임, OI-7)**: **DP-0002/0003에 공급망 artifact 서명·secrets 관리·injection 가드레일이 후보 대안/ATAM에 없다**(OI-7). 즉 KPI 5축 중 서명·secrets·injection 3축의 책임 설계가 비어 있음 → DP 디스커션 위임(보안 tactic 신설).
- **eval/검증 서브시스템 = 신규 DP 후보**: red-team 하네스가 어떤 DP에도 없는 신규 인프라 — NQA-B golden·QA-03·QA-07 게이트와 **공유 자산**이라 한 DP("eval/검증 서브시스템")로 묶어 신설 검토. 이게 OI-7의 큰 줄기.
- **runner 측 KPI**(제시): admission 차단율(권한 외 호출 거부), 서명 검증 통과/차단율, secrets 스캐너 탐지율(로그·trace 마스킹), red-team 세트 처리율, HITL 게이트 우회 탐색 커버리지.

## 판정

| 항목 | round-01 | round-02 | 근거 |
|---|:---:|:---:|---|
| QA 자체가 sound한가 | (신설) | **○** | 신종 공격면 단일 관심사 + QA-03 `⊂` 경계 + ISO Security 앵커. 1급 정식화 자격 |
| KPI가 측정 가능한가 | (신설) | **△** | 5축 measurable 형식은 갖췄으나 4/5 측정 수단(red-team 세트)이 [발표 서사]로 미실현 |
| KPI가 현실적/적절한가 | (신설) | **○** | zero-trust·HITL·서명·injection 다층 방어 현실적. 커버리지=신뢰 상한은 명시됨 |
| 정의↔KPI↔QAS 일치 | (신설) | **○** | QA-03 양방향 cross-link + QAS-A 동기화. HITL KPI SSoT만 명시 권고 |

**verdict(신설 → round-02): Sound ○ / KPI △ · severity Med.** 정식화 가부 = **채택 권장**(Sound·ISO 앵커 충분). KPI 닫힘은 red-team 세트 실측 전환(정식 채택 동반)이 전제.

## Stage 2 권고 (round-02)

- **(정식 채택 + 번호 재정렬, OI-8)** NQA-A를 정식 QA로 승격 + 우선순위 상위 진입(Security를 QA-01급 후보). 재번호 시 전 QA cross-ref·INDEX·glossary 동기화. **채택 시 PoC를 실측으로 전환**(red-team 세트 구축).
- **(공유 하네스 묶음, OI-8)** QA-03 ②(위반0)와 **공유 red-team 하네스** — 두 QA 닫힘은 한 묶음. 채택·실측을 동반 처리.
- **(DP 디스커션 위임, OI-7)** DP-0002/0003에 **공급망 서명·secrets 관리·injection 가드레일** tactic 명시 + **eval/검증 서브시스템 DP 신설**(NQA-B·QA-03·QA-07 공유).
- **(SSoT 명시, Low)** HITL 통과율 KPI의 SSoT = NQA-A(보안), QA-03 ④는 제어 관점 참조임을 명문화.
- **(예시값 확정)** `injection 차단율 95%`는 red-team 세트로 확정(0건·100%는 절대 기준).
