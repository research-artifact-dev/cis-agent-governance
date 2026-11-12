<!-- loioa12bf85b576b4a78a846573ed18cd202 -->

# Open Questions

Questions that arose during concept development where the answer is not yet known, requires external validation, or depends on legal/product/domain expertise outside the scope of this document. Each question includes context and references so it can be picked up and answered independently.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q1"/>

## Q1 — Does Workflow C (Support Relay) Require a CA Even for Sanitized Exports?

**Status:** Open — needs input from SAP legal / FDE legal team

**Context:**

[Security and Data Protection](../Security-and-Data-Protection/security-and-data-protection-90b7501.md) (Security & Data Protection) defines a three-workflow supervisor pattern. Workflow C ("Support Relay") is triggered when a confirmed anomaly requires SAP support engagement. Its purpose is to export a minimal, sanitized trace extract from the customer's BAIP (Business AI Platform) tenant to SAP's support systems, and create a support ticket automatically.

The technical design minimizes what leaves the tenant:

-   Business data fields are stripped in a Code node
-   SAP DPI entity masking is applied via `sapOrchestration (enableMasking=true)`
-   Only structural anomaly evidence is retained: error codes, tool call names (not parameters), timing/latency, agent identity, agent version, correlation ID, tenant ID

The legal question is whether this sanitized export still triggers a Collaboration Agreement (CA) requirement. The chapter's legal alignment table contains:

> *"Support relay (Workflow C) exporting sanitized trace to SAP support → Assess with legal: CA may be required even for sanitized extracts"*

**The specific uncertainty:**

SAP's legal framework for FDE engagements distinguishes:

-   (a) Customer business data → clearly triggers CA
-   (b) Personal data → triggers CA (and DPI masking handles this technically)
-   (c) Operational/telemetry metadata from customer's environment → unclear

If (c) falls below the CA threshold because it contains no personal or business data, Workflow C can be enabled in FDE engagements without the long-lead CA approval process. If any export from a customer's BAIP environment triggers CA regardless of content, Workflow C becomes a CA-gated feature that requires planning from week 3 of Execute.

**Practical implication:**

The CA approval gate requires Shardul Vikram's sign-off and is not a routine approval. If Workflow C always requires a CA, every FDE engagement that includes automated SAP support ticket creation needs this planned early — it cannot be added as an afterthought.

**Who can answer this:** SAP legal / FDE legal team; alternatively, check whether the FDE Operating Model or the book-of-fde legal-agreements documentation addresses telemetry/metadata exports specifically.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q2"/>

## Q2 — Are the Human Approval Threshold Examples in [Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md) Section 4.6 Valid for Their Domains?

**Status:** Open — needs review by domain experts (Regulated Manufacturing, Commodity Management, Revenue Growth Management)

**Context:**

[Precision Guardrails](../Precision-Guardrails/precision-guardrails-87a96ee.md) (Precision Guardrails), includes a table of illustrative thresholds for the Layer 3 human approval gate — the point above which an agent's numerical output requires explicit human sign-off before write-back. The table covers five Industry AI domains:

**Table: Illustrative Layer 3 Human Approval Thresholds**


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

±10% of standard batch size

</td>
<td valign="top">

>5% from batch classification service

</td>
</tr>
<tr>
<td valign="top">

Commodity Management — hedge ratio

</td>
<td valign="top">

>€500K notional position change

</td>
<td valign="top">

>2% from risk model

</td>
</tr>
<tr>
<td valign="top">

Revenue Growth Management — pricing

</td>
<td valign="top">

>€100K contract value

</td>
<td valign="top">

>3% from pricing engine

</td>
</tr>
<tr>
<td valign="top">

Asset Management — maintenance cost

</td>
<td valign="top">

>€50K

</td>
<td valign="top">

>15% from historical cost basis

</td>
</tr>
<tr>
<td valign="top">

Adaptive Production — production order

</td>
<td valign="top">

>500 units on first-time material

</td>
<td valign="top">

>10% from planning system

</td>
</tr>
</table>

These values were constructed from first principles and general domain knowledge. The chapter already includes a disclaimer that they are illustrative order-of-magnitude examples only. However, before this concept is used in a customer conversation or as an FDE planning baseline, the values should be reviewed by someone with hands-on experience in each domain.

**Specific uncertainties:**

-   **Regulated Manufacturing**: GxP batch release environments may have much tighter tolerances than ±10%/5% — regulatory frameworks (EU GMP, FDA 21 CFR) may impose specific deviation limits that override business-defined thresholds
-   **Commodity Management**: the €500K notional threshold may be orders of magnitude off depending on the customer's trading volume and risk appetite
-   **Revenue Growth Management**: whether €100K is a meaningful threshold depends entirely on the customer's deal sizes — it could be too low or too high

**What a good answer looks like:** Either (a) specific validated values per domain from engagement experience, or (b) a confirmed statement that the structure (absolute + deviation, both independently triggering) is correct and only the specific numbers need customer-specific calibration during Discovery.

**Who can answer this:** CIS/FDE colleagues with Regulated Manufacturing, Commodity Management, or RGM engagement experience; alternatively, Industry AI domain PMs.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q3"/>

## Q3 — Does sapMcpClient List Available Tools Dynamically at Runtime, or Does It Require Pre-Configuration?

**Status:** RESOLVED — verified against live n8n instance (`CUSTOM.sapMcpClient` v1, 2026-10-22)

**Verification source:** Joule Studio IBD MCP, `search-nodes-catalog` query against live n8n instance.

**Finding:** Option (2) is confirmed — **build-time tool discovery, not runtime**. The `loadOptionsMethod: "getTools"` fetches available tools from the selected MCP server at **design time** in the Joule Studio n8n canvas, populating a dropdown. At runtime, the tool is fixed — either as a pre-selected list item or as a manually entered tool ID.

The `mcpServers` hidden parameter description confirms this: *"MCP servers injected externally at design time (e.g. by Joule Studio). Set programmatically via workflow JSON — not visible in UI."*

**Confirmed parameters:**

**Table: sapMcpClient Confirmed Parameters**


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

Notes

</th>
</tr>
<tr>
<td valign="top">

`mcpServerName`

</td>
<td valign="top">

option

</td>
<td valign="top">

—

</td>
<td valign="top">

MCP server; list populated at design time from live instance

</td>
</tr>
<tr>
<td valign="top">

`toolMode`

</td>
<td valign="top">

option

</td>
<td valign="top">

`list`

</td>
<td valign="top">

`list` (design-time dropdown) or `id` (manual tool ID entry)

</td>
</tr>
<tr>
<td valign="top">

`toolFromList` / `toolById`

</td>
<td valign="top">

string

</td>
<td valign="top">

—

</td>
<td valign="top">

Tool to invoke; fixed at design time

</td>
</tr>
<tr>
<td valign="top">

`toolArguments`

</td>
<td valign="top">

JSON

</td>
<td valign="top">

`{}`

</td>
<td valign="top">

Arguments passed to the tool at runtime

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

30,000ms

</td>
<td valign="top">

Max 300,000ms — **note: default is 30s, not 180s** like `sapAgent`

</td>
</tr>
<tr>
<td valign="top">

`includeMetadata`

</td>
<td valign="top">

boolean

</td>
<td valign="top">

false

</td>
<td valign="top">

Include server info and tool details in response

</td>
</tr>
</table>

**Impact on the concept:** [Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md) description has been corrected. The original claim *"the node loads available tools from the connected SAP system at runtime"* was inaccurate. Tool discovery is a design-time activity in Joule Studio. At runtime, every tool call is deterministic. This aligns with the design rule already in the chapter (hardcode tool calls for deterministic validation flows) — that rule is now the universal rule, not a special case.

**Timeout note:** The `sapMcpClient` timeout default (30s) is significantly shorter than `sapAgent` (180s). For MCP tool calls that hit SAP backend systems with non-trivial processing time, the timeout must be explicitly set — otherwise the workflow will fail silently on calls that would succeed with more time.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q4"/>

## Q4 — Concept Gaps Identified During Review: Four Topics to Add

**Status:** Addressed — [Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md) (Multi-Agent Orchestration) and [Production Operations](../Production-Operations/production-operations-edd9e33.md) (Production Operations)

The following topics were identified during an earlier concept review as gaps in the then five-chapter concept, ranked by importance:

1.  **Multi-agent orchestration** *(most important)*: addressed in [Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md). Covers three orchestration topologies (Flat, Per-Worker Supervised, Shared Supervisor), cross-worker consistency verification, authorization scope isolation, and FDE delivery implications.

2.  **Agent versioning and rollback**: addressed in [Production Operations](../Production-Operations/production-operations-edd9e33.md), sections 8.2–8.3. Covers the five-component version stack, rollback trigger criteria, rollback mechanics per component, state continuity (audit trail, shared state store, in-flight sessions), and rollback authorization.

3.  **Cost and token monitoring**: addressed in [Production Operations](../Production-Operations/production-operations-edd9e33.md), sections 8.4–8.5. Covers token consumption as operational signal, what to monitor (per-invocation and trend metrics), runaway loop detection, max_tokens guardrail calibration, and the Workflow B extension.

4.  **Agent migration between environments** (dev → test → production): addressed in [Production Operations](../Production-Operations/production-operations-edd9e33.md), sections 8.6–8.8. Covers the four-environment sequence, the artifact migration table (what moves, what must be re-provisioned), three things never to copy between environments, and the environment-agnostic configuration pattern.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q5"/>

## Q5 — Does SAP Build Code Include Built-In Dependency/Security Scanning for Pro-Code Agents?

**Status:** ANSWERED — via Joule (cross-validated, 2026-10-22). Short answer: No built-in SAST or SCA.

**Finding:**

SAP Build Code does **not** include built-in SAST or SCA. The only native security scan is **malware scanning**, which is off by default (`malwareScan.scan: true` in `config.yaml` to enable).

**Table: SAP Build Code Security Scanning Capabilities**


<table>
<tr>
<th valign="top">

Capability

</th>
<th valign="top">

Native in Build Code?

</th>
<th valign="top">

Notes

</th>
</tr>
<tr>
<td valign="top">

Malware scanning

</td>
<td valign="top">

Off by default

</td>
<td valign="top">

`malwareScan.scan: true` in `config.yaml`

</td>
</tr>
<tr>
<td valign="top">

SAST

</td>
<td valign="top">

✗

</td>
<td valign="top">

Must be added externally

</td>
</tr>
<tr>
<td valign="top">

SCA (dependency/OSS scanning)

</td>
<td valign="top">

✗

</td>
<td valign="top">

Must be added externally

</td>
</tr>
<tr>
<td valign="top">

DAST

</td>
<td valign="top">

✗

</td>
<td valign="top">

Must be added externally

</td>
</tr>
</table>

Build Code's CI/CD provides a **Compliance stage** that accepts additional commands, credentials, and variables — this is the hook for external scanning tools, but nothing is pre-wired.

**Three integration paths:**

1.  **Checkmarx One** — SAP Community-documented; add `.cicd/cxone.sh` as Additional Command in Compliance stage with `CLIENT_ID`, `CLIENT_SECRET`, `CX_PROJECT`, `CX_TENANT`, `CX_URI` as pipeline credentials.
2.  **Onapsis Control** — SAP-endorsed, IDE-level; integrates directly into Build Code / BAS for real-time ("spell-check style") SAST scanning. Covers SAP-specific patterns (ABAP, CAP, BTP) that generic tools miss.
3.  **Generic tools via Additional Commands** — Snyk, Semgrep, OWASP Dependency-Check, etc. can be bolted on via the Compliance stage mechanism.

**Agent-specific gap:**

-   **SCA is the bigger risk** for agents: npm/pip transitive dependency chains are large, no built-in CVE or license scan.
-   **SAST for TypeScript/Python**: CodeQL (GitHub Actions) works well; SAP's own architecture-center repos use it.
-   **No "agent-aware" security content** in any tool yet — prompt injection patterns and AI-specific OWASP Top 10 items remain manual review territory.

**Pragmatic CIS recommendation (no Onapsis/Checkmarx license required):**

-   CodeQL via GitHub Actions for SAST
-   `npm audit` / `pip-audit` in the CI pipeline for basic SCA

**Impact on the concept:**

[Security and Data Protection](../Security-and-Data-Protection/security-and-data-protection-90b7501.md) should be updated: the "SAP Build Code may include built-in scanning" option is incorrect — there is no built-in SCA/SAST. The section should note that external tools are required, with the Compliance stage integration hook as the recommended path.

> ### Note:  
> Q6–Q9 were consolidated into other questions during earlier reviews and are intentionally not reused; the numbering jumps from Q5 to Q10.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q10"/>

## Q10 — What Is CIS's Operational Mandate for SAP-Managed Tenant (Topology B) Deployments Post-Productization?

**Status:** Open — needs clarification from CIS leadership / engagement model owners

**Context:**

[CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md) (CIS CoE Operations Model), describes two deployment topologies. Topology B is the SAP-managed tenant model: the agent runs in a SAP-managed BAIP environment and SAP/CIS is the operator. The concept notes that CIS operational responsibility is higher in this topology — because SAP operates the BAIP environment, CIS carries responsibility for both the platform and the agents running on it.

However, the concept does not specify:

-   Which team within CIS holds this operational responsibility (T&O, the FDE team, CIS regional engineering pods, or a dedicated managed service function)
-   What the mandate scope is (platform monitoring only, agent configuration, incident response, proactive improvement, or all of the above)
-   Whether a formal SLA structure exists for SAP-managed tenant deployments, and if so, what it covers and who signs it
-   How this differs from the Topology A (customer tenant) operational model in practice, beyond the default access difference

**The specific uncertainty:**

The T&O team within CIS has clear platform and tooling responsibilities. Whether post-productization operational responsibility for SAP-managed tenant deployments falls to T&O specifically — versus remaining with the FDE delivery team during EAC and then transitioning, or being a separate managed service arrangement — is not defined in this concept and may not yet be defined organizationally.

**Practical implication:**

Before this concept is used in a customer engagement involving a SAP-managed tenant (Topology B), the operational mandate must be defined. A customer signing a SAP-managed Industry AI subscription will ask who is responsible for the agents, what the SLA is, and what happens when something goes wrong. Without a clear answer, committing to Topology B creates undefined post-delivery obligations.

**Who can answer this:** CIS leadership; FDE operating model owners; whoever is defining the commercial and operational model for SAP-managed Industry AI tenant deployments.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q11"/>

## Q11 — Is the Two-Phase EAC Support Model (P&E → CS&D) Still Valid After the July 2026 Reorg?

**Status:** Open — needs confirmation with the FDE/Industry AI operating model owners

**Context:**

[CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md) (CIS CoE Operations Model), describes an EAC support model based on two phases: Phase 1 is led by the development team (P&E), Phase 2 transitions to CS&D once ticket volume and active customer count exceed a threshold.

This description is sourced from the *FDE Operating Model at Scale* document (Version 1.0, October 2025). That document predates the July 1, 2026 organizational changes that:

-   Moved Customer Innovation Services (CVG) into Industry AI and renamed it CIS (Customer Industry Solutions)
-   Restructured the Industry Cluster teams (Manufacturing, Energy & Natural Resources, Services Industries)
-   Established CIS as the central, customer-anchored Industry AI acceleration engine

**The specific uncertainty:**

The October 2025 document describes P&E as the team leading FDE execution and Phase 1 EAC support, with CS&D providing the Phase 2 support structure. After the reorg, it is unclear whether:

-   The P&E / CS&D support handover model still applies as described, or whether the CIS structure (including Regional Engineering Pods) changes who handles Phase 1 and Phase 2 support
-   The threshold that triggers the Phase 1 → Phase 2 transition has been redefined
-   Any new operating model documentation supersedes the October 2025 source

**Practical implication:**

[CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md) uses this model to define when the CIS CoE operating rhythm kicks in ("from Phase 2 onward"). If the organizational model has changed, the phase boundary may be different, which affects how FDE engagements should plan the support handover.

**Related:** [Open Questions Q10](open-questions-a12bf85.md) — operational mandate for Topology B

**Who can answer this:** FDE/Industry AI operating model owners; CIS leadership; check whether a post-reorg version of the FDE Operating Model document exists.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q12"/>

## Q12 — Does `sapAgent` Support Non-Blocking Execution with a Callback URL?

**Status:** RESOLVED — verified against live n8n instance (node `CUSTOM.sapAgent` v1.4.9, 2026-10-21)

**Verification source:** Joule Studio IBD MCP, `search-nodes-catalog` query against live n8n instance.

**Confirmed parameters:**

**Table: sapAgent Confirmed Parameters**


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

Notes

</th>
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

*"If false, you must provide a callback URL"* — confirmed

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

Shown only when `blocking=false`; *"URL to receive push notifications when the agent responds"*

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

For correlating callbacks when multiple workers are in flight

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

Additional context passed to the agent — relevant to [Multi-Agent Orchestration](../Multi-Agent-Orchestration/multi-agent-orchestration-04e5d45.md) authorization scope discussion

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

Max 300,000ms (5 min); applies in blocking mode

</td>
</tr>
</table>

**Finding:** The supervisor pattern's worker-supervisor communication model ([Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md)) is architecturally sound. The `blocking=false` + `callbackUrl` pattern is a first-class feature of the `sapAgent` node, not an assumption. The concept description in [Supervisor Agent Pattern](../Supervisor-Agent-Pattern/supervisor-agent-pattern-16a4089.md) is accurate.

**Secondary finding:** The `timeout` default (180s) and max (300s) are relevant context for the 60-second Joule UI timeout noted in the CMJ/BYOA cheat sheet. The `sapAgent` node itself supports much longer execution windows than the Joule conversational interface does.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q13"/>

## Q13 — Does the BAIP Cloud Logging API Support Intra-Tenant Query from n8n Workflows?

**Status:** RESOLVED — full implementation path confirmed (cross-validated with Joule, 2026-10-22)

**Complete architecture picture:**

*Step 1 — Telemetry ingest (confirmed):*
BAIP agent runtime instruments with OTEL and pushes to Cloud ALM inbound APIs (`/calm-traces/traces`, `/calm-metrics/metrics`). Cloud ALM is the BAIP Govern pillar's observability layer.

*Step 2 — Where it lands:*
Agent telemetry lands under `hm` (Health Monitoring) and `exm`/`im` providers via the standard OTEL path. No dedicated "BAIP" or "AI Agent" content provider exists yet.

*Step 3 — Outbound query (verified):*
Cloud ALM Pull APIs (`/api/calm-traces/v1/traces`, `/api/calm-metrics/v1/metrics`, `/api/calm-analytics/v1/analytics`) can query this data. Agent data is accessible via the `hm` metrics provider — but only after manual service registration (Step 4 below).

*Step 4 — serviceId and registration (new finding, 2026-10-22):*

**There is no pre-built BAIP serviceId.** BAIP / Joule Studio / managed n8n does NOT automatically register a service in Cloud ALM Landscape Management. Workflow B requires a **one-time manual setup**:

1.  **Register a service in Cloud ALM Landscape Management**: `Administration → Landscape Management → Services & Systems`. Cloud ALM assigns the LMS ID — this becomes the `serviceId`. One-time, per deployment.
2.  **Add to Health Monitoring scope**: Health Monitoring → Configuration → add the service and toggle data collection ON.
3.  **Get Cloud ALM API credentials**: create a service binding for the Cloud ALM API service instance in BTP to obtain OAuth2 client credentials and the API endpoint.
4.  **Instrument Workflow B with OTEL**: push metrics to `/calm-metrics/metrics`. The `serviceId` **must be passed as the interface parameter** (not in the OTel payload): `POST /calm-metrics/metrics?serviceId=<your-lms-id>`. Note: `service.name` in the OTel resource attributes is ignored by Cloud ALM — only the `serviceId` query parameter counts.
5.  **At query time**: Workflow B uses the same `serviceId` to query metrics back via the `hm` provider.

**Key deployment constraint:**
The `serviceId` is **tenant-specific and opaque** — it cannot be hardcoded in the workflow. It must be injected at deployment time via `{{$env.CALM_SERVICE_ID}}` (environment variable on Kyma/self-hosted) or via trigger-payload config (`{{$json.config.calmServiceId}}`). This aligns with the environment-agnostic pattern.

**Conclusion:**
Workflow B's approach is architecturally valid and implementable — but requires a one-time Cloud ALM LMS registration step as part of the deployment runbook. The concept's HTTP Request node can call Cloud ALM's `hm` metrics provider. The concept should document this registration prerequisite in the production handover checklist ([Production Operations](../Production-Operations/production-operations-edd9e33.md)).


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q14"/>

## Q14 — Does the SAP Prompt Registry Exist as a Named, Versioned Store Referenceable from `sapOrchestration`?

**Status:** CONFIRMED GAP — verified against live n8n instance (`CUSTOM.sapOrchestration` v1.5.0, 2026-09-08)

**Verification source:** Joule Studio IBD MCP, `search-nodes-catalog` query against live n8n instance.

**Finding:**

The `sapOrchestration` node has a `promptSource` field (type: `hidden`, default: `inline`) with **only one option: `inline`** — "Define the prompt directly in this node." There is no `promptRegistry` parameter. Prompts must currently be defined inline in the workflow node.

A `promptTemplate` source value is referenced in the node's `displayOptions` conditions (several parameters show only when `promptSource` is `inline` OR `promptTemplate`), indicating this capability is **planned but not yet shipped** in v1.5.0.

**Impact on the concept:**

[Production Operations](../Production-Operations/production-operations-edd9e33.md)'s prompt versioning model — store prompts in the Registry, reference by name, update without redeployment — is not currently achievable. The concept's "Versioning in Joule Studio" section and environment-specific configuration pattern both depend on this capability.

**Current fallback approach** (documented in [Production Operations](../Production-Operations/production-operations-edd9e33.md)):

-   Store prompt text as a workflow-level string parameter, configurable per environment via BAIP configuration rather than hardcoded in the node
-   Maintain prompt text in the engagement Git repository alongside the workflow definition
-   Tag workflow versions with the prompt version in the version stack documentation
-   Prompt changes require a workflow redeployment (same as any other node parameter change)

When `promptTemplate` source becomes available, migrating to it will restore the no-redeployment prompt update capability without requiring architectural changes.

**Who can answer this:** Joule Studio product team; SAP Prompt Management documentation (if it exists); generative AI hub developer documentation.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q15"/>

## Q15 — Does BAIP Provide a Runtime Governance Parameter Store Accessible from n8n Workflow Expressions?

**Status:** CONFIRMED GAP — `{{config.x}}` is not a valid n8n expression; no BAIP platform-level config injection exists (cross-validated with Joule, 2026-10-22)

**Finding:**

`{{config.x}}` is **not a valid n8n expression** — it resolves to `undefined` at runtime. There is no `config` root object in n8n's expression scope, neither in vanilla n8n nor in SAP's managed n8n inside Joule Studio.

Valid root-level objects in n8n expressions:

**Table: Valid n8n Expression Root Objects**


<table>
<tr>
<th valign="top">

Expression

</th>
<th valign="top">

What It Gives You

</th>
</tr>
<tr>
<td valign="top">

`{{$json.x}}`

</td>
<td valign="top">

Data from the current/previous node output

</td>
</tr>
<tr>
<td valign="top">

`{{$vars.x}}`

</td>
<td valign="top">

Instance-level custom variables (string-only, set in UI)

</td>
</tr>
<tr>
<td valign="top">

`{{$env.X}}`

</td>
<td valign="top">

Host environment variables (set at deployment)

</td>
</tr>
<tr>
<td valign="top">

`{{$node["Name"].json.x}}`

</td>
<td valign="top">

Data from a specific named node

</td>
</tr>
<tr>
<td valign="top">

`{{$workflow.id}}`, `{{$execution.id}}`

</td>
<td valign="top">

Workflow/execution metadata

</td>
</tr>
</table>

**What BAIP/Joule Studio actually provides at configuration time:**

-   System prompt and LLM configuration (model, temperature) — set when creating the workflow in Joule Studio
-   BTP Destination properties for auth/transport config — infrastructure-level, not workflow-expression-level
-   Managed runtime handling scaling, secrets, and model access — opaque to workflow expressions

None of these create an injectable `config` object in the expression engine.

**Three viable alternatives:**

**Table: Viable Configuration Alternatives**


<table>
<tr>
<th valign="top">

Option

</th>
<th valign="top">

Mechanism

</th>
<th valign="top">

Works in Managed n8n?

</th>
<th valign="top">

Per-Invocation Flexibility?

</th>
</tr>
<tr>
<td valign="top">

**Option 1 — Trigger payload**

</td>
<td valign="top">

Pass config in webhook/trigger JSON body; reference as `{{$json.config.x}}`

</td>
<td valign="top">

Yes — fully supported

</td>
<td valign="top">

Yes — caller sets values each invocation

</td>
</tr>
<tr>
<td valign="top">

**Option 2 — n8n `$vars`**

</td>
<td valign="top">

Set static values in n8n Variables UI; reference as `{{$vars.myParam}}`

</td>
<td valign="top">

Possibly restricted to what SAP exposes in UI

</td>
<td valign="top">

No — static per instance

</td>
</tr>
<tr>
<td valign="top">

**Option 3 — Env vars (`$env`)**

