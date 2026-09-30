# Automatic review after create: the spawn contract

Every ticket this plugin creates is reviewed by a **different model** before the requester sees the
result. This file is the single copy of how that happens — what to spawn, what to pass it, and what
to do with what comes back.

`quick-ticket`, `ticket-create` and `spec-flow` all create tickets, and each carries a short block
naming only what *its* path passes differently. The procedure itself is here, once, for the same
reason the schema and the matrix live in one place: three copies of a contract drift apart, and the
copy that drifts is the one nobody reads twice.

Read this file before the first create of a session. The reviewer's own instructions are in
`../agents/ticket-reviewer.md`.

---

## Why a second model

A model reviewing a ticket it wrote thirty seconds ago is checking its own assumptions. The failures
that matter here are the silent ones — an epic link that didn't take, a description that flattened
into one paragraph, acceptance criteria that read as testable to whoever wrote them, a requirement
invented in drafting that reads exactly like a real one. None of those look wrong from the inside.

So the review goes to someone else, it reads the **live ticket** out of Jira rather than the draft,
and it is **read-only**: it proposes rewrites and cannot apply them.

## How to spawn it

The `Agent` tool, in the foreground — `run_in_background: false`, because the findings are part of
what you report this turn.

| Field | Value |
|-------|-------|
| `subagent_type` | `"ticket-reviewer"` — the plugin's read-only reviewer, pinned to a different model in its own definition |
| `model` | **Only** pass this if the current session is already running the reviewer's model (Sonnet). Then pass `"opus"`, so it is genuinely a second opinion rather than the same model grading its own work. Otherwise omit it and let the agent's own pin stand |

## What to pass, every time

1. **The created key(s).** Real keys, read back from the create — never predicted ones.
2. **Absolute paths**, resolved on this machine, to `references/project-profile.md`,
   `references/ticket-schema.md`, `references/ticket-types.md`, `references/dependency-matrix.md` and
   `references/voice.md`. Add
   `references/repo-context.md` if a repo is checked out, and `references/confluence.md` if a page
   was created or updated. **A path you pass is a file it will read**, so don't pass one that can't
   apply — and do pass `ticket-types.md`, or the reviewer checks the description structure against
   no template at all. `voice.md` is one page and carries the U18b register check. The profile is
   what the reviewer checks keys, prefixes, labels and priorities against — without it, every
   convention finding is a guess.
3. **Which path made the ticket** — fast path, full pass, or spec chain. It changes what counts as a
   drafting error versus what the path deliberately left for the reviewer.
4. **The source material, verbatim.** Whatever the ticket was drafted from — the requester's
   message, the meeting notes, the dev write-up, the spec requirement — pasted in full, plus any
   correction they made to the draft before approving. Give the path or URL too if it came from a
   file or a page.

   **Your summary cannot substitute.** It was written by the model that wrote the ticket, so it
   agrees with the ticket by construction, and a requirement invented in drafting is invented in the
   summary too. This is the one check that cannot be run from Jira alone (U17), and it runs in both
   directions: a line in the ticket that traces to nothing, and a line in the source that never made
   it in. For something genuinely long, quote the passages that matter in full and point at the rest.

   Nothing to pass means faithfulness is not checkable — and the reviewer says so on a `SCOPE` line
   rather than letting "ready for dev" imply a check it couldn't run.
5. **The derived lines, and how each resolved.** Every requirement or AC that carried
   `(derived — confirm)` in the draft, quoted, each marked **confirmed**, **struck** or **still
   open** — plus, for an open one, that you left the question as a Jira comment on the ticket.

   This one is new in the same change that stripped the tags from the description (U18a), and it is
   load-bearing for the same reason the source material is: in the live ticket a derived requirement
   is indistinguishable from a stated one, so a reviewer without this list cannot run U17 on the half
   of the ticket most likely to contain invented scope. It costs a few lines. "Nothing was derived"
   is a valid and useful thing to pass; passing nothing at all is not the same message, and the
   reviewer will report the gap on a `SCOPE` line.
6. **The Confluence answer** — in scope or not. The subagent has nobody to ask, so an unstated answer
   means it makes no Confluence calls and reports U3 as unchecked. Don't let a session where the
   requester said "search for context" become a review that skipped the spec because nobody told it.
7. **The prototype context, if one was published** — the artifact **URL**, one paragraph on what the
   mock **shows** (screens, states, viewports), one on what it is **interactive about** (which
   controls are wired, which states a reviewer can reach), and where the **design tokens** came from
   (repo path, tenant). It cannot discover an artifact and cannot open one, so an unmentioned
   prototype is a prototype it reports as a missing design, and unmentioned interaction is behaviour
   it can't check the AC against.

## What each path passes differently

| Path | Say this | Why |
|------|----------|-----|
| **`quick-ticket`** (fast) | This came from the **fast path**, so U3/U4/U5 were deliberately left to it. No prototype was built — report UI2 as an **open offer** (`/mock`), not a drafting error. Don't pass `prototype.md`. | The checks it skipped are the reviewer's whole job here, and framing them as drafter error is wrong |
| **`ticket-create`** (full) | This came from the **full pass**. On a mixed source, add the **confirmed item list**; on a large epic, this batch's **slice** of the planned children. | Completeness is judged against what the requester confirmed, so an item they dropped isn't a missing ticket |
| **`spec-flow`** (chain) | The spec page **URL and version**, plus the **verbatim text of the requirement(s)** this story implements. | The best source this check ever gets — a durable human-authored artifact rather than a chat message. Quote it; a paraphrase of a requirement is still a paraphrase |

## Batching

| Shape | Reviewers |
|-------|-----------|
| One ticket | One reviewer |
| **Variant set** | **One reviewer for the whole set**, given every key. The body is identical by construction, so ask it the per-variant question explicitly: does each ticket carry the right environment, tenant, prefix and assignee (TE2, U13)? |
| **Dev notes, or a spec split** | **One reviewer per ticket**, spawned in parallel in a single message. These bodies are genuinely different, and a shared review would blur them |
| More than ~6 tickets | Review them all if the requester is waiting on quality; otherwise say which you reviewed and which you didn't. Never let an unreviewed batch read as a reviewed one |

## What to do with the findings

- **Show the review verbatim**, per key. Don't compress it to "looks good", and don't drop findings
  you disagree with — you wrote the ticket, so your disagreement is the weakest evidence available.
  Argue with a finding in one line *after* quoting it.
- **Fixes need approval, one at a time.** The reviewer proposes; it cannot edit. Apply only what the
  requester approves, through the connector.
- **A blocked verdict is not a rollback.** The ticket exists. Report it and offer the fix; never
  delete or quietly rewrite a created ticket to make a review pass.
- **If the subagent is unavailable or fails**, say so and offer `/review-ticket <key>` in-session.
  Never report a review that didn't run.

## The cost, stated honestly

This spends wall-clock on every path, the fast one included. `/ticket` promises **two user turns**,
not a fast clock: after `go` there is a second model reading the schema and the matrix before the
requester sees anything.

That trade is deliberate. The fast path skips those checks while drafting, so the only thing between
a two-turn ticket and an unchecked one is a review that actually runs *before* the requester moves
on — a review delivered after they've closed the tab is a review nobody reads. If they want the
ticket and nothing else, skip it on request, and drop the footer's coverage claims with it.
