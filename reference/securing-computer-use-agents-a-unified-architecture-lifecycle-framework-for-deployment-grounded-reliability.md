# Securing Computer-Use Agents: A Unified Architecture–Lifecycle Framework for Deployment-Grounded Reliability

> **원문 수록 문서** — pdftotext로 추출한 PDF 원문(레이아웃 보존판 + 읽기 순서판, 2본 대조)을
> 전사 순서대로 전부 수록한다 (2026-07-16). PDF 줄바꿈 하이픈은 복원, 그 외 문면은 원문
> 그대로. IEEE 2단 컬럼 레이아웃에서 반복되는 러닝 헤더·페이지 번호 단독 줄은 제거했다.
> 본문 중 로마숫자 절 번호(I, II, III …)와 알파벳 소절 기호(A, B, C …), 그리고 문단 내
> 러닝인 소제목(a), b), c) …)은 원문 표기를 그대로 살리되 가독을 위해 굵게 표시했다.
> 참고문헌은 [1]–[191] 전체를 원문 그대로 옮겼다.
>
> - Zejian Chen, Zhanyuan Liu, Chaozhuo Li, Mengxiang Han, Songyang Liu, Litian Zhang,
>   Feng Gao, Yiming Hei, Xi Zhang, "Securing Computer-Use Agents: A Unified
>   Architecture–Lifecycle Framework for Deployment-Grounded Reliability," arXiv:2605.07110v1
>   [cs.CL], 8 May 2026.

## 앞머리

- **저자·소속**: Zejian Chen∗, Zhanyuan Liu∗, Chaozhuo Li∗, Mengxiang Han†, Songyang Liu∗,
  Litian Zhang∗, Feng Gao†, Yiming Hei‡, Xi Zhang∗
  - ∗ Beijing University of Posts and Telecommunications, Beijing, China. Emails:
    {chenzejian, doubao, lichaozhuo, liusyang, zhanglitian, zhangx}@bupt.edu.cn
  - † China Unicom, Beijing, China. Emails: hanmx12@chinaunicom.cn, gaofeng149@chinaunicom.cn
  - ‡ China Academy of Information and Communications Technology, Artificial Intelligence
    Institute, Beijing, China. Email: heiyiming@caict.ac.cn
  - **Corresponding author**: Xi Zhang (zhangx@bupt.edu.cn).
- **arXiv 메타데이터**: arXiv:2605.07110v1 [cs.CL] 8 May 2026.
- **Index Terms**: computer-use agents, deployment-grounded reliability, architecture–lifecycle
  framework, agent security, runtime oversight

## Abstract

Computer-use agents (CUAs) are moving from bounded benchmarks toward real software environments, where they operate browsers, desktops, mobile applications, filesystems, terminals, and tool backends. In such settings, reliability is no longer captured by task success alone: perception errors, planning drift, memory use, tool mediation, permission scope, and runtime oversight jointly determine whether agent actions remain aligned with user intent. Existing surveys organize the CUA landscape by methods, platforms, benchmarks, or security threats, but less explicitly connect capability formation, authority exposure, failure manifestation, and control placement. To address this gap, the article develops an architecture–lifecycle framework for deployment-grounded reliability in CUAs. The architectural view analyzes Perception, Decision, and Execution as coupled layers that transform software observations into authority-bearing actions. The lifecycle view examines Creation, Deployment, Operation, and Maintenance as stages in which priors are learned, tools and permissions are bound, runtime trajectories are stressed, and assurance must be preserved under drift. Using this lens, the analysis synthesizes representative systems, benchmarks, and security/privacy studies; distinguishes where failures become visible from where their enabling conditions are introduced; and maps recurring intervention surfaces for control, oversight, and assurance. OpenClaw is used only as a public motivating example of an open deployment pattern, not as a verified internal case study. The conclusion highlights open challenges in controllable grounding, long-horizon constraint preservation, safe authority binding, mixed-trust runtime defense, privacy-preserving memory, and continual assurance.

## I. Introduction

Computer-use agents (CUAs) are increasingly studied as agents that can operate real software environments rather than answer prompts alone. Recent systems already span browsers, desktops, mobile applications, filesystems, terminals, and tool backends [1]–[7]. This transition changes the meaning of reliability. In a dialogue-only setting, a local mistake often remains textual. In a live software environment, the same mistake can become a deleted file, a leaked secret, an unintended transfer, or a persistent misconfiguration. Reliable computer use is therefore a systems problem as much as a modeling problem, because perception, planning, execution authority, memory, tool use, and oversight interact under live software conditions.

The recent expansion of CUA deployment settings makes an integrative survey timely. Benchmarks have moved from bounded website tasks toward visually grounded, enterprise, personalized, and open-environment settings [8]–[17]. At the same time, system-building and evaluation work has diversified across grounding, memory, long-horizon planning, tool-augmented execution, safety evaluation, and open-deployment stacks [1], [2], [18]–[20]. The difficulty is no longer only the lack of evidence about CUA capability. It is also the lack of a common coordinate system for interpreting how capability is formed, how it is bound to operational authority, where failures first become visible, and where controls can intervene.

Existing surveys clarify important slices of this landscape. Current overviews summarize method families, platform coverage, benchmark inventories, and high-level ecosystem structure [21]–[26]. More focused surveys examine reinforcement-learning enhancement, phone automation, WebAgents, or safety and security threats [27]–[30]. These works are valuable, but they usually organize the field by method, platform, benchmark, or threat category. Less explicit is a deployment-grounded account of how learned capabilities become authority-bearing actions, how similar runtime failures can originate from different upstream conditions, and how security, privacy, and oversight mechanisms should be placed across both system architecture and lifecycle stage.

To address that gap, the article develops an analytical framework for deployment-grounded reliability in general-purpose CUAs across web, desktop, mobile, and cross-application settings. The framework combines two views. The architectural view analyzes CUAs through three coupled layers: Perception, which reconstructs actionable state from software observations; Decision, which preserves task-conditioned intent under uncertainty and long-horizon pressure; and Execution, which converts decisions into authority-bearing operations. The lifecycle view analyzes four stages: Creation, where priors, grounding habits, objectives, and action abstractions are formed; Deployment, where tools, sessions, permissions, and observation channels are bound to the agent; Operation, where active trajectories are stressed by mixed-trust inputs, partial observability, and asynchronous change; and Maintenance, where models, interfaces, tools, and ecosystems drift after release. Together, these two views provide a diagnostic lens for distinguishing where a failure manifests from where its enabling condition was introduced.

The scope here is deployment-grounded reliability rather than CUA capability alone. The analysis focuses on systems, benchmarks, deployment patterns, and security or privacy studies that materially inform how CUAs behave once they interact with real software surfaces and operational authority. OpenClaw is used only as a public motivating example of an open deployment pattern, not as a verified internal case study [31], [32]. More generally, the analysis targets the broader class of CUAs that combine interface interaction, tooling, memory, and authority-bearing execution under real-world deployment conditions.

The resulting contributions are:

- A deployment-grounded reliability scope for CUAs. CUA reliability is characterized as a joint property of software observation, task-conditioned decision making, authority-bearing execution, memory, tool use, and oversight, rather than as benchmark task success alone.
- An architecture–lifecycle framework. The literature is organized through a tri-layer architecture of Perception, Decision, and Execution, together with a four-stage lifecycle of Creation, Deployment, Operation, and Maintenance.
- A failure-origin analysis. The analysis distinguishes where failures become visible from where their enabling conditions are introduced, relating grounding errors, planning drift, over-privilege, memory misuse, mixed-trust inputs, privacy leakage, and ecosystem drift within one diagnostic view.
- A control and governance map. Recurring intervention surfaces are mapped to architectural layers and lifecycle stages, including data and reward design, permission scoping, provenance-aware tool mediation, runtime verification, human escalation, rollback, and continual assurance.

**a) Scope, Evidence Base, and Non-Goals:** The analysis is organizational and diagnostic. It does not attempt to verify OpenClaw internals, introduce a new benchmark, or provide a formal safety proof. Its goal is to synthesize existing CUA literature and public deployment patterns through a common analytical frame. The coverage is weighted toward recent CUA work, while retaining earlier studies when they contribute concepts that remain necessary for computer-use reasoning, grounding, execution, or oversight. The selection prioritizes representative systems, benchmarks, surveys, deployment-oriented stacks, and security or privacy studies that materially inform deployed CUA behavior [13], [24], [33]–[38]. Adjacent agent-security, tool-use, or governance literature is included only when it clarifies a CUA-relevant mechanism, and is treated as adjacent evidence rather than direct CUA evidence.

Table I situates the proposed architecture–lifecycle view within the literature, not to rank prior work. It is intentionally selective: the goal is to indicate the analytical scope here rather than to compress the full CUA literature into a single table. Many prior studies already address important aspects of safety, lifecycle, and governance. The contribution here is not to replace those lines of work, but to relate them through a common analytical frame centered on capability formation, authority exposure, failure emergence, and control placement. The remainder develops that frame in sequence. Section II defines the problem space and its main design axes; Section III introduces the analytical framework; Sections IV and V develop the architectural and lifecycle dimensions, respectively; Section VI analyzes security and privacy through that joint perspective; and Section VII draws out the resulting governance and control implications.

## II. Problem Definition and Design Axes

Before discussing architecture or lifecycle, this section fixes the control problem that a computer-use agent (CUA) is meant to solve. Given a user goal g, a constraint set c, and a stream of partial observations from a live software environment E, the agent must maintain task-relevant state and emit actions through an authority-bearing interface so that the task progresses without violating user intent or system permissions. The object of study is therefore not interface understanding in isolation. It is control in partially observable, dynamically changing, permission-constrained software environments. The section formalizes that setting and isolates the two design axes that matter most for the following analysis: how the agent constructs executable state from software observations, and how it binds intent to authority-bearing action.

### A. Problem Formulation: Software Interaction as Partially Observable Control

A CUA can be written abstractly as a policy π(at | ht, ot, g, c) operating over a software-facing environment E. At step t, the agent receives a partial observation ot, updates an internal state ht, and emits an action at ∈ A through the environment's available control surface. The output is therefore not text alone, but an action with real operational effect. The objective is to reach a task-complete state while preserving user-specified constraints, respecting authority boundaries, and remaining recoverable when uncertainty is high.

What makes this setting distinctive is that software control is only partially observable and only weakly synchronized. A realistic task can move across browsers, operating-system windows, filesystems, terminals, APIs, and tool backends [1], [3], [45], [46]. Even with screenshots, DOM or accessibility trees, OCR output, memory traces, and tool responses, relevant state may remain hidden, delayed, or already stale: windows change asynchronously, permission prompts appear after a plan is formed, and tool state may become visible only after an invocation succeeds [9], [17], [47].

Three properties follow. First, software environments are semantically dense: a small icon, checkbox, or command argument can carry disproportionate consequences. Second, they are mixed-trust by default: the same observation stream may contain user intent, benign interface content, deceptive prompts, retrieved text, and attacker-controlled or compromised tool output [41], [42], [48], [49]. Third, they are layered with authority: the action the model selects is not identical to the permission the system ultimately exercises. This is why CUA behavior is treated here as controlled interaction under partial observability and constrained authority, not as next-step prediction alone.

**TABLE I**
**HEURISTIC POSITIONING MAP FOR THE PROPOSED ARCHITECTURE–LIFECYCLE VIEW WITHIN RECENT CUA LITERATURE. THE TABLE FOREGROUNDS THE ANALYTICAL GAP ADDRESSED HERE RATHER THAN SERVING AS AN EXHAUSTIVE RELATED-WORK INVENTORY.**

| Literature Family | Representative Anchors | What It Best Clarifies | What Remains Less Explicit in Live Deployment |
|---|---|---|---|
| Overviews and taxonomies | GUI and OS-agent overviews [21]–[23], [26] | Method families, platform coverage, benchmark inventories, and high-level ecosystem structure | How capability formation, authority exposure, failure origin, and control placement interact once the agent leaves the benchmark loop |
| Focused survey slices | RL-enhanced GUI agents, phone automation, WebAgents, and CUA safety/security surveys [27]–[30] | Deeper summaries of one training regime, platform, application surface, or threat family | How those slices fit into one shared account of deployed reliability across architecture, authority, and lifecycle |
| System-building and scaling efforts | OpenCUA, bBoN, ClawGUI, and LongHorizonUI [1], [2], [33], [34] | How CUA capability is being extended through open foundations, scaling, orchestration, and end-to-end stacks | How similar capability can map to different reliability and risk profiles once tools, permissions, sessions, and oversight are bound in deployment |
| Benchmarks, datasets, and environment builders | Mind2Web, OSWorld, WebForge, and Gym-Anything [8], [13], [17], [39] | Capability evaluation, realism expansion, synthetic world generation, and broader experimentation infrastructure | Which failures may be introduced in training, made visible by deployment binding, amplified at runtime, or reopened by post-release drift |
| Safety, misuse, and privacy evaluation | OS-Harm, WAInjectBench, Risky-Bench, and WebPII [36], [40]–[42] | Action-level harm, injection, deployment-grounded risk, and privacy exposure as evaluated in realistic computer-use settings | How those signals can be mapped to architectural layer, lifecycle stage, and early intervention surfaces rather than remaining benchmark-local observations |
| Broad agent security context | Aegis and broader agent-security overviews [43], [44] | Cross-agent governance concepts, protocol design, and security framing beyond one benchmark family | CUA-specific coupling among GUI observation, execution authority, mixed-trust inputs, human oversight, and continual assurance |
| Architecture–lifecycle view | General-purpose CUAs across web, desktop, mobile, and cross-application settings | An integrated architecture–lifecycle account of capability, risk, privacy, and control | Clarifies failure origin versus manifestation, maps risk patterns to intervention points, and outlines deployable control surfaces for live-use CUA settings |

Task-relevant state is also broader than whatever is currently visible. A deployed CUA may need to reason over memory traces, retrieved documents, execution logs, tool outputs, file context, and permission status alongside the active interface [45], [46], [54], [55]. Environment class then determines which observation assumptions remain viable: web settings stress structure and untrusted content, desktop and OS settings mix GUI state with account authority, mobile settings compress semantics into small targets and permission prompts, and cross-app workflows create handoff errors across surfaces. Benchmarks such as Mind2Web, WebArena, and OSWorld are useful here mainly because they reveal which observation assumptions a design can actually survive [8], [9], [17].

The main takeaway of this axis is direct: observation design determines what the agent can know, what it can verify, and what it can trust strongly enough to act on. That is why perception later appears as the first architectural layer rather than as a preprocessing detail.

### B. Observation Modalities and Task-Relevant State Representations

The first major design choice is how raw software observations become task-relevant state. In some environments, DOM trees, accessibility metadata, or view hierarchies provide precise handles for localization and control [9]. In others, screenshot-grounded interaction dominates, which turns computer use into a multimodal grounding problem shaped by OCR quality, scale, localization, theme variation, and small-target robustness. Hybrid approaches sit between those extremes by invoking parsers or specialist grounders only when they improve downstream control [50]–[53].

### C. Execution Interfaces, Action Spaces, and Authority Binding

The second major design choice is how model intent becomes operational authority. Action design is therefore not only a question of granularity. Two action spaces may look equally expressive while exposing very different blast radii, auditability, and rollback properties.

Low-level mouse, keyboard, and touch actions are the most general. They allow the agent to operate almost any visible interface, but they make success depend on precise grounding and stable timing: a coordinate error can become a misclick, and a short delay can become a time-of-check/time-of-use failure. Higher-level actions such as browser APIs, application macros, shell commands, or code execution compress many low-level steps into fewer semantic operations. They often improve efficiency and logging, but they also increase the consequence of each mistake because one invocation may encode a wider authority surface [4], [5], [45], [46].

This is why action abstraction and authority binding are treated as related but distinct axes. A click is low-level and weakly semantic, but it can still be high-risk if it confirms a destructive operation. A shell command or API invocation is high-level and often more auditable, but it usually carries more direct authority over files, networks, or processes. Modern CUAs increasingly combine both forms of action, which makes execution design inseparable from permission scope, auditability, and recoverability.

The main takeaway of this second axis is equally direct: execution design determines what the agent can cause, how tightly those consequences are bound to authority, and how easy they are to inspect or reverse. That is why execution later appears as a distinct architectural layer rather than as the final step of a generic control loop.

**a) Takeaway:** The central problem of computer use is therefore not only recognizing what is on the screen. It is deciding what should count as executable state, and binding that state to action interfaces that carry real authority. Observation design determines what the agent can know; execution design determines what the agent can cause. Their coupling determines which failures appear first as ambiguity, which appear first as overreach, and why the same error can have very different consequences across architectures and deployment stages. The next section turns that problem definition into an explicit analytical framework.

## III. Analytical Framework

The previous section defined the problem space and its main design axes. This section defines how the following analysis reasons about that space. The proposed joint framework combines an architectural view with a lifecycle view. The architectural view explains how a CUA is built across Perception, Decision, and Execution. The lifecycle view explains when capability is created, bound to authority, stressed in live use, and revised over time through Creation, Deployment, Operation, and Maintenance. The framework is used as a heuristic analytical lens for organizing literature and locating intervention surfaces in deployed settings [23], [25], [27].

It should not be read as a formal causal model, nor as a claim that real systems or incidents fall neatly into mutually exclusive classes. Its purpose is narrower and more practical: to provide a working coordinate system for reasoning about how capability, authority, and oversight become coupled in deployed computer-use settings.

### A. Why a Component View Is Not Enough

Architecture alone remains necessary, but it is incomplete. It tells us how an agent reconstructs interface state, forms plans, and acts. That is enough to compare systems at the component level. It is not enough to explain why the same model behaves well in one environment and fails in another, why one action interface is tolerable under one deployment and materially riskier under another, or why a failure first seen at runtime may in fact originate in data curation, permission binding, or post-release drift.

