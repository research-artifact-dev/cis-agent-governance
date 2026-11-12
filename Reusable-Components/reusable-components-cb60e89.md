<!-- loiocb60e894e5814d6087247467f7106b39 -->

# CIS Delivery Efficiency — Reusable Components and Reference Implementations

CIS operates across multiple Industry AI FDE engagements simultaneously. This topic maps every design component to its reusability profile, defines the CIS Starter Kit, and describes the reference implementations that FDE teams inherit as a starting point for each engagement.



<a name="loiocb60e894e5814d6087247467f7106b39__section_why_reusability"/>

## Why Reusability Is a First-Class Design Concern

CIS operates across multiple Industry AI FDE engagements simultaneously, and the number grows as the Autonomous Enterprise vision scales. Without intentional reuse, each engagement rebuilds the same infrastructure from scratch: three supervisor workflows, an audit trail schema, environment migration checklists, a rollback procedure. The build effort is real; the testing effort is real; and a freshly assembled Workflow B has less reliability evidence behind it than one that has run in production across four previous engagements.

Reuse is not just about efficiency. It is also about quality accumulation: production incidents discovered in one engagement feed back into the shared reference implementation so that every subsequent engagement benefits. This is the operational expression of the continuous improvement loop described in [CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md) , applied not just to individual deployed agents but to the CIS delivery patterns themselves.

This topic maps every design component from Chapters 1–8 to its reusability profile: what stays the same across engagements, what requires per-engagement configuration, and what is genuinely engagement-specific. It then defines the **CIS Starter Kit** — the set of reference implementations that CIS T&O should maintain, version, and provide to every FDE team as a starting point.


<a name="loiocb60e894e5814d6087247467f7106b39__section_reusability_stack"/>

## The Reusability Stack

The concept components organize into three layers. The distinction matters because each layer has a different delivery model: the first is inherited, the second is configured, the third is built.

### Layer 1 — Deploy As-Is

Zero per-engagement content. These components are structurally and logically domain-agnostic: they operate on telemetry signals, schema definitions, process checklists, and design rules that do not change regardless of which Industry AI domain is being served or which worker agent is being supervised.

CIS T&O builds these once, maintains them as versioned reference artifacts, and provides them to FDE teams to inherit unchanged.

### Layer 2 — Parameterize and Deploy

Fixed skeleton; per-engagement content in defined slots. The workflow structure, routing logic, and data schemas are the same in every engagement. What varies are values (threshold calibrations, recipient IDs, connection credentials), prompts (LLM critic system prompts), and rules (the specific verification logic for this agent's output type).

CIS T&O maintains the skeleton. The FDE team fills the per-engagement slots during Discovery and Execute, and documents the resulting configuration in the production handover package.

### Layer 3 — Engagement-Specific

Cannot be lifted across engagements. This is the worker agent's business logic, the process KPIs that define what the agent is optimizing for, the verification rules specific to this output type, and the threshold values agreed with the process owner. These require genuine per-engagement design work; attempting to reuse them from a prior engagement introduces the wrong assumptions.

The total build scope of a new FDE engagement is the Layer 3 content plus the parameterization work for Layer 2. Layer 1 is not in scope for the FDE sprint.


<a name="loiocb60e894e5814d6087247467f7106b39__section_component_analysis"/>

## Component-by-Component Analysis

The following tables classify every significant design component from Chapters 1–8.

> ### Note:  
> **Layer denotes reusability profile, not current availability.** A component's Layer is its *reusability profile* — deploy-as-is vs. parameterize vs. engagement-specific — which is independent of whether a forkable reference artifact exists today. Several Layer 1/2 reference artifacts — Supervisor Workflows B and C, the shared-state-store DDL, and the APM archetype config — are **design targets** not yet published as standalone, forkable artifacts. The authoritative statement of what is available now is the status table in the CIS Starter Kit section; read every "Deploy As-Is" / "Fork and configure" entry below against it.

### Layer 1 — Deploy As-Is

**Layer 1 Components**


<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Chapter Ref

</th>
<th valign="top">

What Is Fixed

</th>
<th valign="top">

Parameterization Surface

</th>
<th valign="top">

Maintenance Owner

</th>
</tr>
<tr>
<td valign="top">

Supervisor Workflow C (Support Relay)

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md), [Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md)

</td>
<td valign="top">

Full workflow: sanitization rules, SAP Cloud ALM ITSM Adapter API call, IT admin notification

</td>
<td valign="top">

Customer tenant ID (from environment config, not hardcoded)

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Audit trail schema — base fields

</td>
<td valign="top">

[Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md)

</td>
<td valign="top">

All 16 fields: timestamp, agent_id, agent_version, correlation_id, input_state, agent_output, stage verdicts, human_decision, human_modified_value, committed_value, delta

</td>
<td valign="top">

Table name prefix per engagement

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Audit trail schema — cost extension

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md)

</td>
<td valign="top">

Four additional fields: input_tokens, output_tokens, total_tokens_invocation, model_used

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Telemetry anomaly rule set (Workflow B rule logic)

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md)

</td>
<td valign="top">

All six rule types: error rate, tool call count per invocation, response latency, human override rate, verification rejection rate, identical input pattern

</td>
<td valign="top">

None — rule types are domain-agnostic; threshold values are Layer 2

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Sanitization rules table (what to strip vs. retain in trace exports)

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md)

</td>
<td valign="top">

Field-by-field retain/strip classification

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Quality signal escalation structure (Level 0–3)

</td>
<td valign="top">

[Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md)

</td>
<td valign="top">

Four-level structure: automated → monitored → supervised → blocked; and the logic distinguishing each level

</td>
<td valign="top">

Specific threshold values and recipient IDs are Layer 2

</td>
<td valign="top">

CIS T&O (structure)

</td>
</tr>
<tr>
<td valign="top">

Version stack discipline (5 components)

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md)

</td>
<td valign="top">

Definition of the five-component version stack; change classification table (Low / Medium / High / Very High risk)

</td>
<td valign="top">

Applied per engagement to that engagement's specific component versions

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Supervisor fail-safe rule

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md)

</td>
<td valign="top">

If supervisor unavailable, worker routes all outputs to sapTaskCenter — never to autonomous write-back

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Three things never to copy between environments

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md)

</td>
<td valign="top">

Credentials, dev/test thresholds, agent identities — all must be re-provisioned per environment

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

When NOT to use multi-agent (decision table)

</td>
<td valign="top">

[Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md)

</td>
<td valign="top">

Full decision table with six rows; criteria for single agent vs. multi-agent

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Multi-agent topology selection table

</td>
<td valign="top">

[Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md)

</td>
<td valign="top">

Full comparison matrix: Topology 1 / 2 / 3 across five dimensions

</td>
<td valign="top">

Topology choice is per engagement

</td>
<td valign="top">

FDE team (guided by table)

</td>
</tr>
<tr>
<td valign="top">

Three pre-migration checklists (dev→test, test→EAC, EAC→prod)

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md)

</td>
<td valign="top">

All checklist items across three transitions

</td>
<td valign="top">

None — items are the same for every engagement

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Partial failure handling rules in multi-agent (atomic commit principle)

</td>
<td valign="top">

[Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md)

</td>
<td valign="top">

Rule: partial approval is not approval; all workers or none

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Prompt injection defense system prompt template

</td>
<td valign="top">

[Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md)

</td>
<td valign="top">

Boilerplate instruction text

</td>
<td valign="top">

Role description at the opening line

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

Pro-code decision table

</td>
<td valign="top">

[Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md)

</td>
<td valign="top">

Full table: which requirements are low-code-sufficient vs. require pro-code

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
</table>

### Layer 2 — Parameterize and Deploy

**Layer 2 Components**


<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Chapter Ref

</th>
<th valign="top">

Fixed Skeleton

</th>
<th valign="top">

Parameterization Surface

</th>
<th valign="top">

Maintenance Owner

</th>
</tr>
<tr>
<td valign="top">

Supervisor Workflow B (Telemetry Monitor) — full workflow

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md), [Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md), [Production Operations](../Production-Operations/production-operations-edd9e33.md)

</td>
<td valign="top">

Schedule trigger, HTTP Request to SAP Cloud ALM + AI Core usage API, Code node rule structure, routing logic, IT admin alert path, cost anomaly checks, Workflow C trigger

</td>
<td valign="top">

Anomaly threshold values (calibrated from EAC baseline); schedule interval; Task Center recipient IDs

</td>
<td valign="top">

CIS T&O (template); FDE team (calibration)

</td>
</tr>
<tr>
<td valign="top">

Supervisor Workflow A — routing skeleton

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md), [Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md)

</td>
<td valign="top">

Webhook trigger, If/Switch routing (BOTH CONFIRM / BOTH REJECT / DISAGREE / low confidence), sapAgent approval signal back to worker, Task Center escalation path, audit trail write

</td>
<td valign="top">

Code node verification rules (engagement-specific); LLM critic systemPrompt; approval threshold value

</td>
<td valign="top">

CIS T&O (skeleton); FDE team (Code node + critic prompt)

</td>
</tr>
<tr>
<td valign="top">

Supervisor shared state store schema

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md), [Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md)

</td>
<td valign="top">

Baseline table structure (rolling metrics per agent); Pending Set table structure for multi-agent Topology 3

</td>
<td valign="top">

Table name prefix per engagement; initial state is always empty per environment

</td>
<td valign="top">

CIS T&O (DDL template)

</td>
</tr>
<tr>
<td valign="top">

Four-layer guardrail stack structure

</td>
<td valign="top">

[Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md)

</td>
<td valign="top">

Layer sequence (Pre-Check → Output Validation → Human Approval → Audit Trail); which layers are mandatory vs. optional per output risk level

</td>
<td valign="top">

Layer 2 reference calculation service (which SAP service backs it); Layer 3 threshold values; Layer 1 constraint source

</td>
<td valign="top">

FDE team (configured with process owner for thresholds)

</td>
</tr>
<tr>
<td valign="top">

sapTaskCenter customAttributes pattern

</td>
<td valign="top">

[Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md), [Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md)

</td>
<td valign="top">

Six-attribute set: proposed value, reference value, deviation %, agent confidence, stage verdicts, correlation ID

</td>
<td valign="top">

Attribute display labels (e.g., "Batch quantity" vs. "Pricing recommendation"); recipient IDs; priority mapping

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

Worker agent base pattern (n8n workflow template)

</td>
<td valign="top">

[Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md)

</td>
<td valign="top">

sapOrchestration + sapMcpClient + verification node structure; routing to supervised (sapAgent callback) or unsupervised (inline)

</td>
<td valign="top">

Agent task definition in systemPrompt; SAP system connections; specific MCP tools wired

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

Three-dimension quality framework (evaluation design)

</td>
<td valign="top">

[Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md)

</td>
<td valign="top">

Framework structure (Technical / Process KPI / Business Outcome); Dimension 1 metric types and their Goodhart's Law guard

</td>
<td valign="top">

Dimension 2 process KPIs (engagement-specific, defined in TO-BE process flow); Dimension 3 business outcome metrics

</td>
<td valign="top">

FDE team (with process owner for Dimensions 2–3)

</td>
</tr>
<tr>
<td valign="top">

Eval test suite structure

</td>
<td valign="top">

[Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md)

</td>
<td valign="top">

Scenario type categories (high-freq/low-risk, low-freq/high-risk, exception, novel); acceptance threshold approach per category

</td>
<td valign="top">

Specific test cases; acceptance threshold values; test data strategy

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

Confidence-based routing pattern

</td>
<td valign="top">

[Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md)

</td>
<td valign="top">

Pattern: confidence field in sapOrchestration outputSchema; If node routing on confidence threshold

</td>
<td valign="top">

Confidence threshold value

</td>
<td valign="top">

FDE team (with process owner)

</td>
</tr>
<tr>
<td valign="top">

Cost monitoring extension (Workflow B)

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md)

</td>
<td valign="top">

Token anomaly check logic; runaway detection (5× p99 threshold); daily budget alert; cost-per-committed-output trend logic

</td>
<td valign="top">

Daily budget value; runaway spike threshold; alert routing

</td>
<td valign="top">

CIS T&O (template); customer IT admin (budget values)

</td>
</tr>
<tr>
<td valign="top">

Environment-agnostic configuration pattern

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md)

</td>
<td valign="top">

Naming convention: `{{agent-name.parameter-category.parameter-name}}`; separation of BAIP credential objects vs. configuration parameters

</td>
<td valign="top">

Actual parameter names and values per engagement

</td>
<td valign="top">

CIS T&O (convention); FDE team (values)

</td>
</tr>
<tr>
<td valign="top">

Rollback procedure template

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md)

</td>
<td valign="top">

Trigger criteria (human override rate, rejection rate, KPI degradation, format failures); rollback order (threshold → model → prompt → workflow); state continuity rules (audit trail immutable, baseline reset)

</td>
<td valign="top">

Authorization path per topology; specific component versions being rolled back

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Cross-worker consistency check pattern types

</td>
<td valign="top">

[Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md)

</td>
<td valign="top">

Check categories: quantity cross-consistency, unit consistency, timeline consistency, arithmetic cross-check

</td>
<td valign="top">

Specific field names, quantities, and tolerance values for this multi-agent set

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

Severity tier table and operating rhythm

</td>
<td valign="top">

[CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md)

</td>
<td valign="top">

Four severity tiers (Critical / High / Medium / Low) with response target structure

</td>
<td valign="top">

Actual SLA commitments (agreed per contract); T&O team contact details

</td>
<td valign="top">

CIS T&O (with contract alignment)

</td>
</tr>
<tr>
<td valign="top">

Continuous improvement loop

</td>
<td valign="top">

[CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md)

</td>
<td valign="top">

Loop structure: production findings → categorize → in-place fix vs. build improvement vs. new sprint; iteration pattern analysis signal types

</td>
<td valign="top">

Finding categories and escalation paths for this engagement

</td>
<td valign="top">

CIS T&O

</td>
</tr>
</table>

### Layer 3 — Engagement-Specific

**Layer 3 Components**


<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Chapter Ref

</th>
<th valign="top">

Why It Cannot Be Reused

</th>
<th valign="top">

Who Authors It

</th>
</tr>
<tr>
<td valign="top">

Worker agent business logic

</td>
<td valign="top">

[Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md)

</td>
<td valign="top">

The agent's task, SAP system connections, tools, and goal state are unique to this use case and customer

</td>
<td valign="top">

FDE Application Engineer + Domain PM

</td>
</tr>
<tr>
<td valign="top">

Process KPIs (Dimension 2 of quality framework)

</td>
<td valign="top">

[Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md)

</td>
<td valign="top">

Defined in the TO-BE process flow during Discovery; entirely domain- and customer-specific

</td>
<td valign="top">

FDE team + process owner

</td>
</tr>
<tr>
<td valign="top">

Workflow A Code node verification rules

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md), [Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md)

</td>
<td valign="top">

Specific range checks, unit checks, tolerance values apply only to this agent's output type

</td>
<td valign="top">

FDE Application Engineer

</td>
</tr>
<tr>
<td valign="top">

LLM critic systemPrompt

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md), [Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md)

</td>
<td valign="top">

What a "correct" output means is different for a batch release quantity vs. a pricing recommendation

</td>
<td valign="top">

FDE Application Engineer

</td>
</tr>
<tr>
<td valign="top">

Human approval threshold values

</td>
<td valign="top">

[Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md)

</td>
<td valign="top">

Business decision agreed with process owner in Discovery; not derivable from a template

</td>
<td valign="top">

FDE team + process owner

</td>
</tr>
<tr>
<td valign="top">

Authorization matrices (all agents)

</td>
<td valign="top">

[Security and Data Protection](../Security-and-Data-Protection/security-and-data-protection-90b7501.md), [Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md)

</td>
<td valign="top">

Scoped to specific SAP systems and entities this engagement accesses; differs per customer and per worker

</td>
<td valign="top">

FDE team + customer IT admin

</td>
</tr>
<tr>
<td valign="top">

Anomaly detection threshold values

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md)

</td>
<td valign="top">

Calibrated from EAC baseline observation specific to this agent's behavior; cannot be borrowed from another agent's deployment

</td>
<td valign="top">

CIS T&O (post-EAC calibration)

</td>
</tr>
<tr>
<td valign="top">

Regulatory audit trail retention period

</td>
<td valign="top">

[Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md)

</td>
<td valign="top">

Confirmed with customer's compliance team; jurisdiction- and industry-specific

</td>
<td valign="top">

Customer compliance team

</td>
</tr>
<tr>
<td valign="top">

Cross-worker consistency check logic

</td>
<td valign="top">

[Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md)

</td>
<td valign="top">

Specific field names, quantity relationships, and timeline logic for this multi-agent set

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

Business outcome metrics (Dimension 3)

</td>
<td valign="top">

[Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md)

</td>
<td valign="top">

Financial and operational results tied to this customer's specific value case

</td>
<td valign="top">

FDE team + Value Advisor

</td>
</tr>
<tr>
<td valign="top">

Eval test cases and test data

</td>
<td valign="top">

[Agent Quality Framework](../Agent-Quality-Framework/agent-quality-framework-4a9b908.md)

</td>
<td valign="top">

Test scenarios must cover this agent's boundary conditions and edge cases

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

LLM critic test suite

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md)

</td>
<td valign="top">

The critic prompt's quality is engagement-specific; its test suite must match its content

</td>
<td valign="top">

FDE team

</td>
</tr>
</table>


<a name="loiocb60e894e5814d6087247467f7106b39__section_cis_starter_kit"/>

## The CIS Starter Kit

The CIS Starter Kit is the versioned set of Layer 1 and Layer 2 artifacts that CIS T&O maintains and provides to every FDE engagement as a starting point. It is not a monolithic package — FDE teams take what applies to their engagement's scope and leave what does not. A lean engagement that does not require the full supervisor stack does not need to inherit Workflow B or C; a single-agent deployment without numerical write-back does not need the audit trail extension.

**Component Status**


<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Status

</th>
<th valign="top">

Available Where

</th>
</tr>
<tr>
<td valign="top">

**Workflow A skeleton** (`cis-supervisor-workflow-a-skeleton`)

</td>
<td valign="top">

PoC in progress

</td>
<td valign="top">

`concept/proof-of-concept-plan.md` — Workflow A PoC is the first build step

</td>
</tr>
<tr>
<td valign="top">

**Workflow B reference** (`cis-supervisor-workflow-b-ref`)

</td>
<td valign="top">

Design target

</td>
<td valign="top">

Not yet a standalone artifact; no supervisor workflow exists in the LightSpeed catalog either — build per [Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md) / [Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md) patterns

</td>
</tr>
<tr>
<td valign="top">

**Workflow C reference** (`cis-supervisor-workflow-c-ref`)

</td>
<td valign="top">

Design target

</td>
<td valign="top">

Not yet a standalone artifact; patterns from and [Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md)

</td>
</tr>
<tr>
<td valign="top">

**Worker agent base** (`cis-worker-agent-base`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

LightSpeed `references/supply-chain-risk-agent/` and `references/service-technician-agent/`; cookbook `templates/supply-chain-risk-agent/`

</td>
</tr>
<tr>
<td valign="top">

**Audit trail DDL** (`cis-audit-trail-ddl.sql`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

Schema per [Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md) / [Production Operations](../Production-Operations/production-operations-edd9e33.md); the audit-trail table is provisioned by the LightSpeed BTP Terraform modules (`infra/btp/`) — parameterize per engagement

</td>
</tr>
<tr>
<td valign="top">

**Shared state store DDL** (`cis-shared-state-store-ddl.sql`)

</td>
<td valign="top">

Design target

</td>
<td valign="top">

DDL schema specified in [Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md) / [Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md); standalone file not yet published

</td>
</tr>
<tr>
<td valign="top">

**Pre-migration checklists** (`cis-pre-migration-checklists.md`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md) templates; copy and fill per engagement

</td>
</tr>
<tr>
<td valign="top">

**Config naming convention** (`cis-config-naming-convention.md`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md) — derive from documented pattern

</td>
</tr>
<tr>
<td valign="top">

**Rollback runbook template** (`cis-rollback-runbook-template.md`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

[Production Operations](../Production-Operations/production-operations-edd9e33.md) — template with placeholders

</td>
</tr>
<tr>
<td valign="top">

**Sanitization rules** (`cis-sanitization-rules.md`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

[Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md) table

</td>
</tr>
<tr>
<td valign="top">

**Security checklist** (`cis-security-checklist.md`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

[Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md)

</td>
</tr>
<tr>
<td valign="top">

**APM archetype config** (`apm.yml` per domain)

</td>
<td valign="top">

Design target

</td>
<td valign="top">

Structure defined in the Starter Kit Distribution via APM section (an `apm.yml` declaring the MCP servers, skills, and `CLAUDE.md` context per engagement archetype); exemplar: `vs-template-apm`. Not yet a distributed artifact — see APM advisory below

</td>
</tr>
</table>

**Summary**: worker agent and schema artifacts are usable today via LightSpeed; the supervisor reference workflows (A, B, C) are the build priority. Teams forking LightSpeed components as an interim measure should document the divergence from the eventual Starter Kit version in their productization package.

### Starter Kit Contents

**Joule Studio Reference Workflows** (maintained in a CIS-internal Joule Studio project, version-controlled, tested against the current BAIP platform version):

**Joule Studio Reference Workflows**


<table>
<tr>
<th valign="top">

Artifact

</th>
<th valign="top">

Description

</th>
<th valign="top">

Fork and Configure

</th>
</tr>
<tr>
<td valign="top">

`cis-supervisor-workflow-b-ref`

</td>
<td valign="top">

Workflow B (Telemetry Monitor) with full rule set and cost extension; no engagement-specific content — **design target, not yet a standalone forkable artifact**

</td>
<td valign="top">

When published: fork; set anomaly thresholds after EAC baseline; configure recipient IDs

</td>
</tr>
<tr>
<td valign="top">

`cis-supervisor-workflow-c-ref`

</td>
<td valign="top">

Workflow C (Support Relay) — **design target, not yet a standalone forkable artifact**

</td>
<td valign="top">

When published: fork; verify tenant ID injection from environment config

</td>
</tr>
<tr>
<td valign="top">

`cis-supervisor-workflow-a-skeleton`

</td>
<td valign="top">

Workflow A with routing structure, webhook trigger, sapAgent callback, Task Center escalation, audit trail write; Code node placeholder for verification rules

</td>
<td valign="top">

Fork; author the Code node verification rules and LLM critic systemPrompt for this engagement

</td>
</tr>
<tr>
<td valign="top">

`cis-worker-agent-base`

</td>
<td valign="top">

Worker agent base pattern with sapOrchestration, sapMcpClient, verification routing, and supervised/unsupervised path

</td>
<td valign="top">

Fork; define task systemPrompt, wire SAP system connections, add MCP tools

</td>
</tr>
</table>

**Workflow A scoping: one skeleton per output type, not per agent**

The Workflow A skeleton is forked per output type — not per worker agent and not per engagement. Two workers that produce structurally similar outputs (for example, two pricing agents serving different markets, or two batch release agents for different plants) can share one Workflow A instance: the Code node verification rules and the LLM critic system prompt are the same because the output type is the same; threshold values that differ between workers are looked up by `agent_identity` in the Code node rather than duplicated into separate workflows.

Sharing breaks down when workers produce fundamentally different output types — different units, different validation logic, different reference calculation services. At that point a branching Code node becomes harder to test and maintain than two separate, simpler Workflow A instances. The rule of thumb: if the Code node rules would be identical for two workers, share the workflow; if they diverge significantly, create a separate instance.

In multi-agent deployments ([Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md), Topology 3), the shared supervisor naturally handles multiple workers of different output types by differentiating verification by `agent_identity`. The supervisor sits between the workers and the orchestrator: it verifies each worker's output independently, tracks which callbacks have arrived via the Pending Set table in the shared state store, and releases the combined approved set to the orchestrator only when all workers have been verified. The orchestrator never receives an unverified worker output.

**Schema Artifacts** (DDL scripts; the audit-trail DDL is available now, the shared-state-store DDL is a design target):

**Schema Artifacts**


<table>
<tr>
<th valign="top">

Artifact

</th>
<th valign="top">

Description

</th>
</tr>
<tr>
<td valign="top">

`cis-audit-trail-ddl.sql`

</td>
<td valign="top">

HANA Cloud DDL for the append-only audit trail table; includes all base fields and cost extension fields from [Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md) and [Production Operations](../Production-Operations/production-operations-edd9e33.md) </td>
</tr>
<tr>
<td valign="top">

`cis-shared-state-store-ddl.sql`

</td>
<td valign="top">

**Design target:** HANA Cloud DDL for the supervisor shared state store (baseline table + Pending Set table for multi-agent Topology 3). The schema is specified in [Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md) / [Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md); a standalone DDL file is not yet published

</td>
</tr>
</table>

**HANA Cloud provisioning**: both DDL artifacts target HANA Cloud (not HANA on-premise). HANA Cloud is provisioned as a BTP service in the engagement's BTP sub-account. In Project LightSpeed, `pilot.yaml` handles HANA Cloud provisioning as part of the BTP desired-state model — `cookbookctl apply` provisions the HANA Cloud instance, schema, and all required tables in a single operation. For engagements not using LightSpeed, provision HANA Cloud manually via the BTP cockpit: allocate the HANA Cloud free tier or production plan in the appropriate sub-account, create the schema, and run the DDL scripts. Estimated storage for audit trail and shared state store: plan for 10–50 MB per month of production operation per agent; the append-only audit trail grows continuously — confirm a retention policy with the customer compliance team ([Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md)) and implement scheduled archiving after 90 days.

**Process and Configuration Artifacts**:

**Process and Configuration Artifacts**


<table>
<tr>
<th valign="top">

Artifact

</th>
<th valign="top">

Description

</th>
</tr>
<tr>
<td valign="top">

`cis-pre-migration-checklists.md`

</td>
<td valign="top">

Three environment transition checklists ([Production Operations](../Production-Operations/production-operations-edd9e33.md) ); used unchanged for every engagement

</td>
</tr>
<tr>
<td valign="top">

`cis-config-naming-convention.md`

</td>
<td valign="top">

Configuration parameter naming convention and environment-agnostic wiring pattern ([Production Operations](../Production-Operations/production-operations-edd9e33.md) )

</td>
</tr>
<tr>
<td valign="top">

`cis-rollback-runbook-template.md`

</td>
<td valign="top">

Rollback procedure template with placeholders for engagement-specific component versions and authorization contacts

</td>
</tr>
<tr>
<td valign="top">

`cis-sanitization-rules.md`

</td>
<td valign="top">

Sanitization rules table ([Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md) ); included by reference in the Sirius security concept documentation

</td>
</tr>
<tr>
<td valign="top">

`cis-security-checklist.md`

</td>
<td valign="top">

Per-workflow security checklist ([Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md) ); used as a review gate before every environment transition

</td>
</tr>
</table>

**Evaluation Skills** (Claude Code skills, installable via `npx skills add` or `agpm`):

**Evaluation Skills**


<table>
<tr>
<th valign="top">

Skill

</th>
<th valign="top">

Description

</th>
<th valign="top">

Install Command

</th>
</tr>
<tr>
<td valign="top">

`agentic-eval`

</td>
<td valign="top">

11+1 Agentic Engineering Spec Quality Assessment; evaluates spec completeness, AI-readiness, code-spec alignment (C6), MCP utilization, and SAP standards compliance (C9). Run at end of Discovery and pre-EAC. HTML and Word report export. Skill owners: Apurba Ghosh, Saurabh Kejriwal.

</td>
<td valign="top">

`npx skills add https://github.tools.sap/CISDeveloperEfficiencyTools/developer-efficiency-tools.git --skill agentic-eval`

</td>
</tr>
</table>

### Starter Kit Governance

The Starter Kit is owned by CIS T&O and governed as a product, not as documentation:

-   **Version**: each release of the Starter Kit is tagged. FDE teams record which Starter Kit version their engagement forked from.

-   **Update cadence**: the Starter Kit is updated when BAIP platform changes affect the reference workflows, when a production incident reveals a gap in the rule set or schema, or when a completed engagement produces a generalizable improvement.

-   **Improvement pipeline**: production findings from the continuous improvement loop ([CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md) ) that are generalizable across engagements are proposed as Starter Kit updates. CIS T&O reviews and merges them.

-   **Deprecation**: when BAIP node APIs or Joule Studio behavior changes, the Starter Kit is updated before the next engagement forks from it. Running a stale fork in production without reconciling against the current version is a governance gap.

### Starter Kit Distribution via APM

> ### Note:  
> **Advisory — APM provenance and use scope**: APM (Agent Package Manager) is a **Microsoft-originated, MIT-licensed open-source tool** (github.com/microsoft/apm), *adopted* by SAP for FDE build-time use — it is not an SAP-built tool. Governing rule (SDOL-005 / PPMS): OSS compliance obligations attach to components **shipped to customers**, not to CLI tools that run only on SAP dev workstations or CI. `apm install` is a build-time operation that writes config into harness directories and does not appear in the deployed customer artifact, so **build-time use on SAP-internal CI is fine**. Customer-tenant deployment (Topology A — customer-managed BTP sub-account) is **uncleared until a PPMS entry is confirmed**; the trigger is whether the `apm` binary or its generated output is bundled into a customer deliverable (Docker image, customer-operated CI, MCP server package). No confirmed SAP Software Hub / PPMS registration was found — treat any "Software Hub Adequate / Permitted" claim as unverified until the record is cited. For a binding use-scope ruling, contact **SAP_CIS_ATQ@sap.com**. Track PPMS confirmation as a Starter Kit readiness item before enabling any customer-facing distribution path.

The CIS Starter Kit configuration — which MCP servers, skills, and `CLAUDE.md` context apply to a given engagement type — should be distributed using APM (Agent Package Manager), a per-project agentic setup tool. An `apm.yml` file per engagement archetype (for example, asset management, regulated manufacturing) declares the correct tool set and skills; `apm install` deploys them to each engineer's environment; `apm compile` generates the project `CLAUDE.md`. This gives every FDE engineer on the squad an identical, version-locked starting point and avoids the global-install anti-pattern where each engineer's session starts with a different tool set.

The `apm.yml` template for each engagement archetype is a Starter Kit artifact, owned and versioned by CIS T&O alongside the reference workflows. See [Token Efficiency in Industry AI Delivery](../Token-Efficiency/token-efficiency-47721ff.md) for the per-project agentic setup pattern and the efficiency rationale behind it.

### Customer-Managed Skill Registry

Beyond the Starter Kit's cross-engagement reference artifacts, high-maturity CIS deployments benefit from a **customer-managed Skill Registry** — a governed store of reusable agent behaviors specific to that customer's SAP landscape:

**Customer-Managed Skill Registry Features**


<table>
<tr>
<th valign="top">

Governance Feature

</th>
<th valign="top">

What It Provides

</th>
</tr>
<tr>
<td valign="top">

**Version pinning**

</td>
<td valign="top">

Skills are versioned; agents pin to a specific skill version rather than pulling the latest

</td>
</tr>
<tr>
<td valign="top">

**Approval workflows**

</td>
<td valign="top">

New or updated skills require review and sign-off before they can be called in production

</td>
</tr>
<tr>
<td valign="top">

**Cross-team distribution**

</td>
<td valign="top">

Skills developed for one agent are discoverable and reusable by other agents in the same tenant — turning project-local patterns into tenant-wide assets

</td>
</tr>
<tr>
<td valign="top">

**Compounding knowledge**

</td>
<td valign="top">

Every production fix, edge case, and workaround feeds back into the registry as an updated skill; operational learning accumulates rather than being lost

</td>
</tr>
</table>

The Skill Registry sits at the boundary between the Starter Kit (cross-engagement artifacts maintained by CIS T&O) and engagement-specific Layer 3 content. Skills general enough to apply to multiple agents within a single customer deployment belong in the registry; skills specific to one workflow's logic do not.

**FDE delivery implication**: the Skill Registry is a productization and handover consideration. When the FDE team hands over to the customer's IT admin and CIS T&O, the registry is one of the governance artifacts the customer IT team should own and operate. Include the Skill Registry governance model — who approves new skills, who manages versions, how the registry is backed up — in the productization package alongside the agent authorization matrix and the T&O operational runbook.


<a name="loiocb60e894e5814d6087247467f7106b39__section_project_lightspeed"/>

## Project LightSpeed: CIS Reference Implementation

Project LightSpeed (`github.tools.sap/CIS-Solution-Acceleration/Project-Lightspeed`) is the current, built CIS component library. Where the Starter Kit section describes a specification of what the Starter Kit should contain, LightSpeed is what exists today. FDE teams should treat the LightSpeed component catalog as the authoritative starting point; Starter Kit artifacts that do not yet exist in a standalone CIS project can be bootstrapped from the corresponding LightSpeed component.

### Component Catalog

**LightSpeed Component Catalog**


<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Location in Repo

</th>
<th valign="top">

What It Provides

</th>
</tr>
<tr>
<td valign="top">

OTel/logging middleware

</td>
<td valign="top">

`references/middleware/`

</td>
<td valign="top">

Drop-in Python package; 2-line setup; graceful degradation (Cloud Logging, env-OTLP, logging-only fallback); works in local dev, SAP Cloud Logging, and sovereign regions

</td>
</tr>
<tr>
<td valign="top">

Supply-chain risk reference agent

</td>
<td valign="top">

`references/supply-chain-risk-agent/`

</td>
<td valign="top">

Reference agent implementation with live S/4HANA tooling; starting point for supply-chain domain engagements

</td>
</tr>
<tr>
<td valign="top">

Service technician reference agent

</td>
<td valign="top">

`references/service-technician-agent/`

</td>
<td valign="top">

Reference agent implementation with field service tooling; starting point for asset management domain engagements

</td>
</tr>
<tr>
<td valign="top">

Maintenance planner example agent

</td>
<td valign="top">

`agents/maintenance-planner/`

</td>
<td valign="top">

Complete example agent with spec, code, and Joule capability bundle

</td>
</tr>
<tr>
<td valign="top">

Procurement assistant example agent

</td>
<td valign="top">

`agents/procurement-assistant/`

</td>
<td valign="top">

Complete example agent with spec, code, and Joule capability bundle

</td>
</tr>
<tr>
<td valign="top">

BTP Terraform modules

</td>
<td valign="top">

`infra/btp/`

</td>
<td valign="top">

Reusable Terraform modules: foundation stack, identity stack, joule stack; parameterized for BTP sub-account provisioning

</td>
</tr>
<tr>
<td valign="top">

S/4HANA entity catalog

</td>
<td valign="top">

`catalog/s4_catalog.sqlite`

</td>
<td valign="top">

Offline SQLite catalog of S/4HANA canonical objects, attributes, and APIs; enables entity grounding without an EKX connection at build time

</td>
</tr>
<tr>
<td valign="top">

A2A Agent Toolkit

</td>
<td valign="top">

`toolkits/a2a-agent-toolkit/`

</td>
<td valign="top">

Agent scaffolding toolkit in TypeScript (CAP + Express) and Python (FastAPI + LangGraph) implementing the A2A inter-agent protocol

</td>
</tr>
<tr>
<td valign="top">

Discovery/build skills

</td>
<td valign="top">

30+ skill files across pipeline stages

</td>
<td valign="top">

Reusable Claude Code skills covering all 13 pipeline stages; each skill is independently usable on other projects

</td>
</tr>
<tr>
<td valign="top">

Engagement config template

</td>
<td valign="top">

`pilot.yaml.example`

</td>
<td valign="top">

Template for the BTP desired-state file (park/resume/adopt patterns); copy and parameterize per engagement

</td>
</tr>
</table>

The OTel middleware, Terraform modules, and A2A toolkit are drop-in components that can be adopted into any CIS engagement regardless of whether the full LightSpeed pipeline is used. The S/4HANA entity catalog (`s4_catalog.sqlite`) is particularly useful as a fallback when EKX connectivity is unavailable at build time.

**RAG ingest pipeline** (`fde_rag_ingest.py`): Project LightSpeed now includes a standard RAG ingest component as a stage 9 output. This component handles document chunking, embedding generation, and vector store population for BTP-hosted vector stores. It is parameterized for the engagement's grounding corpus (document types, chunking strategy, embedding model) and is a reusable Layer 2 component — the pipeline structure is fixed; the corpus configuration is per-engagement. For any FDE agent that uses RAG grounding (`sapOrchestration groundingConfig`), the LightSpeed RAG ingest pipeline should be the starting point rather than a custom implementation.

**Context transfer schema** (`fde_context_transfer.py`): LightSpeed also generates a standard context transfer artifact that documents the agent's knowledge extraction — the entity catalog, goal/capability mapping, integration bindings, and process KPIs — in a structured, machine-readable format. This schema is the bridge between discovery artifacts and the agent's runtime grounding. It is produced during stages 1–8 of the LightSpeed pipeline and updated throughout the sprint as grounding content evolves. The context transfer schema is a handover artifact: it documents what the agent knows and where it learned it, which is required for the Sirius security concept and for the continuous improvement loop when the grounding corpus is updated.

### Versioned-Skill Distribution Pattern

The book-of-fde and LightSpeed both use the same distribution model for Claude Code skills: a skill is a Markdown file (`skill-name.md`) stored in a git repository. Installation is a single `gh api` call against `github.tools.sap`; contribution back is a PR. A pre-commit version-bump hook enforces a version increment on every skill change, ensuring the version header in the skill file always reflects the current state.

This distribution model is the standard for CIS CoE skill maintenance:

**Versioned-Skill Distribution Operations**


<table>
<tr>
<th valign="top">

Operation

</th>
<th valign="top">

Mechanism

</th>
</tr>
<tr>
<td valign="top">

Install a skill

</td>
<td valign="top">

`gh api` call against the repository; skill file written to `~/.claude/skills/`

</td>
</tr>
<tr>
<td valign="top">

Contribute a fix or improvement

</td>
<td valign="top">

PR against the source repository

</td>
</tr>
<tr>
<td valign="top">

Enforce version consistency

</td>
<td valign="top">

Pre-commit hook increments version on every change to the skill file

</td>
</tr>
<tr>
<td valign="top">

Pin a specific version

</td>
<td valign="top">

Reference the commit SHA or tag in the install command

</td>
</tr>
<tr>
<td valign="top">

Audit skill contents

</td>
<td valign="top">

Inspect the Markdown file directly — no binary packaging

</td>
</tr>
</table>

### Security Note: Skills Are Customer-Inspectable

Skill files are plain Markdown and are readable by anyone with access to the Claude session or the repository. This is a feature (auditability) and a constraint (confidentiality):

-   **Skills are appropriate for**: process automation, phase routing, checklist enforcement, tool orchestration, and coordination logic that can be shared with customers or disclosed in an audit.

-   **Skills are NOT appropriate for**: proprietary algorithms, confidential business rules, pricing logic, or any logic that must remain confidential relative to the customer.

-   **For IP-sensitive logic**: implement in a code-based agent (Python/TypeScript), not a skill. The deployed agent binary is not disclosed; the skill file is. These serve different confidentiality profiles.

If an FDE team is uncertain whether a piece of logic belongs in a skill or in agent code, the default is agent code.

### Custom Agentic Solutions CoE Cookbook — Second CIS Reference Library

Alongside Project LightSpeed, the **Custom Agentic Solutions CoE Cookbook** (`github.tools.sap/<internal-org>/Custom-Agentic-solutions-CoE`) is a second CIS reference implementation and field delivery resource. Where LightSpeed focuses on the end-to-end FDE pipeline (13 stages from discovery artifacts to deployed multi-agent application), the cookbook focuses on the agent delivery primitives: how to scaffold a BTP-hosted A2A agent, deploy it to CF or Kyma, and wire it into Joule via the customer-managed (CMJ) orchestrator path (the sovereign/legacy option) — under the SMJ default, a BYOA agent instead registers to the SAP-managed Agent Gateway via A2A.

The two libraries are complementary, not competing. LightSpeed uses and embeds the cookbook's A2A Agent Toolkit. For most FDE engagements the cookbook provides the low-level build and deploy patterns; LightSpeed provides the engagement-level orchestration and pipeline.

**Cookbook structure (key components)**:

**Cookbook Key Components**


<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Location in Repo

</th>
<th valign="top">

What It Provides

</th>
</tr>
<tr>
<td valign="top">

A2A Agent Toolkit

</td>
<td valign="top">

`toolkits/a2a-agent-toolkit/`

</td>
<td valign="top">

Claude Code plugin (`sap-a2a-agent-toolkit`, v0.6.0) that scaffolds and deploys A2A agents; drives Checkpoints 1–3

</td>
</tr>
<tr>
<td valign="top">

Supply-chain risk agent template

</td>
<td valign="top">

`templates/supply-chain-risk-agent/`

</td>
<td valign="top">

Canonical Python A2A agent starter (four-file shape: `agent.py`, `agent_executor.py`, `app.py`, `__main__.py`)

</td>
</tr>
<tr>
<td valign="top">

Joule capability skill

</td>
<td valign="top">

`skills/sap-joule-capability/`

</td>
<td valign="top">

4-file YAML bundle structure, deploy commands, pitfall guidance for CMJ Joule registration

</td>
</tr>
<tr>
<td valign="top">

BTP deploy recipes

</td>
<td valign="top">

`recipes/02-deploy-btp/`

</td>
<td valign="top">

CF and Kyma deployment patterns with smoke tests

</td>
</tr>
<tr>
<td valign="top">

Joule wiring recipe

</td>
<td valign="top">

`recipes/03-joule/`

</td>
<td valign="top">

Region availability check, `joule deploy` command, troubleshooting, cleanup

</td>
</tr>
<tr>
<td valign="top">

Sovereign model gateway

</td>
<td valign="top">

`recipes/optional/sovereign-model-gateway/`

</td>
<td valign="top">

Required for sovereign/regulated regions; OpenAI-compatible gateway as AI Core replacement

</td>
</tr>
<tr>
<td valign="top">

Observability recipe

</td>
<td valign="top">

`recipes/optional/observe-and-eval/`

</td>
<td valign="top">

Cloud Logging binding, OTel middleware placeholder, region-specific log targets

</td>
</tr>
<tr>
<td valign="top">

FDE code-first stack reference

</td>
<td valign="top">

`references/fde-code-first-stack.md`

</td>
<td valign="top">

North-star stack (Kyma + A2A v1.0 + SAP Cloud SDK for Python); ownership split; access checklist

</td>
</tr>
<tr>
<td valign="top">

SwissKnife pointer

</td>
<td valign="top">

`references/btp-joule-a2a-pro-code-agent/`

</td>
<td valign="top">

Pointer to public SAP Architecture Center reference (`SAP-samples/btp-joule-a2a-pro-code-agent`)

</td>
</tr>
<tr>
<td valign="top">

cookbookctl

</td>
<td valign="top">

Root

</td>
<td valign="top">

CLI coordinator for manifest-driven BTP provisioning; `pilot.yaml` desired-state model

</td>
</tr>
</table>

**Three-checkpoint delivery playbook**:

**Three-Checkpoint Delivery Playbook**


<table>
<tr>
<th valign="top">

Checkpoint

</th>
<th valign="top">

Recipe

</th>
<th valign="top">

Output Evidence

</th>
</tr>
<tr>
<td valign="top">

**1 — Local agent**

</td>
<td valign="top">

`recipes/01-scaffold-agent/`

</td>
<td valign="top">

`SendMessage` returns non-error response locally

</td>
</tr>
<tr>
<td valign="top">

**2 — BTP deploy**

</td>
<td valign="top">

`recipes/02-deploy-btp/`

</td>
<td valign="top">

Agent Card reachable at public URL; smoke test passes

</td>
</tr>
<tr>
<td valign="top">

**3 — Joule wiring**

</td>
<td valign="top">

`recipes/03-joule/`

</td>
<td valign="top">

`joule list capabilities` shows ACTIVE; Joule routes a real prompt to the agent

</td>
</tr>
</table>

**Field adoption context**: the cookbook is actively used in GDH (Go-to-Market Digital Hub) deployments, with Kai Mühlbauer's team using it as a customer engagement template. It was built initially for SovCloud/NS2 deployments where BAIP is not yet available, and has since been adopted more widely for commercial-region FDE engagements. The cookbook is a source of truth for **BYOA custom-agent runtime delivery patterns** — building a custom agent on the customer's BTP subaccount and registering it to the SAP-managed orchestrator. Note that delivery should be read as **two orthogonal axes, not a single bundled "CMJ/BYOA" path**: the **orchestrator** is SMJ (SAP-Managed Joule), the default for new engagements, with CMJ (Customer-Managed Joule) now a deprecated migration target rather than a delivery choice; the **custom-agent runtime** is either BYOA (customer subaccount — the standard runtime for productization-intent agents, registered to SMJ via A2A) or Joule Studio 2.0 (low-code). **BYOA is not a fallback.**

**Public release track**: the cookbook has an active port to a public SAP OSS repository (`github.com/SAP/custom-agentic-cookbook`). Keep internal-only URLs, dependencies, and customer material out of any content intended for the public track.


<a name="loiocb60e894e5814d6087247467f7106b39__section_discovery_parameterization"/>

## What Parameterization Happens in FDE Discovery

Discovery is where Layer 2 skeletons acquire their per-engagement content. The following decisions must be made in Discovery — before the Development phase begins — to ensure the FDE sprint can proceed without scope uncertainty:

**Discovery Parameterization Decisions**


<table>
<tr>
<th valign="top">

Decision

</th>
<th valign="top">

Where It Is Documented

</th>
<th valign="top">

Who Owns It

</th>
</tr>
<tr>
<td valign="top">

Supervisor scope: which workflows are in scope (A only / A+B / A+B+C)?

</td>
<td valign="top">

Draft Product Design Specs

</td>
<td valign="top">

FDE Domain PM + Discovery Lead

</td>
</tr>
<tr>
<td valign="top">

Multi-agent or single agent? If multi-agent: which topology (1 / 2 / 3)?

</td>
<td valign="top">

Draft Product Design Specs

</td>
<td valign="top">

FDE Dev Architect

</td>
</tr>
<tr>
<td valign="top">

Which SAP calculation service backs Layer 2 of the guardrail stack? (Or: does one exist at all?)

</td>
<td valign="top">

Draft Product Design Specs

</td>
<td valign="top">

FDE Application Engineer + P&E Expert

</td>
</tr>
<tr>
<td valign="top">

Human approval threshold values (absolute and deviation-based) for each numerical output

</td>
<td valign="top">

Draft Product Design Specs

</td>
<td valign="top">

FDE team + process owner

</td>
</tr>
<tr>
<td valign="top">

Authorization matrix for all agents

</td>
<td valign="top">

Draft Product Design Specs

</td>
<td valign="top">

FDE Dev Architect + customer IT admin

</td>
</tr>
<tr>
<td valign="top">

Regulatory audit trail retention requirement

</td>
<td valign="top">

Legal agreement plan

</td>
<td valign="top">

Customer compliance team (confirmed during Discovery)

</td>
</tr>
<tr>
<td valign="top">

Process KPIs (Dimension 2) for the quality framework

</td>
<td valign="top">

TO-BE process flow

</td>
<td valign="top">

FDE Domain PM + process owner

</td>
</tr>
<tr>
<td valign="top">

Legal agreement path for Workflow C (support relay), if in scope

</td>
<td valign="top">

Legal agreement plan

</td>
<td valign="top">

FDE team + SAP legal

</td>
</tr>
<tr>
<td valign="top">

Cost monitoring budget values and alert owners post-handover

</td>
<td valign="top">

Production handover plan

</td>
<td valign="top">

FDE Domain PM + customer IT admin

</td>
</tr>
<tr>
<td valign="top">

Eval test data strategy: synthetic vs. TEA-covered real data

</td>
<td valign="top">

Eval plan

</td>
<td valign="top">

FDE Application Engineer + Data Scientist

</td>
</tr>
</table>

Decisions left unresolved at the end of Discovery become scope risk in the Execute phase. The threshold values and authorization matrices in particular cannot be retrofitted — they shape the workflow design and the Sirius security concept documentation that must be complete before productization.


<a name="loiocb60e894e5814d6087247467f7106b39__section_fde_sprint_scope"/>

## FDE Sprint Scope Implication

The Starter Kit does not eliminate the FDE sprint's build effort — it removes the infrastructure rebuild that otherwise consumes sprint capacity on work that adds no differentiation for this customer.

Approximate effort comparison for the supervisor stack (Workflows A, B, C, shared state store):

**Effort Comparison — With and Without Starter Kit**


<table>
<tr>
<th valign="top">

Activity

</th>
<th valign="top">

Without Starter Kit

</th>
<th valign="top">

With Starter Kit

</th>
</tr>
<tr>
<td valign="top">

Workflow C (Support Relay)

</td>
<td valign="top">

1–2 days build + test

</td>
<td valign="top">

~2 hours: fork, configure, verify sanitization in EAC

</td>
</tr>
<tr>
<td valign="top">

Workflow B (Telemetry Monitor)

</td>
<td valign="top">

2–3 days build + test

</td>
<td valign="top">

~1 day: fork, calibrate thresholds post-EAC baseline, test with injected anomalies

</td>
</tr>
<tr>
<td valign="top">

Workflow A skeleton

</td>
<td valign="top">

2–3 days build

</td>
<td valign="top">

~1 day: fork, author Code node verification rules + critic prompt, test routing

</td>
</tr>
<tr>
<td valign="top">

Shared state store

</td>
<td valign="top">

0.5 days DDL + deploy

</td>
<td valign="top">

~1 hour: deploy from DDL template, verify in each environment

</td>
</tr>
<tr>
<td valign="top">

Audit trail

</td>
<td valign="top">

0.5 days DDL + deploy

</td>
<td valign="top">

~1 hour: deploy from DDL template, verify append-only behavior

</td>
</tr>
</table>

These are estimates for planning purposes, not commitments. Actual effort depends on the complexity of the engagement-specific verification rules in Workflow A and the availability of a deterministic Layer 2 reference service. The "With Starter Kit" figures for Workflow B, Workflow C, and the shared state store assume the published reference artifact; per the status table in the CIS Starter Kit section, these three are currently **design targets**, so until they are published the applicable effort is closer to the "Without Starter Kit" column.

The freed capacity goes into the work that actually differentiates the engagement: the worker agent's business logic, the evaluation test suite, integration testing, and the process owner alignment sessions that determine threshold values.

**Important**: the [Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md) build sequence still applies when using the Starter Kit. Fork and configure the supervisor workflows only after the worker agent is stable — not before. A supervisor configured to govern an unstable worker creates debugging noise that slows the sprint.


<a name="loiocb60e894e5814d6087247467f7106b39__section_agent_hub_relationship"/>

## Relationship to SAP AI Agent Hub

The Starter Kit is a build-time and operations-time artifact; SAP AI Agent Hub is the runtime governance layer. The two are complementary:

-   The Agent Hub's **Agent Registry** records every deployed agent (workers, supervisors, orchestrators) with its identity, version, and authorization scope. The Starter Kit provides the artifacts that get registered.

-   The Agent Hub's **Agent Observability** surfaces the telemetry that Workflow B monitors via the SAP Cloud ALM API. The Starter Kit's Workflow B is the consumer of what the Agent Hub's harness produces.

-   The Agent Hub's **Secure & Govern** capability manages the threshold configuration parameters that the Starter Kit's environment-agnostic configuration pattern exposes as named parameters.

-   The Agent Hub's **Agent Evals & Assessment** runs the eval test suite against the agents the FDE team builds using the Starter Kit worker agent base pattern.

The Starter Kit does not duplicate any of these capabilities. It implements the CIS-side operational layer — the supervisor agent pattern, the audit trail, the escalation logic — that operates within and on top of what the Agent Hub provides.


<a name="loiocb60e894e5814d6087247467f7106b39__section_continuous_improvement"/>

## Continuous Improvement: From Production Back to the Starter Kit

The continuous improvement loop in [CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md) runs within individual engagements. The Starter Kit adds a second loop that runs across engagements.

```
[Production engagement N]
  CIS T&O identifies improvement in Workflow B rule set,
  audit trail field, or pre-migration checklist item
         ↓
[CIS T&O: assess generalizability]
    ├── Engagement-specific finding → feed into that engagement's improvement loop only
    └── Generalizable finding → propose Starter Kit update
         ↓
[CIS T&O: review and merge into Starter Kit (next version)]
         ↓
[Engagement N+1 forks from updated Starter Kit]
  The improvement is inherited without additional build effort
```

Examples of findings that become Starter Kit improvements:

-   A new BAIP telemetry field becomes available that improves anomaly classification in Workflow B's Code node → add to the reference implementation

-   A production incident reveals that a specific audit trail field was missing and required a schema change mid-engagement → add the field to the base DDL

-   A pre-migration checklist item is consistently missed across engagements → add a more explicit verification step to the checklist

-   A new failure mode is observed across two independent engagements in the same Industry AI domain → add a new rule to the Workflow B rule set


<a name="loiocb60e894e5814d6087247467f7106b39__section_summary"/>

## Summary

**Layer Summary**


<table>
<tr>
<th valign="top">

Layer

</th>
<th valign="top">

What It Contains

</th>
<th valign="top">

Delivery Model

</th>
<th valign="top">

Maintained By

</th>
</tr>
<tr>
<td valign="top">

Layer 1 — Deploy as-is

</td>
<td valign="top">

Workflow C, audit trail schemas, anomaly rule set, sanitization rules, pre-migration checklists, version stack discipline, decision tables, fail-safe rules

</td>
<td valign="top">

Inherited by FDE team; no sprint effort

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Layer 2 — Parameterize and deploy

</td>
<td valign="top">

Workflow B (threshold calibration), Workflow A skeleton (Code node + critic prompt authored per engagement), guardrail stack structure, quality framework, eval structure, rollback template, configuration naming convention

</td>
<td valign="top">

Forked and configured during FDE Execute; parameterization decided in Discovery

</td>
<td valign="top">

CIS T&O (skeleton); FDE team (per-engagement content)

</td>
</tr>
<tr>
<td valign="top">

Layer 3 — Engagement-specific

</td>
<td valign="top">

Worker agent business logic, process KPIs, verification rules, threshold values, authorization matrices, regulatory requirements

</td>
<td valign="top">

Built from scratch in FDE Execute

</td>
<td valign="top">

FDE team (with process owner and customer IT admin)

</td>
</tr>
</table>

The division of labour is clear: CIS T&O invests in making Layers 1 and 2 excellent so that FDE teams can invest their sprint capacity in Layer 3. Layer 3 is where the customer's specific business problem is solved — it is the only layer where CIS cannot substitute a prior answer for the work of understanding this customer.

**Related Information**  




[Joule Studio — Glossary](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/1e214183c85c4e4faff59a96c724f4fb.html "Key concepts for Joule Studio: custom agents, managed runtime, SAP Business AI Platform.") :arrow_upper_right:
[Code-Based Agents (Bring Your Own Agent)](https://help.sap.com/docs/JOULE/ba88d1ec6a1b442098863d577c19b0c0/2f9701e60fd24e948c2c74fe9e55ce23.html "How to register a custom code-based agent (BYOA) to Joule via A2A.") :arrow_upper_right:
[Token Efficiency in Industry AI Delivery](../Token-Efficiency/token-efficiency-47721ff.md)

[Production Operations](../Production-Operations/production-operations-edd9e33.md)

[CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md)

[SAP AI Agent Hub — Plan and Build](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/e2a29b0c422e4aceadee658f4bbab71b.html "SAP AI Agent Hub documentation") :arrow_upper_right:

[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Joule Studio product documentation") :arrow_upper_right:
