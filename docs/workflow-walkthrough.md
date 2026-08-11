# Workflow Walkthrough (synthetic example)

This is a narrated, sanitized example run of the `leadgen-sourcing` skill against a fictional
vertical — **wholesale specialty coffee roasters** — so the workflow is visible without any
real client data. Every company, person, email, and phone number below is fake.

## 1. Scoping the campaign

> **User:** "Let's build a lead list for wholesale coffee roasters — Sydney area."
>
> **Claude:** Confirms vertical, geography, and asks whether to cap company size or pull
> broadly first and filter after seeing the data.

## 2. Sourcing

Wholesale coffee roasters are a local/brick-and-mortar-leaning vertical — thin coverage in
B2B sales databases, better coverage in local-business/Maps data. The skill runs a Maps-style
search across a few term variants ("wholesale coffee roaster", "coffee roastery", "coffee
manufacturer") and produces a raw list:

→ [`01_raw_maps_companies.csv`](../demo-data/01_raw_maps_companies.csv) — 15 raw results

## 3. Filtering the noise

Sampling the raw results shows the expected keyword-collision problem: an equipment supplier,
a packaging company, two retail cafes, and two locations of the same franchise chain (whose
cleaning/vendor contracts would be decided at HQ, not per-store) all got swept in by adjacent
category tags.

→ [`02_filtered_companies.csv`](../demo-data/02_filtered_companies.csv) — 9 of 15 kept,
6 excluded with a stated reason each (not silently dropped)

## 4. Matching to decision-makers

The 9 filtered companies' domains get batched into a B2B people-search, filtered to
decision-maker titles for this vertical (Owner/Director/Founder, Operations/General
Manager, Production Manager, Facilities Manager — the same title research step the skill
does for every new vertical, since "who approves a vendor contract" varies a lot by industry).

→ [`03_apollo_contacts_raw.csv`](../demo-data/03_apollo_contacts_raw.csv) — 9 contacts
across 7 of the 9 companies (2 companies had no contact in the B2B database — common for
smaller local businesses, and expected rather than a search-tuning failure)

## 5. Dedupe against the CRM

Matched against a (fictional) existing CRM by company name, email domain, and person name.
In this example, one contact would have come back as an **exact duplicate** and been
excluded; one company had a *different* contact already in the CRM and was kept, flagged
as "same company, new person" rather than treated as a fresh company.

→ [`04_deduped_new_leads.csv`](../demo-data/04_deduped_new_leads.csv) — 9 new leads

## 6. Scoring

Each lead gets an ICP Fit Score (0–100) from decision-making power, company fit, geography,
contactability, and a buying-signal proxy — weighted toward role/seniority since cold-outbound
lists have no real behavioral intent data to lean on.

## 7. Enrichment (cost-aware)

Before enriching, the skill would confirm: email only, or email + phone? Which tier first?
It skips rows that already have a verified email, and picks the cheapest routine that
satisfies what's actually missing — reporting real found/not-found numbers rather than
implying full coverage.

→ [`05_scored_leads_final.csv`](../demo-data/05_scored_leads_final.csv) — the final,
ready-for-outreach list: 4 Tier A, 4 Tier B, 1 Tier C

## What this demonstrates

- **Multi-source sourcing** picked deliberately per vertical, not defaulted to one API
- **Visible filtering**, not silent noise removal — the user sees what got cut and why
- **Cost-conscious enrichment** — dedupe before spend, cheapest-routine-first, real numbers
- **A back-and-forth loop** — the user steers scope (geography, size cutoffs, which tier to
  enrich) at each stage rather than getting a single black-box output