The lifecycle view adds the missing temporal dimension. Creation asks what priors the system learns before release. Deployment asks how those priors are coupled to live channels, tools, sessions, and permissions. Operation asks how the system behaves under long horizons, mixed-trust inputs, and asynchronous change. Maintenance asks whether the same architecture remains valid as interfaces, tools, workflows, and threat models evolve. This second dimension helps separate where a failure becomes visible from where the enabling condition was introduced. That distinction is a main analytical commitment of the framework.

### B. What the Joint Framework Claims

The joint framework makes three claims that organize the rest of the analysis.

First, capability is lifecycle-dependent rather than purely model-dependent. What practitioners often call "agent ability" is jointly shaped by representation choices, post-training, execution interfaces, deployment mediation, memory, and recovery mechanisms. Runtime performance is therefore not an intrinsic property of the base model alone.

Second, failure origin and failure manifestation are different analytical questions. A brittle recovery path visible during operation may reflect weak grounding supervision in creation, over-broad or weakly mediated permission scope in deployment, or untested drift in maintenance. Treating every higher-risk action as a runtime reasoning defect hides the real control surface.

Third, control placement must follow both layer and stage. Perception errors, decision errors, and execution errors do not respond to the same controls, and creation-stage controls differ fundamentally from deploy-time, runtime, or maintenance-time ones. Security and governance are therefore not appended after the agent is built; they are about where constraints attach relative to the system's structure and evolution.

Table II summarizes the working coordinate system used throughout the article. Later tables are narrower instantiations of the same logic rather than competing taxonomies. Whenever later sections compare systems, failures, or controls, the guiding questions remain the same: what stage has changed the object of analysis, and which architectural layer now carries the main pressure?

**a) Assignment Rule:** An issue is assigned primarily by the earliest stage at which it is materially introduced or could still have been constrained. Creation applies when pre-release data, objectives, or action abstractions are the main source of the problem. Deployment applies when live binding decisions about tools, permissions, sessions, observation channels, or mediation dominate. Operation applies when online trajectory evolution under uncertainty is the main source of failure. Maintenance applies when post-release change to the model, environment, or ecosystem is the primary driver. Persistent memory, for example, is a deployment choice; misuse of that memory during a live trajectory is operational; and memory-related regressions after updates belong to maintenance.

**TABLE II**
**ARCHITECTURE–LIFECYCLE COORDINATE SYSTEM FOR THE ANALYTICAL FRAMEWORK. THE TABLE MAPS LIFECYCLE STAGES TO THE OBJECTS THEY SHAPE AND TO THE CORRESPONDING PRESSURE POINTS IN THE PERCEPTION, DECISION, AND EXECUTION LAYERS.**

| Stage | Primary Object Shaped | Perception (Where the Tri-Layer Architecture Is Pressured) | Decision (Where the Tri-Layer Architecture Is Pressured) | Execution (Where the Tri-Layer Architecture Is Pressured) | Why This Stage Is Analytically Distinct |
|---|---|---|---|---|---|
| Creation | Learned priors, grounding habits, and action vocabulary | What counts as reliable executable state is set by data and supervision | Goal preferences, decomposition habits, and caution thresholds are normalized | Action abstractions and default affordances are chosen before release | Upstream choices can stay latent until real authority is later attached |
| Deployment | Live bindings among observations, tools, sessions, and permissions | Observation channels and provenance are coupled to real software surfaces | Memory, orchestration, and policy mediation enter the live loop | Tools and permissions turn plans into operational authority | The same policy acquires a different risk profile once capability is bound to authority |
| Operation | Active trajectories under uncertainty and mixed trust | Ambiguity, deception, stale state, and missing state appear online | Long-horizon coherence, constraint retention, and verification are stressed | Latency, TOCTOU, and delayed effects shape real consequence | Many failures become visible here even when the enabling condition entered earlier |
| Maintenance | Validity of the model–environment–ecosystem stack | UI drift, localization change, and rendering shift erode grounding fit | Recovery logic, evaluators, and policies can regress after updates | Extensions, permissions, and containment layers drift with the ecosystem | Assurance must survive change, not only one release |

### C. Deployment Archetypes and an Open-Deployment Example

To keep the framework concrete without collapsing the analysis into a single case study, three recurring deployment archetypes are useful.

Benchmark-centered research agents are optimized for bounded evaluation. They typically expose narrow authority, controlled observability, and reproducible task structure [13], [14], [39].

Consumer or assistant-style deployed agents sit inside user workflows such as search, browsing, productivity, or messaging. Their defining pressure is that user intent, real accounts, and high-impact actions coexist in one interface loop [3]–[5].

Self-hosted gateway or tool-binding agents can make CUA capability available through persistent ingress channels, longer-lived memory, tool registries, or automation layers. Their defining pressure is the coupling among authority, provenance, and ecosystem evolution rather than raw capability alone [56]–[58].

OpenClaw is used only as a motivating deployment setting. Public materials describe it as a local-first assistant with persistent ingress, tool connectivity, and visible participation in a broader agent community [31], [32], [59]. That public framing is treated as an illustrative deployment pattern rather than as verified system evidence: once one deployment surface is presented as combining channels, tools, sessions, and longer-lived context, deployment choices can account for a large share of the resulting risk profile. The analysis remains about the broader class of systems that share a similar structure.

### D. How To Read the Rest of the Paper

The remaining sections form a guided traversal of Table II. Section IV develops the architectural dimension by asking how perception, decision, and execution interact. Section V develops the temporal dimension by asking how capability, failure, and control move across Creation, Deployment, Operation, and Maintenance. Section VI maps threats back onto both dimensions. Section VII then converts that analysis into control surfaces and governance implications. The intended reading is cumulative rather than parallel: each later section reuses the same framework to answer a different layer of the same reliability problem.

## IV. Architectural Evolution of Computer-Use Agents: A Tri-Layer Framework

The architectural question in CUAs is simple to state and difficult to answer well: how does an agent convert a changing software environment into justified, executable action? A useful synthesis is structural. A deployed CUA must reconstruct actionable state, preserve task-conditioned intent, and finally translate that intent into authority-bearing operations. This section therefore analyzes CUA architectures through three coupled layers: Perception, Decision, and Execution. The value of this decomposition is not that every system literally contains three modules. It is that the decomposition helps indicate where capability is created, where uncertainty accumulates, and where small errors become operationally expensive [23], [26], [27].

The recent literature makes this tri-layer view increasingly necessary. Perception work spans structured web grounding, parser-augmented visual understanding, native screenshot-grounded policies, and newer grounding methods that adapt zoom, attention, or symbolic structure to interface complexity [51]–[53], [60]–[62]. Planning and control work now covers memory-augmented reasoning, long-horizon decomposition, rollback, reflection, and hybrid planners that mix GUI control with programmatic execution [19], [46], [63]–[66]. Execution work is widening at the same time, from atomic GUI actions to code-as-action, dexterous control, and bundled routines, which means architectural choice is now inseparable from authority design [45], [67]–[70]. The field does not lack methods; instead, recent capability gains often shift risk across layers rather than remove it.

Figure 1 depicts a recurrent control loop rather than a one-way pipeline. The live environment supplies dynamic and partially observable state to perception, user goals and constraints shape decision, and execution not only acts on the environment but can also return verification, feedback, rollback, or replanning signals to decision. Perception shapes what the system can treat as actionable state. Decision shapes whether that state is converted into a coherent and constrained trajectory. Execution shapes how much authority is exercised when that trajectory is enacted. Failure pressure can propagate accordingly: weak grounding can distort planning, weak planning can amplify execution risk, and high-authority execution can raise the cost of even small perceptual or decisional mistakes.

Table III summarizes heuristic architectural bets rather than ranking individual systems. The same deployed CUA may combine several rows at once, which is precisely why capability gains often reappear later as cross-layer reliability pressure.

### A. Perception Layer: From Raw Interfaces to Actionable State

The perception layer answers the first architectural question: what does the agent believe the interface contains, and how trustworthy is that belief for downstream action? In CUAs, perception is not only recognition. It is the construction of operational state from screenshots, DOM trees, accessibility metadata, OCR output, parser-derived layouts, and retrieved contextual traces. The design problem is therefore not simply to maximize visual accuracy. It is to decide what form of state remains precise enough to act on and portable enough to survive across environments.

The literature now falls into four recurring families. Structured-state agents rely on DOM, accessibility, or view-hierarchy signals. Their strength is precision when the environment exposes reliable structure, which is why Mind2Web and WebArena remain important reference points for disciplined web control [8], [9]. Their weakness is portability: once interfaces become rendering-heavy, streamed, or weakly instrumented, the same structural advantage can disappear.

Parser-augmented visual agents recover part of that structure from rendered interfaces. Systems such as OmniParser, ScreenAI, SeeClick, Ferret-UI, TRISHUL, and newer complete-screen parsing approaches add OCR, icon captioning, region proposals, or layout parsing before downstream reasoning [50], [51], [60], [71], [73], [74]. This family is attractive because it retains visual generality while recovering some of the handles that make execution easier. Its main failure mode is familiar: the parser becomes a bottleneck, and downstream reasoning can remain confidently wrong when the parsed state is incomplete or distorted.

Native end-to-end visual agents push farther toward portability by leaving more of grounding inside one multimodal policy. CogAgent, Fuyu-style VLMs, Qwen-VL style systems, WinClick, MAI-UI, Step-GUI, Mobile-Agent-v3.5, and high-resolution-aware agents such as AFRAgent illustrate this direction [3]–[5], [61], [75]–[78]. The gain is broad environment coverage. The cost is entanglement. OCR, localization, affordance inference, and actionability all live inside one representation, which makes it harder to know whether a failure arose from vision, reasoning, or the coupling between the two.

Hybrid or compositional grounding agents mix several channels rather than committing to one. They use structured state when it is available, screenshots when it is not, and specialist grounders only when those tools improve downstream control [6], [45], [52], [64]. This family is especially important for cross-platform and open-deployment settings because the observation channel itself may vary within a single task.

These families are best compared along four axes: element-grounding precision, small-target robustness, robustness to scale/theme/localization changes, and downstream executability. No current approach simultaneously maximizes all four. Benchmarks such as ScreenSpot-Pro help make the trade-off concrete by indicating that high-resolution professional interfaces still punish small-target grounding even when broader GUI capability looks competitive elsewhere [79]. The most precise observation channel is rarely the most portable, and the most portable channel is rarely the easiest to verify. That tension is why perception in CUAs should be judged not by recognition accuracy alone, but by whether the resulting state remains trustworthy enough for safe execution.

**a) Observation-channel mismatch:** One under-discussed architectural risk is mismatch between the observation channel assumed during training and the one encountered in deployment. A model trained on parser-normalized or DOM-rich state can look strong on benchmarks yet fail in screenshot-only or streamed settings. A screenshot-trained model can make the opposite mistake by ignoring structure that would have improved precision. The architectural point is that a policy can look transferable while the real grounding problem has already changed underneath it.

This classification remains heuristic rather than exhaustive. Many deployed systems mix structured state, screenshots, parsers, and specialist grounders within one task, so the perception families above capture dominant observation bets rather than clean system boundaries.

**Fig. 1.** Tri-layer pressure map for deployed CUAs. The figure frames deployed CUA behavior as a recurrent control loop between user goals and constraints and a live, partially observable software environment. Perception reconstructs operationally trustworthy state, decision maintains task-conditioned intent under long-horizon pressure, and execution converts the resulting trajectory into authority-bearing action.

**TABLE III**
**ARCHITECTURAL ARCHETYPES IN CONTEMPORARY CUAS. THE TABLE SUMMARIZES RECURRING DESIGN PATTERNS, THEIR PRIMARY CAPABILITY GAINS, THE LAYERS THEY CHIEFLY STRENGTHEN, AND THE RELIABILITY PRESSURES THEY INTRODUCE IN DEPLOYED STACKS.**

| Archetype | Representative Systems | Main Capability Gain | Layer Chiefly Strengthened | Main Reliability Pressure |
|---|---|---|---|---|
| Structured-state web agents | Mind2Web and WebArena-style agents [8], [9] | Precise element grounding and deterministic references when structure is reliable | Perception | Portability narrows once dependable structure disappears or becomes weakly exposed |
| Parser-augmented visual agents | OmniParser, WebVoyager, and SeeClick [47], [60], [71] | Recover action handles from rendered interfaces without full dependence on raw DOM exposure | Perception | Parser errors can propagate downstream as confident but distorted action state |
| Native end-to-end visual agents | WinClick, Mobile-Agent, and MAI-UI [4], [61], [72] | Broad cross-platform reach from screenshot-grounded policies | Perception + Decision | Failures become harder to localize, and small-target robustness remains costly |
| Memory-augmented planners | Synapse, WebATLAS, and ColorBrowserAgent [18], [56], [63] | Stronger long-horizon coherence and task continuity | Decision | Stale summaries, poisoned memory, or dropped constraints can steer later steps |
| Hybrid orchestration agents | UFO, GraphPilot, and LiteWebAgent [19], [57], [64] | Decompose work across applications, tools, or specialized roles | Decision + Execution | Coordination, attribution, and interface-selection errors move burden across components |
| Hybrid execution agents | OS-Copilot, CoAct-1, and AppAgentX [45], [46], [67] | Replace brittle GUI sequences with code, tools, or reusable routines | Execution | Stronger authority binding and harder rollback enlarge the consequence of mistakes |

### B. Decision Layer: Preserving Intent Under Long-Horizon Pressure

If perception answers "what is happening now," the decision layer answers "what should happen next, and how can the system verify that it still matches the user's goal?" In deployed CUAs, this is where autonomy becomes either stable or brittle. The core difficulty is not merely local next-step selection. It is preserving task-conditioned intent across long trajectories, incomplete observations, changing interfaces, and sometimes hostile runtime content.

Four pressures organize the literature here. The first is long-horizon coherence: can the system keep the task moving in the right direction across many screens and tool calls? The second is constraint retention: can it preserve instructions such as "draft, do not send" or "ask before deleting" after dozens of intermediate steps? The third is verification and uncertainty handling: does it know when to check, backtrack, or ask for clarification? The fourth is decomposition: does it solve the task in one policy loop, or distribute work across roles, tools, or subagents?

Memory-augmented systems are an early answer to the coherence problem. Synapse illustrated that state abstraction, trajectory exemplars, and retrieval can materially stabilize computer-control behavior under limited context [63]. AppAgent and InfiGUIAgent push the same idea toward reusable operating knowledge rather than disposable context [80], [81]. Newer systems such as WebATLAS, SecAgent, ColorBrowserAgent, and anchored-memory benchmarks such as AndroTMem make a similar point from different directions by treating summarization, simulation, or anchored recall as first-class planning supports rather than optional add-ons [18], [55], [56], [82]. OSWorld evaluations are consistent with the same concern: long tasks can degrade quickly without some mechanism for compression, recall, or structured recovery [17]. These systems matter because they suggest that CUA performance is not only a function of local reasoning quality. It also depends on how continuity is carried forward.

Constraint retention is the more safety-critical version of the same problem. A CUA can appear productive while gradually dropping confirmation requirements, user preferences, or exposure limits. Reflection-oriented work and safe-planning or interruptibility evaluations are therefore informative not only because they improve task completion, but because they ask whether the original task boundary stays attached to later steps [83]–[87]. The more concerning failure mode is often not that the model stops progressing. It is that it keeps progressing after silently forgetting what should have constrained it.

Verification and uncertainty handling are what keep that drift from becoming irreversible. General agent methods such as ReAct, Tree of Thoughts, and Reflexion contributed three ideas that recur in CUA planners: interleaving reasoning with action, branching over alternatives, and using prior errors as planning signals [88]–[90]. WebVoyager, You Only Look at Screens, ReInAgent, Mobile-Agent, BacktrackAgent, TreeCUA, stable-planner modules, and action-effect verification systems carry those ideas into live interfaces through backtracking, chain-of-action reasoning, active questioning, replanning, explicit recovery after detected errors, and post-action verification [47], [66], [72], [91]–[95]. The literature does not point to one universally superior planner. It does suggest, however, that robust computer use cannot remain purely feedforward once trajectories become long and mixed-trust.

Decomposition offers a second route to scale. UFO separates application routing from local action selection, while CoAct-1 coordinates GUI operation with programmatic execution through a central planner [46], [64]. EE-MCP, GraphPilot, LiteWebAgent, Agent-SAMA, PolySkill, and earlier planning-oriented web agents extend the same design space by balancing GUI interaction against tool calls, reusable skills, finite-state planning, or program synthesis, which makes the coordination problem partly an interface-selection and skill-selection problem rather than only a planning problem [19], [57], [58], [65], [70], [96]. These designs can make complex workflows tractable, but they do not eliminate difficulty. They move difficulty from single-policy context management into coordination, attribution, and trust management across components.

The architectural lesson is therefore narrow but important: decision quality in CUAs is not the ability to choose the next plausible click. It is the ability to preserve user-intended constraints while converting uncertain, partial state into a trajectory that remains coherent over time. Stronger planning without stronger verification usually means stronger drift. This decision-layer classification also has limits. Real systems often fuse memory, reflection, skills, planners, and verification into one controller, so the categories above capture recurring planning pressures rather than strict module types.

### C. Execution Layer: Action Abstraction, Authority, and Recoverability

The execution layer is where the reliability argument becomes concrete. A CUA does not merely choose what should happen. It acts through a specific interface that determines how much authority the decision carries, how easy the action is to audit, and how recoverable the outcome remains when something goes wrong. In other words, execution is where intent becomes consequence.

Three execution styles recur across the literature. Atomic input execution uses clicks, drags, taps, and keystrokes. It is maximally general and often the only option when no trusted higher-level interface exists. Cradle and AndroidEnv illustrate the breadth of this style, while many visual desktop and mobile agents inherit the same strengths and weaknesses [69], [97]–[99]. Its advantage is portability. Its failure modes are misclicks, action rebinding, clickjacking, and time-of-check/time-of-use gaps.

Programmatic execution compresses many low-level steps into semantically richer operations. OS-Copilot and CoAct-1 illustrate how file operations, application macros, shell commands, or code execution can bypass brittle GUI sequences in complex workflows [45], [46]. GUI-360, MAI-UI, and Step-GUI add complementary evidence that hybrid GUI+API or GUI+MCP action spaces are a recurring part of the current CUA design landscape rather than isolated systems choices [4], [5], [68]. The gain is speed, auditability, and leverage. The trade-off is stronger authority binding: one invocation can modify files, alter configurations, or contact external systems directly.

Bundled or macro-action execution sits between those extremes. AppAgentX is a direct example because it evolves recurrent action sequences into higher-level routines that substitute for repeated low-level interaction [67]. PolySkill provides a related skill-abstraction view in which reusable polymorphic skills stand in for repeated interaction fragments across tasks [70]. This can reduce planning burden and improve efficiency, but it also makes failure harder to localize because several consequential substeps may be bundled into one command. More general screenshot-grounded systems such as ScreenAgent and OmegaUse reinforce the field's movement toward broad end-to-end GUI control, but they do not resolve on their own where rollback and verification boundaries should sit [100], [101].

These three styles are best compared through two coupled questions: how abstract is the action, and how much authority is bound to it? That distinction clarifies why shell execution belongs in the execution layer rather than in GUI control. It is a high-authority interface, not a visual interaction primitive. It also clarifies why recoverability deserves its own place in CUA design. Atomic inputs are easy to interrupt but hard to replay semantically. Programmatic actions are easier to log and replay but often harder to roll back safely. Bundled actions reduce step count while blurring rollback boundaries. For deployed CUAs, recoverability can matter more than raw task completion because it determines whether a failure remains local or becomes irreversible.

This execution grouping should not be read as a set of mutually exclusive modes. In practice, many deployed CUAs mix atomic, programmatic, and bundled execution within the same workflow, which is exactly why authority binding and rollback boundaries remain hard to reason about.

### D. Cross-Layer Coupling and Architectural Implications

The three layers should be read together. Perception determines what state is available for planning. Decision determines whether that state is converted into a coherent and constrained trajectory. Execution determines how much real authority that trajectory acquires. Better performance in one layer can therefore shift pressure to another: more portable perception can increase ambiguity, stronger autonomy can increase the need for verification, and more efficient execution can enlarge the blast radius of mistakes.

This is why architecture already contains a latent governance question. A system that combines ambiguous perception, long-horizon autonomy, and high-authority execution is not only an engineering design. It can also create a distinctive configuration of risk. Reliability problems in CUAs often become visible as cross-layer coupling before they appear as isolated bugs [26], [102]. The tri-layer framework is therefore the structural backbone of the analysis. It explains where capability resides, where ambiguity enters, and where authority is exercised. The next section adds the missing temporal backbone by asking how those same layers are pressured differently across Creation, Deployment, Operation, and Maintenance.

## V. A Lifecycle Framework for Computer-Use Agents: From Capability Formation to Continual Adaptation

The previous section explained what a CUA is made of. This section explains how that architecture becomes a live system over time. This distinction matters because the same visible failure can emerge from different upstream causes. A misleading click may reflect weak grounding supervision during training, unsafe permission binding at release, brittle recovery under runtime drift, or a regression introduced during maintenance. The lifecycle view is therefore used not to retell the development timeline, but to distinguish when capability is formed, when authority is attached, when risk first becomes visible, and where effective controls can still enter.

Recent benchmarks and environments make this lifecycle sensitivity easier to see. Newer evaluations increasingly include personalized mobile interaction, privacy-sensitive workflows, collaborative assistance, verification-centered replay, software environments generated at scale, and human-like long-horizon behavior [13], [16], [35], [87], [103]–[105]. These settings do not by themselves establish a lifecycle theory. They do suggest, however, that capability formation, deployment binding, runtime pressure, and maintenance drift leave different empirical signatures once agents are evaluated outside narrow one-shot tasks.

Throughout this section, each stage is read through the same analytical template: What primary object does this stage shape? What risks can first enter here? Why might they only become visible later? Where is the earliest control surface that can still matter? Figure 2 makes that template explicit by organizing every stage into three aligned bands: Primary Object, Main Entry Risks, and Earliest Controls. The categories inside each band are not meant as exhaustive engineering checklists. They summarize the main ways in which a stage can change the eventual behavior of a deployed CUA.

### A. Why These Four Stages and Why These Categories

The four-stage split is intentionally minimal. Creation is separated out because it shapes the policy's priors before the system meets any live account or interface. Deployment is distinct because it changes the binding between learned capability and real authority. Operation is where the active trajectory is exposed to temporal pressure, mixed-trust inputs, and partial observability. Maintenance is separate because post-release drift affects the model, the environment, and the surrounding ecosystem differently. In the revised figure, those differences are captured as different primary objects: priors and grounding habits in creation, authority-bearing bindings in deployment, active trajectories under uncertainty in operation, and validity of the model–environment–ecosystem stack in maintenance.

This separation is analytically useful because coarser decompositions hide important differences in failure origin. If creation and deployment are merged, training-time bias becomes hard to distinguish from over-authority introduced only at release. If operation and maintenance are merged, live runtime stress is confused with regressions or ecosystem drift. The four stages are therefore analytical categories tied to when different enabling conditions are introduced, not just milestones on a project timeline. The revised diagram also makes the analytical sequence more concrete: once the stage-specific object is identified, the next question is what risks first enter there, and then which controls can still act early enough to matter.

The subcategories inside each stage follow the same logic. Creation is decomposed by the mechanisms that shape priors before release, such as data bias, weak supervision, objective bias, and unsafe action abstractions. Deployment is decomposed by the bindings through which capability acquires operational effect, including authority binding, observation channels, tools, sessions, and permissions. Operation is decomposed by the runtime pressures that can turn local uncertainty into higher-risk trajectories, such as long-horizon drift, mixed-trust inputs, TOCTOU, and intent dilution. Maintenance is decomposed by the post-release objects that drift independently, including the model, the environment, and the surrounding ecosystem. Read this way, the lifecycle framework is not an expanded project timeline. It is a map from failure origin to intervention point.

The stage boundaries are operational rather than merely conceptual. Creation is used when a problem is primarily driven by pre-release data, objective design, or action abstraction. Deployment is used when live binding decisions about tools, permissions, sessions, observation channels, or mediation dominate. Operation is used when online trajectory evolution under uncertainty is the main source of failure. Maintenance is used when post-release change to the model, environment, or ecosystem becomes the primary driver. Persistent memory, for example, is a deployment choice; misuse of that memory during a live trajectory is an operational phenomenon; and memory-related regressions after updates belong to maintenance.

**Fig. 2.** Revised four-stage lifecycle framework for deployed CUAs. Each stage is organized into three aligned bands: the primary object shaped at that stage, the main entry risks that can first be introduced there, and the earliest controls that may still change downstream outcomes. The bottom guide emphasizes the main analytical point: failures may be introduced earlier and become visible later, so failure origin can precede failure manifestation.

### B. Creation: Building Priors Before Release

Creation shapes the object that later gets deployed. It determines what counts as actionable state, what trajectories look normal, and what trade-offs the agent learns when task success competes with caution, confirmation, or recoverability. Failures that enter here may remain latent for a long time. They often surface only after the system is trusted with real authority. That is precisely why this stage matters.

**a) Training data source and trajectory quality:** Training data determines which behaviors and edge cases are even visible to the learner. Human trajectory corpora such as AITW, web interaction datasets such as Mind2Web, and generalized agent tuning pipelines suggested that CUAs benefit from task-conditioned action traces rather than generic multimodal pairs [8], [106], [107]. Newer resources such as OpenCUA, TongUI, WebChain, MolmoWeb, and OS-Genesis increase scale, platform diversity, interface coverage, or synthetic trajectory breadth [1], [7], [108]–[110]. Yet scale alone does not solve the main bias problem. Human traces often encode hesitation, confirmation, and recovery. Synthetic or tutorial-derived traces scale faster, but they can overrepresent clean completion paths and underrepresent ambiguity or conservative stopping. Later runtime failures such as overconfident continuation or brittle recovery are often seeded here before they are ever observed live.

**b) Grounding supervision and observation priors:** Creation also decides what the model is taught to trust as executable state. Structured supervision can improve precise element grounding, whereas screenshot-first training improves portability. Systems such as CogAgent, ScreenAI, SeeClick, WinClick, Ferret-UI, TRISHUL, and newer screen-parsing or exploration-based grounding pipelines suggest that downstream reliability can still depend heavily on that choice [50]–[53], [61], [71], [73]–[75]. Explicit grounding supervision makes the operational target clearer. Weak supervision can still produce good benchmark performance, but it often leaves the system relying on unstable salience, parser artifacts, or theme-specific cues that later fail in deployment.

**c) Objective design, reward shaping, and action priors:** Objective design determines what the model is implicitly rewarded to optimize. Work on UI-R1, GUI-R1, ProgRM, Web-Shepherd, UI-Genie, MagicGUI-RMS, Video-Based Reward Modeling, and CUARewardBench suggests that reward-shaped post-training and reward-model design can materially shift action efficiency, verification quality, and task completion behavior [38], [111]–[117]. The lifecycle implication is an inference rather than a direct claim of those papers: if training increasingly rewards low action count, fast completion, or shortcut-taking without parallel incentives for confirmation and recoverability, then a convenience bias may be introduced before any live account is touched. Action abstractions matter for the same reason. If the action vocabulary normalizes broad, weakly mediated authority, that authority may already be treated as ordinary at creation time.

**d) Interactive environments as pre-release stress:** Creation is also where the field can expose delayed consequences before release. Benchmarks such as WebArena, WebForge, Gym-Anything, ClawBench, WorkArena++, and OSWorld matter not only because they measure capability, but because they reveal multi-step recovery, delayed effects, open-environment noise, and broader software coverage while the system is still under development [9], [12]–[14], [17], [39]. They are best read as pre-release stress instruments that surface creation-stage weakness earlier than closed instruction-following benchmarks can.

**e) Why creation-stage failures stay hidden:** The reason creation matters so much is that its failures often look fine until authority is attached. A model can appear capable in benchmark loops while already carrying the wrong observation priors, the wrong convenience bias, or the wrong action habits. The earliest effective control point for creation-stage failures therefore lies in data design, supervision quality, objective shaping, and safe action abstraction.

### C. Deployment: Binding Capability to Authority

Deployment changes the object of analysis from a capable policy into a consequential system. The model is now connected to real observation channels, real tools, real sessions, real permissions, and real users. Capability is not merely exposed to the world here; it is bound to authority. This is why deployment deserves its own stage rather than being collapsed into generic release engineering.

**a) Observation binding:** Observation binding determines what state reaches the model once it leaves the training environment. Screenshot-only deployments maximize reach but accept ambiguity, latency, and mixed-trust content. Screenshot-first systems such as Cradle, Mobile-Agent, Surfer 2, and Mobile-Agent-v3.5 illustrate that reach [3], [6], [72], [97]. Controlled environments such as WebArena reduce ambiguity by exposing cleaner state or more deterministic execution paths [9]. The key trade-off is not simply convenience. It is portability versus controllability.

**b) Tool binding:** Tool binding determines how many external interfaces the agent can call and through what mediation. Systems such as OS-Copilot, UFO, CoAct-1, GraphPilot, LiteWebAgent, MAI-UI, and Step-GUI illustrate how tool augmentation can expand what CUAs can achieve, especially across applications and action channels [4], [5], [19], [45], [46], [57], [64]. At the same time, every new tool call introduces another trust boundary. Tool outputs influence reasoning, tool invocations affect authority, and the tool ecosystem itself becomes part of the deployment surface.

**c) Permission binding:** Permission scope is where abstract capability turns into blast radius. A model that can view or modify local files, send messages, change settings, or install software is not merely a stronger benchmark policy. It can function as a live principal acting under inherited authority. Least privilege, sandboxing, and execution mediation therefore belong to deployment by design, not only to incident response.

**d) Memory, session, and channel binding:** Deployment also decides whether context is ephemeral, task-local, cross-session, or persistent across users and channels. Persistent memory can improve continuity and make assistants feel more competent. It can also enlarge the security and privacy surface because old state remains operationally available long after the original task boundary has passed. Personalized and socially embedded benchmarks such as PSPA-Bench, KnowU-Bench, and CowCorpus help make that shift visible by foregrounding long-lived user context, collaborative intervention, and context carry-over across tasks [15], [16], [105]. The same is true of channel exposure. Messaging routes, automation triggers, browser sessions, shared desktops, and internal tools differ sharply in provenance and trust assumptions.

**e) Why deployment changes the risk profile:** Public materials describe OpenClaw as a gateway-style assistant with persistent ingress, local-first execution, tool connectivity, and longer-lived context [31], [32]. That motivating deployment setting is used only as an illustrative deployment pattern rather than as verified system evidence: once one deployment surface is presented as binding channels, tools, sessions, and permissions together, deployment configuration can account for a large share of the resulting risk profile. Comparable design pressures can also be seen in MCP-enabled assistants, local automation gateways, and other multi-channel CUA deployments [45], [118], [119]. The earliest effective control point for deployment-stage failures therefore lies in mediation, permission scoping, session isolation, provenance, and safe defaults.

### D. Operation: Runtime Pressure, Mixed Trust, and Oversight

Operation is where the deployed system is stressed under live time. The architecture has been chosen, authority has been bound, and the agent now has to remain coherent while the environment changes, the task unfolds, and new content enters the loop. Many failures become visible here even when they did not originate here. That is why runtime incidents should not automatically be read as runtime causes.

**a) Long-horizon drift and stale context:** One source of operational failure is trajectory degradation. WorldGUI, MobileUse, ActionEngine, the Amazing Agent Race, ClawBench, MobileWorldBench, and AndroTMem suggest in different ways that step-wise competence does not automatically produce reliable long-horizon behavior [14], [82], [84], [120]–[123]. An agent can keep taking plausible local steps while losing the original goal, working off stale state, or repeating low-value recovery loops. This is often the main path by which capable-looking systems become operationally unreliable.

**b) Mixed-trust runtime inputs:** Another source is mixed-trust input. During live use, the agent consumes page text, retrieved snippets, messages, document content, and tool outputs that do not share one trust level. Runtime operation is therefore where injection and deceptive content stop being abstract evaluation categories and become active task conditions [124]–[126].

**c) TOCTOU and delayed execution:** Operation also exposes the temporal gap between decision and effect. Interface state can change after the plan is formed but before the click lands, the macro runs, or the command executes. When this happens, a locally sensible action can become globally misbound or otherwise unsafe without the goal itself ever changing. This is one reason execution verification belongs in operation rather than only in design.

**d) User-intent dilution in persistent workflows:** Once sessions persist across tasks, channels, or users, the system can blur ownership and scope. Personalized long-lived evaluations and deployment-oriented analyses are consistent with that concern in settings where memory, task identity, and ingress are modeled as longer-lived than a single interaction [127], [128]. The issue is not only security. It is whether user intent remains the dominant organizing constraint in a long-lived runtime context.

**e) Why runtime control must be explicit:** These pressures are why operation needs an explicit oversight ladder rather than vague references to "human in the loop." In increasing order of control strength, that ladder typically includes logging only, post-action verification, plan preview, step confirmation, and high-impact approval. Systems such as CORA further suggest that this runtime control point can be operationalized through calibrated execute-versus-abstain decisions under an explicit risk budget rather than only through informal approval heuristics [129]. The correct runtime control depends on blast radius, not only on task difficulty. The earliest effective control point for operation-stage failures therefore lies in verification, scoped memory, calibrated escalation, and impact-aware oversight.

### E. Maintenance: Preserving Validity After Release

Maintenance keeps the deployment from silently decaying. A released CUA does not operate against a frozen environment. Models are adapted, interfaces drift, permissions change, tools are updated, and ecosystems evolve around the base system. Treating maintenance as a generic "update" phase hides too much of that complexity. In practice, maintenance governs whether earlier assurance survives change.

**a) Model maintenance:** Model maintenance concerns adaptation, retraining, evaluator changes, and post-release policy updates. Work such as MAGNET, UI-Oceanus, and PolySkill points to the difficulty of updating interface-specific competence without erasing stable knowledge or reintroducing old failure modes [70], [130], [131]. Maintenance is therefore not only about getting a stronger model. It is about preserving the validity of earlier safety and reliability gains under change.

**b) Environment maintenance:** Environment maintenance concerns everything outside the model that changes underneath it: UI redesign, localization drift, new permission prompts, tool API changes, and altered workflow ordering. Benchmarks such as MemGUI-Bench, TimeWarp, Risky-Bench, WebForge, Vision2Web, WebTestBench, GUITester, and OpeFlo matter here because they illustrate how quickly evaluation assumptions can become stale once the deployment environment moves on [13], [42], [132]–[137]. Continuous re-evaluation is therefore a basic operational requirement rather than a benchmarking luxury.

**c) Ecosystem maintenance:** Open deployment adds a third maintenance problem: the surrounding ecosystem changes independently of both model and UI. Plugins, skills, registries, agent identities, watcher layers, and sharing channels all create new drift paths. Your Agent, Their Asset suggests that persistent capability, identity, and knowledge can remain exposed after deployment, while ClawKeeper focuses more directly on skills, plugins, and watcher layers [128], [138]. PASB adds a complementary evaluation frame for input-side corruption, decision-side corruption, and execution-side abuse, together with a heuristic attribution lens of scope overreach (SO), objective corruption (OC), and environmental misbinding (EM). Finally, it maps those patterns back onto the lifecycle so that control placement remains tied to timing rather than only to attack naming.

**d) Why maintenance governs trust continuity:** The key implication is that maintenance governs not only quality, but trust continuity. If models, tools, and registries evolve faster than the assurance stack around them, then the deployment can reopen risks even when the base architecture is unchanged. The earliest effective control point for maintenance-stage failures therefore lies in continual evaluation, controlled adaptation, extension governance, and coordinated patching.

### F. Lifecycle Coupling and Its Analytical Payoff

The four stages are distinct, but not independent. Creation shapes the priors that deployment later binds to authority. Deployment determines which runtime mistakes can become consequential. Operation reveals which assumptions fail under live stress. Maintenance determines whether earlier fixes remain valid as the world changes. Table IV serves as an operational troubleshooting aid: start from the symptom, then work backward to the most likely stage-level diagnosis and earliest useful intervention surface.

The larger lesson is that CUA reliability cannot be inferred from architecture alone. It depends on when capability is formed, when authority is attached, when trajectories are stressed, and when the system is re-evaluated after change. Lifecycle analysis is therefore not supplementary to the architecture chapter. It is the temporal backbone that explains why similar visible failures can demand very different interventions.

## VI. Security and Privacy Analysis

Security is one of the clearest reasons CUAs need the joint framework developed here. These systems read mixed-trust content, retain state across steps, and act through interfaces that may reach files, accounts, tools, and external services. A useful security account must therefore answer three questions at once: where corruption enters, how it becomes operational, and at what lifecycle stage it could have been constrained earlier. Attack names alone are not enough.

The recent CUA security corpus reflects that widening scope. It spans visual prompt injection, harmful-task benchmarking, action rebinding, adversarial backdoors, runtime guardrails, permission scoping, dark-pattern manipulation, privacy-focused evaluation, and runtime monitoring or mediation [36], [124], [139]–[147]. Safety benchmarks such as MobileSafetyBench, ST-WebAgentBench, OS-BLIND, and RiosWorld further indicate that harmful-task completion, policy-noncompliant behavior, and benign-intent failure have become explicit evaluation targets rather than incidental by-products of general task completion [49], [148]–[150]. That literature expands the attack inventory and suggests that risk conditions can enter through different layers and become visible at different stages.

This section therefore proceeds in three steps. It first fixes a threat model. It then introduces a working taxonomy of

**TABLE IV**
**LIFECYCLE DIAGNOSTIC AND INTERVENTION MAP FOR DEPLOYED CUAS. THE TABLE LINKS STAGE-SPECIFIC FAILURE PATTERNS TO THEIR FIRST OBSERVABLE MANIFESTATIONS, LIKELY DIAGNOSIS TARGETS, AND EARLY INTERVENTION PRIORITIES.**

| Stage | Typical Latent Failure Pattern | What It First Looks Like in Practice | First Diagnosis Target | Early Intervention Priorities |
|---|---|---|---|---|
| Creation | Weak grounding priors, shortcut-seeking objectives, or unsafe action abstractions | Brittle localization, over-eager completion, or poor abstention already visible in pre-release evaluation | Data curation, reward / objective design, and action abstraction choices | Stronger grounding supervision, safer reward shaping, and release gating |
| Deployment | Misbound authority, weak provenance, or poorly mediated tool and channel exposure | Correct-seeming plans coupled to over-broad permissions, wrong tools, or channel confusion | Permission scope, tool registry, session isolation, and middleware mediation | Least privilege, sandboxing, provenance-aware tool mediation, and safer defaults |
| Operation | Long-horizon drift or live-state misbinding under mixed-trust runtime inputs | Forgotten constraints, brittle recovery, TOCTOU, or unsafe continuation after local uncertainty | Runtime verification, memory scope, and escalation thresholds | Plan preview, post-action checks, step confirmation, and high-impact approval |
| Maintenance | Regression, stale assurance, or ecosystem drift after release | Sudden behavior change after updates, stale defenses, extension drift, or failed re-evaluation | Model updates, extension inventory, version diffs, and regression suites | Re-evaluation gates, version pinning, extension review, and coordinated patching |

**Fig. 3.** Lifecycle-aligned CUA threats and controls. The figure serves as an intervention map: for each stage, it identifies the salient threat surfaces, the system object under pressure, and the earliest practical control surfaces that may still alter the eventual outcome.

structural threats and a heuristic attribution lens. Figure 3 presents the stage-first version of that argument. The upper band identifies the salient threat surfaces at each stage, the middle band identifies the system object under configuration or stress, and the lower band shows the control surfaces that can still act early enough to matter. The figure therefore links the lifecycle analysis of Section V to the more detailed threat discussion below.

### A. Threat Model

The baseline attacker may control any combination of screen content, retrieved content, and tool outputs. This includes attacker-controlled or deceptive pages, rendered documents, OCR-visible instructions, search results, clipboard contents, agent messages, and tool or MCP responses consumed during execution [48], [49], [151]–[154]. In creation-stage settings, the attacker may additionally influence training data or reward signals. In open deployments, the attacker may also reach the system through persistent communication channels, shared tools, or capability-sharing surfaces.

Protected assets include execution integrity, data confidentiality, and authority boundaries around files, credentials, tools, networks, and operating-system functions. Some realistic CUA deployments expose high-impact actions such as external messaging, deletion, payment, credential entry, or permission changes. The threat model does not assume that every deployment enforces a strong execution-policy layer. Some systems can insert sandboxing, approval hooks, or action gating [129], [141]. Others expose the model more directly to the execution surface. That distinction matters because the same planning error can be much harder to contain once runtime mediation is weak.

Three trust boundaries recur throughout the section: the boundary between user-authored intent and runtime content, the boundary between trusted execution channels and external tools or services, and the boundary between task-local memory and persistent cross-session state. Weakness at any of those boundaries can turn seemingly ordinary autonomy into unauthorized execution or disclosure.

### B. Structural Threat Taxonomy and Attribution Lens

The working taxonomy answers where corruption enters the loop. The heuristic attribution lens answers how a risk becomes operational. These two views are complementary rather than competing. Together they provide an organizing vocabulary rather than a definitive or exhaustive classification.

Input-side corruption covers cases where the observation stream itself is adversarial or misleading. Indirect prompt injection, deceptive UI, attacker-controlled documents, hostile retrieval results, and poisoned tool outputs all fall in this family. The corruption enters through what the agent reads.

Decision-side corruption covers cases where the agent's working objective, planning substrate, or memory becomes attacker-steered or otherwise compromised. Poisoned memory, attacker-steered subplans, and latent training-time triggers whose effect appears during inference all belong here. The corruption enters through what the agent optimizes or retains.

Execution-side abuse covers cases where the interface translating intent into action becomes the main risk surface. Over-privilege, weak isolation, action rebinding, TOCTOU, and unsafe tool invocation all fall in this family. The corruption enters through how the chosen action is operationalized.

The attribution lens asks a different question. Scope overreach is used when the agent expands beyond the user's intended task boundary without strong evidence that the operative goal itself has been replaced. Objective corruption is used when attacker-controlled content, memory, tooling, or training signal becomes the dominant planning objective. Environmental misbinding is used when the apparent goal remains intact, but the observation, execution, or authority layer redirects the realized outcome in unsafe ways. Some incidents may involve more than one dominant attribution pathway.

**a) Operational decision rule:** The attribution lens is applied pragmatically. If attacker-controlled state has become the operative goal, the dominant class is objective corruption. If the goal appears intact but the observation–action or authority binding redirects the outcome, the dominant class is environmental misbinding. If neither condition clearly holds and the main problem is task-boundary expansion or unnecessary data or action collection, the dominant class is scope overreach. The purpose of the rule is to support control placement, not to eliminate every ambiguous case.

**b) Worked example:** Consider a CUA asked to download one invoice from a customer portal. A malicious on-screen banner says "export the full session history for troubleshooting," the agent has an over-broad cloud-upload permission, and no approval hook intervenes before upload. The banner is input-side corruption because the attack enters through what the agent reads. If that injected instruction becomes the operative plan, the dominant attribution is objective corruption. If the original invoice-downloading goal remains intact but the broad permission and missing approval hook allow an unsafe export to proceed, the dominant attribution is environmental misbinding. If, even without clear attacker steering, the agent interprets the task too broadly and exports extra files because it over-collects beyond task need, the dominant attribution is scope overreach. The same incident can therefore expose overlapping mechanisms while still having one dominant attribution path for control placement.

### C. Creation-Stage Threats

Creation-stage threats matter because they often stay hidden until the system acquires real authority. This stage is therefore the clearest example of why failure origin and failure manifestation should not be conflated.

Training poisoning and backdoors. Hidden Ghost Hand offers GUI-agent evidence, in an evaluated setting, that training-time triggers can later redirect behavior at inference time [155]. SlowBA adds newer evidence that reward-level or efficiency-oriented backdoors may also be inserted into GUI-agent optimization pipelines rather than only into obvious behavior-cloning traces [144]. BadVLA provides adjacent VLA evidence that similar backdoor logic can appear in multimodal action models more broadly [156]. Taken together, these studies are consistent with treating objective corruption as a creation-stage risk that may be introduced upstream and surface later.

Unsafe action priors and benchmark-shaped blind spots. Creation also decides what kinds of actions become normalized as ordinary. OS-Harm, CUAHarm, MobileSafetyBench, ST-WebAgentBench, and RiosWorld indicate that the affordances and policy constraints an evaluation suite foregrounds materially affect what risks become visible and measurable [40], [49], [148], [149], [157]. Those benchmarks do not by themselves establish deployment behavior, but they do suggest that high-authority operations can be normalized in the development loop before corresponding safety abstractions are in place.

The main security point is therefore upstream: some of the most consequential later failures may already be latent at creation time. Some failures that appear at runtime can often remain central because they exploit the same content-readiness that makes CUAs useful. InjecAgent, GhostEI-Bench, Chameleon, WebInject, attacker-controlled image-patch attacks, and active environmental injection studies provide evidence that attacker-controlled instructions or cues can be embedded in pages, screenshots, documents, mobile notifications, or on-screen image regions in ways that influence downstream action in evaluated settings [124], [126], [146], [162]–[164]. WAInjectBench and WASP add complementary evaluation evidence that these failures remain measurable in web-agent settings [41], [165]. EIA sharpens the privacy angle by suggesting that environmental injection can steer web agents toward leakage of user information rather than only generic task derailment [166]. Benchmarks centered on semantic-level UI injection, task-redirection, and dark-pattern manipulation further suggest that attacker-controlled input can be persuasive or attention-capturing rather than explicitly imperative [167]–[169]. Depending on how the attack succeeds, the dominant attribution may be objective corruption or environmental misbinding.

Defensive monitoring and mitigation. The corresponding defense literature also makes clear that prompt-injection resilience is not one thing. In-context defenses, localized attack detection, adversarial safety training, real-time audit frameworks, diagnostic guardrails, and policy-reasoning layers all aim at different points in the pipeline [125], [145], [147], [170]–[172]. That diversity is itself informative for the framework developed here: it suggests that input-side corruption is unlikely to be fully managed at one layer or one stage.

Decision-side corruption: memory poisoning and long-horizon steering. Once the agent maintains persistent state, poisoning memory or long-horizon planning can become as valuable as poisoning a single prompt. AgentPoison offers direct evidence for memory or knowledge-base poisoning [173]. LPS-Bench indicates that planning-time safety awareness can erode over long trajectories even when the attack surface is framed at the planning layer rather than as explicit memory poisoning [85]. PASB adds complementary evidence that personalized long-horizon deployments create broader attack surfaces in which attacker-favored behavior can persist across realistic toolchains and contexts [127]. Preference-redirection and benign-input elicitation studies add a related warning: the operative objective may be steered without always looking like a classic memory-poisoning event [174], [175]. Some cases more clearly replace the operative goal; others expand beyond the intended task boundary while still appearing superficially helpful. These can be read more accurately as creation-stage objective or action-space problems that only become visible once the system is trusted with real authority.

### D. Deployment-Stage Threats

Deployment-stage threats appear when learned capability is connected to real tools, permissions, and ingress routes. The central question at this stage is whether capability is bound to authority faster than it is bound to control.

Over-privilege and weak isolation. CaMeLs, CSAgent, and CellMate illustrate a shared deployment concern: in the studied settings, once a CUA is granted broad access without strong mediation, even moderate reasoning mistakes can lead to higher-impact outcomes [48], [158], [159]. These cases are best read as environmental misbinding. The model may not be pursuing an adversarial goal, but the authority configuration allows a local error to become a consequential one.

Supply-chain risk in tools, skills, and MCP servers. Les Dissonances and MCP Security Bench suggest that weakly governed tool or extension ecosystems create a second deployment path to failure [118], [119]. The two papers support different parts of the claim. Les Dissonances highlights cross-tool control-flow corruption in multi-tool agents. MCP Security Bench suggests that tool descriptions, standardized metadata, and protocol-level interfaces can carry attacker influence into planning and invocation. Together they motivate treating the tool layer as a deployment-time trust choice rather than as a neutral capability add-on.

Insufficient deployment-grounded evaluation. RedTeamCUA and HackWorld suggest that many hybrid web–OS attack paths and exploit-oriented failure modes can remain invisible in narrower evaluation settings [160], [161]. The broader lesson is methodological: when a system goes live without testing the actual coupling among observation, tool outputs, permissions, and oversight, the release process itself can become a security weakness.

Taken together, deployment-stage threats can be read as forms of misbound authority. They arise when tools, channels, permissions, and sessions are opened before provenance, isolation, and mediation are strong enough to constrain them.

### E. Operation-Stage Threats

Operation is where many CUA attack settings become consequential because mixed-trust inputs, live authority, and long-horizon planning coexist in one loop. OS-BLIND is a useful complement here because it suggests that higher-risk operational outcomes can also emerge under benign user instructions once contextual cues and subtask decomposition obscure harm [150]. The order below follows the typical path by which corruption becomes consequential: from what the agent reads, to what it retains or optimizes, to how it acts, and finally to how risk-bearing behavior may travel beyond one task instance.

Input-side corruption: injection and deceptive interfaces. Indirect prompt injection and deceptive UI attacks remain central because they exploit the same content-readiness that makes CUAs useful. InjecAgent, GhostEI-Bench, Chameleon, WebInject, attacker-controlled image-patch attacks, and active environmental injection studies provide evidence that attacker-controlled instructions or cues can be embedded in pages, screenshots, documents, mobile notifications, or on-screen image regions in ways that influence downstream action in evaluated settings [124], [126], [146], [162]–[164]. WAInjectBench and WASP add complementary evaluation evidence that these failures remain measurable in web-agent settings [41], [165].

Decision-side corruption: memory poisoning and long-horizon steering can move through interaction and sharing. Prompt Infection and broader multi-agent security work suggest that prompts, coordination patterns, or other risk-bearing behaviors can propagate beyond one-shot exploitation in multi-agent settings [178], [179]. Agent communities such as the Moltbook setting studied in [59] make those sharing channels easier to discuss as a deployment concern, even though that paper is not itself a security evaluation. The implication is not that every capability-sharing surface is attacker-controlled by default. It is that coordination itself becomes part of the trust boundary.

The operational pattern is cumulative. Attacker-controlled or misleading content can enter through observation, distort memory or planning, ride a high-authority execution channel, and then spread through coordination or sharing surfaces if no runtime control interrupts the chain.

### F. Privacy as a Parallel Objective

Privacy deserves separate treatment because it is not reducible to successful attack detection. Many privacy failures can occur even when the system is functioning as designed, yet retains too much, reads too broadly, or exports state through tools and telemetry in ways that exceed user expectations or task scope.

The major privacy surfaces recur across current CUA designs. Credential exposure may arise whenever passwords, session tokens, or secrets become visible in the observation loop. Cross-session leakage may arise when state learned in one task influences another. Memory retention risk may arise when the system keeps or retrieves more data than the task requires, even without adversarial prompting [180], [181]. Screenshot-, OCR-, and retrieval-induced disclosure may arise because incidental interface content can still be operationalized once it is observed. Third-party tool telemetry may arise when the tool ecosystem transmits or logs sensitive state beyond the base model's local context [119], [128]. Privacy-preserving deployment frameworks such as CORE further suggest that even ordinary inference routing can change how much UI state is exposed upstream or retained externally [182]. GUIGuard frames this granularity more explicitly by separating privacy recognition, privacy protection, and protected task execution in cross-platform GUI settings [183]. EIA complements that picture by suggesting that hidden environmental content can induce targeted privacy leakage even when the user-facing task appears ordinary [166]. Finally, data minimization and retention policy help determine whether any of these exposures become persistent.

Privacy failures map differently onto the attribution lens than many attack cases do. Some are straightforward cases of scope overreach, where the system over-collects or over-remembers while still "trying" to help. Others may be better read as objective corruption when attacker-controlled content induces disclosure or exfiltration. The important point is that many privacy controls sit earlier than generic attack detection: in memory scope, screenshot discipline, retention policy, secret handling, and tool-telemetry governance.

### G. Maintenance-Stage Threats

Maintenance-stage threats are what remain after the first release. They matter because a deployed CUA is not static. Interfaces change, registries evolve, extensions are updated, and controls that were once adequate can silently become stale.

Regression and stale assurance. TimeWarp and Risky-Bench suggest that deployment-grounded evaluation can reveal failure modes that narrower or more static settings miss, especially once interfaces and workflows evolve across versions [42], [133]. The lesson is broader than one benchmark: without re-evaluation and release gating after change, old controls can decay faster than deployment teams notice.

Persistent-state abuse and ecosystem drift. Your Agent, Their Asset and ClawKeeper suggest more directly that post-release security can be governed as much by persistent memory, extension hygiene, identity, and watcher or registry discipline as by base policy alone [128], [138]. PASB complements that picture by indicating how personalized long-lived deployments can reopen these risks in evaluation settings that are closer to real use [127]. The same maintenance problem applies to the assurance stack itself: policy reasoners and diagnostic defenses such as ShieldAgent, AgentSentinel, AgentDoG, WebSentinel, and DMAST only help if their detectors remain synchronized with evolving interfaces and attack patterns [145], [147], [170]–[172]. In open deployments, maintenance becomes the stage at which governance either keeps pace with capability diffusion or falls behind it.

**TABLE V**
**WORKING LIFECYCLE-GROUNDED SECURITY AND PRIVACY REFERENCE MAP FOR CUAS. EACH ROW IDENTIFIES WHERE A RISK CONDITION ENTERS, HOW IT BECOMES OPERATIONAL, AND ONE OR TWO ANCHOR PAPERS. "SO" = SCOPE OVERREACH, "OC" = OBJECTIVE CORRUPTION, AND "EM" = ENVIRONMENTAL MISBINDING.**

