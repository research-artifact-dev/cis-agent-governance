<!-- loio4a9b908d1aad459cacf47e8414ae6667 -->

# Agent Quality Framework

This topic explains how to measure and maintain the quality of AI agents built in FDE engagements. It defines three evaluation dimensions, the primary release gate metric, build-time and runtime quality practices, and the implications for FDE delivery across the full project lifecycle.



<a name="loio4a9b908d1aad459cacf47e8414ae6667__section_why_agent_quality_is_different"/>

## Why Agent Quality Is Different

Traditional software quality asks: does the code produce the correct output for a given input? For AI agents the question is harder: does the agent reliably achieve the intended business outcome across the full range of real-world conditions — including inputs it was never explicitly tested against?

The difference matters for Industry AI because agents operate in a reason-act-observe loop. They decompose goals, select tools, interpret outputs, and decide next steps. Each of these decisions introduces uncertainty. A deterministic function either works or it doesn't; an agent can appear to work — completing each individual step correctly — while still failing to achieve the business goal, or achieving it via an unsafe path.

This means quality must be measured at multiple levels simultaneously: how the agent reasons, how it uses tools, and what it ultimately does to the business.


<a name="loio4a9b908d1aad459cacf47e8414ae6667__section_three_evaluation_dimensions"/>

## The Three Evaluation Dimensions

SAP's AI-Native North Star Architecture defines agent quality across three dimensions that must be evaluated together. Optimizing for any single dimension in isolation produces misleading results (Goodhart's Law: a measure that becomes a target ceases to be a good measure).

### Dimension 1 — Technical

Measures how the agent performs as a reasoning and tool-use system.

**Table: Technical Evaluation Metrics**


<table>
<tr>
<th valign="top">

Metric

</th>
<th valign="top">

What It Measures

</th>
<th valign="top">

Why It Can Mislead in Isolation

</th>
</tr>
<tr>
<td valign="top">

Goal completion rate

</td>
<td valign="top">

Did the agent reach the intended end state?

</td>
<td valign="top">

An agent can "complete" a goal incorrectly (wrong output, accepted by the evaluator).

</td>
</tr>
<tr>
<td valign="top">

Tool call efficiency

</td>
<td valign="top">

Did the agent reach the goal with the minimum necessary tool calls? Token cost (LLM spend per successful outcome) is the measurable financial sub-dimension of this metric — see the Token Efficiency topic.

</td>
<td valign="top">

Efficient agents that cut corners may skip validation steps; conversely, minimizing token count at the expense of accuracy inverts the metric.

</td>
</tr>
<tr>
<td valign="top">

Self-verification results

</td>
<td valign="top">

Did the agent's internal feedback loop confirm output correctness before committing?

</td>
<td valign="top">

Self-verification can be fooled if the same model both generates and checks output.

</td>
</tr>
<tr>
<td valign="top">

Hallucination rate

</td>
<td valign="top">

How often does the agent produce plausible-but-wrong factual claims?

</td>
<td valign="top">

Hard to measure without ground-truth reference data.

</td>
</tr>
<tr>
<td valign="top">

Retry/recovery rate

</td>
<td valign="top">

How often does the agent recover from a tool failure or partial result?

</td>
<td valign="top">

High recovery rate may mask systemic data quality issues.

</td>
</tr>
</table>

**Self-verification** deserves special attention. SAP's architecture builds self-verification into the agent loop as an internal feedback mechanism during execution — not as a post-hoc check. In practice this means agents should be designed to validate their own intermediate outputs before proceeding to the next tool call, particularly before any write-back to a system of record. This is not the same as a separate supervisor agent; it is the agent checking its own work inline.

### Dimension 2 — Process KPIs

Measures how the agent affects the business process it automates.

These metrics are specific to the use case and must be defined during the FDE Discovery phase as part of the TO-BE process flow. Examples:

**Table: Representative Process KPIs by Industry AI Domain**


<table>
<tr>
<th valign="top">

Industry AI Domain

</th>
<th valign="top">

Representative Process KPI

</th>
</tr>
<tr>
<td valign="top">

Autonomous Asset Management

</td>
<td valign="top">

Mean time to resolve (MTTR) for maintenance events; false positive rate on anomaly alerts

</td>
</tr>
<tr>
<td valign="top">

Autonomous Regulated Manufacturing

</td>
<td valign="top">

Batch release cycle time; rework rate; deviation detection latency

</td>
</tr>
<tr>
<td valign="top">

Autonomous Commodity Management

</td>
<td valign="top">

Hedge ratio accuracy; position reconciliation exceptions

</td>
</tr>
<tr>
<td valign="top">

Autonomous Revenue Growth Management

</td>
<td valign="top">

Pricing proposal acceptance rate; margin deviation per recommendation

</td>
</tr>
<tr>
<td valign="top">

Autonomous Adaptive Production

</td>
<td valign="top">

Schedule adherence rate; production order exception volume

</td>
</tr>
</table>

Process KPIs are the bridge between what the agent does and whether it is actually useful. They are also the primary evidence for value realization in the FDE value case.

### Dimension 3 — Business Outcomes

Measures the financial and operational results that the agent contributes to.

This is the most lagging indicator — results appear weeks or months after deployment — but it is the dimension that justifies the value-based contract (10–30% of yearly customer benefit). Examples: consumed ACV per agent workflow, cost reduction in manual processing, reduction in exceptions requiring human intervention, throughput improvement.

