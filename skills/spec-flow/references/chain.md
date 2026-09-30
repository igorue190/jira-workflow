# The chain: a spec was accepted, what follows

The `/spec` half of `spec-flow`. Read this file when a spec page has just been published, or the
requester says an existing spec is accepted, approved or signed off.

`/spec-check` never needs this file, and this file never needs the drift checks — that's why they're
separate. The rules that govern both, and the Confluence gate that governs both, are in `SKILL.md`
and are **not** repeated here: read that first.

## When this fires

Right after a spec page is published through
`../../../references/confluence.md` § *Creating a page*, or whenever the requester says an existing
spec is accepted, approved or signed off. **Explicit words, from the requester** — that is the whole
accept signal. There is no other one: nothing here watches git, and a committed or pushed file is not
an approval this plugin can see (README § *The chain and the drift check*).

**Offer once per spec.** Same scoping as the prototype offer, and for the same reason: acceptance is
a property of one spec, not a standing session permission. A decline is final for this spec.

## 1. Read the spec before offering anything

Fetch the page with `getConfluencePage` — body **and version number**. You need three things out of
it before the menu is honest:

- **The requirement list**, numbered as the page numbers them. These numbers are the currency of
  everything downstream; don't renumber them.
- **Whether an epic already exists** — the Overview links it per `doc-template.md` rule 5, and a
  Traceability table (if the page has one) names the stories. Search Jira as well: a spec written
  after the epic is common, and offering to create a second epic for the same feature is the failure
  this step prevents.
- **Which requirements are UI-facing**, so item 3 counts them honestly rather than offering to mock
  a backend spec.

If the page has no Requirements section, or the section is prose rather than a numbered list, say so
and stop. There is nothing to split, and inventing the numbering is inventing the spec.

## 2. Show the menu

```
SPEC ACCEPTED — "Order refund handling" (v1) — https://…
Epic: none found · 8 requirements · 2 UI-facing

What can follow — each approved separately, nothing runs until you name it:
1. Epic in PROJ from the Overview + Requirements                      → "epic"
2. Split into stories — 8 requirements, I'd propose 5 stories         → "split"
3. Mock the UI stories — 2 UI-facing reqs, neither has a design       → "mock"
4. Test matrix per story — 4 axes in play across the 8 reqs           → "matrix"
5. Traceability table on the page, filled in as the stories land      → rides with 1–4
Or "nothing for now" — I won't offer again for this spec.
```

**The menu is not consent.** `"epic"` creates the epic and stops there. Read the item they named,
do that item, report it, and wait. Never continue into the next item because it is obviously next —
that is exactly the behaviour this plugin refuses everywhere else, and a chain is the one place it
would look reasonable.

Item 3's line must count rather than promise — and at menu time no story exists yet, so it counts
what it actually can: **UI-facing requirements**, and how many of those already carry a design link.
The **all four** conditions in `../../../references/prototype.md` § *When to offer a prototype* are
checked **per story, after it exists** (§ 5), so this line never claims a story met a gate nothing
has been measured against. Zero UI-facing requirements is a real answer, and the line then says so
instead of offering.

Item 4's line counts the same way: **how many variation axes the requirements put in play**, read off
the axis table in `../../../references/test-matrix.md`. No story exists yet, so it cannot count
grids — and zero axes is a real answer that declines rather than offers.

## 3. Item 1 — the epic

Draft it through `ticket-create`, `Epic` type, per `../../../references/ticket-types.md` § *Epic*. The
spec's Overview is the description's source; the Requirements section is what the children will
cover, not what goes in the epic body verbatim.

Approval, create, read back the real key — the normal Step F rules. Then put the epic key in the
page's Overview if it isn't there (a Confluence write: needs *search and write* plus approval,
and it batches into item 5 rather than being its own page version).

## 4. Item 2 — the split

**List before you draft.** This is the instrument from
`../../ticket-create/references/source-shapes.md` § *SPLIT a mixed source*, and it matters more
here, because the requester is reviewing your reading of *their own spec*:

```
PROPOSED SPLIT — confirm before I draft
1. PROJ · Story       · refund of a shipped order          · Req 1, 2
2. PROJ · Story       · refund inside the 24h window       · Req 3
3. PROJ · Story       · rejection flow                     · Req 7
4. PROJ · Improvement · refund shows in the audit trail    · Req 4, 5
5. PROJ · Task        · refund reason codes seeded         · Req 6

Not split out — say the word if any should be:
- Req 8 is non-functional (performance) — folded into 1's AC rather than its own story
```

One line per story, with the **requirement numbers it implements**. Requirements grouped into one
story and requirements split across two both need to be visible here, because both are judgements
the requester may disagree with — and after the tickets exist, disagreeing costs a rewrite.

The `Not split out` block is not optional. Same reasoning as `Not read` in the `RESEARCH` block and
`Not drafted` in a mixed source: a requirement you deliberately folded into someone else's AC is
invisible unless you name it, and silence reads as coverage.

Then, per confirmed story, **run `ticket-create` Steps B–G in full**. All of them:

- **Step B, the research gate**, including the epic's Done research siblings (RS5). The spec is not
  a substitute — it says what should be built, and the repo says what exists today
  (`../../../references/repo-context.md` § *Precedence when sources disagree*).
- **Step C, the schema.** **Step D, the whole dependency matrix**, rule by rule.
- **Step G, the second-model `ticket-reviewer`** — spawned per
  `../../../references/auto-review.md`, which is the shared contract for what to pass it.

**The chain is not a fast lane.** A chain that skipped the matrix would turn one approval into five
unchecked stories, which is worse than the manual path it replaces — the requester approved a split,
not five ticket bodies they never saw.

**Pass the reviewer the spec as source material.** This is the part worth getting right. The
reviewer's U17 faithfulness check normally degrades to "not checkable" because a ticket drafted from
a chat message has no durable source to trace back to. A spec-chain story has one by construction:

- the **spec page URL and version**, and
- the **verbatim text of the requirement(s) this story implements** — quoted, not summarised.

That gives the check both directions for real: a requirement in the ticket tracing to no line of the
spec is invented scope, and a line of the requirement that no AC covers is a dropped one. Don't
paraphrase the requirement — a paraphrase written by the model that wrote the ticket agrees with the
ticket by construction (`agents/ticket-reviewer.md` § *Inputs*).

More than about six stories: draft in batches, checking in between, and tell the reviewer which
slice of the planned children this batch is.

## 5. Item 3 — the mocks

Only for stories that meet all four conditions in `../../../references/prototype.md`, checked **per
story after it exists**, not for the set. Hand off to `prototype`; it publishes the artifact and
hands back a URL, and the URL goes onto the ticket per `ticket-create` Step F.

A mock built after its ticket already exists means the reviewer ran without it. Say that plainly and
offer a re-review on the key, exactly as `ticket-create` Step G requires. Never let the earlier
verdict read as though it covered the mock.

## 6. Item 4 — the test matrix

Per story, after it exists — the same scoping as the mocks, and for the same reason: a matrix is a
property of one story's variations, and at menu time no story exists to have any.

Hand off to `test-scope`, which reads `../../../references/test-matrix.md`. The axes come from the
dependency matrix, the rows from that story's requirements and AC, and the states from the mock's
`SHOWS` list where item 3 built one.

Two things specific to running it from the chain:

- **The spec is the source for the requirement rows**, quoted verbatim, exactly as § 4 passes it to
  the reviewer. A paraphrase agrees with the ticket by construction and makes a `⚠️ no AC` row
  unreachable.
- **A story whose axes all collapse gets no matrix**, and that is reported as a line rather than
  padded into a one-row grid. Over about four stories, do them in batches and check in between.

The matrix lands on each **story**, per `test-matrix.md` § *Writing it to Jira* — one approval per
comment, one more for a QA4 subtask. It never goes on the spec page: this chain writes Confluence
once, in item 5, and a matrix is per story rather than per piece of work.

## 7. Item 5 — the Traceability table

One page update, when the chain stops — **not one per story**. A five-story chain that wrote the
table five times would burn five page versions and five Change Log rows for one piece of work, and
every one of them would need its own approval.

Write it per `../../../references/doc-template.md` § *Traceability* and the update rules in
`../../../references/confluence.md` § *Updating a page*: read the page, keep the version, change only
that section plus the Change Log row, show the change as a change, read back and report the new
version. If the page has no Traceability section yet, adding it is the same operation — say that the
update inserts a section.

This table is what `/spec-check` reads later. A chain that skips it isn't broken; the check just
falls back to inferring the mapping, and says it inferred.
