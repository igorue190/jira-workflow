---
name: test-scope
description: >
  Build a **test matrix** for a ticket or a spec — the grid of conditions a change has to be
  exercised under, crossed from the requirements and acceptance criteria, the states a prototype
  shows, and the variation axes the plugin's dependency matrix already declares (viewport, tenant,
  role, toggle state, notification channel, environment, language, boundary values). It reports
  which combinations need a test, which are deliberately out of scope, and which are **undefined in
  the requirements** — then lands the grid on the Jira ticket. Use when the user runs `/test-matrix`,
  or says "test matrix", "testing matrix", "what should I test here", "what combinations do we need
  to cover", "which variations does this affect", "coverage grid", "test plan for this ticket",
  "did we cover mobile / the other tenant / the toggle off case", or asks what QA needs to check
  before a story goes to dev. Also use for chain item 4 of `/spec`.
  Do NOT use this skill to write test cases with steps and expected results — that belongs to QA or
  to a test-case skill the project profile names, and this skill hands off to it. Do NOT use it to
  raise clarification questions for the BA, to search a test management tool for cases that already
  exist, to check whether one ticket is well-formed against the schema and the matrix
  (`ticket-review`), to check whether a spec and its children still agree (`spec-flow`), or
  to build the mockup itself (`prototype`).
---

# Test scope: the matrix of what to test

Every other skill here asks whether an artifact is *well-formed*. This one asks a different question:
**under which conditions does this have to work, and which of those has nobody specified?**

The answer is a grid, and the empty cells are the deliverable. A matrix that lists only what will be
tested cannot tell you what was missed, which is the whole reason to draw one.

**The requester** below means whoever is driving — BA, dev or QA. They approve every write
separately. Nothing here edits Jira on its own.

## Prerequisites

The Atlassian Rovo MCP connector, for Jira. Confluence only when the source is a spec page, and then
only to **read** it — this skill never writes a page.

If the source is a spec and the session's Confluence answer was **Skip**
(`../../references/confluence.md` § *Ask first — every session*), say the pass ran without the spec
and build from the ticket alone, or stop if there is no ticket. Never reconstruct the requirements
from the tickets and then grade the tickets against them — it agrees with itself by construction.

**Read `../../references/test-matrix.md` before building anything.** The axis table, the pruning
rules, the marker vocabulary and the Jira rules all live there, and this file does not repeat them.
Read `../../references/project-profile.md` too — tenancy (§ 10), environments (§ 9), channels (§ 14)
and QA tooling (§ 13) decide which axes exist and which hand-offs are available.

---

## Step 1 — Establish the subject, and read it

`/test-matrix <ticket key>` or `/test-matrix <spec page URL>`. No argument → ask which ticket; don't
guess from recent context.

Fetch it: `getJiraIssue` for the description, **requirements and AC as separate sections** (U8), the
issue type, labels, linked documentation, and any `Prototype:` link. A spec URL instead →
`getConfluencePage`, body and version, and take the requirement numbers **as the page numbers them**.

Three things decide whether there is a matrix to build at all:

| Found | Then |
|-------|------|
| Requirements/AC present, and at least one axis in play | Build it |
| AC absent or prose | Build it anyway — every row becomes `⚠️ no AC` (U9), and that report *is* the finding |
| No axis applies — a data load on a restricted board, a DB script, a one-line copy change | **Say so and stop.** A one-row matrix is theatre |

Declining is a real answer here, the same way `prototype` declines a backend ticket. Name why: "no
variation axis applies — this is a single data fix in one environment."

## Step 2 — Decide which axes are in play

Walk the axis table in `../../references/test-matrix.md` § *Where the axes come from* against the
ticket's type and content. For each axis, exactly one of:

- **In** — the ticket states behaviour varies along it, or a matrix rule for this type demands it be
  stated. It goes in `AXES`.
- **Collapsed** — it does not vary, or it varies identically by construction. It goes in `COLLAPSED`
  **with the reason**.
- **Unanswered** — a rule demands it and the ticket is silent. `COLLAPSED`, carrying the rule ID and
  `⚠️ … unanswered`.

The third case is the one that earns the pass. `⚠️ UI5 unanswered` says nobody decided which browsers
this targets — which is information the ticket did not contain a minute ago. Never resolve it by
inventing a list.

Then pick the cross. **Two axes at most**, and only a pair on the sourced list in the reference.
Everything else is listed one row at a time. Note which axis values you **inferred** rather than read
— those rows carry `~`.

## Step 3 — Build the grid

Rows come from the requirements and AC, plus the prototype's states where there are any (reference
§ *Where the prototype half comes from* — the artifact is **never opened**; the states come from the
`SHOWS` list, the ticket's `Shows:` line, or the requester).

Mark every cell `●` `◐` `—` `⚠️`, and give every non-`●` cell a lettered footnote. Check the cells
against each other before showing anything: a control that doesn't exist makes *both* its directions
unreachable, so a defect belonging to one axis sits in that axis's whole column rather than in one
cell of it.

Cap at about 40 cells. Over → pairwise, and say so.

## Step 4 — Show the matrix, and stop

Print the scan grid exactly as the reference's § *Chat — the scan grid* lays it out: `AXES`,
`CROSSED`, `LISTED`, the lettered footnotes, `COLLAPSED`, `NOT CHECKED`. Then stop.

`NOT CHECKED` is mandatory and goes on every matrix, clean or not — what wasn't read, what wasn't
searched, whether the mock was opened (it wasn't). A grid with no scope line reads as full coverage.

End with what can follow, as named options, not a pipeline:

```
─────────────────────────────────────────────
3 findings · 11 cells need a test · existing cases not searched

What can follow — each approved separately:
- "comment"  post the matrix on PROJ-123
- "subtask"  create the test-case subtask (QA4) — only if profile § 13 names one
- "cases"    hand the 11 rows to the test-case skill for steps — only if one is installed
- "questions" turn the 3 findings into BA questions
```

**Naming one does that one.** Nothing cascades — the same rule the `/spec` menu runs on.

## Step 5 — Write to Jira, on approval

Per the reference § *Writing it to Jira*. The long form goes in the comment, not the scan grid; read
the issue back afterwards and report what actually landed.

The QA4 subtask is a **separate** approval. No new label (U6). Assignee per U13. Never attach a file.

If a `⚠️` row means an acceptance criterion is missing, **propose** the AC and say it belongs in
`ticket-review`'s edit flow — don't edit AC from here, and never close a finding by writing the
missing requirement into the grid.

## Step 6 — Hand off, and say what wasn't available

QA hand-off skills and a test-management connector ship separately from this plugin, if at all —
profile § 13 names the ones this team has, and no test-management MCP is declared in `.mcp.json`.
Offer each hand-off that's available, and when one isn't, say so on `NOT CHECKED` rather than quietly
dropping it.

An omitted `Case` column is honest. An empty one claims existing cases were searched.

---

## Rules that don't bend

1. **Axes are derived from the dependency matrix, never invented.** An axis the ticket is silent about
   and no rule demands is a guess, and a guessed axis makes the grid look more complete than the
   ticket is.
2. **Two axes crossed, never three**, and only a pair on the sourced list.
3. **Every non-`●` cell carries its reason.**
4. **No test case ever leaves this skill.** One-line intent per row. Steps belong to QA.
5. **Never open an artifact page** — private to the requester's session. Ask them instead.
6. **`NOT CHECKED` on every matrix**, including a clean one.
7. **Inferred is marked `~`**, and any finding resting on an inferred row says so.
8. **One approval is one action.** Approving the grid in chat approves no write.
9. **Never invent an acceptance criterion** to fill a hole. The hole is the finding.
10. **Read, never write, in the repo** — `../../references/repo-context.md` rule 7.

## Related skills

- **`ticket-review`** — different axis entirely: it asks whether *this ticket* is well-formed against
  the schema and the matrix. This asks what the change has to be tested under. A ticket can pass that
  review with three variation axes nobody has decided.
- **`spec-flow`** — `/spec` chain item 4 calls this skill once the stories exist; `/spec-check` asks
  whether the family still agrees, which is a different question again.
- **`prototype`** — supplies the states, via its `SHOWS` list and the ticket's `Shows:` line.
- **`quick-ticket` / `ticket-create`** — where a `⚠️` row turns out to need its own ticket.
- **QA hand-off skills** *(separate distribution, named in profile § 13 if the team has any)* — turn
  approved rows into cases with steps, fill the `Case` column from the test management tool, or turn
  findings into questions for the BA.

## Reference files

Paths are relative to this `SKILL.md`.

| File | When to read |
|------|-------------|
| `../../references/test-matrix.md` | **Always, before building.** Axis table, pruning rules, markers, output shapes, Jira rules |
| `../../references/project-profile.md` | **Always, before building.** Tenancy, environments, channels, QA tooling |
| `../../references/dependency-matrix.md` | Step 2, to walk the rules for this ticket's type and quote rule IDs |
| `../../references/ticket-schema.md` | When a finding proposes an AC or requirement rewrite — U8, U9, U15 |
| `../../references/prototype.md` | When the ticket links a mock — § *When to offer a prototype*, and what a prototype is not |
| `../../references/confluence.md` | Before the first Confluence call, when the source is a spec page |
| `../../references/repo-context.md` | When a repo is checked out and current behaviour decides whether an axis varies |