The relationship between Dimension 1 metrics and Dimension 3 outcomes is not linear. An agent with a 95% goal completion rate may still fail to generate business value if it completes goals that were poorly specified, or if it generates outputs that humans override most of the time. Tracking all three dimensions in parallel is the only way to detect this disconnect early.

### The Primary Release Gate: Success Rate

Across all three evaluation dimensions, **success rate** is the primary release gate for FDE sprint milestones. A test case's success rate is binary: 1 if every defined validation passes, 0 if any single validation fails. This binary definition is deliberate — it forces the team to enumerate the exact conditions an agent must satisfy rather than averaging over partial correctness.

All other metrics (tool call correctness, latency, token cost, human override rate) are diagnostic indicators. They explain *why* success rate may be below target, but they do not substitute for it. The SAP aeval framework (TR9 — mandatory offline and online evaluation for all production agents) uses success rate as its primary aggregation metric and leading indicator for release decisions.


<a name="loio4a9b908d1aad459cacf47e8414ae6667__section_evaluation_timeline"/>

## Evaluation Timeline — Full Sequence from Discovery to Production

The evaluation activities in this topic span the full FDE lifecycle. The table below shows where each quality mechanism fits in the engagement timeline.

**Table: Evaluation Timeline**


<table>
<tr>
<th valign="top">

Phase

</th>
<th valign="top">

Evaluation Activity

</th>
<th valign="top">

Reference

</th>
<th valign="top">

Gate or Milestone

</th>
</tr>
<tr>
<td valign="top">

Discovery

</td>
<td valign="top">

G1–G4 readiness gates (entity completeness, goal coverage, actor-process binding, integration readiness)

</td>
<td valign="top">

</td>
<td valign="top">

Pre-Development gate: all gates ≥ 0.8 before sprint commit

</td>
</tr>
<tr>
<td valign="top">

Discovery

</td>
<td valign="top">

RPT-1 approach-first assessment (tabular ML vs. LLM/agent)

</td>
<td valign="top">

Introduction chapter

</td>
<td valign="top">

Part of scope decision; not a scored gate

</td>
</tr>
<tr>
<td valign="top">

Development — Week 1

</td>
<td valign="top">

aeval offline in traceless mode (output correctness + rule compliance from day 1)

</td>
<td valign="top">

[Agent Evaluation](../Agent-Evaluation/agent-evaluation-76db9f7.md)

</td>
<td valign="top">

No gate; establishes baseline

</td>
</tr>
<tr>
<td valign="top">

Development — Weeks 1–5

</td>
<td valign="top">

6-category post-build scorecard (spec integrity, generation drift, Joule config, routing descriptions, MUST-rule compliance, coverage)

</td>
<td valign="top">

[Agent Evaluation](../Agent-Evaluation/agent-evaluation-76db9f7.md)

</td>
<td valign="top">

Iterative; category B (generation drift) is a blocking failure

</td>
</tr>
<tr>
<td valign="top">

Development — Week 6

</td>
<td valign="top">

aeval offline with full trace-enabled metrics; eval suite ≥ 3 iterations; success rate documented

</td>
<td valign="top">

[Agent Evaluation](../Agent-Evaluation/agent-evaluation-76db9f7.md)

</td>
<td valign="top">

Week 6 milestone gate: required for FDE Gate evidence package

</td>
</tr>
<tr>
<td valign="top">

Development — Week 9

</td>
<td valign="top">

Full aeval offline suite passes in CI; online eval active in EAC (Early Adopter Care); TR9 offline compliance satisfied

</td>
<td valign="top">

[Agent Evaluation](../Agent-Evaluation/agent-evaluation-76db9f7.md)

</td>
<td valign="top">

Week 9 milestone gate: required before EAC

</td>
</tr>
<tr>
<td valign="top">

EAC

</td>
<td valign="top">

Online eval active (aeval-online); anomaly detection baselines calibrated; supervisor thresholds set

</td>
<td valign="top">

[Agent Evaluation](../Agent-Evaluation/agent-evaluation-76db9f7.md)

</td>
<td valign="top">

EAC operating condition

</td>
</tr>
<tr>
<td valign="top">

EAC → Productization

</td>
<td valign="top">

Scale gate criteria met: Stage 3 independence + override rate + process KPI + quality stability + zero Critical incidents

</td>
<td valign="top">

Scale Gate

</td>
<td valign="top">

Productization gate

</td>
</tr>
<tr>
<td valign="top">

Production (ongoing)

</td>
<td valign="top">

aeval online eval: statistical trend monitoring via CronJob; Workflow B anomaly detection

</td>
<td valign="top">

[Agent Evaluation](../Agent-Evaluation/agent-evaluation-76db9f7.md)

</td>
<td valign="top">

Continuous; no single gate

</td>
</tr>
<tr>
<td valign="top">

Quarterly

</td>
<td valign="top">

Eval suite review; model suitability review; iteration pattern analysis

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md) / [CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md)

</td>
<td valign="top">

Periodic governance review

</td>
</tr>
</table>


<a name="loio4a9b908d1aad459cacf47e8414ae6667__section_build_time_quality"/>

## Build-Time Quality (Pre-Deployment)

