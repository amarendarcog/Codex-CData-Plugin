# Churn Analysis Report — Output Template

## Template rules (read before producing any report)

These rules are absolute. They ensure the report looks identical in structure every time it is produced, regardless of the data available, so stakeholders can compare reports across periods without disorientation.

1. **Fixed section order.** Produce the Management Churn Intelligence Dashboard first in the Word report, then every numbered section below in the exact order listed. Never add, remove, or rename a numbered section. Keep subsections concise and omit low-signal subsections marked as conditional.
2. **Fixed table schemas.** Every table has defined columns. Never add or remove columns. If a value is unavailable, write `—` in the cell.
3. **No technical language in output.** Field names, table names, system names (CData, Salesforce, Fathom, Slack, SQL), and query logic must never appear. Describe evidence only with business-safe labels such as "company renewal data", "customer-call notes", "account-team messages", or "support history".
4. **Consistent number formatting.** ARR/revenue figures: `$X,XXX,XXX` (USD). Percentages: `XX.X%`. Deal counts: plain integer. Period-over-period change: `+X.X pp` or `−X.X pp`.
5. **No placeholders in the final report.** Every `[bracket]` in this template is an instruction to fill in real data. The delivered report must contain zero unfilled brackets.
6. **Empty sections are never omitted.** If data is unavailable, write exactly: *"Insufficient data available for this analysis period."* under the section heading and continue.
7. **Narrative fields have minimum length.** Every "Key insight" or narrative field must be at least one full sentence with a specific number in it.

---

## STEP 0 — MANAGEMENT CHURN INTELLIGENCE DASHBOARD LAYOUT (always produced first)

**Before rendering any numbered section, generate the Management Churn Intelligence Dashboard in the final Word report.**

The dashboard gives management an immediate portfolio churn read — churned/lost accounts, ARR impact, product line, loss reason, key signal, and next action — without scrolling through every table. The detailed 9-section report follows below the dashboard in the same Word document.

### Filling in each placeholder

| Placeholder | How to determine the value |
|---|---|
| `[HEALTH_CLASS]` | `critical` if GRR < 75%, `at-risk` if 75–85%, `healthy` if > 85% |
| `[HEALTH_LABEL]` | `🔴 CRITICAL` / `🟡 AT RISK` / `🟢 HEALTHY` |
| `[GRR_COLOR_CLASS]` | `critical` / `at-risk` / `healthy` (same thresholds as above) |
| `[GRR_DELTA_TEXT]` | Optional. Use only when a reliable prior period exists, e.g. `↑ 4.2 pp vs prior period`. If no prior period exists, use an empty string. Never write "No prior-period data", "comparison unavailable", or similar warning text. |
| `[GRR_DELTA_CLASS]` | `delta-up-good` if GRR improved vs prior, `delta-down-bad` if declined, `delta-neutral` if flat or blank |
| `[CHURNED_ARR_DELTA_TEXT]` | Optional. Use only when a reliable prior period exists, e.g. `↓ $42K vs prior period`. If no prior period exists, use an empty string. Never write "No prior-period data", "comparison unavailable", or similar warning text. |
| `[CHURNED_ARR_DELTA_CLASS]` | `delta-down-bad` if more ARR churned vs prior, `delta-up-good` if less, `delta-neutral` if flat or blank |
| `[ACCOUNTS_DELTA_TEXT]` | Optional. Use only when a reliable prior period exists, e.g. `↓ 3 vs prior period`. If no prior period exists, use an empty string. Never write "No prior-period data", "comparison unavailable", or similar warning text. |
| `[NRR_COLOR_CLASS]` | `healthy` if NRR >= 100%, `at-risk` if 85–99%, `critical` if < 85%; `delta-neutral` if unavailable |
| `[REASON_N_PCT]` | That reason's % share of all churned accounts (0–100, used directly as CSS bar width) |
| `[PRODUCT_N_PCT]` | That product line's % share of churned ARR (0-100, used directly as CSS bar width) |
| `[CHART_Y_MIN]` | floor(lowest GRR value across all periods − 5), rounded down to nearest 5 |
| `[DATA_SOURCE_SUMMARY]` | e.g. "41 renewal records, 14 customer-call transcripts, 23 account-team messages" |
| `[CONFIDENCE_LEVEL]` | `High` if all three data types available, `Medium` if one missing, `Low` if two missing |
| `[CADENCE_LABEL]` | User-provided reporting period label, e.g. "Jan-Jun 2026" or "FY2026 Q2" |

### Management Actions Required panel

Derive four action lines directly from Section 9 Management Action Plan. Each line = concrete verb + account or cohort + why (one sentence). Owner = the implied team from the recommendation: Sales Manager, VP of Sales, Customer Success Lead, Product, or Operations. Due = urgency label such as "Immediate", "This quarter", or "Next quarter".

### Accounts Requiring Management Review table

List up to 5 accounts with the highest churned ARR in the period for management prioritization. For each row:
- **Churn Type:** Voluntary non-renewal / Mid-term cancellation / Downsell / Expansion loss
- **Loss Reason:** the mapped standard category from Section 4
- **Product Line:** the product family or product name tied to the churn or cancellation, if available
- **Key Signal:** a plain-business-language signal, e.g. "Budget concern raised on call", "Champion departed", "No response x3", "Competitor mentioned in final review"
- **Recommended Action:** the concrete action management should assign (owner, timing, expected outcome)
- **Risk:** `High` if account ARR > average churned deal size; `Med` if at average; `Low` if below

### GRR Trend Chart

Use the four quarterly GRR values from Section 6 (Renewal Trend Analysis). If fewer than four periods are available, use however many exist and set unused array slots to null. Color rule per bar: green (#27ae60) if GRR >= 85%, orange (#e67e22) if 75–84%, red (#c0392b) if < 75%.

---

### EXECUTIVE DASHBOARD LAYOUT — COMPLETE TEMPLATE

Use the structure below as the dashboard layout specification. Substitute every [PLACEHOLDER] with real computed values before rendering. In Word output, recreate the same sections as native Word tables/cards/charts and place the full 9-section report below the dashboard, converting all markdown tables to Word tables.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Churn Analysis Report</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/3.9.1/chart.min.js"></script>
<style>
* { margin: 0; padding: 0; box-sizing: border-box; }
body { font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Arial, sans-serif; background: #f0f2f5; color: #222; font-size: 13px; }

/* Header */
.report-header { background: #111; color: #fff; padding: 20px 32px; display: flex; justify-content: space-between; align-items: flex-start; }
.report-header h1 { font-size: 19px; font-weight: 700; letter-spacing: -0.3px; }
.report-header .sub { font-size: 11px; color: #bbb; margin-top: 5px; }
.report-header .meta-right { font-size: 11px; color: #aaa; text-align: right; line-height: 1.8; }
.health-badge { display: inline-block; padding: 3px 11px; border-radius: 3px; font-size: 11px; font-weight: 700; margin-top: 8px; }
.health-critical { background: #c0392b; color: #fff; }
.health-at-risk  { background: #e67e22; color: #fff; }
.health-healthy  { background: #27ae60; color: #fff; }

/* KPI Cards */
.kpi-row { display: flex; background: #fff; border-bottom: 1px solid #e0e0e0; }
.kpi-card { flex: 1; padding: 18px 24px; border-right: 1px solid #ebebeb; }
.kpi-card:last-child { border-right: none; }
.kpi-label { font-size: 10px; font-weight: 700; color: #999; text-transform: uppercase; letter-spacing: 0.6px; }
.kpi-value { font-size: 30px; font-weight: 700; color: #111; margin: 7px 0 5px; line-height: 1; }
.kpi-value.critical { color: #c0392b; }
.kpi-value.at-risk  { color: #e67e22; }
.kpi-value.healthy  { color: #27ae60; }
.kpi-delta { font-size: 11px; }
.delta-up-good { color: #27ae60; }
.delta-down-bad { color: #c0392b; }
.delta-neutral { color: #999; }

/* Cards */
.card { background: #fff; margin: 12px 0 0; }
.card-header { padding: 12px 24px 10px; font-size: 10px; font-weight: 700; color: #999; text-transform: uppercase; letter-spacing: 0.6px; border-bottom: 1px solid #f2f2f2; }

/* Data Table */
.data-table { width: 100%; border-collapse: collapse; font-size: 12px; }
.data-table th { padding: 8px 20px; text-align: left; font-size: 10px; font-weight: 700; color: #999; text-transform: uppercase; letter-spacing: 0.4px; border-bottom: 1px solid #ebebeb; background: #fafafa; }
.data-table td { padding: 10px 20px; border-bottom: 1px solid #f5f5f5; color: #333; vertical-align: top; }
.data-table tr:last-child td { border-bottom: none; }
.data-table tr:hover td { background: #fafafa; }
.acct-sub { color: #999; font-size: 11px; margin-top: 2px; }
.risk-High { color: #c0392b; font-weight: 700; }
.risk-Med  { color: #e67e22; font-weight: 700; }
.risk-Low  { color: #27ae60; font-weight: 700; }
.stage-chip { display: inline-block; padding: 2px 9px; border-radius: 3px; font-size: 11px; font-weight: 600; background: #eef2ff; color: #3b5bdb; }

/* Two-column layout */
.two-col { display: flex; }
.two-col .left-panel  { flex: 1.2; border-right: 1px solid #e0e0e0; }
.two-col .right-panel { flex: 1; }

/* Distribution bars */
.dist-row { display: flex; align-items: center; padding: 8px 20px; border-bottom: 1px solid #f5f5f5; gap: 12px; }
.dist-row:last-child { border-bottom: none; }
.dist-label { width: 150px; font-size: 12px; color: #444; flex-shrink: 0; }
.dist-bar-wrap { flex: 1; background: #f0f0f0; border-radius: 3px; height: 9px; }
.dist-bar { height: 9px; border-radius: 3px; min-width: 4px; }
.dist-meta { width: 165px; text-align: right; font-size: 11px; color: #888; flex-shrink: 0; }
.dist-meta strong { color: #333; font-size: 12px; }

/* Actions */
.action-item { display: flex; gap: 14px; padding: 10px 20px; border-bottom: 1px solid #f5f5f5; }
.action-item:last-child { border-bottom: none; }
.action-num { font-size: 13px; font-weight: 700; color: #ccc; flex-shrink: 0; width: 18px; padding-top: 1px; }
.action-body { flex: 1; }
.action-text { font-size: 12px; color: #222; line-height: 1.5; }
.action-meta { font-size: 11px; color: #999; margin-top: 3px; }

/* Trend chart */
.trend-section { background: #fff; margin-top: 12px; padding: 16px 24px 20px; }
.trend-title { font-size: 10px; font-weight: 700; color: #999; text-transform: uppercase; letter-spacing: 0.6px; margin-bottom: 14px; }

/* Footer */
.report-footer { font-size: 10px; color: #bbb; text-align: right; padding: 10px 24px; border-top: 1px solid #e0e0e0; background: #fff; margin-top: 2px; }

/* Full report */
.full-report { background: #fff; margin-top: 12px; padding: 36px 40px 48px; }
.full-report h1.report-title { font-size: 20px; font-weight: 700; color: #111; margin-bottom: 4px; }
.full-report .report-byline { font-size: 12px; color: #999; margin-bottom: 36px; line-height: 1.8; }
.full-report h2 { font-size: 14px; font-weight: 700; color: #111; margin: 32px 0 10px; padding-bottom: 8px; border-bottom: 2px solid #f0f0f0; }
.full-report h3 { font-size: 13px; font-weight: 700; color: #333; margin: 20px 0 8px; }
.full-report p { font-size: 13px; color: #444; line-height: 1.75; margin-bottom: 10px; }
.full-report blockquote { border-left: 3px solid #ddd; padding: 8px 16px; color: #666; font-style: italic; margin: 12px 0; }
.full-report table { width: 100%; border-collapse: collapse; font-size: 12px; margin: 12px 0 20px; }
.full-report th { padding: 8px 12px; text-align: left; font-size: 11px; font-weight: 700; color: #666; background: #f8f8f8; border: 1px solid #ebebeb; }
.full-report td { padding: 8px 12px; border: 1px solid #ebebeb; color: #333; }
.full-report ul { padding-left: 20px; margin: 8px 0; }
.full-report li { font-size: 13px; color: #444; line-height: 1.75; }
</style>
</head>
<body>

<!-- ═══════════ HEADER ═══════════ -->
<div class="report-header">
  <div>
    <h1>Churn Analysis Report</h1>
    <div class="sub">Account Health Tracking &nbsp;·&nbsp; Renewal, customer, and account-team evidence</div>
    <div><span class="health-badge health-[HEALTH_CLASS]">[HEALTH_LABEL]</span></div>
  </div>
  <div class="meta-right">
    Week of [ISSUE_DATE]<br>
    Prepared by Account Management<br>
    Period: [PERIOD_LABEL]
  </div>
</div>

<!-- ═══════════ KPI CARDS ═══════════ -->
<div class="kpi-row">
  <div class="kpi-card">
    <div class="kpi-label">Gross Renewal Rate</div>
    <div class="kpi-value [GRR_COLOR_CLASS]">[GRR_VALUE]%</div>
    <div class="kpi-delta [GRR_DELTA_CLASS]">[GRR_DELTA_TEXT]</div>
  </div>
  <div class="kpi-card">
    <div class="kpi-label">Churned ARR</div>
    <div class="kpi-value">[CHURNED_ARR]</div>
    <div class="kpi-delta [CHURNED_ARR_DELTA_CLASS]">[CHURNED_ARR_DELTA_TEXT]</div>
  </div>
  <div class="kpi-card">
    <div class="kpi-label">Accounts Lost</div>
    <div class="kpi-value">[ACCOUNTS_LOST]</div>
    <div class="kpi-delta [ACCOUNTS_DELTA_CLASS]">[ACCOUNTS_DELTA_TEXT]</div>
  </div>
  <div class="kpi-card">
    <div class="kpi-label">Net Revenue Retention</div>
    <div class="kpi-value [NRR_COLOR_CLASS]">[NRR_VALUE]</div>
    <div class="kpi-delta [NRR_DELTA_CLASS]">[NRR_DELTA]</div>
  </div>
  <div class="kpi-card">
    <div class="kpi-label">Avg Churned Deal Size</div>
    <div class="kpi-value">[AVG_DEAL_SIZE]</div>
    <div class="kpi-delta delta-neutral">across [ACCOUNTS_LOST] lost accounts</div>
  </div>
</div>

<!-- ═══════════ ACCOUNTS REQUIRING MANAGEMENT REVIEW ═══════════ -->
<div class="card">
  <div class="card-header">Accounts Requiring Management Review &nbsp;·&nbsp; [PERIOD_LABEL]</div>
  <table class="data-table">
    <thead>
      <tr>
        <th>Account</th>
        <th>ARR Lost</th>
        <th>Churn Type</th>
        <th>Loss Reason</th>
        <th>Product Line</th>
        <th>Key Signal</th>
        <th>Recommended Action</th>
        <th>Risk</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td><strong>[ACCOUNT_1_NAME]</strong><div class="acct-sub">[ACCOUNT_1_SUBTYPE]</div></td>
        <td><strong>[ACCOUNT_1_ARR]</strong></td>
        <td><span class="stage-chip">[ACCOUNT_1_CHURN_TYPE]</span></td>
        <td>[ACCOUNT_1_LOSS_REASON]</td>
        <td>[ACCOUNT_1_PRODUCT_LINE]</td>
        <td style="color:#555">[ACCOUNT_1_SIGNAL]</td>
        <td>[ACCOUNT_1_REC_ACTION]</td>
        <td><span class="risk-[ACCOUNT_1_RISK]">[ACCOUNT_1_RISK]</span></td>
      </tr>
      <tr>
        <td><strong>[ACCOUNT_2_NAME]</strong><div class="acct-sub">[ACCOUNT_2_SUBTYPE]</div></td>
        <td><strong>[ACCOUNT_2_ARR]</strong></td>
        <td><span class="stage-chip">[ACCOUNT_2_CHURN_TYPE]</span></td>
        <td>[ACCOUNT_2_LOSS_REASON]</td>
        <td>[ACCOUNT_2_PRODUCT_LINE]</td>
        <td style="color:#555">[ACCOUNT_2_SIGNAL]</td>
        <td>[ACCOUNT_2_REC_ACTION]</td>
        <td><span class="risk-[ACCOUNT_2_RISK]">[ACCOUNT_2_RISK]</span></td>
      </tr>
      <tr>
        <td><strong>[ACCOUNT_3_NAME]</strong><div class="acct-sub">[ACCOUNT_3_SUBTYPE]</div></td>
        <td><strong>[ACCOUNT_3_ARR]</strong></td>
        <td><span class="stage-chip">[ACCOUNT_3_CHURN_TYPE]</span></td>
        <td>[ACCOUNT_3_LOSS_REASON]</td>
        <td>[ACCOUNT_3_PRODUCT_LINE]</td>
        <td style="color:#555">[ACCOUNT_3_SIGNAL]</td>
        <td>[ACCOUNT_3_REC_ACTION]</td>
        <td><span class="risk-[ACCOUNT_3_RISK]">[ACCOUNT_3_RISK]</span></td>
      </tr>
      <tr>
        <td><strong>[ACCOUNT_4_NAME]</strong><div class="acct-sub">[ACCOUNT_4_SUBTYPE]</div></td>
        <td><strong>[ACCOUNT_4_ARR]</strong></td>
        <td><span class="stage-chip">[ACCOUNT_4_CHURN_TYPE]</span></td>
        <td>[ACCOUNT_4_LOSS_REASON]</td>
        <td>[ACCOUNT_4_PRODUCT_LINE]</td>
        <td style="color:#555">[ACCOUNT_4_SIGNAL]</td>
        <td>[ACCOUNT_4_REC_ACTION]</td>
        <td><span class="risk-[ACCOUNT_4_RISK]">[ACCOUNT_4_RISK]</span></td>
      </tr>
      <tr>
        <td><strong>[ACCOUNT_5_NAME]</strong><div class="acct-sub">[ACCOUNT_5_SUBTYPE]</div></td>
        <td><strong>[ACCOUNT_5_ARR]</strong></td>
        <td><span class="stage-chip">[ACCOUNT_5_CHURN_TYPE]</span></td>
        <td>[ACCOUNT_5_LOSS_REASON]</td>
        <td>[ACCOUNT_5_PRODUCT_LINE]</td>
        <td style="color:#555">[ACCOUNT_5_SIGNAL]</td>
        <td>[ACCOUNT_5_REC_ACTION]</td>
        <td><span class="risk-[ACCOUNT_5_RISK]">[ACCOUNT_5_RISK]</span></td>
      </tr>
    </tbody>
  </table>
</div>

<!-- ═══════════ LOSS REASON DISTRIBUTION + MANAGEMENT ACTIONS ═══════════ -->
<div class="card">
  <div class="two-col">

    <!-- Left: Distribution bars -->
    <div class="left-panel">
      <div class="card-header">Loss Reason Distribution</div>
      <!-- REASON_N_PCT = that reason's % share of all churned accounts (0–100 for CSS bar width) -->
      <div class="dist-row">
        <div class="dist-label">[REASON_1_LABEL]</div>
        <div class="dist-bar-wrap"><div class="dist-bar" style="width:[REASON_1_PCT]%; background:#e74c3c;"></div></div>
        <div class="dist-meta"><strong>[REASON_1_COUNT] accounts</strong> &nbsp;·&nbsp; [REASON_1_ARR]</div>
      </div>
      <div class="dist-row">
        <div class="dist-label">[REASON_2_LABEL]</div>
        <div class="dist-bar-wrap"><div class="dist-bar" style="width:[REASON_2_PCT]%; background:#e67e22;"></div></div>
        <div class="dist-meta"><strong>[REASON_2_COUNT] accounts</strong> &nbsp;·&nbsp; [REASON_2_ARR]</div>
      </div>
      <div class="dist-row">
        <div class="dist-label">[REASON_3_LABEL]</div>
        <div class="dist-bar-wrap"><div class="dist-bar" style="width:[REASON_3_PCT]%; background:#f39c12;"></div></div>
        <div class="dist-meta"><strong>[REASON_3_COUNT] accounts</strong> &nbsp;·&nbsp; [REASON_3_ARR]</div>
      </div>
      <div class="dist-row">
        <div class="dist-label">[REASON_4_LABEL]</div>
        <div class="dist-bar-wrap"><div class="dist-bar" style="width:[REASON_4_PCT]%; background:#3498db;"></div></div>
        <div class="dist-meta"><strong>[REASON_4_COUNT] accounts</strong> &nbsp;·&nbsp; [REASON_4_ARR]</div>
      </div>
      <div class="dist-row">
        <div class="dist-label">[REASON_5_LABEL]</div>
        <div class="dist-bar-wrap"><div class="dist-bar" style="width:[REASON_5_PCT]%; background:#95a5a6;"></div></div>
        <div class="dist-meta"><strong>[REASON_5_COUNT] accounts</strong> &nbsp;·&nbsp; [REASON_5_ARR]</div>
      </div>

      <div class="card-header" style="margin-top:8px;">Product-Specific Cancellation Impact</div>
      <!-- PRODUCT_N_PCT = that product line's % share of churned ARR (0–100 for CSS bar width) -->
      <div class="dist-row">
        <div class="dist-label">[PRODUCT_1_LABEL]</div>
        <div class="dist-bar-wrap"><div class="dist-bar" style="width:[PRODUCT_1_PCT]%; background:#c0392b;"></div></div>
        <div class="dist-meta"><strong>[PRODUCT_1_COUNT] accounts</strong> &nbsp;·&nbsp; [PRODUCT_1_ARR]</div>
      </div>
      <div class="dist-row">
        <div class="dist-label">[PRODUCT_2_LABEL]</div>
        <div class="dist-bar-wrap"><div class="dist-bar" style="width:[PRODUCT_2_PCT]%; background:#e67e22;"></div></div>
        <div class="dist-meta"><strong>[PRODUCT_2_COUNT] accounts</strong> &nbsp;·&nbsp; [PRODUCT_2_ARR]</div>
      </div>
      <div class="dist-row">
        <div class="dist-label">[PRODUCT_3_LABEL]</div>
        <div class="dist-bar-wrap"><div class="dist-bar" style="width:[PRODUCT_3_PCT]%; background:#3498db;"></div></div>
        <div class="dist-meta"><strong>[PRODUCT_3_COUNT] accounts</strong> &nbsp;·&nbsp; [PRODUCT_3_ARR]</div>
      </div>
    </div>

    <!-- Right: Actions Required -->
    <div class="right-panel">
      <div class="card-header">Management Actions Required</div>
      <!-- Derive from Section 9. Text = concrete verb + what + why. Owner = team. Due = urgency. -->
      <div class="action-item">
        <div class="action-num">1</div>
        <div class="action-body">
          <div class="action-text">[ACTION_1_TEXT]</div>
          <div class="action-meta">Owner: [ACTION_1_OWNER] &nbsp;·&nbsp; Due: [ACTION_1_DUE]</div>
        </div>
      </div>
      <div class="action-item">
        <div class="action-num">2</div>
        <div class="action-body">
          <div class="action-text">[ACTION_2_TEXT]</div>
          <div class="action-meta">Owner: [ACTION_2_OWNER] &nbsp;·&nbsp; Due: [ACTION_2_DUE]</div>
        </div>
      </div>
      <div class="action-item">
        <div class="action-num">3</div>
        <div class="action-body">
          <div class="action-text">[ACTION_3_TEXT]</div>
          <div class="action-meta">Owner: [ACTION_3_OWNER] &nbsp;·&nbsp; Due: [ACTION_3_DUE]</div>
        </div>
      </div>
      <div class="action-item">
        <div class="action-num">4</div>
        <div class="action-body">
          <div class="action-text">[ACTION_4_TEXT]</div>
          <div class="action-meta">Owner: [ACTION_4_OWNER] &nbsp;·&nbsp; Due: [ACTION_4_DUE]</div>
        </div>
      </div>
    </div>

  </div>
</div>

<!-- ═══════════ GRR TREND CHART ═══════════ -->
<div class="trend-section">
  <div class="trend-title">Gross Renewal Rate — Quarterly Trend</div>
  <canvas id="grrTrendChart" style="max-height:190px;"></canvas>
</div>

<!-- ═══════════ FOOTER ═══════════ -->
<div class="report-footer">
  AI-synthesized from [DATA_SOURCE_SUMMARY] &nbsp;·&nbsp; Confidence: [CONFIDENCE_LEVEL] &nbsp;·&nbsp; Sources verified
</div>

<!-- ═══════════ FULL DETAILED REPORT ═══════════ -->
<div class="full-report">
  <h1 class="report-title">CHURN ANALYSIS REPORT</h1>
  <div class="report-byline">
    Prepared by: Account Management &nbsp;·&nbsp;
    Period: [PERIOD_LABEL] &nbsp;·&nbsp;
    Reporting cadence: [CADENCE_LABEL] &nbsp;·&nbsp;
    Issued: [ISSUE_DATE] &nbsp;·&nbsp;
    Classification: Internal — Confidential
  </div>

  <!--
    INSERT ALL 9 SECTIONS BELOW IN ORDER.
    Convert every markdown table to an HTML <table> using .full-report table/th/td styles.
    Keep all section headings, narratives, and table schemas exactly as specified in the numbered sections below.

    Section 1: Executive Summary
    Section 2: Accounts Requiring Management Review
    Section 3: Renewal Loss & ARR Impact
    Section 4: Loss Reason Analysis
    Section 5: Product Line Cancellation Analysis
    Section 6: Renewal and Lost Deal Trends
    Section 7: Lifecycle & Account Patterns
    Section 8: Root-Cause Findings
    Section 9: Management Action Plan
  -->

</div>

<!-- ═══════════ CHART SCRIPT ═══════════ -->
<script>
(function () {
  // GRR values from Section 6 — fill in real numeric values, e.g. 88.1 (not strings)
  // Use null for missing periods.
  const labels    = ['[PERIOD_Q1]', '[PERIOD_Q2]', '[PERIOD_Q3]', '[PERIOD_Q4]'];
  const grrValues = [[GRR_Q1], [GRR_Q2], [GRR_Q3], [GRR_Q4]];
  const barColors = grrValues.map(v => v === null ? '#ddd' : v >= 85 ? '#27ae60' : v >= 75 ? '#e67e22' : '#c0392b');

  const ctx = document.getElementById('grrTrendChart').getContext('2d');
  new Chart(ctx, {
    type: 'bar',
    data: {
      labels: labels,
      datasets: [{
        label: 'GRR %',
        data: grrValues,
        backgroundColor: barColors,
        borderRadius: 4,
        borderSkipped: false
      }]
    },
    options: {
      responsive: true,
      plugins: {
        legend: { display: false },
        tooltip: { callbacks: { label: ctx => ctx.parsed.y !== null ? 'GRR: ' + ctx.parsed.y.toFixed(1) + '%' : 'No data' } }
      },
      scales: {
        y: {
          min: [CHART_Y_MIN],
          max: 100,
          ticks: { callback: v => v + '%', font: { size: 11 }, color: '#999' },
          grid: { color: '#f0f0f0' }
        },
        x: {
          ticks: { font: { size: 11 }, color: '#666' },
          grid: { display: false }
        }
      }
    }
  });
})();
</script>

</body>
</html>
```

---

---

# CHURN ANALYSIS REPORT

**Prepared by:** Account Management
**Period:** [Month DD, YYYY] – [Month DD, YYYY]
**Reporting cadence:** [User-provided reporting period]
**Issued:** [Month DD, YYYY]
**Classification:** Internal — Confidential

---

## 1. Executive Summary

**Management Action Priority:** &nbsp; [High Priority / Medium Priority / Monitor]

> [Two to three sentences written for a sales leader or executive audience. State how many accounts were lost or churned, total ARR impacted, the main loss reason or product-line driver, and the most urgent AM follow-up required. Be direct and quantitative. Example: "The trailing 12 months show $1.2M in churned ARR across 18 accounts, with price sensitivity and competitive displacement accounting for 63% of churned revenue. Three high-ARR accounts require immediate AM follow-up because similar upcoming renewals show the same objection pattern."]

---

## 2. Accounts Requiring Management Review

*Which churned, lost, or expansion-loss accounts require management review, assignment, and follow-up.*

**Total accounts requiring management review:** [N]

| Account | Renewal / Deal Status | ARR Impact | Product Line | Loss Reason | Key Signal | Recommended Action |
|---|---|---|---|---|---|---|
| [Account name] | [Lost renewal / Churn / Lost expansion / Downsell] | $[X,XXX,XXX] | [Product] | [Reason] | [Plain-language signal] | [Concrete management-assigned action] |
| [Account name] | [Status] | $[X,XXX,XXX] | [Product] | [Reason] | [Signal] | [Action] |
| [Account name] | [Status] | $[X,XXX,XXX] | [Product] | [Reason] | [Signal] | [Action] |

*Prioritize accounts by ARR impact, renewal/lost-deal status, product-specific cancellation impact, and whether a clear follow-up action exists. If no usable account-level records exist, write exactly: "Insufficient data available for this analysis period."*

**Management focus insight:** [One sentence identifying the highest-priority account or repeat account pattern and the ARR/product impact.]

---

## 3. Renewal Loss & ARR Impact

*Renewal loss, churned ARR, and lost expansion impact for the period, with prior-period comparison.*

| Period | ARR Up for Renewal | Renewed ARR | Churned ARR | Gross Renewal Rate |
|---|---|---|---|---|
| [Q/Period label] | $[X,XXX,XXX] | $[X,XXX,XXX] | $[X,XXX,XXX] | [XX.X%] |
| [Prior period] | $[X,XXX,XXX] | $[X,XXX,XXX] | $[X,XXX,XXX] | [XX.X%] |

**Churn breakdown by type:**

| Churn Type | Accounts | ARR | % of Total Churned ARR |
|---|---|---|---|
| Voluntary non-renewal (at contract end) | [N] | $[X,XXX,XXX] | [XX.X%] |
| Mid-term cancellation | [N] | $[X,XXX,XXX] | [XX.X%] |
| Downsell (partial ARR loss) | [N] | $[X,XXX,XXX] | [XX.X%] |
| Expansion opportunity loss | [N] | $[X,XXX,XXX] | [XX.X%] |
| Involuntary (payment / administrative) | [N] | $[X,XXX,XXX] | [XX.X%] |

**Period commentary:** [One sentence stating whether the renewal rate improved or declined, by how much, and whether this is within or outside normal variance.]

---

## 4. Loss Reason Analysis

*Why accounts chose not to renew, ranked by revenue impact.*

| Reason Category | Accounts | Churned ARR | Share of Churned ARR |
|---|---|---|---|
| [Reason] | [N] | $[X,XXX,XXX] | [XX.X%] |
| [Reason] | [N] | $[X,XXX,XXX] | [XX.X%] |
| [Reason] | [N] | $[X,XXX,XXX] | [XX.X%] |
| [Reason] | [N] | $[X,XXX,XXX] | [XX.X%] |
| [Reason] | [N] | $[X,XXX,XXX] | [XX.X%] |

*Standard reason categories: Price / Budget · Competitor · Product Fit · Champion Departure · No Decision · Service or Support · Contract Terms · Not Recorded*

**Key insight:** [One sentence on the dominant reason category and its revenue implication.]

---

## 5. Product Line Cancellation Analysis

*Churned ARR, lost deals, and cancellation concentration broken down by product or product family.*

| Product / Line | ARR Up for Renewal | Churned ARR | Accounts Lost | Product Churn Rate | Flag |
|---|---|---|---|---|---|
| [Product] | $[X,XXX,XXX] | $[X,XXX,XXX] | [N] | [XX.X%] | [⚠ Above average / ⚠ Concentration risk / —] |
| [Product] | $[X,XXX,XXX] | $[X,XXX,XXX] | [N] | [XX.X%] | [⚠ Above average / ⚠ Concentration risk / —] |

*Concentration risk is flagged when a single product accounts for more than 30% of total churned ARR.*

**Key insight:** [One sentence identifying the product with the greatest cancellation impact and what management should monitor in related accounts.]

---

## 6. Renewal and Lost Deal Trends

*Renewal losses and lost deals over the last four periods to identify directional movement.*

| Period | ARR Up for Renewal | Renewed ARR | Churned ARR | GRR | Direction |
|---|---|---|---|---|---|
| [Q/Period — oldest] | $[X,XXX,XXX] | $[X,XXX,XXX] | $[X,XXX,XXX] | [XX.X%] | — |
| [Q/Period] | $[X,XXX,XXX] | $[X,XXX,XXX] | $[X,XXX,XXX] | [XX.X%] | [↑ / ↓ / →] |
| [Q/Period] | $[X,XXX,XXX] | $[X,XXX,XXX] | $[X,XXX,XXX] | [XX.X%] | [↑ / ↓ / →] |
| [Q/Period — current] | $[X,XXX,XXX] | $[X,XXX,XXX] | $[X,XXX,XXX] | [XX.X%] | [↑ / ↓ / →] |

**Trend assessment:** [Declining / Improving / Stable / Single-period anomaly]

**Trend commentary:** [One to two sentences describing the directional pattern, whether any period fell below the 80% GRR concern threshold, and what that means for management renewal strategy.]

---

## 7. Lifecycle & Account Patterns

*When in the customer lifecycle churn occurs, where material concentration exists, and what that implies for management intervention.*

**Time-to-cancellation since contract start:**

| Lifecycle Stage | Accounts | Churned ARR | Share | Likely Driver |
|---|---|---|---|---|
| Early — within first 90 days | [N] | $[X,XXX,XXX] | [XX.X%] | Onboarding or product fit failure |
| Mid-contract — 91 to 270 days | [N] | $[X,XXX,XXX] | [XX.X%] | Value realisation or engagement failure |
| Mature — beyond 270 days | [N] | $[X,XXX,XXX] | [XX.X%] | Competitive pressure or price sensitivity |

**Material concentration insight:** [One sentence only if an industry, geography, or segment accounts for more than 40% of churned ARR or is clearly actionable. If no material concentration exists, omit this line.]

**Key insight:** [One sentence on the most significant timing or concentration pattern and the management response it warrants.]

---

## 8. Root-Cause Findings

*Summary of confirmed and suspected drivers of churn this period.*

| Finding | Status | Severity | Revenue Impacted | Supporting Evidence |
|---|---|---|---|---|
| Onboarding or product fit failure | [Confirmed / Suspected / Not identified] | [High / Medium / Low / —] | $[X,XXX,XXX] | [Plain-language description of evidence] |
| Value realisation or engagement failure | [Confirmed / Suspected / Not identified] | [High / Medium / Low / —] | $[X,XXX,XXX] | [Plain-language description of evidence] |
| Competitive displacement or price sensitivity | [Confirmed / Suspected / Not identified] | [High / Medium / Low / —] | $[X,XXX,XXX] | [Plain-language description of evidence] |
| Product-specific performance gap | [Confirmed / Suspected / Not identified] | [High / Medium / Low / —] | $[X,XXX,XXX] | [Plain-language description of evidence] |
| Segment or vertical concentration risk | [Confirmed / Suspected / Not identified] | [High / Medium / Low / —] | $[X,XXX,XXX] | [Plain-language description of evidence] |
| Process, security, or procurement blocker | [Confirmed / Suspected / Not identified] | [High / Medium / Low / —] | $[X,XXX,XXX] | [Plain-language description of evidence] |

**Synthesis:** [Two to three sentences connecting the confirmed findings into a coherent narrative. Describe the dominant pattern, whether it is structural or episodic, and where it is concentrated. This paragraph is written for a Chief Revenue Officer or VP of Sales audience.]

---

## 9. Management Action Plan

*Three prioritized management actions, each directly linked to a confirmed or suspected root-cause finding, with ownership and accountability assigned to the appropriate leadership or cross-functional team.*

---

**Action 1 — [Action title]**

| | |
|---|---|
| **Addresses** | [Root-cause finding name] — [Severity] |
| **Business case** | [One sentence quantifying the opportunity. Example: "Reducing early-lifecycle churn by 25% would recover approximately $80K in ARR annually based on current early-cancellation volumes."] |
| **Recommended action** | [Specific, concrete action. Name the management owner (Sales Manager, VP of Sales, Customer Success Lead, Product, Operations), the trigger or timing, and the expected outcome. Example: "Sales Manager to assign save-plan reviews within 10 business days for accounts matching the competitor-loss pattern, targeting the 47% of churned ARR tied to competitive displacement."] |
| **Priority** | [Immediate / Near-term (next quarter) / Monitor] |

---

**Action 2 — [Action title]**

| | |
|---|---|
| **Addresses** | [Root-cause finding name] — [Severity] |
| **Business case** | [One sentence quantifying the opportunity.] |
| **Recommended action** | [Specific, concrete action with management owner (Sales Manager, VP of Sales, CS Lead, Product, Operations), timing, and expected outcome.] |
| **Priority** | [Immediate / Near-term / Monitor] |

---

**Action 3 — [Action title]**

| | |
|---|---|
| **Addresses** | [Root-cause finding name] — [Severity] |
| **Business case** | [One sentence quantifying the opportunity.] |
| **Recommended action** | [Specific, concrete action with management owner (Sales Manager, VP of Sales, CS Lead, Product, Operations), timing, and expected outcome.] |
| **Priority** | [Immediate / Near-term / Monitor] |

---

*Churn Analysis Report · [Period] · Management Intelligence · Internal — Confidential*
