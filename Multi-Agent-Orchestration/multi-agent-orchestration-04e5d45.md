<!-- loio04e5d4514b1a41c6959f295d83ead275 -->

# Multi-Agent Orchestration

Multi-agent orchestration is the target state of SAP's Autonomous Enterprise vision. This topic explains when multi-agent patterns are genuinely warranted, how SAP's orchestration hierarchy works, which topology to choose, and how to extend quality and data protection design across agent graphs.



<a name="loio04e5d4514b1a41c6959f295d83ead275__section_why_multi_agent"/>

## Why Multi-Agent Patterns Emerge — and When They Are Actually Needed

Within a single FDE sprint, the typical deliverable is one or two specialist agents — the design patterns in Chapters 1–6 are built around this scope. But Industry AI domains as SAP envisions them are inherently multi-agent systems: the Autonomous Suite deploys 224 specialized agents coordinated by 51 Joule Assistants across five domains, and Joule Work is explicitly designed to orchestrate agents — plural — in response to a single user intent. Multi-agent orchestration is the target state of the Autonomous Enterprise, not an edge case.

Three Industry AI domain examples show how this structure emerges naturally within FDE sprint scope, before the full domain vision is even reached:

-   **Autonomous Asset Management**: an orchestrator coordinates a fault-diagnosis agent, a parts-availability agent, and a maintenance-scheduling agent. Each needs different SAP data access (PM module, MM module, scheduling services) and different authorization scopes. Running all three as a single agent creates an authorization scope that is too broad and a workflow that is too complex to test reliably.
-   **Autonomous Revenue Growth Management**: a pricing decision involves a base-pricing agent (accesses pricing engine), a customer-segment agent (accesses CRM), and a contract-terms agent (accesses contract management). Each sub-task has a distinct reference source and distinct validation logic.
-   **Autonomous Adaptive Production**: a production-order decision coordinates a capacity agent, a materials-availability agent, and a scheduling-constraints agent — three separate SAP planning domains, each with a different system of record.

In each case, the multi-agent structure is justified by two factors that a single agent cannot satisfy simultaneously: **genuinely different authorization scopes** for different sub-tasks, and **independently verifiable outputs** from each sub-task before they are combined.

### When NOT to Use Multi-Agent

Multi-agent adds latency (inter-agent communication overhead), complexity (more workflows to build, test, and govern), and more failure points. It is the right choice only when the alternatives are genuinely insufficient:

**Table: Multi-Agent Decision Guide**


<table>
<tr>
<th valign="top">

Situation

</th>
<th valign="top">

Recommendation

</th>
</tr>
<tr>
<td valign="top">

Complex single-domain workflow with many tool calls

</td>
<td valign="top">

Single agent with well-structured prompts and self-verification

</td>
</tr>
<tr>
<td valign="top">

Different data retrieval steps, same authorization scope

</td>
<td valign="top">

Single agent with multiple sapMcpClient tools

</td>
</tr>
<tr>
<td valign="top">

Sequential steps where step B always follows step A

</td>
<td valign="top">

Single workflow with explicit nodes; no multi-agent needed

</td>
</tr>
<tr>
<td valign="top">

Parallel sub-tasks requiring different authorization scopes

</td>
<td valign="top">

Multi-agent: each worker holds only the scope it needs

</td>
</tr>
<tr>
<td valign="top">

Sub-task outputs need independent quality verification before aggregation

</td>
<td valign="top">

Multi-agent: each worker has its own verification layer

</td>
</tr>
<tr>
<td valign="top">

Sub-tasks use genuinely different models (e.g., tabular vs. reasoning)

</td>
<td valign="top">

Multi-agent: model selection per worker without conflict

</td>
</tr>
</table>

**Rule of thumb**: if a thoughtful human reviewer could do the whole task with one coherent set of authorizations and one line of reasoning, a single agent with multiple tools can probably do the same. If the task naturally decomposes into roles that a human team would assign to different specialists with different system access, multi-agent is warranted.


<a name="loio04e5d4514b1a41c6959f295d83ead275__section_sap_orchestration_model"/>

## SAP's Orchestration Model

SAP's AI-Native North Star Architecture defines a clear orchestration hierarchy:

> **Joule orchestrates** → **domain agents decompose** → **specialist agents execute**

In practice for FDE engagements:

-   **Joule** receives the user's high-level goal and invokes a domain orchestrator agent
-   **The domain orchestrator** decomposes the goal into sub-tasks and delegates each to a specialist worker agent via `sapAgent`
-   **Each specialist worker** executes its sub-task using its scoped tools and authorization, produces an output, and returns it to the orchestrator
-   **The orchestrator** aggregates the worker outputs, applies cross-worker consistency verification, and either commits the combined result or escalates to human review

This is an extension of the `sapAgent` node's capabilities already described in the supervisor agent pattern topic. In the supervisor pattern, `sapAgent` is used for quality control (worker sends output to supervisor for verification). In multi-agent orchestration, `sapAgent` is used for task delegation (orchestrator sends sub-task to specialist worker). The mechanism is the same; the architectural intent is different.

**The orchestrator is itself an agent registered on the BAIP (Business AI Platform) managed runtime**, with its own identity, authorization scope, and telemetry. It is not a meta-level controller outside the governed system — it is a first-class BAIP principal like any other agent.

### ORD Registration: Normative Requirement for Multi-Agent Topologies

ORD (Open Resource Discovery) registration via UMS is **normatively required** — not merely recommended — for agent discoverability and A2A routing. Agents not ORD-registered with the `sap:ai:v1` policy level and `sap.agt*` namespace are invisible to Agent Gateway. Without Agent Gateway visibility, agent-to-agent calls and tool calls cannot be routed correctly.

