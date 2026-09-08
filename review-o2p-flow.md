---
name: review-o2p-flow
description: >
  Architectural review of an O2P (flows-wiki) artifact by 4 specialist agents in parallel:
  Salesforce Comms Cloud Technical Architect, TM Forum / eTOM / ODA Functional Architect,
  TOGAF Strategic Architect, and CTO Cross Validation. Produces a findings table with
  Section, Original Text, each agent's observation, CTO verdict, severity, and suggested action.
  Use when: reviewing any .md under architecture/flows-wiki/ before promoting to curated.
  Invoke: review-o2p-flow <relative-path-to-md>
---

# Skill: review-o2p-flow

## Purpose

Run a structured architectural review of an O2P artifact before promotion to `status: curated`. Replicates the 4-agent method used in the inaugural review (Order-Cancellation + MACD-Disconnect, August 2026) as a repeatable, versioned process.

## Pre-conditions (read before executing)

Before dispatching agents, read:

1. `SOUL.md` — non-negotiables #7 and #8 (never silently override an ADR; record non-obvious decisions)
2. `architecture/flows-wiki/o2p/O2P.md` — B2C white-label framework canon
3. `architecture/decisions/013-order-closure-sequence-activate-ponr-billing.md` — activate→PONR→billing sequence; scope: Alta and Cambio de Plan; bajas = open point (§9·3, §9·6)
4. `architecture/decisions/014-dro-governed-fallout-two-lanes-by-ponr.md` — all-or-nothing; Partial-Fulfillment disabled in TO-BE
5. `architecture/decisions/015-core-master-of-commercial-asset-lifecycle.md` — SF Core is master of commercial Asset
6. `architecture/decisions/telco-009-order-configuration-efficiency-lean-decomposition.md` — Multi-Site OFF; B2C single-site
7. The target MD itself — read in full before dispatching agents

> **Adapt ADR paths** if your project uses different numbering.

## Execution

### Step 1 — Read the target artifact

Read the provided MD. Extract:
- Frontmatter: `journey`, `step`, `movement`, `status`
- Main sections: Flow (Mermaid), Steps (table), Notes
- References to other flows (`[[...]]`), ADRs, and TMF APIs cited

### Step 2 — Dispatch 3 specialist agents in parallel

Dispatch all 3 agents simultaneously via the Agent tool. Each receives the full MD content + the relevant pre-condition documents for their domain.

---

#### Agent 1 — Salesforce Comms Cloud Technical Architect (DRO/SOM)

**Persona:** Senior Salesforce Communications Cloud Technical Architect, specialist in DRO (Dynamic Revenue Orchestrator), SOM (Service Order Management), Comms Cloud OM, Apex, and platform data model.

**Mission:** Review the artifact from the perspective of implementability in Salesforce Comms Cloud:
- Are the SF Objects cited real and correct? (e.g. `FulfillmentRequest` ≠ `Service Order`)
- Are the DRO steps implementable as orchestration items / compensating actions?
- Are the flow states testable (unit + integration)?
- Is there decomposition logic the DRO needs but the artifact omits?
- Are TMF688 return events and timeout guards present for async states?
- Consistency with ADR-013 (activate→close→billing), ADR-014 (all-or-nothing), ADR-015 (Asset master)?

**Expected output:** list of findings in the format:
```
SECTION | ORIGINAL TEXT | OBSERVATION | SEVERITY (🔴/🟡/🟢/⚪)
```

---

#### Agent 2 — TM Forum / eTOM / ODA Functional Architect

**Persona:** Telecom functional architect, specialist in TM Forum Open APIs, eTOM v2, ODA (Open Digital Architecture), and SID/Information Framework models.

**Mission:** Review the artifact from the perspective of TM Forum standards compliance:
- Is the eTOM mapping in the header correct and complete? (verify ALL covered L2 processes, not just the primary one — Order Handling=1.3.3, SC&A=1.4.5, Bill/Invoice=1.1.1.x)
- Are the TMF APIs cited correct for each operation? (TMF676=Payment capture, NOT rating; TMF678=Bill Management with calculation; TMF641=Service Order emitted by DRO)
- Are the TMF622 states cited canonical? Or custom without declaration?
- Is the `orderItem.action` cycle modeled correctly (add/delete/modify)?
- Are DRO internal sub-states clearly distinguished from canonical TMF641 states?
- Is the emitted TMF641 Service Order lifecycle correctly reflected?
- ODA principles respected: vendor-neutral, event-driven, layer separation?

