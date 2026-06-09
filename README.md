# Operations Weekly Summary Agent

An AI-powered multi-agent system that automatically generates a structured weekly operations summary report from a source document, and publishes it as a formatted Word file to a shared document library.

---

## What It Does

Each week, a source Word document is compiled containing raw operational updates from multiple sites. This system reads that document automatically, applies a standardized structure, generates a formatted three-section summary report, and delivers it to the right people — without manual effort.

**Input:** A source Word document deposited into a monitored folder.
**Output:** A formatted Word document saved to a shared output folder, with an email notification sent to the report recipient.

---

## Report Structure

The generated report has three sections:

| Section | Content |
|---|---|
| Section 1 — Executive Summary | Two priority highlights per site, all sites listed alphabetically |
| Section 2 — Site Summaries | Full summary per site with six standard categories each |
| Section 3 — Appendix | Full reproduction of the source content, organized by site |

Each site summary covers six standard categories in a fixed order: Category-1 through Category-6.

---

## Architecture Overview

The system uses a **multi-agent architecture** in Copilot Studio orchestrated by a **Power Automate flow**.

```
Source Document deposited to SharePoint folder
        ↓
[Power Automate Flow triggered]
        ↓
[Document Reader Agent]
Reads source document, extracts site list,
collects hyperlinks, finds report date
Returns: SiteList, HyperlinkInventory, REPORT_DATE
        ↓
[Loop: Site Summarizer Agent × N sites]
Called once per site, generates full
six-category summary for that site
Returns: SiteSummary (appended to SITE_SUMMARIES)
        ↓
[Executive Summary Writer Agent]
Reads assembled SITE_SUMMARIES, generates
two priority highlights per site
Returns: EXECUTIVE_SUMMARY
        ↓
[Appendix Generator Agent]
Reads source content per site, reproduces
it exactly with page-break structure
Returns: APPENDIX_CONTENT
        ↓
[Power Automate: Populate Word Template]
Maps four variables to Content Controls
        ↓
[Power Automate: Save to SharePoint]
Creates timestamped .docx in output folder
        ↓
[Power Automate: Email Notification]
Sends SharePoint link to report recipient
```

---

## Components

| Component | Type | Role |
|---|---|---|
| Operations Weekly Summary Flow | Power Automate flow | Orchestrates the entire process |
| Document Reader Agent | Copilot Studio agent | Reads source, extracts structured metadata |
| Site Summarizer Agent | Copilot Studio agent | Generates per-site summary (called N times) |
| Executive Summary Writer Agent | Copilot Studio agent | Generates Section 1 from assembled summaries |
| Appendix Generator Agent | Copilot Studio agent | Reproduces source content for Section 3 |
| Word Template | .docx file | Four Content Controls mapped by the flow |

---

## Repository Structure

```
├── README.md
├── CHANGELOG.md
├── docs/
│   ├── architecture/
│   │   ├── overview.md              # Full architecture with design rationale
│   │   ├── design-decisions.md      # Three proposals evaluated; why this one was chosen
│   │   └── lessons-learned.md       # What failed, what worked, and why
│   ├── agents/
│   │   ├── orchestrator.md
│   │   ├── document-reader.md
│   │   ├── site-summarizer.md
│   │   ├── executive-summary-writer.md
│   │   └── appendix-generator.md
│   ├── flow/
│   │   ├── flow-overview.md         # Step-by-step with inputs/outputs
│   │   ├── trigger-setup.md         # SharePoint trigger configuration
│   │   └── troubleshooting.md       # Known issues and fixes
│   └── template/
│       └── word-template-setup.md   # Content Controls setup guide
├── templates/
│   └── Operations_Weekly_Summary_Template.docx
└── test/
    ├── Test_AG_File.docx
    └── test-agent-instructions.md
```

---

## Prerequisites

- Microsoft 365 Copilot license
- Copilot Studio access (included with M365 Copilot)
- Power Automate (standard license)
- SharePoint Online
- Word desktop app (required for inserting Content Controls into the template)

---

## Quick Start

1. Create the four child agents in Copilot Studio using the instructions in `docs/agents/`
2. Upload the Word template (`templates/`) to a SharePoint template library after adding Content Controls (see `docs/template/word-template-setup.md`)
3. Build the Power Automate flow using `docs/flow/flow-overview.md`
4. Configure the SharePoint trigger to watch your source document folder
5. Test by depositing a source document into the trigger folder

Full step-by-step instructions are in `docs/`.

---

## Key Design Principles

**Why multi-agent:** A single agent cannot reliably store large generated content in named variables and pass it to a flow. Variable storage is real only inside Power Automate — not inside Copilot Studio's generative orchestrator. The multi-agent structure ensures each piece of content is a real Power Automate variable before being used downstream.

**Why the flow orchestrates:** Copilot Studio topics cannot natively loop over a dynamic list and collect outputs from each iteration. Power Automate's `Apply to each` action handles this reliably. The flow is the right place for looping logic; agents are the right place for content generation logic.

**Why no topic-based routing:** With generative orchestration enabled, the Copilot Studio orchestrator bypasses topics and handles requests directly. Topics are useful for multi-intent agents; for a single-purpose agent, the orchestrator approach is simpler and more reliable.

See `docs/architecture/design-decisions.md` for the full evaluation.

---

## Version

Current: v1.0 — Multi-agent flow architecture, no audit agent.
Planned: v1.1 — Audit agent added between Appendix Generator and Word template population.

See `CHANGELOG.md` for full history.
