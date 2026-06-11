# Troubleshooting Guide

Common issues and resolutions for the Pipeline Intelligence Agent.

---

## Data Access Issues

### Symptom: Office365 DownloadFile appears stuck after returning FileData
**Cause:** The download already succeeded, but the agent may continue waiting or reasoning over the large base64 payload shown in the tool response.
**Resolution:**
1. Treat any `Status=Success` response with `FileData` as a completed download.
2. Do not inspect, summarize, or echo the base64 payload.
3. Immediately decode the returned payload with `office365_filedata_to_file.py` into `outputs/source-files/`.
4. Continue analysis from the decoded workbook.

### Symptom: available data discovery returns an empty list
**Cause:** The connector may require schema or workspace selection before records are visible.
**Resolution:**
1. Read the connector instructions carefully; they usually contain required setup steps.
2. Check for available schemas or catalogs if the connector exposes them.
3. Re-run discovery after following any setup steps.
4. If still empty, note the gap internally and tell the user: "I couldn't access pipeline records from the connected sales system. Please verify the connection is configured and active."

### Symptom: Fathom discovery returns only headers or no tables
**Cause:** The Fathom CData catalog may expose a data-source label such as `API`, but the actual schema can be different. In the current REST-backed setup, Fathom tables are under the `REST` schema.
**Resolution:**
1. Do not assume the schema from the catalog data-source or driver label.
2. Do not continue to Salesforce-only analysis after an empty `API` schema response.
3. Call `getSchemas` for the `Fathom` catalog.
4. Re-run `getTables` with the returned schema, such as `REST`.
5. Look for the meeting-list view and transcript view before marking Fathom call context unavailable.

### Symptom: Fathom transcript retrieval fails or appears empty
**Cause:** The transcript view requires a `recording_id` input filter even when that input is not listed in the transcript column metadata. The meeting-list transcript field can be null while transcript rows are still available from the separate transcript view.
**Resolution:**
1. Query the Fathom meeting-list view first and collect `recording_id` plus meeting metadata.
2. Query transcript rows separately for each relevant `recording_id`.
3. Tie transcript evidence to active opportunities using account/deal name in the meeting title, participant email/domain, recorder/owner relationship, call URL, and meeting date proximity.
4. Do not mark Fathom call context unavailable based on an unfiltered transcript query or a null transcript field on meeting-list records.

### Symptom: retrieval returns an object-not-found error
**Cause:** Object naming differs from the expected sales-system labels.
**Resolution:**
1. Re-run discovery and look for objects matching opportunities, calls, messages, contacts, and activities.
2. Use the discovered object names internally rather than assumed labels.
3. If no matching table exists, log the gap and skip that signal category.

### Symptom: retrieval returns no rows despite expecting data
**Cause:** Date filtering or timestamp format mismatch.
**Resolution:**
1. Test the connector's supported current-date function internally.
2. Adjust the filter syntax to match the connector's supported date format.
3. Check if timestamps are stored in UTC and adjust the filter window if needed.
4. Temporarily broaden the date window internally to confirm records exist, then restore the intended scope.

---

## Risk Signal Issues

### Symptom: Stage aging cannot be computed
**Cause:** Stage history or equivalent progression data is not available.
**Resolution:** Skip signal #8 (stage aging anomaly), log the gap internally, and use the other eight signals for scoring. Mention the limitation only if it materially affects a deal's risk explanation.

### Symptom: Exec engagement cannot be evaluated
**Cause:** Contact role or contact-title data is not available.
**Resolution:** Skip signal #3 (no exec engagement), note the gap. If Contact table exists but ContactRole does not, use activity descriptions to infer contact involvement.

### Symptom: Competitive mentions not found
**Cause:** Meeting, call, or message records are unavailable, or competitor names are not known.
**Resolution:**
1. If tables are unavailable, log the gap.
2. If tables exist but no competitor list was provided, scan for generic at-risk language instead (signal #6 — negative sentiment).
3. Fall back to negative sentiment scanning silently — do not ask the user for a competitor list.

---

## Output Issues

### Symptom: Too many deals to rank top 5
**Cause:** Large pipeline with many high-risk deals.
**Resolution:** Prioritise by: (1) highest signal count, (2) largest deal value, (3) nearest close date. Take the top 5 from that ranked list.

### Symptom: Rep names not available
**Cause:** Internal owner identifiers are present but display names are not resolvable.
**Resolution:** Group the affected deals under "Unresolved owner" and note internally that owner-name mapping was unavailable. Do not show raw owner identifiers in the report.

---

## Performance Issues

### Symptom: retrieval times out on large pipelines
**Cause:** Too many opportunities are requested in one batch.
**Resolution:**
1. Batch queries into groups of 50 opportunity IDs at a time.
2. Run batches sequentially and merge results before analysis.

### Symptom: meeting or transcript retrieval is very slow
**Cause:** Full-text meeting records are large.
**Resolution:** Limit retrieval to recent meetings tied to the active opportunities and keep the default meeting window to 7 days.
