---
name: ticket-review
description: >
  This skill should be used when the user asks to review, audit, or check an existing Jira ticket
  for completeness, quality, or compliance with team standards. Also triggers on:
  "review PROJ-123", "check this ticket", "is this ticket complete", "audit this story",
  "what's missing in this ticket", "review my AC", "check acceptance criteria",
  "is this ticket ready for dev", "can QA test this", "scan for blockers", "find stale tickets",
  or "check unanswered questions".
  Use this skill any time a BA, dev or QA person wants to validate existing work rather than
  create new work.
---

# Ticket Review & Audit

Review and audit existing Jira tickets against the team's standards.
Catch what's missing, inconsistent, or unclear before it reaches dev/QA.

**The requester** below means whoever is driving the conversation — BA, dev or QA. They approve
every edit this skill makes.

## Prerequisites

Same as `ticket-create` — requires the Atlassian Rovo MCP connector.

## Who runs this review

Two paths, same standard:

- **In-session (this skill).** Someone asks to review a ticket — `/review-ticket PROJ-123`, a
  batch audit, a blocker scan. The ticket already exists and wasn't drafted by this model a minute
  ago, so there's no self-review problem and no reason to spawn anything. Do it here.
- **Automatic, on a different model (the `ticket-reviewer` subagent).** Every ticket
  `quick-ticket` or `ticket-create` creates is reviewed right after the create by
  `agents/ticket-reviewer.md`, which is pinned to a different model. That agent applies *these*
  rules — it reads the same schema and matrix — and is read-only: it proposes rewrites and the main
  session takes them to the requester.

So when you change a rule in the references, both paths change together. When you change something
about *how findings are reported*, check `agents/ticket-reviewer.md` too — its output format is what
the requester actually sees on a fresh ticket.

If a requester asks for a second opinion on a review you just gave, spawn `ticket-reviewer` on the
key rather than re-reading your own findings — and pass it the session's Confluence answer, because it
has no way to ask.

---

## Workflow: REVIEW a single ticket

When someone asks to review a ticket (e.g., "review PROJ-123"):

**Step A — Fetch.** Pull the ticket from Jira via the Atlassian connector. Read
`../../references/project-profile.md` once per session first — every convention check below (keys,
prefixes, labels, priorities, types, tenancy) is judged against it.

**Step B — Schema completeness.** Check against `../../references/ticket-schema.md`.
Every required field must be present and properly formatted.

**Step C — Dependency matrix.** Check every applicable rule from `../../references/dependency-matrix.md`.

Two of those rules are worth naming here because they're checks on the description *text* rather than
on a field, and both are in `../../references/voice.md`:

- **U18a — drafting scaffolding.** Search the description for `RESEARCH`, `Not read`,
  `DEPENDENCY CHECKS`, `⚠️`, `[defaulted]`, `(derived — confirm)`, and for any block restating fields
  Jira already holds. None of it belongs in a ticket. It costs one pass over text you already
  fetched, it is never uncertain, and it is visible to every person who opens the ticket — so it goes
  near the top of the findings, not in a footnote. On an older ticket, say plainly that it predates
  the rule rather than reporting it as someone's error.
- **U18b — register.** Padding, an isolation or regression criterion that names nothing, a paraphrase
  of a name the product already has, generated-text phrasing. **Cap it at two findings**, always in
  the "could be better" category, never blocking. A review where style findings crowd out the
  correctness ones has buried what the requester needed.

**Step D — Consistency check.**
- Search for related tickets to ensure no contradictions.
- Verify linked documentation exists and is current.
- **A contradiction between the ticket and its spec, or between the ticket and a sibling, is a
  finding here and a job for `/spec-check`.** Report it — quote both sides, resolve neither — and
  point at `/spec-check` for the rest of the family. This review is bounded to one ticket by
  construction, so it will find the two lines it happened to fetch and miss the other four children
  saying a third thing. Offer it once per review, not per finding.
- **On a UI ticket, check UI2 as a link check first, then as a coverage check.**
  - Is a design linked at all — Figma, designer mockup, or a prototype artifact URL? Nothing linked is a
    UI2 finding. A mockup **attached** rather than linked is also a finding: it dies with the ticket
    (DB1's reasoning, applied to design).
  - **Say which kind is linked.** A generated prototype satisfies UI2 as something to point at; it is not
    an approved design, and a review that reports it as one has made the gap invisible.
  - **Don't fetch the artifact page to judge it.** Those pages are private to the requester's session, so
    a fetch returns markup or an auth wall — and a finding about a mockup you didn't see is invented.
    What you *can* do, and the `ticket-reviewer` subagent cannot, is ask: "open the mock and tell me
    whether AC 3 is what you see." Do that when the AC and the mock's stated scope look like they
    disagree.
  - With a prototype linked, **UI1/UI3** get sharper — does the ticket still state desktop **and** mobile,
    and which viewports the mock covers? — and **UI7** gains a failure mode: the mock is hand-written
    HTML/CSS, the product is built with the component library in profile § 11, and a ticket reading as
    though the mock's markup is the implementation is a finding.
  - **UI2 unsatisfied and the layout ambiguous from the text alone?** Offer `/mock` alongside the
    rewrite offers — once, and only when both are true (full criteria in `../../references/prototype.md`).
    A review is not a licence to build one unasked, and on a batch audit this collapses to a single
    aggregate line rather than firing per ticket.
- **If a repo is checked out, check the ticket against it** — `../../references/repo-context.md`.
  The findings worth the lookup: a name in the ticket that doesn't exist in the code
  (`InstCode` vs `InstanceCode`), a "current behaviour" claim the docs contradict, a
  documented constraint that should be an acceptance criterion, or a QA ticket duplicating an
  existing suite. Cite the file path. No repo, or nothing documented on this feature, is not a
  finding — say nothing rather than manufacturing one.
- Check that acceptance criteria are testable statements, not prose.
- **Check requirements altitude (U15).** Any requirement naming a table, column, class, service,
  endpoint or file path is a finding: quote it and propose the business rewrite. Check it against the
  four exceptions first (`ticket-schema.md` § *Where exact values still belong*) — a Task's
  `InstanceCode = ACME01` and a Bug's user ID are contracts, not leaked design.
- **You are reviewing a ticket, not a draft, so faithfulness (U17) is usually not checkable here** —
  there's no source material to trace requirements back to. Say that rather than implying the scope
  was verified. If the requester pastes the original notes or request, run it: a requirement tracing
  to nothing anybody asked for is worth more than most findings. A `(derived — confirm)` tag still
  on a created ticket is **two findings at once**: a question the requester never answered — surface
  it to resolve or strike — *and* a U18a scaffolding leak, since the tag should have been resolved
  before the create. Report the open question first; it's the one with consequences for the build.
- **If the ticket's source wasn't English, check U16** — UI labels, menu paths, field names and error
  text should appear verbatim in the original with the English rendering alongside. A translated-only
  label can't be searched for and no longer matches the product.
- Verify parent epic is assigned (U5) — orphan stories are a finding, not a detail.
- Verify labels come from the profile's vocabulary and that no label was coined (U6). No product
  label unless the profile says one is used — the project key already says which product it is.
- Verify the assignee is set and plausible for the work (U13). On per-environment variant sets,
  check the owners actually vary by environment rather than all pointing at one person.
- Verify the issue type is current practice (U14) — a type the profile lists as dormant is a
  finding worth raising.
- Verify the summary follows the profile's prefix convention (U1, U2) and the project is right for
  the work (P1–P3).
- Verify priority looks deliberate against the urgency language in the description (U10).
- On anything created through the fast path (`ai-draft` label, `quick-ticket`), pay extra
  attention to what that path deliberately skips: linked Confluence doc (U3), duplicates (U4)
  and parent epic (U5).

**Step E — Deliver findings.** Present in three categories:

**Missing** — required fields or dependency items not present.
Include exactly what needs to be added.

**Needs improvement** — present but unclear, untestable, or inconsistent.
Suggest specific rewrites, not just "fix this."

**Good** — what's already solid. Don't skip this — it tells the requester what not to touch.

Quote the specific rule ID (`U8`, `RL2`, `IM1`…) behind each finding so the requester can see it came
from the matrix and not from an opinion. Then offer to apply the fixes — edits go through the
Atlassian connector only after the requester approves the specific rewrite, same rule as drafting.
For anything needing a brand-new ticket, hand off to `ticket-create` (or `quick-ticket` if it's a
simple one-liner).

---

## Workflow: BATCH AUDIT

When asked to review multiple tickets (e.g., "check all stories in epic PROJ-456"):

1. Fetch the epic and all child stories from Jira.
2. Run the single-ticket review on each.
3. Summarize findings as a table: one row per ticket, columns for Missing / Needs improvement / Good.
4. Highlight systemic patterns (e.g., "4 out of 6 stories are missing mobile scope").

**A batch audit is n single-ticket reviews, and that is a different question from whether the set
agrees with itself.** Each child can be individually well-formed while two of them state the same
rule two different ways, and nothing in steps 1–4 will notice: the reviews never compare children to
each other or to the requirement they came from. That comparison is `/spec-check` (`spec-flow`) —
offer it alongside the summary table whenever the epic has a linked spec page.

---

## Workflow: CHECK the linked Confluence doc — and update it if asked

Step D says "verify linked documentation exists and is current". Doing that honestly means opening
the page, not confirming the link resolves.

**Ask first.** Opening the page is a Confluence call, so it's gated like every other one: once per
session, before the first call, ask whether to use Confluence — **Search for context** ·
**Search and write** · **Skip Confluence** (`../../references/confluence.md`). Carry the answer; don't
re-ask per ticket, and don't ask at all if the connector isn't authorized — say so and review without
it.

If they skip, U3 is reported as `⚠️ linked doc not checked — Confluence skipped`, not as a passing
check and not as "no doc exists". A review that quietly downgrades an unchecked item to a pass is
worse than one that admits its scope.

**Reading** (where a doc is linked and Confluence is in scope): fetch the page, then report as
findings —

- **U3** — no linked page at all, or a link that 404s.
- The page contradicts the ticket. Quote both sides; don't decide which is right. Whether the spec
  or the ticket is stale is the requester's call, and often the team's.
- The ticket's requirements aren't in the page's Requirements section, so QA reading the spec would
  miss them.
- Open Questions on the page that this ticket's existence answers, still marked Open.
- Open Questions with no owner (template rule 4), or Change Log untouched while the body clearly moved.

**Writing** — needs the "search and write" answer *and* a request to fix the doc. Never rolled into
"apply the fixes" for the ticket. Read `../../references/confluence.md` first; the rules that matter most on a review
are the update rules:

1. Read the page and keep its **version number**; `updateConfluencePage` needs the version you based
   the edit on.
2. Change only the sections the finding is about. Carry everything else through verbatim — you are
   editing someone else's page, and a review is the worst possible excuse for a rewrite.
3. Show a per-section change summary and wait for explicit approval.
4. Add a Change Log row. Read back and report the new version.
5. If the page moved under you between read and write, stop and re-draft against the current version.

When the finding is a question rather than a correction — "is Scenario 3 still valid after
PROJ-123?" — a Confluence comment with an owner beats an edit, and beats a silent finding nobody
follows up.

---

## Workflow: CHECK for blockers and stale questions

When asked to scan for blockers:

1. Search recent tickets in the specified project for:
   - Tickets with unanswered questions older than 2 days
   - Tickets with "Blocked" status and no recent activity
   - Tickets with unresolved comments
2. For each, draft a reminder identifying who needs to respond and what the question is.
3. Present the list for the requester to send or discard each reminder.

---

## Reference files

Paths are relative to this `SKILL.md`. Both files live **one level up, in the plugin's own
`references/` directory** — the same copies `ticket-create` drafts from, so a review can never be
run against a stale duplicate of the standard.

| File | When to read |
|------|-------------|
| `../../references/project-profile.md` | Once per session, before the first review — the conventions every check is judged against |
| `../../references/ticket-schema.md` | Every review — what applies to all types |
| `../../references/ticket-types.md` | The body template for the type under review. **One section, not the file** |
| `../../references/voice.md` | Every review — U18a (drafting scaffolding left in the description) is a pure text check on what you fetched; U18b (register) is capped at two findings and never blocks |
| `../../references/dependency-matrix.md` | Every review |
| `../../references/confluence.md` | When you open the linked page, and always before editing or commenting on one |
| `../../references/doc-template.md` | When checking a page's structure, or drafting a section for one |
| `../../references/repo-context.md` | When a repo is checked out and the ticket makes claims about names or current behaviour |
| `../../references/prototype.md` | When a UI2 finding could be fixed by building a prototype, or when a ticket already links one and you're checking what it must carry alongside it (viewports, component-library mapping, token provenance) |