| Stage | Entry / Pressure Surface | Representative Risk Mechanism | Dominant Lens | Anchor Evidence |
|---|---|---|---|---|
| Creation | Decision-side objective shaping | Training poisoning, PoT backdoors, or over-broad objective priors | OC | Hidden Ghost Hand [155]; BadVLA [156] |
| Creation | Execution-side action shaping | Over-broad action vocabularies and high-impact affordances surfaced in evaluation | EM | OS-Harm [40], CUAHarm [157] |
| Deployment | Execution-side authority exposure | Broad authority exposure, limited isolation, or tool / channel misbinding | EM | CaMeLs [48], CSAgent [158] |
| Deployment | Decision-side dependency surface | Untrusted skills, MCP tools, or extension supply chains | OC | Les Dissonances [119], MCP Security Bench [118] |
| Operation | Input-side manipulation surface | Indirect prompt injection, deceptive UI cues, or manipulated tool outputs | OC / EM | WAInjectBench [41], Chameleon [164] |
| Operation | Decision-side state drift | Memory poisoning, planning-time goal drift, or long-horizon constraint forgetting | OC / SO | AgentPoison [173], LPS-Bench [85], PASB [127] |
| Operation | Execution-side invocation drift | TOCTOU, action rebinding, or delayed invocation misbinding | EM | Zero-Permission Manipulation [176], AgentHazard [177] |
| Operation | Privacy exposure surface | Credential exposure, PII overcollection, or cross-session leakage | SO / OC | GUIGuard [183], EIA [166] |
| Maintenance | Cross-stage assurance drift | Regression, stale controls, ecosystem drift, or persistent-state carryover | EM / OC | Risky-Bench [42], ClawKeeper [138] |

Table V compresses the chapter into one lifecycle-grounded reference map. It is intentionally selective rather than exhaustive: the evidence column names anchor papers, while the analytical value of the table comes from locating each risk by entry surface, operative mechanism, and lifecycle stage.

Taken together, the security and privacy analysis reinforces the organizing view developed here. CUA failures become more informative when read by where corruption entered, how it became operational, and at which lifecycle stage earlier controls might still have changed the outcome. Security is therefore not peripheral to the architecture–lifecycle framework; it is one of the clearer domains in which that framework helps organize evidence and control placement.

## VII. From Threats to Controls: Practical Security Implications

The analysis so far suggests a simple rule: the best control is usually the earliest one that still acts on the relevant failure mechanism. In deployed CUAs, that means control placement must follow both layer and stage. Creation shapes priors. Deployment binds authority. Operation determines whether higher-risk trajectories complete. Maintenance determines whether earlier gains survive environmental and ecosystem drift. A flat checklist cannot capture those differences.

This section converts the preceding analysis into governance implications. Its aim is not to catalogue every possible defense. Instead, it identifies which control surfaces remain meaningful once capability, authority, and mixed-trust content are already interacting in a live system. The broader implication is that open deployment is better understood through a deployment-oriented control framework than through model scaling alone.

### A. Lifecycle-Aligned Control Surfaces

The most defensible control surfaces line up with the four lifecycle stages because each stage changes a different object. Creation changes priors. Deployment changes authority binding. Operation changes the active trajectory. Maintenance changes whether the whole stack remains valid after release. Organizing controls by these stages is therefore more informative than organizing them only by tool category or attack name.

Design-time controls. At creation time, the relevant controls are those that shape priors before the system acquires live authority: grounding supervision, safe action abstractions, release gating on harmful-task suites, and policy design that values confirmation and recoverability rather than completion alone [38], [40], [113], [114], [157], [184]. The decisive question is whether the system is being optimized merely to finish tasks, or to finish them under explicit operational constraints.

Deploy-time controls. At deployment time, the most important controls govern authority binding: least privilege, sandboxing, provenance-aware middleware, tool allowlists, session scoping, channel separation, and explicit trust labeling for inputs and outputs [48], [158], [159], [182]. These are not wrappers around the model. They are part of what determines whether the resulting system is appropriate to operate in a given setting at all.

Runtime controls. During operation, the goal is to reduce the chance that uncertainty silently turns into irreversible action. Policy mediation, post-action verification, plan preview, confirmation at sensitive steps, interrupt or takeover paths, and explanation of why an action is being proposed all belong here [95], [185]–[187]. Recent safeguarded-execution systems make that design space more concrete: CORA calibrates execute-versus-abstain decisions under an explicit risk budget, while VeriSafe Agent verifies proposed mobile actions against formalized task constraints before they fire [129], [188]. Runtime security stacks such as AgentSentinel, AgentDoG, WebSentinel, and GEM add a complementary layer of monitoring, diagnosis, and uncertainty-aware escalation around the agent loop itself [145], [147], [171], [189]. Their purpose is to help keep user intent attached to the loop while the environment is changing underneath the system.

Maintenance controls. After release, the important controls are those that preserve validity over time: regression testing, red teaming, extension review, version pinning, coordinated patching, registry governance, and continuous re-evaluation against changing interfaces and threats [42], [128], [133], [138], [160], [170]. A deployed CUA is more likely to remain governable if its assurance stack evolves alongside its capability stack.

Under the attribution lens, the same logic becomes even more specific. Scope overreach is most directly addressed by task scoping, confirmation thresholds, memory boundaries, and data minimization. Objective corruption is most directly addressed by provenance separation, tool vetting, output mediation, and memory hygiene. Environmental misbinding is most directly addressed by execution mediation, TOCTOU-aware verification, least privilege, and auditable action binding. Control placement therefore follows both what failed and when the enabling condition entered.

### B. Human Oversight Should Be Designed, Not Assumed

Human oversight in CUAs should not be treated as a vague safety slogan. If the system is expected to handle long-horizon tasks under mixed-trust inputs, then the user or operator needs explicit product and policy surfaces through which they can inspect, stop, or redirect execution. Oversight is not a fallback for when the model fails. It is one of the primary mechanisms by which user intent remains attached to machine autonomy.

Five mechanisms recur as the most practical runtime oversight surfaces. Plan preview exposes the intended workflow before consequential execution begins. Step confirmation introduces a checkpoint when the next action is irreversible, ambiguous, or outside ordinary scope. Interrupt, pause, or takeover keeps changing environments from outrunning human correction. Action explanation preserves provenance about why the system believes a step is appropriate. Reversible-action preference biases the system toward drafts, dry runs, recycle-bin semantics, or staged execution whenever the task permits it. Benchmarks for interruptibility, collaborative assistance, and human interaction styles suggest that these are not merely interface niceties but measurable dimensions of deployed-agent quality [86], [87], [105].

These mechanisms are not just usability features. They are the points at which human intent remains operational inside the loop. The same logic should scale with action impact. Financial transfer, deletion, credential handling, permission changes, and external messaging do not need the same oversight regime, but they should all be mapped to one explicitly. Low-risk navigation may tolerate logging plus post-action verification. High-impact actions generally require preview, confirmation, or explicit approval.

### C. Control Ownership Is Distributed Across Actors

No single team can absorb the whole governance burden of a deployed CUA. Control ownership is distributed because the control surfaces themselves are distributed across the lifecycle. Model and system developers shape priors. Deployment teams bind the model to channels, permissions, tools, and sessions. Operators and end users govern escalation and approval during runtime. Ecosystem stewards govern registries, plugins, skills, and identity surfaces after release.

This separation matters because different failures trace back to different control owners. Insufficiently constrained development can introduce risky priors before anyone else has a chance to intervene. Loose deployment practice can expose too much authority even when the model itself is reasonable. Weak runtime oversight can let user intent dissolve during long trajectories. Loose ecosystem governance can reopen a previously bounded system through extensions, registries, or agent-to-agent exchange.

These roles are analytically distinct but operationally coupled. Careful training may not rescue a deployment that binds the model to broad permissions. Strong runtime approval may not fully rescue a system whose extension ecosystem is weakly governed. Public materials describing OpenClaw-like gateway assistants help illustrate this coupling in settings where persistent channels, tool bindings, sessions, and community interaction are presented as converging in one assistant surface [31], [32]. Such public descriptions are used only as illustrative deployment patterns rather than as verified system evidence. The broader point is that any open-deployment CUA that combines ingress, tooling, memory, and execution authority needs control ownership to remain traceable across those surfaces.

### D. A Conservative Baseline Stack for Open Deployment

The literature does not support any single defense as sufficient. It does, however, suggest a conservative baseline stack that repeatedly appears across safer deployment patterns. The list below is not an exhaustive defense catalogue. It is a compact control stack for open deployment: constrain what the system can do, preserve where instructions come from, bind authority safely, verify live execution, and keep assurance current after change.

- Constrained action interfaces. High-impact operations should prefer semantically narrow, auditable interfaces over unrestricted low-level authority whenever possible [152], [158], [184].
- Provenance-aware mediation. User instructions, retrieved content, tool outputs, and environmental text should remain distinguishable inside the control loop so that both the model and the operator can inspect source [185], [190].
- Sandboxed and scoped authority. Filesystem, network, tool, and account permissions should be deliberately bounded before deployment rather than tightened only after an incident [48], [158], [159].
- Runtime verification plus escalation. Post-action checking, plan preview, impact-aware approval, takeover paths, explicit safeguarded-execution layers, and runtime diagnosis are necessary to keep autonomy aligned once live use begins [95], [129], [171], [186]–[188].
- Continuous assurance. Regression suites, red teaming, extension review, replay against changed interfaces, detector updates, and signed or reviewed registries are needed so that maintenance-stage drift does not quietly reopen known weaknesses [42], [133], [138], [147], [160], [170].

These controls are cumulative. Constraining actions without provenance still leaves instruction confusion unresolved. Provenance without sandboxing still leaves broad authority intact. Runtime verification without maintenance discipline still decays as tools, interfaces, and ecosystems change. The practical implication is that dependable CUA deployment is unlikely to be achieved by any single alignment technique alone. It is more likely to depend on whether capability, authority, and oversight are connected through one coherent control stack.

## VIII. Open Problems and Future Directions

The framework clarifies deployed CUA reliability without claiming that the problem has been closed. Even if models continue to improve, several important questions remain unresolved. The hard part is no longer only capability. It is also attribution, evaluation, control placement, and post-release governance.

### A. Limits of the Analytical Lens

The lens emphasizes deployment-grounded diagnosis: where reliability issues are introduced, how they become operational, and where meaningful controls can still be attached. That emphasis leaves some questions only partially covered. The article does not aim to provide a formal causal model of agent failure, an exhaustive benchmark comparison, or a substitute for learning-theoretic, formal-methods, or proof-oriented security analysis. Its value is organizational and diagnostic rather than complete in every theoretical dimension.

### B. Attribution Still Lags Observation

The distinction among scope overreach, objective corruption, and environmental misbinding is analytically useful, but operationalizing it remains difficult. In realistic deployments, off-task helpfulness, silent goal drift, and attacker-induced exfiltration may produce similar surface behavior. Better provenance, telemetry, and causal tracing are still needed if attribution is to guide control placement reliably rather than merely explain incidents after the fact [48], [152].

### C. Benchmark Success Is Not Yet Release Readiness

Current evaluation suites capture important parts of the CUA problem, but they still underrepresent long-lived sessions, changing permissions, maintenance-stage drift, and open ecosystem interaction. As a result, benchmark scores should not yet be read as translating directly into deployment readiness. A major open direction is to connect benchmark design more tightly to release gating, regression testing, and post-release monitoring [13]–[15], [39], [40], [42], [49], [85], [157].

### D. Cross-Modal Robustness Remains Immature

Injection in CUAs is not only text injection. It may travel through screenshots, layout cues, parser artifacts, OCR-visible instructions, time-varying UI state, adversarial image patches, and attention-steering interface elements [41], [124], [126], [146], [164], [167], [174], [176]. Defenses that work on prompt text alone are therefore insufficient. A robust CUA will likely need perception and execution defenses that explicitly account for how multimodal grounding and authority binding interact.

### E. Capability Gains Still Create Policy Tension

Stronger models, richer tools, and better post-training do not automatically reduce policy tension; in some settings they can increase it. The more capable the agent becomes, the less obvious it is how that capability should translate into authority, autonomy, and approval requirements. Future work therefore needs to evaluate safety as a function of tool access, impact level, and runtime pressure rather than as a static property of the base model [157]. Broader agent-safety evidence such as PropensityBench points in the same direction, even though it is not CUA-specific: risk depends on both what a model can do in principle and what it is likely to attempt once consequential tools are available [191].

### F. Open Ecosystems Need Stronger Governance Primitives

A growing subset of open-deployment CUAs is embedded in tool registries, extension marketplaces, MCP-style ecosystems, and agent-to-agent environments. These settings appear to require stronger primitives for identity, attestation, registry hygiene, and capability containment than the field currently has [118], [128], [138], [178]. At the same time, stronger isolation can reduce collaboration quality, which means governance cannot be framed as pure restriction without considering utility trade-offs [179]. The field still lacks a stable answer for how open capability ecosystems should remain both useful and governable.

These open problems reinforce the central claim. Progress in CUAs is unlikely to come from model quality alone. It will more likely depend on better ways of linking architecture, lifecycle, authority, and control so that deployment remains intelligible after release rather than only impressive before it.

Those tensions suggest three priorities. First, the field needs controllable visual grounding that remains portable without sacrificing auditable execution. Second, it needs deployment-aware evaluation that measures authority binding, misuse resistance, and oversight behavior alongside task capability. Third, it needs maintenance-aware governance so that models, tools, memory, identities, and extensions remain governable after release instead of drifting into insecurity.

Taken together, the surveyed literature and public deployment patterns suggest that dependable computer use is unlikely to be achieved by stronger models alone. It also depends on how capability is bound to authority, how runtime uncertainty is mediated, and how assurance is maintained after deployment. The main value of the architecture–lifecycle view is therefore to help locate where problems are introduced, where they become visible, and where meaningful controls can still be applied.

## IX. Conclusion

Computer-use agents can be productively analyzed as deployed interactive systems rather than as benchmark policies with stronger prompting alone. Their reliability depends jointly on how they reconstruct interface state, preserve task intent, execute through real authority surfaces, and continue to evolve after release.

The preceding sections developed that argument through a joint architecture–lifecycle framework. The tri-layer architecture traced where actionable state is reconstructed, where intent is stabilized, and where authority is exercised. The lifecycle account traced when capability is formed, when authority is bound, when failures first become visible, and where controls may still intervene earliest. The value of combining the two is organizational and diagnostic: it helps separate visible failure from upstream cause and ties practical control to the right stage of system evolution.

Three tensions remain central. The first is portability versus controllability: the observation and execution channels that generalize most broadly are often the hardest to mediate and verify precisely. The second is autonomy versus oversight: long-horizon execution becomes valuable only when user intent, provenance, and approval remain attached to the loop. The third is adaptation versus regression: deployed CUAs need to evolve with changing interfaces and ecosystems, yet every update can reopen earlier weaknesses or create new ones.

Those tensions suggest three priorities. First, the field needs controllable visual grounding that remains portable without sacrificing auditable execution. Second, it needs deployment-aware evaluation that measures authority binding, misuse resistance, and oversight behavior alongside task capability. Third, it needs maintenance-aware governance so that models, tools, memory, identities, and extensions remain governable after release instead of drifting into insecurity.

Taken together, the surveyed literature and public deployment patterns suggest that dependable computer use is unlikely to be achieved by stronger models alone. It also depends on how capability is bound to authority, how runtime uncertainty is mediated, and how assurance is maintained after deployment. The main value of the architecture–lifecycle view is therefore to help locate where problems are introduced, where they become visible, and where meaningful controls can still be applied.

## References

[1] Xinyuan Wang, Bowen Wang, Dunjie Lu, Junlin Yang, Tianbao Xie, Junli Wang, Jiaqi Deng, Xiaole Guo, Yiheng Xu, Chen Henry Wu, et al. Opencua: Open foundations for computer-use agents. arXiv preprint arXiv:2508.09123, 2025.

[2] Gonzalo Gonzalez-Pumariega, Vincent Tu, Chih-Lun Lee, Jiachen Yang, Ang Li, and Xin Eric Wang. The unreasonable effectiveness of scaling agents for computer use. arXiv preprint arXiv:2510.02250, 2025.

[3] Haiyang Xu, Xi Zhang, Haowei Liu, Junyang Wang, Zhaozai Zhu, Shengjie Zhou, Xuhao Hu, Feiyu Gao, Junjie Cao, Zihua Wang, et al. Mobile-agent-v3.5: Multi-platform fundamental gui agents. arXiv preprint arXiv:2602.16855, 2026.

[4] Hanzhang Zhou, Xu Zhang, Panrong Tong, Jianan Zhang, Liangyu Chen, Quyu Kong, Chenglin Cai, Chen Liu, Yue Wang, Jingren Zhou, et al. Mai-ui technical report: Real-world centric foundation gui agents. arXiv preprint arXiv:2512.22047, 2025.

[5] Haolong Yan, Jia Wang, Xin Huang, Yeqing Shen, Ziyang Meng, Zhimin Fan, Kaijun Tan, Jin Gao, Lieyu Shi, Mi Yang, et al. Step-gui technical report. arXiv preprint arXiv:2512.15431, 2025.

[6] Mathieu Andreux, Märt Bakler, Yanael Barbier, Hamza Benchekroun, Emilien Biré, Antoine Bonnet, Riaz Bordie, Nathan Bout, Matthias Brunel, Aleix Cambray, et al. Surfer 2: The next generation of cross-platform computer use agents. arXiv preprint arXiv:2510.19949, 2025.

[7] Tanmay Gupta, Piper Wolters, Zixian Ma, Peter Sushko, Rock Yuren Pang, Diego Llanes, Yue Yang, Taira Anderson, Boyuan Zheng, Zhongzheng Ren, et al. Molmoweb: Open visual web agent and open data for the open web. arXiv preprint arXiv:2604.08516, 2026.

[8] Xiang Deng, Yu Gu, Boyuan Zheng, Shijie Chen, Sam Stevens, Boshi Wang, Huan Sun, and Yu Su. Mind2web: Towards a generalist agent for the web. Advances in Neural Information Processing Systems, 36:28091–28114, 2023.

[9] Shuyan Zhou, Frank F Xu, Hao Zhu, Xuhui Zhou, Robert Lo, Abishek Sridhar, Xianyi Cheng, Tianyue Ou, Yonatan Bisk, Daniel Fried, et al. Webarena: A realistic web environment for building autonomous agents. arXiv preprint arXiv:2307.13854, 2023.

[10] Jing Yu Koh, Robert Lo, Lawrence Jang, Vikram Duvvur, Ming Lim, Po-Yu Huang, Graham Neubig, Shuyan Zhou, Russ Salakhutdinov, and Daniel Fried. Visualwebarena: Evaluating multimodal agents on realistic visual web tasks. In Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pages 881–905, 2024.

[11] Alexandre Drouin, Maxime Gasse, Massimo Caccia, Issam H Laradji, Manuel Del Verme, Tom Marty, Léo Boisvert, Megh Thakkar, Quentin Cappart, David Vazquez, et al. Workarena: How capable are web agents at solving common knowledge work tasks? arXiv preprint arXiv:2403.07718, 2024.

[12] Léo Boisvert, Megh Thakkar, Maxime Gasse, Massimo Caccia, Thibault L De Chezelles, Quentin Cappart, Nicolas Chapados, Alexandre Lacoste, and Alexandre Drouin. Workarena++: Towards compositional planning and reasoning-based common knowledge work tasks. Advances in Neural Information Processing Systems, 37:5996–6051, 2024.

[13] Peng Yuan, Yuyang Yin, Yuxuan Cai, and Zheng Wei. Webforge: Breaking the realism-reproducibility-scalability trilemma in browser agent benchmark. arXiv preprint arXiv:2604.10988, 2026.

[14] Yuxuan Zhang, Yubo Wang, Yipeng Zhu, Penghui Du, Junwen Miao, Xuan Lu, Wendong Xu, Yunzhuo Hao, Songcheng Cai, Xiaochen Wang, et al. Clawbench: Can ai agents complete everyday online tasks? arXiv preprint arXiv:2604.08523, 2026.

[15] Tongbo Chen, Zhengxi Lu, Zhan Xu, Guocheng Shao, Shaohan Zhao, Fei Tang, Yong Du, Kaitao Song, Yizhou Liu, Yuchen Yan, et al. Knowu-bench: Towards interactive, proactive, and personalized mobile agent evaluation. arXiv preprint arXiv:2604.08455, 2026.

[16] Hongyi Nie, Xunyuan Liu, Yudong Bai, Yaqing Wang, Yang Liu, Quanming Yao, and Zhen Wang. Pspa-bench: A personalized benchmark for smartphone gui agent. arXiv preprint arXiv:2603.29318, 2026.

[17] Tianbao Xie, Danyang Zhang, Jixuan Chen, Xiaochuan Li, Siheng Zhao, Ruisheng Cao, Toh J Hua, Zhoujun Cheng, Dongchan Shin, Fangyu Lei, et al. Osworld: Benchmarking multimodal agents for open-ended tasks in real computer environments. Advances in Neural Information Processing Systems, 37:52040–52094, 2024.

[18] Jihong Wang, Jiamu Zhou, Weiming Zhang, Teng Wang, Weiwen Liu, Zhuosheng Zhang, Xingyu Lou, Weinan Zhang, HUARONG DENG, and Jun Wang. Colorbrowseragent: Complex long-horizon browser agent with adaptive knowledge evolution. In The 64th Annual Meeting of the Association for Computational Linguistics–Industry Track, 2026.

[19] Mingxian Yu, Siqi Luo, and Xu Chen. GraphPilot: GUI Task Automation with One-Step LLM Reasoning Powered by Knowledge Graph, jan 2026. Journal of Intelligent Computing and Networking.

[20] Yichun Zhang, Xiangwu Guo, Yauhong Goh, Jessica Hu, Zhiheng Chen, Xin Wang, Difei Gao, and Mike Zheng Shou. Showui-aloha: Human-taught gui agent. arXiv preprint arXiv:2601.07181, 2026.

[21] Dang Nguyen, Jian Chen, Yu Wang, Gang Wu, Namyong Park, Zhengmian Hu, Hanjia Lyu, Junda Wu, Ryan Aponte, Yu Xia, et al. Gui agents: A survey. In Findings of the Association for Computational Linguistics: ACL 2025, pages 22522–22538, 2025.

[22] Xueyu Hu, Tao Xiong, Biao Yi, Zishu Wei, Ruixuan Xiao, Yurun Chen, Jiasheng Ye, Meiling Tao, Xiangxin Zhou, Ziyu Zhao, et al. Os agents: A survey on mllm-based agents for computer, phone and browser use. In Proceedings of the 63rd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pages 7436–7465, 2025.

[23] Fei Tang, Haolei Xu, Hang Zhang, Siqi Chen, Xingyu Wu, Yongliang Shen, Wenqi Zhang, Guiyang Hou, Zeqi Tan, Yuchen Yan, et al. A survey on (m)llm-based gui agents. arXiv preprint arXiv:2504.13865, 2025.

[24] Shuai Wang, Weiwen Liu, Jingxuan Chen, Yuqi Zhou, Weinan Gan, Xingshan Zeng, Yuhan Che, Shuai Yu, Xinlong Hao, Kun Shao, et al. Gui agents with foundation models: A comprehensive survey. arXiv preprint arXiv:2411.04890, 2024.

[25] Minghe Gao, Wendong Bu, Bingchen Miao, Yang Wu, Yunfei Li, Juncheng Li, Siliang Tang, Qi Wu, Yueting Zhuang, and Meng Wang. Generalist virtual agents: A survey on autonomous agents across digital platforms. arXiv preprint arXiv:2411.10943, 2024.

[26] Yucheng Shi, Wenhao Yu, Wenlin Yao, Wenhu Chen, and Ninghao Liu. Towards trustworthy gui agents: A survey. arXiv preprint arXiv:2503.23434, 2025.

[27] Jiahao Li and Kaer Huang. A survey on gui agents with foundation models enhanced by reinforcement learning. arXiv preprint arXiv:2504.20464, 2025.

[28] Guangyi Liu, Pengxiang Zhao, Yaozhen Liang, Liang Liu, Yaxuan Guo, Han Xiao, Weifeng Lin, Yuxiang Chai, Yue Han, Shuai Ren, et al. Llm-powered gui agents in phone automation: Surveying progress and prospects. arXiv preprint arXiv:2504.19838, 2025.

[29] Liangbo Ning, Ziran Liang, Zhuohang Jiang, Haohao Qu, Yujuan Ding, Wenqi Fan, Xiao-yong Wei, Shanru Lin, Hui Liu, Philip S Yu, et al. A survey of webagents: Towards next-generation ai agents for web automation with large foundation models. In 31st ACM SIGKDD Conference on Knowledge Discovery and Data Mining (KDD 2025), pages 6140–6150. Association for Computing Machinery, 2025.

[30] Ada Chen, Yongjiang Wu, Junyuan Zhang, Jingyu Xiao, Shu Yang, Jentse Huang, Kun Wang, Wenxuan Wang, and Shuai Wang. A survey on the safety and security threats of computer-using agents: Jarvis or ultron? arXiv preprint arXiv:2505.10924, 2025.

[31] OpenClaw. Openclaw — personal ai assistant. https://openclaw.ai/, 2026. Website, accessed: 2026-04-11.

[32] Aili McConnon. Openclaw, moltbook and the future of ai agents. https://www.ibm.com/think/news/clawdbot-ai-agent-testing-limits-vertical-integration, 2026. IBM Think article, accessed: 2026-04-11.

[33] Fei Tang, Zhiqiong Lu, Boxuan Zhang, Weiming Lu, Jun Xiao, Yueting Zhuang, and Yongliang Shen. Clawgui: A unified framework for training, evaluating, and deploying gui agents. arXiv preprint arXiv:2604.11784, 2026.

[34] Bin Kang, Shaoguo Wen, Yifei Bi, Shunlong Wu, Xinbin Yuan, Rui Shao, Junle Wang, and Zhuotao Tian. LongHorizonUI: A Unified Framework for Robust long-horizon Task Automation of GUI Agent. In International Conference on Learning Representations, 2026. Poster.

[35] Shuyan Zhou. Webarena-infinity: Generating browser environments with verifiable tasks at scale. shuyanzhou.com, March 2026.

[36] Nathan Zhao. Webpii: Benchmarking visual pii detection for computer-use agents. arXiv preprint arXiv:2603.17357, 2026.

[37] Yuwen Zhai, Runze Li, Liang Wang, Nian Shi, Liwu Xu, Wei Zhang, Ran Lin, Bo Xu, and Benlei Cui. Guide: Interpretable gui agent evaluation via hierarchical diagnosis. arXiv preprint arXiv:2604.04399, 2026.

[38] Haojia Lin, Xiaoyu Tan, Yulei Qin, Zihan Xu, Yuchen Shi, Zongyi Li, Gang Li, Shaofei Cai, Siqi Cai, Chaoyou Fu, et al. Cuarewardbench: A benchmark for evaluating reward models on computer-using agent. arXiv preprint arXiv:2510.18596, 2025.

[39] Pranjal Aggarwal, Graham Neubig, and Sean Welleck. Gym-anything: Turn any software into an agent environment. arXiv preprint arXiv:2604.06126, 2026.

[40] Thomas Kuntz, Agatha Duzan, Hao Zhao, Francesco Croce, Zico Kolter, Nicolas Flammarion, and Maksym Andriushchenko. Os-harm: A benchmark for measuring safety of computer use agents. arXiv preprint arXiv:2506.14866, 2025.

[41] Yinuo Liu, Ruohan Xu, Xilong Wang, Yuqi Jia, and Neil Zhenqiang Gong. Wainjectbench: Benchmarking prompt injection detections for web agents. arXiv preprint arXiv:2510.01354, 2025.

[42] Jingnan Zheng, Yanzhen Luo, Jingjun Xu, Bingnan Liu, Yuxin Chen, Chenhang Cui, Gelei Deng, Chaochao Lu, Xiang Wang, An Zhang, et al. Risky-bench: Probing agentic safety risks under real-world deployment. arXiv preprint arXiv:2602.03100, 2026.

[43] Sai Teja Reddy Adapala and Yashwanth Reddy Alugubelly. The aegis protocol: A foundational security framework for autonomous ai agents. arXiv preprint arXiv:2508.19267, 2025.

[44] Sahaya Jestus Lazer, Kshitiz Aryal, Maanak Gupta, and Elisa Bertino. A survey of agentic ai and cybersecurity: Challenges, opportunities and use-case prototypes. arXiv preprint arXiv:2601.05293, 2026.

[45] Zhiyong Wu, Chengcheng Han, Zichen Ding, Zhenmin Weng, Zhoumianze Liu, Shunyu Yao, Tao Yu, and Lingpeng Kong. Os-copilot: Towards generalist computer agents with self-improvement. arXiv preprint arXiv:2402.07456, 2024.

[46] Linxin Song, Yutong Dai, Viraj Prabhu, Jieyu Zhang, Taiwei Shi, Li Li, Junnan Li, Silvio Savarese, Zeyuan Chen, Jieyu Zhao, et al. Coact-1: Computer-using agents with coding as actions. arXiv preprint arXiv:2508.03923, 2025.

[47] Hongliang He, Wenlin Yao, Kaixin Ma, Wenhao Yu, Yong Dai, Hongming Zhang, Zhenzhong Lan, and Dong Yu. Webvoyager: Building an end-to-end web agent with large multimodal models. In Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pages 6864–6890, 2024.

[48] Hanna Foerster, Robert Mullins, Tom Blanchard, Nicolas Papernot, Kristina Nikolić, Florian Tramèr, Ilia Shumailov, Cheng Zhang, and Yiren Zhao. Camels can use computers too: System-level security for computer use agents. arXiv preprint arXiv:2601.09923, 2026.

[49] Jingyi Yang, Shuai Shao, Dongrui Liu, and Jing Shao. Riosworld: Benchmarking the risk of multimodal computer-use agents. arXiv preprint arXiv:2506.00618, 2025.

[50] Keen You, Haotian Zhang, Eldon Schoop, Floris Weers, Amanda Swearngin, Jeffrey Nichols, Yinfei Yang, and Zhe Gan. Ferret-ui: Grounded mobile ui understanding with multimodal llms. In European Conference on Computer Vision, pages 240–255. Springer, 2024.

[51] Kunal Singh, Shreyas Singh, and Mukund Khanna. Trishul: Towards region identification and screen hierarchy understanding for large vlm based gui agents. In Proceedings of the Computer Vision and Pattern Recognition Conference, pages 170–179, 2025.

[52] Yue Fan, Handong Zhao, Ruiyi Zhang, Yu Shen, Xin Eric Wang, and Gang Wu. Gui-bee: Align gui action grounding to novel environments via autonomous exploration. In Proceedings of the 2025 Conference on Empirical Methods in Natural Language Processing, pages 33249–33266, 2025.

[53] Xinbin Yuan, Jian Zhang, Kaixin Li, Zhuoxuan Cai, Lujian Yao, Jie Chen, Enguang Wang, Qibin Hou, Jinwei Chen, Peng-Tao Jiang, and Bo Li. Enhancing visual grounding for gui agents via self-evolutionary reinforcement learning. arXiv preprint arXiv:2505.12370, 2025.

[54] Dezhao Luo, Bohan Tang, Kang Li, Georgios Papoudakis, Jifei Song, Shaogang Gong, Jianye Hao, Jun Wang, and Kun Shao. Vimo: A generative visual gui world model for app agent. arXiv preprint arXiv:2504.13936, 2025.

[55] Yiping Xie, Song Chen, Jingxuan Xing, Wei Jiang, Zekun Zhu, Yingyao Wang, Pi Bu, Jun Song, Yuning Jiang, and Bo Zheng. Secagent: Efficient mobile gui agent with semantic context. arXiv preprint arXiv:2603.08533, 2026.

[56] Jiali Cheng, Anjishnu Kumar, Roshan Lal, Rishi Rajasekaran, Hani Ramezani, Omar Zia Khan, Oleg Rokhlenko, Sunny Chiu-Webster, Gang Hua, and Hadi Amiri. Webatlas: An llm agent with experience-driven memory and action simulation. arXiv preprint arXiv:2510.22732, 2025.

[57] Danqing Zhang, Balaji Rama, Jingyi Ni, Shiying He, Fu Zhao, Kunyu Chen, Arnold Chen, and Junyu Cao. Litewebagent: The open-source suite for vlm-based web-agent applications. In Proceedings of the 2025 Conference of the Nations of the Americas Chapter of the Association for Computational Linguistics: Human Language Technologies (System Demonstrations), pages 449–455, 2025.

[58] Izzeddin Gur, Hiroki Furuta, Austin Huang, Mustafa Safdari, Yutaka Matsuo, Douglas Eck, and Aleksandra Faust. A real-world webagent with planning, long context understanding, and program synthesis. arXiv preprint arXiv:2307.12856, 2023.

[59] Eason Chen, Ce Guan, Ahmed Elshafiey, Zhonghao Zhao, Joshua Zekeri, Afeez Edeifo Shaibu, and Emmanuel Osadebe Prince. When openclaw ai agents teach each other: Peer learning patterns in the moltbook community. arXiv preprint arXiv:2602.14477, 2026.

[60] Yadong Lu, Jianwei Yang, Yelong Shen, and Ahmed Awadallah. Omniparser for pure vision based gui agent. arXiv preprint arXiv:2408.00203, 2024.

[61] Zheng Hui, Yinheng Li, Tianyi Chen, Colby Banbury, Kazuhito Koishida, et al. Winclick: Gui grounding with multimodal large language models. arXiv preprint arXiv:2503.04730, 2025.

[62] Siqi Pei, Liang Tang, Tiaonan Duan, Long Chen, Shuxian Li, Kaer Huang, Yanzhe Jing, Yiqiang Yan, Bo Zhang, Chenghao Jiang, et al. Adazoom-gui: Adaptive zoom-based gui grounding with instruction refinement. arXiv preprint arXiv:2603.17441, 2026.

[63] Longtao Zheng, Rundong Wang, Xinrun Wang, and Bo An. Synapse: Trajectory-as-exemplar prompting with memory for computer control. arXiv preprint arXiv:2306.07863, 2023.

[64] Chaoyun Zhang, Liqun Li, Shilin He, Xu Zhang, Bo Qiao, Si Qin, Minghua Ma, Yu Kang, Qingwei Lin, Saravan Rajmohan, et al. Ufo: A ui-focused agent for windows os interaction. In Proceedings of the 2025 Conference of the Nations of the Americas Chapter of the Association for Computational Linguistics: Human Language Technologies (Volume 1: Long Papers), pages 597–622, 2025.

[65] Tiantian He, Yihang Chen, Keyue Jiang, Ka Yiu Lee, Kaiwen Zhou, Kun Shao, and Shuai Wang. Ee-mcp: Self-evolving mcp-gui agents via automated environment generation and experience learning. arXiv preprint arXiv:2604.09815, 2026.

[66] Deyang Jiang, Jing Huang, Xuanle Zhao, Lei Chen, Liming Zheng, Fanfan Liu, Haibo Qiu, Peng Shi, and Zhixiong Zeng. Treecua: Efficiently scaling gui automation with tree-structured verifiable evolution. arXiv preprint arXiv:2602.09662, 2026.

[67] Wenjia Jiang, Yangyang Zhuang, Chenxi Song, Xu Yang, Joey Tianyi Zhou, and Chi Zhang. Appagentx: Evolving gui agents as proficient smartphone users. arXiv preprint arXiv:2503.02268, 2025.

[68] Jian Mu, Chaoyun Zhang, Chiming Ni, Lu Wang, Bo Qiao, Kartik Mathur, Qianhui Wu, Yuhang Xie, Xiaojun Ma, Mengyu Zhou, et al. Gui-360°: A comprehensive dataset and benchmark for computer-using agents. arXiv preprint arXiv:2511.04307, 2025.

