# Initial RAID Log Generator

An Agent Flow that transforms project deliverables and requirements documents into draft Statements of Work (SOWs) with integrated RAID logs, tailored for UK public sector digital projects.

> **This flow produces initial AI-generated drafts for human review — not final commercial commitments.**

---

## Overview

Given one or more requirements documents, the flow produces a professional, formatted SOW per document. Each SOW includes a delivery approach, team shape, phased timeline, indicative costs (calculated from a rate card), and a RAID log pre-populated using configurable rules.

Designed for delivery consultants, bid teams, and project managers who need to accelerate the initial scoping phase while maintaining clear human review gates.

---

## Flow Architecture

The pipeline consists of 9 nodes:

| # | Node Type | Description |
|---|-----------|-------------|
| 1 | **Input** | Reads deliverables/requirements `.docx` files from `data/` |
| 2 | **Reference** | Loads `raid-log-pre-fill-rules.docx` — rules for RAID item classification |
| 3 | **Reference** | Loads `digital-team-rate-card.xlsx` — role day rates for costing |
| 4 | **LLM** | Generates structured SOW JSON as a Senior Delivery Consultant |
| 5 | **Code** | Calculates indicative costs by looking up rates from the rate card |
| 6 | **Code** | Extracts the SOW array to enable fan-out |
| 7 | **Fan-Out** | Splits the array so each SOW is processed independently |
| 8 | **Code** | Formats each SOW as Markdown (tables, sections, GBP currency) |
| 9 | **Output** | Renders each SOW as a `.docx` Word document |

---

## Prerequisites

- Agent Flow platform (Impacting / agentflow)
- All required data files present in the `data/` directory (see Inputs below)

---

## Inputs

| Input | Path | Format | Purpose |
|-------|------|--------|---------|
| Requirements documents | `data/Deliverables and requirements docs/*.docx` | Word (.docx) | Source project requirements that drive SOW generation |
| RAID pre-fill rules | `data/raid-log-pre-fill-rules.docx` | Word (.docx) | Structured rules for identifying and classifying RAID items |
| Digital team rate card | `data/digital-team-rate-card.xlsx` | Excel (.xlsx) | Role + seniority day rates used to calculate indicative costs |

### Sample Data

Ten sample requirements documents are included for demo and testing:

| File | Project |
|------|---------|
| `01-cloud-migration-azure.docx` | Cloud Migration to Azure |
| `02-ai-chatbot-virtual-assistant.docx` | AI Chatbot / Virtual Assistant |
| `03-data-mesh-platform.docx` | Data Mesh Platform |
| `04-mobile-app-personal-tax.docx` | Mobile App — Personal Tax |
| `05-open-data-api-platform.docx` | Open Data API Platform |
| `06-siem-implementation.docx` | SIEM Implementation |
| `07-rpa-accounts-payable.docx` | RPA — Accounts Payable |
| `08-govuk-marriage-allowance.docx` | GOV.UK Marriage Allowance |
| `09-salesforce-crm-compliance.docx` | Salesforce CRM Compliance |
| `10-mlops-risk-scoring.docx` | MLOps Risk Scoring |

---

## Outputs

One `.docx` Statement of Work is produced per input requirements document. Each SOW contains:

- **Executive Summary** — high-level project overview
- **Requirements Understanding** — bulleted interpretation of the source document
- **Delivery Approach** — phased delivery with descriptions and outputs per phase
- **Team Shape** — table of roles, seniority, allocation (days/week), duration, cost, and responsibilities
- **Timeline** — phased table with week ranges, key activities, outcomes, and checkpoints
- **Indicative Costs** — total cost, monthly breakdown, and blended team day rate in GBP
- **RAID Log** — pre-populated risks, assumptions, issues, and dependencies
- **Human Review Checklist** — assumptions to validate, open questions, and decisions required

---

## RAID Log

RAID items are generated using the rules in `raid-log-pre-fill-rules.docx`. Each item follows a consistent structure:

| Field | Description |
|-------|-------------|
| ID | Sequential per category per SOW — `R001`, `A001`, `I001`, `D001`... |
| Category | Risk / Assumption / Issue / Dependency |
| Title | Short label |
| Severity | High / Medium / Low |
| Description | Detail of the item |
| Rationale | Which rule triggered this item |
| Suggested Owner | Responsible role |

---

## Cost Calculation

Costs are calculated deterministically from the rate card using each team member's **role** and **seniority band**:

```
line cost = quantity × days_per_week × duration_weeks × mid_day_rate
```

The output includes:
- **Total cost** across all team members
- **Indicative monthly cost**
- **Blended team day rate**
- **Pricing warnings** for any role/seniority combinations not found in the rate card

**Excluded from costs:** VAT, travel and expenses, software licences, hosting, infrastructure, contingency.

---

## Supported Roles & Seniority

**Roles (15):**
Delivery Manager, Product Manager, User Researcher, UX/Interaction Designer, Content Designer, Software Engineer, DevOps/Platform Engineer, Data Engineer, Data Scientist, BI/Data Analyst, Solution Architect, Cyber Security Architect, Penetration Tester, QA/Test Engineer, Business Analyst

**Seniority bands (4):**
Junior, Mid, Senior, Lead

---

## Human Review

Every generated SOW is explicitly marked as an **initial AI-generated draft**. The Human Review section of each document includes:

- **Assumptions to validate** — items that need stakeholder confirmation
- **Open questions** — unresolved points requiring client input
- **Decisions required** — commercial or delivery choices not yet made

These are not omissions — they are intentional prompts to guide the review conversation before any commercial commitment is made.

---

## Configuration

The flow definition is in [`initial-raid-log-generator.json`](initial-raid-log-generator.json). This file contains all node definitions, edges, the LLM prompt, and the JavaScript code for cost calculation and formatting.