Build-time quality covers the practices and tooling applied before an agent is released to a productive or even a test environment. It happens during the FDE Development phase.

### Pre-Build Readiness Gates (G1–G4)

Before committing to agent spec generation, the extracted knowledge from the discovery phase must satisfy four readiness gates. These gates run once, before sprint execution begins — they are distinct from the Quality Flywheel (see [Agent Evaluation](../Agent-Evaluation/agent-evaluation-76db9f7.md)), which governs quality progression throughout development. Scores are 0–1.

**Table: Pre-Build Readiness Gates**


<table>
<tr>
<th valign="top">

Gate

</th>
<th valign="top">

Checks

</th>
<th valign="top">

Threshold

</th>
</tr>
<tr>
<td valign="top">

G1 — Entity Completeness

</td>
<td valign="top">

Proportion of extracted entities resolved against SAP canonical catalog (EKX or offline S/4 catalog)

</td>
<td valign="top">

≥ 0.8

</td>
</tr>
<tr>
<td valign="top">

G2 — Goal Coverage

</td>
<td valign="top">

Every Capability has a parent Goal

</td>
<td valign="top">

≥ 0.8

</td>
</tr>
<tr>
<td valign="top">

G3 — Actor-Process Binding

</td>
<td valign="top">

Every Process has an assigned Actor

</td>
<td valign="top">

≥ 0.8

</td>
</tr>
<tr>
<td valign="top">

G4 — Integration Readiness

</td>
<td valign="top">

Every Integration has an API binding

</td>
<td valign="top">

≥ 0.8

</td>
</tr>
</table>

**Scoring interpretation:**

-   ≥ 0.8 — proceed to spec generation
-   0.5–0.8 — proceed with documented gaps; auto-generate "coaching questions" to bring back to the customer
-   < 0.5 — return to discovery; insufficient grounding for reliable agent design

When a gate scores 0.5–0.8, the framework generates targeted gap-filling queries — structured follow-up questions for the customer that address the specific missing elements. This turns discovery gaps into actionable items rather than sprint blockers. The coaching questions become an input to the next Discovery sub-phase before execution begins.

The G1–G4 readiness gates are implemented in Project LightSpeed's discovery pipeline (Stages 1–8) and represent the earliest measurable quality checkpoint in the FDE delivery lifecycle. They complement the "Done Enough" threshold guidance in the section below: the readiness gates confirm sufficient grounding before design begins; the "Done Enough" thresholds govern when a built agent is ready to proceed to the next sprint milestone.

### Eval Design in Joule Studio

Every agent built in Joule Studio has an associated evaluation configuration. The eval design should answer:

1.  **What is the agent supposed to achieve?** — Define the goal state as a machine-checkable condition, not a vague description. "Generate a purchase order recommendation" is vague; "generate a PO recommendation with a valid vendor, quantity within ±5% of historical baseline, and price within approved supplier price list" is checkable.

2.  **What are the boundary conditions?** — Define cases where the agent must NOT act (e.g., transaction value above approval threshold, vendor not in approved list, output confidence below threshold). These are the guardrail test cases.

3.  **What is the test data strategy?** — This is a legal question as much as a technical one. During the FDE Development phase, anonymized/synthetic data can be used under a TEA (Test & Evaluation Agreement). Real customer data requires a Collaboration Agreement. Build-time evals should be designed to work with synthetic data so they can run continuously without legal friction.

4.  **What is acceptable failure behavior?** — Define what the agent should do when it cannot complete the goal: escalate, do nothing, or produce a partial result with explicit uncertainty flagging. Undefined failure behavior is the single most common source of production incidents.

### SAP AI Agent Hub: Agent Evals and Assessment

The SAP AI Agent Hub provides a dedicated evaluation capability for agents across their full lifecycle: Plan & Build → Discover & Provision → Observe & Analyze → Secure & Govern → Optimize & Decommission.

During the build phase, the Agent Evals & Assessment capability enables structured evaluation runs against defined test suites. The output of these eval runs becomes the evidence package for the FDE Gate — the mandatory checkpoint before execution can proceed. The FDE Gate specifically requires a feasibility assessment by Product Engineering; eval results from the Agent Hub provide the technical grounding for that assessment.

The concrete evaluation mechanism — SAP's `aeval` framework, the test case DSL, the offline/online evaluation split, and how evaluation maps to sprint weeks — is covered in [Agent Evaluation in FDE Sprints](../Agent-Evaluation/agent-evaluation-76db9f7.md). TR9 (mandatory offline + online evaluation for all production agents) requires the framework described there.

### The "Done Enough" Threshold

A common FDE pitfall is treating build-time eval as a pass/fail gate with an arbitrary threshold (e.g., "95% goal completion"). This creates two failure modes:

-   **Under-threshold blocking**: The agent is held back because it fails edge cases that will never occur in the customer's actual process. This delays delivery.
-   **Over-threshold acceptance**: The agent passes the eval suite but the suite did not cover the cases that matter. This creates false confidence.

The correct approach is to define acceptance criteria per scenario type:

**Table: Acceptance Criteria by Scenario Type**


<table>
<tr>
<th valign="top">

Scenario Type

</th>
<th valign="top">

Acceptance Threshold Approach

</th>
</tr>
<tr>
<td valign="top">

High-frequency, low-risk (read-only)

</td>
<td valign="top">

Statistical threshold on a representative sample

</td>
</tr>
<tr>
<td valign="top">

Low-frequency, high-risk (write to system of record)

</td>
<td valign="top">

100% coverage of defined test cases; any failure blocks

</td>
</tr>
<tr>
<td valign="top">

Exception/escalation paths

</td>
<td valign="top">

Verify escalation fires; do NOT require resolution

</td>
</tr>
<tr>
<td valign="top">

Novel/out-of-distribution inputs

</td>
<td valign="top">

Verify graceful degradation; do not require correct output

</td>
</tr>
</table>

### Domain Primitive Vocabulary

For structured knowledge extraction from customer artifacts, CIS uses 12 canonical domain primitive types that map directly to agent design elements. These types form a shared vocabulary between the discovery process and the agent build phase: customer knowledge is first extracted into these primitives, then primitives are mapped to agent topology and quality controls.

**Table: Domain Primitive Types**


<table>
<tr>
<th valign="top">

Type

</th>
<th valign="top">

Maps To

</th>
</tr>
<tr>
<td valign="top">

Entity

</td>
<td valign="top">

Data object (SAP business object or custom)

</td>
</tr>
<tr>
<td valign="top">

Actor

</td>
<td valign="top">

Customer persona → one dedicated sub-agent

</td>
</tr>
<tr>
<td valign="top">

BusinessObject

</td>
<td valign="top">

SAP business object with API binding

</td>
</tr>
<tr>
<td valign="top">

System

</td>
<td valign="top">

Backend system (SAP or third-party)

</td>
</tr>
<tr>
<td valign="top">

Process

</td>
<td valign="top">

Business process step

</td>
</tr>
<tr>
<td valign="top">

Capability

</td>
<td valign="top">

Agent capability / tool

</td>
</tr>
<tr>
<td valign="top">

Rule

</td>
<td valign="top">

Business rule (source for MUST rules and HITL gates)

</td>
</tr>
<tr>
<td valign="top">

Relationship

</td>
<td valign="top">

Entity-to-entity link

</td>
</tr>
<tr>
<td valign="top">

Goal

</td>
<td valign="top">

Business goal (parent of Capabilities; checked by G2)

</td>
</tr>
<tr>
<td valign="top">

KPI

</td>
<td valign="top">

Measurable success metric (Dimension 2 — Process KPIs)

</td>
</tr>
<tr>
<td valign="top">

Risk

</td>
<td valign="top">

Risk factor (source for guardrail design)

</td>
</tr>
<tr>
<td valign="top">

Integration

</td>
<td valign="top">

System integration point (source for API binding; checked by G4)

</td>
</tr>
</table>

Three primitive types have direct implications for agent quality design: **Actors** drive topology (one Actor per sub-agent), **Rules** are the source material for MUST rules and human-in-the-loop gates, and **Risks** inform the guardrail stack. The G1–G4 readiness gates directly test the completeness of the Goal, Actor, and Integration primitives before spec generation begins.

### Evaluation Design Philosophy — TDD/BDD, Test Onion, and 11 Dimensions

Three principles from the SAP AI Golden Path evaluation framework govern how eval suites should be designed in FDE sprints.

### TDD/BDD: Tests Before Code

Evaluation test cases are a **Week 0 sprint deliverable**, not something created after the agent is built. Define BDD-style test cases — given / when / then scenarios that describe exactly what the agent must do — before writing any workflow code. These test cases are the agent's specification. Scenario runs use the Joule Studio CLI's `joule test` (which executes your BDD feature files) or the Joule Studio editor Test button; pre-build validation uses the Solution Management CLI (`jl solution validate`); and the broader eval suite runs in CI via aeval throughout development.

This is not optional: writing tests after the fact tests what the agent does, not what it should do. Retroactive test suites accumulate false coverage of edge cases the implementation happened to get right. Tests written first express business intent and become the ground truth for the Quality Flywheel (see [Agent Evaluation](../Agent-Evaluation/agent-evaluation-76db9f7.md)).

### Test Onion — Layered Testing Strategy

Agent quality testing is organized in three concentric layers:

**Table: Test Onion Layers**


<table>
<tr>
<th valign="top">

Layer

</th>
<th valign="top">

What It Tests

</th>
<th valign="top">

Investment Priority

</th>
</tr>
<tr>
<td valign="top">

Inner — Tool-level unit tests

</td>
<td valign="top">

Each individual tool/API call returns correct data; connectors are wired correctly

</td>
<td valign="top">

Low to medium — validates data plumbing

</td>
</tr>
<tr>
<td valign="top">

Middle — Agent evaluation (aeval)

</td>
<td valign="top">

The agent's reasoning, tool selection, and decision-making across defined scenarios

</td>
<td valign="top">

Highest — mandatory for production (TR9); this is the core layer

</td>
</tr>
<tr>
<td valign="top">

Outer — Production integration tests

</td>
<td valign="top">

End-to-end behavior in the real customer environment; correct output to the system of record

</td>
<td valign="top">

Medium — validates final wiring

</td>
</tr>
</table>

The aeval layer is the most important investment. Tool-level tests confirm that connectors work; integration tests confirm deployment. Only the eval layer validates whether the agent is actually making correct decisions.

### 11 Evaluation Dimensions

The SAP AI Golden Path defines a set of evaluation dimensions for comprehensive agent evaluation. **The canonical set — 11 dimensions with their priorities — is maintained in [Agent Evaluation](../Agent-Evaluation/agent-evaluation-76db9f7.md) (Evaluation Dimensions Checklist); the table below matches it row-for-row.** Cover at minimum the High-priority dimensions before the Week 6 gate and all Medium-priority dimensions before production go-live.

**Table: 11 Evaluation Dimensions**


<table>
<tr>
<th valign="top">

Dimension

</th>
<th valign="top">

Priority

</th>
<th valign="top">

What It Tests

</th>
</tr>
<tr>
<td valign="top">

Correctness

</td>
<td valign="top">

HIGH

</td>
<td valign="top">

Does the agent produce the right answer for defined scenarios?

</td>
</tr>
<tr>
<td valign="top">

Hallucinations

</td>
<td valign="top">

HIGH

</td>
<td valign="top">

Does the agent avoid inventing facts not present in the grounding context?

</td>
</tr>
<tr>
<td valign="top">

Safety guardrails

</td>
<td valign="top">

HIGH

</td>
<td valign="top">

Do precision guardrails fire correctly on boundary inputs; does Layer 3 escalation trigger?

</td>
</tr>
<tr>
<td valign="top">

Stability

</td>
<td valign="top">

HIGH

</td>
<td valign="top">

Does the agent behave consistently across repeated identical inputs (≥ 3 iterations per case)?

</td>
</tr>
<tr>
<td valign="top">

Summarization

</td>
<td valign="top">

HIGH

</td>
<td valign="top">

Does the agent correctly synthesize and summarize retrieved information?

</td>
</tr>
<tr>
<td valign="top">

Security

</td>
<td valign="top">

HIGH

</td>
<td valign="top">

Does the agent resist prompt injection, privilege escalation, and harmful instructions?

</td>
</tr>
<tr>
<td valign="top">

Memory

</td>
<td valign="top">

MEDIUM

</td>
<td valign="top">

Does the agent maintain relevant context across conversation turns?

</td>
</tr>
<tr>
<td valign="top">

RAG retrieval

</td>
<td valign="top">

MEDIUM

</td>
<td valign="top">

Does RAG grounding surface the correct context for the agent's task?

</td>
</tr>
<tr>
<td valign="top">

Performance

</td>
<td valign="top">

MEDIUM

</td>
<td valign="top">

Does the agent meet latency and throughput targets under load?

</td>
</tr>
<tr>
<td valign="top">

UI/UX output

</td>
<td valign="top">

MEDIUM

</td>
<td valign="top">

Does the agent's output render correctly and usably in the Joule surface?

</td>
</tr>
<tr>
<td valign="top">

Digression / scope

</td>
<td valign="top">

LOW

</td>
<td valign="top">

Does the agent stay within its defined scope and reject out-of-scope instructions?

</td>
</tr>
</table>

**Security/Safety is HIGH priority — not optional.** For FDE-delivered Industry AI agents handling financial commitments, production data, or regulated outputs, a security/safety failure is a production incident regardless of correctness performance. Eval suites that omit prompt injection resistance, privilege constraint verification, and HITL gate behavior are incomplete. These dimensions must be included from week 1.

**Judge verdict model — three-way C/I/N**: The aeval LLM-as-judge returns one of three verdicts per check: **Complete**, **Incomplete**, or **Not Applicable**. Not Applicable (N) means the check criterion was not triggered in this session — it is the *absence* of a data point, not a failure. From the aeval OTel schema (November 2026): `NOT_APPLICABLE` scores are **not emitted** to OpenTelemetry output at all; absence in the trace is the signal. This matters for dashboards and aggregations — a check that never fired appears as no data, not as `0`. Build dashboards that distinguish "no score emitted" from "score = 0" to avoid silently skewing aggregate success rates. For online eval specifically (`validate_online`), the 3-way C/I/N judge is now the standard template.

For the full aeval framework, offline CI evaluation (GitHub Actions, deterministic + LLM-as-judge), online DwC pipeline evaluation (sampled production traffic), and TR9 compliance requirements, see [Agent Evaluation in FDE Sprints](../Agent-Evaluation/agent-evaluation-76db9f7.md).


<a name="loio4a9b908d1aad459cacf47e8414ae6667__section_runtime_quality"/>

## Runtime Quality (Post-Deployment)

Build-time quality can only cover what was anticipated. Runtime quality monitoring detects what was not.

### BAIP Managed Agent Runtime Harness

When an agent is created in SAP's BAIP managed agent runtime, the platform automatically enables a governance harness that captures:

-   **Traces**: full execution trace per agent invocation, including each tool call, input/output, latency, and branch taken
-   **Logs**: structured log entries at key decision points
-   **Metrics**: aggregated counts and distributions (call volume, error rate, latency percentiles, tool call counts per invocation)

This telemetry is captured via OpenTelemetry. Business and personal data and the LLM inference stay within the customer's BAIP tenant (personal data masked via DPI). The operational telemetry itself (traces, metrics, error codes) is pushed to **SAP Cloud ALM** — an SAP-operated service, region-bound to the customer's contracted data-residency region. So the accurate data-protection statement is not that telemetry never leaves the tenant, but that it is stored within the customer's SAP-operated data-residency boundary; business data and reasoning stay local by default.

The harness is not optional — it is automatically applied to all agents running on the BAIP managed runtime. This means quality observability is built in, not added on.

### SAP AI Agent Hub: Agent Observability

The Agent Observability capability in SAP AI Agent Hub surfaces the BAIP harness telemetry in a governed, role-based view. Relevant runtime signals:

-   **Anomaly detection**: deviations from established baseline behavior (unusual tool call sequences, response time spikes, unexpected output distributions)
-   **Drift detection**: gradual degradation of output quality over time, detectable via process KPI tracking even when individual tool calls appear healthy
-   **Error rate tracking**: tool failures, LLM refusals, system unavailability events
-   **Human override rate**: how often downstream humans reject or modify agent outputs — a direct signal of output quality that technical metrics alone cannot capture

The human override rate is the most underused runtime quality signal. If agents produce recommendations that humans override 40% of the time, the agent is generating work rather than saving it. This signal requires connecting agent output tracking to the process step that follows the agent — typically a task in SAP Task Center.

### Self-Verification in the Runtime Loop

Self-verification is an architectural pattern, not a specific product feature. In the Joule Studio n8n canvas, it is implemented by adding a validation step before the `sapMcpClient` write-back node:

```
[sapOrchestration: generate output]
        ↓
[sapOrchestration: validate output against rules]
        ↓
[If: validation passed?]
    ├── YES → [sapMcpClient: write to system of record]
    └── NO  → [sapTaskCenter: escalate to human] or [retry / abort]
```

The validation step uses a second `sapOrchestration` call with a different system prompt explicitly designed for critique rather than generation. This is a lightweight version of a supervisor pattern operating within a single agent's workflow rather than as a separate agent process. It catches the most common failure mode — confident but wrong output — without requiring inter-agent communication overhead.

**Limitation**: using the same underlying model for both generation and validation reduces the independence of the check. For high-stakes outputs, the validation step should use a different model (e.g., a smaller, deterministic model for rule-checking, or a rule-based code node for numerical validation).

### Multi-Stage Verification: Combining Deterministic and LLM-Based Checks

The self-verification pattern above uses a second LLM call for validation. This is often the only option — but not always. For many Industry AI scenarios, a deterministic check of the agent's output is also possible, because a simplified rule-based or algorithmic computation of the "correct" answer exists independently of the AI reasoning.

The deterministic check is implemented as:

-   A `Code` node (JS/Python, sandboxed) for rules expressible in code — e.g., "is the recommended PO quantity within ±X% of the historical baseline for this material?"
-   A `sapMcpClient` call to an existing SAP calculation service — e.g., batch classification rules in S/4HANA, a pricing engine, or a valuation model in SAP Analytics Cloud
-   An `HTTP Request` to a custom BAIP function for domain-specific computations

The key advantage: **deterministic algorithms produce binary, explainable verdicts with no hallucination risk**. When a deterministic check is available, the two approaches can be combined into a multi-stage verification pattern:

```
[sapOrchestration: generate output]
        ↓
[Stage 1 — Deterministic: Code node / sapMcpClient calculation]
[Stage 2 — LLM: second sapOrchestration call, critic system prompt]
        ↓
[If: both stages agree?]
    ├── BOTH CONFIRM → high-confidence write-back to system of record
    ├── BOTH REJECT  → escalate to human (agent was clearly wrong)
    └── DISAGREE     → escalate to human (uncertain, requires judgment)
```

Two independent control mechanisms agreeing constitutes a strong verification signal — stronger than either check alone. Disagreement between the two is itself informative: it surfaces cases where the deterministic rule does not capture the full complexity of the situation, which is exactly the case a human should review.

A further benefit: every confirmed or rejected output becomes a **labeled data point**. Over time, the collected (output, deterministic verdict, LLM verdict, human decision) records constitute a ground-truth dataset that can be used to improve build-time evals and to measure hallucination rate — the metric that the Technical dimension identified as hard to quantify without reference data.

When a deterministic check does not exist (the reason the AI agent was built in the first place is that no computable answer is available), the LLM-as-verifier remains the appropriate fallback. The choice of which verification stages to apply is a design decision made during FDE Discovery, informed by what calculation capabilities already exist in the customer's SAP landscape.

### Per-Step Iteration Count as an Optimization Signal

Beyond output quality, the number of internal iterations an agent takes on a single step is a runtime signal with optimization value. Just as a chatbot user who must rephrase the same request multiple times signals a context or capability gap, an agent that retries a tool call, cycles through multiple self-correction loops, or consumes disproportionate tokens on one step signals a design mismatch at that point in the workflow.

The BAIP harness captures the raw data via OpenTelemetry traces — tool call counts per invocation, latency per step, and self-correction loop depth. The optimization value lies not in individual spikes (those are incidents, handled by the supervisor) but in **patterns across invocations**: a step that consistently requires more iterations than others is a candidate for redesign, not just monitoring.

This signal connects directly to the tool-selection principle in the Precision Guardrails topic: a step with chronically high retry counts is often one where the LLM is computing something that a deterministic SAP service could resolve in a single call.

### Agent Behavior Mining — Forward-Looking Capability

A complementary approach to OpenTelemetry-based observability is **agent behavior mining** via SAP Signavio Process Intelligence: analyzing production agent execution traces at the process level to surface behavioral patterns, flow variants, conformance deviations, telemetry-to-KPI conformance mapping, process KPI impact, and value vs. decommission assessments — each execution pattern compared against the original business case to determine whether the agent continues to generate measurable value.

**Important availability caveat**: agent behavior mining in Signavio is designed for the next generation of SAP agents built on **Joule Studio 2.0**. GA is phased: the Joule Studio 2.0 build environment is itself in phased GA, but the Signavio agent-mining capability built on these next-generation agents is **not yet generally available**. SAP Architecture Center (RA0029/9) is explicit: *"This page describes the planned agent mining model for the next generation of SAP AI agents built on Joule Studio 2.0 which is yet to be made generally available (GA). It does not apply to the current agent implementations."*

For current FDE-built agents — including BYOA custom agents and Joule Studio canvas workflows — **OpenTelemetry-based observability via the BAIP harness and the supervisor telemetry patterns are the production options**. Signavio agent mining is a capability to plan for in the productization roadmap, not a current deliverable for FDE engagements.

When Joule Studio 2.0 agents become GA: mining is enabled at the agent type level per customer tenant — configured in the SAP AI Agent Hub and trackable via LeanIX (SAP's enterprise architecture management tool) for broader application inventory governance — not per running instance. Confirm Signavio Process Intelligence availability during FDE Discovery before building any customer operational dependency on it.


<a name="loio4a9b908d1aad459cacf47e8414ae6667__section_goodharts_law"/>

## The Goodhart's Law Problem

Goodhart's Law applied to business agents: once goal completion rate becomes the primary metric an agent is optimized for, agents learn to complete goals in ways that satisfy the metric without achieving the business intent.

Concrete examples in Industry AI:

-   An asset management agent maximizes "maintenance task completion rate" by recommending fewer complex maintenance tasks and more simple ones, inflating the metric while deferring actual risk.
-   A procurement agent maximizes "PO generation speed" by reusing existing vendor quotes rather than checking for better alternatives, optimizing the metric while increasing spend.
-   A batch release agent maximizes "release approval rate" by flagging fewer anomalies, reducing false positives at the cost of missed genuine deviations.

The safeguard is multi-objective evaluation: optimize across all three dimensions simultaneously, with explicit trade-off thresholds defined for each use case. In practice this means:

1.  Define at least one metric per dimension before the agent is deployed.
2.  Alert when any dimension degrades beyond threshold, even if others improve.
3.  Review aggregate performance monthly; review after any model or data change.
4.  Define a reversion trigger: if Dimension 3 (business outcomes) degrades despite Dimension 1 (technical) appearing healthy, investigate the gap before the next model update.

**A note on this concept's own metrics**: the human override rate, verification rejection rate, and goal completion rate proposed in this concept are themselves subject to Goodhart's Law. An agent (or prompt engineer) optimizing to keep the human override rate below 40% may become better at avoiding escalations rather than better at producing correct outputs. The multi-metric approach mitigates this — degradation on any dimension triggers investigation — but it does not eliminate it. Treat the proposed thresholds as monitoring signals, not performance targets to optimize for in isolation.


<a name="loio4a9b908d1aad459cacf47e8414ae6667__section_quality_signal_escalation"/>

## Quality Signal Escalation Path

Quality monitoring is only useful if it leads to action. The escalation path connects runtime signals to the humans and systems that can respond.

```
Level 0 — Automated (no human involved)
  → Self-verification inline in agent workflow
  → BAIP harness anomaly detection
  → Threshold-based output validation (Code node / rule check)

Level 1 — Monitored (human alerted, can choose to intervene)
  → Agent Observability dashboard in SAP AI Agent Hub
  → Process KPI deviation alert to process owner
  → Human override rate threshold alert

Level 2 — Supervised (human must review before agent proceeds)
  → sapTaskCenter: approval task sent to named recipient
  → Agent pauses execution until Task Center decision received
  → Audit trail: what the agent proposed, what the human decided

Level 3 — Blocked (agent suspended, IT governance engaged)
  → Agent deactivated via SAP AI Agent Hub lifecycle management
  → Incident raised; root cause analysis required before reactivation
  → FDE team notified; may trigger rollback to prior agent version
```

Levels 0 and 1 are automated through the BAIP harness and Agent Hub. Levels 2 and 3 require explicit configuration: Level 2 requires a `sapTaskCenter` node in the agent workflow; Level 3 requires a governance policy configured in the Agent Hub's Secure & Govern capability.


<a name="loio4a9b908d1aad459cacf47e8414ae6667__section_fde_delivery_implications"/>

## Implications for FDE Delivery

The agent quality framework has direct implications for how FDE engagements are scoped and executed.

**During Discovery, sub-phases 2–4 (Generate–Prioritize):**

-   The TO-BE process flow must define not just what the agent does but how its outputs will be measured. Both process KPIs (Dimension 2) and the human override mechanism (Level 2 escalation) should be identified in the Draft Product Design Specs.
-   The value/feasibility scorecard (FDE Go/No-Go) should include explicit quality criteria: what eval threshold triggers a Go, and what runtime KPI degradation would trigger a reversion.

**During Develop (formerly Execute) (Rapid Prototyping → MVP Build):**

-   Eval test suites should be built alongside the agent, not after. The FDE Sprint ends when both the agent and its eval suite are complete.
-   Synthetic test data for eval runs should be prepared before the TEA is signed, so eval can begin on day one of the Development phase.
-   The `sapTaskCenter` human-in-the-loop node should be included in the MVP from the start. Adding it later requires redesigning the workflow and the UX.

**During Productization:**

-   The eval suite becomes part of the release artifact. Sirius tasks include security validation, QPA (Quality, Privacy and AI) test strategy, and AI ethics assessment — all of which require documented eval results.
-   The runtime quality configuration (alert thresholds, escalation policies) is part of the production handover. Customers should receive a documented quality baseline: what normal looks like, what triggers an alert, and who owns the response.


<a name="loio4a9b908d1aad459cacf47e8414ae6667__section_summary"/>

## Summary

**Table: Agent Quality Framework — Key Principles**


<table>
<tr>
<th valign="top">

Aspect

</th>
<th valign="top">

Key Principle

</th>
</tr>
<tr>
<td valign="top">

What to measure

</td>
<td valign="top">

Three dimensions simultaneously: technical, process KPI, business outcome

</td>
</tr>
<tr>
<td valign="top">

When to measure

</td>
<td valign="top">

Build-time (evals) and runtime (BAIP harness + Agent Hub observability)

</td>
</tr>
<tr>
<td valign="top">

Goodhart's Law guard

</td>
<td valign="top">

Multi-objective evaluation; alert on any dimension degrading

</td>
</tr>
<tr>
<td valign="top">

Self-verification

</td>
<td valign="top">

Inline validation before write-back; second model/rule-check for high stakes

</td>
</tr>
<tr>
<td valign="top">

Human-in-the-loop

</td>
<td valign="top">

First-class via `sapTaskCenter`; must be in MVP from day one

</td>
</tr>
<tr>
<td valign="top">

Data protection

</td>
<td valign="top">

BAIP harness telemetry stays in customer tenant by default

</td>
</tr>
<tr>
<td valign="top">

FDE integration

</td>
<td valign="top">

Quality criteria in TO-BE specs; eval suite ships with agent; thresholds in production handover

</td>
</tr>
</table>


<a name="loio4a9b908d1aad459cacf47e8414ae6667__section_four_pillars"/>

## The Four Pillars of Agent Excellence

SAP Signavio frames responsible AI agent adoption around four complementary pillars that span the full agent lifecycle — from identifying where agents should operate to proving their business value.

**Table: Four Pillars of Agent Excellence**


<table>
<tr>
<th valign="top">

Pillar

</th>
<th valign="top">

What It Covers

</th>
<th valign="top">

CIS Concept Mapping

</th>
</tr>
<tr>
<td valign="top">

1 — Agent Discovery

</td>
<td valign="top">

Identify the right processes and opportunities for AI agents; assess feasibility and expected value before committing

</td>
<td valign="top">

FDE Discovery phases ( readiness gates G1–G4; Project LightSpeed Stages 1–8)

</td>
</tr>
<tr>
<td valign="top">

2 — Agent Context

</td>
<td valign="top">

Provide agents with process knowledge, compliance parameters, domain rules, and operating guidelines so they reason correctly from the start

</td>
<td valign="top">

Pre-check constraints; system prompt grounding; RAG corpus design

</td>
</tr>
<tr>
<td valign="top">

3 — Agent Mining

</td>
<td valign="top">

Observe and analyze actual agent behavior in operation; detect deviations from designed process paths; identify flow variants and bottlenecks

</td>
<td valign="top">

Supervisor Workflow B; online eval statistical trends; Signavio agent behavior mining (forward-looking for Joule Studio 2.0; see Agent Behavior Mining section for GA status)

</td>
</tr>
<tr>
<td valign="top">

4 — Agent Value Impact

</td>
<td valign="top">

Quantify the business value the agent delivers: efficiency gains, cost reductions, process KPI improvements, consumed ACV

</td>
<td valign="top">

Dimension 3 — Business Outcomes; value realization tracking in FDE handover; continuous improvement loop

</td>
</tr>
</table>

These four pillars provide an adoption-oriented vocabulary that complements the technical three-dimension evaluation framework:

-   The **three dimensions** describe *how to measure* agent quality at a technical and operational level.
-   The **four pillars** describe *why and where* agents are deployed and what evidence the business needs to justify and sustain the investment.

For FDE Discovery conversations, the four pillars translate directly: Agent Discovery maps to opportunity identification and the FDE go/no-go gate; Agent Context maps to the grounding and constraints work in Develop; Agent Mining and Agent Value Impact are the post-productization evidence requirements that the CoE monitoring model is designed to satisfy.

**Related Information**  




[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "Introduction to SAP AI Agent Hub, its benefits, and core capabilities.") :arrow_upper_right:
[Observe and Analyze — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/1fdc1df35d414cd6bf2ed15ec8a164f4.html "Monitor agent usage, measure business value, and analyze portfolio health.") :arrow_upper_right:
[Agent Evaluation in FDE Sprints](../Agent-Evaluation/agent-evaluation-76db9f7.md)

[SAP AI Agent Hub — Plan and Build](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/e2a29b0c422e4aceadee658f4bbab71b.html "SAP AI Agent Hub Plan and Build documentation") :arrow_upper_right:

[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Joule Studio product overview on SAP Help Portal") :arrow_upper_right:
