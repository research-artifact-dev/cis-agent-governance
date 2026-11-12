<!-- loio05fedccf487f4a119e89f4f296c9d300 -->

# Customer Enablement and Adoption

This chapter covers the Enable track — the mandatory parallel workstream that ensures agents are not only built and operated, but actually adopted. It defines the Adoption Strategy artifact, the gradual release of responsibility model, role-based training, and the enablement readiness checklist required before productization handover.



<a name="loio05fedccf487f4a119e89f4f296c9d300__section_enablement_first_class"/>

## Why Enablement Is a First-Class Delivery Concern

Earlier chapters address how agents are built, secured, evaluated, and operated. This chapter addresses a different failure mode: technically complete agents that are never adopted. Industry AI FDE engagements have delivered working agents that customers did not use — not because the agent was wrong, but because the customer organization was not prepared to change how it worked.

The FDE Engagement Playbook (Oct 2026) formalizes the **Enable track** as a mandatory, parallel workstream alongside Build and Operate. Enablement is not a Phase 4 activity. It begins in Discovery and targets Stage 4 independence — the point at which the customer team can operate, monitor, adjust, and evolve the agent without FDE support. **Stage 3 is the minimum required at handover; Stage 4 is the target** (see the *Gradual Release of Responsibility* section below).

The key premise: **delivering an agent and enabling adoption of that agent are two distinct deliverables.** The first is a software artifact. The second is an organizational capability. Both must be complete before an FDE engagement is done.



<a name="loio05fedccf487f4a119e89f4f296c9d300__section_three_track_model"/>

## The Three-Track FDE Model

The FDE Engagement Playbook introduces a three-track model that runs in parallel throughout the engagement:

**Three-Track FDE Model**


<table>
<tr>
<th valign="top">

Track

</th>
<th valign="top">

What It Delivers

</th>
<th valign="top">

Owner

</th>
</tr>
<tr>
<td valign="top">

**Build**

</td>
<td valign="top">

The agent: workflow, prompts, evaluation suite, infrastructure

</td>
<td valign="top">

FDE Application Engineers + Dev Architect

</td>
</tr>
<tr>
<td valign="top">

**Operate**

</td>
<td valign="top">

The monitoring layer: supervisor workflows, operational runbook, threshold calibration, CIS T&O setup

</td>
<td valign="top">

FDE Dev Architect + CIS T&O

</td>
</tr>
<tr>
<td valign="top">

**Enable**

</td>
<td valign="top">

The organizational capability: Adoption Strategy, role-based training, gradual responsibility transfer, handover readiness

</td>
<td valign="top">

FDE Domain PM + Adoption Lead

</td>
</tr>
</table>

The three tracks converge at productization. None can substitute for the others: a fully built and monitored agent that hasn't been adopted is a cost center. A well-adopted agent without operational monitoring drifts without detection.



<a name="loio05fedccf487f4a119e89f4f296c9d300__section_adoption_strategy"/>

## Adoption Strategy — The Formal Engagement Deliverable

The Adoption Strategy is the primary Enable track artifact. It is not a communication plan or a training calendar. It is a structured accountability document that defines what adoption success looks like, who is accountable, and how the customer organization will sustain and evolve the agent after the FDE team departs.

### The Seven-Section Structure

The Adoption Strategy (defined by the Adoption Strategy Skill v2.4 in the `book-of-fde` repository) has seven mandatory sections:

**Adoption Strategy — Seven-Section Structure**


<table>
<tr>
<th valign="top">

Section

</th>
<th valign="top">

Content

</th>
</tr>
<tr>
<td valign="top">

**1. AI Use Case Details**

</td>
<td valign="top">

Use case description, business value hypothesis, success criteria, and measurable outcomes the deployment is expected to achieve

</td>
</tr>
<tr>
<td valign="top">

**2. Expected Business Value**

</td>
<td valign="top">

Quantified value case: cost reduction, time saved, error rate reduction, throughput improvement — the metrics the customer and CIS will track together

</td>
</tr>
<tr>
<td valign="top">

**3. Adoption Governance & Accountability**

</td>
<td valign="top">

Nine defined governance roles with named individuals, decision rights, and escalation paths. The full nine-role roster is maintained in the Adoption Strategy governance section

</td>
</tr>
<tr>
<td valign="top">

**4. Adoption Baseline & Action Plan**

</td>
<td valign="top">

Five readiness areas assessed and scored: technical, data, process, user, and governance readiness (see the *Change Management and Organizational Readiness* section). Action plan for any gap areas

</td>
</tr>
<tr>
<td valign="top">

**5. Deployment Strategy + Scale Gate**

</td>
<td valign="top">

Rollout plan: pilot group → broader rollout → full adoption. Scale gate criteria: what conditions must be met before each phase expansion

</td>
</tr>
<tr>
<td valign="top">

**6. Adoption Deployment Plan**

</td>
<td valign="top">

Milestone table with owners and dates: training completions, technical go-lives, readiness checkpoints

</td>
</tr>
<tr>
<td valign="top">

**7. Top Adoption Risks & Mitigation**

</td>
<td valign="top">

The top 5–7 adoption risks (typically: user resistance, data quality, process change management, IT capacity, regulatory) with named risk owners and mitigation actions

</td>
</tr>
</table>

**The nine governance roles** required by Section 3 are: Executive Sponsor, Adoption Lead, Business Lead, Customer IT Lead, Account Executive, Enterprise Architect, Discovery/Execution Lead, Product/Solution Manager, and Test Lead. All nine must be named by end of Discovery. The delivery-side and advisory roles that appear in this chapter are **not** part of that nine-role customer-governance roster: the **FDE Domain PM** (Enable-track owner on the FDE side) and the **Value Advisor** are FDE/advisory roles, while "**product owner**" (used in the scale gate criteria) maps to the roster's **Product / Solution Manager**.

### Production of the Adoption Strategy Across the Engagement Lifecycle

**Adoption Strategy Production Schedule**


<table>
<tr>
<th valign="top">

Phase

</th>
<th valign="top">

What Is Produced

</th>
</tr>
<tr>
<td valign="top">

**Discovery (weeks 1–3)**

</td>
<td valign="top">

Sections 1–2: Use Case Details (what the agent does, what success looks like) and Expected Business Value (quantified, measurable)

</td>
</tr>
<tr>
<td valign="top">

**Discovery (weeks 4–6)**

</td>
<td valign="top">

Section 3: Governance & Accountability (nine roles named) and Section 7: Top Adoption Risks (first draft)

</td>
</tr>
<tr>
<td valign="top">

**Develop (weeks 1–6)**

</td>
<td valign="top">

Sections 4–6: Readiness Baseline, Deployment Strategy, Milestone Table — refined as agent takes shape

</td>
</tr>
<tr>
<td valign="top">

**EAC + Productization**

</td>
<td valign="top">

All sections reviewed, updated from EAC learnings, signed by Executive Sponsor

</td>
</tr>
</table>

The Adoption Strategy is a living document. It is never "done" in a sprint — it is updated each time new information changes the adoption picture.

### Adoption Strategy Quality Gate

At the productization handover gate, the Adoption Strategy must meet:

-   All 7 sections complete with specific, measurable content (not placeholder text)
-   All 9 governance roles named with actual individuals (not job titles)
-   Section 5 scale gate criteria confirmed with the Business Lead
-   Section 7 risks reviewed and risk owners confirmed
-   Executive Sponsor signature obtained

An Adoption Strategy that fails this gate is a productization blocker, not a deferred action. Without it, the agent enters production with no organizational accountability structure.



<a name="loio05fedccf487f4a119e89f4f296c9d300__section_gradual_release"/>

## Gradual Release of Responsibility

### The Four-Stage Model

The Enable track transfers operational responsibility to the customer using a structured four-stage model. Stages are sequential — each requires explicit readiness confirmation before the next begins. Do not skip stages under time pressure; a customer who hasn't completed Stage 3 (Reverse Shadowing) is not ready for independent operation.

**Four-Stage Responsibility Transfer Model**


<table>
<tr>
<th valign="top">

Stage

</th>
<th valign="top">

What Happens

</th>
<th valign="top">

Who Leads

</th>
<th valign="top">

Evidence That Stage Is Complete

</th>
</tr>
<tr>
<td valign="top">

**Stage 1 — Demonstration**

</td>
<td valign="top">

FDE team demonstrates the agent in full operational mode. Customer stakeholders observe all scenarios: normal operation, HITL approval, anomaly alert, rollback.

</td>
<td valign="top">

FDE team

</td>
<td valign="top">

Customer team can accurately describe agent behavior in their own words; no questions about basic operation remain

</td>
</tr>
<tr>
<td valign="top">

**Stage 2 — Shadowing**

</td>
<td valign="top">

Customer IT admin and process owner shadow FDE team during live operations. All decisions are made by the FDE team; customer team asks questions in context.

</td>
<td valign="top">

FDE team

</td>
<td valign="top">

Customer team can predict FDE team's decisions before they are made 80% of the time

</td>
</tr>
<tr>
<td valign="top">

**Stage 3 — Reverse Shadowing**

</td>
<td valign="top">

Customer team takes the wheel. FDE team observes silently. FDE team may coach after a decision, not before.

</td>
<td valign="top">

Customer team

</td>
<td valign="top">

Customer team completes one full operational cycle (including at least one alert or approval event) without FDE intervention

</td>
</tr>
<tr>
<td valign="top">

**Stage 4 — Independent Operation**

</td>
<td valign="top">

Customer team operates without FDE support. FDE team is available as escalation contact only.

</td>
<td valign="top">

Customer team

</td>
<td valign="top">

Customer team handles three consecutive incidents or threshold adjustment requests independently; FDE team is not contacted for routine operations

</td>
</tr>
</table>

**Enable exit criterion for FDE handover: Stage 3 minimum; Stage 4 target.**

If the engagement timeline does not permit Stage 4 before the FDE team exits, document this as a post-handover risk in the Adoption Strategy Section 7, name the FDE contact for post-handover coaching support, and set a 30-day target for Stage 4 achievement.

### Handover Activities by Stage

**Required Handover Activities by Stage**


<table>
<tr>
<th valign="top">

Stage Transition

</th>
<th valign="top">

Required Handover Activities

</th>
</tr>
<tr>
<td valign="top">

Stage 1 → 2

</td>
<td valign="top">

FDE team provides: agent architecture briefing, BAIP access setup for customer IT admin, copy of operational runbook

</td>
</tr>
<tr>
<td valign="top">

Stage 2 → 3

</td>
<td valign="top">

FDE team delivers: threshold calibration workshop (customer IT admin sets thresholds based on EAC data), Joule Studio hands-on session (customer IT admin creates a test workflow), Workflow B alert drill (FDE injects a synthetic alert; customer team triages)

</td>
</tr>
<tr>
<td valign="top">

Stage 3 → 4

</td>
<td valign="top">

Customer IT admin independently completes: one prompt rollback exercise, one threshold adjustment (following the documented procedure), one full runbook walkthrough with the CIS T&O team

</td>
</tr>
</table>

### Scale Gate Criteria Template

The transition from EAC (Early Adopter Care) operation to standard production scale requires explicit criteria agreed between the FDE team, the customer, and the product owner. The structure below is standardized — the values in brackets are engagement-specific and must be agreed in Discovery or during EAC, not at productization.

**Scale Gate Criteria Template**


<table>
<tr>
<th valign="top">

Criterion

</th>
<th valign="top">

Standard Structure

</th>
<th valign="top">

Engagement-Specific Value

</th>
</tr>
<tr>
<td valign="top">

**Stage 3 independence**

</td>
<td valign="top">

Customer team has completed Stage 3 (Reverse Shadowing)

</td>
<td valign="top">

Mandatory — no exception

</td>
</tr>
<tr>
<td valign="top">

**HITL override rate**

</td>
<td valign="top">

Override rate is below [X]% for a rolling 30-day window

</td>
<td valign="top">

Set [X] with the Business Lead during Discovery; typical range 15–35% depending on process risk

</td>
</tr>
<tr>
<td valign="top">

**Process KPI achievement**

</td>
<td valign="top">

Primary process KPI (Dimension 2) is within [Z]% of the TO-BE target

</td>
<td valign="top">

[Z] = tolerance agreed with Value Advisor; typically ±10–20%

</td>
</tr>
<tr>
<td valign="top">

**Quality stability**

</td>
<td valign="top">

aeval online eval success rate ≥ [threshold]% with no declining trend over 30 days

</td>
<td valign="top">

[threshold] typically 80–90%; agreed with the Business Lead in Discovery and recorded in Adoption Strategy Section 5. Set independently — the Week-9 milestone is a qualitative stability gate, not a numeric success-rate threshold

</td>
</tr>
<tr>
<td valign="top">

**Incident record**

</td>
<td valign="top">

Zero Critical incidents and ≤ [N] High incidents in the prior 30 days

</td>
<td valign="top">

[N] typically 1–2; zero tolerance for Critical

</td>
</tr>
<tr>
<td valign="top">

**Model currency**

</td>
<td valign="top">

No pinned model versions with an active deprecation date in the next 90 days

</td>
<td valign="top">

Fixed criterion — no engagement-specific value

</td>
</tr>
<tr>
<td valign="top">

**Handover artifacts**

</td>
<td valign="top">

All enablement artifacts present and accepted by the customer (see the *Enablement Artifacts Summary* section)

</td>
<td valign="top">

Fixed criterion — no exception

</td>
</tr>
</table>

**How to use this template**: populate the engagement-specific values during Discovery (or at the latest during the first EAC review) and record them in the Adoption Strategy Section 5. These values become the formal scale gate criteria — reaching them is the trigger for productization sign-off and standard production transition. If the criteria have not been agreed at the time of scale gate review, the gate cannot be called passed.



<a name="loio05fedccf487f4a119e89f4f296c9d300__section_role_based_training"/>

## Role-Based Training

Training is not one-size-fits-all. The Enable track delivers seven distinct training audiences, each with different content depth and format requirements.

### Training Matrix

**Seven-Audience Training Matrix**


<table>
<tr>
<th valign="top">

Audience

</th>
<th valign="top">

Depth

</th>
<th valign="top">

Primary Format

</th>
<th valign="top">

Key Content

</th>
</tr>
<tr>
<td valign="top">

**Executive Sponsor**

</td>
<td valign="top">

Conceptual

</td>
<td valign="top">

30-min briefing

</td>
<td valign="top">

Business outcomes, value case progress, escalation path, what success looks like at 30/60/90 days

</td>
</tr>
<tr>
<td valign="top">

**Business Lead / Process Owner**

</td>
<td valign="top">

Functional

</td>
<td valign="top">

2-hour workshop

</td>
<td valign="top">

HITL responsibilities (when to approve, reject, modify), threshold rationale, process KPI dashboard, how to request a change

</td>
</tr>
<tr>
<td valign="top">

**Customer IT Admin**

</td>
<td valign="top">

Technical

</td>
<td valign="top">

Half-day hands-on

</td>
<td valign="top">

Agent architecture, BAIP registrations, Joule Studio configuration, Workflow B alert interpretation, rollback procedure (hands-on exercise), AI Agent Hub navigation

</td>
</tr>
<tr>
<td valign="top">

**Joule End Users**

</td>
<td valign="top">

User-facing

</td>
<td valign="top">

1-hour guided session + quick reference card

</td>
<td valign="top">

How to invoke the agent in Joule, what to do when an approval task arrives in Task Center, how to flag a wrong answer

</td>
</tr>
<tr>
<td valign="top">

**CIS T&O Operators**

</td>
<td valign="top">

Operational

</td>
<td valign="top">

Runbook handover session

</td>
<td valign="top">

Alert escalation tree, in-place fix procedures, when to trigger a new sprint, how to access Topology A customer data with consent

</td>
</tr>
<tr>
<td valign="top">

**Customer Security / Compliance**

</td>
<td valign="top">

Governance

</td>
<td valign="top">

1-hour evidence walkthrough

</td>
<td valign="top">

Authorization matrix, audit trail structure, how to pull a specific transaction record, data protection boundaries

</td>
</tr>
<tr>
<td valign="top">

**Customer IT Support / Help Desk**

</td>
<td valign="top">

Triage

</td>
<td valign="top">

Quick reference card + 30-min briefing

</td>
<td valign="top">

First-line triage checklist: is this an agent issue or an SAP platform issue? Who to call. How to identify the correlation_id for escalation.

</td>
</tr>
</table>

### Training Materials as Handover Artifacts

All training materials must be delivered as durable artifacts, not just session slides:

**Training Materials — Handover Artifact Formats**


<table>
<tr>
<th valign="top">

Artifact

</th>
<th valign="top">

Format

</th>
<th valign="top">

Owner Post-Handover

</th>
</tr>
<tr>
<td valign="top">

Executive value case summary

</td>
<td valign="top">

1-page PDF

</td>
<td valign="top">

Account Executive

</td>
</tr>
<tr>
<td valign="top">

Business Lead HITL decision guide

</td>
<td valign="top">

Laminated reference card + slide deck

</td>
<td valign="top">

Business Lead

</td>
</tr>
<tr>
<td valign="top">

IT Admin operational runbook

</td>
<td valign="top">

Markdown file in customer's git / Confluence

</td>
<td valign="top">

Customer IT Admin + CIS T&O

</td>
</tr>
<tr>
<td valign="top">

End user quick reference card

</td>
<td valign="top">

PDF, 2 pages

</td>
<td valign="top">

Customer IT Admin

</td>
</tr>
<tr>
<td valign="top">

T&O runbook (6 scenarios)

</td>
<td valign="top">

Markdown file

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Security evidence guide

</td>
<td valign="top">

Document in customer's compliance portal

</td>
<td valign="top">

Customer Security / Compliance

</td>
</tr>
<tr>
<td valign="top">

Help Desk triage card

</td>
<td valign="top">

IT support ticketing system template

</td>
<td valign="top">

Customer IT Support

</td>
</tr>
</table>

Training materials that exist only in the FDE team's heads or in a sprint archive are not handover artifacts. They must be in a system the customer organization owns and can maintain.



<a name="loio05fedccf487f4a119e89f4f296c9d300__section_readiness_checklist"/>

## Enablement Readiness Checklist

The following checklist must be complete before productization handover is signed off. This is the Enable track's equivalent of the technical readiness checklist.

**Governance readiness:**

-   All 9 adoption governance roles named with specific individuals
-   Executive Sponsor has confirmed business case and success criteria
-   Business Lead has accepted HITL decision responsibility in writing
-   Adoption Strategy Sections 1–7 complete and signed

**Technical transfer:**

-   Customer IT Admin has Joule Studio builder access confirmed
-   Customer IT Admin has AI Agent Hub read access confirmed
-   Customer IT Admin can navigate to Workflow B telemetry independently
-   At least one rollback exercise completed by customer IT admin (not by FDE team)

**Training completions:**

-   Executive Sponsor briefing delivered and acknowledged
-   Business Lead workshop completed (HITL decision guide delivered)
-   IT Admin technical session completed (hands-on, not just slides)
-   End user training session completed for pilot user group
-   T&O runbook handover session completed with CIS T&O team
-   Security evidence walkthrough completed with compliance team
-   Help Desk triage briefing completed

**Drills:**

-   At least one Workflow B alert drill completed (synthetic alert → customer team triages → resolution without FDE)
-   At least one threshold adjustment exercise completed (customer IT admin makes adjustment following the documented procedure)
-   At least one HITL approval decision executed by the Business Lead (not FDE team guidance)

**Enable stage:**

-   Stage 3 achieved (customer team completes an operational cycle without FDE intervention)
-   Stage 4 target date confirmed if Stage 4 not yet reached

**Post-handover plan:**

-   30/60/90-day review schedule confirmed with customer contacts
-   CIS T&O contact list confirmed and handed to customer IT Admin
-   Hypercare period start date and scope agreed



<a name="loio05fedccf487f4a119e89f4f296c9d300__section_change_management"/>

## Change Management and Organizational Readiness

The five readiness areas from the Adoption Strategy Section 4 are assessed during Discovery and reviewed at EAC. Each area has specific implications for the Enable track:

**Five Readiness Areas — Enable Track Response**


<table>
<tr>
<th valign="top">

Readiness Area

</th>
<th valign="top">

What It Measures

</th>
<th valign="top">

Enable Track Response

</th>
</tr>
<tr>
<td valign="top">

**Technical readiness**

</td>
<td valign="top">

Is the customer's SAP landscape and BAIP/Joule setup suitable for the agent?

</td>
<td valign="top">

Confirm in Discovery; address blocking gaps before the Develop sprint starts

</td>
</tr>
<tr>
<td valign="top">

**Data readiness**

</td>
<td valign="top">

Is the grounding data accurate, complete, and properly governed?

</td>
<td valign="top">

RAG corpus quality assessment; data owner identified; CDH access confirmed if real data used

</td>
</tr>
<tr>
<td valign="top">

**Process readiness**

</td>
<td valign="top">

Has the TO-BE process been designed, validated, and communicated to affected staff?

</td>
<td valign="top">

HITL decision procedures documented; escalation paths confirmed with process owner

</td>
</tr>
<tr>
<td valign="top">

**User readiness**

</td>
<td valign="top">

Do the affected users know the agent is coming and what it will change about their work?

</td>
<td valign="top">

Change communication plan; end user training delivered before EAC; pilot user group identified

</td>
</tr>
<tr>
<td valign="top">

**Governance readiness**

</td>
<td valign="top">

Are the nine adoption governance roles in place and active?

</td>
<td valign="top">

Governance table complete; accountability confirmed; Adoption Strategy Section 3 finalized

</td>
</tr>
</table>

### Managing Resistance

The most common adoption failure mode is not technical — it is user resistance. An agent that recommends decisions that humans consistently override is not a failing agent; it is an agent that hasn't been adopted. Distinguish these:

**Resistance Signal Interpretation**


<table>
<tr>
<th valign="top">

Signal

</th>
<th valign="top">

Interpretation

</th>
<th valign="top">

Enable Track Response

</th>
</tr>
<tr>
<td valign="top">

High human override rate within first 30 days

</td>
<td valign="top">

Users are overriding without reading the agent's reasoning

</td>
<td valign="top">

Review HITL decision guide training; check if users have sufficient context in Task Center

</td>
</tr>
<tr>
<td valign="top">

High override rate stable after 60 days

</td>
<td valign="top">

Agent outputs are systematically wrong for a subset of cases

</td>
<td valign="top">

Quality issue: trigger eval review and prompt improvement cycle

</td>
</tr>
<tr>
<td valign="top">

Low agent invocation rate

</td>
<td valign="top">

Users are bypassing the agent

</td>
<td valign="top">

Process change management issue: confirm the agent is in the workflow path; escalate to Business Lead

</td>
</tr>
<tr>
<td valign="top">

Users asking IT to disable the agent

</td>
<td valign="top">

Fundamental resistance or a visible failure event

</td>
<td valign="top">

Executive Sponsor escalation; temporary HITL gate tightening while root cause is addressed

</td>
</tr>
</table>

### Business Lead HITL Decision Guide — Legitimate Override vs. Operator Error

The Business Lead is the accountability owner for HITL decisions (see the *Role-Based Training* section above). Distinguishing a *legitimate override* (the Business Lead had correct contextual information the agent lacked) from an *operator error* (the Business Lead overrode a correct agent output due to unfamiliarity or mistrust) is critical for the feedback loop.

**A legitimate override** is one where the Business Lead can articulate a specific reason the agent output was wrong for this decision, citing information not available to the agent at the time (for example, "the plant is under maintenance this week and the system doesn't reflect that yet" or "this order has a negotiated exception approved by procurement last week"). Legitimate overrides should be documented with the specific reason.

**An operator error** is one where the Business Lead overrides a correct output because: (a) they don't trust the agent yet, (b) they prefer the old process habit, or (c) they misread the agent's recommendation. These are training signals, not spec change triggers.

**HITL Override Classification Guide**


<table>
<tr>
<th valign="top">

Override Behavior

</th>
<th valign="top">

Classification

</th>
<th valign="top">

Correct Response

</th>
</tr>
<tr>
<td valign="top">

Business Lead cites specific real-world context not in the agent's data

</td>
<td valign="top">

Legitimate override

</td>
<td valign="top">

Log as override; review whether the missing context should be added to the agent's grounding or as a HITL decision criterion

</td>
</tr>
<tr>
<td valign="top">

Business Lead cannot articulate why the agent output was wrong

</td>
<td valign="top">

Likely operator error

</td>
<td valign="top">

Flag for operator training; do NOT revise the spec; increase training depth on the Business Lead HITL decision guide

</td>
</tr>
<tr>
<td valign="top">

Override rate > 30% within first 2 weeks

</td>
<td valign="top">

Expected (trust calibration phase)

</td>
<td valign="top">

Normal; review training completeness; re-deliver HITL decision guide as needed

</td>
</tr>
<tr>
<td valign="top">

Override rate > 30% after 60 days

</td>
<td valign="top">

Systematic issue

</td>
<td valign="top">

Escalate: either the agent has a quality problem (trigger eval review) or the Business Lead role is not adequately staffed

</td>
</tr>
<tr>
<td valign="top">

Consistent override on a specific output type

</td>
<td valign="top">

Pattern signal

</td>
<td valign="top">

Review whether that output type needs a tighter precision guardrail, a different approval threshold, or a separate HITL stage

</td>
</tr>
</table>

This guide should be included in the Business Lead training materials (see training matrix above) and kept as a laminated reference card alongside the HITL decision guide artifact.



<a name="loio05fedccf487f4a119e89f4f296c9d300__section_coe_support_model"/>

## CoE Support Model Post-Handover

After productization handover, the CIS CoE provides three tiers of ongoing support.

### Tier 1 — Operational Support (via CIS T&O)

Available through the post-engagement operating rhythm:

-   Weekly hypercare reviews (first 30 days)
-   Monthly quality reviews
-   Quarterly improvement reviews

**What CIS T&O provides:**

-   Advice on Workflow B alert interpretation
-   In-place fix proposals (threshold adjustments, prompt updates)
-   Root cause analysis for quality regressions
-   Guidance on when to trigger a new sprint vs. in-place fix

**What CIS T&O does NOT provide without a new engagement:**

-   Workflow redesign
-   New tool integrations
-   Scope expansions
-   Regulatory compliance updates requiring workflow changes

### Tier 2 — Product Evolution (via Microdelivery or New FDE Sprint)

When an issue requires more than configuration-level intervention:

-   **Microdelivery**: an incremental feature addition to an existing productized agent or workflow; LoB-led; shorter cycle than a full FDE sprint (typically 2–6 weeks). Used when a specific, well-scoped capability gap is identified post-productization and can be delivered without a full sprint scope. Microdelivery is part of the standard SAP product evolution process for Industry AI agents and is distinct from a new FDE engagement. The trigger and scope must be agreed between the LoB, CIS T&O, and the customer before committing to a Microdelivery cycle.
-   **New FDE engagement**: for scope expansions, new use cases within the domain, or fundamental architectural changes

The continuous improvement loop is the mechanism by which production observations feed back into Microdelivery or sprint planning. CIS T&O must have a defined path to propose improvements, and the customer must have a defined path to accept or defer them.

### Tier 3 — Long-Horizon Governance (Customer IT + CIS CoE)

Beyond the first year, the agent enters steady-state governance:

-   Annual eval suite review (update test cases to reflect evolved process conditions)
-   Annual model suitability review (confirm the model selection is still appropriate)
-   Deprecation classification awareness (Group 1 vs. Group 2)
-   Roadmap alignment: confirm the agent's scope against upcoming SAP product releases



<a name="loio05fedccf487f4a119e89f4f296c9d300__section_enablement_artifacts"/>

## Enablement Artifacts Summary

All Enable track artifacts must be present in the productization handover package:

**Enable Track Handover Artifacts**


<table>
<tr>
<th valign="top">

Artifact

</th>
<th valign="top">

Format

</th>
<th valign="top">

Produced

</th>
<th valign="top">

Owner

</th>
</tr>
<tr>
<td valign="top">

Adoption Strategy (7 sections, signed)

</td>
<td valign="top">

Markdown / document

</td>
<td valign="top">

During engagement; final at productization

</td>
<td valign="top">

Adoption Lead

</td>
</tr>
<tr>
<td valign="top">

IT Admin operational runbook

</td>
<td valign="top">

Markdown

</td>
<td valign="top">

During Develop; finalized at EAC

</td>
<td valign="top">

CIS T&O + Customer IT Admin

</td>
</tr>
<tr>
<td valign="top">

CIS T&O runbook (6 scenarios)

</td>
<td valign="top">

Markdown

</td>
<td valign="top">

During Develop

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

7-audience training materials

</td>
<td valign="top">

Mixed formats per audience

</td>
<td valign="top">

During Develop; delivered before EAC

</td>
<td valign="top">

FDE Domain PM

</td>
</tr>
<tr>
<td valign="top">

Enable track readiness checklist

</td>
<td valign="top">

Checklist

</td>
<td valign="top">

At productization gate

</td>
<td valign="top">

FDE Domain PM

</td>
</tr>
<tr>
<td valign="top">

30/60/90-day review schedule

</td>
<td valign="top">

Calendar invites + scope

</td>
<td valign="top">

At productization handover

</td>
<td valign="top">

CIS T&O + Customer Adoption Lead

</td>
</tr>
<tr>
<td valign="top">

Adoption risks log (Section 7 of Adoption Strategy)

</td>
<td valign="top">

Living document

</td>
<td valign="top">

Maintained during engagement; transferred at handover

</td>
<td valign="top">

Adoption Lead → Customer Adoption Lead

</td>
</tr>
</table>



<a name="loio05fedccf487f4a119e89f4f296c9d300__section_summary"/>

## Summary

**Enable Track Summary**


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

Enable track

</td>
<td valign="top">

Mandatory parallel workstream alongside Build and Operate; begins in Discovery; Stage 3 minimum at handover, Stage 4 independence the target

</td>
</tr>
<tr>
<td valign="top">

Adoption Strategy

</td>
<td valign="top">

7-section living document; produced during engagement; all 9 governance roles named; required productization handover artifact

</td>
</tr>
<tr>
<td valign="top">

Gradual release

</td>
<td valign="top">

4-stage model: Demonstration → Shadowing → Reverse Shadowing → Independent; Stage 3 minimum at handover; Stage 4 target

</td>
</tr>
<tr>
<td valign="top">

Role-based training

</td>
<td valign="top">

7 distinct audiences; each gets appropriate depth and format; training materials must be durable artifacts in customer-owned systems

</td>
</tr>
<tr>
<td valign="top">

Enablement checklist

</td>
<td valign="top">

The enablement readiness checklist is the Enable track's equivalent of TR1–TR14; must be complete before productization sign-off

</td>
</tr>
<tr>
<td valign="top">

Organizational readiness

</td>
<td valign="top">

5 readiness areas assessed in Discovery; user resistance is the most common adoption failure mode; distinguish override-from-resistance from override-from-quality-issue

</td>
</tr>
<tr>
<td valign="top">

Post-handover support

</td>
<td valign="top">

Tier 1 (CIS T&O operating rhythm); Tier 2 (Microdelivery / new sprint); Tier 3 (annual governance); each with defined scope and escalation path

</td>
</tr>
<tr>
<td valign="top">

Hypercare

</td>
<td valign="top">

First 30 days: weekly cadence; 30-day review: technical stability; 60-day: adoption trend; 90-day: business value assessment + hypercare exit decision

</td>
</tr>
</table>

**Related Information**  




[AI Governance — SAP LeanIX](https://help.sap.com/docs/LEANIX/72d375467c1e4dcb872dfa2998b6328d/275887e57a441014876a9243fb67f92b.html "Implement AI governance practices in SAP LeanIX.") :arrow_upper_right:
[Secure and Govern — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/3ea62b45d7f64a8aab15321424ffb37c.html "Verify AI assets against governance criteria and build a verified inventory.") :arrow_upper_right:
[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html)

[SAP AI Agent Hub — Plan and Build](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/e2a29b0c422e4aceadee658f4bbab71b.html)
