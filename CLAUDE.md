# Company Profiler. Runtime rules

A local, markdown-first company research tool. Run via `/profile <company>` inside Claude Code.
Two files per company. The brain map is always written from the notes file, never from raw web pages.

The full spec lives in the conversation that built this repo. These are the runtime rules.

---

## The five-stage pipeline

| Stage | Job | Produces |
|-------|-----|----------|
| 1. Direction | Resolve which company, and what we are preparing for | Locked target and purpose |
| 2. Collection | Gather non-obvious facts, each tied to a source | `companies/<slug>-notes.md` |
| 3. Processing | Keep only signal, verify every claim traces to a fetched page | Clean, grounded notes |
| 4. Analysis | Turn facts into a read through four operator lenses | Strategic angles inside notes |
| 5. Delivery | Write the brain map from the notes, matched to the exemplar | `companies/<slug>.md` |

The discipline. **Research first, write second.** The brain map is a view on the notes file.

---

## The fact + source atom (non-negotiable)

Every line in the notes is a claim with its link attached.
- `Claim. [source label](url)` for a confirmed fact from a real page.
- `Claim. [source](url) (aggregator)` for LinkedIn, The Org, Crunchbase, RocketReach.
- `Claim. [Local: <filename>](file:<absolute-path>)` for a fact pulled from a user-supplied local file.
- `Claim. [UNVERIFIED]` when believed but unsourced. These auto-route to Could Not Confirm.
- Verbatim quotes labeled `VERBATIM`, in quote marks, with their source. Never paraphrase a real quote.

A specific claim (a number, date, name, or title) is always sourced, even when it feels well-known. Famous companies are the highest fabrication risk.

---

## The grounding rule (block hallucination)

Before writing the brain map:
- Every factual claim must trace to a page fetched in this run.
- Knowledge from training, not from a fetched source, is excluded or marked `[UNVERIFIED]` and routed to Could Not Confirm.
- Never invent names, dates, numbers, or quotes.

The trap. Asked about a well-known company, the model will write a fluent profile from memory and never search. Refuse to do this.

---

## The signal filter

Capture a fact only if it would change what you say or do in a conversation with this company.

**Keep.** Reveals strategy. Names a proprietary system and what it does. A specific number, date, name, or title. A verbatim quote that shows how someone thinks. Signals what a key person values. A risk, controversy, or public friction. Recent (last ~twelve months).

**Skip.** Generic descriptions of what the company obviously does. Boilerplate mission fluff. Background true of any company in the category. Restating the obvious from the name. Marketing adjectives with no fact underneath.

Two edges to hold:
- **Specific is not obvious.** Specific claims always get sourced, never skipped.
- **Keep all primary-source voice.** Filter the paraphrasable background. Never filter a real quote.

---

## The stop rule

Before writing the map, check coverage.
- Leadership, recent news, differentiators, strategic context covered?
- Each person passed in `people` has a filled block?
- If `job_url` was passed, is The Role section complete?
- If `local_sources` were passed, each one has been read once and its signal pulled into the notes?

Rough budget. ~20 to 35 fetches per fresh company. If coverage is thin after the budget, say so in Could Not Confirm rather than padding.

## Capped single retry on fetch failure

If a `WebFetch` returns a timeout, 403, or empty body, do **one** fallback attempt via `WebSearch` for the same target (article title, press-release headline, bio name). If that fallback still produces no usable content, route the gap to Could Not Confirm with what was tried. One fetch, one fallback, then move on. Do not loop.

## Local sources (compression rule)

If `local_sources` were passed, treat each file the same way as a web page. Read each one **once** during Collection and pull signal into the notes file with `[Local: <filename>]` as the source tag. The notes file is then the only ground truth the brain map writes from. **Never re-read the raw local file in Stage 4 or Stage 5.** This is the same discipline as web pages and it is what keeps the map small enough to write well.

Group local sources under a `### Local sources` subgroup in both the notes Source List and the brain map Sources section, so it is obvious which facts rest on user-provided files. A rich local file (CEO podcast transcript, founder essay) can justify the brain map's Section 5 (High-Signal Deep Dive) on the same bar as any other source.

---

## The exemplar-load rule (biggest quality lever)

On every run, load `companies/stripe.md` and `companies/stripe-notes.md` into context as the worked example.

An empty template teaches section order. A filled exemplar teaches depth, length per section, how to handle verbatim quotes, and voice. The model matches quality it can see far better than quality described in a schema. This is not optional.

If the exemplar files are missing, stop and tell the user.

---

## Hypotheses, not facts

Lens 3 (Where I contribute) and Lens 4 (Automation opportunities) are operator hypotheses about the company's pain. They are not facts about the company. Mark them as hypotheses to test in conversation. Tie each to a specific fact from the notes so the hypothesis is grounded.

---

## Style rules

Brain maps and notes should read like a senior operator wrote them.

- **TLDR at top, details below.** Always.
- **Lead with the answer.** Analysis states the read first, then the reasoning.
- **Short paragraphs.** Two or three sentences. No walls of text.
- **Scannable things are lists. Analysis is prose.** Leadership, lineups, logistics, sources are bullets or tables. Angles and differentiators are tight paragraphs.
- **No em dashes, semicolons, or colons in prose.** Use commas, periods, and parentheses.
- **Bold-label-period pattern for label-value lines.** Example. `**Location.** Hybrid in LA.`
- **No filler adjectives.** Avoid revolutionary, seamless, robust, leverage, crucial, keen, delve.
- **Consistent section order.** Identical across every company. Drop empty sections, do not pad.
- **Hyperlink sources.** Grouped, labeled, primary versus aggregator flagged.

---

## Source rules

1. **Primary first.** Company site, newsroom, blog, earnings calls, launch materials, direct interviews.
2. **Press.** Recent coverage.
3. **People.** Personal sites, talks, podcasts, posts, GitHub.
4. **Aggregators last.** LinkedIn, The Org, Crunchbase, RocketReach. Useful for arc and titles, but they lag and repeat each other. Tag anything that rests only on these.

Do not scrape X or LinkedIn. They block fetches and break ToS. Build around what is publicly accessible.

---

## Hard non-goals

- No web app, server, database, or UI.
- No React, no HTML build step.
- No multi-user or auth.
- No paid social API integrations or scraping.
- No padding. Empty sections get dropped.
- No writing the brain map from raw pages. Always from the notes file.
- No unsourced facts in the body. Specific claims are always tied to a source or flagged.
