# RevenueOS V2: AI Prospecting & Sales Intelligence

## System Architecture

**Prospect Intake → Salesforce Account & Contact Deduplication → Clay Enrichment → Official Website Research → AI Business Model Classification → ICP Scoring → AI Account Research → Quality Gate → Salesforce Update → Task Deduplication → Sales Follow-Up**


## The Business Problem

At **Lolia**, a fictional B2B AI-powered Revenue Intelligence SaaS company, preparing an account for sales engagement involved several disconnected steps: checking CRM records, researching the business, enriching company data, assessing customer fit, and preparing a relevant sales approach.

This created three operational challenges:

1. **Fragmented prospect data:** Company, contact, technology, and growth information needed to be gathered from multiple sources.
2. **Inconsistent qualification:** Sales prioritization depended on manual research and interpretation rather than a repeatable Ideal Customer Profile (ICP) model.
3. **Repetitive CRM administration:** Duplicate records, incomplete account intelligence, and follow-up tasks increased the operational workload.

The underlying problem was the absence of a consistent process for turning a new prospect into an enriched, qualified, and actionable Salesforce account.

---

## The Solution

**RevenueOS V2** connects prospecting, enrichment, research, qualification, and CRM execution in one automated GTM workflow.

The system:

1. Normalizes prospect information and checks Salesforce for an existing Account and Contact.
2. Reuses existing records or creates new ones when needed.
3. Sends the prospect to Clay for company, technology, and growth enrichment.
4. Retrieves official website evidence to establish the company's primary business model.
5. Uses DeepSeek to produce a structured, evidence-based business classification.
6. Calculates a **deterministic ICP score** and assigns a fit tier.
7. Generates account research, potential sales challenges, an outreach angle, and contact personalization.
8. Validates data completeness and classification evidence before writing results to Salesforce.
9. Creates a sales task for qualified prospects only when a matching task does not already exist.

### Workflow

**Prospect Intake → Salesforce Deduplication → Clay Enrichment → Website Evidence → AI Classification → ICP Scoring → Account Research → Validation → Salesforce Update → Sales Follow-Up**

## Business Impact

RevenueOS V2 is designed to improve prospecting and sales operations by:

- Reducing repetitive account research and manual CRM updates.
- Giving sales representatives a consistent view of account fit and supporting intelligence.
- Prioritizing high-fit prospects for timely follow-up.
- Preventing duplicate Account, Contact, and follow-up Task creation.
- Keeping incomplete or unverified enrichment from automatically updating Salesforce.
- Connecting prospect research directly to the CRM where sales teams work.

The core outcome is a more consistent progression from **raw prospect → researched account → qualified sales action**.

---

## System Architecture & Logic

RevenueOS V2 is orchestrated in **n8n**, with **Salesforce** as the CRM, **Clay** as the enrichment layer, **Tavily** for official website research, and **DeepSeek** for classification and account research. JavaScript controls scoring, validation, and operational decisions.

The architecture separates AI interpretation from the business rules that determine scores, CRM writeback, and task creation.

### 1. Prospect Intake & CRM Deduplication

The workflow normalizes incoming company and contact details before querying Salesforce.

- **Account lookup:** Matches existing Accounts using the company domain.
- **Contact lookup:** Matches existing Contacts using the email address.
- **Record creation:** Creates a Salesforce Account or Contact only when no matching record is found.
- **Record continuity:** Carries Salesforce record IDs into the enrichment process, allowing results to be written back to the original records.

This reduces unnecessary record duplication and maintains a consistent account-to-contact relationship.

### 2. Company Enrichment & Business Classification

Clay combines company information from enrichment providers:

- **Datagma:** Company description.
- **HG Insights:** Revenue, employee count, industry information, and technology usage.
- **Crunchbase:** Growth score, 90-day change, and growth direction.

The enriched record is returned to an authenticated n8n webhook.

Tavily searches the company's official website. A JavaScript node selects relevant company and product evidence while filtering out careers and recruitment pages.

DeepSeek classifies the company's primary business model using a structured output:

- `B2B_SAAS`
- `B2B_SOFTWARE`
- `MIXED_TECH`
- `NOT_ICP`
- `UNKNOWN`

The classification includes supporting evidence, reasoning, confidence, and a review indicator. The original provider industry label is retained separately from the final business-model classification.

### 3. Deterministic ICP Scoring

A JavaScript scoring engine evaluates five dimensions of Lolia's Ideal Customer Profile.

| ICP Dimension | Maximum Points | Evaluation |
| --- | ---: | --- |
| Business model | 25 | B2B SaaS, B2B software, or diversified technology |
| Revenue | 25 | Higher scores for revenue above $10 million |
| Employee count | 20 | Strongest fit at 50–2,000 employees |
| Growth | 15 | Growth score, 90-day movement, and growth direction |
| CRM technology | 15 | Verified Salesforce or HubSpot product signals |
| **Total** | **100** | |

**Fit tiers:**

- **High:** 80–100
- **Medium:** 60–79
- **Low:** 0–59

The score is calculated from explicit rules rather than assigned by the AI model.

A separate validation step checks required data and supporting industry evidence. Incomplete or unverified results receive a **Provisional** status and are held before CRM writeback. Only **Complete**, verified records proceed to the Salesforce update stage.

### 4. AI Account Research & Salesforce Writeback

DeepSeek generates structured account intelligence from the enriched company data:

- Company summary
- Potential revenue-operations challenges, identified as hypotheses
- Recommended sales approach
- Personalized outreach opening
- Information requiring further verification

For validated records, Salesforce is updated with:

- **Account:** ICP score, fit tier, and account research report
- **Contact:** Personalized outreach text

The AI research layer provides context for sales outreach without changing the deterministic ICP score.

### 5. Sales Task Creation & Deduplication

A Salesforce follow-up task is considered when the account has a verified, Complete score of at least **60** and belongs to a relevant business-model category.

- **High-fit accounts (80–100):** Sales Follow-Up
- **Medium-fit accounts (60–79):** Research & Qualify

Before creating a task, the workflow searches Salesforce for a matching task associated with the same Account and Contact. Existing tasks are skipped. New tasks are assigned to the Account owner.

Low-fit accounts do not trigger automatic sales tasks.

### 6. Error Handling & Monitoring

A separate n8n error-handling workflow receives failed executions, captures the workflow name, failed node, error message, and execution reference, and sends an email notification through SMTP.

This provides visibility into automation failures without relying solely on manual execution checks.

### Architecture Principles

RevenueOS V2 is built around four principles:

1. **Resolve records before enrichment** — preserve CRM identity and limit duplicate creation.
2. **Ground AI classifications in evidence** — use official company information and structured outputs.
3. **Keep scoring and CRM actions deterministic** — apply explicit rules to qualification, writeback, and task creation.
4. **Validate before execution** — prevent incomplete intelligence and duplicate tasks from generating unnecessary sales activity.

---

## Why This Project Matters

RevenueOS V2 extends the GTM process beyond basic lead qualification by connecting **prospecting data, account intelligence, AI research, and sales execution**.

It demonstrates how GTM engineering can reduce operational friction between data enrichment and CRM action while keeping scoring decisions transparent and follow-up workflows consistent.

The result is a system designed to give sales representatives more useful account context and fewer administrative steps before engagement.

---

## Project Status

**Status: Implemented and tested end-to-end**

Validation covered:

- Existing and newly created Salesforce Account and Contact paths
- Business-model classification and ICP scoring
- Repeated scoring consistency checks
- Holding Provisional records before CRM writeback
- Successful Salesforce Account and Contact updates
- Task creation and duplicate-task prevention
- Failure notifications from the Error Handler

## Workflow Exports

| Workflow | Responsibility |
| --- | --- |
| [Prospecting Engine](workflows/01-prospecting-engine.json) | Prospect intake, normalization, Salesforce Account/Contact matching, and Clay handoff |
| [Enrichment & Sales Intelligence](workflows/02-enrichment-sales-intelligence.json) | Enrichment receiver, business classification, ICP scoring, account research, CRM writeback, and sales tasks |
| [Error Handler](workflows/03-error-handler.json) | Execution failure alerts |

*Published workflow exports exclude live credentials and private integration endpoints.*

---

## Next Version

RevenueOS V3 will extend the platform into **Quote-to-Cash automation**, connecting the qualified sales process to quoting, approvals, and downstream revenue operations.
