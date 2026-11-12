<!-- loio16a4089179844b558b5905e084e431fa -->

# The Supervisor Agent Pattern

The supervisor agent is the CIS implementation of SAP's Supervision tier for Industry AI FDE engagements. It enforces architectural quality gates on agent write-backs, detects cross-invocation drift, and provides a data-minimizing escalation path to SAP support — all within the customer's own tenant boundary.



<a name="loio16a4089179844b558b5905e084e431fa__section_supervision_tier"/>

## The Supervision Tier and Why It Exists

SAP's AI-Native North Star Architecture defines a three-tier AI defense model for enterprise-grade agentic systems:

1.  **Foundation tier** — zero-trust authentication, traffic inspection, prompt-injection detection, behavioral analytics, cryptographic agent credentials. Provided by the BAIP platform; always on.
2.  **Supervision tier** — deterministic control agents that monitor, sanitize, and authorize other AI agents. Input validation, output filtering, anomaly detection, human oversight for high-risk decisions. Must be explicitly designed and deployed.
3.  **Automation tier** — AI agents scaling security operations; continuous policy audits, risk assessment, identity management.

The Foundation tier handles platform-level threats. The Automation tier handles organizational-scale security operations. The Supervision tier handles the gap between the two: **business-logic-level quality and safety for individual agent workflows**, within the customer's own environment, without requiring external data movement.

The supervisor agent is the CIS implementation of the Supervision tier for Industry AI FDE engagements. It is not a replacement for the Foundation tier (which the BAIP (Business AI Platform) harness provides automatically) or for the Agent Hub's governance capabilities (which operate at IT-admin level). It is the layer that has visibility into business-logic correctness in a way that neither the platform nor the IT admin can provide.

**The supervisor also serves as an autonomous CIS proxy on the customer tenant — in the deployments where CIS has no default access.** This rationale applies specifically to **BYOA runtimes on the customer's own BTP subaccount and to sovereign-cloud deployments**: there, the agent runs in the customer's own environment and CIS has no default access for monitoring. The supervisor bridges this gap: it performs the monitoring and intervention that CIS would otherwise need direct access to do — and only escalates to CIS when it has exhausted what it can handle autonomously. This design reduces the need for permanent CIS access to customer tenants to genuinely exceptional cases, without sacrificing operational visibility.

Under the **SMJ default** for new engagements (the SAP-managed orchestrator, typically paired with a low-code Joule Studio 2.0 runtime), the picture differs: the agent runs on SAP-managed infrastructure, so telemetry and governance are already reachable through the SAP AI Agent Hub and the managed harness, and the "no default CIS access" premise is weaker. The supervisor's proxy function is therefore most load-bearing for BYOA and sovereign runtimes; on the SMJ-managed path it is primarily a business-logic quality and write-gate layer rather than an access bridge. See [CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md) for the full CIS CoE (Center of Excellence) operations model that builds on this pattern.


<a name="loio16a4089179844b558b5905e084e431fa__section_alternative_approaches"/>

## Alternative Approaches and When Each Is Sufficient

Before committing to a supervisor agent, it is worth understanding the full range of design alternatives — what each covers, where it falls short, and under what conditions it is genuinely sufficient. This section covers nine alternatives in order of increasing capability, then explains why the supervisor pattern is the correct choice for most Industry AI FDE deployments operating beyond the initial EAC (Early Adopter Care) phase.

