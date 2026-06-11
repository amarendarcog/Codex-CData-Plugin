# Pipeline Intelligence Report — Output Template

Use this exact structure for every Pipeline Intelligence Report. The report is designed for a VP of Sales or CRO — concise, visual-first, and action-oriented. Adapt section content to what the data shows. If data is insufficient for a section, state that briefly in business terms. Do not omit sections except the conditional Rep Activity Exceptions section when there are no exceptions.

When rendering as a `.docx` report (Step 7), include the cover page block below. In chat output, omit the cover page and start directly from the Overview section.

---

## COVER PAGE (docx report only)

```
Pipeline Intelligence Report
[DATE] · [Quarter Scope, e.g., Q2 FY2026 + Q3 FY2026] · Confidential — Internal Use Only
```

Keep cover metadata visually small and on one line under the title. After the cover header, render the full executive dashboard immediately. Avoid hard page breaks that create empty pages or unnecessary discontinuity; only insert a page break when the next section would otherwise start awkwardly at the very bottom of a page.

---

### Row A — Six KPI Cards

Two rows of three cards. Each card: shaded `#F2F7FB`, thick left-border accent, metric name in small caps at top, large bold value in the middle, sub-label in smaller text at the bottom.

| Card | Metric Name | Primary Value | Sub-Label Logic | Accent |
|------|-------------|---------------|-----------------|--------|
| 1 | TOTAL PIPELINE | $X.XM | Show up/down $XK vs last week only if reliable prior-period data exists; otherwise leave the sub-label blank | `#2E75B6` |
| 2 | AT-RISK ARR | $X.XM | "N deals flagged High Risk" | `#C00000` |
| 3 | COMMIT COVERAGE | $X.XM commit | Use quota/target coverage if available; otherwise show commit share of pipeline and label it as such, not target coverage | `#70AD47` |
| 4 | OPEN OPPORTUNITIES | N | "N at risk" amber (High + Medium Risk count) | `#2E75B6` |
| 5 | AVG DAYS IN STAGE | Xd | "+Xd vs target" amber if above 14d median; "-Xd vs target" green if below | `#ED7D31` |
| 6 | CLOSING THIS QUARTER | N deals | "$X.XM due this quarter" | `#5B9BD5` |

---

### Row B — Top Deals Requiring Attention

Full-width table. Header row shaded `#1F3864`, white bold text.

| ACCOUNT | TYPE | VALUE | STAGE | DAYS STALLED | RISK | LAST SIGNAL |
|---------|------|-------|-------|-------------|------|-------------|
| [Account name] | [New logo / Renewal / Expansion / Upsell] | $[X]K | [Stage] | [N]d | 🔴 High | [e.g. Budget concern raised] |

Rules:
- Top 5–6 deals, ranked High → Medium → Low, then Days Stalled descending
- Risk cell: `#C00000` High / `#ED7D31` Medium / `#70AD47` Low — white text
- Days Stalled: red (`#C00000`) for 30+ days, amber (`#ED7D31`) for 14–29 days, black for < 14 days
- Last Signal: one short phrase (e.g. "Budget concern raised", "No response x3", "Strong exec interest", "Open escalation ticket")
- Alternate row shading `#F2F7FB` / white

---

### Row C — Stage Distribution (left) + Actions Needed This Week (right)

Two equal-width columns:

**Left — Stage Distribution**
One row per stage. Stage name (bold), proportional fill bar (shaded table cell), deal count and "$X.XM" right-aligned.
- Discovery: `#BDD7EE`
- Demo / Eval: `#9DC3E6`
- Proposal / Quote Sent: `#5B9BD5`
- Negotiation / Contract: `#2E75B6`
- Verbal Commit / Pending: `#1F3864`

**Right — Actions Needed This Week**
Heading: "ACTIONS NEEDED THIS WEEK" bold, `#1F3864`.
Numbered list, 3–5 items, ordered by urgency. Each action names the specific account or rep and states what the executive should do — one sentence, no jargon.

Example tone:
> 1. Re-engage [Account] — ghosted 34 days. Escalate to exec sponsor before Friday.
> 2. Review [Rep]'s pipeline in the 1:1 — 3 deals have not moved in 3+ weeks.
> 3. Approve revised proposal for [Account] to unblock the negotiation stage.

---

### Row D — Risk Distribution (left) + Forecast Category (right)

Two equal-width columns:

**Left — Risk Distribution**
Three proportional blocks side by side, with cell widths proportional to pipeline value:
- 🔴 High: `#C00000` fill, white text — "N deals · $X.XM"
- 🟡 Medium: `#ED7D31` fill, white text — "N deals · $X.XM"
- 🟢 Low: `#70AD47` fill, white text — "N deals · $X.XM"

**Right — Forecast Category**
Single horizontal segmented bar (3-cell row, widths proportional to value):
- Commit: `#70AD47` fill — "$X.XM · N deals"
- Best Case: `#5B9BD5` fill — "$X.XM · N deals"
- Pipeline: `#BDD7EE` fill — "$X.XM · N deals"

---

Use shaded table cells for all visuals. The dashboard must be visually complete and immediately legible to a sales leader with 60 seconds. If data is unavailable for a metric, show "N/A" with a brief business-language note.

---

## Section 2 — Executive Overview

2–3 sentences covering total pipeline value, open deal count, quarter scope, and where pipeline weight is concentrated. Example: "Most pipeline is concentrated in Proposal and Negotiation stages. $X.XM is still early-stage for this quarter, with [N] deals in Discovery or Demo." No stage breakdown table — the dashboard visual covers it.

One-line forecast read: [One sentence interpreting whether commit, best case, and pipeline mix is enough for the quarter; do not repeat dashboard totals unless needed for the conclusion.]

---

**Risk note:** 🔴 High means immediate leadership action is needed, 🟡 Medium means watch closely, and 🟢 Low means progressing normally. Limited-visibility notes appear only when activity, buyer engagement, or stage-history evidence is materially incomplete.

---

## Section 3 — Risk & Forecast

### Risk Drivers

Short paragraph (3–5 sentences) identifying the top 3 signals driving risk across the pipeline, with deal count and value exposure for each. Focus on explaining *why* deals are at risk — not repeating the High / Medium / Low count split, which is already on the dashboard.

Example: "The most common risk driver this period is deal inactivity, affecting [N] deals worth $X.XM — [N] of these are closing this quarter. Missing next steps compound the inactivity risk on [N] additional deals. Competitive pressure has emerged in [N] Negotiation-stage deals totalling $X.XM."

### Forecast Interpretation

[Two to three sentences interpreting the dashboard forecast categories instead of repeating totals. Explain whether the quarter is safe, which forecast category has the most risk-adjusted exposure, and whether commit/best-case movement requires leadership action.]

---

## Section 4 — Top Deals Requiring Immediate Attention

*(Up to 5 deals, each as a sub-section)*

### 1. [Deal Name]

| Account | Amount | Close Date | Risk Level | Stage |
|---------|--------|-----------|-----------|-------|
| [Account] | $[X]K | [Date] | 🔴 High | [Stage] |

**Key concerns:**
- [Plain-language concern, e.g. "No customer engagement in over 3 weeks"]
- [Plain-language concern, e.g. "No executive contact on record for this deal"]
- [If applicable, state slip risk here: "Close date is [N] days away while the deal remains in [Stage] with [risk signal]."]

**Action:** [One specific, executive-level next step tied to the identified risk — e.g. "Call [Account] CEO directly this week — the rep has been ghosted for 34 days and this deal closes Jun 30."]

*(Add only if evidence is weak: "Limited visibility: no recent activity or buyer engagement found.")*

---

### 2. [Deal Name]

| Account | Amount | Close Date | Risk Level | Stage |
|---------|--------|-----------|-----------|-------|
| [Account] | $[X]K | [Date] | 🟡 Medium | [Stage] |

**Key concerns:**
- [Plain-language concern]

**Action:** [Specific next step]

*(Continue for up to 5 deals)*

---

## Section 5 — Rep Activity Exceptions

*(Conditional section. Include only when reps have 2+ inactive deals, material at-risk ARR, or deals tied to current-quarter slippage. If there are no exceptions, omit this section entirely. Do not list every rep.)*

| Rep Name | Open Deals | Deals Needing Attention | Last Engagement | At-Risk Value |
|----------|------------|------------------------|----------------|--------------|
| [Name] | [N] | [N] | [X] days ago | $[X]K |

*Exception rows shaded `#FFF2CC`. Header row shaded `#2E75B6`, white bold text.*

---

## Section 6 — Full Pipeline Risk Table

*(All open opportunities in scope — grouped by risk level, sorted by value descending within each group. Reference only — no narrative.)*

**🔴 HIGH RISK** *(header row: `#C00000` fill, white bold text)*

| ACCOUNT | OWNER | VALUE | STAGE | DAYS STALLED | KEY SIGNAL |
|---------|-------|-------|-------|-------------|-----------|
| [Account] | [Rep] | $[X]K | [Stage] | [N]d | [Signal phrase] |

**🟡 MEDIUM RISK** *(header row: `#ED7D31` fill, white bold text)*

| ACCOUNT | OWNER | VALUE | STAGE | DAYS STALLED | KEY SIGNAL |
|---------|-------|-------|-------|-------------|-----------|
| [Account] | [Rep] | $[X]K | [Stage] | [N]d | [Signal phrase] |

**🟢 LOW RISK** *(header row: `#70AD47` fill, white bold text)*

| ACCOUNT | OWNER | VALUE | STAGE | DAYS STALLED | KEY SIGNAL |
|---------|-------|-------|-------|-------------|-----------|
| [Account] | [Rep] | $[X]K | [Stage] | [N]d | [Signal phrase] |

*Alternate row shading `#F2F7FB` / white within each group.*

---

*Pipeline Intelligence Report — [DATE] | Scope: [QUARTER RANGE] | Prepared by Pipeline Intelligence*
