---
description: Profile a company into a sourced research notes file and a brain map.
argument-hint: <company> [parent_org=...] [job_url=...] [people="A, B"] [purpose=interview|outreach|partnership] [local_sources="path1,path2"]
allowed-tools: WebSearch, WebFetch, Read, Write, Edit, Bash, Glob
---

# /profile — five-stage company research pipeline

You are running the Company Profiler. Read `CLAUDE.md` at the repo root before
anything else. Then load the committed exemplar (`companies/stripe.md`
and `companies/stripe-notes.md`) into context as the worked example.
This is not optional. The model matches quality it can see.

## Inputs
Parse `$ARGUMENTS` into:
- **company** (required, positional).
- **parent_org** (optional). Helps disambiguate name collisions.
- **job_url** (optional). Turns on Section 3 (The Role) and Section 7 (What NOT to Do).
- **people** (optional, comma-separated). Forces a full block for each in Section 2.
- **purpose** (optional, default `outreach`). One of `interview`, `outreach`, `partnership`. Decides which optional sections fire and tilts the analysis.
- **local_sources** (optional, comma-separated absolute paths). Local files (podcast transcripts, PDFs, saved articles) the user wants treated as primary research. See "Local sources" below for the compression rule.

If `purpose=interview` and no `job_url` is provided, ask before continuing.

## Slug rule
`<slug>` = lowercase company name, spaces→hyphens, drop punctuation.
Output files:
- `companies/<slug>-notes.md`
- `companies/<slug>.md`

If either file already exists, ask before overwriting.

---

## Stage 1. Direction

Do this in a short narrated block in chat before any searching.

1. **Resolve the target.** If the name is ambiguous, use `parent_org` and `purpose` to disambiguate. Pick the canonical name and the official domain. Record which entity was chosen and why.
2. **Set the purpose.** State it. Note which optional sections will fire.
3. **Inject today's date.** Use it to filter for "last twelve months" findings later.
4. **Lock 3 to 5 spine URLs.** Official site, newsroom, job posting if any, key person profiles. These anchor collection.

Do not skip this. A name collision poisons every later stage.

---

## Stage 2. Collection. Write the notes file first

Create `companies/<slug>-notes.md` from `templates/research-notes.md`. Fill it as you go.

### The atom. Fact plus source
Every line is a claim with its link attached. No naked facts.
- `Claim. [source label](url)` for a confirmed fact from a real page.
- `Claim. [source](url) (aggregator)` for LinkedIn, The Org, Crunchbase, RocketReach.
- `Claim. [UNVERIFIED]` when you believe it but cannot source it. These auto-route to Section 4 (Could Not Confirm).
- Verbatim quotes labeled `VERBATIM`, in quote marks, with their source.

### What to capture. Signal only
Capture only if the fact would change what you say or do in a conversation with this company.

Keep:
- Reveals strategy, or the why behind a bet.
- Names a differentiator or proprietary system, and what it does.
- A specific number, date, name, or title.
- A verbatim quote that shows how a person thinks or talks.
- Signals what a key person values.
- A risk, controversy, or public friction.
- Recent (last ~twelve months).

Skip:
- Generic descriptions of what the company obviously does.
- Boilerplate mission fluff.
- Background true of any company in the category.
- Restating the obvious from the company name.
- Marketing adjectives with no fact underneath.

Two edges to hold:
- **Specific is not the same as obvious.** "Founded 2019," "raised $40M Series B," "reports to the CEO" are specific, so they are never skipped. The skip rule applies only to generic background.
- **Keep all primary-source voice.** Verbatim mission lines and direct quotes read plain but are high signal. Never filter a real quote.

### Where to look (in order)
1. **Primary first.** Company site, official newsroom, blog, earnings calls, launch materials, direct interviews.
2. **Press.** Recent coverage for what just happened in their world.
3. **People.** Personal sites, talks, podcasts, posts, GitHub. Voice and signal live here.
4. **Aggregators last.** Useful for arc and titles but they lag. Tag anything that rests only on these.

### Tool use
- Use `WebSearch` to find candidate sources.
- Use `WebFetch` to actually read pages and pull facts.
- **Capped single retry on failure.** If a `WebFetch` returns a timeout, 403, or empty body, do **one** fallback attempt via `WebSearch` for the same target (e.g., the article title, the press-release headline, the bio name). If that fallback still produces no usable content, route the gap to Section 4 (Could Not Confirm) with what was tried. Do not loop. One fetch, one fallback, then move on.
- Do **not** scrape X or LinkedIn. They block fetches and break ToS. Use the official newsroom, blog, leadership posts, and press instead.

### Local sources (the compression rule)
If `local_sources` was passed, treat each file the same way you treat a web page. Collect, then compress.

1. **Read each local file once during Stage 2.** Use the `Read` tool. Pull signal into the notes file the same way you would from a fetched page. Same atom rule: `Claim. [Local: <filename>](file:<absolute-path>)`. Verbatim quotes go in the notes verbatim, labeled `VERBATIM`, with the same `[Local: ...]` source tag.
2. **Once the notes file holds the extracted signal, the raw local file is done.** Do not re-read it in Stage 4 or Stage 5. The notes file is the only ground truth for the brain map. This is the same discipline as web pages, and it is the reason the map stays small enough to write well.
3. **Group local sources separately** in both the notes Source List (Section 7) and the brain map Sources section (under a `### Local sources` subgroup), so it is obvious which facts rest on user-provided files rather than public pages.
4. **A local file may justify the brain map's Section 5 (High-Signal Deep Dive)** if it is unusually rich (e.g., a CEO podcast transcript, a founder essay). Same rule as web sources. Skip Section 5 if no source clears that bar.

### Notes file structure
Fill the sections that have signal. Drop empty ones. Use the field-by-field structure in `templates/research-notes.md`.

---

## Stage 3. Processing. Grounding and the stop rule

### Grounding rule (mandatory)
Before writing the brain map:
- Every factual claim in the notes must trace to a page fetched in this run.
- Knowledge from training, not from a fetched source, is excluded or marked `[UNVERIFIED]` and routed to Section 4.
- Never invent names, dates, numbers, or quotes. Especially for famous companies, where the model is tempted to write fluent prose from memory and skip the search. Block this.

### Stop rule
Check coverage before allowing the brain map to be written:
- Do you have leadership, recent news, differentiators, and strategic context?
- Does each person passed in `people` have a filled block in Section 2?
- If `job_url` was passed, is Section 3 (The Role) complete?

Rough search budget. ~20 to 35 fetches per run for a fresh company. Stop when coverage is met or the budget is spent. If coverage is thin, say so in Section 4 rather than padding.

---

## Stage 4. Analysis. The operator lenses

This stage runs **inside the notes file**, in Section 6 (Strategic Angles, raw). Lead each with the read, then the reasoning. Tie every point to a specific fact already captured in the notes.

### Lens 1. Conversation strategy
How to talk to each leader. Their language, what they value, what lands and what loses credibility. Pull from their voice quotes and signal.

### Lens 2. The operating read
What is straining inside this company or team right now. Where the operating rhythm probably breaks. Where things drop between functions. Infer from stage, growth, recent launches, and team shape.

### Lens 3. Where I contribute (hypotheses, not facts)
Specific problems matched to a senior ops lead or PM skill set. Concrete. The pilot pipeline they likely lack, the cross-functional coordination their stage demands, the decision rhythm a young team has not built yet. **Mark these as hypotheses to test, not facts.**

### Lens 4. Automation opportunities (hypotheses, not facts)
Specific agentic workflows that could be built for their actual pain. Status synthesis, partner comms drafting, dependency mapping, intake triage, reporting. Tie each to a real workflow surfaced in the research. **Mark these as hypotheses to test, not facts.**

If `purpose=interview`, also draft:
- Smart questions grouped by who is being asked, tied to specifics.
- A short "what not to do" list of tone and factual landmines.

---

## Stage 5. Delivery. Write the brain map from the notes

Create `companies/<slug>.md` from `templates/brain-map.md`. **Write it from the notes file, not from the raw pages.** This is the single discipline that separates a tool from an autocomplete that occasionally searches.

### Derivation rules
- **Sources** = every link in the notes, deduped and grouped, primary versus aggregator flagged.
- **Could Not Verify** = notes Section 4, copied.
- **Strategic Angles** = notes Section 6, sharpened through the four lenses.
- **Everything else** = a view on the notes file. If it is not in the notes, it does not go in the map.

### Section order (identical for every company, drop empty sections)
1. Header. `# <Company> Brain Map`. Italic subhead with purpose and build date.
2. TLDR. Six to nine punchy bullets. Include the single best opening line and a pointer to the highest-signal section.
3. The Org / Team.
4. Key People.
5. The Role (role mode only).
6. Strategic Angles (the four operator lenses, tightened).
7. High-Signal Deep Dive (optional, only when one source is unusually rich).
8. Smart Questions to Ask (interview or meeting mode).
9. What NOT to Do (interview or meeting mode).
10. Could Not Verify.
11. Sources.

### Quality bar
Open the committed exemplar (`companies/stripe.md`) alongside and match it for:
- depth per section
- length per paragraph
- how verbatim quotes are handled (italic, with attribution context)
- voice and rhythm

### After writing
Report briefly in chat:
- File paths written.
- Coverage gaps (anything in Section 4).
- Two or three honest gaps where the map falls short of the exemplar.