</td>
<td valign="top">

Inject env vars into n8n container at deployment; reference as `{{$env.MY_PARAM}}`

</td>
<td valign="top">

Only on self-hosted n8n (Kyma path)

</td>
<td valign="top">

No — static per deployment

</td>
</tr>
</table>

**Recommended approach:** Option 1 — pass config in the trigger/webhook payload as `{{$json.config.x}}`. This works in both managed and self-hosted n8n and gives per-invocation flexibility. BAIP doesn't provide a platform-level equivalent.

**Impact on the concept:**

Chapters 4 and 8 reference `{{config.approval_threshold}}` as a "BAIP governance parameter store" enabling IT admins to adjust thresholds without redeployment. This pattern **does not exist** as described. The chapters need revision:

-   Replace `{{config.x}}` with `{{$json.config.x}}` (trigger-payload pattern) or `{{$vars.x}}` ($vars pattern)
-   Acknowledge that true "change without redeployment" requires either the trigger-payload pattern (caller supplies config per invocation) or `$vars` (static, requires UI edit but no redeployment)


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q16"/>

## Q16 — Can Hyperspace LLM Proxy Costs Be Tracked Per FDE Engagement or Project?

**Status:** Partially answered — see [Token Efficiency in Industry AI Delivery](../Token-Efficiency/token-efficiency-47721ff.md), for full analysis and recommended approach. Structural solution (Option B) requires HAI proxy roadmap input; near-term workaround (Option C sidecar) is feasible without proxy changes.

**Context:**

[Token Efficiency in Industry AI Delivery](../Token-Efficiency/token-efficiency-47721ff.md) (Token Efficiency), proposes build-time token efficiency measurement as an engagement-level practice. SAP's Token Consumption Governance Framework (live August 17, 2026) provides individual developer consumption tracking via the HAI proxy (Hyperspace LLM Proxy). The proxy has no project grouping today.

**Analysis summary (based on HAI proxy documentation review):**

The proxy backend already captures per-request data (user, model, token counts, EUR cost) but attributes it exclusively to the individual SSO identity. No project identifier is attached to requests. No programmatic spending API exists. Shared keys and shared proxies are prohibited. Three options were evaluated:

**Table: Hyperspace Project Cost Tracking Options**


<table>
<tr>
<th valign="top">

Option

</th>
<th valign="top">

Approach

</th>
<th valign="top">

Feasibility

</th>
<th valign="top">

Timeline

</th>
</tr>
<tr>
<td valign="top">

A — Project API key (shared)

</td>
<td valign="top">

One key per engagement used by all engineers

</td>
<td valign="top">

Not feasible — auth is SSO-bound, not key-bound; shared proxies prohibited

</td>
<td valign="top">

Not on roadmap

</td>
</tr>
<tr>
<td valign="top">

B — Project tracking in HAI proxy backend

</td>
<td valign="top">

Add project/team entities + attribution + management API to proxy

</td>
<td valign="top">

Requires backend work; aligned with planned workspace features

</td>
<td valign="top">

Medium-term (roadmap)

</td>
</tr>
<tr>
<td valign="top">

C — Local accounting sidecar

</td>
<td valign="top">

Lightweight local HTTP middleware intercepts responses, reads token counts from Anthropic API response body, logs with project context

</td>
<td valign="top">

Feasible now; project ID auto-detected from repo

</td>
<td valign="top">

Near-term (CIS tooling initiative)

</td>
</tr>
</table>

**Recommended actions:**

1.  **Near-term**: build Option C as a CIS-internal tool; integrate into `apm install` setup
2.  **Medium-term**: submit Option B as a formal FDE requirement to the HAI proxy workspace roadmap

**Who can answer / action this:** CIS T&O (Option C implementation); HAI proxy / DTS tooling team (Option B roadmap input).

**External links:**

-   [Submit feature request](https://github.tools.sap/hAIperspace/hai-requests) — `hAIperspace/hai-requests` GitHub Issues; use "Submit Feature Request" template
-   [HAI proxy FAQ — workspace roadmap](https://github.tools.sap/hAIperspace/hai-docs/blob/main/documentation/docs/llm-proxy/faq.md) — search "workspace" for the planned workspace-level dashboards and rate limits
-   [HAI proxy What's New](https://github.tools.sap/hAIperspace/hai-docs/blob/main/documentation/docs/llm-proxy/whats-new.md) — search "Simplified Authentication" for the February 2026 entry documenting why workspace selection was removed and what's coming


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q17"/>

## Q17 — AI Units vs. Hyperspace: Separate Cost Streams? Any Runtime Proxy Available?

**Status:** Open — Hyperspace is SAP-internal; not documented in public SAP Help; BAIP commercial team required

**Additional evidence (2026-10-22):**

A SAP Help search for "Hyperspace SAP AI runtime token consumption metering" returned no results for Hyperspace. This confirms Hyperspace is SAP-internal only and will not appear in public product documentation. The AI Core metering page confirms the runtime billing structure: model tokens → GenAI tokens (virtual unit, model-specific conversion rate) → capacity units (CU, billing metric for SAP BTP). A third line item is "Inference Observability" (storing inference records, measured as DataVolume), referenced in SAP Note 3720903.

The two-cost-stream assumption in the concept (build-time via Hyperspace, runtime via AI Units / CU) is broadly correct as an architectural distinction. The specific question about whether a unified cross-stream view exists for CIS program management remains open.

**Context:**

[Production Operations](../Production-Operations/production-operations-edd9e33.md) and [Token Efficiency in Industry AI Delivery](../Token-Efficiency/token-efficiency-47721ff.md) distinguish two cost streams:

-   **Build-time (Hyperspace)**: tracks LLM token consumption for FDE engineers using Claude Code and coding tools; SAP-internal only
-   **Runtime (AI Units)**: commercial billing unit for deployed Industry AI agents; visible in the customer's SAP subscription; measured per agent step at three tiers (Basic / Standard / Advanced)

**The specific uncertainties:**

1.  Are these entirely separate billing systems with no shared reporting or governance structure? Or is there a unified cost view available to CIS/FDE program management that covers both pools?

2.  Is there any operator-side tool equivalent to Hyperspace for monitoring deployed agent token costs from SAP's perspective — outside of the customer-visible AI Units billing? For example, an SAP-internal view of AI Core usage across customer tenants, accessible to the operating CIS team?

3.  The current concept assumes no such tool exists for runtime (SAP-internal only constraint). If a runtime equivalent exists, [Token Efficiency in Industry AI Delivery](../Token-Efficiency/token-efficiency-47721ff.md) would need updating.

**Practical implication:**

If a runtime-side operator view exists (even read-only), CIS could potentially monitor runtime token efficiency across deployed agents without requiring per-customer Topology A consent. This would significantly simplify the operational monitoring model described in Chapters 8 and 10.

**Who can answer this:** BAIP commercial team; SAP AI Core product team; DTS/Hyperspace team; CIS T&O team responsible for BAIP subscription management.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q18"/>

## Q18 — Is Model Availability Consistent Across All Industry AI Customer Regions?

**Status:** ANSWERED — real gaps confirmed; model names in concept also outdated (cross-validated with Joule, 2026-10-22)

**Model naming correction (critical):**

The concept references GPT-4o and Claude 3.5 Sonnet — both are **deprecated** in SAP GenAI Hub as of October 2026:

-   **GPT-4o** → being replaced by the **GPT-5.6 series** (Sol, Terra, Luna); gpt-4o explicitly deprecated in SDK release notes
-   **Claude 3.5 Sonnet** → superseded by **Claude 4.x / Claude Sonnet 4.x**; AWS Bedrock (which backs SAP's Anthropic models) is ending Claude 3.5 Sonnet support on **Nov 13, 2026**
-   **Current SAP GenAI Hub frontier (Q2 2026)**: OpenAI GPT-5.2 / GPT-5.6, Anthropic Claude 4.x / Opus 4.x, Gemini 3.x

**Regional availability gaps:**

SAP explicitly states model availability is **not uniform** across BTP regions:

**Table: SAP BTP Region Model Coverage**


<table>
<tr>
<th valign="top">

Region Group

</th>
<th valign="top">

Frontier Model Coverage

</th>
<th valign="top">

Notes

</th>
</tr>
<tr>
<td valign="top">

eu10 (EU Frankfurt), us10 (US East)

</td>
<td valign="top">

Broadest

</td>
<td valign="top">

Core hyperscaler regions — most models available

</td>
</tr>
<tr>
<td valign="top">

ap10 (Sydney), ap11 (Singapore), jp10 (Tokyo)

</td>
<td valign="top">

Strong

</td>
<td valign="top">

AP regions documented as broad-coverage

</td>
</tr>
<tr>
<td valign="top">

eu11 (EU Frankfurt EU Access)

</td>
<td valign="top">

Partial

</td>
<td valign="top">

EU data residency enforced — models must run on EU Azure/AWS; not all providers comply

</td>
</tr>
<tr>
<td valign="top">

eu20, eu30

</td>
<td valign="top">

Unconfirmed

</td>
<td valign="top">

Likely partial; not explicitly confirmed in public docs

</td>
</tr>
<tr>
<td valign="top">

us21, us30

</td>
<td valign="top">

Unconfirmed

</td>
<td valign="top">

Not mapped to model entitlements in public sources

</td>
</tr>
<tr>
<td valign="top">

br10 (Brazil), in30 (India)

</td>
<td valign="top">

Likely partial

</td>
<td valign="top">

Secondary regions; sources flag as potentially having reduced catalogs

</td>
</tr>
<tr>
<td valign="top">

China

</td>
<td valign="top">

Not available

</td>
<td valign="top">

SAP AI Core / GenAI Hub not available in China BTP regions

</td>
</tr>
</table>

**What drives the gaps:**

1.  **Hyperscaler backend**: Azure OpenAI backs GPT; AWS Bedrock backs Anthropic. If a BTP region has no matching hyperscaler region for a model, it simply won't appear in the catalog.
2.  **Data residency**: eu11 enforces data never leaves EU — restricts which Azure/AWS endpoints SAP can route to.
3.  **Extended plan requirement**: GenAI Hub only activates on the Extended SAP AI Core service plan; Free/Standard customers won't see any models regardless of region.
4.  **Dynamic catalog**: model list changes quarterly; models get added, versioned, and deprecated.

**Authoritative source:**

**SAP Note 3437766** ("Availability of Generative AI Models") is the only definitive per-region model matrix. Includes model IDs/versions per region, token conversion rates, rate limits, and deprecation dates. SAP Help's "Supported Models" page redirects to this Note.

**Practical implication:**

For Industry AI customer conversations in eu11, br10, in30, or China: **do not assume eu10/us10 model availability**. Build a region compatibility check into the onboarding flow ([Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md) and [Agent Evaluation in FDE Sprints](../Agent-Evaluation/agent-evaluation-76db9f7.md) eval setup may need model substitutions requiring re-calibration).


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q19"/>

## Q19 — Is the Week-by-Week Build Sequence () Realistic for a Standard FDE Sprint?

**Status:** Open — needs review by FDE practitioners with active engagement experience

**Context:**

[Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md) describes a 12-week build sequence: from scaffold in week 1 to production readiness in week 12. *(Note: was restructured in the November 2026 Wave 2/3 pass into a Minimum-Viable / Full-Stack two-path sequence; this summary predates that and should be re-synced when Q19 is actioned.)* The sequence covers: environment setup, local agent scaffold, JS2.0/SMJ (default) or CMJ wiring, Workflow A (supervisor), Workflow B (telemetry), Workflow C (support relay), multi-agent patterns, TR1–TR14 compliance, and Enable track delivery.

**The specific uncertainty:**

The full 12-week sequence was designed for maximum scope. In a standard ~90-day FDE sprint (the majority of engagements), the realistic question is: which subset of this sequence is achievable, and in what order should items be deprioritized if time is short?

Also: the sequence was originally designed with CMJ provisioning as a potential early bottleneck. **With JS2.0/SMJ as the default path, this concern is simpler** — no CMJ provisioning step; the pre-condition is that the customer's BAIP/SMJ subscription is active (a standard BTP subscription, not a custom CMJ provisioning process). For new engagements the pre-week-0 checklist should confirm the BAIP/SMJ subscription is in place, not CMJ provisioning. For in-progress CMJ engagements, the original CMJ provisioning concern still applies.

**Practical implication:**

A clarifying note in [Joule Studio Build Map](../Joule-Studio-Build-Map/joule-studio-build-map-dcd2de0.md) about which items are mandatory (core agent + Workflow A + TR1–TR14 minimum) vs. optional (Workflow B, Workflow C, multi-agent) would make the build sequence more directly usable by FDE teams planning sprint scope. The JS2.0/SMJ default also simplifies the early environment setup steps (no manual CMJ provisioning, no manual Joule capability bundle) — reducing the risk of week 1 delays.

**Who can answer this:** FDE practitioners with Joule Studio engagement experience; CIS T&O team.


<a name="loioa12bf85b576b4a78a846573ed18cd202__section_q20"/>

## Q20 — How Should Inter-Agent Structured-Output Contracts Be Managed in Multi-Agent Rollbacks?

**Status:** Open — no concrete tooling identified in the BAIP harness

**Context:**

[Production Operations](../Production-Operations/production-operations-edd9e33.md) notes that in multi-agent deployments, rolling back one worker agent may break interface contracts with other agents if the rolled-back version produces different structured output than the version the other agents were designed to consume.

The CIS T&O runbook ([CIS CoE Operations Model](../CIS-CoE-Operations-Model/cis-coe-operations-model-1af392f.md)) should include a dependency map of inter-agent structured-output contracts. However, no concrete tooling for tracking these contracts exists in the current BAIP harness or Joule Studio environment.

**The specific uncertainty:**

-   Should inter-agent contracts be tracked manually (e.g., a dependency map document maintained alongside the workflow versions) or can they be inferred from the workflow configuration?
-   When the BAIP harness records a version event, does it capture the interface schema of the agent's A2A output, enabling automated contract compatibility checking?
-   Is there a recommended versioning pattern (semver for structured outputs?) that would make breaking vs. non-breaking changes detectable without full integration testing?

**Practical implication:**

Until tooling exists, the CIS T&O operational runbook for multi-agent deployments must include a manual "interface contract map" — listing which agents consume which other agents' outputs and what schema version they expect. The map must be updated as part of any version event. Without it, rollback of a low-level worker agent in a multi-agent configuration is operationally risky.

**Who can answer this:** BAIP/Joule Studio platform team; CIS T&O; FDE architects with multi-agent delivery experience.

**Related Information**  




[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "Introduction to SAP AI Agent Hub, its benefits, and core capabilities.") :arrow_upper_right:
[Code-Based Agents (Bring Your Own Agent)](https://help.sap.com/docs/JOULE/ba88d1ec6a1b442098863d577c19b0c0/2f9701e60fd24e948c2c74fe9e55ce23.html "How to register a custom code-based agent (BYOA) to Joule via A2A.") :arrow_upper_right:
[Proof of Concept Plan](../Proof-of-Concept-Plan/proof-of-concept-plan-b1c9aa0.md)

[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html)

[SAP Managed n8n Workflows](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/5f16a77e21d047dc968de9f7e83bca54.html)
