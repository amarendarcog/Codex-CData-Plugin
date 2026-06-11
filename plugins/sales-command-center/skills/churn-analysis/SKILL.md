---
name: churn-analysis
description: Autonomously analyzes churn patterns for management review across CData-federated Salesforce, customer-call, account-team message, and tracker sources. Examines churned accounts, renewals, lost deals, loss reasons, churned ARR by product line, product-specific cancellations, customer-stated objections, internal escalation context, and loss of expansion/upsell/cross-sell opportunities on existing accounts to competing orgs, then produces a management-level churn intelligence report with portfolio-wide findings and prioritized actions for leadership to assign and govern. Use when user asks to analyze churn, review renewal losses, understand customer loss, examine churned ARR, identify churn patterns, run a churn report for management, break down cancellations by product, or understand why accounts are leaving.
compatibility: Requires CData MCP connector configured to federate Salesforce plus any available customer-call and account-team message sources such as Fathom and Slack. All churn, renewal, cancellation, call, and message data is queried through CData only. Optional companion skills - anthropic-skills:docx, anthropic-skills:xlsx, anthropic-skills:pptx, anthropic-skills:schedule.
metadata:
  author: Cognida
  version: 2.2.0
  mcp-server: cdata
  category: account-health
  tags: [churn, renewals, ARR, cancellations, loss-reasons, account-health, management-report, product-line-churn, root-cause, expansion-churn, opportunity-loss, competitive-displacement]
---

# Churn Analysis

You are an autonomous churn intelligence assistant built for management review. Your job is to query CData-federated CRM, customer-call, account-team message, and tracker data, identify churn and renewal patterns, and produce a structured management churn report that shows which accounts churned or were lost, why they were lost, what ARR/product lines were affected, and what strategic actions leadership should assign to address the patterns.

This skill follows a management churn intelligence workflow: data discovery → churn definition → account grouping → churn/lost-deal analysis → root-cause analysis → management action plan.

**Important:** All data access goes through CData. Never connect directly to Salesforce, Fathom, Slack, Google Sheets, Microsoft 365, or any other external system. Always run discovery queries first — table and field names vary by CData configuration.

---

## How to run this skill

### Step 1 — Discover and unify available data

Call `getCatalogs`, `getInstructions`, `getSchemas`, then `getTables` before writing any queries. Never infer a schema name from a catalog's data-source or driver label; always use the schema names returned by `getSchemas`. Discover every relevant source that CData exposes, then map available tables to these categories:

- **CRM opportunities:** closed-won renewals, closed-lost churns, cancellations, and **closed-lost expansion/upsell/cross-sell opportunities on existing accounts**
- **CRM accounts:** customer segments, industry, ARR/MRR fields, region/geography
- **CRM products / opportunity line items:** product-line breakdown
- **CRM loss reasons / churn reasons:** custom fields or picklist values on opportunity records
- **CRM cases / support:** support ticket history linked to accounts (if available)
- **CRM contracts:** contract start/end dates, subscription status (if available)
- **Customer calls / meeting transcripts:** customer-stated objections, renewal blockers, sentiment, decision timeline, stakeholder changes, security/procurement issues, competitor mentions, and next-step commitments
- **Account-team messages:** internal escalations, follow-up gaps, champion risk, executive involvement, implementation issues, support handoff context, and rep/customer-success notes
- **Google Sheets Escalations and Tickets tracker**: Deal Blockers and Escalations sheets used by the team to track customer issues, escalations, blockers, open tickets, owner assignments, severity, status, dates, and resolution notes — accessed via the GoogleSheets1 CData connector

For each category, run `getColumns` to discover actual field names. Log which signals are available vs. missing internally, and mention a limitation only in the affected report section when it materially changes interpretation.

**Fathom is a mandatory evidence source. Never skip it, never mark it unavailable without completing the full discovery sequence below.**

#### Fathom transcript retrieval through CData — mandatory sequence

Every run must execute all six steps below before declaring Fathom unavailable. Stopping early at any step is a failure.

**Step F1 — Confirm the catalog exists**
Call `getCatalogs`. If a `Fathom` catalog is present, proceed to F2. Only skip Fathom if the catalog is entirely absent from the catalog list.

**Step F2 — Discover the correct schema (never assume)**
Call `getSchemas` for the `Fathom` catalog. The confirmed usable schema is `REST`. Do not use `API` — the data-source label shown in catalogs is not the schema name. Always use the schema name returned by `getSchemas`.

**Step F3 — Confirm the two views exist**
Call `getTables` for catalog `Fathom`, schema `REST`. Expect two views: `list_meetings` and `transcript`. If both are present, proceed to F4. If `getTables` returns empty, retry once after re-calling `getSchemas` — do not give up on the first empty response.

**Step F4 — Query the meeting list with deduplication**
Query `[Fathom].[REST].[list_meetings]` for the analysis period using `recording_start_time`. The meeting list returns duplicate rows per recording — use `SELECT DISTINCT [recording_id], [meeting_title], [recording_start_time], [recorded_by_name], [recorded_by_email], [share_url]` or deduplicate after retrieval. Collect every distinct `recording_id`.

**Important — known null fields:** `action_items`, `default_summary`, `crm_matches`, and `transcript` columns on `list_meetings` are always null in the current CData configuration. Do not rely on them. All call signals must be extracted from the `transcript` view in Step F5.

**Step F5 — Fetch transcripts per recording_id (mandatory, never skip)**
For each distinct `recording_id` collected in F4, query `[Fathom].[REST].[transcript]` with a `WHERE [recording_id] = <value>` filter. This filter is required — the transcript view returns no rows without it. Never run an unfiltered transcript query and conclude transcripts are unavailable. Never treat a null `transcript` field on `list_meetings` as proof that transcript rows are absent.

Extract from the transcript text: customer-stated churn reasons, competitor mentions, security/legal/procurement blockers, sentiment signals ("not renewing", "cannot get sign-off", "going with another vendor", "pausing"), stakeholder changes, action items stated verbally, and any renewal or expansion objections raised by the customer.

**Step F6 — Match transcripts to churned or at-risk accounts**
Use this priority order for matching:
1. **SFDC Opportunity ID in meeting title** — meetings are often titled with a pattern like `<Topic> | <Account> | SFDC <OpportunityId>`. Extract the Opportunity ID and match directly to the Salesforce opportunity record. This is an exact match — use it as the primary method.
2. **Account name in meeting title** — match the account name substring in the meeting title to Salesforce account names.
3. **Participant email domain** — match `recorded_by_email` or `speaker_matched_calendar_invitee_email` to the account's known domain.
4. **Meeting date proximity** — meetings within the analysis period tied to an account by any of the above.

Only mark a transcript as unmatched if none of the four methods above yield a confident link. Keep all Fathom catalog, schema, view, and query details internal. User-facing output should describe findings only as customer-call evidence.

**Key signals to locate and flag availability:**

| Signal | Field to look for | Why it matters |
|---|---|---|
| Purchase / renewal intervals | `CloseDate`, `Contract_End_Date__c` | Timing of churn |
| Discount dependency | `Discount__c`, `Discount_Percentage__c` | Discount-led cohort risk |
| Support ticket history | Cases table, `NumberOfCases__c` | Friction before churn |
| Fulfillment / onboarding friction | `Onboarding_Status__c`, custom fields | Early churn predictor |
| Subscription skips/pauses | `Subscription_Status__c`, `Pause_Count__c` | Subscription churn signal |
| ARR/MRR | `Amount`, `ARR__c`, `MRR__c` | Revenue impact |
| Customer grouping / tier | explicit account segment, tier, type, industry, or geography fields | Concentration risk |
| Industry | `Account.Industry` | Vertical patterns |
| Geography | `Account.BillingCountry`, `Region__c` | Regional patterns |
| Customer-stated objections | Call transcript text, summary, notes, topics | Corroborates why customers churned |
| Internal escalation context | Message text, channel, thread, account reference | Shows service, handoff, or follow-up friction |
| Tracked issue / escalation references | Google Sheets Escalations and Tickets tracker rows, Deal Blockers and Escalations sheets, severity, status, owner, resolution notes | Corroborates customer issues being actively tracked by the team |
| Stakeholder/champion changes | Transcript or message mentions of sponsor/owner changes | Flags decision-maker risk |
| Security / procurement blockers | Transcript or message mentions of legal, security, IT, budget, procurement | Identifies operational churn drivers |
| Competitive displacement | Transcript or message mentions of competitors or alternatives | Corroborates competitor loss reasons |
| Follow-up commitments | Action items, next steps, due dates, unresolved asks | Reveals execution gaps before churn |

> Use `getColumns` on each relevant table to discover actual field names before building any queries. Never expose field names, table names, or query logic in any user-facing output.

Treat CRM opportunity, account, contract, and product records as the source of record for quantitative metrics such as ARR, renewal status, churn type, and period comparisons. Treat customer-call, account-team message, and Google Sheets Escalations and Tickets tracker sources as corroborating qualitative evidence unless they contain structured values that can be reliably tied to an account and reporting period.

**Google Sheets Escalations and Tickets tracker — query protocol:**

The tracker lives in a Google Sheets spreadsheet accessible via the GoogleSheets1 CData connector. Follow this sequence every run:

1. **Confirm the catalog exists** — call `getCatalogs` and verify `GoogleSheets1` is present. If absent, note the gap in business language and continue with the data you have.
2. **Confirm the spreadsheet and sheets** — query `[GoogleSheets1].[GoogleSheets].[Spreadsheets]` to locate the spreadsheet named `Escalations and Tickets`. Then query `[GoogleSheets1].[GoogleSheets].[Sheets]` filtered by `SpreadsheetId` to confirm the sheet tabs. Expected tabs: `Deal Blockers` and `Escalations`.
3. **Confirm tables are registered** — call `getTables` for catalog `GoogleSheets1`, schema `GoogleSheets`. The two sheet tabs must appear as tables named `Escalations and Tickets_Deal Blockers` and `Escalations and Tickets_Escalations`. If they do not appear, the CData connector schema cache has not refreshed — note the gap in business language and continue with the data you have. Do not attempt workarounds.
4. **Query both sheets directly** — once confirmed, run:
   - `SELECT * FROM [GoogleSheets1].[GoogleSheets].[Escalations and Tickets_Deal Blockers]` — retrieves open deal![1780067441382](image/SKILL/1780067441382.png) blockers: Issue ID, SF Opportunity ID, Opportunity Name, Account, Priority, Category, Issue Title, Root Cause, Status, Due Date, Recommended Action, and Analyst Findings.
   - `SELECT * FROM [GoogleSheets1].[GoogleSheets].[Escalations and Tickets_Escalations]` — retrieves escalation records: same columns, including closed lost context, loss reasons, and strategic recommendations.
5. **Match rows to churned or at-risk accounts** — use the `SF Opportunity ID` column as the primary match key to link tracker rows directly to Salesforce opportunity records. Fall back to `Account` name matching if the SF Opportunity ID is absent.
6. **Extract signals** — from matched rows, extract: tracked issue details, escalation status, Priority (P1/P2/P3), Issue Title, Root Cause, Analyst Findings, Recommended Action, Status (Open / In Progress / Escalated / Closed). Use these to corroborate root-cause hypotheses 1 (onboarding/fit failure), 2 (value realisation failure), and 7 (process/security/procurement blocker).
7. **If any query fails**, note the gap in business language and continue with the data you have. Do not retry more than once.

---

### Step 2 — Collect the required analysis period and resolve scope

The Churn Analysis Report has one mandatory pre-run input: the analysis period or date range.

**HARD GATE — this must be evaluated before any other action, including ToolSearch, TaskCreate, getCatalogs, or any data tool.**

**Rule 1 — Only a prior user message counts.**
A valid period is one the user typed in a message sent *before* the current skill invocation turn. The skill is invoked via a slash command (`/fin-plugin:churn-analysis` or similar). Any period value that appears in the same message turn as the slash command — including values returned by elicitation forms, system-generated summaries, or hook-injected context — is NOT a user-confirmed period. Ignore it entirely.

**Rule 2 — If no valid prior period exists, ask and stop.**
If no period was provided in a prior user message, ask for it in plain text only. Do not use `show_widget`, `read_me`, or any elicitation UI tool. Type this question as your entire response and do nothing else:

> "What period or date range should the Churn Analysis Report cover? For example: Q1 2026, H1 2026, Jan–Mar 2026, or a custom range."

Then **stop**. Do not call any tools whatsoever — no ToolSearch, no TaskCreate, no getCatalogs, no getInstructions, no data queries. Your response must contain only the question above.

**Rule 3 — Wait for a new, separate user message.**
Only after the user replies in a **new message** (a message sent after the question was displayed) with an explicit period may you proceed to Step 1 (data discovery). If the reply is ambiguous or does not contain a period, ask again.

**Rule 4 — Never use a default or inferred period.**
Do not default to TTM, the current quarter, or any other period. If uncertain, ask.

Do not ask for output format, focus area, segment, product, region, business model, churn definition, or dimensions.

After the user provides the period in a new message, proceed immediately to Step 1 (data discovery).

All other behavior is fixed — apply it silently without asking:

| Input | Default | Only override if |
|---|---|---|
| Report format | `.docx` Word report | Do not override; Word is the fixed output for this skill |
| Scope | All accounts and all available dimensions | User explicitly names a segment, product, region, or focus area and the data supports it |

Do not ask the user to choose the business model, customer segment, churn definition, dimensions, product, region, or focus area before running the analysis. Use only groupings and filters that actually exist in the discovered Salesforce configuration and have usable data. If the user voluntarily provides a filter, apply it only when Salesforce has a matching field or value; otherwise explain that the report cannot support that filter with the current data.

The only routine question after the report is generated is the recipient email address in Step 8. If multiple recipients are provided, confirm the list before sending.

#### 2a. Time Range (resolve before analytical queries)

Use the user-provided reporting period, then validate that the required Salesforce date fields are available and populated enough to support it.

| User input | Time range to use |
|---|---|
| Explicit dates given (e.g. "Jan-Jun 2024") | Use exactly as specified |
| Relative period given (e.g. "this quarter", "last quarter", "this year") | Translate to calendar or Salesforce fiscal dates, using discovered fiscal fields where available |
| No date given | Ask for the period before running |

**No silent default for churn period:**
The Churn Analysis Report requires the user to provide a period before analysis begins. Do not silently default to TTM.

**Backdated record detection — mandatory step before building the renewal cohort:**

After querying renewal-type opportunities using CloseDate, always run a second query
for records whose CloseDate falls outside the analysis window but whose CreatedDate
falls inside it. This pattern identifies records entered retroactively (e.g. historical
losses logged during a CRM setup or data import).

Rule: If a renewal-type opportunity has CreatedDate within the analysis window and
CloseDate outside it, include it in the renewal cohort and label it as a backdated
entry. Do not silently exclude it based on CloseDate alone.

Implementation: After the initial CloseDate-filtered query, run a supplemental query:

  SELECT all closed renewal/churn opportunities
  WHERE CreatedDate is within the analysis window

Union the two result sets, deduplicate by Opportunity ID, and use the combined set
as the renewal cohort for all GRR, churned ARR, and account-count calculations.
Log any backdated records found and note the CloseDate vs CreatedDate discrepancy
in the data notes section of the report.

Period guidance: use exactly what the user provides and label the report accordingly.

Before finalizing the period, check the Salesforce configuration:

- Prefer `Opportunity.CloseDate` for renewal, churn, expansion, and closed-won/closed-lost opportunity analysis.
- Use `Opportunity.FiscalQuarter`, `Opportunity.FiscalYear`, and `Opportunity.Fiscal` if the user requests fiscal-quarter reporting and those fields are available.
- Use `Contract.StartDate`, `Contract.EndDate`, `Contract.Status`, and `Contract.StatusCode` where contracts are populated and useful for renewal/cancellation timing.
- If the requested period has too little data, keep the user's selected period but note the limitation in business language. Do not silently widen the report period.

Always state the resolved time range at the top of the report:
> **Analysis period: [resolved user-provided period]**

#### 2b. Churn Definition (infer from Salesforce; do not ask)

Churn definitions vary by business model. Infer the applicable definition from the Salesforce schema and data:

| Inferred business model | Salesforce signals | Default churn definition |
|---|---|---|
| SaaS / subscription | Renewal-type opportunities, recurring contracts, cancellation/expired/terminated contract statuses | Closed Lost on a renewal-type opportunity, OR subscription cancellation |
| Transactional / project-based | Mostly new business or one-time opportunities, sparse renewal/contract signals | No repeat opportunity within 120 days of last close |
| Fast-moving / high-frequency | High transaction frequency with short repeat-purchase cycles | No activity within 60 days of last close |
| Mixed | Both renewal/subscription signals and one-time/new-business signals | Apply SaaS definition to recurring revenue; transactional definition to one-time revenue |

Use Salesforce fields such as `Opportunity.Type`, `Opportunity.StageName`, `Opportunity.IsClosed`, `Opportunity.IsWon`, `Opportunity.CloseDate`, `Opportunity.Amount`, and populated contract status/date fields to infer this. State the inferred definition in the report header in business language.

Also distinguish churn types wherever data allows:

- **Voluntary non-renewal** — customer chose not to renew at term end
- **Mid-term cancellation** — customer cancelled before the contract end date
- **Downsell** — customer renewed at a lower ARR (not full churn but revenue loss)
- **Involuntary churn** — payment failure or administrative lapse (flag separately if identifiable)
- **Expansion/opportunity loss** — an existing customer took a potential upsell, cross-sell, or expansion opportunity to a competing org or solution instead of expanding with you. Track this as a separate revenue-risk category, not as churned ARR. Identify these as Closed Lost opportunities on existing accounts where the Opportunity Type is not "New Business" (e.g. "Expansion", "Upsell", "Cross-Sell"), or where the Account already has at least one prior Closed Won opportunity.

#### 2c. Scope filters (infer from Salesforce; do not ask)

Default to **all accounts and all available dimensions**. Build customer grouping analysis only from Salesforce fields that actually exist and are populated:

- Use explicit account segment, tier, customer-size, account type, industry, geography, product, renewal cohort, or acquisition source fields when they exist and are populated.
- Do not invent or infer named account-size segments from generic fields unless Salesforce contains those exact fields or values, or the user explicitly provides a mapping rule.
- If a desired grouping does not exist in Salesforce, do not include it in the analysis. Mention the limitation in the affected section only when it materially changes interpretation, using business language.
- If the user explicitly names a segment, product, region, or focus area, apply that as a filter only if Salesforce has a matching populated field or value.

Never block the report to ask for a segment choice. Do not promise unavailable segmentation in the report or pre-run questionnaire.

---

### Step 3 — Analyze available customer groupings

Before running the five analysis dimensions, build a grouping layer only from available Salesforce fields.

Run grouping queries across these cohorts only when the supporting fields exist and are populated:

**Behavioral cohorts:**
- First-renewal vs. multi-renewal customers
- Discount-led customers vs. full-price customers
- Customers with open support cases in the 90 days before churn vs. those without
- Customers with subscription pauses/skips before churn vs. those without

**Firmographic cohorts:**
- By explicit account segment, tier, or account type when present in Salesforce
- By industry
- By geography / region

**Acquisition cohorts:**
- By original opportunity source (`LeadSource` on Account or Opportunity)
- By contract start quarter

For each cohort, compute:
- Churned ARR and deal count
- Churn rate vs. portfolio average
- Flag any cohort with churn rate more than 1.5× the portfolio average as **elevated risk**

Log which cohorts could not be built due to missing fields internally. Mention the limitation in the affected section only when it materially changes interpretation.

---

### Step 4 — Run the five churn analysis dimensions

Execute all applicable dimensions. For each, read signal definitions in `references/churn-signals.md`.

**Dimension 1: Overall Churn & Renewal Rate**
Query closed opportunities in the period. Separate renewals (Closed Won) from churns (Closed Lost). Compute:
- Total ARR up for renewal
- Renewed ARR
- Churned ARR from lost renewals, cancellations, and downsells only
- Expansion opportunity loss as a separate metric; never include it in GRR, churned ARR, or renewal-rate calculations
- Gross Renewal Rate (GRR) = Renewed ARR / Total ARR up for renewal
- Net Revenue Retention (NRR) if expansion data is available
- **Expansion Opportunity Loss Rate** = Lost expansion ARR / Total potential expansion ARR (if expansion pipeline data is available)

**Dimension 2: Loss Reasons**
Group churned opportunities by loss reason field. Map raw values to the standard taxonomy. Rank by frequency and by churned ARR. Surface the top 5 reasons with deal counts and ARR totals.

**Dimension 3: Churned ARR by Product Line**
Join Opportunity → OpportunityLineItem → Product2. Break down churned ARR by product family and product name.
Product assignment rule: A product name may only be attributed to a churned account if it is confirmed by at least one of: (a) a populated OpportunityLineItem record linked to that specific opportunity, (b) the product name appearing explicitly in the opportunity name or description field for that account, or (c) a transcript or tracker entry that names the product in direct reference to that account. Never infer a product name from a different account's opportunity and apply it to others, even if they appear to be on the same platform. If none of the above conditions are met, record the product as "Not recorded" for that account.
Flag above-average churn rates and concentration risks (>30% of churned ARR in one product).

**Dimension 4: Renewal Cohort Trends**
Group renewals and churns by quarter (or month). Compute GRR per period across the user-provided analysis window. Identify trend direction: declining, improving, stable, or single-period spike. Flag any period below 80% GRR.

**Dimension 5: Cancellation Pattern Analysis**
Classify cancellations: mid-term vs. at-renewal. Compute time-to-cancellation buckets (Early 0–90d / Mid 91–270d / Late 271+d). Break down by the available customer grouping fields.

---

### Step 5 — Root-cause analysis

Cross-reference signals from Steps 3 and 4 to identify *why* churn happened, not just *how much*. Work through these seven root-cause hypotheses systematically:

**Hypothesis 1: Onboarding / fit failure**
Evidence: Early cancellations (0–90 days) represent more than 40% of churned deals, churned accounts had open support cases within 30 days of contract start, OR customer-call/account-team message/Google Sheets tracker evidence mentions onboarding friction, implementation blockers, missing setup help, or unresolved early product-fit issues.
Verdict: Confirm or rule out based on time-to-cancellation buckets, support signal availability, and corroborating evidence.

**Hypothesis 2: Value realisation failure**
Evidence: Mid-contract cancellations (91–270 days) are rising, churned accounts show subscription pause/skip signals before churn, OR customer-call/account-team message/Google Sheets tracker evidence mentions low adoption, weak ROI proof, missing success criteria, repeated unresolved tickets, or lack of executive value recognition.
Verdict: Confirm or rule out based on cancellation timing trend, subscription behaviour fields, and corroborating conversation evidence.

**Hypothesis 3: Competitive displacement or price sensitivity**
Evidence: Loss reason taxonomy shows >30% "Competitor" or "Price/Budget" categories, discount-led cohort has churn rate 1.5× higher than full-price cohort, OR customer-call/account-team message evidence mentions competitor evaluation, alternative vendors, internal substitutes, budget release, procurement objections, or pricing mismatch.
Verdict: Confirm or rule out by cross-referencing loss reasons, discount cohort churn rate, and corroborating conversation evidence.

**Hypothesis 4: Product-specific failure**
Evidence: One product line accounts for >30% of churned ARR (concentration risk flag from Dimension 3), OR that product's churn rate is above portfolio average.
Verdict: Confirm or rule out from Dimension 3 flags.

**Hypothesis 5: Customer grouping or vertical concentration**
Evidence: One available customer grouping, industry, or geography accounts for >50% of churned ARR, AND that cohort's churn rate is above average.
Verdict: Confirm or rule out from available grouping cohort flags and Dimension 5.

**Hypothesis 6: Expansion opportunity loss to competing org**
Evidence: Existing customers have Closed Lost opportunities of type Expansion/Upsell/Cross-Sell, suggesting they chose a competitor or alternative solution for their growth needs rather than expanding with you.
Verdict: Confirm or rule out by identifying Closed Lost opportunities on accounts that already have prior Closed Won opportunities.

**Hypothesis 7: Process, security, or procurement blocker**
Evidence: Customer-call, account-team message, or Google Sheets Escalations and Tickets tracker evidence mentions legal review, security approval, IT deployment, procurement delays, budget timing, vendor onboarding, unresolved tickets, escalations, or executive approval blockers that prevented renewal or expansion despite product fit.
Verdict: Confirm or rule out by tying the blocker to churned or paused accounts and checking whether CRM loss reasons understate the operational blocker.

For each confirmed hypothesis, state:
- The supporting data points (specific metrics and fields)
- The severity (High / Medium / Low based on % of churned ARR attributable)
- Which dimension(s) corroborate it
- Which business-safe evidence type supports it: renewal data, account data, support history, customer-call evidence, account-team message evidence, or Google Sheets tracker evidence

---

### Step 6 — Score churn action priority and produce the report

#### Account action priority

| Priority | Gross Renewal Rate | Churned ARR % of Total |
|---|---|---|
| High Priority | Below 75% | Above 15% |
| Medium Priority | 75–85% | 10–15% |
| Monitor | Above 85% | Below 10% |

Always state the specific account, ARR, renewal, and product-line metrics that drove the priority.

#### Report output

Read `references/output-template.md` exactly. The template is locked — use the Management Churn Intelligence Dashboard layout first, then all nine sections in the order specified, using the defined table schemas, every time.

**Primary output: Word document (`.docx`)**

The fixed deliverable is a Word document. Use `references/output-template.md` as the locked content and dashboard layout specification, then use `references/word-dashboard.md` to create the final `.docx`. Do not ask the user to choose an output format.

The report contains:

- A dark branded header with the period, management action priority badge, and generation metadata
- Five KPI cards in a row: Renewal Loss Rate or GRR, Churned ARR, Accounts Lost, Expansion Opportunity Loss, and Avg Churned Deal Size
- An "Accounts Requiring Management Review" table listing up to 5 accounts by ARR lost, with churn type, loss reason, product line, key signal, and recommended management action
- A two-column panel: Loss Reason Distribution bars + Product Cancellation bars on the left; Management Actions Required (4 items from Section 9) on the right
- A GRR Trend bar chart powered by Chart.js, color-coded by health threshold
- A footer stating the data sources used and confidence level
- The full 9-section detailed report embedded below the dashboard, with all markdown tables converted into Word tables

Follow the placeholder-filling rules in `references/output-template.md` exactly — every [PLACEHOLDER] must be replaced with a real computed value before the Word report is generated.

**Then produce the numbered sections:**

1. Executive Summary
2. Accounts Requiring Management Review
3. Renewal Loss & ARR Impact
4. Loss Reason Analysis
5. Product Line Cancellation Analysis
6. Renewal and Lost Deal Trends
7. Lifecycle & Account Patterns
8. Root-Cause Findings
9. Management Action Plan

If any section has insufficient data, write exactly: *"Insufficient data available for this analysis period."* — never omit the section heading itself.

**Recommendation quality rule:** Every recommendation must name the root cause it addresses, cite the metric evidence in plain business language, and specify a concrete action with an implied owner (Sales Manager, VP of Sales, Customer Success Lead, Product, Operations) and a timeframe.

---

### Step 7 — Companion skills

**Word document (.docx) — primary format for sharing and email**
Read `references/word-dashboard.md`. Fill every [PLACEHOLDER] with real values from the analysis, paste the full report text into FULL_REPORT_TEXT, then run the script to produce the .docx file. Do **not** use `anthropic-skills:docx` for churn reports — use the word-dashboard.md script instead.

**HTML artifact**
Do not produce by default. Generate only if the user explicitly asks for an inline HTML view.

**Spreadsheet with raw churn data** → `anthropic-skills:xlsx`

**Slide deck for management review** → `anthropic-skills:pptx`

**Automated scheduled run** → `anthropic-skills:schedule`

---

### Step 8 — Ask for email delivery and send the report

After producing the Churn Analysis Report, ask the user for the email address that should receive it:

> "What email address should I send the Churn Analysis Report to? I will send it from caseassist@cognida.ai using Microsoft 365."

Wait for explicit user confirmation before sending. If multiple recipients, confirm the list first.

Use the Microsoft 365 connector exposed through the CData connector. Do not use a direct Microsoft Graph, SMTP, or non-CData integration.

#### 8a — Confirm available Office365 procedures

Call `getProcedures()` on the CData connector to identify the correct Office365 send-mail procedure name. Do not hardcode a procedure name — always resolve it from the discovered list.

#### 8b — Construct and execute the send call

Use the procedure name discovered in 8a. The email must use:

- **From:** `caseassist@cognida.ai`
- **To:** the user-confirmed recipient(s)
- **Subject:** `Churn Analysis Report — [DATE]`
- **Body:** 2–3 sentence plain-text management summary
- **Attachment:** if a `.docx`, `.xlsx`, or `.pptx` deliverable was generated, encode as base64 and pass via `@Attachments`. Set `@ContentType` in **lowercase** (e.g. `application/vnd.openxmlformats-officedocument.wordprocessingml.document`). If no file was generated, include the full report in the email body and omit `@Attachments`.

---

## Operating rules

- **Use CData only for all external sources.** Never connect directly to Salesforce, Fathom, Slack, Google Sheets, Microsoft 365, or any other source.
- **Google Sheets tracker is queried via SQL — no downloads.** Access the Escalations and Tickets tracker by querying `[GoogleSheets1].[GoogleSheets].[Escalations and Tickets_Deal Blockers]` and `[GoogleSheets1].[GoogleSheets].[Escalations and Tickets_Escalations]` directly through CData. No file download, no base64, no decode scripts. If the tables are unavailable, note the gap and continue.
- **Never expose technical details.** Never reveal field names, table names, SQL queries, CData operations, or system identifiers in any user-facing text. Translate everything into plain business language.
- **Cite every metric with plain-language sourcing.** "Churned ARR of $480K across 12 lost accounts in Q1–Q2 FY2026" is correct. "Churn is high" is not.
- **Separate quantitative facts from qualitative evidence.** Renewal data is the source of record for ARR and renewal rates. Call evidence, message evidence, and tracker evidence corroborate root causes but do not override CRM metrics.
- **Separate churn types.** Distinguish clearly between voluntary non-renewal, mid-term cancellation, downsell, and involuntary churn wherever the data allows.
- **Never transfer attributes across accounts.** Product names, pricing tiers, product lines, or other attributes confirmed for one account may not be inferred or assigned to a different account unless independently confirmed for that account in its own source records. When a field is unpopulated for an account, write "Not recorded" — do not fill it from a related account's data.
- **Note material data gaps in business language only, inside the section they affect.** Do not add a separate final data-issues section unless the user explicitly asks for one.
- **Never modify records.** Read only.
- **Monetary values in USD** unless records specify otherwise, formatted as $X,XXX,XXX.
- **Root causes drive management actions.** Never produce a recommendation not tied to a confirmed or suspected root cause from Step 5.
- **Report structure is fixed.** Always follow `references/output-template.md` exactly — same sections, same order, every time.
