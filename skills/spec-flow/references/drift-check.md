# The drift check: does the set still agree with itself

The `/spec-check` half of `spec-flow`. Read this file when someone asks whether a spec still matches
its tickets, whether a grooming change broke something else, or runs `/spec-check`.

The chain never needs this file. The rules that govern both — including *never resolve a
disagreement*, *never open an artifact page* and *report what you didn't check* — are in `SKILL.md`
and are **not** repeated here: read that first.

## Where to start

`/spec-check <page URL | epic key>`. Given either end, resolve the other: the page's Overview links
the epic (`doc-template.md` rule 5), and the epic's `LINKED DOCUMENTATION` / description links the
page. If neither link exists, ask which epic goes with which page rather than guessing from titles.

This is a **read-only pass that ends in proposals.** No edit is made until the requester approves
that specific edit.

## Step A — Assemble the set

1. `getConfluencePage` — the spec body and its **version and last-modified date**.
2. `getJiraIssue` on the epic; list its children. Fetch each child's description, AC, status,
   `updated` date, and its prototype/design link.
3. Note what you could not fetch. That goes in `NOT CHECKED`, not in silence.

## Step B — Map requirements to stories

The mapping comes from the page's **Traceability table**. That table is the registry, and it lives
on the page deliberately — there is no hidden state file anywhere in this plugin, so the mapping is
visible to every human who opens the spec and editable by them without going through Claude.

| Situation | What to do |
|-----------|-----------|
| Table present and complete | Use it. Mark those rows **confirmed**. |
| Table present, some requirements or children missing from it | Use what's there; infer the rest. |
| No table (any page written before the section existed) | Infer the whole mapping by text. **This is not a finding** — most existing pages have no table. Offer to add one. |

**Every inferred row is marked `~` and reported as inferred.** An inferred mapping is a guess about
which requirement a story came from, and every finding built on it inherits that uncertainty. Marking
it is the same instrument as `(derived — confirm)` on a requirement: the inference is wanted, and the
unmarked version of it is not, because nobody downstream can tell which is which.

## Step C — Run the checks

Order in the report is worst-first, which is roughly this order:

| Block | What it catches | Bounds |
|-------|-----------------|--------|
| **CONTRADICTS** | A story's AC states its requirement's rule differently — a threshold, a window, a role, a state. | Quote **both sides verbatim** and resolve **neither**. |
| **UNBUILT** | A requirement no story implements. The spec moved ahead of Jira. | A requirement explicitly `Out of Scope` on the page is not unbuilt. |
| **UNSPECCED** | A child ticket that traces to no requirement. U17's invented-scope check, applied across artifacts instead of inside one draft. | A ticket that legitimately predates the spec, or is linked as related rather than as a child of this outcome, is not a finding — say which. |
| **SIBLINGS DISAGREE** | Two children stating the same shared rule two different ways. | This is the grooming case: an AC edited in one story during refinement, its siblings left behind. It needs no spec at all to be a real finding. |
| **STALE PROTOTYPE** | A story links a mock published before that story's requirement last changed. | **Link and timestamps only.** Never open the artifact page — see the rules below. |
| **SPEC HYGIENE** | Open Questions a ticket now answers, still marked Open; an Open Question with no owner (`doc-template.md` rule 4); Change Log untouched while the body clearly moved. | Same findings as `ticket-review` § *CHECK the linked Confluence doc* — read from the set's angle rather than one ticket's. |

**Timestamps are a hint, never a verdict.** Confluence gives one `lastmodified` for the **whole
page**, not per section; Jira gives one `updated` per issue, not per field. So "the page moved two
days ago and the story hasn't moved in nine" is worth reporting *next to* the two quotes, and is
never grounds to conclude which side is stale. That call belongs to the requester and often to the
team — the same rule `repo-context.md` applies when Confluence and the repo disagree.

## Step D — Report

```
SPEC DRIFT — "Order refund handling" (v7, edited 2d ago) · epic PROJ-300 · 6 children
─────────────────────────────────────────────
MAPPING   4 confirmed from the Traceability table · 2 inferred (~) · 1 requirement unmapped

CONTRADICTS
- Req 5 "refund must complete within 24h" (page v7)
  vs PROJ-302 AC 3 "the refund completes within 48 hours"
  Page edited 2d ago; story last updated 9d ago. Which one is right is your call.

UNBUILT
- Req 7 (rejection flow) has no story. Present on the page at v7.

UNSPECCED
- PROJ-306 "bulk refund export" traces to no requirement (U17). ~inferred mapping.

SIBLINGS DISAGREE
- PROJ-302 AC 3 "within 48 hours" vs PROJ-304 AC 2 "within 24h" — same rule, two numbers.

STALE PROTOTYPE
- PROJ-303 links a mock published on 14 Aug; Req 5 changed after that. Page not opened.

SPEC HYGIENE
- Open Question 2 ("who can refund a shipped order?") is answered by PROJ-302 — still Open.

NOT CHECKED
- Confluence in scope · page read at v7
- 6 of 6 children fetched
- artifact pages not opened — private to your session
─────────────────────────────────────────────
6 proposed edits — approve individually, or name the ones you want.
```

**`NOT CHECKED` is mandatory.** Same instrument as `Not read` in the `RESEARCH` block, and the same
reason: a drift report with no scope line reads as full coverage of the set, and the one thing worse
than finding drift late is believing you looked. Confluence skipped, children you couldn't fetch, a
child whose description didn't parse, artifact pages — all of it goes there.

A clean set is reported as clean on the header line, explicitly. Don't show an empty report and let
the absence of findings do the talking.

## Step E — Propose, one artifact at a time

Every finding gets a proposed edit, and every edit is approved on its own. There is no "apply all" —
the whole value of this pass is that the requester decides *which* side of each disagreement moves,
and a bulk approval throws that away.

- **Jira edits** go through `editJiraIssue` after approval on the specific rewrite, per
  `ticket-review` § Step E.
- **Confluence edits** need the *search and write* answer and follow
  `../../../references/confluence.md` § *Updating a page* — read, keep the version, change only the
  section, Change Log row, read back the new version.
- **A new story** for an `UNBUILT` requirement hands off to `ticket-create` (or `quick-ticket` if
  it's genuinely a one-liner) — and it is a real ticket, so it gets the full pass and the reviewer.
- **A question, not a correction** — "is Req 5 still 24h after PROJ-302?" — is a Confluence
  comment with an owner, or a Jira comment, not an edit. Approval rule is the same: comments notify
  watchers.
