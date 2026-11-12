<!-- loioedd9e33b3f6c4879a3d8c0379eba47ca -->

# Production Operations

Production operations for Industry AI agents covers three core concerns that must be designed in Discovery: agent versioning and rollback, cost and token monitoring, and environment migration from dev through test to production. This chapter also specifies production shipping readiness requirements (TR1–TR14) and BTP deployment patterns.



<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_what_this_chapter_covers"/>

## What This Chapter Covers

The concept so far addresses quality, security, guardrails, and build patterns. What it does not yet address is what happens once an agent is in production and the FDE team has moved on. Three operational concerns arise in every long-running Industry AI deployment that the earlier chapters do not design for explicitly:

1.  **Agent versioning and rollback**: agents evolve — prompts are updated, models change, thresholds are adjusted. When a version change degrades quality, how is the rollback executed, who authorizes it, and what happens to in-flight transactions?

2.  **Cost and token monitoring**: the concept covers quality and security telemetry but not LLM token consumption and cost visibility. Runaway agent loops, model version changes, and expanding context windows all affect costs in ways that can surface before quality signals degrade.

3.  **Environment migration (dev → test → production)**: the FDE sprint moves through multiple environments, but the artifact checklist for each transition — what moves, what must be re-provisioned, what changes between environments — is not explicit in the build sequence.

These are not afterthoughts. They are production readiness requirements that should be designed in Discovery and tested in Develop (formerly Execute), not improvised at handover.

Beyond these three core concerns, the chapter also specifies the production monitoring additions that surround cost tracking — synthetic monitoring and SAP-operated agentic remediation; production shipping readiness, the AI Golden Path's mandatory TR1–TR14 Technical Requirements checklist; and BTP deployment patterns — the 9-step autonomous deploy loop, the OTel observability middleware, and the Dynatrace CF integration.


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_agent_versioning_strategy"/>

## Agent Versioning Strategy

### The Version Stack

A deployed agent is not a single artifact — it is a coordinated set of versioned components that must be updated and rolled back as a group:

**Table: Agent Version Stack Components**


<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

What It Governs

</th>
<th valign="top">

Where It Is Versioned

</th>
</tr>
<tr>
<td valign="top">

Joule Studio workflow definition

</td>
<td valign="top">

The workflow's node structure, routing logic, and wiring

</td>
<td valign="top">

SAP AI Agent Hub / Joule Studio version history

</td>
</tr>
<tr>
<td valign="top">

SAP Prompt Registry entry

</td>
<td valign="top">

The agent's system prompt (and critic prompts for supervisor)

</td>
<td valign="top">

Git repo + workflow parameter (current); SAP Prompt Registry when `promptTemplate` source ships

</td>
</tr>
<tr>
<td valign="top">

Model selection

</td>
<td valign="top">

Which LLM is used in each `sapOrchestration` node

</td>
<td valign="top">

`sapOrchestration` model parameter (BAIP configuration)

</td>
</tr>
<tr>
<td valign="top">

Threshold configuration

</td>
<td valign="top">

Human approval thresholds, anomaly detection baselines, tolerance bands

</td>
<td valign="top">

n8n workflow variables (`$vars`) in Joule Studio UI — no redeployment required to change

</td>
</tr>
<tr>
<td valign="top">

Authorization scope

</td>
<td valign="top">

Which tools and entities the agent can access

</td>
<td valign="top">

BAIP application registration (policy assignments)

</td>
</tr>
</table>

**The risk of partial updates**: updating one component without coordinating the others is the most common source of post-deployment quality degradation that is difficult to diagnose. A model update without a prompt update may break structured output formatting. A threshold change without an eval re-run may create an approval gate that fires too frequently or too rarely. A workflow version change without updating the authorization scope may silently over-scope the agent.

**Design principle**: treat a version bump as a version of the entire stack, not of individual components. Document the intended version of all five components together in the Agent Hub, and execute updates as a coordinated release.

### Versioning in Joule Studio

**Current state (node v1.5.0, verified 2026-10-22)**: the `sapOrchestration` node supports only inline prompts (`promptSource: inline`). A `promptTemplate` source — which would enable external registry references — is planned but not yet shipped. Until it is available, the following fallback approach applies.

**Prompt versioning fallback approach:**

-   Store prompt text as a **workflow-level string parameter**, configurable per environment via BAIP configuration rather than hardcoded in the node
-   Maintain the canonical prompt text in the **engagement Git repository** alongside the workflow definition — the commit history is the version history
-   Tag workflow releases with the prompt version in the version stack documentation (e.g., `prompt: v1.2` alongside model and workflow version)
-   Prompt changes require a **workflow redeployment** — treat them as a workflow-level change in the change classification table below

**When `promptTemplate` source becomes available**, migrating to it will restore no-redeployment prompt updates and env-specific version references without requiring architectural changes. The version stack model remains valid; only the storage location changes from inline to registry.

Benefits that will be restored when `promptTemplate` ships:

-   Prompt changes without redeploying the workflow
-   Each prompt version auditable: who changed it, when, what changed
-   Rollback to a prior prompt version as a Registry operation only
-   Different environments referencing different named versions of the same prompt

### Change Classification

Not all changes carry the same rollback risk. Classify changes before deploying:

**Table: Change Classification and Risk**


<table>
<tr>
<th valign="top">

Change Type

</th>
<th valign="top">

Risk Level

</th>
<th valign="top">

Required Before Deploying

</th>
</tr>
<tr>
<td valign="top">

Threshold adjustment

</td>
<td valign="top">

Low

</td>
<td valign="top">

Process owner approval; log in audit trail

</td>
</tr>
<tr>
<td valign="top">

Prompt update (minor wording) — **SAP Prompt Registry path** *(future — when `promptTemplate` ships)*

</td>
<td valign="top">

Medium

</td>
<td valign="top">

Eval run against existing test suite; review diff

</td>
</tr>
<tr>
<td valign="top">

Prompt update (minor wording) — **inline prompt fallback path**

</td>
<td valign="top">

High

</td>
<td valign="top">

Full eval run; workflow redeployment required; treat as Workflow structure change

</td>
</tr>
<tr>
<td valign="top">

Prompt update (structural change to output schema)

</td>
<td valign="top">

High

</td>
<td valign="top">

Full eval run; boundary condition tests; EAC re-validation

</td>
</tr>
<tr>
<td valign="top">

Model version change

</td>
<td valign="top">

High

</td>
<td valign="top">

Full eval run; compare Dimension 1 metrics before/after

</td>
</tr>
<tr>
<td valign="top">

Workflow structure change

</td>
<td valign="top">

High

</td>
<td valign="top">

Full eval run; security concept update; process owner review

</td>
</tr>
<tr>
<td valign="top">

Authorization scope change

</td>
<td valign="top">

Very high

</td>
<td valign="top">

DPCE (Data Protection and Compliance Engineering) review; CA assessment if scope expands

</td>
</tr>
</table>

**Inline prompt risk collapse**: as of mid-2026, `promptTemplate` is not yet available in the `sapOrchestration` node (v1.5.0). Agents built before it becomes available must embed prompts inline in the workflow. This collapses prompt updates — even minor wording changes — to the same risk tier as Workflow structure changes, because any prompt edit requires a workflow redeployment. Apply the full High-risk protocol for all prompt changes on inline-prompt agents, and use a lightweight subset of the eval suite (targeted test cases covering the changed behavior only) to keep iteration time manageable. When `promptTemplate` becomes available, migrate inline prompts to the Registry as a planned sprint item and reclassify prompt updates to the Medium risk tier.

### Multi-Agent Version Coordination

In a multi-agent system, each worker agent has its own version stack. This introduces a compatibility dimension that single-agent version management does not have: rolling back one worker agent may break its interface contract with the orchestrator or another worker agent if they exchange structured outputs.

Before executing any version change or rollback in a multi-agent deployment:

-   **Map interface contracts**: any structured output from the affected agent consumed by another agent is an interface contract. Document these contracts in the release plan.
-   **Check schema compatibility**: confirm that the changed version's output schema is compatible with every consuming agent's current version. A model or prompt change that alters structured output field names or types can silently break a downstream agent.
-   **Roll back as a group when in doubt**: if compatibility impact is uncertain, treat the multi-agent system as a coordinated stack and roll back all affected agents together, applying the five-component discipline to each.

The CIS T&O operating runbook should include a dependency map of inter-agent interface contracts for any multi-agent deployment — without it, a rollback that is safe for one agent may introduce a new failure in another.


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_rollback_triggers_and_mechanics"/>

## Rollback: Triggers and Mechanics

### Rollback Triggers

A rollback is warranted when production quality metrics show a confirmed regression correlated with a version change. The following signals, correlated with a known deployment event, constitute rollback triggers:

**Table: Rollback Trigger Signals**


<table>
<tr>
<th valign="top">

Signal

</th>
<th valign="top">

Threshold

</th>
<th valign="top">

Source

</th>
</tr>
<tr>
<td valign="top">

Human override rate increase

</td>
<td valign="top">

> 20 percentage points above pre-deployment baseline

</td>
<td valign="top">

Audit trail `delta` field

</td>
</tr>
<tr>
<td valign="top">

Verification rejection rate increase

</td>
<td valign="top">

> 15 percentage points above pre-deployment baseline

</td>
<td valign="top">

Supervisor Workflow A telemetry

</td>
</tr>
<tr>
<td valign="top">

Process KPI degradation

</td>
<td valign="top">

> 20% below pre-deployment baseline

</td>
<td valign="top">

Process owner report or Agent Hub process KPI tracking

</td>
</tr>
<tr>
<td valign="top">

Structured output format failures

</td>
<td valign="top">

> 5% of invocations producing malformed output

</td>
<td valign="top">

BAIP harness error rate telemetry

</td>
</tr>
<tr>
<td valign="top">

LLM refusal rate

</td>
<td valign="top">

Any non-zero refusal rate in production (zero expected baseline)

</td>
<td valign="top">

SAP Cloud ALM error classification

</td>
</tr>
</table>

The CIS CoE operating rhythm classifies rollback-triggering regressions as Critical or High severity, with same-business-day or 2-business-day response targets respectively. See [CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md).

**Correlation requirement**: before triggering a rollback, confirm that the signal change is correlated with a version deployment event. Random variation should not trigger rollback; a step-change in a metric following a deployment event should. The audit trail's `agent_version` field makes this correlation explicit — every committed output carries the agent version that produced it.

### Rollback Mechanics

A rollback reverses one or more version stack components. The scope of the rollback depends on which component caused the regression:

**Prompt rollback**:

-   **Current state (inline prompts):** a prompt change or rollback requires a **workflow redeployment** today — prompts are stored inline / as a workflow-level parameter, so reverting the prompt text means restoring the prior value and redeploying the workflow. Treat it as a workflow-level change and apply the workflow-rollback mechanics below.
-   **(future — when `promptTemplate` ships):** restore the prior version in the SAP Prompt Registry with **no workflow redeployment required**; the change takes effect on the next agent invocation. This is the fastest, lowest-risk path once available.
-   In either case, audit the prompt diff to understand what change caused the regression

**Model rollback**:

-   Change the `model` parameter in the `sapOrchestration` node's BAIP configuration
-   No workflow redeployment required (if model is referenced by name as BAIP configuration, not hardcoded in the workflow node)
-   Verify that the prior model version is still available in the generative AI hub (model deprecation can prevent rollback — check SAP Note 3437766 for model availability)

**Workflow rollback**:

-   Restore the prior workflow version from Joule Studio / Agent Hub version history
-   Requires a deployment operation; more disruptive than prompt or model rollback
-   Confirm that the prior workflow version is compatible with the current authorization scope and BAIP application registration

**Threshold rollback**:

-   Restore prior threshold values in the n8n Variables UI (`$vars`)
-   Immediate effect; no deployment required
-   The change must be logged in the audit trail as a configuration change event

**Full stack rollback** (last resort):

-   Restore all five components to the prior coordinated version
-   Execute in the order: threshold → model → prompt → authorization scope → workflow (workflow last, as it is the most disruptive)
-   Verify the full stack in a test environment before applying to production if time permits

### State Continuity During Rollback

Rollback does not erase in-flight transactions or the audit trail:

-   **In-flight transactions**: active agent sessions should be allowed to complete before rollback takes effect. Before relying on graceful drain, **confirm it is supported by the BAIP managed runtime configuration for the customer's tenant**. If it is supported, new invocations can be routed to the rolled-back version while existing sessions finish on the current one. If graceful drain is **not** supported, execute an explicit hard cutover:
    1.  **Pause new invocations** — set the agent's routing to inactive in Agent Hub or at the BAIP application registration level; new requests will receive a standard unavailability response.
    2.  **Wait for in-flight sessions to complete** — monitor the active session count in BAIP telemetry; confirm the count reaches zero (allow up to 5 minutes; longer sessions may indicate a stuck worker that requires manual intervention).
    3.  **Execute the rollback** — apply the version change to the affected stack component(s).
    4.  **Re-enable invocations** — restore routing. The first post-rollback invocation serves as a live smoke test; review its trace before declaring the rollback complete.
    This procedure must be verified as feasible during EAC for any customer tenant where graceful drain availability cannot be confirmed in advance.
-   **Audit trail**: the audit trail is append-only and version-tagged. Records produced by the reverted version remain in the audit trail; the `agent_version` field identifies them. Do not delete or modify audit trail records during rollback — they are the evidence needed to understand what went wrong.
-   **Shared state store**: the supervisor's rolling baseline should be reset after a rollback. The baseline built up under the degraded version is not representative of the rolled-back version's behavior. Reset the baseline and allow the supervisor to rebuild it from fresh observations over the next 24–48 hours.
-   **Human approval queue**: any pending `sapTaskCenter` tasks created by the degraded version should be reviewed by the process owner before approval. Flag the version change in the Task Center notification so approvers know the context has changed.

### Rollback Authorization

Who authorizes a rollback depends on the deployment topology:

**Table: Rollback Authorization by Topology**


<table>
<tr>
<th valign="top">

Topology

</th>
<th valign="top">

Rollback Authorization

</th>
</tr>
<tr>
<td valign="top">

Customer tenant (Topology A)

</td>
<td valign="top">

Customer IT admin authorizes; CIS T&O proposes and may execute with consent

</td>
</tr>
<tr>
<td valign="top">

SAP managed tenant (Topology B)

</td>
<td valign="top">

CIS T&O can execute rollback directly as the operator; notify customer

</td>
</tr>
</table>

For all rollbacks: document the trigger, the scope of the rollback, the authorization, and the post-rollback verification result in the CIS T&O operating log.

### Model Lifecycle and Deprecation

Model deprecation is an architectural risk, not just a rollback edge case. SAP Note 3437766 is the authoritative source for model availability, deprecation notices, and retirement dates in the generative AI hub. Key facts that shape the design:

**Two version modes** — choose deliberately at build time:

-   `latest`: automatically moved to the newest model version when the current one is deprecated. No manual action required, but model behavior can change without a deployment event, making regressions harder to correlate and rollback impossible (there is no prior "version" to restore).
-   **Pinned version** (recommended for production): the specific model version is used until explicitly changed. When deprecated, the team must migrate to a successor — but the migration is a deliberate, tested event.

**No guaranteed deprecation notice period**: SAP Note 3437766 states that some models may retire without an advance deprecation period. Alerts are sent when a retirement date is known, but "known" is not guaranteed to be far in advance. This means a pinned model could be retired before a validated successor is in place — making rollback to the prior model version impossible.

**Proactive model lifecycle strategy**:

1.  **Monitor** SAP Note 3437766 actively. When a successor model appears in the note (or a deprecation date is set for the current model), treat it as a sprint-level work item.
2.  **Test the successor** against the engagement's eval suite before the current version is deprecated. Pay particular attention to structured output formatting, refusal behavior, and precision-sensitive outputs — these are the dimensions most likely to shift between model versions.
3.  **Version the model reference** in the `sapOrchestration` BAIP configuration as part of the five-component version stack, so any model change is a tracked deployment event with a corresponding baseline measurement.
4.  **Document the validated successor** in the CIS T&O operating log for the engagement, so the team that inherits the agent post-productization knows the tested migration path.

The transition from `latest` to a pinned successor should follow the same dev → test → prod migration path as any other version change.

### Emergency Procedure When Model Rollback Is Impossible

If the current pinned model is deprecated with no advance notice and the validated successor is not yet in place, rollback is impossible. This is not a theoretical edge case — SAP Note 3437766 explicitly states that some model retirements occur without an advance deprecation period.

**Emergency response when model rollback is not possible:**

1.  **Activate conservative LLM critic configuration**: the Workflow A LLM critic prompt includes a tolerance band for threshold deviations. When the production model changes unexpectedly, immediately tighten the LLM critic's tolerance to the most conservative setting (typically ±5% of the reference value for numerical outputs). This increases false-positive escalations to sapTaskCenter but prevents incorrect outputs from committing to the system of record while the team evaluates the new model's behavior. This is a `$vars` configuration change (Low risk tier) — it does not require workflow redeployment.

2.  **Run a compressed eval against the successor**: prioritize the boundary condition test cases — these are the scenarios most likely to expose behavioral differences. A 2-hour compressed eval sprint across the critical test cases provides a decision-basis before committing to continued operation or requesting temporary suspension.

3.  **Authorization**: model configuration changes under this procedure require authorization from the customer IT admin (Topology A) or CIS T&O operator (Topology B). Do not make emergency configuration changes without documented consent. Use the CIS T&O operating log as the record.

4.  **Customer communication**: notify the customer's Business Lead and IT Admin that an unexpected model change has occurred, that conservative guardrails are active (meaning more HITL approvals will be required until the successor is validated), and the estimated timeline for full validation. Do not withhold this — customers should not discover unexpected behavior changes before they receive the notification.

