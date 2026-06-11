# Tool Extraction Guide

Read this when you're actively pulling data for a brief. It maps every section of `assets/account_brief_template.md` to the tool you should reach for first, second, and third — and explains *why*, so you can adapt when a tool is missing.

## Tool Responsibility Model

| Tool | Primary Role | Why this is the right shape of tool | Access via |
|------|--------------|--------------------------------------|------------|
| **Apollo.io** | Structured B2B data — company firmographics + contact records | Pre-cleaned, filterable. Best for snapshot fields and stakeholder lists. | CData — catalog `Apollo` |
| **Exa** | Unstructured intelligence — news, narratives, engineering blogs | Surfaces the *story* — funding rounds, leadership changes, strategic posture. | CData — catalog `Exa` |
| **Salesforce** | CRM — accounts, opportunities, contacts, activity | Tells you what your own company already knows about this account. | CData catalog `Salesforce1` |
| **Apify** | Deep scraping — LinkedIn profiles, job postings, career pages | Captures detail that doesn't exist in databases (job description text, recent posts, exact tenure). | CData — catalog `Apify` |

## Non-obvious filters:
- Apollo OrganizationEnrichment requires WHERE [domain] = 'company.com' (not name).
- Apollo OrganizationJobPostings requires WHERE [OrganizationId]. Get the ID from OrganizationEnrichment first.
- Exa search requires WHERE [query] = 'your search string'.
- Apollo People title filters must be exact match strings; LIKE on organization name causes pagination errors.

If a tool is unavailable, fall back to the next-best alternative and lower the section's confidence rating accordingly.

---

## Section 1 — Account Snapshot

1. **Apollo.io** (primary): pull the company profile. Fields available: size, industry, HQ, domain, employee count, founding year, revenue band.
2. **Apify** (validate): LinkedIn company scraper — confirm employee count, HQ city, official description, canonical LinkedIn URL.
3. **Exa** (fill gaps): fetch homepage + about page. Useful for revenue mentions, business-model nuance, NAICS code if not in Apollo.

---

## Section 2 — Market & Tech Intelligence

**Research Summary**
1. **Exa** (primary). Run these queries:
   - `"[company] news 2024"`
   - `"[company] funding"`
   - `"[company] acquisition"`
   - `"[company] award"`
   Extract from press releases, TechCrunch, BusinessWire, PR Newswire, and the company blog.
2. **Apollo.io**: funding rounds and M&A signals are sometimes in the company record — quick check.

**Technographics**
1. **Apify** (primary): scrape job descriptions. Tool names show up explicitly — "Salesforce", "Snowflake", "dbt", "AWS", etc.
2. **Exa**: search `"[company] tech stack"`, `"[company] engineering blog"`, `"[company] case study [tool]"`.
3. **Apollo.io** (supplementary): pull technographic fields if available.

---

## Section 3 — Insights & Buying Signals

**Insights** — This is an LLM synthesis step, not a retrieval step. Once you have triggers, tech gaps, stage signals, and hiring patterns from sections 1–2, generate reasoned pain hypotheses and opportunity hypotheses using this chain:

> trigger → operational problem it creates → your solution → outcome

Write conclusions only — no reasoning chains in the document.

**Buying Signals**
1. **Exa** (primary): `"[company] funding"`, `"[company] new hire"`, `"[company] expansion"`, `"[company] partnership"`.
2. **Apify**: LinkedIn Jobs scraper — count open roles by department, detect hiring spikes.
3. **Apollo.io**: hiring signals plus recently added contacts as an organic-growth proxy.

---

## Section 4 — Stakeholders

1. **Apollo.io** (primary): filter contacts by department, seniority (VP / Director / Head), and title keywords. Pull name, title, email, LinkedIn URL.
2. **Apify**: LinkedIn people scraper — expand the list beyond Apollo, validate roles and tenure.
3. **Exa** (targeted): `"Head of [function] at [company]"` or `"[company] [VP/Director] [department]"`.

Aim for 3–5 stakeholders across decision makers, influencers, and champions.

---

## Section 5 — Qualification & Value Fit

**ICP Fit & Scoring**
1. **Apollo.io**: company size, industry, segment — compare to your ICP criteria.
2. **Exa**: growth trajectory, funding stage — informs tier and deal potential.
3. **LLM synthesis**: combine signals → ICP score, tier, deal-size estimate.

**Value Proposition Mapping** — Built from pain hypotheses (Section 3) plus your internal product knowledge:

> "If [pain X] → your product solves via [capability Y] → outcome Z"

Pull use cases from similar customers at equivalent stage and size.

Write conclusions only — no reasoning chains in the document.

---

## Section 6 — Outreach Strategy

1. **Exa**: news + announcements → personalization hooks.
2. **Apify**: persona context (role, team size, recent hires).
3. **Apollo.io**: contact-level data (title, function, seniority).
4. **LLM synthesis**: feed all signals → email hook, LinkedIn message, call opener, three messaging angles, day-wise sequence.

Keep all copy tight — hooks, angles, and openers should be immediately usable, not drafts that need trimming.

---

## Section 7 — Competitive Context

1. **Exa**: `"[company] uses [category]"`, `"[company] vs [competitor]"`, `"[company] replaced [tool]"`.
2. **Apify**: job descriptions mentioning competitor tools — infer current stack and displacement opportunity.
3. **LLM synthesis**: reason through positioning and the differentiated entry point.

Write conclusions only — state the implication for this account, not background on the market.

---

## Section 8 — Next Best Actions

Synthesize from ICP score (Section 5), buying signals (Section 3), and the stakeholder map (Section 4). Prioritize the contact with the highest title, most aligned pain, and most recent signal. Include a timing recommendation based on signal recency.

Write conclusions only — action and rationale, nothing else.

---

## Section 9 — Sources

Log every Exa URL, Apollo record, LinkedIn profile, and Apify page used. Include direct links to funding announcements, job postings, and leadership pages.

---

## Research Quality Guidelines

**Specific over generic**
- Bad: "They're in growth stage."
- Good: "Raised $80M Series C 4 months ago, now at ~300 employees."

**Triggers over background**
- Bad: "They use Salesforce."
- Good: "Posting 12 RevOps roles → likely expanding sales motion, Salesforce admin listed as required."

**Reasoned pain over stated facts**
- Bad: "They have a data team."
- Good: "Data team doubled in 6 months + Redshift in job descriptions → likely hitting warehouse scaling limits."

## Confidence Rating

After completing the brief, rate overall research confidence and put it in the doc header:

- **High** — Apollo data confirmed, 3+ personas extracted, triggers verified via Exa/Apify, tech stack identified, LinkedIn and website scraped.
- **Medium** — Some gaps (tech stack inferred, 1–2 personas missing, limited news coverage).
- **Low** — Limited Apollo data, no public news, personas unknown, website thin.