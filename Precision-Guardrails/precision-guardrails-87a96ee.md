<!-- loio87a96eeabb444fd7a0fad07c8547671f -->

# Precision Guardrails for Numerical Agent Outputs

Numerical outputs from AI agents carry a distinct risk profile: errors in values committed to systems of record can trigger regulatory deviations, financial losses, and schedule disruptions. This topic describes the four-layer precision guardrail stack — pre-check, output validation, human approval thresholds, and audit trail — and explains how to configure and deliver it in Industry AI FDE engagements.



<a name="loio87a96eeabb444fd7a0fad07c8547671f__section_why_numerical_outputs_highest_risk"/>

## Why Numerical Outputs Are the Highest-Risk Agent Output

Most AI agent failures are visible and recoverable: a wrong summary can be corrected, a misclassified ticket can be rerouted, a poorly worded draft can be rewritten. When an agent produces a wrong number and that number is committed to a system of record, the consequences are different in kind.

A batch release quantity that is off by a decimal point dispatches the wrong amount of material. A hedge ratio with an inverted sign creates an unintended exposure. A pricing recommendation with the wrong currency unit generates revenue losses that take weeks to surface. In regulated industries, a numerical error in a batch release decision can trigger a regulatory deviation that requires investigation, documentation, and potential product recall.

The risk is compounded by two properties of LLMs:

**LLMs are not calculators.** They produce numerically plausible outputs — numbers that look correct, appear in the right range, and are formatted appropriately. But plausibility is not accuracy. An LLM can produce 12,500 kg when the correct figure is 1,250 kg, and the output will be formatted, labelled, and presented with the same confidence as a correct answer.

**Errors are not uniformly distributed.** LLM numerical errors cluster around patterns that are hard to detect without a reference check: unit confusion (kg vs. metric ton), magnitude errors (10x, 100x), sign errors (positive vs. negative position), and rounding behavior that differs from the system of record's expected precision.

Precision guardrails are the design layer that intercepts numerical outputs before they become committed business facts. They are not generic output validation applied to numbers — they are a purpose-built protection stack for high-stakes quantitative decisions.


<a name="loio87a96eeabb444fd7a0fad07c8547671f__section_domain_examples_industry_ai"/>

## Domain Examples by Industry AI Domain

The following table maps representative numerical outputs to their failure modes and downstream consequences for the most affected Industry AI domains:

**Table: Numerical Output Failure Modes by Industry AI Domain**


<table>
<tr>
<th valign="top">

Industry AI Domain

</th>
<th valign="top">

Numerical Output

</th>
<th valign="top">

Failure Mode

</th>
<th valign="top">

Consequence

</th>
</tr>
<tr>
<td valign="top">

Autonomous Regulated Manufacturing

</td>
<td valign="top">

Batch release quantity (kg, units, volume)

</td>
<td valign="top">

Unit confusion: kg vs. metric ton

</td>
<td valign="top">

Regulatory deviation; incorrect shipment quantity; potential recall

</td>
</tr>
<tr>
<td valign="top">

Autonomous Regulated Manufacturing

</td>
<td valign="top">

Deviation classification threshold

</td>
<td valign="top">

Magnitude error: 0.01 misread as 0.1

</td>
<td valign="top">

False deviation approval; missed genuine deviation

</td>
</tr>
<tr>
<td valign="top">

Autonomous Commodity Management

</td>
<td valign="top">

Hedge ratio

</td>
<td valign="top">

Sign error: long/short inversion

</td>
<td valign="top">

Unintended directional exposure; financial loss

</td>
</tr>
<tr>
<td valign="top">

Autonomous Commodity Management

</td>
<td valign="top">

Forward contract quantity

</td>
<td valign="top">

10x magnitude error

</td>
<td valign="top">

Over-hedged position; locked-in losses

</td>
</tr>
<tr>
<td valign="top">

Autonomous Revenue Growth Management

</td>
<td valign="top">

Pricing recommendation

</td>
<td valign="top">

Currency unit confusion (e.g., EUR vs. USD)

</td>
<td valign="top">

Systematic under- or over-pricing

</td>
</tr>
<tr>
<td valign="top">

Autonomous Revenue Growth Management

</td>
<td valign="top">

Discount percentage

</td>
<td valign="top">

Rounding: 0.15 applied as 15%

</td>
<td valign="top">

Margin erosion at scale

</td>
</tr>
<tr>
<td valign="top">

Autonomous Asset Management

</td>
<td valign="top">

Maintenance cost estimate

</td>
<td valign="top">

Wrong order of magnitude

</td>
<td valign="top">

Budget overrun; deferred maintenance misclassification

</td>
</tr>
<tr>
<td valign="top">

Autonomous Adaptive Production

</td>
<td valign="top">

Production order quantity

</td>
<td valign="top">

Integer vs. float: 1.5 batches vs. 1 or 2

</td>
<td valign="top">

Schedule disruption; material over-ordering

</td>
</tr>
<tr>
<td valign="top">

Autonomous Project Delivery

</td>
<td valign="top">

Milestone cost at completion

</td>
<td valign="top">

Percentage vs. absolute value confusion

</td>
<td valign="top">

Incorrect budget forecast; contract exposure

</td>
</tr>
</table>

Common thread: the errors that matter most are not random noise — they are systematic confusions about units, magnitude, and sign that a deterministic reference check can reliably detect.


<a name="loio87a96eeabb444fd7a0fad07c8547671f__section_the_guardrail_stack"/>

## The Guardrail Stack

### FAIL / WARN Guardrail Classification

Guardrail checks use a two-tier classification that governs whether a failing check blocks the operation or merely flags it for review:

**Table: Guardrail Check Classification**


<table>
<tr>
<th valign="top">

Classification

</th>
<th valign="top">

Behaviour

</th>
<th valign="top">

Used When

</th>
</tr>
<tr>
<td valign="top">

**FAIL**

</td>
<td valign="top">

Blocking — the operation is halted; the issue must be resolved before proceeding

</td>
<td valign="top">

Proceeding would cause incorrect behaviour, security failure, or compliance violation

</td>
</tr>
<tr>
<td valign="top">

**WARN**

</td>
<td valign="top">

Advisory — the operation may proceed; the finding is logged and flagged for review

</td>
<td valign="top">

The condition is sub-optimal but does not prevent correct operation

</td>
</tr>
</table>

This classification allows the guardrail system to catch hard blockers while still surfacing softer quality issues without blocking delivery. WARN findings accumulate in the deployment record and should be reviewed at the sprint retrospective.

**Example FAIL conditions** (from the Project LightSpeed deploy guardrails):

-   Credential Secret not in the same namespace as the consuming pod
-   `spec_sha256` mismatch (generation drift — built artifact does not match the spec that was approved)
-   Joule capability config schema violation
-   Credential file not in `.gitignore`

**Example WARN conditions**:

-   Routing description below quality threshold (advisory — agent may route incorrectly under ambiguous user phrasing)
-   Test coverage below target (advisory — does not block deploy but flags risk)

This two-tier model applies to both deploy-time guardrail checks and to runtime output validation. The FAIL/WARN distinction is a design decision made during FDE Discovery for each check — not a property of the check framework itself.

### The Priority: Use Deterministic Tools Where They Exist

The guardrail stack described below is the safety net for cases where LLM reasoning is unavoidable. Before reaching for the stack, apply the upstream design principle: **if a deterministic SAP service can compute the answer authoritatively, the agent should call that service as a tool and use its output — not ask the LLM to reason to the same number.**

The LLM's role in a well-designed numerical agent is to provide context, framing, and judgment ("does this price make sense given the customer relationship and current market conditions?"), not to perform the arithmetic. This maps directly to a Joule Studio build decision:

**Table: Task Type to Correct Node Mapping**


<table>
<tr>
<th valign="top">

Task Type

</th>
<th valign="top">

Correct Node

</th>
<th valign="top">

Do NOT Use

</th>
</tr>
<tr>
<td valign="top">

Numerical calculation with a SAP service equivalent

</td>
<td valign="top">

`sapMcpClient` → authoritative SAP service (pricing engine, batch classification, valuation model, MRP)

</td>
<td valign="top">

`sapOrchestration`: LLM computing the number

</td>
</tr>
<tr>
<td valign="top">

Lookup from a defined authoritative source

</td>
<td valign="top">

`sapMcpClient` → lookup API

</td>
<td valign="top">

LLM retrieval from training data or RAG

</td>
</tr>
<tr>
<td valign="top">

Business rule evaluation with a rules engine

</td>
<td valign="top">

`sapMcpClient` → rules service

</td>
<td valign="top">

LLM rule interpretation

</td>
</tr>
<tr>
<td valign="top">

Contextual judgment — "does this make business sense?"

</td>
<td valign="top">

`sapOrchestration` with a critic prompt

</td>
<td valign="top">

Rigid rule-based check

</td>
</tr>
</table>

An agent that routes numerical computation to deterministic services will require a narrower, more reliable guardrail stack than one that relies on LLM arithmetic. The four layers below assume this tool-selection principle has already been applied — they are not a substitute for it.

### Input Data Quality Assumption

The four layers below validate agent *outputs*. They assume that the authoritative data retrieved in Layer 1 (constraints from `sapMcpClient`) is itself correct.

If the source system contains incorrect master data — wrong standard batch quantities, stale approved vendor lists, out-of-date pricing tiers — the agent will produce an output that is internally consistent with the bad input it received and may pass all four guardrail layers. The audit trail's `input_state` field records what the agent received for forensic purposes; it does not validate whether that input was accurate upstream.

Source data quality is a prerequisite for guardrail effectiveness, not something the stack provides. During FDE Discovery, identify the `sapMcpClient` calls that retrieve authoritative constraints, confirm the currency and accuracy of the underlying master data with the process owner, and flag any known data quality risks as dependencies in the Draft Product Design Specs.

### The Stack

Precision guardrails operate as a four-layer stack. Each layer addresses a different point in the agent's decision process and a different failure mode.

```
Layer 1 — Pre-Check
  Agent receives only valid input space (constrained before reasoning begins)
         ↓
Layer 2 — Output Validation
  Agent's numerical output verified before it can reach write-back
  (deterministic check + LLM critic where appropriate)
         ↓
Layer 3 — Human Approval Threshold
  Outputs above defined value/risk threshold require human sign-off
         ↓
Layer 4 — Audit Trail
  All committed numerical decisions recorded with full context
```

The layers are cumulative, not alternative. A high-stakes numerical output should pass through all four layers. A lower-stakes output (e.g., a cost estimate in an advisory workflow where no automatic action follows) may only need layers 2 and 4. The appropriate stack for each use case is defined during FDE Discovery.

### Business Rule Constraints — MUST Rules in `golden_scenarios.yaml`

For agents that execute business-critical actions (batch releases, financial postings, purchase orders), business constraints should be encoded as explicit MUST rules in a `golden_scenarios.yaml` file. Each MUST rule is a natural-language business invariant that the agent must satisfy:

```yaml
# golden_scenarios.yaml — example MUST rules
must_rules:
  - id: MR-001
    description: "Agent must always display the quantity and unit of measure before
                  confirming a batch release"
    severity: FAIL
  - id: MR-002
    description: "Agent must never post a financial document without displaying the
                  vendor name"
    severity: FAIL
  - id: MR-003
    description: "Agent must confirm the material number matches the purchase order
                  line item before writing back"
    severity: FAIL
  - id: MR-004
    description: "Agent should display the tolerance band used for validation when
                  presenting a recommendation to a human approver"
    severity: WARN
```

MUST rules are evaluated as LLM-graded guardrails during post-build evaluation and can be included in the aeval offline test suite. This transforms informal business requirements captured during FDE Discovery into machine-checkable constraints.

Each MUST rule has an associated severity (`FAIL` or `WARN`). A `FAIL`-severity MUST rule whose condition is not satisfied in an eval test case blocks the deployment. A `WARN`-severity MUST rule produces an advisory finding.

The `golden_scenarios.yaml` file is a collaboration artifact: the Domain PM and process owner define the MUST rule content (the business invariant); the Dev Architect assigns the severity and writes the eval test case that exercises it. Both must agree on the rule definition before the eval suite is run against the first MVP build.

### Input Compression — Unsafe for Numerical and Financial Content

> ### Note:  
> Input compression must not be applied to prompts containing numerical or financial content without explicit validation.

ADR 011 (SAP-internal benchmark, 30 enterprise cases, 2026-07-26) provides the authoritative operating-point data for LLM-based input compression:

**Table: Input Compression Technique Comparison**


<table>
<tr>
<th valign="top">

Technique

</th>
<th valign="top">

Retention

</th>
<th valign="top">

Input Reduction

</th>
<th valign="top">

Quality Score

</th>
<th valign="top">

Numeric Recall

</th>
</tr>
<tr>
<td valign="top">

LLMLingua

</td>
<td valign="top">

50%

</td>
<td valign="top">

~50%

</td>
<td valign="top">

0.989

</td>
<td valign="top">

**0.914 — 9% numeric drop**

</td>
</tr>
<tr>
<td valign="top">

LLMLingua

</td>
<td valign="top">

25%

</td>
<td valign="top">

~75%

</td>
<td valign="top">

0.882 — below acceptable

</td>
<td valign="top">

—

</td>
</tr>
<tr>
<td valign="top">

Caveman

</td>
<td valign="top">

—

</td>
<td valign="top">

~29%

</td>
<td valign="top">

1.000 (lossless)

</td>
<td valign="top">

**1.000**

</td>
</tr>
</table>

For batch release quantities, financial postings, or any prompt containing business-critical numbers: LLMLingua at 50% retention drops approximately 9% of numeric values. This loss rate is unacceptable for any use case covered by the precision guardrail stack. At 25% retention, LLMLingua quality falls to 0.882 — below any acceptable threshold for business-critical content.

**Safe alternative**: Caveman compression achieves ~29% input reduction with quality score 1.000 and lossless numeric recall. Use Caveman for any prompt that contains numerical values, quantities, prices, dates, or identifiers.

**Caching incompatibility**: input compression defeats prompt caching if applied to cached prompt segments. Only apply compression to non-cached portions of the prompt. Applying compression to a cached prefix invalidates the cache entry on every call, eliminating the latency and cost benefits of caching.


<a name="loio87a96eeabb444fd7a0fad07c8547671f__section_layer1_pre_check"/>

## Layer 1 — Pre-Check: Constraining the Agent's Input Space

The most effective guardrail prevents errors before the agent reasons. By constraining what the agent can work with — valid value ranges, approved reference lists, boundary conditions — the pre-check reduces the space of outputs the agent can produce.

### Authoritative Reference Data via sapMcpClient

Before the worker agent begins its reasoning, a `sapMcpClient` call retrieves the authoritative boundaries from the SAP system of record:

```
[sapMcpClient: get_material_unit_of_measure → returns canonical UoM for this material]
[sapMcpClient: get_approved_vendor_list → returns vendors and approved price ranges]
[sapMcpClient: get_production_order_constraints → returns min/max batch sizes]
         ↓
[sapOrchestration: worker agent, grounded with retrieved constraints]
  System prompt includes: "You must express all quantities in {canonical_uom}.
  The valid range for this batch is {min_quantity} to {max_quantity}."
```

This pattern — retrieving authoritative constraints first and including them explicitly in the agent's grounding context — is the primary mechanism for unit confusion prevention. The agent reasons within a constrained context, not in an open space where it must infer the correct unit.

### Approved Value Lists

For categorical numerical decisions (pricing tiers, discount bands, contract quantities), the pre-check retrieves the approved list from the SAP system before the agent proposes a value. The agent can only select from the approved list; it cannot propose an arbitrary value.

This is implemented as a structured output constraint in `sapOrchestration`: the output schema specifies that the recommendation must be one of the values in the approved list, not a free-form number.


<a name="loio87a96eeabb444fd7a0fad07c8547671f__section_layer2_output_validation"/>

## Layer 2 — Output Validation

Layer 2 is the multi-stage verification pattern applied specifically to numerical outputs. The deterministic check is the primary mechanism here; the LLM critic plays a supporting role.

### Deterministic Checks for Numerical Outputs

The following deterministic checks cover the most common failure modes:

**Table: Deterministic Check Types for Numerical Outputs**


<table>
<tr>
<th valign="top">

Check Type

</th>
<th valign="top">

Implementation

</th>
<th valign="top">

Failure Modes Caught

</th>
</tr>
<tr>
<td valign="top">

Range check

</td>
<td valign="top">

Code node: `value >= min && value <= max`

</td>
<td valign="top">

Out-of-range values, magnitude errors

</td>
</tr>
<tr>
<td valign="top">

Unit consistency

</td>
<td valign="top">

Code node: verify output UoM matches canonical UoM from Layer 1

</td>
<td valign="top">

Unit confusion

</td>
</tr>
<tr>
<td valign="top">

Sign check

</td>
<td valign="top">

Code node: verify sign matches expected direction

</td>
<td valign="top">

Sign errors (long/short, positive/negative)

</td>
</tr>
<tr>
<td valign="top">

Precision check

</td>
<td valign="top">

Code node: verify decimal places match system of record expectations

</td>
<td valign="top">

Rounding errors

</td>
</tr>
<tr>
<td valign="top">

Cross-field consistency

</td>
<td valign="top">

Code node: verify related fields are internally consistent (e.g., total = unit_price × quantity)

</td>
<td valign="top">

Calculation errors

</td>
</tr>
<tr>
<td valign="top">

Reference value comparison

</td>
<td valign="top">

sapMcpClient: retrieve authoritative calculation, compare to agent output within tolerance band

</td>
<td valign="top">

Magnitude errors relative to ground truth

</td>
</tr>
</table>

The tolerance band for the reference value comparison is a design parameter set during FDE Discovery and owned by the business process owner. A ±5% tolerance on a purchase order quantity is a different risk decision than a ±0.1% tolerance on a hedge ratio. The tolerance values should be documented in the Draft Product Design Specs and revisited during the EAC phase based on observed agent behavior.

### Reference Calculation via sapMcpClient

For use cases where an authoritative calculation service exists in the SAP landscape (pricing engine, batch classification service, valuation model in SAP Analytics Cloud), the strongest possible deterministic check is to call that service independently and compare its result to the agent's output:

```
[sapOrchestration: agent generates pricing recommendation → output: {price: 42.50, currency: EUR}]
         ↓
[sapMcpClient: call SAP pricing engine with same input parameters → returns: {price: 42.48, currency: EUR}]
         ↓
[Code node: |agent_price - engine_price| / engine_price → 0.047% deviation → within 0.5% tolerance]
         ↓
[If: within tolerance?]
    ├── YES → proceed to Layer 3 (or write-back if below human approval threshold)
    └── NO  → escalate to sapTaskCenter with both values and deviation percentage
```

This pattern directly addresses the key limitation: the LLM critic is the right tool when no deterministic reference exists, but when one does exist, it should be used in preference to or in combination with the LLM check.

### LLM Critic for Contextual Plausibility

The LLM critic (second `sapOrchestration` call with a critic system prompt) adds a check that deterministic rules cannot perform: does the output make sense in context? A quantity within the valid range and within ±5% of the reference calculation may still be wrong if it violates a contextual constraint the rules do not capture (e.g., "this customer's contract specifies quarterly minimums that make this weekly order quantity implausible").

The LLM critic system prompt for numerical validation should be explicit:

> "You are a numerical validation agent. You have received a proposed {output_type} of {value} {unit}. The authoritative reference value is {reference_value} {unit}. Your task is to determine whether the proposed value is plausible given the following context: {context}. Output a JSON object with fields: verdict (CONFIRM or REJECT), reason (one sentence), confidence (0.0–1.0)."

The critic's structured output feeds directly into the routing logic: the downstream If node in the Joule Studio workflow branches on `verdict` — CONFIRM proceeds to Layer 3; REJECT escalates to `sapTaskCenter` with the critic's `reason` and `confidence` attached. No further parsing is needed between the critic call and the routing decision, which is why the output schema is fixed rather than free-form.

### When No Deterministic Check Exists — the `UNAVAILABLE` Verdict

Not every numerical output has an authoritative reference service or a codable deterministic rule. When the Layer 2 deterministic stage cannot be evaluated — no reference calculation service exists in the customer landscape, and no range / unit / sign / precision rule applies to this output type — the stage returns **`stage1_verdict = UNAVAILABLE`** (recorded in the audit trail), distinct from CONFIRM (checked and passed) and REJECT (checked and failed).

`UNAVAILABLE` must never be treated as a pass. Routing on `UNAVAILABLE`:

-   The LLM critic (if configured) still runs and provides the only automated signal.
-   Because the strongest check is absent, the output is routed to **Layer 3 human approval regardless of absolute value or deviation** — the missing deterministic reference is itself a reason for human oversight, the same posture used for low-confidence outputs.
-   The gap is documented during Discovery so a rule-based Code node check or a reference service can be added over time, converting `UNAVAILABLE` into CONFIRM/REJECT coverage.

### Who Performs the Write — the Committer Pattern

Throughout Layers 2–4, "write-back" is never performed by the worker agent. The worker is **read/recommend-only** and holds no system-of-record write scope — the Agent Gateway denies a direct worker write at the API-call level (the `min(user, agent)` intersection). The write to the system of record is performed by a separate **supervisor/committer identity** after the output has cleared Layer 2 validation and any Layer 3 approval. So "proceed to write-back" in the routing above means "hand the approved value to the committer," not "the worker writes." This makes the no-direct-write property architecturally enforced rather than a design convention; an optional S/4HANA RAP instance-authorization backstop can additionally reject any write to a record not in an approved state. The audit trail attributes the committed write to the committer identity.


<a name="loio87a96eeabb444fd7a0fad07c8547671f__section_layer3_human_approval_thresholds"/>

## Layer 3 — Human Approval Thresholds

Not every numerical output warrants a human approval step — that would eliminate the efficiency case for agents. Layer 3 defines the thresholds above which the agent's output must receive explicit human sign-off before being committed.

### Threshold Design Principles

Thresholds operate on two dimensions simultaneously:

1.  **Absolute value**: the transaction size in monetary or operational terms
2.  **Deviation from reference**: how far the agent's output is from the authoritative reference (even if the absolute value is modest, a 30% deviation from the reference calculation is significant)

Both dimensions can independently trigger a human approval requirement.

### Domain-Specific Threshold Examples

**Table: Domain-Specific Human Approval Thresholds (Illustrative)**


<table>
<tr>
<th valign="top">

Domain

</th>
<th valign="top">

Absolute Threshold

</th>
<th valign="top">

Deviation Threshold

</th>
</tr>
<tr>
<td valign="top">

Regulated Manufacturing — batch quantity

</td>
<td valign="top">

Any output exceeding ±10% of standard batch size

</td>
<td valign="top">

>5% deviation from batch classification service result

</td>
</tr>
<tr>
<td valign="top">

Commodity Management — hedge ratio

</td>
<td valign="top">

Any position change >€500K notional

</td>
<td valign="top">

>2% deviation from risk model output

</td>
</tr>
<tr>
<td valign="top">

Revenue Growth Management — pricing

</td>
<td valign="top">

Any recommendation >€100K contract value

</td>
<td valign="top">

>3% deviation from pricing engine

</td>
</tr>
<tr>
<td valign="top">

Asset Management — maintenance cost

</td>
<td valign="top">

Any recommendation >€50K

</td>
<td valign="top">

>15% deviation from historical cost basis

</td>
</tr>
<tr>
<td valign="top">

Adaptive Production — production order

</td>
<td valign="top">

Any order >500 units on first-time material

</td>
<td valign="top">

>10% deviation from planning system

</td>
</tr>
</table>

These thresholds are **illustrative order-of-magnitude examples only** and have not been validated against actual FDE engagement data or domain expert review. They are intended to show the structure of the decision (absolute value + deviation from reference, both independently triggering approval), not to prescribe specific values. The actual values are a business decision made during FDE Discovery in collaboration with the process owner. They should be captured in the Draft Product Design Specs and may be adjusted during EAC based on observed human override patterns.

### Implementation via sapTaskCenter

The `sapTaskCenter` node carries the full context needed for a meaningful human decision:

```
sapTaskCenter parameters for Layer 3 approval:
  subject: "Pricing approval required: {agent_output} {unit} for {context}"
  priority: HIGH  (or VERY_HIGH for outputs above a secondary threshold)
  recipients: [process_owner_userId]
  customAttributes:
    - name: "Proposed value",    value: "{agent_output} {unit}"
    - name: "Reference value",   value: "{reference_value} {unit}"
    - name: "Deviation",         value: "{deviation_pct}%"
    - name: "Agent confidence",  value: "{confidence_score}"
    - name: "Validation result", value: "{stage1_verdict} / {stage2_verdict}"
    - name: "Context summary",   value: "{brief_context}"
    - name: "Transaction ID",    value: "{correlation_id}"
```

The `customAttributes` are the key: a human approver who receives only "Agent recommends 42.50 EUR" cannot make a good decision. A human approver who sees the proposed value, the reference value, the deviation, the agent's confidence, and the validation result has everything needed to approve or reject in seconds.

**Task Center is the single mandatory HITL channel.** Per SAP's AI Golden Path (April 2026 architectural decision), all human-in-the-loop requests from agents must route through Task Center via Agent Gateway. Custom approval flows, email-based approvals, and ad-hoc webhook implementations are not compliant and must not be used in production agents. The `sapTaskCenter` node in Joule Studio implements this routing; no additional integration is needed.

**Current HITL capability state (as of mid-2026):** Only **Permission Elevation (PE)** is currently available — the agent provides the responsible user's global user ID, and Task Center creates an approval task on their behalf. Two additional HITL modes — Mandatory Approval and Request for Further Input — are on the roadmap but not yet released. Design all precision guardrail escalation flows within the PE constraint; do not plan approval patterns that depend on capability that is not yet in production.

**Reconciling the Layer 3 design with PE-only availability.** The Layer 3 gate in this topic is a *blocking* approval that can also capture a human-*modified* value (see the `human_decision = APPROVED / REJECTED / MODIFIED` and `human_modified_value` audit fields below). Those semantics resemble the roadmap-only Mandatory Approval / Request-for-Further-Input modes, but they do **not** depend on them — they are delivered at the workflow / committer level under PE today:

-   **Blocking** is a property of the workflow, not of a platform HITL mode. The worker is read/recommend-only and never writes; the write is performed by the supervisor/committer step, which is sequenced *after* the `sapTaskCenter` task resolves. The workflow does not advance to the committer until the task is completed, so the system-of-record write is blocked until a human acts. This holds under PE.
-   **MODIFIED value** is captured from the Task Center task-completion payload: the approver can return an edited value in the task's output fields, which the workflow reads back and the committer writes as `committed_value` (recording `human_decision = MODIFIED` and `human_modified_value`). This uses the standard Task Center completion response, not the unreleased Mandatory-Approval mode.

If a given customer's Task Center configuration cannot return a modified value in the task response (approve/reject only), treat the MODIFIED path as **roadmap-dependent** and design the flow as approve/reject-only until Mandatory Approval / Request-for-Further-Input ship.

### Threshold Governance

Thresholds are configuration, not code — but there is no BAIP platform-level configuration store that injects them into a running agent. In the n8n-based workflow, thresholds live in one of two places: **workflow-level `$vars`** (resolved at execution, e.g. `{{$vars.approval_threshold_eur}}`) for values that are stable across invocations, or the **trigger payload** (`{{$json.config.approval_threshold_eur}}`) for values supplied per request by the calling context. They are never hardcoded in node parameters. Changing a `$vars` value does not require re-authoring the workflow logic, but it is a workflow-configuration change under version control — not a silent platform-config edit, and the Agent Hub does not expose a managed threshold parameter set.

Changes to thresholds should require approval from the process owner and be logged in the audit trail. A threshold that was silently lowered from €500K to €5M is a governance failure.

### Outcome-Backed Escalation

Static guardrail thresholds should be supplemented by outcome-backed escalation. The mechanism:

> When `outcome_confirmed_bad > 0` for a given guardrail check — i.e., a previously approved action that passed that check subsequently produced a confirmed bad outcome — the severity of that check escalates automatically.

Escalation paths:

-   WARN → FAIL: a check that was advisory becomes blocking
-   Conditional → Blocking: a check that fired only above a threshold fires for all subsequent outputs of the same type until the root cause is resolved

This ensures that repeated failure patterns are caught more aggressively over time, without requiring manual threshold updates after each incident. The escalation is driven by outcome data from the audit trail: the `human_decision = REJECTED` and `committed_value ≠ agent_output` fields, combined with downstream outcome confirmation, are the inputs to the escalation logic.

**Design note**: outcome confirmation is not instantaneous. For many Industry AI use cases, the outcome of a committed action is only visible days or weeks later (batch quality results, financial close, production completion). The escalation mechanism must support configurable confirmation horizons — the time window after commitment within which an outcome signal is expected. If no outcome signal arrives within the horizon, the record is treated as inconclusive (neither confirming nor escalating).

The confirmation horizon is a domain-specific parameter set during FDE Discovery. For regulated manufacturing batch releases, a horizon of 1–5 working days is typical (time from release to in-process quality check result). For financial postings, the horizon aligns with the period-end close cycle.


<a name="loio87a96eeabb444fd7a0fad07c8547671f__section_layer4_audit_trail"/>

## Layer 4 — Audit Trail

Every numerical decision that an agent commits to the system of record must have an immutable audit trail. This is both a governance requirement and a feedback mechanism that generates the ground-truth data needed to improve the agent over time.

### What the Audit Trail Must Capture

For each committed numerical output:

**Table: Audit Trail Required Fields**


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

`timestamp`

</td>
<td valign="top">

ISO-8601 timestamp of commitment

</td>
</tr>
<tr>
<td valign="top">

`agent_id`

</td>
<td valign="top">

Identity of the recommending worker agent (BAIP application registration ID); read/recommend-only, holds no write scope

</td>
</tr>
<tr>
<td valign="top">

`committer_id`

</td>
<td valign="top">

Identity that performed the write to the system of record — the supervisor/committer; all committed writes are attributed here, not to the worker

</td>
</tr>
<tr>
<td valign="top">

`agent_version`

</td>
<td valign="top">

Version of the agent workflow at time of execution

</td>
</tr>
<tr>
<td valign="top">

`correlation_id`

</td>
<td valign="top">

Links to BAIP harness trace for the full execution

</td>
</tr>
<tr>
<td valign="top">

`input_state`

</td>
<td valign="top">

The data the agent had when it produced the output (snapshot)

</td>
</tr>
<tr>
<td valign="top">

`agent_output`

</td>
<td valign="top">

The value the agent proposed, with unit

</td>
</tr>
<tr>
<td valign="top">

`stage1_verdict`

</td>
<td valign="top">

Deterministic check result (CONFIRM/REJECT/UNAVAILABLE)

</td>
</tr>
<tr>
<td valign="top">

`stage1_reference`

</td>
<td valign="top">

The reference calculation value used for comparison

</td>
</tr>
<tr>
<td valign="top">

`stage1_deviation`

</td>
<td valign="top">

Deviation percentage

</td>
</tr>
<tr>
<td valign="top">

`stage2_verdict`

</td>
<td valign="top">

LLM critic verdict (CONFIRM/REJECT) and confidence

</td>
</tr>
<tr>
<td valign="top">

`stage2_reason`

</td>
<td valign="top">

LLM critic's stated reason

</td>
</tr>
<tr>
<td valign="top">

`human_decision`

</td>
<td valign="top">

APPROVED / REJECTED / MODIFIED (if Layer 3 was triggered)

</td>
</tr>
<tr>
<td valign="top">

`human_modified_value`

</td>
<td valign="top">

If the human modified the value, the modified value

</td>
</tr>
<tr>
<td valign="top">

`committed_value`

</td>
<td valign="top">

The final value written to the system of record

</td>
</tr>
<tr>
<td valign="top">

`delta`

</td>
<td valign="top">

Difference between agent output and committed value (measures human override)

</td>
</tr>
</table>

### Storage and Immutability

The audit trail should be stored in a BAIP-managed data store (HANA Cloud is the natural choice for Industry AI engagements) with the following properties:

-   Append-only: records cannot be modified after creation
-   Only the committer identity writes the committed record; the worker holds no write scope; no agent can delete
-   Customer IT admin and process owner have read access
-   Retention period aligned with the regulatory requirements of the industry (e.g., GxP (Good Practice — pharmaceutical regulatory) batch records in Regulated Manufacturing require 10+ year retention under EU GMP Annex 11 and FDA 21 CFR Part 11 as a planning assumption — the customer's quality/compliance team must confirm the exact period for their specific regulatory framework, jurisdiction, and record type)

### Using the Audit Trail for Continuous Improvement

The audit trail is not just a compliance artifact. Each record is a labeled data point: `(input_state, agent_output, reference_value, human_decision)`. Over time this dataset enables:

-   **Calibration of tolerance bands**: if the deviation distribution shows agents consistently within ±1% of reference, the tolerance band can be tightened. If outliers cluster around a specific input pattern, targeted improvement is possible.
-   **Human override rate tracking**: the `delta` field aggregated over time is the direct measure of how often humans correct the agent — the most honest quality signal available.
-   **Model drift detection**: if the agent's accuracy degrades after a model version update, the audit trail captures this before it becomes a pattern in process KPIs.


<a name="loio87a96eeabb444fd7a0fad07c8547671f__section_confidence_uncertainty_quantification"/>

## Confidence and Uncertainty Quantification

Numerical outputs should carry an explicit confidence signal, not just a value. A confident correct answer and a low-confidence correct answer have different downstream implications: the former can be routed to direct write-back (below the human approval threshold); the latter may warrant human review regardless of whether the absolute value is within tolerance.

### Structured Output with Confidence Field

The `sapOrchestration` node supports structured output schemas. A precision-sensitive agent's output schema should include a confidence field:

```json
{
  "type": "object",
  "properties": {
    "value": { "type": "number" },
    "unit": { "type": "string" },
    "confidence": { "type": "number", "minimum": 0, "maximum": 1 },
    "confidence_basis": { "type": "string" }
  },
  "required": ["value", "unit", "confidence"]
}
```

The `confidence_basis` field asks the agent to state what it is confident or uncertain about: "Confidence reduced because input data contained two conflicting quantity fields" is actionable; a raw confidence score of 0.6 is not.

### Confidence-Based Routing

The confidence score integrates into the guardrail stack as a routing signal:

```
[If: confidence < 0.7]
    └── Route to Layer 3 (human approval) regardless of absolute value or deviation
        Reason: agent uncertainty is itself a reason for human oversight

[If: confidence >= 0.7 AND within tolerance AND below absolute threshold]
    └── Proceed to write-back + audit trail
```

Confidence thresholds, like value thresholds, are configuration parameters owned by the process owner and are held in n8n `$vars` or the trigger payload (`{{$json.config.x}}`) under version control — not in a BAIP platform config store.


<a name="loio87a96eeabb444fd7a0fad07c8547671f__section_precision_guardrails_fde_delivery"/>

## Precision Guardrails in FDE Delivery

### During Discovery, Sub-Phases 3–4 (Define–Prioritize)

-   For each numerical output in the TO-BE process flow, define the four-layer stack requirement: which layers apply, what the thresholds are, who owns human approval.
-   Identify which SAP calculation services exist in the customer's landscape and can serve as the Layer 2 reference (pricing engine, planning system, batch classifier). If no reference service exists for a given output, document the gap and plan for a Code node rule-based check instead.
-   The human approval threshold values are a business decision, not a technical one. They should be agreed with the process owner during Discovery and included in the Draft Product Design Specs.
-   The audit trail retention requirement is a regulatory decision. Confirm with the customer's compliance team, especially for Regulated Manufacturing (GxP, EU GMP).

### During Develop Phase (MVP Build)

-   Build the guardrail stack concurrently with the worker agent. The Layer 2 validation and Layer 3 Task Center integration are not post-MVP additions; they define what a production-ready MVP looks like for precision-sensitive use cases.
-   The audit trail table should be provisioned and tested in the MVP environment. Testing should verify: records are created for every committed output; records include all required fields; records cannot be modified after creation.
-   The guardrail test suite should include explicit boundary condition tests: outputs at exactly the threshold, outputs 1 unit above and below, unit-confused inputs, sign-inverted inputs, magnitude-error inputs.

### During Productization

-   Threshold configuration parameters must be in the production handover package with clear ownership: who can change them, via what mechanism, with what approval.
-   The Sirius QPA test strategy must include precision guardrail boundary tests. A test that only verifies "agent produces a number" is insufficient; the test must verify "agent produces a number that passes the full guardrail stack."
-   The audit trail table structure and retention policy must be included in the DPCE data protection review.


<a name="loio87a96eeabb444fd7a0fad07c8547671f__section_summary"/>

## Summary

**Table: Precision Guardrail Stack Summary**


<table>
<tr>
<th valign="top">

Layer

</th>
<th valign="top">

What It Does

</th>
<th valign="top">

Primary Implementation

</th>
<th valign="top">

When It Is Required

</th>
</tr>
<tr>
<td valign="top">

Pre-check (1)

</td>
<td valign="top">

Constrain agent's input space to valid ranges and approved values

</td>
<td valign="top">

sapMcpClient → system of record; structured output schema

</td>
<td valign="top">

Always for write-back agents operating on numerical outputs

</td>
</tr>
<tr>
<td valign="top">

Output validation (2)

</td>
<td valign="top">

Verify numerical output against deterministic reference and/or LLM critic

</td>
<td valign="top">

Code node range check + sapMcpClient reference call + sapOrchestration critic

</td>
<td valign="top">

Always for write-back agents; critic optional if deterministic check available

</td>
</tr>
<tr>
<td valign="top">

Human approval (3)

</td>
<td valign="top">

Gate outputs above value/deviation thresholds on explicit human sign-off

</td>
<td valign="top">

sapTaskCenter with full context custom attributes

</td>
<td valign="top">

For high-value or high-deviation outputs; thresholds set by process owner

</td>
</tr>
<tr>
<td valign="top">

Audit trail (4)

</td>
<td valign="top">

Immutable record of every committed numerical decision

</td>
<td valign="top">

Append-only HANA Cloud table; includes input, output, verdicts, human decision

</td>
<td valign="top">

Always for committed outputs; retention aligned to regulatory requirements

</td>
</tr>
<tr>
<td valign="top">

Confidence routing

</td>
<td valign="top">

Route low-confidence outputs to human regardless of absolute value

</td>
<td valign="top">

sapOrchestration structured output with confidence field; If node routing

</td>
<td valign="top">

For any output where agent uncertainty is operationally significant

</td>
</tr>
</table>

Key rule: **precision guardrails are not optional for write-back agents in regulated or high-value Industry AI use cases**. The choice is not whether to have them; the choice is how to configure them. The configuration (tolerances, thresholds, reference services, audit retention) is a business and regulatory decision made during Discovery.

**Table: Additional Precision Guardrail Principles**


<table>
<tr>
<th valign="top">

Addition

</th>
<th valign="top">

Key Principle

</th>
</tr>
<tr>
<td valign="top">

FAIL / WARN classification

</td>
<td valign="top">

FAIL = blocking (deploy/operation aborted); WARN = advisory (proceed with logged finding); classification is a design decision per check, made in Discovery

</td>
</tr>
<tr>
<td valign="top">

MUST rules

</td>
<td valign="top">

Business invariants encoded in `golden_scenarios.yaml`; LLM-graded in eval suite; FAIL-severity MUST rule blocks deployment; authored jointly by Domain PM (content) and Dev Architect (severity + test case)

</td>
</tr>
<tr>
<td valign="top">

Outcome-backed escalation

</td>
<td valign="top">

`outcome_confirmed_bad > 0` auto-escalates severity; configurable confirmation horizons per domain; prevents silent accumulation of repeated failure patterns

</td>
</tr>
<tr>
<td valign="top">

Input compression

</td>
<td valign="top">

LLMLingua unsafe for numerical/financial prompts at any retention level (9% numeric drop at 50% retention); use Caveman (lossless, ~29% reduction) for all prompts containing business-critical numbers; compression defeats prompt caching on cached segments

</td>
</tr>
</table>

**Related Information**  




[Secure and Govern — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/3ea62b45d7f64a8aab15321424ffb37c.html "Verify AI assets against governance criteria and build a verified inventory.") :arrow_upper_right:
[Security and Data Protection for AI Agents](../Security-and-Data-Protection/security-and-data-protection-90b7501.md)

[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html)

[SAP AI Agent Hub — Plan and Build](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/e2a29b0c422e4aceadee658f4bbab71b.html)
