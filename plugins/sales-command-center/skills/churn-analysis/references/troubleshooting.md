# Troubleshooting Guide — Churn Analysis Agent

---

## Data Discovery Issues

### getTables() returns no results
**Resolution:** Call `getInstructions()` first and follow any required setup steps (schema selection, workspace selection). Then retry `getTables()`. If still empty, inform the user that CData is not returning any tables and ask them to verify the connector configuration.

### Customer-call or account-team message sources are not visible
**Resolution:**
1. Confirm that the CData configuration includes the call and message connectors.
2. Retry discovery through CData only; do not connect directly to the underlying systems.
3. If only CRM data is available, complete the quantitative churn analysis and mention the missing corroborating context only in affected sections where it materially changes interpretation.

### Fathom discovery returns no tables
**Cause:** The Fathom CData catalog may expose a data-source label such as `API`, but the usable schema can be different. In the current REST-backed setup, Fathom tables are under the `REST` schema.
**Resolution:**
1. Do not assume the schema from the catalog's data-source or driver label.
2. Do not continue to CRM-only analysis after an empty `API` schema response.
3. Call `getSchemas` for the `Fathom` catalog.
4. Re-run `getTables` with the returned schema, such as `REST`.
5. Look for the meeting-list view and transcript view before deciding customer-call evidence is unavailable.

### Fathom transcript query fails or returns no rows
**Cause:** The transcript view requires a `recording_id` input filter even if that input is not listed in the transcript column metadata. Meeting-list records may also have a null transcript field even when transcript rows are available separately.
**Resolution:**
1. Query the Fathom meeting-list view first and collect `recording_id` plus meeting metadata.
2. Query the transcript view separately for each relevant `recording_id`.
3. Match transcript rows back to churned accounts using meeting title, participant email/domain, recorder/owner, call URL, and meeting date proximity.
4. Do not mark customer-call evidence unavailable based on an unfiltered transcript query or a null transcript field in meeting-list records.

### Call or message records cannot be matched to accounts
**Resolution:**
1. Look for account names, domains, participant emails, opportunity names, or owner names that can link conversation evidence to CRM records.
2. Use conservative matching only; do not attribute a quote or message to an account unless the match is clear.
3. If matching is ambiguous, use the evidence only as an aggregate theme and note the limitation in business language.

### Cannot find renewal vs new business distinction
**Cause:** The `Type` field on Opportunity may not be populated, or values differ from standard Salesforce picklist.
**Resolution:**
1. Inspect the distinct values present in `Type`.
2. If populated, use the values that mean renewal, such as "Renewal", "Add-On", or "Existing Business".
3. If `Type` is blank on most records, fall back to opportunity name pattern matching for renewal-like names.
4. Mention the fallback in the affected section only if it materially changes interpretation, using business language.

### Loss reason field not found
**Resolution:**
1. Run `getColumns('Opportunity')` and search for fields containing "reason", "churn", "loss", or "cancel".
2. Use the first matching field found.
3. If none exist, check the `Description` field for free-text loss reason notes.
4. If no loss reason data is available at all, skip Dimension 2 and note the gap.

---

## Query Issues

### Office365 DownloadFile appears stuck after returning FileData
**Cause:** The download already succeeded, but the agent may continue waiting or reasoning over the large base64 payload shown in the tool response.
**Resolution:**
1. Treat any `Status=Success` response with `FileData` as a completed download.
2. Do not inspect, summarize, or echo the base64 payload.
3. Immediately decode the returned payload with `office365_filedata_to_file.py` into `outputs/source-files/`.
4. Continue analysis from the decoded workbook.

### Date arithmetic fails
**Cause:** CData driver may not support the date functions used by the current retrieval pattern.
**Resolution:** Test which current-date function the connector supports internally. If date arithmetic is unreliable, compute the date boundary before retrieval and use that resolved date value.

### QUARTER() or YEAR() functions unsupported
**Resolution:** Fetch `CloseDate` as a raw string and compute quarter groupings in post-processing after receiving the results.

### Combining Opportunity and Account records fails
**Cause:** CData may not support combining multiple objects in a single retrieval depending on configuration.
**Resolution:**
1. Retrieve Opportunity records first to collect `AccountId` values.
2. Retrieve matching Account records separately using those account identifiers.
3. Merge the results in memory before analysis.

### OpportunityLineItem not available
**Resolution:** Fall back to product fields directly on Opportunity (`Product_Line__c`, `Product_Family__c`, `RecordType.Name`). Run `getColumns('Opportunity')` to find the best available alternative. Mention the fallback in the product analysis only if it materially changes interpretation.

---

## Analysis Issues

### GRR cannot be computed — no renewals found
**Cause:** Either no renewals exist in the period, or renewal opportunities are not distinguishable from new business.
**Resolution:** Widen the date range internally only if the user did not explicitly request a fixed period. If still empty, complete the report with the available churn/loss evidence and state the limitation in business language. Ask how renewals are tracked only if GRR is the requested deliverable and cannot be computed from any available records.

### Churned ARR appears negative or zero
**Cause:** Amount field may be null on closed-lost opportunities, or opportunities are tracked differently (e.g., using a separate churn object).
**Resolution:** Use null-safe handling for `Amount`. Search discovered objects and fields for reliable churned ARR alternatives before asking the user. Ask whether churned ARR is stored elsewhere only if no usable revenue-impact source exists and the report cannot provide a meaningful ARR view.

### Product line breakdown shows only one product
**Cause:** Product or product family field is sparsely populated.
**Resolution:** Note the data quality issue. Use whatever grouping exists and flag that product-line data quality may limit accuracy.

### Conversation evidence conflicts with CRM loss reason
**Cause:** A loss reason may have been selected quickly in CRM while customer calls or account-team messages describe a more specific blocker.
**Resolution:** Keep CRM as the metric source of record, but use customer-call and account-team message evidence to refine the root-cause narrative. State the mismatch in business language, such as "Recorded cancellation reasons understate security and procurement blockers raised in customer conversations."

---

## Output Issues

### Too many loss reasons to display meaningfully
**Resolution:** Show the top 10 by churned ARR. Group remaining values into "Other" with a combined ARR and count. State clearly how many reasons were consolidated.

### Cohort trend has fewer than 3 periods
**Resolution:** Still produce the trend table with what is available. Note: "Trend analysis is limited — fewer than 3 periods of data available. Extend the date range for a more meaningful trend view."
