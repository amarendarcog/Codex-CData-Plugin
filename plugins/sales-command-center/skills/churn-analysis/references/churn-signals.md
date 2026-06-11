# Churn Signal Definitions

Detailed definitions for each of the five analysis dimensions. Read this file when executing Step 3 of the skill.

---

## Dimension 1: Overall Churn & Renewal Rate

### Key metrics to compute

| Metric | Formula | Source Fields |
|--------|---------|---------------|
| Total ARR up for renewal | SUM(Amount) where Type = 'Renewal' AND CloseDate in period | Opportunity.Amount |
| Renewed ARR | SUM(Amount) where Type = 'Renewal' AND StageName = 'Closed Won' | Opportunity.Amount |
| Churned ARR | SUM(Amount) where Type = 'Renewal' AND StageName = 'Closed Lost' | Opportunity.Amount |
| Expansion opportunity loss | SUM(Amount) where existing-customer expansion, upsell, or cross-sell opportunities are Closed Lost | Opportunity.Amount |
| Gross Renewal Rate (GRR) | Renewed ARR / Total ARR up for renewal | Computed |
| Net Revenue Retention (NRR) | (Renewed ARR + Expansion ARR) / Starting ARR | Computed (if expansion data available) |

### Churn health thresholds

| Rating | Gross Renewal Rate | Churned ARR % of Total |
|--------|--------------------|------------------------|
| 🔴 Critical | Below 75% | Above 15% |
| 🟡 At Risk | 75% – 85% | 10% – 15% |
| 🟢 Healthy | Above 85% | Below 10% |

### Fallback: if Type field is unavailable

Filter by opportunity name patterns: opportunities named with "Renewal", "Renew", or "RNW" are renewal-type. Opportunities named with "Expansion", "Upsell", "Cross-sell", or similar growth motions are expansion-type and must stay separate from churned ARR and GRR. New business opportunities are the remainder. Note this fallback in the output.

---

## Dimension 2: Loss Reasons

### Fields to check (in priority order)

1. `Loss_Reason__c` — most common custom field name
2. `Churn_Reason__c`
3. `Closed_Lost_Reason__c`
4. `Primary_Loss_Reason__c`
5. `Description` or `Notes` — parse free text if structured fields are blank

Run `getColumns` on the Opportunity table to find which exists.

### Standard loss reason taxonomy

Map whatever values exist in the data to these standard categories for cleaner reporting:

| Standard Category | Example Raw Values |
|-------------------|--------------------|
| Price / Budget | "Too expensive", "Budget cut", "Cost", "Price" |
| Competitor | "Went with competitor", "Chose alternative", "Lost to [name]" |
| Product fit | "Missing features", "Doesn't meet needs", "Functionality gap" |
| Champion left | "Contact left company", "Sponsor departed", "Internal change" |
| No decision | "Stalled", "No budget approved", "Project cancelled" |
| Service / Support | "Poor support", "Implementation issues", "Onboarding failed" |
| Contract terms | "Terms unacceptable", "Legal issue", "Compliance" |
| Unknown / blank | Empty, null, "Other" |

### Internal data quality flag

If more than 20% of churned opportunity records have a blank or "Other" loss reason, account for that internally when interpreting loss reasons. Do not add a standalone data-completeness note to the report; mention the limitation only if it materially changes the executive conclusion.

---

## Dimension 3: Churned ARR by Product Line

### Join pattern

```
Opportunity → OpportunityLineItem → Product2
```

Group by `Product2.Family` (product family) or `Product2.Name` (specific product).

If `OpportunityLineItem` is unavailable, fall back to:
- `Opportunity.Product_Line__c` or `Opportunity.Product_Family__c` (custom fields)
- `Opportunity.RecordType.Name` (if record types map to products)

### Metrics per product line

| Metric | Definition |
|--------|-----------|
| Churned ARR | SUM(Amount) of Closed Lost renewals for this product |
| Churn Rate | Churned ARR / Total ARR up for renewal for this product |
| Deal Count | COUNT of churned opportunities for this product |
| Avg Deal Size | Churned ARR / Deal Count |

### Flags

- Product churn rate above the portfolio average → flag as "above-average churn"
- Product accounting for more than 30% of total churned ARR → flag as "concentration risk"

---

## Dimension 4: Renewal Cohort Trends

### Grouping

Group closed opportunities by quarter (or month if the user requests it) using `CloseDate`.

Compute GRR per period:
```
GRR[period] = Renewed ARR[period] / (Renewed ARR[period] + Churned ARR[period])
```

### Trend signals

| Signal | Definition |
|--------|-----------|
| Declining trend | GRR has fallen for 2 or more consecutive periods |
| Single-period spike | GRR dropped below 80% in one period only |
| Improving trend | GRR has risen for 2 or more consecutive periods |
| Stable | GRR variance less than 5 percentage points across all periods |

Flag: any period where GRR falls below 80% as a concern regardless of trend direction.

---

## Dimension 5: Cancellation Pattern Analysis

### Mid-term vs at-renewal

- **At renewal:** CloseDate is within 30 days of the contract end date (use `Contract_End_Date__c` or `Renewal_Date__c` if available)
- **Mid-term:** CloseDate is more than 30 days before the contract end date

### Time-to-cancellation

Compute days from `Contract_Start_Date__c` (or `CreatedDate` as fallback) to cancellation `CloseDate`. Group into buckets:
- Early (0–90 days): likely onboarding or fit failure
- Mid-contract (91–270 days): engagement or value realisation failure
- Late (271+ days): competitive displacement or price sensitivity

### Segmentation

Break cancellations down only by populated fields that exist in Salesforce:
- Explicit account segment, tier, customer type, or other customer grouping fields
- Industry when available
- Geography when available

Do not invent named account-size groupings from unrelated fields. Use account-size labels only if Salesforce contains those exact values or the user provides a mapping rule.

### Pattern flags

| Pattern | Flag |
|---------|------|
| More than 40% of cancellations are early (0–90 days) | Onboarding risk — flag for CS team |
| One available customer grouping accounts for more than 50% of churned ARR | Concentration risk |
| Cancellations concentrated in one industry | Vertical-specific issue |
| Rising mid-term cancellations over last 2 periods | Engagement or product value issue |

---

## Segmentation Cohorts (Step 3)

### Behavioral cohort signals

**Discount-led vs. full-price cohort**

A customer is "discount-led" if their original or most recent closed opportunity had a non-zero discount applied (`Discount__c > 0` or `Discount_Percentage__c > 0`). Full-price customers have no discount on record.

Compute churn rate for each cohort:
```
Churn rate = Churned ARR (cohort) / Total ARR up for renewal (cohort)
```
Flag if discount-led churn rate ≥ 1.5× full-price churn rate → **discount dependency risk**.

**Support friction cohort**

A customer has "pre-churn support friction" if they had one or more open or escalated Cases linked to their Account within 90 days before their churn CloseDate. Compare churn rate of this cohort vs. accounts with no pre-churn cases.

Flag if support-friction cohort churn rate ≥ 1.5× no-friction cohort → **service-induced churn signal**.

**Subscription pause/skip cohort**

A customer has "subscription stress" if they have `Pause_Count__c > 0` or `Skip_Count__c > 0` (or equivalent custom fields) in the period before churn. Compare churn rate vs. customers with no pauses.

**First-renewal vs. multi-renewal cohort**

First-renewal: opportunities where this is the customer's first renewal after their initial contract (use contract start date + renewal date to infer). Multi-renewal: 2+ renewals completed.

Flag if first-renewal churn rate ≥ 1.5× multi-renewal → **early lifecycle retention risk**.

### Firmographic cohort thresholds

| Flag | Condition |
|---|---|
| Customer grouping concentration risk | One available customer grouping accounts for >50% of churned ARR |
| Vertical concentration risk | One industry accounts for >40% of churned ARR |
| Geographic concentration risk | One region/country accounts for >40% of churned ARR |
| Elevated cohort | Any cohort with churn rate >1.5× portfolio average |

---

## Root-Cause Analysis (Step 5)

### Cross-reference matrix

Use this matrix to connect evidence from multiple dimensions to a hypothesis:

| Hypothesis | Primary evidence source | Corroborating signals |
|---|---|---|
| Onboarding / fit failure | Dimension 5 (Early 0–90d bucket >40%) | Support friction cohort elevated; "Product fit" in loss reasons; customer-call or account-team message evidence mentions onboarding, setup, migration, or implementation blockers |
| Value realisation failure | Dimension 5 (Mid 91–270d rising) | Subscription pause/skip cohort elevated; Dimension 4 declining trend; customer-call or account-team message evidence mentions low adoption, weak ROI, missing success criteria, or lack of executive value recognition |
| Competitive / price | Dimension 2 (Competitor + Price/Budget >30%) | Discount-led cohort 1.5× churn rate; Late 271+d bucket dominant; customer-call or account-team message evidence mentions competitor evaluation, internal substitutes, budget release, procurement objections, or pricing mismatch |
| Product-specific failure | Dimension 3 (concentration risk flag) | One product >30% churned ARR; above-average product churn rate |
| Customer grouping / vertical concentration | Dimension 5 (available grouping breakdown) | Available grouping cohort >50% churned ARR; elevated cohort flag |
| Process / security / procurement blocker | Customer-call or account-team message evidence | Renewal data shows paused, lost, or delayed opportunities; loss reasons understate legal, security, IT, procurement, budget timing, or vendor onboarding blockers |

### Severity classification

| Severity | Definition |
|---|---|
| **High** | Confirmed hypothesis accounts for >30% of total churned ARR in the period |
| **Medium** | Confirmed hypothesis accounts for 15–30% of total churned ARR |
| **Low** | Suspected but data is incomplete, or accounts for <15% of churned ARR |

A hypothesis is **confirmed** when primary evidence and at least one corroborating signal both point to the same conclusion. It is **suspected** when primary evidence is present but corroborating signals are unavailable due to data gaps. Customer-call and account-team message evidence can corroborate root causes, but ARR impact and renewal status should come from renewal/account records unless a structured source is reliably tied to the account and period.

### Recommendation linkage

Every recommendation in the report must:
1. Name the hypothesis it addresses (e.g. "Hypothesis 1: Onboarding failure — High severity")
2. Cite the specific metric evidence (e.g. "47% of churned deals are in the Early 0–90 day bucket, representing $320K churned ARR")
3. Specify a concrete action with implied owner and timeframe
