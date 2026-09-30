---
name: quick-ticket
description: >
  Fast lane for creating a single Jira ticket from a one-line description — two turns, one
  compact drafting reference, no full dependency-matrix pass. Use when the user runs `/ticket`, or says
  "quick ticket", "fast ticket", "just create a ticket for…", "log a bug for…", "raise a ticket
  for this", "one-liner ticket", or otherwise describes one small, self-contained piece of work
  they already understand. Works for BA, dev and QA alike.
  Do NOT use this skill for epics with children, variant/bulk sets across environments or pages,
  converting developer research notes, or anything needing a Confluence page — those go to
  `ticket-create`, which runs the full pass.
---

# Quick Ticket

One ticket, fast, still to standard. Two user turns: pick type + project, then approve.

This skill exists because the full `ticket-create` pass reads ~43 KB of reference material and
runs 5–10 connector calls. That is right for a spec'd story and wrong for "the modal overflows on
mobile". Speed here comes from reading less — **not** from inventing values or skipping approval.

## What stays true even in fast mode

1. **Nothing is created without explicit approval.** The draft stops and waits for `go`.
2. **No invented values.** Anything you can't get from the request is a `⚠️` line in the draft,
   never a guess. Two or more unresolved `⚠️` lines means this isn't a fast ticket — escalate.
   Scope the request implies but never states — the other direction of a rule, a boundary left open —
   is included **tagged `(derived — confirm)`** so the requester can strike it in one word (U17).
   Untagged, it reads exactly like something they asked for, and on a two-turn ticket nobody else is
   looking.
3. **The naming and field rules hold** — summary prefix, label vocabulary, priority defaulting,
   assignee from the connector session, no dormant issue types. All of it is in the quick card and
   the project profile.
4. **Say what was skipped.** The draft footer names what the fast path did not do.

---

## Workflow

**Step 1 — Pick type and project.** Ask in a *single* `AskUserQuestion` call — still one turn, which
is the whole point of this path. Order the options so the value inferred from the one-liner comes
first, and say what you inferred in the option description.

`AskUserQuestion` allows at most 4 options per question. Offer the four most likely and let the
auto-added "Other" cover the rest:

| Question | Options to show | "Other" covers |
|----------|-----------------|----------------|
| Type | Bug · Story · Task · Improvement | Research, Epic |
| Project | The four likeliest keys from `../../references/project-profile.md` § 2 — or, if that table is empty, from the projects visible through the connector | the rest |
| Confluence | Search for context · Search and write · Skip Confluence | — |

Skip the project question if the profile lists exactly one project. Skip the type/project questions
if the user already named both explicitly (`/ticket bug in PROJ: …`). Skip the **Confluence** question if it was already answered this
session — the answer carries — or if the connector isn't authorized, in which case say Confluence is
unavailable and carry on. If all three would be skipped, don't call `AskUserQuestion` at all.

If the type answer is **Epic**, stop and hand off to `ticket-create`.

The Confluence answer doesn't change what *this* skill does — the fast path makes no Confluence calls
either way (Step 2). It's asked here because it's free to ask in a call you're already making, and
because the reviewer in Step 5 needs it: without it, U3 goes unchecked on the one path that most
needs someone else to check it.

**Step 2 — Draft from the quick card and the profile.** Read `references/quick-card.md` (relative
to this `SKILL.md`) and `../../references/project-profile.md` — the card says what to do, the profile
says what the values are on this team's boards. Those two are the only references this skill reads to
*draft*. The one other file it opens is `../../references/auto-review.md`, once per session, and not
until after the create (Step 5). Read the profile once per session, before Step 1, since the project
options come from it. Do **not** read the plugin's
`../../references/ticket-schema.md` or `../../references/dependency-matrix.md`; if you find
yourself needing them, this ticket belongs in `ticket-create`.

**The minimum research floor** (quick card, § Minimum research) is three things, and it is a floor
rather than a budget — do these and stop:

1. **Duplicate search** — one `searchJiraIssuesUsingJql` text search (U4). A close match gets shown,
   and you ask before continuing.
2. **Parent epic read**, when the requester named one or the duplicate search surfaces an obvious
   one. One `getJiraIssue`. If the epic has **Done research children**, stop — their findings need
   reading and that's a full pass (RS5, § Escalate below).
3. **On a UI ticket, open the component** — one `Glob`, then the template and its TS. This is the one
   repo read the fast path keeps, because a UI ticket that contradicts the controls the product
   actually has is wrong in a way no reviewer can catch from Jira alone (`repo-context.md`
   § *Reading source, not just docs*).

**No Confluence search and no `docs/` sweep**, even inside a checkout — that's full-pass work, and
the reviewer in Step 5 does it on the other model. Beyond the floor, the rule is unchanged: one
`Glob` or `Grep` to confirm a name the requester gave you is fine, and anything more means this
ticket wants `ticket-create`.

**Step 3 — Show the compact draft and stop.**

```
DRAFT (fast) — reply "go" to create, or "go, no review" to skip the second-model pass
─────────────────────────────────────────────
TICKET:    <prefix - summary>
Project:   <KEY>          Type: <type>
Priority:  <level>  [defaulted — no urgency signal in source]
Assignee:  <resolved name> (you)
Labels:    <feature area>, ai-draft
Parent:    ⚠️ no epic identified

<body per the quick card — Story/Task: Requirements + AC.
 Bug: Preconditions / Steps / Actual / Expected + AC.>

⚠️ <anything that had to be assumed or is missing>
─────────────────────────────────────────────
Read:     duplicate search · PROJ-210 (epic) · chatbot.component.html/.ts
Not read: Confluence · docs/ · no full dependency-matrix pass
A second model checks all of that right after the ticket is created.
No prototype either — say "mock it up" and I'll switch to the full pass for it.
Say "full" for the complete pass up front instead.
```

Keep the `[defaulted]`, `⚠️` and `(derived — confirm)` markers **in the draft**. They are what makes a
two-turn ticket safe to trust — each says a different thing: a value nobody chose, a check that
didn't pass, and a line the request never contained.

They do not survive into Jira. All three are stripped in Step 4, before the create — they are the
requester's decision aids, and a description carrying them tells everyone who opens the ticket that
nobody re-read it (U18a, quick card § *Voice*).

The `Read` / `Not read` pair is the same instrument as the full pass's `RESEARCH` block, shrunk to two
lines. Name what you actually opened — a footer claiming "no repo docs" on a ticket where you read
the component misleads in the same direction as one claiming coverage it doesn't have, and both cost
the footer its credibility.

Make the rest of it match reality too. If the requester chose **Skip Confluence** in Step 1, the
`Parent`/doc `⚠️` lines say *not checked*, never "none exists". On `go, no review` the review line
goes away entirely and is replaced by `Nothing has checked U3/U4/U5 on this ticket — run
review-ticket <key> when you want that.` The footer's job is to state the ticket's actual coverage; a
footer that overstates it is worse than no footer.

The prototype line is **UI-facing tickets only**. On a DB, report-data, notification-token or
infrastructure ticket it's noise, and a footer listing things nobody could have wanted is a footer people
stop reading — which costs the Confluence and matrix lines their credibility too. Note that it's the one
skipped item the second model does **not** pick up: the reviewer can flag UI2, but it can't build
anything.

**Step 4 — On `go`, strip the markers, create, read back.** Follow the connector sequence in the
quick card — it now starts at step 0: **reduce the approved draft to ticket text.** The `⚠️` lines,
`[defaulted]`, the `Read`/`Not read` footer and the field block stay in the chat, and every
`(derived — confirm)` resolves to confirmed (drop the tag), struck (drop the line) or still open
(keep the requirement plainly worded, question goes in a Jira comment after the create). That's U18a,
and on this path it matters more than on the full pass: a two-turn ticket is the one most likely to
be approved with markers still in it.

Then pull `getJiraIssueTypeMetaWithFields` before the first create in a project this session, send the
description as ADF (not raw markdown), create, then **read the issue back** and report the real
key and URL — never a predicted one. If the parent didn't take, set it in a follow-up edit.

**Step 5 — Independent review, automatically.** The fast path skips checks; it does not skip
review. Once the key is real, say in one line that the review is running — it's a second model reading
the schema and the matrix, so the pause is real and an unexplained pause reads as a hang — then spawn
the `ticket-reviewer` subagent on it. See "Automatic review after create" below.

The requester can decline it: `go, no review` (or "just the ticket") creates and stops. Honour that
without argument, and then **say what the ticket is missing** — on that path nothing has checked U3, U4
or U5, so the footer's coverage line must drop its claim rather than carry a review that never ran.
Offer `/review-ticket <key>` for later in the same breath. Report its findings under the created key, then offer to
apply the fixes it proposes (each one approved individually) or a Confluence page via
`ticket-create`.

Anything other than `go` or `go, no review` — a correction, a question, a new field — is an edit to the
draft, not approval. Redraft and stop again.

---

## Automatic review after create

Every ticket this skill creates gets reviewed by a **different model** before you hand it back. The
fast path's whole premise is reading less, which means the drafting model is the *least* reliable
judge of what it left out — so the judging is done by someone else.

**The procedure is in `../../references/auto-review.md`** — how to spawn it, what to pass, and what
to do with the findings. Read it before the first create of a session. It's the same contract
`ticket-create` and `spec-flow` use, so a fast-path ticket is reviewed to the same standard.

What's specific to the fast path:

- Say the ticket came from the **fast path**, so the reviewer knows U3/U4/U5 were deliberately left
  to it and reports them as *its* job rather than as drafter error.
- If a repo is checked out, pass `../../references/repo-context.md` and the repo root. Checking the
  ticket's names against the code is work this path skipped and the reviewer should pick up.
- **The source material is the one-liner**, exactly as the requester typed it, plus any correction
  they made before saying `go`. One line, costs nothing — and fast-path tickets are the ones most
  likely to have filled a gap from inference rather than from the request.
- **The derived lines and how each resolved**, for the same reason and with more weight here: a
  one-line request means more of the ticket was inferred than on any other path, and the
  `(derived — confirm)` tags that flagged it are gone from the description by the time the reviewer
  fetches it. "Nothing derived" is a fine answer; passing nothing says something different.
- **Say that no prototype was built**, so a UI2 gap is reported as an **open offer** (`/mock`, or
  `ticket-create`) rather than a drafting error. Do **not** pass `../../references/prototype.md`:
  there's never a prototype here to check, and a path you pass is a file it will read.

---

## Escalate to `ticket-create` instead of drafting

Say which one applies, then hand off:

- an **Epic**, or anything with child tickets
- a **variant / bulk set** — same ticket across environments, pages or tenants
- **developer research notes** to be converted into tickets
- a **Confluence page** is needed
- a **prototype or mockup** is wanted — `/mock`, "mock this up", "show me what it looks like".
  Building one means reading the product repo's design tokens and publishing an artifact, which is more
  work than this whole path, and `ticket-create` owns the offer gate (UI2). Escalate only when the mockup
  is **asked for**, or when the ticket genuinely can't be specified without one. A UI ticket with no
  Figma link is *not* an escalation — "the modal overflows on mobile" is a fast ticket and always was
- a **restricted-board item that needs code, QA or a deployment** — it must spawn a linked ticket in
  the main delivery project (P3/P4), which is a two-ticket job
- **work under an epic that has Done research children** — their findings live in comments and
  attachments, not in their descriptions, and reading them is a full pass (RS5). Drafting from the
  brief re-asks a question the team already paid to answer
- **2+ `⚠️` lines** you can't resolve from the request

## Reference files

| File | When to read |
|------|-------------|
| `references/quick-card.md` | Every fast ticket — what to do |
| `../../references/project-profile.md` | Once per session, before Step 1 — the project keys, prefixes, types, priorities and labels the card refers to. Pass its path to the reviewer too |
| `../../references/auto-review.md` | Before the first create of a session — the spawn contract for the reviewer |
| `../../references/ticket-schema.md`, `../../references/ticket-types.md`, `../../references/dependency-matrix.md`, `../../references/voice.md` | Never here. You resolve their absolute paths and pass them to the `ticket-reviewer` subagent, which does that reading on the other model — that's the whole trade. `ticket-types.md` is not optional to pass: without it the reviewer checks the description structure against no template at all. `voice.md` is condensed into the quick card's § *Voice* for drafting, and still passed in full so the reviewer's U18b check runs against the real thing |
| `../../references/repo-context.md` | Not read here either — but its § *Reading source, not just docs* is the rule behind Step 2's mandatory component read, and its path goes to the reviewer whenever a repo is checked out |