5.  **Restore normal operation**: once the compressed eval confirms that the successor model meets the pre-deprecation success rate baseline, restore the LLM critic to the calibrated tolerance, update the version stack with the new pinned model reference, and document the change in the audit trail and operating log.


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_cost_and_token_monitoring"/>

## Cost and Token Monitoring

### CALM Monitoring Baseline at Customer Sites

> ### Note:  
> CALM-based monitoring cannot be assumed to be available in customer environments and must not be used as the default production monitoring baseline for CAE (Cloud Application and Extension) deployments.

PSR update CISATQ-131 (2026-09-09) formally documents this constraint:

-   "CALM usage at customer cannot be enforced; project-specific requirement needed"
-   OS-222 (Business Process Content) and OS-224 (Implementation Task Content) are delivered only by standard SAP products, not custom one-off developments
-   SEC-379 (Support Central API for Customer Managed Security Settings) is not applicable for CAE — CIS solutions are not Tier-1/Tier-2

**Practical implication**: the production monitoring baseline for Industry AI / CAE agents must be agreed as a project-specific requirement with each customer during FDE Discovery. The default is a combination of:

-   The agent's built-in OTel telemetry (see the BTP Deployment Patterns section below)
-   Whatever monitoring infrastructure the customer already has available
-   Cost and quality signals surfaced by Supervisor Workflow B

Do not plan a production operations model that assumes CALM is present at the customer site unless the customer's IT team has explicitly confirmed it and agreed to operate it for the engagement's agents. This agreement must be documented in the Draft Product Design Specs.

**Customer-centric observability**: when CALM is deployed and configured for AI agent telemetry ingestion, the agent's operational metrics can be surfaced directly to the customer and their CSM via CALM. This is a separate view from the CIS-side Workflow B monitoring — it gives the customer team direct visibility into agent health without requiring CIS access. Confirm CALM availability during Discovery if this construct is relevant to the customer's operational model. See also [CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md).

### Synthetic Monitoring

Beyond reactive telemetry collection, **synthetic monitoring** provides active health checks of agent endpoints and platform components across regions. SAP operates three tools for this at the platform level:

-   **BTP Availability Service**: active endpoint probing for BTP-deployed services
-   **Site 24/7**: cross-region uptime monitoring for agent-accessible endpoints
-   **Monterey**: SAP internal synthetic monitoring platform for core infrastructure components

For FDE-delivered agents, synthetic monitoring of the underlying platform is a SAP-side SRE concern. The FDE team's synthetic monitoring responsibility is:

-   Confirm the agent's A2A endpoint is reachable (the A2A smoke test at step 7 of the 9-step deploy loop provides this baseline check)
-   Include a reachability verification step in the EAC → Production migration checklist confirming the agent is healthy from the external call path

### Agentic Remediation (Topology B Only)

For agents deployed in **Topology B** (SAP Managed Observability scope — SAP-managed tenant), SAP SRE operates **Agentic Remediation**: a proactive capability that detects platform-level issues affecting agent availability and takes automated remediation steps before customers notice degradation. This is an SAP-operated capability, not a CIS or customer responsibility.

For FDE engagements in Topology B, Agentic Remediation reduces the operational burden on CIS T&O for infrastructure-level incidents — the SRE layer handles platform issues proactively. CIS T&O's scope in Topology B remains application-layer quality monitoring (Workflow B signals, eval trends, human override rates), not platform maintenance.

### Two Cost Pools: Build-Time vs. Runtime

Token costs in Industry AI delivery split into two separate pools that must not be conflated. **Build-time costs** are incurred by FDE engineers using Claude Code and AI coding tools during the sprint; these are tracked via the SAP-internal Hyperspace LLM Proxy and are not visible in the customer's environment. **Runtime costs** are incurred by deployed agents processing business transactions in the customer's BAIP tenant; these are measured via AI Units (commercial billing) and OpenTelemetry token counts (operational signal). The Hyperspace Proxy is SAP-internal only and is not available for monitoring deployed agent costs. This section covers runtime cost monitoring exclusively.

### Why Token Costs Are an Operational Signal

Token consumption is not just a billing concern — it is a quality and operational signal:

-   **Runaway loops**: an agent stuck in an infinite retry loop will consume tokens exponentially. A single runaway invocation can exhaust a significant portion of a daily token budget in minutes before any quality signal fires. Token spike detection is the fastest available early-warning for this failure mode.

-   **Context expansion creep**: as agents are used in production, the grounding context passed to `sapOrchestration` tends to grow — more historical records retrieved, longer accumulated conversation context, richer tool outputs. Token count trends upward even when the agent's task has not changed. This is a slow drift that quality metrics alone do not reveal.

-   **Model change impact**: a model version change may increase output verbosity, changing output token consumption per invocation. The impact on the value case (agent cost vs. cost savings) should be trackable.

-   **FDE value case**: the value-based contract (10–30% of yearly customer benefit) is premised partly on cost reduction through automation. If LLM operational costs consume a significant fraction of the savings, the net value case changes. Token cost tracking provides the data to monitor this ratio over time.

### What to Monitor

**Table: Token Cost Monitoring Metrics**


<table>
<tr>
<th valign="top">

Metric

</th>
<th valign="top">

What It Measures

</th>
<th valign="top">

Alert Threshold

</th>
</tr>
<tr>
<td valign="top">

Input tokens per invocation

</td>
<td valign="top">

Context size; grounding data volume

</td>
<td valign="top">

> 2× p95 baseline for 5 consecutive invocations

</td>
</tr>
<tr>
<td valign="top">

Output tokens per invocation

</td>
<td valign="top">

Response verbosity; structured output completeness

</td>
<td valign="top">

> 3× p95 baseline for 3 consecutive invocations

</td>
</tr>
<tr>
<td valign="top">

Total tokens per invocation

</td>
<td valign="top">

Composite cost signal

</td>
<td valign="top">

> 3× p95 baseline (runaway indicator)

</td>
</tr>
<tr>
<td valign="top">

Cost per invocation

</td>
<td valign="top">

Token count × model price rate

</td>
<td valign="top">

> 3× p95 baseline

</td>
</tr>
<tr>
<td valign="top">

Cost per committed output

</td>
<td valign="top">

Total cost ÷ outputs successfully written to system of record

</td>
<td valign="top">

Trend upward > 30% over 30-day window

</td>
</tr>
<tr>
<td valign="top">

Cost per escalation

</td>
<td valign="top">

Total cost ÷ escalations to sapTaskCenter

</td>
<td valign="top">

High ratio = agent is consuming tokens before failing; needs review

</td>
</tr>
<tr>
<td valign="top">

Daily total cost

</td>
<td valign="top">

Budget tracking

</td>
<td valign="top">

> 150% of expected daily budget (alert IT admin)

</td>
</tr>
</table>

*p95 (95th percentile): the latency or token count below which 95% of invocations fall. Using p95 as a baseline captures normal peak behavior without being distorted by rare outliers. A threshold of "2× p95" flags invocations that are significantly beyond the normal high-end — not just slightly above average.*

### Model Price Rates

Model price rates vary by model and are subject to change. The SAP generative AI hub exposes token consumption metrics per invocation; actual billing rates are in the customer's BAIP/SAP AI subscription. For operational monitoring, use the generative AI hub token counts as the primary signal — absolute cost calculation can be done offline using the current rate card.

**SAP-hosted models** (SAP-RPT-1.5, Llama, IBM Granite) have consumption rates governed by the SAP subscription rather than per-token billing — their cost model differs from third-party-routed models. Confirm the cost model with the customer's contract before configuring cost-based alert thresholds.

### AI Units: The Commercial Billing Layer

Token consumption is the internal operational signal for cost monitoring. The commercial billing model for deployed FDE agents uses a separate unit: **AI Units per agent step**, charged through the SAP AI Units consumption model. Three tiers apply:

**Table: AI Units by Step Type**


<table>
<tr>
<th valign="top">

Step Type

</th>
<th valign="top">

AI Units / Step

</th>
<th valign="top">

Net Price Range (EUR)

</th>
</tr>
<tr>
<td valign="top">

Basic (simple lookups, single tool calls, content retrieval)

</td>
<td valign="top">

0.005

</td>
<td valign="top">

€0.013–€0.035

</td>
</tr>
<tr>
<td valign="top">

Standard (multi-tool orchestration, moderate reasoning)

</td>
<td valign="top">

0.010

</td>
<td valign="top">

€0.026–€0.070

</td>
</tr>
<tr>
<td valign="top">

Advanced (complex reasoning, multi-system orchestration, voice)

</td>
<td valign="top">

0.025

</td>
<td valign="top">

€0.066–€0.175

</td>
</tr>
</table>

Token consumption monitoring tracks the internal LLM cost signal. AI Units tracking is the commercial measurement visible in the customer's SAP subscription. Both are in scope for the FDE value case: token costs inform the SAP-side operator cost model; AI Units inform the customer-visible cost and the value case return calculation. In practice, a single agent invocation may span multiple agent steps — the step count and step tier are determined at productization when the GenAI Commercialization Template is submitted for Global Pricing approval.

**"Agent step" and "Agent Action" are the same metering unit.** The floor-pricing model calls this unit an **Agent Action**, defined as 1 LLM call + 1 simple tool invocation = 1 Agent Action. The AI Units consumption view meters the identical unit; the two terms are synonyms for one billing unit.

**Floor pricing gate timing**: floor pricing applies at commercial release, not at sprint start. The action decomposition required for the Global Pricing submission cannot be accurate until the agent design is sufficiently frozen — typically after the MVP is built and validated, around sprint week 6–8 at the earliest. Do not attempt a formal floor pricing submission at sprint start; the estimate will require complete revision. Initiate the formal submission at week 8, when the step count and step tier are stable. The FDE executive co-innovation commitment (FBA/TEA) is established at sprint kickoff; floor pricing calculation follows once the agent's capability scope is known.


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_cost_monitoring_implementation"/>

## Cost Monitoring Implementation

### Extending Supervisor Workflow B

The most practical implementation extends Supervisor Workflow B (Telemetry Monitor) to include token consumption alongside quality signals. The SAP Cloud ALM API and the generative AI hub usage API both provide per-invocation token counts.

```
[Workflow B: Schedule Trigger]
         ↓
[HTTP Request: SAP Cloud ALM API — retrieve traces from last polling window]
[HTTP Request: SAP AI Core usage API — retrieve token counts for same window]
         ↓
[Code node: join traces with token counts by correlation_id]
         ↓
[Code node: existing quality anomaly checks]
[Code node: token anomaly checks]
  - per_invocation_tokens vs. rolling_baseline → flag if > 3× for 3 consecutive
  - runaway_detection: single invocation > 5× p99 → immediate alert
  - daily_total_cost vs. daily_budget → flag if > 150%
  - cost_per_committed_output trend → flag if 30-day trend > 30% upward
         ↓
[Decision routing: same as existing Workflow B]
    ├── No anomaly → update baseline
    ├── Cost anomaly → sapTaskCenter alert to IT admin (with cost details)
    └── Runaway indicator → immediate sapTaskCenter (VERY_HIGH priority) + trigger Workflow C
```

**Runaway loop handling**: a single-invocation token spike (> 5× p99) is treated as a Critical anomaly, not a medium-priority alert. The agent should be suspended via the Agent Hub Secure & Govern capability pending investigation.

**ServiceNow as alert target**: for customers whose primary IT service management platform is ServiceNow, Workflow B alerts can be routed to ServiceNow incidents via SAP Cloud Service Management (in addition to, or instead of, `sapTaskCenter`). The primary candidates for ServiceNow routing are guardrail violations and runaway loop scenarios — these are infrastructure-level incidents that belong in the IT service management queue, not in the business process task queue. Configure the ServiceNow connector via BAIP → Cloud Service Management → ServiceNow. Confirm the customer's ITSM platform and preferred routing in FDE Discovery — include the alert target decision in the Draft Product Design Specs.

### Token Budget Hard Cap via sapOrchestration

A hard ceiling on per-call token consumption can be set in the `sapOrchestration` node via the `max_tokens` parameter (controls maximum output tokens) and by limiting the input context window size in the system prompt design. This is a last-resort protection against runaway generation:

```
sapOrchestration configuration:
  max_tokens: [set to 2× the p95 output token count observed during EAC]
```

**Caution**: set `max_tokens` high enough not to truncate legitimate outputs. An agent that hits the token ceiling mid-response will produce a malformed structured output, which the Layer 2 deterministic check will catch and escalate. This is the correct fail-safe behavior — truncation causes escalation rather than a silently wrong committed value. But an overly tight ceiling will cause excessive escalations, defeating the efficiency case for the agent.

### Audit Trail Cost Extension

The audit trail should include token consumption fields for committed outputs. This enables the cost-per-committed-output metric and supports the FDE value case tracking:

**Table: Audit Trail Token Consumption Fields**


<table>
<tr>
<th valign="top">

Additional Field

</th>
<th valign="top">

Content

</th>
</tr>
<tr>
<td valign="top">

`input_tokens`

</td>
<td valign="top">

Input token count for the primary `sapOrchestration` call in this invocation

</td>
</tr>
<tr>
<td valign="top">

`output_tokens`

</td>
<td valign="top">

Output token count for the primary `sapOrchestration` call

</td>
</tr>
<tr>
<td valign="top">

`total_tokens_invocation`

</td>
<td valign="top">

Sum of all `sapOrchestration` calls in this invocation (worker + critic)

</td>
</tr>
<tr>
<td valign="top">

`model_used`

</td>
<td valign="top">

Model identifier used in this invocation

</td>
</tr>
</table>

**ALS v3 NG — platform-level immutable audit log**: the business-focused audit trail above captures decision-quality data for CIS T&O use. In parallel, **ALS v3 NG** (Audit Log Service Next Generation) provides the platform-level immutable audit log for agent identity events, security-relevant actions, and authorization decisions — with schema-validated events and an Event Catalog. ALS v3 NG is the compliance-grade source for regulatory audits; the business audit trail is the operational quality monitoring source. Both are in scope for productization: confirm which is required for the customer's compliance framework during FDE Discovery.

### Standard Observability Middleware Integration

The recommended observability pattern for BTP-deployed agents is a drop-in Python OTel middleware package (available in Project LightSpeed at `references/middleware/`). It requires two lines of setup code:

```python
from middleware import setup_observability
setup_observability("my-agent-service")
```

Once initialized, it emits structured JSON logs on stdout and OTel traces via OTLP. The middleware resolves its export destination using the following order of preference:

1.  `OTEL_EXPORTER_OTLP_ENDPOINT` environment variable — used for custom OTLP endpoints (sovereign region deployments where Cloud Logging is unavailable)
2.  Cloud Foundry `cloud-logging` service binding — used in standard SAP regions with SAP Cloud Logging
3.  Disabled / logging-only fallback — used in CN40, canary, and any environment where neither of the above is configured; the middleware starts without error and emits to stdout only

This resolution order means the same middleware code works unchanged across local development, standard region deployment, and sovereign region deployment — no environment-specific code branches are needed.

**OTel semantic conventions**: the middleware emits traces using the GenAI semantic conventions, satisfying TR12 (OpenTelemetry with GenAI semantic conventions) without additional instrumentation.

For sovereign region deployments (CN40, NS2, KSA), set the `OTEL_EXPORTER_OTLP_ENDPOINT` to the region's approved OTLP collection endpoint. This is a deployment configuration decision, not an application code change.


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_environment_migration"/>

## Environment Migration

### The Four Environments

FDE engagements move through a predictable sequence of environments before reaching production. Each transition has a defined purpose and a distinct artifact checklist:

**Table: Deployment Environments**


<table>
<tr>
<th valign="top">

Environment

</th>
<th valign="top">

Purpose

</th>
<th valign="top">

Who Operates

</th>
</tr>
<tr>
<td valign="top">

Developer sandbox

</td>
<td valign="top">

Initial build, rapid iteration, no governance constraints

</td>
<td valign="top">

FDE Application Engineer

</td>
</tr>
<tr>
<td valign="top">

Internal test

</td>
<td valign="top">

Integration testing, eval runs, security review

</td>
<td valign="top">

FDE team (all roles)

</td>
</tr>
<tr>
<td valign="top">

Customer non-production (EAC)

</td>
<td valign="top">

Customer validation with Early Adopter Care contacts; behavioral baseline establishment

</td>
<td valign="top">

FDE team + customer IT admin

</td>
</tr>
<tr>
<td valign="top">

Customer production

</td>
<td valign="top">

Live operation; handover complete

</td>
<td valign="top">

Customer IT admin + CIS T&O

</td>
</tr>
</table>

### What Moves Between Environments — and What Does Not

**Table: Artifact Migration Rules**


<table>
<tr>
<th valign="top">

Artifact

</th>
<th valign="top">

Moves?

</th>
<th valign="top">

How

</th>
<th valign="top">

Notes

</th>
</tr>
<tr>
<td valign="top">

Joule Studio workflow definition

</td>
<td valign="top">

Yes — export

</td>
<td valign="top">

Workflow export from source env; import in target env

</td>
<td valign="top">

Credentials are NOT included in exports — re-provision per environment

</td>
</tr>
<tr>
<td valign="top">

SAP Prompt Registry entries

</td>
<td valign="top">

Yes — promote

</td>
<td valign="top">

Create new version in Registry with env tag; reference by name in workflow

</td>
<td valign="top">

Use env-tagged versions: `agent-name-prompt-dev`, `...-test`, `...-prod`

</td>
</tr>
<tr>
<td valign="top">

Eval test suite

</td>
<td valign="top">

Yes — same suite in all envs

</td>
<td valign="top">

Git-versioned alongside the concept; runs unchanged in each environment

</td>
<td valign="top">

Synthetic test data must be available in each environment

</td>
</tr>
<tr>
<td valign="top">

BAIP application registrations (agent identities)

</td>
<td valign="top">

No — re-create

</td>
<td valign="top">

New registration per environment

</td>
<td valign="top">

Each environment needs a separate identity with appropriate scoped authorizations

</td>
</tr>
<tr>
<td valign="top">

Authorization scope definitions

</td>
<td valign="top">

No — re-define

</td>
<td valign="top">

Tighter scope in production than in dev/test; prod scope reviewed and approved

</td>
<td valign="top">

Dev may have broader read access for debugging; prod should be minimum necessary

</td>
</tr>
<tr>
<td valign="top">

Threshold configuration parameters

</td>
<td valign="top">

No — re-set

</td>
<td valign="top">

Copy dev values as starting point; adjust test values based on eval results; calibrate prod values from EAC behavioral data

</td>
<td valign="top">

Never copy dev thresholds directly to production

</td>
</tr>
<tr>
<td valign="top">

HANA Cloud connection credentials

</td>
<td valign="top">

No — re-provision

</td>
<td valign="top">

New service credentials per environment

</td>
<td valign="top">

Never copy credentials between environments

</td>
</tr>
<tr>
<td valign="top">

Shared state store schema

</td>
<td valign="top">

No — re-create

</td>
<td valign="top">

DDL script creates the schema; initial state is empty in each environment

</td>
<td valign="top">

State does NOT migrate — each environment builds its own baseline from scratch

</td>
</tr>
<tr>
<td valign="top">

Audit trail table schema

</td>
<td valign="top">

No — re-create

</td>
<td valign="top">

Same DDL, empty table per environment

</td>
<td valign="top">

Audit records from lower environments are NOT migrated to production

</td>
</tr>
<tr>
<td valign="top">

Supervisor Workflow B baseline

</td>
<td valign="top">

No — rebuild

</td>
<td valign="top">

Allow 48–72 hours of observation in new environment before enabling anomaly alerts

</td>
<td valign="top">

Imported baseline from prior environment is invalid in new environment

</td>
</tr>
</table>

### The Three Things That Must Never Be Copied Between Environments

1.  **Credentials and API keys**: a credential that works in the developer sandbox gives access to a non-production SAP system. Reusing it in production connects the agent to the wrong system. Every environment has its own service credentials — provisioned, not copied.

2.  **Dev/test thresholds directly to production**: thresholds used during development are intentionally permissive (to allow testing edge cases without constant escalation). Production thresholds should be calibrated from the EAC behavioral baseline — the first 2–4 weeks of observation in the customer's non-production environment under real-world conditions. Using dev thresholds in production produces the wrong escalation rate from day one.

3.  **Agent identities (BAIP application registrations)**: each environment's agent identity is registered in that environment's BAIP tenant with authorizations scoped to that environment's systems. Reusing a dev identity in production means the agent is authorized against the wrong system registrations.


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_environment_specific_config"/>

## Environment-Specific Configuration Pattern

The n8n workflow definition should be environment-agnostic. All environment-specific values should be injected at runtime through:

-   **Named BAIP credential objects** (for system connections, API keys, service bindings)
-   **n8n workflow variables (`$vars`)** for static, per-environment values such as thresholds, model selection, and recipient IDs — set in the n8n Variables UI without redeployment. In Joule Studio's managed n8n, `$vars` availability may depend on what SAP exposes in the UI.
-   **Trigger-payload config (`$json.config.x`)** for per-invocation values — the workflow's trigger/webhook body carries a `config` object that the caller (Joule, another agent, a scheduled webhook) provides at invocation time.

> ### Note:  
> `{{config.x}}` is **not a valid n8n expression** — it resolves to `undefined` at runtime. The correct syntax is `{{$vars.x}}` for instance-level variables or `{{$json.config.x}}` for trigger-payload config.

```
sapOrchestration configuration (environment-agnostic):
  model:          {{$vars.modelSelection}}              ← n8n variable, set in Variables UI
  systemPrompt:   {{$vars.systemPrompt}}                ← n8n variable (env-specific prompt text)
  connection:     {{credentials.sap_system_connection}} ← from BAIP credential object
  [future: promptRegistry: {{$vars.promptVersion}}  ← when promptTemplate source ships]

sapTaskCenter configuration (environment-agnostic):
  recipients:     {{$vars.approvalRecipients}}          ← n8n variable (env-specific user IDs)

Threshold parameters:
  approval_threshold:  {{$vars.approvalThreshold}}      ← n8n variable (calibrated per env)
  tolerance_band:      {{$vars.toleranceBand}}          ← n8n variable
```

**Implementation**: establish the naming convention for `$vars` keys in Discovery (e.g., `agentName_threshold_approval`) and document all environment-specific values in the production handover package with their owners. When the n8n Variables UI is restricted in Joule Studio's managed runtime, use the trigger-payload pattern (`$json.config.x`) as the fallback — the caller supplies configuration at invocation time.

### Environment-Specific Values That Change Per Environment

**Table: Environment-Specific Configuration Values**


<table>
<tr>
<th valign="top">

Parameter

</th>
<th valign="top">

Dev

</th>
<th valign="top">

Internal Test

</th>
<th valign="top">

EAC (Non-Prod)

</th>
<th valign="top">

Production

</th>
</tr>
<tr>
<td valign="top">

SAP system endpoint

</td>
<td valign="top">

Dev S/4HANA

</td>
<td valign="top">

Test S/4HANA

</td>
<td valign="top">

Customer non-prod S/4HANA

</td>
<td valign="top">

Customer prod S/4HANA

</td>
</tr>
<tr>
<td valign="top">

Approval threshold

</td>
<td valign="top">

Permissive (for testing)

</td>
<td valign="top">

Mid-range (for eval coverage)

</td>
<td valign="top">

Calibrated from initial observation

</td>
<td valign="top">

Calibrated from EAC baseline

</td>
</tr>
<tr>
<td valign="top">

Prompt Registry version

</td>
<td valign="top">

`dev` tagged version

</td>
<td valign="top">

`test` tagged version

</td>
<td valign="top">

`uat` or `v1.0-rc`

</td>
<td valign="top">

`v1.0` or later

</td>
</tr>
<tr>
<td valign="top">

Task Center recipient

</td>
<td valign="top">

FDE engineer userId

</td>
<td valign="top">

FDE team group

</td>
<td valign="top">

Customer process owner (UAT)

</td>
<td valign="top">

Customer process owner (prod)

</td>
</tr>
<tr>
<td valign="top">

Anomaly detection thresholds

</td>
<td valign="top">

Not active in dev sandbox

</td>
<td valign="top">

Conservative (broad detection for testing)

</td>
<td valign="top">

Calibrated after 48h observation

</td>
<td valign="top">

Calibrated from EAC baseline

</td>
</tr>
<tr>
<td valign="top">

Audit trail retention

</td>
<td valign="top">

Short (dev data)

</td>
<td valign="top">

Medium (test cycles)

</td>
<td valign="top">

Per regulatory requirement (EAC data may be real)

</td>
<td valign="top">

Per regulatory requirement (confirmed with compliance team)

</td>
</tr>
</table>


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_pre_migration_checklist"/>

## Pre-Migration Checklist

Before each environment transition, verify:

### Dev → Internal Test

-   All eval test cases pass in dev with documented acceptance thresholds
-   Credential handling reviewed: no hardcoded secrets in any workflow node
-   BAIP application registration created for internal test environment
-   Authorization scope reviewed and tightened from dev (remove any debug-scope permissions)
-   Prompt Registry entries tagged for test environment
-   Shared state store schema deployed in test environment (empty initial state)
-   Audit trail table schema deployed in test environment

### Internal Test → EAC (Customer Non-Production)

-   Full eval suite passes with production-equivalent data (synthetic or TEA-covered real data)
-   Security concept documentation complete and DPCE review initiated (can run in parallel with EAC)
-   New BAIP application registrations for EAC environment (separate from internal test)
-   Customer IT admin has received and confirmed role assignments in Joule Studio
-   Legal agreement confirmed: CA required if EAC uses personal or productive customer data
-   Supervisor Workflow B deployed and tested in EAC environment (inject test anomalies to verify)
-   Task Center recipients configured with actual customer process owner user IDs
-   Threshold configuration initialized (starting from test values; will be recalibrated after 48h observation)
-   Runaway detection enabled (token spike alert via Workflow B)

### EAC → Production

-   EAC behavioral baseline captured (48–72h minimum observation period complete)
-   **Go-live semantic quality baseline captured** (if drift monitoring is in scope): multi-dimensional quality snapshot taken across the final two weeks of EAC — task completion rate, escalation rate, LLM-judge faithfulness scores, and output embedding centroid, stored as a signed per-customer-agent artifact in SAP AI Core. This baseline is the reference point for all post-deploy drift detection. Requires minimum 500 production interactions for statistical validity; confirm expected interaction volume in Discovery before committing to drift monitoring in the engagement scope.
-   Production thresholds calibrated from EAC baseline and agreed with process owner
-   Sirius productization tasks complete: DPCE threat model, security concept, QPA (Quality, Privacy and AI) test strategy, AI ethics assessment
-   Production BAIP application registrations created with minimum-necessary scopes
-   Prompt Registry entries tagged `v1.0` (or equivalent release version)
-   Audit trail retention policy confirmed with customer compliance team
-   Supervisor Workflow B anomaly baselines reset (will rebuild from production observations)
-   If Workflow B routes to Cloud ALM: Cloud ALM LMS service registration complete (`serviceId` obtained) and `CALM_SERVICE_ID` configured as env var in BTP Cockpit for runtime injection
-   CIS T&O operational runbook delivered as part of handover package. See [CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md).
-   Production handover documentation complete (agent authorization matrix, threshold configuration ownership, escalation paths)


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_fde_delivery_implications"/>

## FDE Delivery Implications

### During Discovery

-   Confirm how many environments exist in the customer's landscape and whether a non-production tenant for EAC is available. If not, plan for synthetic data-only EAC and address the implications for baseline calibration.
-   Identify the customer IT admin who will manage BAIP application registrations and threshold configuration in production. This person should be engaged in Discovery, not introduced at handover.
-   Define the cost monitoring responsibility post-handover: who reviews the daily token cost report, what threshold triggers a call to CIS T&O?
-   If the engagement will route Workflow B telemetry to Cloud ALM: confirm early that the customer has Cloud ALM provisioned and that the IT admin can complete the 5-step LMS service registration (register service → obtain `serviceId` → configure destination → assign agent → validate ingestion). The `serviceId` is tenant-specific and assigned by Cloud ALM — it cannot be predetermined and must be injected at deployment time (as `CALM_SERVICE_ID` env var or trigger payload). Confirm in Discovery to avoid a late-sprint blocker at EAC go-live.
-   Decide in Discovery whether **post-deploy semantic drift monitoring** is in scope for the engagement. If yes: (a) OTel instrumentation including `gen_ai.conversation.id` trace correlation (TR12) must be active from sprint week 1 — drift detection requires this linkage across traces; (b) plan the EAC stabilization window to reach the minimum 500 production interactions required for a statistically valid go-live baseline; (c) agree with the customer IT admin who will receive and act on drift alerts post-handover.

### During Develop

-   The FDE sprint has a hard cap of **300 Person Days (PD) and 90 calendar days**. Environment migration tasks (deploy to internal test, deploy to EAC) must be planned within this budget — they are not free activities and should appear explicitly in the sprint plan.
-   Build the configuration parameter framework from Week 0, not as a post-MVP activity. Every environment-specific value should be a named parameter from the first workflow version. Retrofitting this pattern into a workflow that has hardcoded values is expensive and error-prone.
-   Include environment migration in the sprint plan as explicit tasks: "deploy to internal test" and "deploy to EAC" are not automatic consequences of code being ready — they each require the checklist above.
-   Run the eval suite in each environment, not just in dev. An eval failure in the internal test environment before customer EAC access is much less disruptive than a failure discovered in EAC.
-   Start **AI Onboarding (AI101 task)** early in Development — the AI Feature ID is required before EAC launch and the onboarding process takes several weeks. Deferring it to the end of the sprint risks blocking EAC go-live.

### During Productization

**Productization starts at RTC EAC (first go-live).** The previous model where productization begins at FDE Sprint End is replaced: productization is now triggered by RTC EAC (the customer's first go-live with Early Adopter Care contacts). This means the productization track must be prepared and the formal submission initiated during the Develop phase — not deferred to after EAC completion.

-   Productization takes **15 days to 3 months** from RTC EAC to GA, depending on the GA path:
    -   **Microdelivery** (new feature in an existing product): LoB leads the release; faster release cycle; LoB Program Manager owns Sirius
    -   **Independent Release** (new product, no existing program): Application AI leads end-to-end; standard release with full Sirius task set
    The path is determined at the FDE Gate by Application AI reviewing scenarios with the LoB. Both paths require full Sirius task completion — the difference is ownership and cycle time, not the quality gate.
-   The production handover package must include the complete configuration parameter reference: every named parameter, its current value, who owns it, and the process for changing it (approval required? logged where?).
-   Version control for prompts and thresholds is a governance requirement, not an engineering nice-to-have. Sirius productization requires an audit trail for all changes to production configuration — Prompt Registry versioning (when available) and the n8n Variables UI change history provide this audit trail if maintained correctly.
-   Document the rollback procedure explicitly in the T&O operational runbook: what triggers a rollback, who authorizes it, the step-by-step mechanics, and the post-rollback verification steps. A rollback that happens for the first time during a production incident is not the time to figure out the procedure.

-   **Adoption Strategy** is a required handover artifact alongside the technical runbook. Before productization handover is signed off, the Adoption Strategy must be complete: all 7 sections drafted, all 9 governance roles named, and the Executive Sponsor signature obtained. A production handover without an Adoption Strategy has no organizational accountability structure for what happens when adoption stalls. See [CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md).
-   **Enable track exit criteria** must be met before handover is complete. At minimum, the customer IT admin must have reached Stage 3 (Reverse Shadowing) — they have demonstrated the ability to operate the agent independently, with coaching available. Stage 4 (Independent Operation) is the target. Handing over an agent to a team that has never operated it without the FDE team present is a support risk, not a handover. See [CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md).


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_production_shipping_readiness"/>

## Production Shipping Readiness — TR1–TR14

Every agent developed in an FDE sprint that will be productized as a commercial SAP offering must satisfy the AI Golden Path's 14 mandatory Technical Requirements before commercial release. These are not optional and cannot be deferred to post-launch. The FDE Development phase should build toward TR1–TR14 compliance from sprint week 1; retroactive compliance is expensive and often requires architectural rework.

### The Checklist

**Table: TR1–TR14 Technical Requirements**


<table>
<tr>
<th valign="top">

TR

</th>
<th valign="top">

Requirement

</th>
<th valign="top">

FDE Implication

</th>
</tr>
<tr>
<td valign="top">

TR1

</td>
<td valign="top">

A2A Server exposure

</td>
<td valign="top">

Agent must expose an A2A protocol endpoint. Required for eval harness integration and ecosystem interoperability

</td>
</tr>
<tr>
<td valign="top">

TR2

</td>
<td valign="top">

AppFND (Application Foundation) runtime

</td>
<td valign="top">

Agents run on BTP Kyma via AppFND. A formal exception process exists but requires justification

</td>
</tr>
<tr>
<td valign="top">

TR3

</td>
<td valign="top">

IAS app2app tokens

</td>
<td valign="top">

No API keys or username/password for agent-to-agent calls. IAS app2app tokens throughout

</td>
</tr>
<tr>
<td valign="top">

TR4

</td>
<td valign="top">

Unified Services onboarding

</td>
<td valign="top">

SAP-managed provisioning via Unified Metadata Service (UMS). Required for Agent Gateway routing

</td>
</tr>
<tr>
<td valign="top">

TR5

</td>
<td valign="top">

UCL onboarding

</td>
<td valign="top">

SAP-managed integrations. Required for production deployment

</td>
</tr>
<tr>
<td valign="top">

TR6

</td>
<td valign="top">

ORD with `sap:ai:v1` policy level

</td>
<td valign="top">

ORD endpoint published with AI policy level; agent namespace must use `sap.agt*` pattern. Without this, the agent is invisible to Agent Gateway

</td>
</tr>
<tr>
<td valign="top">

TR7

</td>
<td valign="top">

SPII for Agent Gateway

</td>
<td valign="top">

Bi-directional IAS app2app trust for Agent Gateway. Required for tool invocation routing

</td>
</tr>
<tr>
<td valign="top">

TR8

</td>
<td valign="top">

BTP Test Blueprint

</td>
<td valign="top">

Test blueprint covering functional, security, and performance validation

</td>
</tr>
<tr>
<td valign="top">

TR9

</td>
<td valign="top">

Offline + online evaluations (aeval)

</td>
<td valign="top">

Offline eval suite in CI before each release; online eval monitoring active in production

</td>
</tr>
<tr>
<td valign="top">

TR10

</td>
<td valign="top">

Agent Gateway for all tool/agent calls

</td>
<td valign="top">

Every tool invocation and agent-to-agent call routes through Agent Gateway — no direct external API calls bypass governance

</td>
</tr>
<tr>
<td valign="top">

TR11

</td>
<td valign="top">

MCP servers for tool calls

</td>
<td valign="top">

All tool calls use MCP protocol. `sapMcpClient` in Joule Studio satisfies this for SAP system connections

</td>
</tr>
<tr>
<td valign="top">

TR12

</td>
<td valign="top">

OpenTelemetry with GenAI semantic conventions

</td>
<td valign="top">

Mandatory, not recommended. Enables token monitoring, evaluation metrics, and the audit trail

</td>
</tr>
<tr>
<td valign="top">

TR13

</td>
<td valign="top">

Metering via OctoRoute + Unified Metering Service

</td>
<td valign="top">

Revenue-critical. Origin propagated via `x-sap-origin` header; `sap.ai.agent.trigger.type` in metering payload. Origin propagates transitively through agent-to-agent calls

</td>
</tr>
<tr>
<td valign="top">

TR14

</td>
<td valign="top">

Extensibility support

</td>
<td valign="top">

Every production agent must support Prompt/Tool Extensions, Pre-Extension Hooks, and Post-Extension Hooks with configurable timeout and circuit-breaker failure policies

</td>
</tr>
</table>

### Sprint Integration Points

TR1–TR14 compliance is a design constraint from week 1, not a productization-phase check:

**Table: TR Readiness by Sprint Phase**


<table>
<tr>
<th valign="top">

Sprint Phase

</th>
<th valign="top">

TR Readiness Targets

</th>
</tr>
<tr>
<td valign="top">

Week 0 — Architecture

</td>
<td valign="top">

A2A endpoint design (TR1); AppFND runtime confirmed (TR2); ORD namespace reserved (TR6)

</td>
</tr>
<tr>
<td valign="top">

Weeks 1–6 — MVP

</td>
<td valign="top">

IAS app2app tokens configured (TR3); OTel instrumentation active (TR12); Agent Gateway routing established (TR10, TR11)

</td>
</tr>
<tr>
<td valign="top">

Weeks 7–9 — Pre-deployment

</td>
<td valign="top">

UMS/UCL onboarding started (TR4, TR5); offline eval in CI (TR9); metering payload implemented (TR13); extensibility hooks defined (TR14)

</td>
</tr>
<tr>
<td valign="top">

Weeks 9–12 — EAC

</td>
<td valign="top">

SPII configured (TR7); BTP Test Blueprint complete (TR8); online eval active (TR9); TR self-assessment signed off

</td>
</tr>
</table>

**Floor pricing approval** is the commercial counterpart to TR1–TR14: mandatory before every commercial release, requiring LLM cost, infrastructure/app management cost, and tool cost estimates. Initiate no later than week 8 of the sprint — it is not a fast process. Agent Actions (1 LLM call + 1 simple tool invocation = 1 Agent Action) are the metering unit; accurate action decomposition is needed for the floor pricing submission. An Agent Action is the same metering unit as the **agent step** — the two terms are synonymous.

### AI Agent Hub Production Gate

For all FDE-delivered agents proceeding through the SAP productization track, **AI Agent Hub approval is a mandatory production gate** before the agent enters customer production. This gate is distinct from TR1–TR14 compliance: TR compliance addresses technical readiness; AI Agent Hub approval addresses governance registration, agent identity provisioning, and lifecycle enrollment under the SAP AI Agent Hub's Plan & Build → Discover & Provision → Observe & Analyze → Secure & Govern → Optimize & Decommission lifecycle.

FDE teams should initiate AI Agent Hub registration no later than week 9, alongside TR7 (SPII configuration) and TR8 (BTP Test Blueprint):

-   Register agent identity in the **Agent & MCP Registry**
-   Assign policy bindings and scoped authorization grants
-   Confirm tenant isolation, observability routing, and Agent Evals & Assessment enrollment

*Scope note: the AI Agent Hub production gate applies to all agent deployments — both FDE/CIS-delivered agents going through productization and customer-built Joule Studio agents. For customer-built agents, this is a self-service approval step in Joule Studio. For FDE-delivered agents, it is part of the productization handover to the customer IT admin and CIS T&O.*

### Group 1 / Group 2 Deprecation Classification

SAP's formal AI use case lifecycle policy classifies deprecated AI use cases into two groups. FDE teams must understand which classification applies when planning long-term operational commitments with customers:

**Table: Deprecation Classification Groups**


<table>
<tr>
<th valign="top">

Classification

</th>
<th valign="top">

Policy

</th>
<th valign="top">

Implication for FDE-Delivered Agents

</th>
</tr>
<tr>
<td valign="top">

Group 1

</td>
<td valign="top">

Customer continues using the deprecated capability through the end of their current cloud subscription term; no forced migration during the contract period

</td>
<td valign="top">

CIS commits to supporting the agent through the subscription term; microdeliveries continue as planned

</td>
</tr>
<tr>
<td valign="top">

Group 2

</td>
<td valign="top">

6-month transition period from deprecation announcement; migration to the replacement capability is mandatory after 6 months

</td>
<td valign="top">

CIS must plan and support a migration path; the CoE operating rhythm must include migration sprint planning within the 6-month window

</td>
</tr>
</table>

The deprecation classification is determined by SAP Product at end-of-life, not at FDE delivery time. During FDE Discovery and sprint planning:

-   Assume **Group 1** for major Industry AI domain agents (batch release, commodity hedging, asset management) — these are central to the value case and customers require continuity guarantees
-   Build the microdelivery process to accommodate migration paths, not just incremental improvements — a Group 2 classification requires customers to migrate to a new agent version within a defined window
-   Document agent capabilities, extension points, and data dependencies at productization in sufficient detail to support a Group 2 migration if a future deprecation event occurs


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_btp_deployment_patterns"/>

## BTP Deployment Patterns

### Nine-Step Autonomous Deploy Loop

The Project LightSpeed deploy pattern (`/fde-deploy`) provides a repeatable 9-step procedure for BTP Kyma agent deployments. Each step has a defined success condition; FAIL-level guardrail failures abort the loop at step 3. WARN findings are logged and the loop continues.

**Table: Nine-Step Autonomous Deploy Loop**


<table>
<tr>
<th valign="top">

Step

</th>
<th valign="top">

Action

</th>
<th valign="top">

Abort Condition

</th>
</tr>
<tr>
<td valign="top">

1

</td>
<td valign="top">

Read deployment profile (`pilot.yaml`)

</td>
<td valign="top">

Profile missing or malformed

</td>
</tr>
<tr>
<td valign="top">

2

</td>
<td valign="top">

Verify BTP environment — subaccount, entitlements, service instances

</td>
<td valign="top">

Required service instances not present

</td>
</tr>
<tr>
<td valign="top">

3

</td>
<td valign="top">

Run 21-check deploy guardrails (FAIL = abort, WARN = continue with log)

</td>
<td valign="top">

Any FAIL-level check fails

</td>
</tr>
<tr>
<td valign="top">

4

</td>
<td valign="top">

Build + push container image (3 retries on transient failure)

</td>
<td valign="top">

Build or push fails after 3 retries

</td>
</tr>
<tr>
<td valign="top">

5

</td>
<td valign="top">

Apply Kubernetes manifests (or CF push for Cloud Foundry deployments)

</td>
<td valign="top">

Apply fails; rollback initiated

</td>
</tr>
<tr>
<td valign="top">

6

</td>
<td valign="top">

Backend service deployment

</td>
<td valign="top">

Service deployment fails

</td>
</tr>
<tr>
<td valign="top">

7

</td>
<td valign="top">

A2A smoke test — verify agent responds on the A2A endpoint

</td>
<td valign="top">

A2A endpoint unreachable or returns error

</td>
</tr>
<tr>
<td valign="top">

8

</td>
<td valign="top">

Joule capability registration — connect agent to Joule

</td>
<td valign="top">

Registration fails or capability bundle rejected

</td>
</tr>
<tr>
<td valign="top">

9

</td>
<td valign="top">

Update deployment profile and generate status report

</td>
<td valign="top">

Non-blocking; logged

</td>
</tr>
</table>

The 21-check guardrail suite at step 3 includes FAIL-level checks: K8s Secret namespace colocation, `.fde/.env` gitignore enforcement, spec_sha256 mismatch, Joule config schema, plus operational checks (image tag present, resource limits defined, liveness probe configured).

**Execution modes**: the loop can be run autonomously (all steps execute with pre-approved guardrails, no human interaction required for a clean deployment) or interactively (the loop pauses at each FAIL for human resolution before continuing). The `pilot.yaml` deployment profile includes an `execution_mode` field (`autonomous` or `interactive`). Production deployments should use `interactive` until the deploy pattern has been validated in the specific customer environment at least twice.

### OTel Observability Middleware

See the Standard Observability Middleware Integration section above for the drop-in Python package setup. Region-specific deployment paths:

**Table: OTel Middleware Region Deployment Paths**


<table>
<tr>
<th valign="top">

Environment

</th>
<th valign="top">

Export Destination

</th>
<th valign="top">

Configuration

</th>
</tr>
<tr>
<td valign="top">

Local development

</td>
<td valign="top">

stdout (logging-only)

</td>
<td valign="top">

No configuration needed; graceful fallback

</td>
</tr>
<tr>
<td valign="top">

Standard SAP regions (EU, US East, AP)

</td>
<td valign="top">

SAP Cloud Logging via CF `cloud-logging` binding

</td>
<td valign="top">

Binding auto-detected; no env var needed

</td>
</tr>
<tr>
<td valign="top">

Sovereign regions (CN40, NS2, KSA)

</td>
<td valign="top">

Custom OTLP endpoint

</td>
<td valign="top">

Set `OTEL_EXPORTER_OTLP_ENDPOINT`

</td>
</tr>
<tr>
<td valign="top">

Canary / sandbox

</td>
<td valign="top">

stdout (logging-only)

</td>
<td valign="top">

Cloud Logging unavailable; graceful fallback

</td>
</tr>
</table>

The middleware's graceful fallback behaviour (logging-only when no Cloud Logging binding is present) satisfies the design requirement that agents must not fail to start when Cloud Logging is unavailable.

**SAP OTel pipeline infrastructure**: within SAP-managed regions, OTel telemetry exported to SAP Cloud Logging routes through SAP's internal observability pipeline: **OctoRoute** (intelligent routing layer), **OctoBus** (event bus for telemetry distribution), and **CaaS** (Collector-as-a-Service). This infrastructure is transparent to FDE engineers — the two-line middleware setup routes telemetry correctly without knowledge of the underlying pipeline. It becomes relevant when investigating delayed or missing telemetry in SAP-managed deployments: the issue may be in the OctoRoute/OctoBus pipeline rather than in the agent's OTel instrumentation. Raise a SAP support ticket referencing the OTel pipeline component if telemetry is confirmed absent after verifying the agent-side middleware configuration is correct.

### Dynatrace CF Integration — Scoped API Token Requirement

> ### Note:  
> When integrating Dynatrace APM via Cloud Foundry service binding, use a scoped API token to avoid 403 errors on the Dynatrace ingest endpoint.

The correct procedure:

-   Use `cf bind-service` with the Dynatrace service instance (do not configure the Dynatrace integration manually via environment variables)
-   Ensure the Dynatrace API token used for the CF service instance has the **ingest scope** (`metrics.ingest`, `logs.ingest`, `traces.ingest`) — not a full admin token

A mis-scoped token causes silent metric drop: the application starts successfully, the CF binding is visible in `cf env`, but no metrics, logs, or traces appear in Dynatrace. There is no error in the application logs — the failure is on the Dynatrace ingest side and requires checking Dynatrace audit logs to diagnose.

This is a known production failure pattern encountered in BTP CF + Dynatrace integrations. Include a Dynatrace ingest validation step in the EAC → Production migration checklist for any engagement that uses Dynatrace APM.


<a name="loioedd9e33b3f6c4879a3d8c0379eba47ca__section_summary"/>

## Summary

**Table: Key Principles Summary**


<table>
<tr>
<th valign="top">

Topic

</th>
<th valign="top">

Key Principle

</th>
</tr>
<tr>
<td valign="top">

Version stack

</td>
<td valign="top">

An agent version is five components: workflow, prompt, model, thresholds, authorization scope — update and roll back as a group

</td>
</tr>
<tr>
<td valign="top">

Rollback triggers

</td>
<td valign="top">

Confirmed regression correlated with a deployment event: human override rate, verification rejection rate, process KPI degradation, format failures

</td>
</tr>
<tr>
<td valign="top">

Rollback mechanics

</td>
<td valign="top">

Prompt rollback currently requires a workflow redeployment (inline prompts) — Registry-based no-redeploy prompt rollback is future, when `promptTemplate` ships; model rollback requires no redeployment; workflow rollback is most disruptive; full stack rollback last resort

</td>
</tr>
<tr>
<td valign="top">

State during rollback

</td>
<td valign="top">

Audit trail is immutable and version-tagged; shared state store baseline must be reset after rollback; in-flight sessions should drain before cutover

</td>
</tr>
<tr>
<td valign="top">

Token monitoring

</td>
<td valign="top">

Runaway loop indicator (single-invocation spike) is Critical severity; daily cost trend tracks value case health; extend Workflow B with token anomaly checks

</td>
</tr>
<tr>
<td valign="top">

Production shipping

</td>
<td valign="top">

TR1–TR14 must all be satisfied before commercial release; design for compliance from week 1 of Develop, not at productization

</td>
</tr>
<tr>
<td valign="top">

Token budget cap

</td>
<td valign="top">

Set max_tokens to 2× p95 output token count from EAC — prevents runaway but calibrated to not truncate legitimate outputs

</td>
</tr>
<tr>
<td valign="top">

Audit trail cost fields

</td>
<td valign="top">

Add input_tokens, output_tokens, total_tokens_invocation, model_used to support cost-per-committed-output tracking

</td>
</tr>
<tr>
<td valign="top">

Three things never to copy

</td>
<td valign="top">

Credentials, dev/test thresholds, agent identities — all must be re-provisioned per environment

</td>
</tr>
<tr>
<td valign="top">

Environment-agnostic workflow

</td>
<td valign="top">

All environment-specific values in named BAIP credential objects and configuration parameters; workflow definition is the same across environments

</td>
</tr>
<tr>
<td valign="top">

Pre-migration checklist

</td>
<td valign="top">

Three checklists: dev→test, test→EAC, EAC→production — each with specific conditions that must be met before transition

</td>
</tr>
<tr>
<td valign="top">

Rollback runbook

</td>
<td valign="top">

Write the procedure during Productization, not during the first incident; include trigger criteria, authorization path, step-by-step mechanics, and post-rollback verification

</td>
</tr>
<tr>
<td valign="top">

CALM monitoring

</td>
<td valign="top">

Cannot be mandated at customer sites for CAE deployments (CISATQ-131); production monitoring baseline is project-specific — agree in Discovery; default is OTel telemetry + Workflow B signals. If Cloud ALM is used: requires 5-step LMS service registration; the `serviceId` assigned by Cloud ALM must be injected at deployment (`CALM_SERVICE_ID` env var); OTel `service.name` is ignored — only `serviceId` routes correctly

</td>
</tr>
<tr>
<td valign="top">

OTel middleware

</td>
<td valign="top">

Two-line setup (`setup_observability`); auto-resolves to OTLP env var → CF Cloud Logging binding → logging-only fallback; works unchanged across all deployment environments

</td>
</tr>
<tr>
<td valign="top">

9-step deploy loop

</td>
<td valign="top">

Repeatable autonomous deploy procedure: read profile → verify BTP env → 21-check guardrails (FAIL = abort) → build + push → apply manifests → backend deploy → A2A smoke test → Joule registration → status report

</td>
</tr>
<tr>
<td valign="top">

Dynatrace CF binding

</td>
<td valign="top">

Use scoped API token (ingest scope only), not admin token; mis-scoped token causes silent metric drop with no application-side error

</td>
</tr>
<tr>
<td valign="top">

Synthetic monitoring

</td>
<td valign="top">

Active health checks at platform level via BTP Availability Service, Site 24/7, Monterey (SAP-operated); FDE responsibility: A2A endpoint smoke test (step 7 of deploy loop) + endpoint reachability check in EAC→prod migration checklist

</td>
</tr>
<tr>
<td valign="top">

Agentic Remediation

</td>
<td valign="top">

SAP-operated proactive remediation of platform-level agent issues (Topology B only); reduces CIS T&O infrastructure overhead; application-layer quality monitoring remains CIS T&O scope

</td>
</tr>
<tr>
<td valign="top">

ServiceNow

</td>
<td valign="top">

Route Workflow B guardrail/runaway alerts to ServiceNow incidents via Cloud Service Management for ITSM-focused customers; confirm alert target in Discovery; include in Draft Product Design Specs

</td>
</tr>
<tr>
<td valign="top">

ALS v3 NG

</td>
<td valign="top">

Platform-level immutable audit log (Audit Log Service Next Gen) for agent security events and authorization decisions; complements the business-focused audit trail; confirm which is required for customer's compliance framework in Discovery

</td>
</tr>
<tr>
<td valign="top">

OTel pipeline (SAP internal)

</td>
<td valign="top">

OctoRoute / OctoBus / CaaS routes SAP Cloud ALM telemetry in SAP-managed regions; transparent to FDE; relevant when diagnosing missing telemetry — raise SAP support ticket if agent-side OTel config is confirmed correct but telemetry is absent

</td>
</tr>
<tr>
<td valign="top">

Customer-centric observability

</td>
<td valign="top">

When CALM is deployed: agent telemetry surfaced to customer and CSM via CALM; separate from CIS-side Workflow B monitoring; cannot be mandated; confirm in Discovery

</td>
</tr>
</table>

**Related Information**  




[Code-Based Agents (Bring Your Own Agent)](https://help.sap.com/docs/JOULE/ba88d1ec6a1b442098863d577c19b0c0/2f9701e60fd24e948c2c74fe9e55ce23.html "How to register a custom code-based agent (BYOA) to Joule via A2A.") :arrow_upper_right:
[Observe and Analyze — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/1fdc1df35d414cd6bf2ed15ec8a164f4.html "Monitor agent usage, measure business value, and analyze portfolio health.") :arrow_upper_right:
[CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md)

[SAP Managed n8n Workflows](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/5f16a77e21d047dc968de9f7e83bca54.html "SAP Managed n8n Workflows documentation") :arrow_upper_right:

[SAP AI Agent Hub — Plan and Build](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/e2a29b0c422e4aceadee658f4bbab71b.html "SAP AI Agent Hub Plan and Build documentation") :arrow_upper_right:

[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "SAP Joule Studio product documentation") :arrow_upper_right:
