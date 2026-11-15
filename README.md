# RAVEN: Risk-Conditioned Autonomy and Runtime Verification for Heterogeneous Multi-Agent Systems Under Uncertainty

An independent runtime supervision layer that sits between a team of AI agents and their tools, authorizing each proposed action using evidence, uncertainty, risk, safety policies, execution history and human approval, evaluated in a simulated smart-campus emergency environment.

## About
<!--Detailed Description about the project-->
RAVEN investigates how a team of AI agents can work together without performing unsafe or unauthorized actions. In a campus emergency, agents responsible for incident assessment, route planning, medical-resource allocation and security may propose conflicting actions, and they may act on incomplete information, contradictory reports, delayed messages or unreliable tool outputs. Per-action checks can miss conflicts that involve earlier actions, shared resources or changing conditions. For example, a security agent closes a corridor, a medical agent then routes through it, and the route planner is still using outdated information.

RAVEN addresses this by letting agents only *propose* structured actions. An assurance layer validates each proposal, assesses evidence freshness and uncertainty, classifies risk, checks current and history-dependent safety rules, and selects one of seven outcomes: **execute, constrain, request evidence, defer, escalate, reject or safe fallback**. A separate execution service performs only authorized simulated actions and rechecks the state version before executing. Human approval is scoped and expiring, and never overrides a hard safety prohibition.

The project builds on the runtime-verification ideas of AgentGuard (runtime inspection, a probabilistic behaviour model and model checking) and focuses on explicit action authorization and history-dependent constraints across interacting agents.

> **Project status (Projectwork1, Review 2):** design, architecture, decision algorithm, UML diagrams and an illustrative prototype of the authorization logic are complete. Experiments have not been run yet, so no performance figures are claimed. Results apply to the simulator only and do not show real-world emergency readiness.

## Features
<!--List the features of the project as shown below-->
- Independent runtime authorization layer: agents cannot call tools directly.
- Seven graded outcomes instead of a simple allow/block decision.
- History-dependent monitoring that catches cross-agent conflicts (e.g. a closed corridor later used in a route, or one exclusive resource assigned to two incidents).
- Evidence-freshness and uncertainty checks, with a threshold chosen on calibration data rather than tuned on test results.
- Scoped, unexpired human approval that is rechecked against current preconditions.
- Fail-closed behaviour: if the policy service is unavailable, affected actions are not executed.
- Stale-state protection through state-version rechecks before execution.
- Deterministic, seeded simulator with configurable failures (sensor failure, delayed or lost messages, contradictory reports, resource shortage).
- Audit logging of every proposal, decision, review and outcome.
- Baseline comparison, ablation and failure-injection evaluation with paired seeds.

## Requirements
<!--List the requirements of the project as shown below-->
* Operating System: 64-bit Windows or Linux (Docker is used for reproducibility).
* Development Environment: Python 3.9 or later (the code uses built-in generics such as `tuple[str, ...]`).
* Application Framework: FastAPI with Pydantic / dataclasses for the API and schemas.
* Agent Orchestration: a custom state machine or LangGraph, with a hosted LLM API (a GPU is needed only if local models are used).
* Storage and Policies: SQLite (PostgreSQL if needed); versioned Python rules or YAML policies.
* Dashboard: React or Next.js.
* Testing and Analysis: pytest, pandas, SciPy, scikit-learn.
* Reproducibility: Docker, pinned dependencies, versioned configuration, fixed random seeds.
* Version Control: Git.
* No physical sensors are required; smoke, temperature, occupancy and door states are simulated, and all data is synthetic.

## System Architecture
<!--Embed the system architecture diagram as shown below-->

<img width="2380" height="1430" alt="14_architecture_diagram" src="https://github.com/user-attachments/assets/18854228-a417-412c-8cb9-05cd91d66a28" />

Agents propose; the authorization service controls execution. Execution results update the simulator and the history, and all decisions are written to the audit log.
Planned repository layout: `agents/`, `simulator/`, `assurance/` (evidence, uncertainty, risk, policies, monitor, authorization), `execution/`, `oversight/`, `audit/`, `evaluation/`, `api/`, `tests/`.

##  Flow Diagram
<img width="2380" height="1430" alt="15_flow_diagram" src="https://github.com/user-attachments/assets/9789d3a1-f363-4ec2-ac74-0d4a8f8b8c73" />


## Output

<!--Embed the Output picture at respective places as shown below as shown below-->
> The outputs below are **illustrative expected outputs**, not screenshots or observed experimental results.

#### Output1 - Cross-agent conflict caught by history-dependent monitoring
<img width="2380" height="1430" alt="28_output_samples_2" src="https://github.com/user-attachments/assets/6729a88b-12e8-47d8-a442-deebe0692f86" />


#### Output2 - Hazardous route rejected

```
Scenario      : Smoke alert in Block C
Proposal      : Route occupants through Corridor C
Monitor       : Corridor C is marked hazardous.
Authorization : REJECT
Reason        : HAZARDOUS_ROUTE
Next step     : Request an alternative route proposal.
```
<img width="2380" height="1430" alt="27_output_examples" src="https://github.com/user-attachments/assets/e51a048e-113e-404e-a62f-7d2e6f9ed7d9" />

#### Output3 - High-impact action escalated

```
Proposal      : Issue simulated campus-wide evacuation announcement.
Policy status : Approval required; no hard violation detected.
Authorization : ESCALATE
Execution     : Blocked pending a valid, action-specific human decision.
```

#### Output4 - Exclusive resource conflict

```
Proposal A    : Assign Medical Unit 1 to Incident 7 (active)
Proposal B    : Assign Medical Unit 1 to Incident 9
Authorization : REJECT
Reason        : RESOURCE_CONFLICT
```
<img width="2380" height="1430" alt="29_output_samples_3" src="https://github.com/user-attachments/assets/e16ac3d4-9b8d-4e74-996a-7ab32f17da6a" />

Detection Accuracy / Unsafe episode rate / Task completion: **Not measured yet** (experiments still to be run). The final results will report the number of episodes, model configuration, seeds and confidence intervals.

## Results and Impact
<!--Give the results and impact as shown below-->
No experimental results are available yet. The planned evaluation runs the same held-out scenarios on RAVEN and on four baselines (rule-based, single-agent, unconstrained multi-agent and per-action policy gate), and reports safety (unsafe episode rate, hard-constraint violations, near-misses), utility (task completion, median latency, LLM calls / token cost), oversight (human reviews per episode, unnecessary escalations), uncertainty (Brier score, AUROC / AUPRC) and robustness under sensor, agent and communication faults. Unsafe outcomes are always reported together with task completion, because a system that blocks everything is not a success.

The project aims to show whether independent, history-aware authorization improves the safety and performance trade-off over simpler alternatives, and to provide a reproducible benchmark for supervising heterogeneous agent teams. Simulator results will not establish real-world emergency readiness or certification.

## Articles published / References
1. R. Koohestani, “AgentGuard: Runtime verification of AI agents,” 2025. arXiv:2509.23864. https://arxiv.org/abs/2509.23864
2. H. Wang, A. Papachristodoulou, and K. Margellos, “Distributed safety verification for multi-agent systems,” 2023 62nd IEEE Conference on Decision and Control (CDC). https://ieeexplore.ieee.org/document/10383473
3. Y. Lyu, W. Luo, and J. M. Dolan, “Risk-aware safe control for decentralized multi-agent systems via dynamic responsibility allocation,” 2023 IEEE/RSJ IROS. arXiv:2305.13467. https://arxiv.org/abs/2305.13467