For a quick comparison across all alternatives, see the [summary table](#loio16a4089179844b558b5905e084e431fa__section_summary_comparison) at the end of this section.

The organizing question for each alternative: *what failure mode does it prevent, and how late is too late to catch it?* For irreversible writes — batch release quantities committed to a regulated system of record, financial postings, physical dispatches — post-hoc detection is not acceptable. The design must prevent the wrong output from being committed in the first place.

### Alternative A: Inline Self-Verification Only

A second validation step within the worker agent's own workflow, before write-back. The worker runs a Code node check and a second `sapOrchestration` critic call, then routes itself to write-back or Task Center based on the result.

**What it covers**: output validation for a single invocation at the write-back moment.

**What it does NOT cover**:

-   Cross-invocation patterns, drift over time, or repeated failure modes
-   Platform-level anomalies (error rate spikes, latency outliers)
-   Any issue that occurs after write-back
-   The CIS proxy function — the worker still controls the write-back decision

**Critical limitation**: the worker *can* be modified to bypass the verification step — it is code inside the same workflow. The quality gate is a design convention, not an architectural enforcement. For regulated outputs or high-value transactions, this distinction matters.

**Sufficient when**: the risk is concentrated at the write-back moment, the output can be verified inline, cross-invocation monitoring is not required, and the consequences of a bypassed gate are acceptable.

### Alternative B: SAP AI Agent Hub Governance Only

The Agent Hub's Observe & Analyze and Secure & Govern capabilities provide IT-admin-level visibility: agent registry, access control, lifecycle management, usage telemetry, policy enforcement.

**What it covers**: who is running which agents, with what permissions, at what frequency; policy enforcement and lifecycle events (deactivation, version rollback).

**What it does NOT cover**:

-   Whether individual agent outputs are correct for the business process
-   Whether the agent's outputs are drifting in a direction the platform cannot see — a batch release agent that is systematically recommending quantities 15% below correct values will not trigger any Agent Hub alert
-   Business-logic quality of any kind

**Critical limitation**: the Agent Hub operates at the IT-governance layer. It is authoritative for "is this agent authorized to run and does it comply with policy?" It has no visibility into "is this agent producing the right answer for this business process?"

**Sufficient when**: the primary concern is IT governance and access control, not business-logic quality. Can be used in combination with Alternative A for a lightweight two-layer approach.

### Alternative C: Human-in-the-Loop on Every Output

Every agent output goes through a `sapTaskCenter` approval step before write-back. Humans review and approve or reject each output individually.

**What it covers**: all errors — humans catch everything, by definition. No automated verification can match the judgment of a domain expert reviewing each output.

**What it does NOT cover**: the business case for AI automation. If every output requires human approval, the agent adds latency without reducing effort. The value-based contract (10–30% of yearly customer benefit) is premised on the agent reducing human work — not on routing every decision through a human faster.

**Critical limitation**: this is not a quality pattern for a mature deployment; it is a safe mode for situations where confidence in the agent has not yet been established. Running it beyond the initial stabilization period means the agent is generating process overhead rather than eliminating it.

**Sufficient when**: during the first weeks of a deployment before the behavioral baseline is established, or for specific output categories where regulatory requirements mandate human sign-off on every decision regardless of agent confidence.

### Alternative D: Asynchronous Batch Evaluation

Rather than intercepting outputs in real-time, run a periodic evaluation job (nightly or weekly) that scores a sample of *already committed* outputs against defined quality criteria. The SAP AI Agent Hub's Agent Evals & Assessment capability supports this pattern.

**What it covers**:

-   Quality trend analysis over time across many invocations
-   Ground-truth dataset construction: labeled (input, output, reference, verdict) records for improving build-time eval suites
-   Model drift detection at the population level (aggregate accuracy trends)
-   Low operational overhead — no impact on the agent's critical path

**What it does NOT cover**:

-   Prevention: by definition, outputs have already been committed when the eval runs. Any errors in the committed outputs have already had their downstream consequences.
-   Real-time anomaly detection or escalation

**Critical limitation**: too late for irreversible writes. A batch release quantity that was wrong when committed cannot be corrected by a nightly eval that detects the error the following morning. For regulated manufacturing, financial, or procurement write-backs, post-hoc detection is insufficient.

**Where it genuinely adds value**: as a *complement* to the supervisor pattern, not a replacement. Running async batch eval alongside the supervisor builds the ground-truth dataset that the continuous improvement loop ([CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md) ) consumes. The supervisor prevents committed errors in real-time; the async eval improves the agent's eval suite so future errors become less likely.

### Alternative E: Rule-Based Policy Engine Only (No LLM Critic)

Implement only the deterministic verification layer — a code-based gate the output must pass before write-back — without the LLM critic and without any telemetry monitoring. This is essentially Layer 2's deterministic stage from [Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md) , used as the sole quality control mechanism.

**What it covers**:

-   Detectable, enumerable failure modes: out-of-range values, wrong units, sign errors, precision violations, cross-field arithmetic inconsistency
-   Fully auditable and explainable verdicts — the Code node is inspectable by a quality engineer or auditor without requiring LLM interpretability

**What it does NOT cover**:

-   Contextual plausibility that rules cannot express: a quantity within the valid range that is nevertheless wrong given the customer's contract terms or current market conditions
-   Any failure mode not explicitly anticipated and codified in the rules
-   Drift detection, cross-invocation monitoring, CIS proxy function

**Interesting regulatory angle**: in some regulated contexts (GxP, financial controls), a pure rule-based gate may be *preferred* over an LLM critic precisely because it is fully auditable. A quality engineer can read and sign off on `value >= min && value <= max`; they cannot do the same for an LLM's internal reasoning. When regulatory sign-off on the verification logic itself is required, a rule-based engine is more defensible.

**Sufficient when**: all meaningful failure modes can be enumerated and expressed as rules; regulatory auditability of the verification logic is a hard requirement; or as a starting point before the LLM critic is added during the EAC calibration phase.

### Alternative F: Confidence Gating Only

Rely entirely on the agent's own confidence score — above threshold → write-back, below threshold → escalate. No independent verification stage.

**What it covers**: low-confidence outputs are flagged for human review before write-back.

**What it does NOT cover**: confident-but-wrong outputs.

**Critical limitation**: LLM confidence scores are unreliable as a sole quality gate. Large language models are well-documented to produce high-confidence incorrect outputs — the phenomenon of "hallucination with certainty" is precisely the failure mode that confidence gating alone cannot catch. An agent that produces 12,500 kg when the correct figure is 1,250 kg may do so with 0.94 confidence.

Confidence is a useful *supplementary* routing signal within a multi-stage verification stack ([Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md) ) — a low confidence score adds a reason to escalate even when deterministic checks pass. It is not a standalone quality gate for write-back agents.

**Sufficient when**: advisory outputs only, where the human reading the output exercises their own judgment anyway and the agent's confidence is a useful context signal rather than a decision gate.

### Alternative G: Downstream Reconciliation (Optimistic Write + Post-Hoc Correction)

The agent writes optimistically — without a quality gate — and a reconciliation job runs periodically to detect anomalies in committed data, flagging them for correction. This is the compensating transaction pattern from distributed systems applied to agent outputs.

**What it covers**:

-   Eventual detection of systematic errors across a population of committed outputs
-   No impact on write-back latency — the agent's critical path is uninterrupted

**What it does NOT cover**:

-   The window between commitment and detection: wrong data may have propagated to downstream processes, triggered physical actions, or been included in regulatory reports
-   Irreversible writes: a posted financial transaction, a released production batch, a dispatched shipment cannot be "uncommitted" by a reconciliation job

**Critical limitation**: this pattern assumes errors are reversible and cheap to correct. For the Industry AI domains this concept covers — Regulated Manufacturing, Commodity Management, Revenue Growth Management — that assumption does not hold. A wrong hedge ratio that propagates for 24 hours before detection creates real financial exposure. A released batch that is recalled creates regulatory, operational, and reputational consequences that correction cannot undo.

**Sufficient when**: outputs are advisory (no automatic write-back), or write-back actions are genuinely reversible with low correction cost, in non-regulated contexts.

### Alternative H: Multi-Model Consensus (Ensemble Approach)

Run the same task through N independent LLMs or agent configurations simultaneously. Commit only when their outputs agree within a defined tolerance; escalate on disagreement.

**What it covers**:

-   Reduces the probability of confident-but-wrong outputs: different models have different systematic failure modes; correlated errors across independent models are significantly less likely than errors in any single model
-   Disagreement between models is itself an informative signal — it surfaces cases of genuine ambiguity that no single model can reliably resolve

**What it does NOT cover**:

-   Failure modes shared by all models in the ensemble (systematic biases from similar training data)
-   Drift detection, cross-invocation monitoring, CIS proxy function
-   Cost: N× LLM calls per invocation

**Cost constraint**: the AI Units billing model ([Production Operations](../Production-Operations/production-operations-edd9e33.md) ) makes the cost implication concrete. Three Advanced-tier LLM calls per invocation instead of one changes the cost-per-committed-output metric significantly — and the FDE value case (10–30% of yearly customer benefit) depends on agent cost being substantially lower than the cost reduction generated. For high-frequency agents, ensemble approaches may consume the efficiency savings that justify the agent in the first place.

**Where it genuinely adds value**: for extremely high-stakes, low-frequency decisions — a major financial commitment, a one-off regulatory submission — where the cost of N× inference is small relative to the cost of an error, and where the additional confidence from multi-model agreement provides material risk reduction. Rare in typical FDE engagement scope; worth considering for specific decision types within an engagement.

### Alternative I: Asynchronous Human Review (Non-Blocking)

The agent writes back immediately; a queue of recent decisions is routed to human reviewers asynchronously. Reviewers work through the queue and can flag corrections after the fact.

**What it covers**: human oversight without adding latency to the agent's critical path. Useful for high-volume, latency-sensitive workflows where blocking approval would eliminate the throughput benefit.

**What it does NOT cover**: prevention. The output is committed before human review. This shares the fundamental limitation of Alternatives G and D: if the write is irreversible, post-hoc review cannot undo the consequences.

**Sufficient when**: the agent handles high-volume advisory outputs or easily reversible writes where throughput matters more than prevention, and the human review queue is genuinely being processed (not accumulating unreviewed).


<a name="loio16a4089179844b558b5905e084e431fa__section_summary_comparison"/>

## Summary: Comparison Across All Alternatives

**Table: Alternative Approaches Compared**


<table>
<tr>
<th valign="top">

Alternative

</th>
<th valign="top">

Prevents Wrong Commits?

</th>
<th valign="top">

Detects Drift?

</th>
<th valign="top">

CIS Proxy?

</th>
<th valign="top">

Key Constraint

</th>
</tr>
<tr>
<td valign="top">

A: Inline self-verification

</td>
<td valign="top">

Partially (worker controlled)

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

Worker can bypass gate

</td>
</tr>
<tr>
<td valign="top">

B: Agent Hub governance only

</td>
<td valign="top">

No

</td>
<td valign="top">

No (IT-level only)

</td>
<td valign="top">

No

</td>
<td valign="top">

No business-logic quality visibility

</td>
</tr>
<tr>
<td valign="top">

C: Human on every output

</td>
<td valign="top">

Yes

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

Eliminates automation efficiency

</td>
</tr>
<tr>
<td valign="top">

D: Async batch eval

</td>
<td valign="top">

No (post-hoc only)

</td>
<td valign="top">

Yes (trend)

</td>
<td valign="top">

No

</td>
<td valign="top">

Too late for irreversible writes

</td>
</tr>
<tr>
<td valign="top">

E: Rule-based engine only

</td>
<td valign="top">

Yes (known failure modes)

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

Cannot catch contextual errors

</td>
</tr>
<tr>
<td valign="top">

F: Confidence gating only

</td>
<td valign="top">

No (confident errors pass)

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

LLM confidence scores are unreliable

</td>
</tr>
<tr>
<td valign="top">

G: Downstream reconciliation

</td>
<td valign="top">

No (optimistic write)

</td>
<td valign="top">

Partially

</td>
<td valign="top">

No

</td>
<td valign="top">

Irreversible writes cannot be corrected

</td>
</tr>
<tr>
<td valign="top">

H: Multi-model consensus

</td>
<td valign="top">

Yes (for enumerable disagreements)

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

N× token cost; shared systematic failures

</td>
</tr>
<tr>
<td valign="top">

I: Async human review

</td>
<td valign="top">

No (post-hoc)

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

Does not prevent committed errors

</td>
</tr>
<tr>
<td valign="top">

**Supervisor pattern**

</td>
<td valign="top">

**Yes (architecturally enforced)**

</td>
<td valign="top">

**Yes**

</td>
<td valign="top">

**Yes**

</td>
<td valign="top">

**Build complexity; Workflow A per output type**

</td>
</tr>
</table>

### The Supervisor Agent: When It Is Needed

The supervisor pattern is the only approach that simultaneously prevents wrong outputs from being committed through architectural enforcement — the **committer pattern** (), where the worker holds no system-of-record write scope and a separate committer identity performs the write, not a design convention — detects cross-invocation patterns and drift without requiring CIS access to the customer tenant, and provides a data-minimizing escalation path to SAP support. No single alternative covers all three.

The supervisor agent is the right choice when:

-   The agent is deployed at production scale with significant autonomy (not every output reviewed by a human)
-   Business-logic quality cannot be fully captured by inline self-verification alone
-   Cross-invocation patterns matter (drift, anomalies, repeated failure modes)
-   Data protection requires that telemetry monitoring stays within the customer's tenant
-   Selective escalation to SAP support — with customer-controlled data sanitization — is a design requirement
-   Write-back actions are irreversible or regulated — the cost of a committed error exceeds the cost of building and governing the supervisor

In practice this describes most Industry AI FDE scenarios operating beyond the initial EAC phase. Alternatives A and D are the most viable complements: Alternative A as the lightweight path for advisory or lower-stakes outputs within the same engagement, and Alternative D (async batch eval) as a continuous improvement mechanism running alongside the supervisor — not instead of it.

**Mixed-stakes engagements**: a realistic FDE engagement typically contains agents with different risk profiles — one or two high-stakes write-back agents (batch release quantities, hedge ratios, financial postings) alongside several advisory or reporting agents (maintenance summaries, procurement analysis, scheduling views). The supervisor pattern is warranted for write-back agents where errors are irreversible. For advisory agents in the same engagement whose outputs are reviewed by a human before any action is taken, Alternative A (inline self-verification) is often sufficient. Scope the supervisor to the write-back risk surface within the engagement, not to every agent uniformly.


<a name="loio16a4089179844b558b5905e084e431fa__section_three_functions"/>

## The Supervisor Agent: Three Functions

The supervisor is not a single monolithic agent. It is a set of three distinct functions, each with a different trigger, execution frequency, and escalation path. Decomposing them into separate Joule Studio workflows makes each function independently deployable, testable, and governed.

### Function 1: Inline Quality Relay

**Purpose**: receive the worker agent's output before it is committed to the system of record, run multi-stage verification, and route to write-back or escalation.

**Trigger**: each worker agent invocation (via callback from async worker call).

**Execution pattern**:

```
Worker agent invoked (sapAgent, blocking=false, callbackUrl=supervisor/callback)
         ↓
Supervisor Workflow A receives callback
         ↓
[Stage 1 — Deterministic check: Code node or sapMcpClient calculation]
[Stage 2 — LLM critic: sapOrchestration with critic system prompt]
         ↓
Decision routing (If/Switch node):
    ├── BOTH CONFIRM  → commit: supervisor/committer identity writes (sapMcpClient / OData)
    ├── BOTH REJECT   → escalate: sapTaskCenter with full context
    ├── DISAGREE      → escalate: sapTaskCenter (uncertainty requires human judgment)
    └── DETERMINISTIC UNAVAILABLE → LLM verdict only: route on the LLM-critic confidence threshold (default 0.7, calibrated during EAC — see)
```

The worker agent cannot write to the system of record directly — it holds no write scope at all. It is scoped **read/recommend-only** and returns a **structured proposal** (JSON: the proposed values, the supporting context, and the worker's own confidence) to the supervisor callback. On BOTH CONFIRM, the **supervisor/committer identity** — which holds the write scope the worker lacks — performs the write itself (via `sapMcpClient` or a direct OData call); the worker is never re-invoked to write. On any other verdict the supervisor escalates without a write.

This is the **committer pattern**, and it is what makes the gate *architecturally enforced* rather than a design convention:

-   The worker's effective permission is `min(user, agent)`; with no write scope on the agent identity, the **SAP Agent Gateway denies a direct worker write at the API-call level** — the worker cannot bypass the supervisor even if its own workflow were modified.
-   The write is attributed in the audit trail to the **committer identity**, not the worker: the system of record records which identity actually committed the value.
-   An optional **S/4HANA RAP instance-authorization backstop** rejects the write unless the target record is in an approved state — a third independent gate behind the Gateway scope denial and the committer-held write scope.

This inversion of control is the core of the supervisor pattern, and it aligns with least privilege (SEC-374): the worker produces a proposal, the supervisor decides, and the committer identity — not the worker — writes.

### Function 2: Telemetry Monitor

**Purpose**: continuously monitor the BAIP managed runtime harness telemetry within the customer's tenant, detect anomalies and drift across invocations, and trigger escalation when patterns exceed defined thresholds.

**Trigger**: Schedule Trigger (e.g., every 5–15 minutes depending on agent call volume).

**Execution pattern**:

```
Supervisor Workflow B fires on schedule
         ↓
HTTP Request → SAP Cloud ALM pull APIs (`hm` provider; SAP-operated, region-bound tenant)
         ↓
Code node: rule-based anomaly classification
  - Error rate vs. rolling baseline
  - Tool call count deviation (agent taking unusual paths)
  - Response time percentile drift
  - Repeated identical inputs (possible loop condition)
  - Unusual output distribution (e.g., all outputs above threshold)
         ↓
Decision routing:
    ├── No anomaly → log baseline update, no action
    ├── Anomaly — internal  → sapTaskCenter: alert customer IT admin
    └── Anomaly — SAP support needed → trigger Workflow C (Support Relay)
```

**Critical design constraint**: the anomaly classifier in Workflow B must be **rule-based, not LLM-based**. Sending raw BAIP traces to an LLM for classification would route business data through a model, which re-introduces the exact privacy concern the intra-tenant design was meant to prevent. Rules, thresholds, and code-based pattern matching are the correct tools here.

### Function 3: Support Relay

**Purpose**: when Workflow B identifies an anomaly that requires SAP support engagement, sanitize the relevant trace extract, create a support ticket, and notify the customer's IT team — without exporting raw operational data.

**Trigger**: event published by Workflow B when anomaly severity exceeds the external engagement threshold.

**Execution pattern**:

```
Supervisor Workflow C receives anomaly event from Workflow B
         ↓
Retrieve specific trace from SAP Cloud ALM (the anomalous invocation only)
         ↓
Sanitization pass (Code node or sapOrchestration with `enableMasking`):
  - Remove PII (Personally Identifiable Information) via DPI (Data Privacy Integration) entity masking
  - Remove operational business data (order IDs, material numbers, quantities)
  - Retain: agent ID, tool call sequence, error codes, timing, exception messages
         ↓
SAP Cloud ALM ITSM Adapter API → create support ticket
  (attachment: sanitized trace extract, severity, agent identity, customer tenant ID)
         ↓
HTTP notification → customer IT admin (email / SAP BAIP notification service)
```

The customer controls what gets exported by defining the sanitization rules in Workflow C. The raw trace is never attached to the outbound support ticket. Only the structural anomaly evidence — what sequence of steps failed, which error codes appeared — is included.


<a name="loio16a4089179844b558b5905e084e431fa__section_architecture_joule_studio"/>

## Supervisor Architecture in Joule Studio

### Topology Rule: One Actor, One Agent

When deriving agent topology from customer discovery, a single deterministic rule governs the initial topology:

**Each customer persona (Actor) gets its own dedicated specialist agent. No merging, no sharing.**

A single MasterSupervisorAgent (or equivalent orchestration supervisor) routes between specialist agents based on user intent and routing descriptions. This rule enforces separation of concerns: each specialist agent handles only the domain its Actor works in, making routing deterministic and avoiding mixed-context responses.

**Rationale**: merged agents — where one agent handles the responsibilities of two or more Actors — require complex routing logic embedded within the agent itself and tend to produce responses that mix contexts from different Actor domains. One Actor per agent keeps routing descriptions distinct, agent behavior predictable, and specialist prompts tightly scoped.

**Important distinction**: the orchestration supervisor (routes intent to the correct specialist) is distinct from the governance supervisor described in this chapter (monitors, guards, validates). A production deployment typically has both:

**Table: Supervisor Types Compared**


<table>
<tr>
<th valign="top">

Supervisor Type

</th>
<th valign="top">

Function

</th>
<th valign="top">

Position in Architecture

</th>
</tr>
<tr>
<td valign="top">

Orchestration supervisor (MasterSupervisorAgent)

</td>
<td valign="top">

Routes user intent to the correct specialist agent

</td>
<td valign="top">

Entry point; upstream of all specialist agents

</td>
</tr>
<tr>
<td valign="top">

Governance supervisor (this chapter)

</td>
<td valign="top">

Monitors output quality, guards write-backs, validates results

</td>
<td valign="top">

Workflow A: per write-back agent (1:1); Workflow B: system-wide across all agents (1:N)

</td>
</tr>
</table>

The governance supervisor's three workflows have different scoping rules — this is the key design decision when combining the governance pattern with a multi-agent orchestration topology:

**Table: Governance Workflow Scoping**


<table>
<tr>
<th valign="top">

Workflow

</th>
<th valign="top">

Natural Scope

</th>
<th valign="top">

Relationship

</th>
</tr>
<tr>
<td valign="top">

**Workflow A** — Inline quality relay

</td>
<td valign="top">

Per write-back agent (or output type)

</td>
<td valign="top">

**1:1** — each agent doing irreversible write-backs has its own Workflow A

</td>
</tr>
<tr>
<td valign="top">

**Workflow B** — Telemetry monitor

</td>
<td valign="top">

Entire topology

</td>
<td valign="top">

**1:N** — one monitoring loop watches all agents' telemetry

</td>
</tr>
<tr>
<td valign="top">

**Workflow C** — Support relay

</td>
<td valign="top">

Triggered by Workflow B

</td>
<td valign="top">

On-demand; inherits Workflow B's system-wide scope

</td>
</tr>
</table>

The 1:1 relationship is preserved exactly where it matters: the inline quality gate (Workflow A) sits between each write-back agent and the system of record. Advisory agents and routing agents that do not write back irreversibly do not need Workflow A at all. The 1:N monitoring (Workflow B) is appropriate for telemetry because it is metric collection — not an architectural gate on any individual transaction.

The resulting topology for a multi-agent engagement:

```
Workflow B  (telemetry monitor — 1:N, watches all)
  └─ MasterSupervisorAgent + SpecialistAgent_1 .. SpecialistAgent_N

Workflow A_1  (quality gate — 1:1, write-back agents only)
  └─→ SpecialistAgent_Actor1  ← does irreversible write-backs

[No Workflow A for SpecialistAgent_Actor2 if it is advisory / read-only]

MasterSupervisorAgent  (orchestration — routes intent, no write-back)
  ├─→ SpecialistAgent_Actor1
  └─→ SpecialistAgent_Actor2
```

The one-actor-one-agent rule applies to the specialist agents. The governance supervisor is external to the routing chain and does not participate in routing decisions — Workflow B can therefore monitor the MasterSupervisorAgent itself (detecting mis-routing, loops, or anomalous routing patterns) in addition to the specialist agents.

As noted in the alternatives section: **scope the supervisor to the write-back risk surface within the engagement, not to every agent uniformly.** In most multi-agent FDE engagements, only one or two specialist agents perform irreversible write-backs. The remaining agents are advisory or read-only and do not require Workflow A.

This rule is enforced automatically in Project LightSpeed's topology generation phase (Stage 5), which maps each discovered Actor primitive ([Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md),) to a dedicated agent specification.

### Worker-Supervisor Communication

The `CUSTOM.sapAgent` node provides the mechanism for inter-agent communication. The key parameter is `blocking`:

-   `blocking: true` — the calling workflow waits for the worker agent to complete. Simple, but ties up the calling workflow thread for the full duration of the worker.
-   `blocking: false` — the calling workflow fires the worker and immediately releases. The worker sends its result to the `callbackUrl` when complete. The supervisor Workflow A registers a `Webhook` node as the callback endpoint.

For the supervisor pattern, `blocking: false` is the correct choice. It decouples the worker's execution time from the supervisor's evaluation time, and allows multiple workers to run concurrently with a single supervisor receiving their callbacks.

**Verified node parameters** (`CUSTOM.sapAgent` v1.4.9, verified 2026-10-21):

**Table: sapAgent Node Parameters**


<table>
<tr>
<th valign="top">

Parameter

</th>
<th valign="top">

Type

</th>
<th valign="top">

Default

</th>
<th valign="top">

Purpose

</th>
</tr>
<tr>
<td valign="top">

`agentName`

</td>
<td valign="top">

options

</td>
<td valign="top">

—

</td>
<td valign="top">

Select the SAP Agent to invoke

</td>
</tr>
<tr>
<td valign="top">

`blocking`

</td>
<td valign="top">

boolean

</td>
<td valign="top">

`true`

</td>
<td valign="top">

`false` = fire-and-release; requires `callbackUrl`

</td>
</tr>
<tr>
<td valign="top">

`callbackUrl`

</td>
<td valign="top">

string

</td>
<td valign="top">

—

</td>
<td valign="top">

Webhook URL for async result delivery (non-blocking only)

</td>
</tr>
<tr>
<td valign="top">

`sessionId`

</td>
<td valign="top">

string

</td>
<td valign="top">

—

</td>
<td valign="top">

Correlates callbacks when multiple workers are in flight

</td>
</tr>
<tr>
<td valign="top">

`customContext`

</td>
<td valign="top">

JSON

</td>
<td valign="top">

`{}`

</td>
<td valign="top">

Additional context passed to the agent

</td>
</tr>
<tr>
<td valign="top">

`timeout`

</td>
<td valign="top">

number

</td>
<td valign="top">

180,000ms

</td>
<td valign="top">

Max 300,000ms (5 min); applies in blocking mode only

</td>
</tr>
</table>

```
[Initiating workflow]
    └── sapAgent: agentName=worker, blocking=false, callbackUrl={{supervisor_webhook_url}}
               ↓ (fires and releases; does not wait)
[Worker agent executes autonomously]
               ↓ (completes; sends result to callbackUrl)
[Supervisor Workflow A: Webhook trigger]
    └── receives worker output → runs multi-stage verification → routes decision
```

The `sessionId` parameter in `sapAgent` is used to correlate the callback to the original request. This is essential when multiple concurrent worker invocations are in flight — the supervisor must know which verification result belongs to which request.

### Decomposition into Three Workflows

The three functions map to three independent Joule Studio n8n workflows:

**Table: Three Supervisor Workflows**


<table>
<tr>
<th valign="top">

Workflow

</th>
<th valign="top">

Trigger

</th>
<th valign="top">

Primary Nodes

</th>
<th valign="top">

Execution Frequency

</th>
</tr>
<tr>
<td valign="top">

A — Inline Quality Relay

</td>
<td valign="top">

Webhook (worker callback)

</td>
<td valign="top">

Code / sapMcpClient (verify), sapOrchestration (LLM critic), If, sapTaskCenter, sapMcpClient / OData (committer write)

</td>
<td valign="top">

Per worker invocation

</td>
</tr>
<tr>
<td valign="top">

B — Telemetry Monitor

</td>
<td valign="top">

Schedule Trigger

</td>
<td valign="top">

HTTP Request (SAP Cloud ALM pull APIs, `hm` provider), Code (rule-based classifier), If, sapTaskCenter, n8n workflow call (→ Workflow C)

</td>
<td valign="top">

Periodic (5–15 min)

</td>
</tr>
<tr>
<td valign="top">

C — Support Relay

</td>
<td valign="top">

Called by Workflow B

</td>
<td valign="top">

Code (sanitize), HTTP Request (SAP Cloud ALM ITSM Adapter API), HTTP Request (notification)

</td>
<td valign="top">

On demand (anomaly events only)

</td>
</tr>
</table>

Decomposing into three workflows has practical advantages beyond clean separation of concerns: each workflow can be versioned, tested, and governed independently in the SAP AI Agent Hub. If Workflow B needs a threshold adjustment, it can be updated without touching the inline quality logic in Workflow A. Each workflow also has its own agent identity and scoped authorizations in the BAIP harness.

### Shared State

The three workflows need a common data store for:

-   The rolling baseline used by Workflow B's anomaly classifier (invocation counts, error rates, tool call distributions)
-   Pending escalations from Workflow A (Task Center tasks that have not yet been resolved by a human)
-   Correlation between worker `sessionId` and verification status

In BAIP, this shared state can be held in:

-   A SAP HANA Cloud table (persistent, queryable, supports concurrent writes)
-   A SAP Build Process Automation work item store (if the process automation module is part of the BAIP subscription)
-   A BAIP Key-Value store (for lightweight, ephemeral state)

The choice is a deployment decision that depends on the customer's BAIP entitlements. For FDE engagements, SAP HANA Cloud is the most consistently available option across Industry AI target customers.

### LoopCraft — KG-Enriched Execution Cycle

For agents with access to a Knowledge Graph (HANA KG, EKX, or the CIS Context Layer KG), the recommended execution pattern is a three-step enriched cycle:

**Query KG → Execute Action → Verify Result**

```
[Query KG]
  → Retrieve entity attributes, constraints, prior decisions, related processes
        ↓
[Execute Action]
  → Tool call or decision with full KG context
        ↓
[Verify Result]
  → Check result against KG constraints or expected values
  → If passes: surface to user or write to system of record
  → If fails: retry with enriched context or escalate to supervisor
```

Each step in detail:

-   **Query KG**: before executing, the agent retrieves relevant context from the knowledge graph — entity attributes, business constraints, prior decisions about related entities, and related process state. This grounds the execution in verified facts rather than model-generated inference.
-   **Execute Action**: the agent executes the tool call or decision with full KG context available. The context from the KG reduces the probability of hallucinated values and provides the agent with the expected range of correct outputs.
-   **Verify Result**: the agent checks the result against KG constraints or expected values before surfacing it to the user or writing it back to the system of record. This is the agent's own internal quality gate.

This cycle is the agentic equivalent of the self-verification loop in the North Star Architecture Layer 2 (see also [Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md),). It operates as an **internal quality gate within the agent's own execution** — distinct from the governance supervisor pattern (which monitors from outside). The supervisor remains the architectural enforcement layer; LoopCraft is the agent's own discipline during execution.

The KG-enriched cycle is most valuable for agents handling Entities and BusinessObjects where SAP canonical attributes and constraints are already encoded in the knowledge graph, particularly for numerical outputs subject to the precision guardrails in [Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md).


<a name="loio16a4089179844b558b5905e084e431fa__section_data_protection"/>

## Data Protection Design

### What Stays in the Tenant

The following data never leaves the customer's BAIP tenant under normal operation:

-   Raw BAIP harness traces (agent execution traces, tool call inputs/outputs)
-   Worker agent output before verification approval
-   LLM call history (prompts, responses) from both worker and supervisor
-   Business data used as grounding context (order data, material data, etc.)
-   Intermediate verification results (stage 1 and stage 2 verdicts)
-   Human decisions from SAP Task Center

Intra-tenant data movement (between workflows, to the shared state store) is not "export" — it stays within the customer's tenant boundary. The one exception is the telemetry pull path to **SAP Cloud ALM**: it moves data out of the customer's BTP subaccount into the SAP-operated Cloud ALM tenant. The accurate statement is therefore not "telemetry does not leave the tenant" but that this telemetry is **stored in the SAP Cloud ALM tenant provisioned in your contracted data-residency region (SAP-operated, region-bound)**.

### What Can Leave the Tenant (and Under What Conditions)

Data leaves the tenant in exactly two cases:

1.  **Sanitized trace extract** — only when Workflow C fires, only for invocations where an anomaly was confirmed, only after the sanitization pass. The customer's IT admin can review and block Workflow C if they choose not to permit any export.
2.  **Escalation metadata** — `sapTaskCenter` tasks are within the SAP Task Center service, which may be multi-tenant infrastructure. The task subject, description, and custom attributes should not contain business-specific data; they should contain agent identity, severity, and the verification verdict only.

### Sanitization Rules

Workflow C's sanitization pass should be governed by explicit rules, not LLM judgment:

**Table: Trace Field Sanitization Rules**


<table>
<tr>
<th valign="top">

Trace Field

</th>
<th valign="top">

Rule

</th>
</tr>
<tr>
<td valign="top">

Error message / exception text

</td>
<td valign="top">

Retain as-is (structural, not business data)

</td>
</tr>
<tr>
<td valign="top">

Tool call name (e.g., `get_purchase_order`)

</td>
<td valign="top">

Retain as-is (reveals capability, not data)

</td>
</tr>
<tr>
<td valign="top">

Tool call parameters (e.g., PO number, vendor ID)

</td>
<td valign="top">

**Strip** — replace with `[REDACTED]`

</td>
</tr>
<tr>
<td valign="top">

LLM response content

</td>
<td valign="top">

**Strip** — may contain business data, reasoning

</td>
</tr>
<tr>
<td valign="top">

LLM system prompt

</td>
<td valign="top">

**Strip** — may reveal internal business logic

</td>
</tr>
<tr>
<td valign="top">

Agent session ID / correlation ID

</td>
<td valign="top">

Retain (needed for support investigation)

</td>
</tr>
<tr>
<td valign="top">

Timing / latency data

</td>
<td valign="top">

Retain (needed for performance diagnosis)

</td>
</tr>
<tr>
<td valign="top">

Agent identity (agent name, version)

</td>
<td valign="top">

Retain

</td>
</tr>
<tr>
<td valign="top">

Tenant ID

</td>
<td valign="top">

Retain (SAP support needs this)

</td>
</tr>
</table>

These rules should be reviewed and approved by the customer's data protection officer before Workflow C is deployed. The FDE Collaboration Agreement (CA) — which is required if any operational data leaves the customer's environment — may apply to the support relay function even though the data is sanitized. This is a legal question, not a technical one: the legal review for Workflow C must be completed during the FDE Discovery phase as part of the legal agreement planning deliverable ([Security and Data Protection](../Security-and-Data-Protection/security-and-data-protection-90b7501.md) and [Reusable Components and Reference Implementations](../Reusable-Components/reusable-components-cb60e89.md)). The responsible party for initiating this review is the FDE Delivery Lead, in coordination with CIS Legal. Do not defer this to the Development phase — CA approval can take 10–20 business days ([Security and Data Protection](../Security-and-Data-Protection/security-and-data-protection-90b7501.md)), and a late CA blocks Workflow C deployment without a workaround.


<a name="loio16a4089179844b558b5905e084e431fa__section_escalation_thresholds"/>

## Escalation Thresholds and Decision Logic

The supervisor must have clearly defined thresholds to avoid two failure modes: **over-escalation** (humans overwhelmed by false alarms, stop trusting the system) and **under-escalation** (real issues pass undetected because the threshold was too permissive).

Thresholds should be calibrated per agent, not globally. A procurement agent handling €10M orders requires tighter thresholds than a reporting assistant.

### Inline Quality Relay (Workflow A)

**Table: Workflow A Decision Routing**


<table>
<tr>
<th valign="top">

Condition

</th>
<th valign="top">

Action

</th>
</tr>
<tr>
<td valign="top">

Both stages confirm

</td>
<td valign="top">

Approve write-back; log to baseline

</td>
</tr>
<tr>
<td valign="top">

Both stages reject

</td>
<td valign="top">

Block write-back; escalate to named process owner via sapTaskCenter

</td>
</tr>
<tr>
<td valign="top">

Stages disagree

</td>
<td valign="top">

Block write-back; escalate with both verdicts visible to reviewer

</td>
</tr>
<tr>
<td valign="top">

LLM critic confidence below threshold (default 0.7)

</td>
<td valign="top">

Treat as DISAGREE; escalate. Calibrate the threshold during the EAC phase

</td>
</tr>
<tr>
<td valign="top">

Deterministic check unavailable

</td>
<td valign="top">

Proceed on LLM verdict alone; log the gap

</td>
</tr>
<tr>
<td valign="top">

Worker timed out / error

</td>
<td valign="top">

Block write-back; escalate with error context

</td>
</tr>
</table>

### Telemetry Monitor (Workflow B)

**Table: Workflow B Anomaly Thresholds**


<table>
<tr>
<th valign="top">

Signal

</th>
<th valign="top">

Threshold

</th>
<th valign="top">

Action

</th>
</tr>
<tr>
<td valign="top">

Error rate

</td>
<td valign="top">

> 3× rolling 7-day average for 3 consecutive periods

</td>
<td valign="top">

IT admin alert

</td>
</tr>
<tr>
<td valign="top">

Tool call count per invocation

</td>
<td valign="top">

> 2× p95 baseline for 5 consecutive invocations

</td>
<td valign="top">

IT admin alert + log investigation flag

</td>
</tr>
<tr>
<td valign="top">

Response latency

</td>
<td valign="top">

> 5× p95 baseline for 2 consecutive invocations

</td>
<td valign="top">

IT admin alert

</td>
</tr>
<tr>
<td valign="top">

Human override rate (from Task Center)

</td>
<td valign="top">

> 40% over 7-day window

</td>
<td valign="top">

Process owner review request

</td>
</tr>
<tr>
<td valign="top">

Verification rejection rate (from Workflow A)

</td>
<td valign="top">

> 20% over 24-hour window

</td>
<td valign="top">

Process owner review + Workflow C candidate

</td>
</tr>
<tr>
<td valign="top">

Repeated identical input pattern

</td>
<td valign="top">

Same input parameters 3× within 5 minutes

</td>
<td valign="top">

Possible loop — immediate IT admin alert

</td>
</tr>
</table>

These thresholds are starting points. They should be calibrated during the EAC phase using observed baseline behavior before GA deployment.

### Day-1 Cold-Start Guidance

Every threshold above references rolling baselines and p95 values that require historical data to be meaningful. A new deployment has none. Without explicit guidance, baseline-relative thresholds are either undefined (producing alert storms or silence depending on implementation) or computed from too few data points to be statistically reliable.

**For the initial observation period — Day 1 through the first 50+ invocations:**

-   **Workflow A (Inline Quality Relay)**: active from day one. It makes per-invocation decisions without requiring a baseline.
-   **Workflow B (Telemetry Monitor)**: use static absolute thresholds rather than baseline-relative ones. Example starting values for the observation period: error rate > 15% absolute, human override rate > 60% absolute. Switch to baseline-relative thresholds once the observation period is complete.
-   **Anomaly detection alerts**: disabled until 50+ invocations have been recorded, or 5+ business days of operation — whichever is longer for low-frequency agents.

**Low-frequency agents and baseline fragility**: for agents processing one or two invocations per hour, 72 hours of observation yields ~72–144 data points. A p95 computed from 72 points is dominated by noise; a single outlier early on can permanently skew the rolling baseline. For agents with fewer than ~200 weekly invocations, extend the initial observation period to 2–3 weeks before enabling baseline-relative thresholds.

Document the observation period and the static starting thresholds in the production handover package as a planned configuration transition — so the customer IT admin and process owner understand that anomaly detection is calibrating, not broken, during the first weeks of operation.


<a name="loio16a4089179844b558b5905e084e431fa__section_supervisor_governance"/>

## The Supervisor's Own Governance

The supervisor agent is itself a BAIP-managed agent with a first-class identity, scoped authorizations, and its own telemetry. This is important: a malfunctioning supervisor is a systemic failure, not just an individual transaction failure.

The supervisor's governance requirements:

-   **Agent identity**: separate application registration in BAIP for each supervisor workflow (A, B, C). Each has its own service credentials and authorization policies.
-   **Least-privilege**: the supervisor/committer identity holds read access to the SAP Cloud ALM telemetry pull path, write access to SAP Task Center, **and** the system-of-record write scope needed to commit approved outputs. The **worker** holds no system-of-record write scope at all (committer pattern,): it is read/recommend-only, so the SAP Agent Gateway denies any direct worker write. This inverts the naive arrangement — the write scope lives with the governed committer identity, not with the worker — which is exactly what makes the gate enforceable and what aligns the design with least privilege (SEC-374).
-   **Supervisor observability**: the supervisor's own telemetry is visible in the SAP AI Agent Hub, just like the worker's. If the supervisor itself is malfunctioning (failing to receive callbacks, not triggering escalations), the Agent Hub's IT-admin layer detects this.
-   **Supervisor deactivation**: if the supervisor becomes unavailable, the default behavior of the worker should be to route all outputs to human review (`sapTaskCenter`), not to write-back autonomously. The fail-safe is always toward more human oversight, not less.


<a name="loio16a4089179844b558b5905e084e431fa__section_limitations"/>

## Limitations and Honest Constraints

### Deterministic Verification Requires That the Algorithm Exists

The multi-stage verification pattern assumes a deterministic check can be implemented. For some use cases this is not possible: if the agent's task is genuinely open-ended or judgment-based (e.g., summarizing a maintenance report, drafting a customer communication), there is no algorithmic ground truth. In these cases, the LLM critic remains the only automated check, and its independence limitation applies.

### The Supervisor Cannot Verify What It Cannot Observe

The supervisor sees the agent's inputs, outputs, and tool call traces. It cannot observe what the agent "thinks" — the internal LLM reasoning steps are not exposed in the trace unless the agent is explicitly designed to log them. Implicit reasoning failures (where the agent produces a plausible-but-wrong output via an internally flawed reasoning chain) may pass both verification stages if neither the deterministic check nor the LLM critic is designed to catch that specific failure mode.

### Callback Architecture Adds Latency

The non-blocking worker-supervisor pattern introduces a round-trip: worker proposes → supervisor verifies → committer identity writes. For high-frequency, low-stakes operations this latency may be unacceptable. In those cases, inline self-verification ([Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md), 1.4.3) is the right choice, and the supervisor monitors telemetry in the background without being on the critical path.

### LLM Critic Quality Depends on Prompt Engineering

The LLM critic in Workflow A (Stage 2 verification) is only as good as its system prompt and the examples provided. A poorly designed critic prompt can systematically approve wrong outputs or systematically reject correct ones. The critic prompt is itself a quality artifact that requires its own test suite. This is not a theoretical concern — it is one of the first things that degrades silently over time as the worker agent's behavior or the underlying model version changes.

### LLM Critic Non-Determinism

LLM outputs are probabilistic. Given identical inputs, the critic can return different verdicts across invocations — particularly for outputs near the plausibility boundary where the correct verdict is genuinely ambiguous. The supervisor's routing is therefore not fully deterministic for borderline cases: an output may receive CONFIRM on one invocation and REJECT on another.

Mitigations:

-   Set temperature to 0 (or the lowest available setting) in the critic's `sapOrchestration` call. This reduces but does not eliminate variance.
-   Design the critic prompt to require explicit reasoning before the verdict field: "First state what you observe about the value, then output your verdict." Anchoring the critic in observable facts before it declares a verdict increases consistency.
-   The critic's own eval suite (see below) should test borderline cases across multiple runs — not just single-run pass/fail — to characterize variance at the decision boundary.

### Building the LLM Critic's Eval Suite

The critic prompt requires its own test suite, but constructing test cases is non-trivial: ground-truth verdicts require knowing whether an output is correct, which is exactly what the critic assesses for novel inputs.

Practical approaches:

-   *Audit trail retrospective* (available after first weeks of production): each record where a human overrode or approved an output provides a labeled example. Retrospectively score the critic against these to identify systematic false-negatives (wrong outputs approved) and false-positives (correct outputs rejected).
-   *Synthetic adversarial examples* (before production): manufacture outputs with known failure modes — unit confusion, magnitude errors, contextually implausible values — and verify the critic catches them. These test the false-negative rate at build time.
-   *Near-threshold boundary cases*: construct outputs that pass deterministic checks but are contextually wrong, and outputs that are plausible but slightly outside tolerance. These test resolution at the decision boundary where non-determinism matters most.

Track both false-negative rate and false-positive rate. A critic that is too aggressive reduces throughput; a critic that is too permissive defeats its purpose.

### The Support Relay Requires Contractual Alignment

Workflow C creates a data flow from the customer's tenant to SAP support systems. Even with sanitization, this may trigger obligations under the CA (Collaboration Agreement) or require explicit consent in the Cloud Contract + SAP AI Terms. The technical architecture supports data minimization; the legal alignment must happen separately before Workflow C is activated.


<a name="loio16a4089179844b558b5905e084e431fa__section_fde_implications"/>

## FDE Implications

**Discovery (Phases 3–4):**

-   The TO-BE process flow should explicitly mark where the supervisor intercepts the worker's output. This is a visible step in the process design, not a background implementation detail.
-   The supervisor's three functions should be scoped explicitly: does this FDE engagement require all three, or only Workflow A (inline quality relay)? Workflows B and C add complexity and governance overhead; they are not always warranted for the MVP.
-   Data protection requirements for Workflow C (support relay) should be identified during Discovery and included in the legal agreement planning. If the customer cannot consent to sanitized trace export, Workflow C must be excluded from scope.

**Develop (formerly Execute) (MVP Build):**

-   The supervisor workflows should be built in parallel with the worker agent, not after. The callback mechanism (worker → supervisor via `sapAgent` non-blocking) is a fundamental workflow design decision that cannot be retrofitted cleanly.
-   The shared state store should be provisioned as part of the BAIP environment setup (FDE Develop step 4: Environment Setup and MVP Build). The one-time **Cloud ALM LMS `serviceId` registration** (Landscape Management) that enables the Workflow B / Workflow C telemetry pull path (–) is a prerequisite of the same step — provision it before the Telemetry Monitor is wired up.
-   During Internal Testing and Checkpoint (FDE Develop step 5), the supervisor's behavior should be tested explicitly: does it correctly block invalid outputs, does it escalate via Task Center, does Workflow B fire anomalies on injected test data?

**Productization:**

-   The Sirius tasks for security validation include DPCE (Data Protection and Compliance Engineering) threat modelling and security concept documentation. The supervisor architecture — its agent identities, authorization policies, data flows, and sanitization rules — must be included in the security concept.
-   The production handover includes threshold configuration as a documented parameter set. The customer's IT team should understand every threshold and be able to adjust them via the Agent Hub governance interface.
-   The production handover must also confirm that the one-time **Cloud ALM LMS `serviceId` registration** (Landscape Management) for the telemetry pull path is in place, and record the Cloud ALM tenant's data-residency region — so the customer IT team knows the Workflow B/C telemetry path is provisioned and where that telemetry is stored.
-   The supervisor's own telemetry and the shared baseline state are part of the production system. They require the same backup, recovery, and support model as the worker agent.


<a name="loio16a4089179844b558b5905e084e431fa__section_summary"/>

## Summary

**Table: Supervisor Agent Pattern — Key Principles**


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

Architectural position

</td>
<td valign="top">

SAP's Supervision tier — between platform governance and individual agent transactions

</td>
</tr>
<tr>
<td valign="top">

Three functions

</td>
<td valign="top">

Inline quality relay (Workflow A), telemetry monitor (Workflow B), support relay (Workflow C)

</td>
</tr>
<tr>
<td valign="top">

Worker-supervisor communication

</td>
<td valign="top">

sapAgent non-blocking + Webhook callback; sessionId for correlation

</td>
</tr>
<tr>
<td valign="top">

Verification stages

</td>
<td valign="top">

Deterministic first (if available), LLM critic second; disagreement → escalate

</td>
</tr>
<tr>
<td valign="top">

Data protection

</td>
<td valign="top">

Business data and reasoning stay in tenant; agent telemetry is stored in SAP Cloud ALM (SAP-operated, region-bound); sanitized extracts only leave via Workflow C on confirmed anomalies

</td>
</tr>
<tr>
<td valign="top">

Anomaly classifier

</td>
<td valign="top">

Rule-based only — no LLM on raw trace data

</td>
</tr>
<tr>
<td valign="top">

Supervisor governance

</td>
<td valign="top">

Separate agent identity per workflow; least-privilege; fail-safe toward human review

</td>
</tr>
<tr>
<td valign="top">

Fail-safe default

</td>
<td valign="top">

If supervisor unavailable, worker routes all outputs to sapTaskCenter

</td>
</tr>
<tr>
<td valign="top">

Key limitation

</td>
<td valign="top">

Latency on critical path; LLM critic quality requires its own test suite

</td>
</tr>
<tr>
<td valign="top">

FDE integration

</td>
<td valign="top">

Supervisor scope decided in Discovery; built in parallel with worker; included in security concept

</td>
</tr>
</table>

**Related Information**  




[Code-Based Agents (Bring Your Own Agent)](https://help.sap.com/docs/JOULE/ba88d1ec6a1b442098863d577c19b0c0/2f9701e60fd24e948c2c74fe9e55ce23.html "How to register a custom code-based agent (BYOA) to Joule via A2A.") :arrow_upper_right:
[Transitioning from Customer-Managed Joule](https://help.sap.com/docs/JOULE_WORK/621104a90a01407fbb2d1613db39a54d/ed5033369d554472b601504b33d10e2a.html "Transition from CMJ to SAP-managed provisioning to access Joule Work and all current Joule innovations.") :arrow_upper_right:
[Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md)

[Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md)

[Security and Data Protection](../Security-and-Data-Protection/security-and-data-protection-90b7501.md)

[Production Operations](../Production-Operations/production-operations-edd9e33.md)

[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Overview of Joule Studio capabilities and navigation") :arrow_upper_right:

[SAP AI Agent Hub — Plan and Build](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/e2a29b0c422e4aceadee658f4bbab71b.html "SAP AI Agent Hub planning and build guidance") :arrow_upper_right:
