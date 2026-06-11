# Research Process — Suggested Sequencing

This is a 30-minute end-to-end run. The order matters because each step feeds the next: firmographics give you the right Apollo filter, signals give you the synthesis material, and synthesis is what makes the outreach assets specific instead of generic.

## Step 1 — Firmographics & snapshot (≈5 min)

1. Apollo.io → company profile (snapshot fields, founding year, revenue, NAICS).
2. Apify LinkedIn scraper → validate + enrich, capture canonical LinkedIn URL.
3. Exa → fetch homepage + about page, summarize business overview and model.

Output: Section 1 (Account Snapshot) is fully filled.

## Step 2 — Research summary & signals (≈10 min)

1. Exa → recent news, funding, M&A, awards, strategic announcements.
2. Apify LinkedIn Jobs scraper → count open roles by department; flag hiring spikes.
3. Apollo.io → check hiring signals and recently added contacts.

Output: Section 2 (Market & Tech Intelligence) and Section 3 (Insights & Buying Signals)

## Step 3 — Tech stack & org mapping (≈10 min)

1. Apify → scrape job descriptions for tool mentions.
2. Exa → engineering blogs and case studies for stack signals.
3. Apollo.io → filter contacts by seniority + function; extract 3–5 personas.
4. Apify LinkedIn people scraper → validate and expand the stakeholder list.

Output: Section 2 (Market & Tech Intelligence) and Section 4 (Stakeholders)

## Step 4 — Synthesis & personalization (≈5 min)

1. Synthesize signals → pain hypotheses, ICP score, opportunity hypotheses (Sections 3 and 5).
2. Map pain → product capability → outcome (Section 5).
3. Draft outreach assets: hooks, LinkedIn message, call opener, sequence (Section 6).
4. Compile competitive context and next best actions (Sections 7 and 8).
5. CData Connect AI → if connected, run the CRM lookup from `references/crm_lookup.md`.
6. Compile sources used during research (Section 9).

Output: every section filled; the content is ready to hand to the `docx` skill, which builds the final Word document.

---

## Why this order

If you start with synthesis, you'll write generic-sounding hypotheses because you don't yet have the specific triggers to anchor them. If you start with outreach, you'll repeat the same hook across every angle because you haven't separated trigger-based from pain-based yet. The sequence above forces specificity by the time you get to Sections 5–8.
