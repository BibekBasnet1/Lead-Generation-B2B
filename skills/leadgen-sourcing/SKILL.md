---
name: leadgen-sourcing
description: >
  Reusable end-to-end workflow for building outbound B2B lead lists: source companies
  from the right discovery method for the vertical (local-business/Maps data or direct
  B2B company search), filter out noise, match to decision-makers, dedupe against the
  existing CRM, score by ICP fit, and enrich contact details cost-consciously. Invoke
  for any new vertical/campaign — the skill interviews the user to build out an
  ICP if one doesn't already exist for that vertical.
---

# Lead-Gen Sourcing Skill

**Invocation:** `/leadgen-sourcing <vertical>` — e.g. `/leadgen-sourcing commercial bakeries`,
`/leadgen-sourcing dental clinics`, `/leadgen-sourcing warehouse operators`.

This is a demo/reference version of a skill used for real client lead-gen work. Provider
names below (Apollo, Apify, a generic CRM) are illustrative — swap in whatever your stack
actually uses; the *shape* of the workflow is the reusable part.

## Prerequisites

- A B2B contact-enrichment provider with company + people search (e.g. Apollo.io, Clay CLI) via MCP, CLI, or API
- A local-business discovery source for verticals that skew small/local (e.g. an Apify Maps-scraping actor, or any equivalent business-listing API)
- Optionally, a CRM to dedupe against (the skill degrades gracefully without one — it just skips Step 4)
- An `ICP.md` (or equivalent) per vertical — source of truth for target market, personas, disqualifiers. If the vertical isn't covered yet, interview the user to build it (see bottom of this file).

## Step 1 — Confirm scope with the user

Before pulling anything, confirm out loud, one question at a time (don't dump a form):
- Which vertical/campaign
- Geography
- Company size proxy, if any (many B2B databases have no direct "size" field for local SMBs — employee-count bands are a reasonable proxy)
- Any exclusions specific to this campaign (e.g. franchise chains whose contracts are decided at HQ, not site level)

## Step 2 — Pick the right sourcing method, per vertical

Not every vertical lives in a B2B sales database. Decide up front:

- **B2B-database-native verticals** (companies that themselves do outbound sales, have marketing sites optimized for lead capture, etc.) → direct company search on your enrichment provider, tiered narrowest-to-broadest by keyword tag.
- **Local/brick-and-mortar verticals** (bakeries, clinics, repair shops, trades) → B2B databases have thin coverage. Use a local-business/Maps data source instead, and reserve the B2B provider purely for the people-matching step (Step 4).

Watch for **keyword collision noise** either way: generic search terms pull in unrelated categories (equipment suppliers, adjacent trades, franchise/chain locations whose contracts are decided at HQ). Sample the first batch of results before committing to a full pull — if you see obviously off-topic entries, narrow the terms.

## Step 3 — Filter & categorize

Apply a category/keyword filter pass and report the breakdown back to the user rather than silently dropping rows:

- What's clearly in scope
- What's clearly noise (and why — e.g. "equipment supplier", "franchise chain, HQ-level contract")
- What's ambiguous and needs a quick manual glance

If the user wants to **widen scope** to hit a volume target, do it as a deliberate, visible decision (e.g. "include independent operators, exclude only franchise chains") — not by loosening filters silently.

## Step 4 — Match companies to decision-makers

Batch company domains into your enrichment provider's people-search (commonly capped at 100 domains/companies per call). Filter to the vertical's actual decision-maker titles — **this varies a lot by industry**, so don't default to "Owner" or "Director" without checking. A quick web search on "who approves vendor contracts at [vertical] businesses" is often worth doing before finalizing the title list.

Cap company size (e.g. employee count) to exclude enterprise/national accounts that would go to corporate procurement rather than a reachable local decision-maker — unless the user explicitly wants those as a separate, clearly-labeled enterprise tier.

**Filter title-matching for noise too**: broad title search (e.g. "include similar titles") surfaces large adjacent companies via generic terms like "Manager" or "Chef". Tighten to exact titles if a first broad pass returns mostly noise.

## Step 5 — Dedupe against the existing CRM

**Always do this before spending enrichment credits.** Match new leads against existing CRM records by company name, email domain, AND person name (all three — company name alone catches most true duplicates, but not all). Split into:
- Exact duplicates → exclude
- Same company, different person → keep, flagged as "company already in pipeline"
- Fully new → keep

Report backlog counts too — if there's already a large pool of unworked leads in the CRM for this vertical, say so before recommending more sourcing. Volume is rarely the bottleneck; unworked backlog usually is.

## Step 6 — Score by ICP fit (0–100)

A transparent, adjustable model — adapt category weights per vertical, but keep the shape:

| Category | Typical weight | Logic |
|---|---|---|
| Decision-making power | 30–35% | Owner/Director highest; the vertical's actual budget-approver role next; junior/assistant titles lowest |
| Company fit | 20–25% | Exact-match category highest; adjacent/widened-scope category lower |
| Geography | 10–15% | Core service zone highest; secondary zone lower; unknown data treated as neutral, not penalized |
| Contactability | 10–15% | Both LinkedIn + verified domain present highest |
| Buying-signal proxy | 5–10% | Title keywords suggesting active vendor evaluation ("New Business", "Procurement") |

Cold-outbound lists have no real behavioral intent data — weight role/seniority above what a SaaS-style ICP model would (local-services buying decisions concentrate with the owner/manager), and keep the buying-signal category small since title-based proxies are weak.

Tiers: **A (80+)** priority outreach, **B (60–79)** standard sequence, **C (<60)** low priority/nurture.

## Step 7 — Enrich, deliberately

**Confirm scope before enriching — don't enrich the whole list by default.** Ask (conversationally):
1. What's actually missing (email / phone) and how many rows need each
2. Priority order — everything at once, or top tier first
3. Confirm cost-optimized enrichment, not the most expensive all-in-one option

**Cost discipline:**
- Skip rows that already have verified data
- Pick the narrowest/cheapest routine for the job (e.g. an email-only lookup is often 10x cheaper than a combined email+phone routine)
- Check remaining credit balance and estimated total cost *before* running anything non-trivial, and say the actual numbers out loud
- If a provider's enrichment cost is genuinely variable (e.g. a cascading waterfall across multiple data vendors), say so plainly rather than quoting a fixed number

Phone and email are separate opt-in steps. Do-not-call list status is typically not something enrichment providers check — flag it as an external follow-up rather than silently skipping it.

## Step 8 — Deliverables

Keep outputs in clearly named files, one per pipeline stage, never overwriting prior runs:
- `01_raw_<source>_companies.csv`
- `02_filtered_companies.csv`
- `03_contacts_raw.csv`
- `04_deduped_new_leads.csv`
- `05_scored_leads_final.csv`

Update the vertical's `ICP.md` with a dated summary after each run (companies found, contacts scored, tier breakdown) so the next session has the history without re-deriving it.

## Building a new ICP section (for a vertical not yet documented)

Interview the user one question at a time: what they sell for this vertical, who the buyer
is, geography, company size proxy, disqualifiers, buying signals, and any existing clients
in this vertical to use as lookalike seeds. Research industry-specific buying signals via
web search before finalizing — don't carry over assumptions from a different vertical's
persona list (procurement patterns vary a lot by industry). Write the new section into
`ICP.md` following the same structure as existing vertical sections, then proceed to Step 1.
