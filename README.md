# Claude Lead-Gen Toolkit

A reusable **Claude Code Skill** for building B2B outbound lead lists through a conversational, human-in-the-loop workflow — combining local-business discovery (Google Maps / Apify) with B2B contact enrichment (Apollo.io, Clay CLI, or an equivalent provider), plus a scoring model to prioritize outreach.

This repo is a public, sanitized demo of a workflow I use for real client work. All data in `demo-data/` is **synthetic** — fake companies, fake people, fake emails and phone numbers — generated to illustrate the shape of the pipeline without exposing any real client's information.

![Pipeline data flow: external APIs source and match companies and contacts, the skill's own logic filters, dedupes, and scores them, a CRM check keeps enrichment spend to net-new contacts, then a final deliberate enrichment call goes back out to Apollo.io or Clay CLI.](docs/assets/pipeline-diagram.svg)

*Three lanes, six steps: external APIs handle sourcing/matching/enrichment, the skill's own logic filters and scores, and a CRM check sits between them so enrichment credits only get spent on net-new contacts.*

## Why this exists

Most "lead gen with AI" demos are a single API call. The interesting part is everything around it:

- **Sourcing isn't one API.** B2B databases like Apollo have poor coverage of small local businesses. Google Maps / local-business data sources fill that gap for anything that isn't itself a "sales-outbound-shaped" company (a bakery, a strata manager, a clinic). The skill picks the right source per vertical instead of defaulting to one API.
- **The list needs cleaning before it needs enriching.** Keyword-based sourcing pulls in noise (adjacent trades, franchise chains, equipment suppliers). The workflow includes an explicit filter/dedupe pass, with the noise categories reported back to the user rather than silently dropped.
- **Cost-aware by default.** Enrichment (email/phone reveal) costs money per contact. The skill checks existing CRM records before spending credits, picks the cheapest enrichment routine that satisfies the ask, and reports real cost before running anything non-trivial.
- **Back-and-forth, not one-shot.** The user steers scope at each stage — vertical, geography, company-size cutoffs, which enrichment tier to spend credits on — rather than the skill guessing all of it up front and delivering a black-box CSV.

## What's in here

```
claude-leadgen-toolkit/
├── skills/
│   └── leadgen-sourcing/
│       └── SKILL.md          # the actual Claude Code skill definition
├── demo-data/                # synthetic example outputs at each pipeline stage
│   ├── 01_raw_maps_companies.csv
│   ├── 02_filtered_companies.csv
│   ├── 03_apollo_contacts_raw.csv
│   ├── 04_deduped_new_leads.csv
│   └── 05_scored_leads_final.csv
├── docs/
│   └── workflow-walkthrough.md   # a sanitized, narrated example run
└── README.md
```

## The pipeline, in short

1. **Source companies** — pick the right discovery method for the vertical (local-business/Maps data for brick-and-mortar SMBs, direct Apollo company search for anything that behaves like a normal B2B market).
2. **Filter & categorize** — drop noise (adjacent trades, franchises, equipment vendors), tag what's left by tier/fit.
3. **Match to decision-makers** — Apollo people search by company domain, filtered to the right titles for that vertical (this differs a lot by industry — don't assume "Owner" is always the buyer).
4. **Dedupe against the existing CRM** — never re-spend enrichment credits on a contact you already have.
5. **Score** — a transparent, adjustable ICP fit model (decision-making power, company fit, geography, contactability, buying-signal proxy).
6. **Enrich, deliberately** — email/phone reveal only after scope and cost are confirmed; cheapest routine that satisfies the ask; skip already-verified rows.

See [`docs/workflow-walkthrough.md`](docs/workflow-walkthrough.md) for a full narrated run against the synthetic demo data.

## Example run

What it actually looks like to invoke `/leadgen-sourcing <vertical>` — sanitized, using the synthetic **wholesale coffee roasters, Sydney** example in `demo-data/`. Every name, email, and phone number below is fake.

> **You:** Let's build a lead list for wholesale coffee roasters — Sydney area.
>
> **Skill:** Before I pull anything — cap company size, or pull broadly and filter after seeing the data? Anything to exclude up front, like franchise chains?
>
> **You:** Pull broadly, exclude franchise chains.
>
> **Skill:** This vertical skews local/brick-and-mortar, so I'll source from Maps data rather than Apollo — thin coverage of independent roasters there. Pulled 15 companies. Filtering now: 6 excluded (an equipment supplier, a packaging company, 2 retail cafes, 2 franchise locations decided at HQ), 9 kept.
>
> Matching the 9 to decision-makers — Owner/Director/Founder/Ops Manager/Production Manager/Facilities Manager for this vertical. Got 9 contacts across 7 of the 9 companies (2 had no match in the B2B database).
>
> Checked against your CRM: 1 exact duplicate excluded, 1 new contact at a company you already track (kept, flagged). 9 net-new leads, scored.
>
> **You:** What's the enrichment cost look like?
>
> **Skill:** 9 rows need email, 5 also need phone. Email-only lookup is the cheaper route — running that first. Want phone too, or top tier only?
>
> **You:** Top tier only.
>
> **Skill:** Done — 4 Tier A rows got phone lookups, the rest stayed email-only.

**Output** (`05_scored_leads_final.csv`, top rows):

| Name | Title | Company | ICP Fit | Tier |
|---|---|---|---|---|
| Jordan Ashby | Owner | Northbridge Roasting Co | 92 | A |
| Priya Kapoor | Director | Riverside Bean Works | 90 | A |
| Aisha Bello | Managing Director | Highland Roasting Collective | 88 | A |
| Renee Park | Facilities Manager | Ironbark Coffee Manufacturing | 79 | B |

Full 9-row output, plus every intermediate CSV (raw sourcing → filtered → matched → deduped), is in [`demo-data/`](demo-data/).

## Using the skill

Drop `skills/leadgen-sourcing/SKILL.md` into your project's `.claude/skills/` directory (or your Claude Code skills path) and invoke it with `/leadgen-sourcing <vertical>`. It expects:

- A Maps/local-business data source (e.g. an Apify actor, or any equivalent business-listing API)
- Apollo.io MCP access, the Clay CLI, or an equivalent B2B enrichment provider (adapt the "B2B enrichment" step to whichever your stack uses)
- A CRM to dedupe against (optional — the skill degrades gracefully without one)

None of these credentials are included here. See `docs/workflow-walkthrough.md` for what the tool calls look like without needing real access to run the read-through.

## Contributing

PRs welcome — especially:
- Adapters for other B2B enrichment providers (Clearbit, ZoomInfo, Hunter, etc.)
- Adapters for other local-business data sources
- Additional vertical-specific persona/title mappings (what "decision-maker" means differs a lot between, say, medical clinics and warehouses)
- Improvements to the ICP scoring model

Open an issue first for anything beyond a small fix, so we can agree on shape before you put in the work.

## License

MIT — see [LICENSE](LICENSE).
