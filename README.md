# Company Profiler

A local, markdown-first company research tool that runs inside Claude Code. Turns a company name into a decision-ready "brain map" prepared for a conversation with that company's leadership, sourced fact by fact.

No web app. No server. No UI. Markdown files are the only output.

## Why I built this

I built this during a 2026 job search to test whether an AI-native research workflow could produce sharper interview prep than I could write by hand. The answer was yes when the discipline (fact-plus-source, notes-before-map, the exemplar load) was tight, and forgettable when it wasn't.

## What it does

Two files per company:

- **Research notes** at `companies/<slug>-notes.md`. Every non-obvious fact with its source link. The holding pen.
- **Brain map** at `companies/<slug>.md`. The polished read, written from the notes file. The deliverable.

## The framework

A five-stage pipeline modeled on how intelligence analysts work.

| Stage | Job | Produces |
|-------|-----|----------|
| 1. Direction | Resolve which company, and what we are preparing for | Locked target and purpose |
| 2. Collection | Gather non-obvious facts, each tied to a source | Research notes file |
| 3. Processing | Keep only signal, verify every claim traces to a fetched page | Clean, grounded notes |
| 4. Analysis | Turn facts into a read through four operator lenses | Strategic angles |
| 5. Delivery | Write the brain map from the notes, matched to the exemplar | Brain map file |

Three rules carry the quality. **Every fact is tied to a source.** **Only signal gets captured, never obvious noise.** **A finished reference map rides along on every run** so the model matches a real bar, not an empty outline.

## Run it

From inside Claude Code at the repo root:

```
/profile <company> [parent_org=...] [job_url=...] [people="Name 1, Name 2"] [purpose=interview|outreach|partnership]
```

Examples.

```
/profile "Stripe" people="Patrick Collison, John Collison" purpose=outreach

/profile "Some Startup" purpose=outreach
```

The committed [`companies/stripe.md`](companies/stripe.md) was generated with the first command above.

The richer the input, the richer the map. A company name alone still produces a strong core map.

## Repo layout

```
company-profiler/
  README.md
  CLAUDE.md                  # runtime rules loaded on every session
  .claude/commands/
    profile.md               # the /profile slash command
  templates/
    brain-map.md             # empty structure
    research-notes.md        # empty structure
  companies/                 # output, plus the committed reference exemplar
    stripe.md
    stripe-notes.md
```

## The exemplar

[`companies/stripe.md`](companies/stripe.md) is the worked example. The pipeline loads it on every run so generated maps match a real quality bar, not an empty template. An empty template teaches section order. A filled exemplar teaches depth, length per section, how to handle verbatim quotes, and voice.

## Hard non-goals

- No web app, server, database, or UI.
- No React, no HTML build step.
- No multi-user, no auth.
- No paid social API integrations or scraping.
- No padding. Empty sections get dropped.
- No writing the brain map from raw pages. Always from the notes file.
- No unsourced facts in the body. Specific claims are sourced or flagged.

## Author

Built by Rennie Soga. [LinkedIn](https://www.linkedin.com/in/rsoga/).
