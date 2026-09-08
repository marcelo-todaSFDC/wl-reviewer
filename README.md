# wl-reviewer

> Architectural review of O2P flow artifacts using 4 specialist agents in parallel.

A [Claude Code](https://claude.ai/code) skill that automates the review of O2P (Order-to-Payment) business flow `.md` files — structured, repeatable, and multi-perspective.

---

## What it does

Given an O2P flow artifact, the skill dispatches **3 specialist agents in parallel** followed by a **CTO arbiter** that consolidates and verdicts:

| Agent | Perspective |
|-------|-------------|
| 🔧 Salesforce Technical Architect | Comms Cloud implementability — DRO, SOM, SF Objects, testability |
| 📡 TM Forum Functional Architect | TMF Open API compliance, eTOM v2, ODA, canonical states |
| 🏛️ TOGAF Strategic Architect | ADR governance, traceability, vocabulary, compliance concerns |
| ⚖️ CTO Validation | Cross arbiter — confirms, moderates, or refutes each finding against reference documents |

**Output:** an Excel file with the original text, each architect's observation, CTO verdict, severity (🔴/🟡/🟢), suggested action — and an empty column for the 5th human architect to fill in.

**Final verdict:** `GO` · `GO-CONDITIONAL` · `NO-GO`

---

## Requirements

- [Claude Code](https://claude.ai/code) with access to the ADP project
- ADP repo cloned with reference ADRs under `architecture/decisions/`
- Flow artifact `.md` in `flows-wiki` format with frontmatter, Mermaid diagram, Steps table, and Notes

---

## Installation

Copy the skill file to the `.claude/skills/` folder of your ADP project:

```bash
cp review-o2p-flow.md <your-project>/.claude/skills/
```

Claude Code will discover it automatically on the next session.

---

## Usage

Inside Claude Code, from your ADP project:

```
/review-o2p-flow architecture/flows-wiki/o2p/O2P-Fulfillment.md
```

---

## What the skill reads before reviewing

The skill automatically loads the following reference documents from your project before dispatching the agents:

- `SOUL.md` — architectural non-negotiables (#7: never silently override an ADR; #8: record non-obvious decisions)
- `architecture/flows-wiki/o2p/O2P.md` — B2C white-label framework canon
- Relevant project ADRs (013, 014, 015, telco-009)

> **Adapt the ADR paths** in the skill if your project uses different numbering.

---

## Quick glossary

| Term | Meaning |
|------|---------|
| PONR | Point of No Return = work order closure (not service activation) |
| DRO | Dynamic Revenue Orchestrator — orchestrates O2P, emits ONE TMF641 |
| SOM | Service Order Management — executes outside the core |
| SVA | Value-Added Service = add-on |
| ETF | Early Termination Fee |
| FulfillmentRequest | Salesforce object that materializes TMF641 (Service Order) |
| TMF622/641/640 | Product Order / Service Order / Activation APIs |
| TMF637/638 | Product/Service Inventory (assetization) |
| TMF666/678 | Account/Bill Management (billing) |
| TMF676 | Payment Management |
| TMF688 | Event Management |
| TMF646 | Appointment Management |

---

## License

MIT — use, adapt, and share.

---

*Built as part of the [ADP — Agentic Delivery Platform](https://github.com/p-amers-mx-totalplay-ari/adp) method for Salesforce/TMF architectural flow reviews.*