**Expected output:** list of findings in the same format above.

---

#### Agent 3 — TOGAF Strategic Architect / Governance

**Persona:** Strategic architect, specialist in TOGAF ADM (4 domains: Business/Data/Application/Technology), Architecture Repository, requirements traceability, and ADR governance.

**Mission:** Review the artifact from the perspective of governance and architectural completeness:
- Non-trivial decisions assumed without an ADR? (SOUL #7 and #8). SPECIAL ATTENTION: if the flow defines PONR for bajas (MACD-DELETE), verify a dedicated ADR exists — ADR-013 §9·6 and ADR-014 §9·3 declare bajas as an open point.
- ADR references present in the notes? (traceability)
- Contradictions with existing ADRs?
- Coverage of all 4 TOGAF domains in the artifact?
- Vocabulary consistent with the canon? Ambiguous or semantically overloaded terms?
- Security, data retention, or LGPD compliance concerns missing? (especially in flows that close accounts or retire assets)
- Cross-flow conflicts with T2C.md, R2C.md, or other artifacts?

**Expected output:** list of findings in the same format above.

---

### Step 3 — CTO Cross Validation

After receiving the 3 agents' outputs, dispatch a **4th agent** with all consolidated findings.

**Persona:** CTO / Cross Architect — systemic view, no domain bias. Role: arbiter and synthesizer.

**Mission:**
1. For each finding from the 3 agents: confirm, moderate, or refute — with justification based on the reference documents (not opinion)
2. Escalate severity if 2+ agents converge on the same problem
3. Downgrade severity if the finding conflicts with what the ADRs establish (e.g. agent confuses activation with PONR — refute citing ADR-013)
4. Identify complementary cross-agent findings and consolidate them
5. Produce final verdict: **GO**, **GO-CONDITIONAL**, or **NO-GO** with blockers listed

**Expected output:**
- Consolidated table: `# | Section | Original Text | SF Arch | TMF Arch | TOGAF Arch | CTO Verdict | Final Sev | Suggested Action | Owner`
- Overall verdict with justification

---

### Step 4 — Produce final output

With the consolidated table from Agent 4:

1. **Display the summary** to the user: verdict, critical blockers, strengths
2. **Generate an Excel file** (`.xlsx`) using openpyxl in the same folder as the reviewed artifact:
   - Single sheet named after the artifact (e.g. `O2P-Order-Cancellation`)
   - Row 1: title with artifact name and verdict
   - Row 2: path of the reviewed file
   - Row 4: header — `# | Section / Line | Original Text | 🔧 SF Arch | 📡 TMF Arch | 🏛️ TOGAF Arch | ⚖️ CTO Verdict | Sev | Suggested Action | Owner | ✍️ [Architect] Review`
   - Following rows: one row per finding
   - `✍️ Review` column always empty — for the human architect to fill in
   - `Sev` column with background color: 🔴 = light red, 🟡 = light yellow, 🟢 = light green
   - Text columns with wrap and adequate width
   - File name: `Review-<md-basename>.xlsx`
3. Inform the user of the generated file path and open it if possible

## Quick glossary

| Term | Meaning |
|------|---------|
| PONR | Point of No Return = work order closure (not service activation) |
| DRO | Dynamic Revenue Orchestrator — orchestrates O2P, emits ONE TMF641 |
| SOM | Service Order Management — executes outside the core |
| SVA | Value-Added Service = add-on (e.g. extra TV, antivirus) |
| ONT/CPE | Optical Network Terminal / Customer Premises Equipment |
| ETF | Early Termination Fee |
| TMF622 | Product Order Management API |
| TMF641 | Service Order Management API — emitted by DRO |
| TMF639/652 | Resource Inventory / Resource Order — external, SOM-internal |
| TMF640 | Service Activation API |
| TMF637/638 | Product Inventory / Service Inventory (assetization) |
| TMF666/678 | Account Management / Customer Bill Management (billing) |
| TMF676 | Payment Management — payment capture and refund |
| TMF688 | Event Management — async notifications |
| TMF646 | Appointment Management — field visit scheduling |
| eTOM 1.3.3 | Order Handling (primary L2 process) |
| eTOM 1.4.5 | Service Config & Activation (delegated for deactivate/recover only) |
| Cancel-Replace | Comms Cloud OM model where a new order supersedes the original |
| FulfillmentRequest | Salesforce object that materializes TMF641 (Service Order) |