[69] Siyuan Hu, Kevin Qinghong Lin, and Mike Zheng Shou. Showui-π: Flow-based generative models as gui dexterous hands. arXiv preprint arXiv:2512.24965, 2025.

[70] Simon Yu, Gang Li, Weiyan Shi, and Peng Qi. Polyskill: Learning generalizable skills through polymorphic abstraction. arXiv preprint arXiv:2510.15863, 2025.

[71] Kanzhi Cheng, Qiushi Sun, Yougang Chu, Fangzhi Xu, Li YanTao, Jianbing Zhang, and Zhiyong Wu. Seeclick: Harnessing gui grounding for advanced visual gui agents. In Proceedings of the 62nd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pages 9313–9332, 2024.

[72] Junyang Wang, Haiyang Xu, Jiabo Ye, Ming Yan, Weizhou Shen, Ji Zhang, Fei Huang, and Jitao Sang. Mobile-agent: Autonomous multi-modal mobile device agent with visual perception. arXiv preprint arXiv:2401.16158, 2024.

[73] Gilles Baechler, Srinivas Sunkara, Maria Wang, Fedir Zubach, Hassan Mansoor, Vincent Etter, Victor Cărbune, Jason Lin, Jindong Chen, and Abhanshu Sharma. Screenai: A vision-language model for ui and infographics understanding. arXiv preprint arXiv:2402.04615, 2024.

[74] A Said Gurbuz, Sunghwan Hong, Ahmed Nassar, Marc Pollefeys, and Peter Staar. Moving beyond sparse grounding with complete screen parsing supervision. arXiv preprint arXiv:2602.14276, 2026.

[75] Wenyi Hong, Weihan Wang, Qingsong Lv, Jiazheng Xu, Wenmeng Yu, Junhui Ji, Yan Wang, Zihan Wang, Yuxiao Dong, Ming Ding, et al. Cogagent: A visual language model for gui agents. In Proceedings of the IEEE/CVF conference on computer vision and pattern recognition, pages 14281–14290, 2024.

[76] Rohan Bavishi, Erich Elsen, Curtis Hawthorne, Maxwell Nye, Augustus Odena, Arushi Somani, and Sağnak Taşırlar. Introducing our multimodal models, 2023.

[77] Jinze Bai, Shuai Bai, Shusheng Yang, Shijie Wang, Sinan Tan, Peng Wang, Junyang Lin, Chang Zhou, and J Qwen-VL Zhou. A versatile vision-language model for understanding, localization, text reading, and beyond. arXiv preprint arXiv:2308.12966, 6:3, 2023.

[78] Neeraj Anand, Rishabh Jain, Sohan Patnaik, Balaji Krishnamurthy, and Mausoom Sarkar. Afragent: An adaptive feature renormalization based high resolution aware gui agent. In Proceedings of the IEEE/CVF Winter Conference on Applications of Computer Vision, pages 1147–1158, 2026.

[79] Kaixin Li, Ziyang Meng, Hongzhan Lin, Ziyang Luo, Yuchen Tian, Jing Ma, Zhiyong Huang, and Tat-Seng Chua. ScreenSpot-Pro: GUI Grounding for Professional High-Resolution Computer Use. In Proceedings of the 33rd ACM International Conference on Multimedia, pages 8778–8786. ACM, 2025.

[80] Chi Zhang, Zhao Yang, Jiaxuan Liu, Yanda Li, Yucheng Han, Xin Chen, Zebiao Huang, Bin Fu, and Gang Yu. Appagent: Multimodal agents as smartphone users. In Proceedings of the 2025 CHI Conference on Human Factors in Computing Systems, pages 1–20, 2025.

[81] Yuhang Liu, Pengxiang Li, Zishu Wei, Congkai Xie, Xueyu Hu, Xinchen Xu, Shengyu Zhang, Xiaotian Han, Hongxia Yang, and Fei Wu. Infiguiagent: A multimodal generalist gui agent with native reasoning and reflection. arXiv preprint arXiv:2501.04575, 2025.

[82] Yibo Shi, Jungang Li, Linghao Zhang, Zihao Dongfang, Biao Wu, Sicheng Tao, Yibo Yan, Chenxi Qin, Weiting Liu, Zhixin Lin, et al. Androtmem: From interaction trajectories to anchored memory in long-horizon gui agents. arXiv preprint arXiv:2603.18429, 2026.

[83] Penghao Wu, Shengnan Ma, Bo Wang, Jiaheng Yu, Lewei Lu, and Ziwei Liu. Gui-reflection: Empowering multimodal gui models with self-reflection behavior. arXiv preprint arXiv:2506.08012, 2025.

[84] Ning Li, Xiangmou Qu, Jiamu Zhou, Jun Wang, Muning Wen, Kounianhua Du, Xingyu Lou, Qiuying Peng, Jun Wang, and Weinan Zhang. Mobileuse: A gui agent with hierarchical reflection for autonomous mobile operation. arXiv preprint arXiv:2507.16853, 2025.

[85] Tianyu Chen, Chujia Hu, Ge Gao, Dongrui Liu, Xia Hu, and Wenjie Wang. Lps-bench: Benchmarking safety awareness of computer-use agents in long-horizon planning under benign and adversarial scenarios. arXiv preprint arXiv:2602.03255, 2026.

[86] Henry Peng Zou, Chunyu Miao, Wei-Chieh Huang, Yankai Chen, Yue Zhou, Hanrong Zhang, Yaozu Wu, Liancheng Fang, Zhengyao Gu, Zhen Zhang, et al. When users change their mind: Evaluating interruptible agents in long-horizon web navigation. arXiv preprint arXiv:2604.00892, 2026.

[87] Saelyne Yang, Jaesang Yu, Yi-Hao Peng, Kevin Qinghong Lin, Jae Won Cho, Yale Song, and Juho Kim. Guide: A benchmark for understanding and assisting users in open-ended gui tasks. arXiv preprint arXiv:2603.25864, 2026.

[88] Shunyu Yao, Jeffrey Zhao, Dian Yu, Nan Du, Izhak Shafran, Karthik R Narasimhan, and Yuan Cao. React: Synergizing reasoning and acting in language models. In The eleventh international conference on learning representations, 2022.

[89] Shunyu Yao, Dian Yu, Jeffrey Zhao, Izhak Shafran, Tom Griffiths, Yuan Cao, and Karthik Narasimhan. Tree of thoughts: Deliberate problem solving with large language models. Advances in neural information processing systems, 36:11809–11822, 2023.

[90] Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik Narasimhan, and Shunyu Yao. Reflexion: Language agents with verbal reinforcement learning. Advances in neural information processing systems, 36:8634–8652, 2023.

[91] Zhuosheng Zhang and Aston Zhang. You only look at screens: Multimodal chain-of-action agents. In Findings of the Association for Computational Linguistics: ACL 2024, pages 3132–3149, 2024.

[92] Haitao Jia, Ming He, Zimo Yin, Likang Wu, Jianping Fan, and Jitao Sang. Reinagent: A context-aware gui agent enabling human-in-the-loop mobile task navigation. arXiv preprint arXiv:2510.07988, 2025.

[93] Qinzhuo Wu, Pengzhi Gao, Wei Liu, and Jian Luan. Backtrackagent: Enhancing gui agent with error detection and backtracking mechanism. In Proceedings of the 2025 Conference on Empirical Methods in Natural Language Processing, pages 4250–4272, 2025.

[94] Fanglin Mo, Junzhe Chen, Haoxuan Zhu, and Xuming Hu. Building a stable planner: An extended finite state machine based planning module for mobile gui agent. arXiv preprint arXiv:2505.14141, 2025.

[95] Yuzhe Zhang, Xianwei Xue, Xingyong Wu, Mengke Chen, Chen Liu, Xinran He, Run Shao, Feiran Liu, Huanmin Xu, Qiutong Pan, et al. Don't act blindly: Robust gui automation via action-effect verification and self-correction. arXiv preprint arXiv:2604.05477, 2026.

[96] Linqiang Guo, Wei Liu, Yi Wen Heng, Tse-Hsun Peter Chen, and Yang Wang. Agent-sama: State-aware mobile assistant. In Proceedings of the AAAI Conference on Artificial Intelligence, volume 40, pages 29459–29467, 2026.

[97] Weihao Tan, Wentao Zhang, Xinrun Xu, Haochong Xia, Ziluo Ding, Boyu Li, Bohan Zhou, Junpeng Yue, Jiechuan Jiang, Yewen Li, et al. Cradle: Empowering foundation agents towards general computer control. arXiv preprint arXiv:2403.03186, 2024.

[98] Daniel Toyama, Philippe Hamel, Anita Gergely, Gheorghe Comanici, Amelia Glaese, Zafarali Ahmed, Tyler Jackson, Shibl Mourad, and Doina Precup. Androidenv: A reinforcement learning platform for android. arXiv preprint arXiv:2105.13231, 2021.

[99] Haoming Wang, Haoyang Zou, Huatong Song, Jiazhan Feng, Junjie Fang, Junting Lu, Longxiang Liu, Qinyu Luo, Shihao Liang, Shijue Huang, et al. Ui-tars-2 technical report: Advancing gui agent with multi-turn reinforcement learning. arXiv preprint arXiv:2509.02544, 2025.

[100] Runliang Niu, Jindong Li, Shiqi Wang, Yali Fu, Xiyu Hu, Xueyuan Leng, He Kong, Yi Chang, and Qi Wang. Screenagent: A vision language model-driven computer control agent. arXiv preprint arXiv:2402.07945, 2024.

[101] Le Zhang, Yixiong Xiao, Xinjiang Lu, Jingjia Cao, Yusai Zhao, Jingbo Zhou, Lang An, Zikan Feng, Wanxiang Sha, Yu Shi, et al. Omegause: Building a general-purpose gui agent for autonomous task execution. arXiv preprint arXiv:2601.20380, 2026.

[102] Pei Yang, Hai Ci, and Mike Zheng Shou. macosworld: A multilingual interactive benchmark for gui agents. arXiv preprint arXiv:2506.04135, 2025.

[103] Guruprasad Viswanathan Ramesh, Asmit Nayak, Basieem Siddique, and Kassem Fawaz. Websp-eval: Evaluating web agents on website security and privacy tasks. arXiv preprint arXiv:2604.06367, 2026.

[104] Jiachen Zhu, Lingyu Yang, Rong Shan, Congmin Zheng, Zeyu Zheng, Weiwen Liu, Yong Yu, Weinan Zhang, and Jianghao Lin. Turing test on screen: A benchmark for mobile gui agent humanization. arXiv preprint arXiv:2604.09574, 2026.

[105] Faria Huq, Zora Zhiruo Wang, Zhanqiu Guo, Venu Arvind Arangarajan, Tianyue Ou, Frank Xu, Shuyan Zhou, Graham Neubig, and Jeffrey P Bigham. Modeling distinct human interaction in web agents. arXiv preprint arXiv:2602.17588, 2026.

[106] Christopher Rawles, Alice Li, Daniel Rodriguez, Oriana Riva, and Timothy Lillicrap. Androidinthewild: A large-scale dataset for android device control. Advances in Neural Information Processing Systems, 36:59708–59728, 2023.

[107] Aohan Zeng, Mingdao Liu, Rui Lu, Bowen Wang, Xiao Liu, Yuxiao Dong, and Jie Tang. Agenttuning: Enabling generalized agent abilities for llms. In Findings of the Association for Computational Linguistics: ACL 2024, pages 3053–3077, 2024.

[108] Bofei Zhang, Zirui Shang, Zhi Gao, Wang Zhang, Rui Xie, Xiaojian Ma, Tao Yuan, Xinxiao Wu, Song-Chun Zhu, and Qing Li. Tongui: Internet-scale trajectories from multimodal web tutorials for generalized gui agents. arXiv preprint arXiv:2504.12679, 2025.

[109] Sicheng Fan, Rui Wan, Yifei Leng, Gaoning Liang, Li Ling, Yanyi Shang, and Dehan Kong. Webchain: A large-scale human-annotated dataset of real-world web interaction traces. arXiv preprint arXiv:2603.05295, 2026.

[110] Qiushi Sun, Kanzhi Cheng, Zichen Ding, Chuanyang Jin, Yian Wang, Fangzhi Xu, Zhenyu Wu, Chengyou Jia, Liheng Chen, Zhoumianze Liu, et al. Os-genesis: Automating gui agent trajectory construction via reverse task synthesis. In Proceedings of the 63rd Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers), pages 5555–5579, 2025.

[111] Zhengxi Lu, Yuxiang Chai, Yaxuan Guo, Xi Yin, Liang Liu, Hao Wang, Han Xiao, Shuai Ren, Guanjing Xiong, and Hongsheng Li. Ui-r1: Enhancing efficient action prediction of gui agents by reinforcement learning. arXiv preprint arXiv:2503.21620, 2025.

[112] Run Luo, Lu Wang, Wanwei He, Longze Chen, Jiaming Li, and Xiaobo Xia. Gui-r1: A generalist r1-style vision-language action model for gui agents. arXiv preprint arXiv:2504.10458, 2025.

[113] Danyang Zhang, Situo Zhang, Ziyue Yang, Zichen Zhu, Zihan Zhao, Ruisheng Cao, Lu Chen, and Kai Yu. Progrm: Build better gui agents with progress rewards. arXiv preprint arXiv:2505.18121, 2025.

[114] Hyungjoo Chae, Sunghwan Kim, Junhee Cho, Seungone Kim, Seungjun Moon, Gyeom Hwangbo, Dongha Lim, Minjin Kim, Yeonjun Hwang, Minju Gwak, et al. Web-shepherd: Advancing prms for reinforcing web agents. arXiv preprint arXiv:2505.15277, 2025.

[115] Han Xiao, Guozhi Wang, Yuxiang Chai, Zimu Lu, Weifeng Lin, Hao He, Lue Fan, Liuyang Bian, Rui Hu, Liang Liu, et al. Ui-genie: A self-improving approach for iteratively boosting mllm-based mobile gui agents. arXiv preprint arXiv:2505.21496, 2025.

[116] Zecheng Li, Zhihui Cao, Wenke Huang, Yudong Zhang, Keying Qi, Rui Wang, Zeyu Zheng, Jian Zhao, Hao Zhu, Hengxin Wu, et al. Magicgui-rms: A multi-agent reward model system for self-evolving gui agents via automated feedback reflux. arXiv preprint arXiv:2601.13060, 2026.

[117] Linxin Song, Jieyu Zhang, Huanxin Sheng, Taiwei Shi, Gupta Rahul, Yang Liu, Ranjay Krishna, Jian Kang, and Jieyu Zhao. Video-based reward modeling for computer-use agents. arXiv preprint arXiv:2603.10178, 2026.

[118] Dongsen Zhang, Zekun Li, Xu Luo, Xuannan Liu, Peipei Li, and Wenjun Xu. Mcp security bench (msb): Benchmarking attacks against model context protocol in llm agents. arXiv preprint arXiv:2510.15994, 2025.

[119] Zichuan Li, Jian Cui, Xiaojing Liao, and Luyi Xing. Les dissonances: Cross-tool harvesting and polluting in pool-of-tools empowered llm agents. arXiv preprint arXiv:2504.03111, 2025.

[120] Henry Hengyuan Zhao, Kaiming Yang, Wendi Yu, Difei Gao, and Mike Zheng Shou. Worldgui: An interactive benchmark for desktop gui automation from any starting point. arXiv preprint arXiv:2502.08047, 2025.

[121] Hongbin Zhong, Fazle Faisal, Luis França, Tanakorn Leesataporn-wongsa, Adriana Szekeres, Kexin Rong, and Suman Nath. Actionengine: From reactive to programmatic gui agents via state machine memory. arXiv preprint arXiv:2602.20502, 2026.

[122] Zae Myung Kim, Dongseok Lee, Jaehyung Kim, Vipul Raheja, and Dongyeop Kang. The amazing agent race: Strong tool users, weak navigators. arXiv preprint arXiv:2604.10261, 2026.

[123] Shufan Li, Konstantinos Kallidromitis, Akash Gokul, Yusuke Kato, Kazuki Kozuka, and Aditya Grover. Mobileworldbench: Towards semantic world modeling for mobile agents. arXiv preprint arXiv:2512.14014, 2025.

[124] Xilong Wang, John Bloch, Zedian Shao, Yuepeng Hu, Shuyan Zhou, and Neil Zhenqiang Gong. Webinject: Prompt injection attack to web agents. In Proceedings of the 2025 Conference on Empirical Methods in Natural Language Processing, pages 2010–2030, 2025.

[125] Pei Yang, Hai Ci, and Mike Zheng Shou. In-context defense in computer agents: An empirical study. arXiv preprint arXiv:2503.09241, 2025.

[126] Yurun Chen, Xueyu Hu, Keting Yin, Juncheng Li, and Shengyu Zhang. Evaluating the robustness of multimodal agents against active environmental injection attacks. In Proceedings of the 33rd ACM International Conference on Multimedia, pages 11648–11656, 2025.

[127] Yuhang Wang, Feiming Xu, Zheng Lin, Guangyu He, Yuzhe Huang, Haichang Gao, Zhenxing Niu, Shiguo Lian, and Zhaoxiang Liu. From assistant to double agent: Formalizing and benchmarking attacks on openclaw for personalized local ai agent. arXiv preprint arXiv:2602.08412, 2026.

[128] Zijun Wang, Haoqin Tu, Letian Zhang, Hardy Chen, Juncheng Wu, Xiangyan Liu, Zhenlong Yuan, Tianyu Pang, Michael Qizhe Shieh, Fengze Liu, et al. Your agent, their asset: A real-world safety analysis of openclaw. arXiv preprint arXiv:2604.04759, 2026.

[129] Yushi Feng, Junye Du, Qifan Wang, Zizhan Ma, Qian Niu, Yutaka Matsuo, Long Feng, and Lequan Yu. Cora: Conformal risk-controlled agents for safeguarded mobile gui automation. arXiv preprint arXiv:2604.09155, 2026.

