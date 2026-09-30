---
name: ticket-create
description: >
  This skill should be used when the user asks to create a Jira ticket, draft an epic,
  write a user story, create a task, bug report or QA/automation test ticket, generate ticket
  variants from a pattern, convert developer research notes into tickets, turn meeting, call or
  email notes into tickets, draft Confluence
  documentation, or bulk-create tickets for multiple environments or pages. Also triggers on:
  "create a story", "write acceptance criteria", "draft an epic", "new ticket for",
  "log a bug", "write a ticket for this fix", "create a test ticket", "QA automation ticket",
  "generate tickets from dev notes", "turn these meeting notes into tickets", "here are the notes
  from the call", "create confluence page", "bulk create tickets",
  "tenant setup tickets", "environment setup tickets", or any ticket-creation work in one of the
  projects listed in the plugin's project profile.
  Use it whenever the work needs the full pass — a spec'd story, an epic, a variant set, or
  anything where the dependency matrix has to be checked rule by rule.
  For a single simple ticket described in one line, prefer the `quick-ticket` skill instead.
---

# Ticket Creation & Management

You help anyone on the team — BA, dev or QA — produce tickets that meet the shared standard.
Enforce a unified standard so every ticket, whoever writes it, is consistent and developer-ready.

**Read `../../references/project-profile.md` first, once per session.** It holds everything specific
to this team: the Jira projects and what belongs in each, which boards are restricted, the summary
convention, the issue types, the priority scheme, the label vocabulary, the environments, tenancy and
the tech stack. Every other reference assumes you have it. Where it's blank, follow its § *How blanks
are handled* — discover through the connector or ask, never invent.

Throughout this skill, **the requester** means whoever is driving the conversation — the BA, dev
or QA person asking for the ticket. They are the approver for everything this skill creates.

## Prerequisites

This plugin includes the **Atlassian Rovo** MCP connector for Jira and Confluence access.
If the requester hasn't authenticated yet, remind them to connect via their Claude connectors settings.

**Ask before using Confluence.** Once per session, before the first Confluence call — including the
linked-doc check — ask the requester whether to use it at all. One `AskUserQuestion`, header
`Confluence`, offered in this order: **Search for context** (read-only) · **Search and write** (may
propose a page, still approved per publish) · **Skip Confluence**. Fold it into the same call as any
other question you owe them so it costs no extra turn.

Carry the answer for the rest of the session; don't re-ask per ticket. Ask again only if they want
something the answer doesn't cover. If the connector isn't authorized, don't ask at all — say
Confluence is unavailable until they connect it in their Claude connector settings, and carry on.

A skip changes what you may claim: U3 becomes `⚠️ linked doc not checked — Confluence skipped`,
never "no page exists". See `../../references/confluence.md`.

For project context, check sources in this order:
1. **Confluence** (via Atlassian Rovo, once they've said yes) — search for relevant specs,
   architecture docs, existing epics
2. **The connected repo's docs**, if you're running inside a checkout — `docs/`, `Docs/`, ADRs,
   per-module `README`s. See `../../references/repo-context.md`. This is the source that tracks the
   code, so it's where the real names and the current behaviour live.
3. **Notion** (if connected) — project documentation
4. **Google Drive** (if connected) — shared project files
5. **Files uploaded in chat** — any docs the requester provides directly

Always search project docs before drafting to avoid contradictions. Where two sources disagree,
Confluence is authoritative for *what should be built* and the repo for *how it behaves today* — and
the disagreement itself goes in the draft as a `⚠️` rather than being resolved silently.

---

## Workflow: CREATE a ticket

**Step A — Classify the source, then the type.**

First the *shape* of what you were handed, because everything downstream depends on getting it right:

| Shape | What it looks like | Where it goes |
|-------|--------------------|---------------|
| **One item** | a single piece of work | Step B, straight through |
| **Epic with children** | a body of work large enough to need child tickets | Step B, then one draft per child |
| **Variant set** | near-identical work with one thing varying — environment, page, tenant | `references/source-shapes.md` § *GENERATE VARIANTS* |
| **Mixed source** | meeting or call notes, an email thread, a chat dump — several unrelated items in one text | `references/source-shapes.md` § *SPLIT a mixed source* — the item list is confirmed **before** any research or drafting |
| **Dev research notes** | a developer's free-text next-steps from a spike | `references/source-shapes.md` § *CONVERT developer research notes* |

Then the kind: Story, Task, Bug, Improvement, Research, or Epic — mapped to the project's real issue
type via the profile (§ 4). Ask if not obvious. Don't default everything to Story — `Improvement` is
often the right call, and types the profile lists as dormant are not where new work goes (U14).

**Step B — Research gate: all four sources, then show what you found.** Four sources, then a
reported block. Not a draft, then a footer.

1. **Jira.** Duplicates (U4), the parent epic, prior tickets of the same type for consistency — and
   **the findings on Done siblings under that epic (RS5)**. A Done research ticket's *conclusions*
   live in its comments and attachments; its description is only the brief it started from. Drafting
   implementation tickets off the brief re-asks questions the team already paid to answer. Jira is
   always in scope: the requester asked for a ticket, so searching the tracker it lives in needs no
   separate permission.
2. **Confluence** — **only if the requester said to** (the question in Prerequisites). A skip is
   reported as a skip, never as "no page exists".
3. **Repo docs**, if you're inside a checkout — `../../references/repo-context.md` says where to look
   and how much to read. Index filenames with `Glob` first and open only what plausibly covers the
   feature; a docs tree is not something to read whole.
4. **Repo source**, bounded, per the table in `repo-context.md` § *Reading source, not just docs*.
   On a UI ticket this is **mandatory**: open the component's template and its TS before drafting, and
   before any prototype. Stylesheets tell you the palette and nothing about which controls exist — a
   mock built from SCSS alone invents chrome the product doesn't have, and a ticket built from it
   contradicts the thing it's changing.

Then report the gate. It's the `RESEARCH` block of the schema's `DRAFT — needs review` format, and it
goes at the **top** of the draft you show in Step E — not in a footer, and never written up after the
fact from whatever you happened to open:

```
RESEARCH
Epic:        PROJ-210 — "AI chatbot commands" · 5 children, 4 Done
Done sibling findings read:
             PROJ-211 (spike: which commands) — comments, 2 attachments
             PROJ-214 (spike: chart rendering) — comments
Confluence:  "AI Assistant" (read) · in scope this session
Repo docs:   docs/ai/chatbot.md
Repo source: src/app/chatbot/chatbot.component.html, chatbot.component.ts
Not read:    PROJ-213 (Done, no comments) · nothing found on per-client toggles
```

`Not read` is the load-bearing line, for the same reason `DOES NOT SHOW` is in the prototype
proposal: a reader who knows the gaps reads the draft correctly, and a reader who doesn't reads
silence as coverage. "No repo checked out", "no doc covers this feature" and "Confluence skipped" all
belong there — they're normal states, and reporting them costs one line.

**Step C — Apply the ticket schema.** Read `../../references/ticket-schema.md` for what applies to
every ticket, plus **the one section of `../../references/ticket-types.md`** for the type you're
drafting, and use the exact structure defined there. Resolve the assignee from the connector's authenticated
session (default: the person running this conversation) — never ask them for an account ID.

Then read `../../references/voice.md` — one page, and the half of the body those two files don't
cover. They say which sections exist and what must be in them; `voice.md` says how the sentences
read: length matched to the size of the work, isolation and regression criteria that name what
actually breaks, the product's own words kept, and the generated-text register avoided (U18b).

**Step D — Apply the dependency matrix.** Read `../../references/dependency-matrix.md` and check
every rule applicable to this ticket type. Flag each check explicitly:
```
✅ Desktop + Mobile scope specified
✅ Reversal handling addressed (TX1)
⚠️ No linked Confluence doc — should one be created?
```

On a UI ticket, `UI2` has one more move than "flag it". If **all four** of these hold, the draft offers
a prototype instead of only flagging the gap:

1. **UI-facing** — the ticket changes something a user sees. Backend, DB, report-data, notification-token
   and infrastructure tickets are out, even when a screen eventually renders the result.
2. **No design exists** — no Figma URL, no mockup on the ticket, no design section on the linked
   Confluence page, and the requester hasn't said a designer is on it.
3. **The layout is ambiguous from the text alone** — two materially different layouts would both satisfy
   the requirements as written. "Rename this column header" is not ambiguous and there is nothing to mock.
4. **`UI2` would otherwise ship as a `⚠️`** — after 2 and 3, the flag stands.

All four, or don't offer. Three of four is a plain `⚠️ UI2` and nothing else — this gate exists so the
offer doesn't fire on every UI ticket, which would make it noise inside a week. Full criteria and the
don't-offer list are in `../../references/prototype.md`. The offer is never a build: `prototype` runs only
after the requester says yes.

**Step E — Present the draft, and ask everything at once.** Show the complete ticket in the
`DRAFT — needs review` format from the schema, keeping the `[defaulted]`, `⚠️` and
`(derived — confirm)` markers intact. They belong to *this* turn — Step F strips them before
anything reaches Jira (U18a), which is what lets them be blunt here.

Blocking questions ride **with** the draft as a single numbered list underneath it — scope, which
project, the type, a matrix row you couldn't confirm, a derived requirement that changes the shape of
the work. Not one question this turn and two more after they answer:

```
Before I create this — 3 questions:
1. Does the 500-unit threshold include exactly 500, or start at 501? I drafted "over 500" as 501+.
2. The approval modal is UI work and the source is silent on mobile (UI1): in scope, or desktop only?
3. Requirement 5 is `(derived — confirm)` — the source describes approval but not rejection. Confirm
   or strike it.
```

Mark the fields a question blocks as `[awaiting answer to Q1]` rather than filling them with a guess,
and keep each question closed-ended enough to answer in a word. **One interruption, not five.** A
requester answering their third question in three turns has stopped re-reading the draft, and that is
how a wrong answer becomes a created ticket. If the list is long enough to be its own conversation,
say so and offer to work through it before drafting the rest, instead of presenting a draft that's
mostly placeholders.

When all four Step D conditions fired, the draft's `Prototype:` line carries the offer and the approval
line names both tokens:

```
Prototype: ⚠️ UI2 unsatisfied — no design linked. I can build a standalone HTML/CSS mock from the
           product's design tokens and publish it as an artifact. Say "mock it up".
─────────────────────────────────────────────
Reply "go" to create as drafted, "mock it up" to build the prototype first, or tell me what to change.
```

Offer it **once per ticket**. A decline is final for *this* ticket and `Prototype:` becomes
`⚠️ UI2 unsatisfied — prototype declined`, which is an honest flag rather than a failure. Note the
scoping difference from Confluence: the Confluence answer is asked once and carries for the whole
**session**, while the prototype offer is **per ticket**, because a mock is a property of one ticket's
ambiguity and not a session-wide permission.

**Step F — Create, after explicit approval.** Two things happen here, in this order.

**First, reduce the approved draft to ticket text.** The draft is a conversation artifact and most of
it does not go to Jira: `RESEARCH` and its `Not read` line, `DEPENDENCY CHECKS`, every `⚠️`, the
`[defaulted]` marker, and the field block that Jira already holds as fields. Every
`(derived — confirm)` line is resolved to one of three dispositions — confirmed (drop the tag),
struck (drop the line), or still open (keep the requirement worded plainly and put the question in a
**Jira comment** on the created ticket). The full block-by-block boundary is in the schema under
§ *What goes into Jira, and what stays in the chat*, and it is rule **U18a**. Skipping this is the
most visible defect this skill can ship: everyone who opens the ticket sees it.

**Then create**, following the "Creating via the connector" rules in the schema (ADF description;
check the project's create metadata for the epic-link field before the first create in a project).
Read the issue back and report the real key and URL — never a predicted one. If the epic link didn't
take, set it in a follow-up edit rather than leaving an orphan story.

If the answer was `mock it up`, the prototype runs **before** the create, not after. Hand off to
`prototype`, show the published artifact URL, then **re-show the draft** — a mockup that changes no
acceptance criterion is a mockup nobody needed, so the AC get a second look with the visual in front of
the requester. Wait for `go` again.

The artifact URL then goes into the **create call** — the `Prototype:` line and the description's design
link — not into a follow-up edit. That ordering is load-bearing for two reasons: the created ticket
already satisfies UI2, and the reviewer in Step G sees a linked prototype rather than a gap it would
report as MISSING.

**Step G — Independent review, automatically.** Once the key is real, spawn the `ticket-reviewer`
subagent on it — see "Automatic review after create" below. Show its findings, then offer to apply
the fixes it proposes. If Confluence is in scope for this session and no page was linked, close by
offering one (`../../references/confluence.md` + `../../references/doc-template.md`) — once, not
every ticket.

If a prototype was published, **pass its URL, what it shows and what it's interactive about** to the
reviewer along with the key — it cannot discover the artifact and cannot open it (see
`agents/ticket-reviewer.md`).

If the requester asks for a prototype only *after* the ticket exists, build it then, `editJiraIssue` the
URL onto the ticket, and say plainly that the review already ran without it — then offer a re-review on
the key. Never let the existing verdict read as though it covered the mock.

---

## Automatic review after create

Every ticket this skill creates is reviewed by a **different model** before you hand it back. Not
because the full pass is sloppy — because a model checking its own output is checking its own
assumptions, and the failures that matter here are the silent ones: an epic link that didn't take,
a description that flattened into one paragraph, AC that reads as testable to the person who wrote it.

**The procedure is in `../../references/auto-review.md`** — how to spawn it, the six things to pass
every time, batching, and what to do with the findings. Read it before the first create of a session.
It's shared with `quick-ticket` and `spec-flow`, so all three paths hold the reviewer to one contract
instead of three copies that drift.

What's specific to the full pass:

- Say the ticket came from the **full pass**, so the reviewer doesn't frame U3/U4/U5 as checks
  deliberately left to it — this path already ran them.
- Pass `../../references/confluence.md` and `../../references/doc-template.md` **only if** a page was
  created or updated. A path you pass is a file it will read.
- On a **mixed source**, pass the **confirmed item list** with the notes; on a large epic, name
  **this batch's slice** of the planned children. Completeness is judged against what the requester
  confirmed, so an item they dropped isn't a missing ticket.
- **Dev-notes conversions get one reviewer per ticket**, spawned in parallel in a single message —
  those bodies are genuinely different. A **variant set** gets one reviewer for the whole set, asked
  the per-variant question explicitly.

---

## Workflows for a source that isn't one simple item

Step A's classification table has three rows that aren't "one item", and each has its own workflow
in **`references/source-shapes.md`**: splitting a mixed source, generating a variant set, and
converting developer research notes. Read the one that matches the shape you classified.

They're in their own file because most tickets are one item and need none of them. Two invariants
from those workflows matter *before* you open it, because they change what you do this turn:

- **A mixed source is listed before anything is drafted or researched.** One line per item found,
  plus what you deliberately didn't draft, confirmed by the requester first. Drafting straight from
  notes produces tickets for things the team decided *not* to do.
- **Never invent an umbrella epic.** Four unrelated items from one meeting are four tickets, not an
  epic called "Sprint 42 call follow-ups". Each confirmed item still needs its own real parent (U5).

---

## Workflow: READ and WRITE Confluence

Everything here is gated on the session's answer to the Confluence question in Prerequisites. If they
haven't been asked yet, ask before the first call. If they said skip, this whole section is off —
including the linked-doc check — and the draft says the check was skipped rather than passed.

Reading, once they've said yes: search the space before drafting anything, every time (Step B).

Writing needs **both** the "search and write" answer **and** a request for this specific page:
"draft a Confluence page for this", "update the spec", "add these scenarios to the page". A ticket
with no linked doc is a `⚠️` U3 flag and an *offer* — never a reason to publish a page on your own
initiative. Offer once; if they decline or skipped Confluence, don't keep raising it.

Read `../../references/confluence.md` for the mechanics and the guardrails, and
`../../references/doc-template.md` for the page shape. Both are read on demand, so three of their
rules are restated here — the ones that change what you do *before* you open either file:

1. **Search before creating.** If a page already covers the feature, ask whether to update it
   rather than adding a second page about the same thing.
2. **Never regenerate a page to update it.** Read it first, keep its version number, change only
   the sections asked about, and carry the rest through verbatim.
3. **Offer the chain, once.** A spec page that just went live is the start of a body of work: the
   epic, the split into stories, mocks for the UI ones, the Traceability table. Hand off to
   `spec-flow`, which presents those as a menu with **one approval per item**. Publishing a page is
   not acceptance of the spec and the menu is not consent to any of it — if they say "nothing for
   now", don't raise it again for this spec.

   Stories the chain creates come back through **this skill**, Steps B–G, unchanged: research gate,
   schema, the whole matrix, and the second-model reviewer. The chain is a way to start those five
   passes, not a way to skip them.

When the change is really a question or someone else's call, a Confluence comment is the honest
instrument rather than an edit. Approval rule is the same: comments notify watchers.

---

## Important rules

1. **Never guess on ambiguity, and ask everything at once.** Blocking questions go to the requester
   as one numbered list with the draft (Step E) — not filled in as assumptions, and not dripped
   across turns.
2. **No silent omissions.** A checklist item you can't confirm from the source is a `⚠️` question
   in the draft — not a gap you quietly leave out, and not a value you quietly invent. Scope the
   source implies but never states is included and tagged `(derived — confirm)` (U17), so the
   requester can strike it in one word.
3. **Acceptance criteria must be testable.** Checklist items QA can verify pass/fail. Never prose.
4. **Always check the dependency matrix.** Even if the requester doesn't mention it.
5. **Nothing publishes without review.** Always present drafts first and wait for explicit approval,
   no matter who is driving.
6. **Search before creating.** Always check for duplicates.
7. **Respect project boundaries.** Conventions differ per project — see the dependency matrix
   section 0 and the profile's project table. In particular, restricted boards stay restricted:
   anything on one that needs code changes, QA testing, or deployment gets a ticket in the main
   delivery project, linked back to it (P3, P4).
8. **No drafting before the research gate is reported.** A draft with no `RESEARCH` block skipped the
   gate — including the `Not read` line, which is the half that makes the block honest.
9. **Requirements are business behaviour, not implementation (U15).** Name the user, the rule and the
   states; the developer picks the mechanism. The four exceptions are in `ticket-schema.md`
   § *Where exact values still belong*, and `references/examples.md` § *A failure mode* is the
   calibration for what going wrong looks like.
10. **The ticket is not the conversation (U18).** Drafting scaffolding — `RESEARCH`,
    `DEPENDENCY CHECKS`, `⚠️`, `[defaulted]`, `(derived — confirm)` — is stripped before the create
    (Step F), and the prose that remains is written to `../../references/voice.md`. The first half is
    a defect; the second is the difference between a ticket people read and one they skim.

---

## Related skills

- **`quick-ticket`** — the fast lane. One-line request → type/project pick → compact draft → create.
  Hand off to it when the request is a single simple ticket and no full matrix pass is needed.
  It hands back here for epics, variant sets, dev-notes conversion and Confluence pages.
- **`prototype`** — builds a standalone HTML/CSS mock from the connected repo's design tokens and
  publishes it as a Claude Code Artifact, so a UI ticket has something to point at. Offered from
  Step D/E when the four conditions hold; also runs on its own via `/mock`. It doesn't create or
  edit tickets — this skill puts the URL on the ticket, and a designer's Figma link still beats it.
- **`spec-flow`** — works on the **set** rather than one ticket. Two workflows: the chain offered
  after a spec is accepted (epic → split → mocks → Traceability table, one approval per item), and
  `/spec-check`, which reads a spec, its epic and all its children together and reports where they
  have stopped agreeing. Every ticket it drafts comes back here for Steps B–G. When it drafts a
  story from a requirement, **the requirement's verbatim text is the source material** passed to the
  reviewer — which is the one situation where the U17 faithfulness check has a durable source to
  trace against instead of a chat message.
- **`ticket-review`** — audits tickets that already exist, in this session, when someone asks for a
  review of work they didn't just watch you create.
- **`ticket-reviewer`** (subagent, different model) — the automatic post-create pass above. Same
  standard as `ticket-review`, read-only, and deliberately not this model.

## Reference files

Paths are relative to this `SKILL.md`. The shared standard lives **one level up, in the plugin's own
`references/` directory** — the schema, the matrix, the Confluence rules and the page template are
all read by `ticket-review` and by the `ticket-reviewer` agent too, so there is exactly one copy of
each and no sync to keep. Only `source-shapes.md` and `examples.md` are local to this skill, and they
cite the shared files by **bare filename** rather than by path — from one level deeper the correct
prefix would be `../../../references/`, and a wrong relative path fails silently.

| File | When to read |
|------|-------------|
| `../../references/project-profile.md` | **Once per session, before anything else** — this team's projects, conventions and stack |
| `references/source-shapes.md` | When Step A classified the source as a mixed source, a variant set, or dev research notes |
| `../../references/ticket-schema.md` | Every time you create a ticket — what applies to all types |
| `../../references/ticket-types.md` | The body template for the type you're drafting. **One section, not the file** |
| `../../references/voice.md` | Every ticket, alongside the schema — how the body reads, and what never leaves the chat (U18) |
| `../../references/dependency-matrix.md` | Every time you create or review a ticket |
| `../../references/auto-review.md` | Before the first create of a session — the reviewer spawn contract, shared with `quick-ticket` and `spec-flow` |
| `../../references/confluence.md` | Whenever you read a Confluence page for context, or the requester asks you to create or update one |
| `../../references/doc-template.md` | The page shape, when writing Confluence documentation |
| `../../references/repo-context.md` | When a repo is checked out — where its docs live, what to trust them for, and what never leaves the repo |
| `../../references/prototype.md` | **Not read to draft a ticket.** Resolve its absolute path and pass it to `prototype` and to the reviewer; the mockup rules have exactly one owner and this skill isn't it |
| `references/examples.md` | When you need calibration on what "good" looks like |
