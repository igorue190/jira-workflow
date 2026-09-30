---
name: spec-flow
description: >
  Use this skill for the two things that happen around a Confluence spec rather than inside one
  ticket. **The chain:** once a spec page is accepted, offer what follows — the epic, the split into
  stories, a mock for the UI ones, and the page's Traceability table — as a menu where every item is
  approved separately. **The drift check:** read a spec, its epic and all its children together and
  report where they have stopped agreeing with each other. Triggers on: "the spec is accepted",
  "spec approved", "the spec is signed off", "what's next for this spec", "split the spec into
  stories", "create the epic from the spec", "does the spec still match the tickets", "check the spec
  against the epic", "are these stories still consistent", "did that AC change break anything",
  "what else needs updating", "check the epic for drift", or the commands `/spec` and `/spec-check`.
  Do NOT use this skill to draft the spec page itself (that's `ticket-create`), to review one ticket
  against the schema (`ticket-review`), or to build a mockup (`prototype`).
---

# Spec flow: the chain, and the drift check

Every other skill in this plugin works on **one artifact**. `ticket-create` drafts a ticket,
`ticket-review` audits a ticket, `prototype` draws one screen. This skill is the only one that works
on the **set** — a spec page, the epic under it, that epic's children, and the mocks those children
link — and both of its workflows exist because a set can be broken while every member of it passes
its own review.

**The requester** below means whoever is driving the conversation — BA, dev or QA. They approve
everything, one item at a time. Nothing here runs on its own.

## Prerequisites

The Atlassian Rovo MCP connector, for both Jira and Confluence.

**Both workflows are gated on the session's Confluence answer** (`../../references/confluence.md`
§ *Ask first — every session*): **Search for context** · **Search and write** · **Skip Confluence**.
Ask once, before the first Confluence call, folded into whatever else you owe the requester; carry
the answer for the session.

A skip is not a soft no here the way it is elsewhere — **it takes the spec away, and the spec is the
thing both workflows are about**:

- The **chain** has nothing to chain from. Say so and offer the normal `ticket-create` path instead.
- The **drift check** cannot run. Report it as not run. Do **not** reconstruct what the spec
  probably says from the tickets and then grade the tickets against that: it would agree with itself
  by construction and find nothing, which is the worst possible output — a clean report that checked
  nothing.

Connector not authorized? Don't ask the question at all. Say Confluence is unavailable until they
connect it in their Claude connector settings, and stop — neither workflow degrades usefully.

---

## The two workflows — read one

They share a subject and nothing else: `/spec` never needs the drift checks, `/spec-check` never
needs the chain. So each lives in its own file, and you read the one you're in.

| Entry | Read | What it does |
|-------|------|--------------|
| `/spec`, "the spec is accepted", "split the spec into stories", a page you just published | **`references/chain.md`** | Reads the spec, then offers a **menu**: the epic · the split into stories · mocks for the UI ones · a test matrix per story · the page's Traceability table. One approval per item |
| `/spec-check`, "does the spec still match the tickets", "we changed an AC — what else needs updating" | **`references/drift-check.md`** | Reads the spec, its epic and every child together and reports where they've stopped agreeing. Read-only until a specific edit is approved |

If the request is genuinely both — "the spec changed, re-split it" — run the drift check first. It
tells you what already exists and what's already inconsistent, and a chain that starts by
duplicating three existing stories is worse than no chain.

**The rules below apply to both, and they are not repeated in either file.** Read them here.

---

## Rules that don't bend

1. **Explicit approval is the only accept signal.** The requester's words. Not a commit, not a push,
   not a published page, not the passage of time.
2. **The menu is a menu.** One item approved is one item done. Nothing cascades, ever.
3. **The chain runs the full pass.** Steps B–G per story, matrix rule by rule, second-model review.
   A story created from a spec is not pre-verified by the spec's existence.
4. **Never resolve a disagreement.** Quote both sides, name both sources, report the timestamps as
   context, and stop. Picking a winner silently is how a spec the code diverged from gets buried.
5. **Never open an artifact page.** Prototype artifacts are private to the requester's session, so
   anything fetched back is an auth wall or raw markup, and a finding based on it is invented. You
   check the **link and the dates**. What you *can* do — and `ticket-reviewer` cannot — is ask the
   requester to open the mock and confirm.
6. **Inferred is marked.** Every mapping row you guessed carries `~`, and every finding resting on
   one says so.
7. **Report what you didn't check.** `NOT CHECKED` on every drift report, `Not split out` on every
   split, `Not read` on every story's research gate.
8. **Confluence skipped means the check didn't run.** Never reconstruct the spec from the tickets.
9. **Read, never write, in the repo.** Unchanged from `../../references/repo-context.md` rule 7 —
   this skill adds no repo writes, and specs live in Confluence, not in files.

---

## Related skills

- **`ticket-create`** — drafts the spec page in the first place, and every ticket the chain creates.
  This skill hands off to it rather than drafting tickets itself; there is one ticket-drafting
  implementation and it isn't here.
- **`quick-ticket`** — the fast lane, for a one-liner fix that falls out of a drift finding.
- **`ticket-review`** — audits **one** ticket against the schema and the matrix. Different axis: it
  asks "is this ticket well-formed", `/spec-check` asks "does this family still agree". A batch audit
  there reviews each child separately; it will not notice two children contradicting each other.
- **`prototype`** — builds the mocks in chain item 3, via `/mock`.
- **`test-scope`** — builds the test matrix in chain item 4, via `/test-matrix`. Different question
  again: not "is this story well-formed" but "under which conditions must it work, and which of those
  has nobody specified".
- **`ticket-reviewer`** (subagent, different model) — reviews each story the chain creates, and gets
  the spec requirement as its source material.

## Reference files

Paths are relative to this `SKILL.md`. The two workflow files are **local to this skill**; everything
else is one level up in the plugin's own `references/` — the same copies the other skills read, so a
drift finding cites the same rule IDs a draft was written to.

| File | When to read |
|------|-------------|
| `references/chain.md` | **The `/spec` workflow.** A spec was accepted — read this, not the other one |
| `references/drift-check.md` | **The `/spec-check` workflow.** Read this, not the other one |
| `../../references/project-profile.md` | Once per session — the project for the epic and stories, the default spec spaces, the team's conventions |
| `../../references/confluence.md` | Before the first Confluence call, and always before updating a page or leaving a comment |
| `../../references/doc-template.md` | The page shape, and the Traceability section's format |
| `../../references/ticket-schema.md` | Whenever the chain drafts an epic or a story — what applies to all types |
| `../../references/ticket-types.md` | The body template for the type being drafted. **One section, not the file** |
| `../../references/voice.md` | Alongside the schema whenever the chain drafts a ticket — how the body reads, and what never leaves the chat (U18) |
| `../../references/dependency-matrix.md` | Every ticket the chain creates; rule IDs for drift findings |
| `../../references/auto-review.md` | Before the chain's first create — the reviewer spawn contract |
| `../../references/prototype.md` | The four conditions, before counting anything in chain item 3 |
| `../../references/test-matrix.md` | The axis table, before counting anything in chain item 4 — and in full before building one |
| `../../references/repo-context.md` | When a repo is checked out and a story's research gate needs current behaviour |

Note that the two workflow files use `../../../references/…` for the shared standard, since they sit
one level deeper than this file.