[130] Libo Sun, Jiwen Zhang, Siyuan Wang, and Zhongyu Wei. Magnet: Towards adaptive gui agents with memory-driven knowledge evolution. arXiv preprint arXiv:2601.19199, 2026.

[131] Mengzhou Wu, Yuzhe Guo, Yuan Cao, Haochuan Lu, Songhe Zhu, Pingzhe Qu, Xin Chen, Kang Qin, Zhongpu Wang, Xiaode Zhang, et al. Ui-oceanus: Scaling gui agents with synthetic environmental dynamics. arXiv preprint arXiv:2604.02345, 2026.

[132] Guangyi Liu, Pengxiang Zhao, Yaozhen Liang, Qinyi Luo, Shunye Tang, Yuxiang Chai, Weifeng Lin, Han Xiao, WenHao Wang, Siheng Chen, et al. Memgui-bench: Benchmarking memory of mobile gui agents in dynamic environments. arXiv preprint arXiv:2602.06075, 2026.

[133] Md Farhan Ishmam and Kenneth Marino. TimeWarp: Evaluating Web Agents by Revisiting the Past, mar 2026. arXiv preprint arXiv:2603.04949.

[134] Zehai He, Wenyi Hong, Zhen Yang, Ziyang Pan, Mingdao Liu, Xiaotao Gu, and Jie Tang. Vision2web: A hierarchical benchmark for visual website development with agent verification. arXiv preprint arXiv:2603.26648, 2026.

[135] Fanheng Kong, Jingyuan Zhang, Yang Yue, Chenxi Sun, Yang Tian, Shi Feng, Xiaocui Yang, Daling Wang, Yu Tian, Jun Du, et al. Webtestbench: Evaluating computer-use agents towards end-to-end automated web testing. arXiv preprint arXiv:2603.25226, 2026.

[136] Yifei Gao, Jiang Wu, Xiaoyi Chen, Yifan Yang, Zhe Cui, Tianyi Ma, Jiaming Zhang, and Jitao Sang. Guitester: Enabling gui agents for exploratory defect discovery. arXiv preprint arXiv:2601.04500, 2026.

[137] Wee Joe Tan, Zi Rui Lucas Lim, Shashank Durgad, Karim Obegi, and Aiden Yiliu Li. Opeflo: Automated ux evaluation via simulated human web interaction with gui grounding. arXiv preprint arXiv:2604.09581, 2026.

[138] Songyang Liu, Chaozhuo Li, Chenxu Wang, Jinyu Hou, Zejian Chen, Litian Zhang, Zheng Liu, Qiwei Ye, Yiming Hei, Xi Zhang, et al. Clawkeeper: Comprehensive safety protection for openclaw agents through skills, plugins, and watchers. arXiv preprint arXiv:2603.24414, 2026.

[139] Tri Cao, Bennett Lim, Yue Liu, Yuan Sui, Yuexin Li, Shumin Deng, Lin Lu, Nay Oo, Shuicheng Yan, and Bryan Hooi. Vpi-bench: Visual prompt injection attacks for computer-use agents. arXiv preprint arXiv:2506.02456, 2025.

[140] Yutao Luo, Haotian Zhu, Shuchao Pang, Zhigang Lu, Tian Dong, Yongbin Zhou, and Minhui Xue. Agentrae: Remote action execution through notification-based visual backdoors against screenshots-based mobile gui agents. arXiv preprint arXiv:2603.23007, 2026.

[141] Yurun Chen, Zeyi Liao, Ping Yin, Taotao Xie, Keting Yin, and Shengyu Zhang. Safepred: A predictive guardrail for computer-using agents via world models. arXiv preprint arXiv:2602.01725, 2026.

[142] Samuele Marro, Alan Chan, Xinxing Ren, Lewis Hammond, Jesse Wright, Gurjyot Wanga, Tiziano Piccardi, Nuno Campos, Tobin South, Jialin Yu, et al. Permission manifests for web agents. arXiv preprint arXiv:2601.02371, 2025.

[143] Phil Cuvin, Hao Zhu, and Diyi Yang. Decepticon: How dark patterns manipulate web agents. arXiv preprint arXiv:2512.22894, 2025.

[144] Junxian Li, Tu Lan, Haozhen Tan, Yan Meng, and Haojin Zhu. Slowba: An efficiency backdoor attack towards vlm-based gui agents. arXiv preprint arXiv:2603.08316, 2026.

[145] Haitao Hu, Peng Chen, Yanpeng Zhao, and Yuqi Chen. Agentsentinel: An end-to-end and real-time security defense framework for computer-use agents. In Proceedings of the 2025 ACM SIGSAC Conference on Computer and Communications Security, pages 3535–3549, 2025.

[146] Lukas Aichberger, Alasdair Paren, Guohao Li, Philip Torr, Yarin Gal, and Adel Bibi. Mip against agent: Malicious image patches hijacking multimodal os agents. arXiv preprint arXiv:2503.10809, 2025.

[147] Xilong Wang, Yinuo Liu, Zhun Wang, Dawn Song, and Neil Gong. Websentinel: Detecting and localizing prompt injection attacks for web agents. arXiv preprint arXiv:2602.03792, 2026.

[148] Juyong Lee, Dongyoon Hahm, June Suk Choi, W. Bradley Knox, and Kimin Lee. MobileSafetyBench: Evaluating Safety of Autonomous Agents in Mobile Device Control. In Fortieth AAAI Conference on Artificial Intelligence, pages 37565–37573. AAAI Press, 2026.

[149] Ido Levy, Ben Wiesel, Sami Marreed, Alon Oved, Avi Yaeli, Nir Mashkif, and Segev Shlomov. ST-WebAgentBench: A Benchmark for Evaluating Safety and Trustworthiness in Web Agents. In International Conference on Learning Representations, 2026. Poster.

[150] Xuwei Ding, Skylar Zhai, Linxin Song, Jiate Li, Taiwei Shi, Nicholas Meade, Siva Reddy, Jian Kang, and Jieyu Zhao. The blind spot of agent safety: How benign user instructions expose critical vulnerabilities in computer-use agents. arXiv preprint arXiv:2604.10577, 2026.

[151] Jianshuo Dong, Sheng Guo, Hao Wang, Xun Chen, Zhuotao Liu, Tianwei Zhang, Ke Xu, Minlie Huang, and Han Qiu. Safesearch: Automated red-teaming for the safety of llm-based search agents. arXiv preprint arXiv:2509.23694, 2025.

[152] Daniel Jones, Giorgio Severi, Martin Pouliot, Gary Lopez, Joris de Gruyter, Santiago Zanella-Beguelin, Justin Song, Blake Bullwinkel, Pamela Cortez, and Amanda Minnich. A systematization of security vulnerabilities in computer use agents. arXiv preprint arXiv:2507.05445, 2025.

[153] Hanrong Zhang, Jingyuan Huang, Kai Mei, Yifei Yao, Zhenting Wang, Chenlu Zhan, Hongwei Wang, and Yongfeng Zhang. Agent security bench (asb): Formalizing and benchmarking attacks and defenses in llm-based agents. arXiv preprint arXiv:2410.02644, 2024.

[154] Yu Tian, Xiao Yang, Jingyuan Zhang, Yinpeng Dong, and Hang Su. Evil geniuses: Delving into the safety of llm-based agents. arXiv preprint arXiv:2311.11855, 2023.

[155] Pengzhou Cheng, Haowen Hu, Zheng Wu, Zongru Wu, Tianjie Ju, Zhuosheng Zhang, and Gongshen Liu. Hidden ghost hand: Unveiling backdoor vulnerabilities in mllm-powered mobile gui agents. arXiv preprint arXiv:2505.14418, 2025.

[156] Xueyang Zhou, Guiyao Tie, Guowen Zhang, Hechang Wang, Pan Zhou, and Lichao Sun. Badvla: Towards backdoor attacks on vision-language-action models via objective-decoupled optimization. arXiv preprint arXiv:2505.16640, 2025.

[157] Aaron Xuxiang Tian, Ruofan Zhang, Janet Tang, Ji Wang, Tianyu Shi, and Jiaxin Wen. Measuring harmfulness of computer-using agents. arXiv preprint arXiv:2508.00935, 2025.

[158] Haochen Gong, Chenxiao Li, Rui Chang, and Wenbo Shen. Secure and efficient access control for computer-use agents via context space. arXiv preprint arXiv:2509.22256, 2025.

[159] Luoxi Meng, Henry Feng, Ilia Shumailov, and Earlence Fernandes. cellmate: Sandboxing browser ai agents. arXiv preprint arXiv:2512.12594, 2025.

[160] Zeyi Liao, Jaylen Jones, Linxi Jiang, Yuting Ning, Eric Fosler-Lussier, Yu Su, Zhiqiang Lin, and Huan Sun. Redteamcua: Realistic adversarial testing of computer-use agents in hybrid web-os environments. arXiv preprint arXiv:2505.21936, 2025.

[161] Xiaoxue Ren, Penghao Jiang, Kaixin Li, Zhiyong Huang, Xiaoning Du, Jiaojiao Jiang, Zhenchang Xing, Jiamou Sun, and Terry Yue Zhuo. Hackworld: Evaluating computer-use agents on exploiting web application vulnerabilities. arXiv preprint arXiv:2510.12200, 2025.

[162] Qiusi Zhan, Zhixiang Liang, Zifan Ying, and Daniel Kang. Injecagent: Benchmarking indirect prompt injections in tool-integrated large language model agents. In Findings of the Association for Computational Linguistics: ACL 2024, pages 10471–10506, 2024.

[163] Chiyu Chen, Xinhao Song, Yunkai Chai, Yang Yao, Haodong Zhao, Lijun Li, Jie Li, Yan Teng, Gongshen Liu, and Yingchun Wang. Ghostei-bench: Do mobile agents resilience to environmental injection in dynamic on-device environments? arXiv preprint arXiv:2510.20333, 2025.

[164] Yitong Zhang, Ximo Li, Liyi Cai, and Jia Li. Environmental injection attacks against gui agents in realistic dynamic environments. arXiv preprint arXiv:2509.11250, 2026.

[165] Ivan Evtimov, Arman Zharmagambetov, Aaron Grattafiori, Chuan Guo, and Kamalika Chaudhuri. Wasp: Benchmarking web agent security against prompt injection attacks. arXiv preprint arXiv:2504.18575, 2025.

[166] Zeyi Liao, Lingbo Mo, Chejian Xu, Mintong Kang, Jiawei Zhang, Chaowei Xiao, Yuan Tian, Bo Li, and Huan Sun. EIA: Environmental Injection Attack on Generalist Web Agents for Privacy Leakage. In International Conference on Learning Representations, 2025. Poster.

[167] Wenkui Yang, Chao Jin, Haisu Zhu, Weilin Luo, Derek Yuen, Kun Shao, Huaibo Huang, Junxian Duan, Jie Cao, and Ran He. Are gui agents focused enough? automated distraction via semantic-level ui element injection. arXiv preprint arXiv:2604.07831, 2026.

[168] Karolina Korgul, Yushi Yang, Arkadiusz Drohomirecki, Will Howard, Lukas Aichberger, Chris Russell, Philip HS Torr, Adam Mahdi, Adel Bibi, et al. It's a trap! task-redirecting agent persuasion benchmark for web agents. arXiv preprint arXiv:2512.23128, 2025.

[169] Devin Ersoy, Brandon Lee, Ananth Shreekumar, Arjun Arunasalam, Muhammad Ibrahim, Antonio Bianchi, and Z Berkay Celik. Investigating the impact of dark patterns on llm-based web agents. arXiv preprint arXiv:2510.18113, 2025.

[170] Haoyu Liu, Dingcheng Li, Lukas Rutishauser, and Zeyu Zheng. Dual-modality multi-stage adversarial safety training: Robustifying multimodal web agents against cross-modal attacks. arXiv preprint arXiv:2603.04364, 2026.

[171] Dongrui Liu, Qihan Ren, Chen Qian, Shuai Shao, Yuejin Xie, Yu Li, Zhonghao Yang, Haoyu Luo, Peng Wang, Qingyu Liu, et al. Agentdog: A diagnostic guardrail framework for ai agent safety and security. arXiv preprint arXiv:2601.18491, 2026.

[172] Zhaorun Chen, Mintong Kang, and Bo Li. Shieldagent: Shielding agents via verifiable safety policy reasoning. arXiv preprint arXiv:2503.22738, 2025.

[173] Zhaorun Chen, Zhen Xiang, Chaowei Xiao, Dawn Song, and Bo Li. Agentpoison: Red-teaming llm agents via poisoning memory or knowledge bases. Advances in Neural Information Processing Systems, 37:130185–130213, 2024.

[174] Dominik Seip and Matthias Hein. Preference redirection via attention concentration: An attack on computer use agents. arXiv preprint arXiv:2604.08005, 2026.

[175] Jaylen Jones, Zhehao Zhang, Yuting Ning, Eric Fosler-Lussier, Pierre-Luc St-Charles, Yoshua Bengio, Dawn Song, Yu Su, and Huan Sun. When benign inputs lead to severe harms: Eliciting unsafe unintended behaviors of computer-use agents. arXiv preprint arXiv:2602.08235, 2026.

[176] Yi Qian, Kunwei Qian, Xingbang He, Ligeng Chen, Jikang Zhang, Tiantai Zhang, Haiyang Wei, Linzhang Wang, Hao Wu, and Bing Mao. Zero-permission manipulation: Can we trust large multimodal model powered gui agents? arXiv preprint arXiv:2601.12349, 2026.

[177] Guohong Liu, Jialei Ye, Jiacheng Liu, Yuanchun Li, Wei Liu, Pengzhi Gao, Jian Luan, and Yunxin Liu. Hijacking jarvis: Benchmarking mobile gui agents against unprivileged third parties. In Proceedings of the 2nd International Workshop on Edge and Mobile Foundation Models, pages 12–18, 2025.

[178] Donghyun Lee and Mo Tiwari. Prompt infection: Llm-to-llm prompt injection within multi-agent systems. arXiv preprint arXiv:2410.07283, 2024.

[179] Pierre Peigné, Mikolaj Kniejski, Filip Sondej, Matthieu David, Jason Hoelscher-Obermaier, Christian Schroeder de Witt, and Esben Kran. Multi-agent security tax: Trading off security and collaboration capabilities in multi-agent systems. In Proceedings of the AAAI Conference on Artificial Intelligence, volume 39, pages 27573–27581, 2025.

[180] Arman Zharmagambetov, Chuan Guo, Ivan Evtimov, Maya Pavlova, Ruslan Salakhutdinov, and Kamalika Chaudhuri. Agentdam: Privacy leakage evaluation for autonomous web agents. arXiv preprint arXiv:2503.09780, 2025.

[181] Shouju Wang, Fenglin Yu, Xirui Liu, Xiaoting Qin, Jue Zhang, Qingwei Lin, Dongmei Zhang, and Saravan Rajmohan. Privacy in action: Towards realistic privacy mitigation and evaluation for llm-powered agents. arXiv preprint arXiv:2509.17488, 2025.

[182] Gucongcong Fan, Chaoyue Niu, Chengfei Lyu, Fan Wu, and Guihai Chen. Core: Reducing ui exposure in mobile agents via collaboration between cloud and local llms. arXiv preprint arXiv:2510.15455, 2025.

[183] Yanxi Wang, Zhiling Zhang, Wenbo Zhou, Weiming Zhang, Jie Zhang, Qiannan Zhu, Yu Shi, Shuxin Zheng, and Jiyan He. GUIGuard: Toward a General Framework for Privacy-Preserving GUI Agents, jan 2026. arXiv preprint arXiv:2601.18842.

[184] Lesly Miculicich, Mihir Parmar, Hamid Palangi, Krishnamurthy Dj Dvijotham, Mirko Montanari, Tomas Pfister, and Long T Le. Veriguard: Enhancing llm agent safety via verified code generation. arXiv preprint arXiv:2510.05156, 2025.

[185] Mintong Kang, Chong Xiang, Sanjay Kariyappa, Chaowei Xiao, Bo Li, and Edward Suh. Mitigating indirect prompt injection via instruction-following intent analysis. arXiv preprint arXiv:2512.00966, 2025.

[186] Yuting Ning, Jaylen Jones, Zhehao Zhang, Chentao Ye, Weitong Ruan, Junyi Li, Rahul Gupta, and Huan Sun. When actions go off-task: Detecting and correcting misaligned actions in computer-use agents. arXiv preprint arXiv:2602.08995, 2026.

[187] Wenqi Zhang, Yulin Shen, Changyue Jiang, Jiarun Dai, Geng Hong, and Xudong Pan. Mirrorguard: Toward secure computer-use agents via simulation-to-real reasoning correction. arXiv preprint arXiv:2601.12822, 2026.

[188] Jungjae Lee, Dongjae Lee, Chihun Choi, Youngmin Im, Jaeyoung Wi, Kihong Heo, Sangeun Oh, Sunjae Lee, and Insik Shin. VeriSafe Agent: Safeguarding Mobile GUI Agent via Logic-based Action Verification. In Proceedings of the 31st Annual International Conference on Mobile Computing and Networking, pages 817–831. ACM, 2025.

[189] Zheng Wu, Pengzhou Cheng, Zongru Wu, Lingzhong Dong, and Zhuosheng Zhang. Gem: Gaussian embedding modeling for out-of-distribution detection in gui agents. In Proceedings of the AAAI Conference on Artificial Intelligence, volume 40, pages 33989–33997, 2026.

[190] Kaiyuan Zhang, Mark Tenenholtz, Kyle Polley, Jerry Ma, Denis Yarats, and Ninghui Li. Browsesafe: Understanding and preventing prompt injection within ai browser agents. arXiv preprint arXiv:2511.20597, 2025.

[191] Udari Madhushani Sehwag, Shayan Shabihi, Alex McAvoy, Vikash Sehwag, Yuancheng Xu, Dalton Towers, and Furong Huang. Propensitybench: Evaluating latent safety risks in large language models via an agentic approach. arXiv preprint arXiv:2511.20703, 2025.