In a multi-agent topology, this constraint is load-bearing: if any worker agent is not ORD-registered, the orchestrator's `sapAgent` call to that worker will fail at routing. ORD registration must be completed before any agent is tested in a multi-agent topology — it is not a post-build step. The Week 0 scope checklist must confirm that all agents in the topology will have ORD registrations in place before integration testing begins.

**ORD registration — process guidance for FDE teams:**

**Table: ORD Registration Process**


<table>
<tr>
<th valign="top">

Step

</th>
<th valign="top">

Detail

</th>
</tr>
<tr>
<td valign="top">

**Who initiates**

</td>
<td valign="top">

FDE AI Architect files the ORD registration request; the BAIP/Agent Hub team or SAP platform operator processes it

</td>
</tr>
<tr>
<td valign="top">

**When to initiate**

</td>
<td valign="top">

In Discovery — confirm before the Development sprint starts; lead time can be 3–10 business days depending on the tenant configuration

</td>
</tr>
<tr>
<td valign="top">

**Minimum information required**

</td>
<td valign="top">

Agent name, `sap.agt*` namespace identifier, `sap:ai:v1` policy level, BAIP application registration ID, SAP BTP subaccount

</td>
</tr>
<tr>
<td valign="top">

**Contact**

</td>
<td valign="top">

Raise via the standard BAIP platform request channel; if the tenant is customer-managed, coordinate with the customer IT admin for the registration

</td>
</tr>
<tr>
<td valign="top">

**If registration is delayed**

</td>
<td valign="top">

Integration testing with multi-agent routing can proceed in a SAP-managed test tenant while customer tenant registration is pending — do not defer integration testing until the production tenant is ready

</td>
</tr>
</table>

Confirm ORD registration completion as a pre-Development gate item among the Discovery deliverables (see).


<a name="loio04e5d4514b1a41c6959f295d83ead275__section_topology_options"/>

## Three Topology Options

### Topology 1 — Orchestrator → N Workers (Flat)

The simplest multi-agent structure. The orchestrator decomposes the task, invokes N workers in parallel (or sequence), and aggregates results.

```
[Orchestrator Agent]
  ├── sapAgent(blocking=false) → Worker A (authorization scope: PM module)
  ├── sapAgent(blocking=false) → Worker B (authorization scope: MM module)
  └── sapAgent(blocking=false) → Worker C (authorization scope: scheduling)
           ↓ (all three callbacks arrive; Merge node collects)
[Orchestrator: aggregate + cross-worker consistency check]
           ↓
[sapTaskCenter / sapMcpClient write-back]
```

**Verification responsibility**: the orchestrator holds verification for the aggregated output. Individual worker outputs are treated as intermediate results; only the combined output goes through the full guardrail stack.

**When to use**: when individual worker outputs are only meaningful in combination and not independently committed to any system of record. A fault-diagnosis agent's output has no value without the parts-availability result — both are needed before any decision is made.

### Topology 2 — Orchestrator + Independent Supervisor per Worker

Each worker has its own supervisor (see [The Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md)). The orchestrator receives only supervisor-approved outputs.

```
[Orchestrator Agent]
  ├── sapAgent → Worker A → [Supervisor A: Workflow A inline quality relay]
  │                                  ↓ (approved or escalated)
  ├── sapAgent → Worker B → [Supervisor B: Workflow A inline quality relay]
  │                                  ↓ (approved or escalated)
  └── sapAgent → Worker C → [Supervisor C: Workflow A inline quality relay]
                                      ↓
[Orchestrator receives supervisor-approved outputs only]
           ↓
[Aggregation verification + human approval if needed]
```

**Verification responsibility**: distributed. Each supervisor verifies its worker's output independently. The orchestrator applies an additional cross-worker consistency check on the approved set.

**When to use**: when individual worker outputs are each independently high-stakes and could be committed to different systems of record. A pricing recommendation and a discount authorization are both independently meaningful; each warrants its own quality gate before being fed into the combined recommendation.

**FDE complexity**: this topology requires building and governing three separate supervisor workflow sets (A, B, C) plus the orchestrator. Budget accordingly — this is a full-stack multi-agent deployment, not a simple extension.

### Topology 3 — Orchestrator + Shared Supervisor

A single supervisor governs all workers. It receives callbacks from all workers, applies per-worker verification, and releases the complete approved set to the orchestrator.

```
[Orchestrator Agent]
  ├── sapAgent(blocking=false, callbackUrl=shared_supervisor) → Worker A
  ├── sapAgent(blocking=false, callbackUrl=shared_supervisor) → Worker B
  └── sapAgent(blocking=false, callbackUrl=shared_supervisor) → Worker C
           ↓ (callbacks arrive at shared Supervisor Workflow A)
[Shared Supervisor: verifies each worker output; tracks pending set via sessionId]
  - When all workers in a set are verified → releases combined output to orchestrator
  - If any worker is rejected → pauses full set; escalates to sapTaskCenter
```

**Verification responsibility**: centralized. The shared supervisor has visibility across all workers and can detect cross-worker inconsistencies that no individual per-worker supervisor can see.

**When to use**: when the primary verification concern is the combined output, not individual worker outputs. The shared supervisor can check cross-worker consistency (e.g., "does the materials availability figure support the production order quantity?") in a way that per-worker supervisors cannot.

**Implementation note**: the shared supervisor uses `sessionId` to group callbacks from the same orchestrator invocation. A new Pending Set table in the shared state store tracks which workers in each set have been verified, and releases the orchestrator's callback only when all are complete or when a rejection triggers escalation.

**Workers producing different output types**: one Workflow A skeleton per output type rather than per agent is recommended. In Topology 3, if workers produce different output types (e.g., Worker A produces a batch release quantity and Worker B produces a maintenance cost estimate), the shared supervisor must handle both. Two approaches:

-   *Conditional routing within a single Workflow A*: an If/Switch node on `output_type` (included in the callback payload) routes each callback to the appropriate verification branch within the same workflow. Simpler to operate; recommended when there are 2–3 output types with similar verification complexity.
-   *Multiple Workflow A instances with a dispatcher webhook*: a lightweight router receives all callbacks and forwards each to the correct Workflow A instance by output type. Each instance remains focused; preferable when output types have significantly different verification complexity or different governance owners.

In both cases, the worker must include an `output_type` field in its callback payload so the supervisor can route correctly.

### Pending Set Failure Modes

The Pending Set table tracks which worker callbacks have arrived for a given orchestrator invocation. Three failure modes must be explicitly handled:

**Table: Pending Set Failure Modes**


<table>
<tr>
<th valign="top">

Failure Mode

</th>
<th valign="top">

How to Detect

</th>
<th valign="top">

Required Response

</th>
</tr>
<tr>
<td valign="top">

**Callback timeout** — a worker does not return within the expected window

</td>
<td valign="top">

Workflow B scheduled check queries the Pending Set for entries where `timeout_at` has passed and `status = 'pending'`

</td>
<td valign="top">

Escalate to `sapTaskCenter` with a "worker timeout" reason code; include the partial results from workers that did return. Human approver decides: proceed with partial results, retry the timed-out worker, or abort the orchestration. Do NOT silently hold the orchestrator indefinitely.

</td>
</tr>
<tr>
<td valign="top">

**Callback with error payload** — worker returns but signals a failure

</td>
<td valign="top">

Worker payload includes `status: 'error'` or an error code in the callback body

</td>
<td valign="top">

Classify as rejection (same path as supervisor rejection): pause the full worker set; escalate to `sapTaskCenter` with the error details; do not commit any partial results until a human approver resolves the error

</td>
</tr>
<tr>
<td valign="top">

**Orphan cleanup** — a Pending Set entry is never resolved (e.g., all workers returned but supervisor never released the orchestrator callback due to a mid-flight failure)

</td>
<td valign="top">

Workflow B scheduled check queries for Pending Set entries past their `timeout_at` with `status != 'complete'`

</td>
<td valign="top">

Treat as orphan: log with the orchestration's `sessionId`; escalate to `sapTaskCenter` with an "orphan cleanup" reason; notify CIS T&O. Do not delete the Pending Set entry until a human operator has reviewed and resolved it

</td>
</tr>
</table>

**Timeout threshold configuration**: the `timeout_at` field in the Pending Set is populated by the orchestrator at dispatch time. The timeout threshold should be set to the worst-case worker execution time plus a safety margin (typically 2× the p95 worker latency observed during EAC). This is a `$vars` configuration parameter — calibrate it from EAC baseline data, not from a design assumption.

The Workflow B scheduled check that drives orphan detection should run every 5–15 minutes in production. This is a separate trigger from the anomaly detection rules in the supervisor pattern — it targets incomplete workflow states, not behavioral anomalies.

### Choosing a Topology

**Table: Topology Selection Guide**


<table>
<tr>
<th valign="top">

Consideration

</th>
<th valign="top">

Topology 1

</th>
<th valign="top">

Topology 2

</th>
<th valign="top">

Topology 3

</th>
</tr>
<tr>
<td valign="top">

Individual worker outputs independently committed

</td>
<td valign="top">

No

</td>
<td valign="top">

Yes

</td>
<td valign="top">

No

</td>
</tr>
<tr>
<td valign="top">

Cross-worker consistency is the primary concern

</td>
<td valign="top">

No

</td>
<td valign="top">

Partial

</td>
<td valign="top">

Yes

</td>
</tr>
<tr>
<td valign="top">

FDE build complexity

</td>
<td valign="top">

Low

</td>
<td valign="top">

High

</td>
<td valign="top">

Medium

</td>
</tr>
<tr>
<td valign="top">

Escalation granularity

</td>
<td valign="top">

Aggregate only

</td>
<td valign="top">

Per-worker + aggregate

</td>
<td valign="top">

Aggregate, with per-worker evidence

</td>
</tr>
<tr>
<td valign="top">

Supervisor build effort

</td>
<td valign="top">

None (orchestrator handles)

</td>
<td valign="top">

3× [Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md) supervisor stack

</td>
<td valign="top">

1× [Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md) supervisor stack (extended)

</td>
</tr>
</table>


<a name="loio04e5d4514b1a41c6959f295d83ead275__section_joule_studio_implementation"/>

## Joule Studio Implementation

### Orchestrator as a Deterministic Workflow

Where the decomposition logic is rule-based (always decompose this task type into these three sub-tasks), the orchestrator should be a deterministic Joule Studio n8n workflow, not an LLM-powered agent. Deterministic orchestration is:

-   Faster (no LLM call for decomposition)
-   More predictable (decomposition cannot vary at runtime)
-   Easier to test (decomposition logic is a Code node)
-   More auditable (every decomposition decision is logged without LLM reasoning traces)

```
[Trigger: incoming task request]
         ↓
[Code node: decompose task into sub-task parameters]
  Input: task_type, entity_id, context
  Output: [{worker: "A", params: {...}}, {worker: "B", params: {...}}, ...]
         ↓
[Loop Over Items: for each sub-task]
  [sapAgent: invoke worker, blocking=false, callbackUrl=supervisor_or_merge]
         ↓
[Merge node: wait for all callbacks (timeout: configured)]
         ↓
[Code node: cross-worker consistency check]
         ↓
[If: consistent and within thresholds?]
    ├── YES → sapMcpClient write-back + audit trail
    └── NO  → sapTaskCenter escalation with full context
```

### Orchestrator as an LLM-Powered Agent

When the decomposition logic is dynamic — the orchestrator must reason about how to decompose the task based on context — use the n8n LangChain AI Agent node with `sapAgentTool` variants as its available tools. The agent reasons, selects which specialist workers to invoke, and decides how to aggregate responses.

Use this only when the deterministic approach is genuinely insufficient. LLM-powered orchestration introduces the same quality, hallucination, and drift risks as any LLM step. An orchestrator that makes wrong decomposition decisions is harder to debug than an orchestrator that follows predictable rules.

### Parallel vs. Sequential Worker Invocation

Workers can be invoked in parallel (all fire simultaneously, orchestrator waits for all callbacks) or sequentially (Worker B cannot start until Worker A completes).

**Table: Worker Invocation Patterns**


<table>
<tr>
<th valign="top">

Pattern

</th>
<th valign="top">

When to Use

</th>
<th valign="top">

Implementation

</th>
</tr>
<tr>
<td valign="top">

Parallel

</td>
<td valign="top">

Workers are independent; no output from A is needed as input to B

</td>
<td valign="top">

Loop Over Items + sapAgent(blocking=false); Merge node collects all callbacks

</td>
</tr>
<tr>
<td valign="top">

Sequential

</td>
<td valign="top">

Worker B needs Worker A's output as part of its context

</td>
<td valign="top">

sapAgent(blocking=true) for A; pass A's output into B's customContext

</td>
</tr>
<tr>
<td valign="top">

Conditional

</td>
<td valign="top">

Whether to invoke Worker B depends on Worker A's output

</td>
<td valign="top">

If node on Worker A's result; conditional sapAgent call for Worker B

</td>
</tr>
</table>

**Joule 60-second constraint**: Joule expects a response from the outermost A2A-connected agent (the orchestrator) within **60 seconds** for synchronous calls. In parallel topologies, this window covers the full chain: Joule → orchestrator → workers → aggregation → response. For orchestrations that routinely exceed 60 seconds under realistic load (multiple parallel LLM-powered workers, external data fetches), use the asynchronous A2A pattern — the orchestrator responds immediately with a task ID and delivers the result via push notification when the orchestration completes. Do not assume the synchronous path is viable without measuring p99 latency under realistic parallel load conditions; retrofitting async behavior into a synchronous design is substantially harder than planning for it from the start.

**Latency note**: parallel invocation is the most efficient but requires a Merge node with a configured timeout. Set the timeout to 2× the p95 worker latency, calibrated from EAC baseline data — not arbitrarily. A timeout that fires before slow-but-correct workers complete will create false escalations.

**Declaring execution mode in the topology spec**: the `agent.spec.json` topology structure supports two explicit execution modes for multi-agent workflows. Declare execution mode explicitly in the spec — do not rely on runtime inference:

-   `"execution": "sequential"` — agents execute in order, each receiving the output of the previous. Use when agent B depends on agent A's result.
-   `"execution": "parallel"` — agents execute simultaneously and results are merged. Use for independent sub-tasks that can be dispatched in parallel.

The orchestration supervisor reads the declared execution mode from the spec to determine routing behaviour at runtime. A spec that omits execution mode requires manual runtime configuration and is treated as a spec integrity failure during the post-build eval scorecard (Category A; the 6-category A–F scorecard is defined in [Agent Evaluation in FDE Sprints](../Agent-Evaluation/agent-evaluation-76db9f7.md)).

### Recommended Orchestration Stack

For FDE-delivered production agents, the following stack is recommended. These choices align with the A2A Agent Toolkit scaffolding and the post-build eval scorecard requirements.

This is the **pro-code** orchestration path. The low-code n8n orchestrator is the default and preferred choice; reach for this pro-code stack only when the orchestration is genuinely too complex for the deterministic n8n workflow or the n8n LangChain agent to express cleanly — for example, stateful multi-turn coordination, intricate conditional routing, or persistent shared memory across invocations.

**Table: Recommended Pro-Code Orchestration Stack**


<table>
<tr>
<th valign="top">

Layer

</th>
<th valign="top">

Technology

</th>
<th valign="top">

Notes

</th>
</tr>
<tr>
<td valign="top">

Orchestration framework

</td>
<td valign="top">

LangGraph (Python) with Compounding Shared State

</td>
<td valign="top">

Provides native state persistence, conditional routing, and structured tool execution. Compounding Shared State enables persistent memory across conversation turns.

</td>
</tr>
<tr>
<td valign="top">

Inter-agent communication

</td>
<td valign="top">

A2A (Agent-to-Agent) protocol

</td>
<td valign="top">

Mandated by TR1 and the standard for all SAP-platform agent communication. Agents expose A2A endpoints from day one to enable eval harness integration and inter-agent routing without rework.

</td>
</tr>
<tr>
<td valign="top">

Agent scaffolding

</td>
<td valign="top">

A2A Agent Toolkit

</td>
<td valign="top">

Available in `context/fde/project-lightspeed/toolkits/a2a-agent-toolkit/`. Provides TypeScript (CAP + Express) and Python (FastAPI + LangGraph) scaffolding with the correct four-file spine: `agent.py`, `agent_executor.py`, `app.py`, `__main__.py`.

</td>
</tr>
</table>

**Why A2A from day one**: agents that initially expose only a proprietary interface require rework before they can participate in multi-agent topologies or be tested by the eval harness. Scaffolding with A2A endpoints from the start avoids this rework and ensures the agent is discoverable via Agent Gateway from the first deployment.

**LangGraph + Compounding Shared State**: Compounding Shared State is the recommended memory pattern for agents that need to maintain context across conversation turns or across multiple orchestrator invocations. It persists accumulated state as a structured object rather than a raw conversation history, which is more token-efficient and more amenable to deterministic inspection and audit.

**TypeScript path**: for FDE teams building on the CAP + TypeScript stack (SwissKnife reference architecture), Project LightSpeed provides two standard pipeline outputs: `multi_agent_topology.py` (Python LangGraph topology scaffold) and `multi_agent_overlay_ts.py` (TypeScript A2A overlay). Both are standard outputs of the LightSpeed post-build stage and provide the correct four-file spine for the chosen language. For TypeScript-first builds, use the A2A Agent Toolkit's CAP + Express scaffold from `toolkits/a2a-agent-toolkit/` — it produces an A2A-compliant TypeScript agent with the same structural contract as the Python path.

### Multi-Agent Joule Surface — The Hub Pattern

When exposing multiple agents as a unified Joule experience, use the hub pattern. Without it, each agent appears as a separate Joule Assistant, requiring users to navigate between them manually.

**Structure:**

-   A single `_joule-hub/da.sapdas.yaml` bundles all specialist agents into one Digital Assistant bundle visible in Joule.
-   Each agent has its own `capability.sapdas.yaml` with a routing description that must be **distinct** — Joule uses these descriptions to route user intent to the correct agent.

**Routing description rules** (enforced by post-build eval scorecard, Category D):

**Table: Routing Description Rules**


<table>
<tr>
<th valign="top">

Rule

</th>
<th valign="top">

Rationale

</th>
</tr>
<tr>
<td valign="top">

No personal pronouns

</td>
<td valign="top">

Routing descriptions are written from Joule's perspective, not the agent's

</td>
</tr>
<tr>
<td valign="top">

Use domain nouns specific to the agent's scope

</td>
<td valign="top">

Generic descriptions ("handles requests") cause misrouting

</td>
</tr>
<tr>
<td valign="top">

Include explicit negative boundary statements

</td>
<td valign="top">

e.g., "does NOT handle procurement requests" — prevents intent bleed between agents

</td>
</tr>
<tr>
<td valign="top">

Must be distinct from all other agents in the hub

</td>
<td valign="top">

Joule cannot route correctly if two agents have overlapping descriptions

</td>
</tr>
</table>

**Example**: an Autonomous Asset Management hub with three agents:

-   Fault Diagnosis Agent: `"Routes fault diagnosis requests for production equipment in PM module. Does NOT handle parts ordering or scheduling."`
-   Parts Availability Agent: `"Routes material availability queries against MM module stock. Does NOT handle fault analysis or work order scheduling."`
-   Maintenance Scheduling Agent: `"Routes maintenance work order scheduling requests. Does NOT handle fault diagnosis or material procurement."`

The post-build eval scorecard (Category D) validates routing description quality and distinctness before deployment. A failing score in Category D blocks deployment — not merely advisory.

> ### Note:  
> **Delivery path — updated November 2026.** Delivery decomposes into two *orthogonal* axes — do not bundle them as a single "CMJ/BYOA" choice:
> 
> **Orchestrator / Joule activation (Axis 1): SMJ always** for new engagements. SAP-Managed Joule ("THE BOX", GA 2026-06-01) is the default and the only activation path receiving new features. **CMJ is legacy/deprecated** — a migration target, not a delivery choice (see).
> 
> **Custom-agent runtime (Axis 2):** one of two standard runtimes, both registering to the SMJ orchestrator via A2A — neither is a fallback: *Joule Studio 2.0 (low-code, SAP-managed)*: build multiple agents in Joule Studio 2.0; each integrates into Joule Work automatically, with intent-based routing handled by the Joule platform. *BYOA (pro-code, customer BTP subaccount)*: bundle multiple capabilities in one `_joule-hub/da.sapdas.yaml`; each agent runs on BTP with its own A2A endpoint and capability bundle and registers to SMJ via A2A. This is the **standard** runtime for productization-intent FDE agents — not a fallback.
> 
> A distinct, related pattern — the **multi-agent front-door/proxy**, where a single Joule capability targets a proxy agent that then fans out to specialist agents — was planned for SAP-Managed Joule (SMJ) v3.0.0. With JS2.0 now GA, verify the current availability of this capability before promising it to a customer. Do not commit to a front-door/proxy architecture for an engagement without explicit confirmation that it is available in the customer's BAIP tenant.
> 
> **Inbound direction — GA as of October 2026.** The scenarios above are outbound (Joule calls your agent). The reverse — an external AI client or orchestrator calling into SAP Agents — is now generally available via the **Agent Gateway**. The integration pattern: (1) Customer admin registers the external AI platform as an **Agent Gateway Consumer** via SAP for Me. (2) Three sequential **SAP-managed formations** establish identity, trust, and access control: *Identity* (OIDC agent identity via Cloud Identity Services) → *Trust* (between agent identity and Agent Gateway) → *Authorization* (ACL specifying which SAP Agents the consumer may invoke). (3) At runtime, the Agent Gateway validates the token, resolves the customer tenant, checks the ACL, and routes the A2A call to the target SAP Agent.
> 
> For FDE engagements involving heterogeneous multi-agent architectures where a non-SAP orchestrator needs to call into SAP domain agents, this is now a current GA integration path rather than a roadmap item.


<a name="loio04e5d4514b1a41c6959f295d83ead275__section_quality_framework_extension"/>

## Quality Framework Extension for Multi-Agent Graphs

The three-dimension quality framework ([Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md), ) extends to multi-agent with additional considerations at each level.

### Per-Worker Quality

Each worker agent retains its own self-verification ([Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md) –1.4.4) and, in Topology 2/3, its own supervisor quality relay. The per-worker quality design is identical to the single-agent case.

### Cross-Worker Consistency Verification

A new quality dimension that only exists in multi-agent contexts: do the outputs from different workers form a coherent, non-contradictory whole?

Cross-worker consistency checks are always **deterministic** (Code node), not LLM-based. Examples:

**Table: Cross-Worker Consistency Checks**


<table>
<tr>
<th valign="top">

Check

</th>
<th valign="top">

Implementation

</th>
</tr>
<tr>
<td valign="top">

Quantity cross-consistency

</td>
<td valign="top">

Worker A: demand quantity 1,000 units. Worker B: available stock 800 units. Check: order ≤ available stock or escalate

</td>
</tr>
<tr>
<td valign="top">

Unit consistency across workers

</td>
<td valign="top">

Worker A output in EUR, Worker B output in EUR/unit. Check: units are compatible

</td>
</tr>
<tr>
<td valign="top">

Timeline consistency

</td>
<td valign="top">

Worker A: recommended order date 2026-09-01. Worker C: earliest delivery date 2026-09-15. Check: order date < delivery date

</td>
</tr>
<tr>
<td valign="top">

Arithmetic cross-check

</td>
<td valign="top">

total cost = unit_price × quantity (from different workers). Verify within tolerance

</td>
</tr>
</table>

These checks are implemented in the Code node that follows the Merge node in the orchestrator workflow.

### Aggregate Output Verification

The orchestrator's final combined output goes through the full guardrail stack from [Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md) (precision guardrails) if it produces a numerical decision committed to a system of record. The orchestrator is the point at which the four-layer stack applies — not individual worker outputs (unless those are also independently committed).

### Escalation Handling in Multi-Agent Graphs

Partial failure requires explicit design. Options:

**Table: Escalation Handling for Partial Failures**


<table>
<tr>
<th valign="top">

Scenario

</th>
<th valign="top">

Recommended Handling

</th>
</tr>
<tr>
<td valign="top">

One worker is rejected; others confirmed

</td>
<td valign="top">

Pause full set; escalate all to sapTaskCenter with clear indication of which worker failed

</td>
</tr>
<tr>
<td valign="top">

One worker times out

</td>
<td valign="top">

Treat as rejection; do not commit any worker's output; escalate with timeout evidence

</td>
</tr>
<tr>
<td valign="top">

Workers produce inconsistent outputs

</td>
<td valign="top">

Escalate all to sapTaskCenter; present all outputs with the specific inconsistency flagged

</td>
</tr>
<tr>
<td valign="top">

All workers confirmed but orchestrator's aggregate fails Layer 2

</td>
<td valign="top">

Escalate aggregate to sapTaskCenter; per-worker approvals do not imply aggregate approval

</td>
</tr>
</table>

The key principle: **partial approval is not approval**. A multi-agent write-back must be treated as an atomic operation — either all sub-task outputs are confirmed and the combined result passes verification, or the full operation is escalated. Committing Worker A's output while Worker B's is pending creates inconsistent state in the system of record.


<a name="loio04e5d4514b1a41c6959f295d83ead275__section_data_protection_multi_agent"/>

## Data Protection in Multi-Agent Graphs

### Agent-to-Agent Data Flows Stay Within the Tenant

All `sapAgent` inter-agent communication flows through the BAIP managed runtime within the customer's tenant. Data passed via `customContext` from orchestrator to workers, and via callback payloads from workers back to orchestrator, is intra-tenant by design. This is the same property as the worker-supervisor communication described in [The Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md).

### Authorization Scopes Are Not Inherited

This is the most important data protection principle in multi-agent architectures:

> **An orchestrator cannot grant a worker agent permissions the orchestrator itself holds, and a worker cannot acquire permissions it does not need for its specific task.**

Each worker agent has its own BAIP application registration with its own scoped authorization. The orchestrator's authorization scope covers coordination and aggregation, not the data access that each worker performs. If Worker A needs read access to MM module entities and the orchestrator does not, the orchestrator cannot retrieve those entities and pass them to Worker A's context — Worker A must retrieve them itself using its own authorized tools.

**What the Agent Gateway platform-guarantees:**

-   **Mandatory intersection rule**: effective permission = `min(user permissions, agent permissions)` — enforced at the gateway runtime, not advisory. An agent can never exceed what the calling user is entitled to do.
-   **Authentication at every hop**: IAS App2App tokens with named user context; no anonymous tool calls between agents.
-   **Second enforcement point**: user context propagation carries the user's identity to backend systems (S/4HANA, etc.), which enforce PFCG/CDS/OData authorization independently — regardless of what the gateway permitted.

**Where design discipline owns the risk:**

-   **Capability authorization conditions**: SAP Cloud Identity Services enforces functional policy at the tool/MCP call level — but only if the capability developer registered the right authorization conditions. A tool published with no restrictions gives the gateway nothing to enforce beyond agent identity.
-   **Custom and BYOA tools**: custom MCP servers and BYOA agents via BTP Destinations must validate the `ias_apis` claim in the inbound token themselves. A custom tool that skips this check is accessible to any authenticated agent despite the gateway.
-   **Non-SAP connectors**: if a connector uses a technical service account rather than principal propagation, the backend authorization check does not apply. Always verify that principal propagation is active on every backend connector.

**The `customContext` data-passing caveat**: the gateway intersection rule governs *API calls*, not *data passing*. If an orchestrator retrieves authorized data and passes it to a worker via `customContext`, the worker gains access to that data even if it could not retrieve it directly through its own tools. This is not prevented by the platform. The design rule:

> Orchestrators must apply data minimization — pass to each downstream worker only the fields that worker needs for its specific task, not the full output from prior workers or the orchestrator's own data access.

### PII in Inter-Agent Messages

If Worker A produces output that contains PII (Personally Identifiable Information) and passes it to the orchestrator, the orchestrator must apply PII masking before including that data in Worker B's context. PII masking at the `sapOrchestration` node level ([Security and Data Protection](../Security-and-Data-Protection/security-and-data-protection-90b7501.md) ) only masks data before LLM calls — it does not automatically mask data in `customContext` payloads passed between agents.

**Design rule**: orchestrators that aggregate outputs from multiple workers must apply data minimization — pass to each downstream worker only the fields that worker needs, not the full output from prior workers.

### Supervisor Workflow C in Multi-Agent Contexts

Workflow C (support relay, described in [The Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md)) operates at the tenant level, not per-worker. In a multi-agent deployment, a single Workflow C can service anomalies from any worker or the orchestrator. The sanitization rules apply to all traces equally; the `agent_identity` field in the sanitized extract identifies which worker or orchestrator generated the anomaly.


<a name="loio04e5d4514b1a41c6959f295d83ead275__section_supervisor_pattern_extension"/>

## Supervisor Pattern Extension for Multi-Agent

### Workflow B (Telemetry Monitor) in Multi-Agent Context

Workflow B monitors the SAP Cloud ALM pull APIs (via the `hm` provider) for the customer's tenant. In a multi-agent deployment, the same Workflow B can cover all agents in the tenant — the worker A traces, worker B traces, orchestrator traces, and supervisor traces are all in the same tenant's log stream. The Code node anomaly classifier applies the same rule set to each, with the `agent_identity` field used to identify which agent triggered each anomaly.

**Extended anomaly signals for multi-agent:**

**Table: Extended Anomaly Signals for Multi-Agent Deployments**


<table>
<tr>
<th valign="top">

Signal

</th>
<th valign="top">

Threshold

</th>
<th valign="top">

Interpretation

</th>
</tr>
<tr>
<td valign="top">

Orchestrator timeout rate

</td>
<td valign="top">

> 10% of invocations over 24 hours

</td>
<td valign="top">

Workers are slow or failing; orchestrator is waiting too long

</td>
</tr>
<tr>
<td valign="top">

Partial completion rate

</td>
<td valign="top">

> 5% of orchestrator invocations result in partial worker set

</td>
<td valign="top">

One or more workers failing consistently

</td>
</tr>
<tr>
<td valign="top">

Cross-worker inconsistency rate

</td>
<td valign="top">

> 15% of multi-worker sets

</td>
<td valign="top">

Workers are diverging; may indicate model drift in one worker

</td>
</tr>
<tr>
<td valign="top">

Inter-agent communication latency

</td>
<td valign="top">

> 3× p95 baseline

</td>
<td valign="top">

BAIP runtime latency issue; check infrastructure

</td>
</tr>
</table>

### Shared Supervisor State Store (Topology 3)

In Topology 3, the shared supervisor's state store needs an additional Pending Set table:

**Table: Pending Set Table Schema**


<table>
<tr>
<th valign="top">

Field

</th>
<th valign="top">

Content

</th>
</tr>
<tr>
<td valign="top">

`orchestration_session_id`

</td>
<td valign="top">

The orchestrator invocation's session ID

</td>
</tr>
<tr>
<td valign="top">

`expected_workers`

</td>
<td valign="top">

Set of worker agent names expected in this invocation

</td>
</tr>
<tr>
<td valign="top">

`received_callbacks`

</td>
<td valign="top">

Set of worker names that have completed and been verified

</td>
</tr>
<tr>
<td valign="top">

`pending_callbacks`

</td>
<td valign="top">

Set of worker names not yet received

</td>
</tr>
<tr>
<td valign="top">

`any_rejected`

</td>
<td valign="top">

Boolean: true if any worker was rejected

</td>
</tr>
<tr>
<td valign="top">

`rejection_details`

</td>
<td valign="top">

Which worker was rejected and why

</td>
</tr>
<tr>
<td valign="top">

`timeout_at`

</td>
<td valign="top">

Timestamp after which the pending set is treated as failed

</td>
</tr>
</table>

The supervisor releases the orchestrator's callback only when `received_callbacks` equals `expected_workers` and `any_rejected` is false. If `timeout_at` is reached before all callbacks arrive, the supervisor marks the set as failed and escalates.


<a name="loio04e5d4514b1a41c6959f295d83ead275__section_fde_delivery_implications"/>

## FDE Delivery Implications

### During Discovery, Sub-Phases 3–4 (Define–Prioritize)

-   **Justify multi-agent explicitly.** The TO-BE process flow should document why multi-agent is warranted — which sub-tasks require genuinely different authorization scopes, which sub-task outputs need independent verification. If the justification is not clear in Discovery, a single well-designed agent is probably sufficient.
-   **Choose topology in Discovery.** The topology choice (flat / per-worker supervised / shared supervisor) has significant build complexity implications. It must be in the Draft Product Design Specs before the Development phase begins.
-   **Define the orchestrator's authorization scope separately** from each worker's scope. All five authorization matrices (orchestrator + 3 workers + supervisor) should be documented.

### During Develop (formerly Execute) — MVP Build

Build in strict order:

1.  **Workers first** — build, test, and eval each worker agent independently. A worker that is not independently stable will cause debugging confusion once the orchestrator is added.
2.  **Orchestrator second** — wire the workers together with the orchestrator workflow. Test decomposition, invocation, and aggregation with known worker outputs (mocked callbacks from step 1 test data).
3.  **Supervisor layer third** — add supervisor quality relay (per-worker or shared). Test the full stack end-to-end.
4.  **Full integration and UAT last** — test with the complete stack under realistic load conditions, including partial failure scenarios.

**Budget**: each additional worker agent beyond the first adds approximately 2–4 days of build + test + security documentation effort. A three-worker orchestration pattern with a shared supervisor adds approximately 8–12 days to a standard FDE Development phase.

### During Productization

-   Each agent (orchestrator + all workers + supervisor workflows) requires its own entry in the Sirius security concept documentation and its own BAIP application registration.
-   The Agent Hub registry should include the full multi-agent topology: which agents are workers, which is the orchestrator, what the expected inter-agent communication flows are.
-   The production handover package must include the orchestrator's decomposition logic and the aggregation verification rules as documented configuration, not just as workflow code.
-   **MCP server governance**: all MCP servers used by agents in the topology must be registered and verified in the AI Agent Hub's Agent & MCP Registry before production deployment. Customer IT admins can verify and govern MCP servers at runtime via the AI Agent Hub — this is a customer-side governance action, not automatic. Include the MCP server registry entries and the IT admin activation steps in the production handover checklist alongside the agent registry entries.


<a name="loio04e5d4514b1a41c6959f295d83ead275__section_summary"/>

## Summary

**Table: Multi-Agent Orchestration — Key Principles**


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

When to use

</td>
<td valign="top">

Parallel sub-tasks need different authorization scopes OR independent quality verification OR different models — not merely because the workflow is complex

</td>
</tr>
<tr>
<td valign="top">

Topology choice

</td>
<td valign="top">

Flat (orchestrator + workers, no per-worker supervisor) for lower stakes; per-worker supervised for high-stakes independent outputs; shared supervisor for combined-output quality

</td>
</tr>
<tr>
<td valign="top">

Orchestrator type

</td>
<td valign="top">

Deterministic workflow preferred; LLM-powered only when decomposition must be dynamic

</td>
</tr>
<tr>
<td valign="top">

Parallel invocation

</td>
<td valign="top">

sapAgent(blocking=false) + Merge node; set timeout to 2× p95 worker latency, calibrated in EAC

</td>
</tr>
<tr>
<td valign="top">

Cross-worker consistency

</td>
<td valign="top">

Always deterministic Code node checks; LLM not appropriate for cross-worker verification

</td>
</tr>
<tr>
<td valign="top">

Escalation

</td>
<td valign="top">

Partial approval is not approval; all workers or none — atomic commit design

</td>
</tr>
<tr>
<td valign="top">

Authorization

</td>
<td valign="top">

Workers cannot inherit orchestrator scope; each holds minimum permissions for its sub-task

</td>
</tr>
<tr>
<td valign="top">

PII in inter-agent messages

</td>
<td valign="top">

Apply masking before passing worker output into next worker's context

</td>
</tr>
<tr>
<td valign="top">

Supervisor

</td>
<td valign="top">

Workflow B covers all agents in tenant; Topology 3 needs Pending Set table in shared state store

</td>
</tr>
<tr>
<td valign="top">

FDE build order

</td>
<td valign="top">

Workers → orchestrator → supervisor → integration; never add orchestration to unstable workers

</td>
</tr>
<tr>
<td valign="top">

Build cost

</td>
<td valign="top">

2–4 days per additional worker; 8–12 days total for a 3-worker orchestration pattern

</td>
</tr>
<tr>
<td valign="top">

MCP server governance

</td>
<td valign="top">

All MCP servers in the topology must be registered and verified in AI Agent Hub before production; IT admin verification is a handover checklist item, not automatic

</td>
</tr>
</table>

**Related Information**  




[Code-Based Agents (Bring Your Own Agent)](https://help.sap.com/docs/JOULE/ba88d1ec6a1b442098863d577c19b0c0/2f9701e60fd24e948c2c74fe9e55ce23.html "How to register a custom code-based agent (BYOA) to Joule via A2A.") :arrow_upper_right:
[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "Introduction to SAP AI Agent Hub, its benefits, and core capabilities.") :arrow_upper_right:
[The Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md)

[Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md)

[Security and Data Protection](../Security-and-Data-Protection/security-and-data-protection-90b7501.md)

[Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md)

[Production Operations](../Production-Operations/production-operations-edd9e33.md)

[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Overview of Joule Studio capabilities and navigation") :arrow_upper_right:

[SAP Managed n8n Workflows](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/5f16a77e21d047dc968de9f7e83bca54.html "SAP Managed n8n Workflows documentation") :arrow_upper_right:

[SAP AI Agent Hub — Plan and Build](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/e2a29b0c422e4aceadee658f4bbab71b.html "SAP AI Agent Hub planning and build guidance") :arrow_upper_right:
