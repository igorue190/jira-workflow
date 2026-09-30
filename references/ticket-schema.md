# Ticket Schema

Every Jira ticket must follow this structure. No exceptions, regardless of who creates it.

**Project-specific values live in `project-profile.md`** — the project keys, summary prefixes, issue
types, priority scheme, label vocabulary, environments and tenancy. This file says how a ticket is
built; the profile says what the values are on *your* boards. Where this file names a section of the
profile, read that section; where the profile leaves it blank, follow its § *How blanks are handled*.

---

## Before anything else: pick the project

The candidate projects, what belongs in each, and which boards are restricted are in
`project-profile.md` § 2. Route by that table (P1), resolve aliases that aren't real keys (P2), and
keep code work off restricted boards (P3). An empty table means: list visible projects through the
connector and ask — never guess a key.

---

## Draft preview format

Every draft is shown to the requester in this shape before anything is created, so a draft is never
mistaken for a live ticket. One block per ticket; repeat the block for variant sets — except
`RESEARCH`, which is shown **once for the whole set**, since the research is what the set shares.

```
DRAFT — needs review
─────────────────────────────────────────────
RESEARCH
Epic:        <key + title, and how many children are Done>
Done sibling findings read: <keys, and whether from comments or attachments — RS5>
Confluence:  <page(s) read, or: skipped this session / connector not authorized>
Repo docs:   <paths — docs/orders/refunds.md — or: no repo checked out>
Repo source: <paths — the component template + TS on a UI ticket>
Not read:    <every gap, named>
─────────────────────────────────────────────
[EPIC] Epic name                    (only when drafting an epic with children)

TICKET: <summary, with the profile's prefix convention>
Project:     <KEY from project-profile.md § 2>
Type:        <issue type, mapped per project-profile.md § 4>
Assignee:    <resolved name> (assign to me)
Priority:    <level>   [defaulted — no urgency signal in source]
Labels:      <feature area>, <client>, ai-draft
Parent:      <epic key>   [or: ⚠️ no epic identified]

DESCRIPTION
<As a / I want / So that for stories — plain prose is equally fine.
 Current → desired behaviour for improvements.
 Preconditions / Steps / Actual / Expected for bugs.>

REQUIREMENTS
1. …
2. … (derived — confirm)      <only on scope the source implies but never states>

ACCEPTANCE CRITERIA
1. …
2. …
3. <isolation — affects only the intended tenant>
4. <regression — existing behaviour unaffected>

DEPENDENCY CHECKS
✅ <check that passed>
⚠️ <check that needs the requester's answer>

LINKED DOCUMENTATION
Spec:        <Confluence page, or: ⚠️ none exists — should one be created?>
Prototype:   <Figma / designer mockup URL, or published artifact URL, or:
              ⚠️ UI2 unsatisfied — no design linked>   (UI-facing tickets only — omit the line otherwise)
─────────────────────────────────────────────
```

Keep the `DRAFT — needs review` header and the rules until the requester approves. The `[defaulted]`
marker on Priority, the `⚠️` markers and the `(derived — confirm)` tags are not decoration — they're
the requester's cue for what still needs a decision, so don't drop them to make a draft look finished. The
three say different things: `[defaulted]` is *a value nobody chose*, `⚠️` is *a check that didn't
pass*, and `(derived — confirm)` is *a line the source never said* (rule 5 under Writing
Requirements). Blocking questions about any of them go to the requester in one numbered list with the draft,
not one turn at a time.

All three are **draft markers, not ticket text**: they are resolved before the create call and none
of them appears in the Jira description. See § *What goes into Jira, and what stays in the chat*
below (U18a).

`RESEARCH` is what makes the draft checkable: a named key or file path can be opened by the next
person, "per the docs" cannot. It is a **gate, not a footer** — `ticket-create` reports it before
drafting, because a research block written after the fact describes what you happened to read rather
than what you set out to cover.

Two lines carry most of its weight. **`Not read`** is the honest half: a reader who knows the gaps
reads the draft correctly, and a reader who doesn't reads silence as coverage. "No repo checked out",
"Confluence skipped" and "no doc covers this feature" all belong there — they're normal states, and
naming one costs a line. **`Done sibling findings read`** is RS5: a Done spike's conclusions live in
its comments and attachments, never in the description, and drafting from the brief re-asks a question
the team already paid to answer.

It's also where a source *conflict* surfaces — if Confluence and the repo disagree on the same fact,
quote both and name both, and leave the call to the requester (see `repo-context.md`).

`Prototype` appears **only on UI-facing tickets**, and holds a URL — never a file. A designer's Figma
link is the best value it can have. When no design exists and the layout is ambiguous from the words
alone, `ticket-create` offers to build one and publish it as a Claude Code Artifact, and the draft then
carries that artifact URL (see `dependency-matrix.md` UI2 and `prototype.md`). On a DB, report-data,
notification or infrastructure ticket the line is omitted entirely — a block that is usually `N/A` is a
block people stop reading.

Three things that line must not become:

- **Not an attachment.** Attach a mockup file and it dies with the ticket, for the same reason DB1 keeps
  scripts out of Jira. A link to something durable, every time.
- **Not a design approval.** A generated mockup satisfies UI2 as *something to point at*. If a designer
  still owes a real design, the line says so alongside the URL rather than reading as though the gap
  closed.
- **Not a new label.** A prototype adds no label. Labels stay inside the live vocabulary (U6) — a
  `prototype`/`has-design` label the board has never seen is noise in every future query. The URL in
  this block is the record.

`ai-draft` stays a statement about the **ticket text**. A human-written ticket that gets a generated
mockup does not become `ai-draft`; the prototype's own provenance travels with the artifact URL.

If a ticket on a restricted board (profile § 2) turns out to need code changes, QA testing, or a
deployment, that work gets a ticket in the **main delivery project**, linked back to the original (P3,
P4). Don't try to do code work on those boards.

---

## What goes into Jira, and what stays in the chat

The `DRAFT — needs review` block above is a **conversation artifact**. Most of it never reaches Jira,
and the parts that do go into *fields and links* rather than into description text. This is rule
**U18(a)**, and it is checked on every created ticket.

| Block in the draft | Where it actually goes |
|--------------------|------------------------|
| `DRAFT — needs review`, the dividers | Chat only |
| `RESEARCH`, including `Not read` | **Chat only.** It's the gate the requester checks the draft against, not something a developer needs in the ticket six weeks later |
| `Project` · `Type` · `Assignee` · `Priority` · `Labels` · `Parent` | Jira **fields**. Never restated as description text — a description that repeats its own fields goes stale the moment someone changes one |
| `[defaulted]` on Priority | Chat only. The field carries the value; the marker is a question to the requester |
| `⚠️` markers | Chat only. An unresolved `⚠️` at create time is either a blocking question (so don't create yet) or a real gap worth a **Jira comment** — never a symbol in the description |
| `DEPENDENCY CHECKS` | **Chat only, always.** Nothing from this block belongs in the ticket |
| `DESCRIPTION` · `REQUIREMENTS` · `ACCEPTANCE CRITERIA` | The description, as ADF |
| `LINKED DOCUMENTATION` | Real Jira **links** — the Confluence page as a link, the design URL in the description or its own field. Not a text block reading `Spec: ⚠️ none exists` |
| `(derived — confirm)` | Resolved before the create — see below |

### Resolving `(derived — confirm)` before the create

By the time you call create, every derived line has one of three dispositions, and none of them is
"leave the tag in the description":

1. **Confirmed** — drop the tag. It is now a requirement like any other.
2. **Struck** — the line goes with it.
3. **Still open**, and the requester wants the ticket now — keep the requirement, worded plainly, and
   put the question in a **Jira comment** on the created ticket, addressed to whoever can answer it.

Option 3 is what keeps U17 honest without leaving `(derived — confirm)` in a description someone
reads six weeks later. A comment is what a person would actually do: it's visible to the team, it
notifies, and it outlives the chat session the draft lived in.

The reviewer still runs the U17 faithfulness check, because it is **passed the derived lines and how
each one resolved** (`auto-review.md` § *What to pass, every time*) rather than reading a tag out of
the ticket. That is the one place the derived/stated distinction survives the create, which is
exactly why passing it is not optional.

### Why this is a defect and not tidiness

A Jira description carrying `DEPENDENCY CHECKS ✅⚠️`, a `Not read:` line, or a `(derived — confirm)`
tag tells everyone who opens it that the ticket was generated and never re-read. That reputation
attaches to the next ticket too, and to the requester who approved it. Register — how the remaining
prose *sounds* — is the softer half of the same rule, and it lives in **`voice.md`**.

---

## Summary line

Apply the convention in `project-profile.md` § 3 — a discipline prefix if the team uses one (U1), and
any per-project variant (U2). With no convention defined, write a plain summary and don't import one.

What holds regardless of convention: keep it under ~80 characters where you can, and make it readable
without opening the ticket. A summary names the thing and what happens to it — `Order list overflows
the container on mobile`, not `Mobile issue`.

---

## Issue types in use

The templates in `ticket-types.md` are keyed by **kind of work** — Story, Task, Bug, Improvement,
Research, Epic. `project-profile.md` § 4 maps each kind to the issue type your projects actually have;
a kind with no matching type falls back as the profile says (an Improvement with no `Improvement` type
is a `Story` that opens with current → desired behaviour). Profile blank → read the project's issue
types from the connector's create metadata and map by name.

Pick deliberately. **Improvement** — a change to something that already exists and works — is usually
the right kind when a Story would overstate the work, even when it's filed as a Story.

Once you've picked, the body shape for that kind is in **`ticket-types.md`** — one section each.

### Types that exist but are dormant — don't route new work to them

Boards accumulate types nobody uses any more. Profile § 4 lists them and says where their work goes
instead (U14). Jira will happily accept a dormant type, so the only thing keeping new work out of one
is this check. Only use one if the requester explicitly asks for it.

Don't invent a type that isn't configured on the project — if a request doesn't fit, ask rather than
stretching a type to cover it.

## Priority

The values and the signal mapping are in `project-profile.md` § 5 (the default is Jira's
`Highest · High · Medium · Low · Lowest`). Read the priority off the language in the source rather than
guessing:

| Signal in the source | Priority band |
|----------------------|---------------|
| "production down", "blocking the release", "can't ship without", data loss or a broken money path | top of the scheme |
| "customer-requested", "needed this sprint", named customer waiting, a committed date | high |
| No urgency language present | the profile's default — **and say it was defaulted** |
| "when there's time", "nice to have", cosmetic with nobody waiting | bottom of the scheme |

Never silently pick a priority. If it was defaulted, the draft says so, so the requester can override.

## Assignee

Default to **the person running the conversation** ("assign to me").

Resolve that identity from the connector's own authenticated session — the current-user /
`atlassianUserInfo` lookup. **Never ask the requester to paste their own account ID**, and never
hard-code an account ID into a draft. If the lookup fails, say so and leave the field unset
rather than guessing.

Assign to someone else only when the source or the requester says so explicitly. Two cases where it
usually is explicit:

- **Per-environment variant sets** often get different owners per environment — production and
  staging to one engineer, QA to another. Vary the assignee with the environment when the source
  says so; don't copy one owner across all of them.
- **Named work inside the description** ("Note: this ticket is for @name") should also be
  reflected in the Assignee field, not just the prose.

Resolve any other named person with an account lookup by display name; if the name is ambiguous
or not found, flag it instead of picking the closest match.

## Labels

Use the live vocabulary in `project-profile.md` § 6: feature areas and customer labels the team
actually uses, the AI-drafted marker (`ai-draft` by default) on anything AI-drafted, and the
not-testable marker where QA has ruled work out of automation and the profile defines one.

**Never coin a label.** A label the board has never seen is noise in every future query. With no
vocabulary in the profile, add only `ai-draft`, plus labels the requester names or that sibling
tickets under the same epic already carry.

**No product label** unless the profile says one is used — the project key already says which
product this is. **Components** follow the profile too: left empty unless it lists live ones.

---

## Writing Requirements that don't come back with questions

Applies to every type that carries a Requirements section. Rule 0 governs the four below it: they
are about being *precise*, and rule 0 is about being precise at the right altitude.

0. **Business behaviour, not implementation.** A requirement says what the product must do and for
   whom. The developer chooses how. Name the user, the rule, every state, and the invalid case — do
   **not** name tables, columns, classes, services, endpoints, file paths or migration mechanics. A
   requirement only a developer already fluent in this codebase can read has moved the design
   decision out of the sprint and into the ticket, where nobody reviewed it.

   Need a new role, permission or setting? Say so plainly and point at precedent. *"Add a new role —
   Team Lead — who can see their own team's data only. See PROJ-xxxx for the last role we added."*
   Not how roles are stored.

   | Instead of | Write |
   |------------|-------|
   | "Extend `AI.CommandLkp` with an audience column, seeded via the post-deployment script, IDs immutable (DB5)" | "Add a new role. A manager sees a different set of commands than an employee. See how this was done for `<precedent ticket>`." |
   | "New `POST` endpoint returning an envelope with text plus an optional chart dataset" | "Clicking a command shows an answer in the chat. Some answers include a simple chart." |
   | "Register the command→flag mapping in `AICommandFeatures`" | "Each command can be switched on or off per client." |
   | "Generalise `ActivityStreamSummaryService` into a dispatch service" | "Every command follows the same path, so adding the next one is cheap." |

   The right-hand column is not vaguer than the left — it is the same decision stated where a BA, a
   QA analyst and a stakeholder can all check it. The left-hand column reads as authoritative and is
   frequently wrong, because the drafter picked a design from outside the code.

1. **Enumerate every state separately — not just the happy path.** If a feature has on/off,
   cascade, prerequisite or toggle behaviour, spell out what happens in *each* direction as its
   own numbered item: "Field A ON forces Field B ON", "Field B OFF forces Field A OFF",
   "Field A OFF leaves Field B unchanged". One summarizing bullet ("the fields are linked")
   guarantees a wrong implementation of at least one direction.
2. **Name the invalid state explicitly.** Not "validate the input" — name the specific
   combination or value that must be rejected and can never be persisted. That sentence becomes
   an AC too.
3. **Add an inline example wherever a rule could be read two ways.**
   *e.g. "a single row assigns the group to one user"* — one parenthetical kills the ambiguity.
4. **For a file the client sends us, give the schema.** Exact column/field names as they appear in
   *the file*, and the cross-tab or cross-entity processing order — what must exist or be validated
   before what. This rule is about upload payloads (IM1), not about internal storage: the columns of
   a spreadsheet a client fills in are a contract with the client, and the columns of a table we own
   are rule 0's business.
5. **Mark what you derived — don't drop it, and don't smuggle it in.** Some requirements are
   legitimately *implied* rather than stated: the complementary direction of a rule the source only
   gives one way (it describes approval — what happens on rejection?), a boundary the source leaves
   open, the cascade RL1/RL3 ask for that nobody wrote down. Include them, each tagged
   `(derived — confirm)`:

   > 5. *(derived — confirm)* Director rejection returns the request to the manager with a reason.

   The tag is how thoroughness and faithfulness coexist. A marked derivation is a **proposal the
   requester can strike in one word**; the same line unmarked is invented scope that reads exactly
   like something the client asked for, and nobody downstream can tell the difference — so it ships.
   Carry every material one into the questions list too (`ticket-create` Step E).

   The tag lives in the **draft**, never in the created ticket. All three ways it can resolve before
   the create — confirmed, struck, or still open and carried into a Jira comment — are in § *What
   goes into Jira, and what stays in the chat*.

   Two things that are *not* derivations and need no tag: an inline example (*e.g. …*) illustrating a
   rule the source already states, and a matrix row you answered from the codebase — that one cites
   its evidence instead.

### Where exact values still belong

Rule 0 is about internal design the developer owns. Four things are **business contracts**, not
design decisions, and they stay exact in the Requirements:

| Exception | What stays verbatim | Why it isn't implementation |
|-----------|--------------------|-----------------------------|
| **Task config values** | `InstanceCode = ACME01`, `Instance ID: 44`, `Run NewTenantSetup.ps1` | The value *is* the request. "Configure the instance code" is not a task anyone can complete |
| **Bug identifiers** | Host, tenant, UserID, GUID, order number, error text verbatim | You are describing an observed event, not designing one. A reproduction with the identifiers filed off doesn't reproduce (BG2, BG4) |
| **Upload schemas (IM1)** | The file's columns, types and required flags | The client writes that file. Its shape is agreed with them, not chosen by a developer |
| **Notification events and tokens (NT1)** | Event name and the full token list, spelled as implemented | The contract with whoever writes the template. A token list is the interface, not the internals behind it |

Everything outside those four is rule 0's: state the behaviour and let the developer pick the
mechanism.

---

## When the source isn't in English

Customer calls, support requests and QA notes arrive in whatever language they were written in. Draft
the ticket in **English** — that's what the boards are in and what every discipline reads — with one
exception that is not a matter of style.

**Every string someone will search for, or type, stays verbatim in the original**, with the English
rendering in parentheses. UI labels, menu paths, field and column names, error text, notification
subject lines, and quoted client wording:

```
Orders - Кнопка "Зберегти" ("Save") does nothing on the order edit modal
...
Actual result
The page shows Не вдалося завершити операцію ("the operation could not be completed")
and the order stays in Pending.
```

A translated label is unsearchable and unverifiable: the developer greps for "Save" and finds
nothing, QA can't tell whether they're looking at the right control, and the string in the ticket no
longer matches the string in the product. Translate the *narrative*; never the *identifiers*. This is
the same rule BG3 applies to error messages and IM1 to file columns, extended to the case where the
verbatim string happens not to be English.

Read urgency and scope by **meaning, not by keyword** — the priority table above lists English
phrases, but "горить", "клієнт чекає" and "не критично" carry the same signals as "blocking",
"client is waiting" and "nice to have". A source with no English urgency words in it is not a source
with no urgency signal, and defaulting it to `Medium` on that basis is a mistake the `[defaulted]`
marker will not catch.

## The per-type templates

The body shape for each kind — Story, Task, Bug, Improvement, Research, Epic — plus the note on how
little a request on a restricted board needs, is in **`ticket-types.md`**. Read the one section for the type you're
drafting.

They live in their own file because you draft one type at a time: carrying all six through here
meant reading five templates you didn't need on every ticket. Everything below this line applies to
every ticket regardless of type, so none of it moved.

**`voice.md` is the other half of the body.** The template says which sections exist and this file
says what must be in them; `voice.md` says how the sentences read — length matched to the work, the
isolation and regression criteria naming what actually breaks, the product's own words kept, and the
generated-text register avoided (U18b). It's one page and it applies to every type.

---

## Field rules

| Field | Rule |
|-------|------|
| Project | One of the keys in profile § 2 (P1). Aliases resolved per P2. Code work stays off restricted boards (P3). |
| Summary | The profile's prefix convention (U1) and any per-project variant (U2). Aim for under 80 chars. |
| Requirements | Numbered list. What must exist. Separate section from AC. |
| Acceptance Criteria | Numbered list of testable pass/fail statements, or Given/When/Then. Never prose. Include isolation and regression checks. |
| Priority | The profile's scheme (§ 5). Map from the source's urgency language; default *and say so*. |
| Assignee | Defaults to the person running the conversation, resolved via the connector's current-user lookup. Never ask for an account ID. Vary per environment on variant sets. |
| Labels | Profile § 6 vocabulary, the AI-drafted marker, the not-testable marker where defined. Never coin one. |
| Components | Per profile § 6 — empty unless it lists live ones. |
| Parent | Per profile § 7 — by default every story belongs to an epic. See "Creating via the connector" below for how the link is set. |
| Dependencies | Link actual ticket keys. Cross-project links must be created, not just mentioned (P4). |
| Documentation | Link the Confluence page. If none exists, flag it. |
| Design / prototype | UI-facing tickets link a design: Figma if one exists, otherwise the published prototype artifact URL (UI2). A link, never an attachment, and the link text says which kind it is. Non-UI tickets omit it entirely. |
| Tenant | Multi-tenant products (profile § 10): named explicitly whenever behaviour is tenant-specific. |
| Status on close | Per profile § 8, where the team defines statuses that mean different things (U12). |

---

## Creating via the connector

Field-level mechanics, so a create call doesn't fail after the requester has already approved the draft.

**Description is rich text (ADF).** The Jira API takes Atlassian Document Format, not raw
markdown. Section headings (`REQUIREMENTS`, `ACCEPTANCE CRITERIA`) and numbered lists must come
through as real structure — if a draft round-trips as one flat paragraph with literal `1.`
characters, the formatting was lost and the ticket needs fixing before hand-off.

**Epic link — check each project's style.** The epic relationship is set differently on
company-managed (classic) projects than on team-managed (next-gen) ones: `parent` is the modern field
and is what to try first, but a company-managed project may still expect the **Epic Link custom
field** instead. Projects on one site are often a mix of both.

Don't guess per project. Before the first create in a project you haven't written to this
session, pull the create metadata for that project + issue type (`getJiraIssueTypeMetaWithFields`)
and use whichever epic field it actually exposes. If the epic link can't be set in the create
call, create the ticket and then set the parent in a follow-up edit — never leave a story orphaned
because the field name was wrong (U5).

**Verify after creating.** Read the created issue back and confirm the type, parent, priority,
assignee and labels landed as drafted. Report the real key and URL — never a predicted key.
