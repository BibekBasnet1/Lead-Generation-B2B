# Contributing

Thanks for considering a contribution. This repo is a reference implementation of a
lead-gen skill for Claude Code — the goal is to keep the workflow provider-agnostic and
easy to adapt, not to bolt on every possible integration.

## Good first contributions

- **A new provider adapter** — if you use a different B2B enrichment tool (Clearbit, ZoomInfo,
  Hunter, Lusha, etc.) or a different local-business data source, document the equivalent
  steps and add a short adapter note under `skills/leadgen-sourcing/`.
- **A new vertical's persona/title mapping** — decision-maker titles vary a lot by industry.
  If you've researched who actually approves vendor contracts in a vertical not yet covered
  (healthcare, trades, hospitality, etc.), a short write-up is genuinely useful to others.
- **Improvements to the ICP scoring model** — the weights in `SKILL.md` are a reasonable
  default, not gospel. If you've found a better weighting for a particular type of campaign,
  open an issue with your reasoning first.

## Ground rules

- **No real data, ever.** Every example, sample, or test fixture in this repo must be
  synthetic. Don't add real company names, real people, real emails, or real phone numbers —
  including your own clients' data with names changed. Fully invented data only.
- **No credentials or API keys** in any commit, including in git history. If you accidentally
  commit one, rotate it — don't just remove it in a follow-up commit.
- **Keep the skill provider-agnostic** where reasonably possible. If a step is genuinely
  provider-specific, say so explicitly rather than hard-coding one vendor's naming into the
  general workflow description.

## Process

1. Open an issue describing what you want to change, before writing a large PR — this saves
   both of us time if the shape needs discussion first.
2. Small fixes (typos, broken links, clarifying wording) can go straight to a PR.
3. Keep PRs focused — one adapter or one improvement per PR, not a grab-bag.
