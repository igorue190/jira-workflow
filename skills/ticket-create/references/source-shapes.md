# When the source isn't one simple item

Three workflows, selected by the **shape** classified in `SKILL.md` Step A — the three rows of that
table that aren't "one item". Read the one that matches; a single straightforward ticket needs none
of this.

| Shape | Workflow here |
|-------|---------------|
| **Mixed source** — meeting or call notes, an email thread, a chat dump | *SPLIT a mixed source* |
| **Variant set** — near-identical work with one thing varying | *GENERATE VARIANTS from a pattern* |
| **Dev research notes** — a developer's free-text next-steps | *CONVERT developer research notes* |

All three end the same way they would in `SKILL.md`: nothing is created without explicit approval,
every drafted ticket runs Steps B–G — the research gate included — and the second-model reviewer runs
on what lands.

---

## Workflow: SPLIT a mixed source (meeting notes, a call, an email thread)

Notes from a call are not a ticket and not an epic — they're several unrelated items sharing a
timestamp. Drafting straight from them produces tickets for things the team decided *not* to do, and
misses the one line that mattered.

**1. List the items first. Draft nothing.** One line each — proposed project, type, and the scope in a
half-sentence. This list is the whole turn's output: no research gate yet, no `DRAFT` blocks, no
connector calls beyond what you need to name a project.

```
FROM THIS SOURCE — confirm before I draft
1. PROJ   · Bug         · order modal overflows below 375px on the orders page
2. PROJ   · Improvement · rename the "Amount Granted" column to "Initial Amount"
3. DATA   · Task        · zero out Acme's archived-user balances for July
4. OPS    · Research    · why the nightly order sync retries twice for one tenant

Not drafted — say the word if any of these should be:
- decision only: team agreed to stay on the current UI library version until Q4
- someone else's action: the designer to send the approval-modal Figma
- unclear whether it's work at all: "the reports page feels slow lately"
```

**2. Stop for confirmation.** They confirm, drop, merge or re-scope. Only confirmed items are
researched and drafted — a dropped item costs nothing once it's dropped, and a Jira search on it is
work nobody asked for.

**3. The `Not drafted` block is not optional.** Everything in the source that you read and deliberately
did not turn into a ticket goes there: decisions, FYIs, other people's actions, things too vague to
scope. Same instrument as the `RESEARCH` block's `Not read` line — a reader who can see what you
skipped can correct you, and a reader who can't reads silence as coverage. It's also where a "we
already discussed this last week" item surfaces before anyone spends a research pass on it.

**4. Never invent an umbrella epic.** Four unrelated items from one meeting are four tickets, not an
epic called "Sprint 42 call follow-ups". Propose a parent only when the source names one, or when the
items genuinely are children of a single outcome — and then say which outcome and why. Each confirmed
item still needs its own real parent epic (U5); "they came from the same call" is not one.

**5. Then run the normal flow per confirmed item** — Step B onward. Report the research gate once for
what the items share (the epic, the Confluence page, the repo) and per item where they diverge. Items
that span projects get one ticket per project plus the actual cross-project link (P4), and an item on
a restricted board that turns out to need code, QA or a deploy spawns its linked ticket in the main
delivery project (P3).

**6. More than about six confirmed items** — draft them in batches, checking in between, rather than
compressing every ticket's sections to fit one response. Say which batch you're on.

If the source isn't in English, the item list is in English too — but keep the original strings per
`ticket-schema.md` § *When the source isn't in English*, because those are what make the item findable
later.

---

## Workflow: GENERATE VARIANTS from a pattern

For repetitive work (same epic × multiple environments, same story × multiple pages):

1. Have the requester describe the pattern once: the base ticket, logic, dependencies.
2. Draft the shared logic, requirements and acceptance criteria **once**, then identify what
   varies (environment name, page name, tenant, assignee).
3. **State explicitly which parts are shared and which vary**, before generating anything — that
   list is what the requester is actually reviewing, and a mistake there is repeated across every variant.
4. Generate all variants by substituting only what differs, running dependency checks on each.
   Keep the body identical; vary the prefix and the assignee (TE2, U13).
5. Present the full batch for review before creating in Jira.

---

## Workflow: CONVERT developer research notes into tickets

When a developer writes up next-steps from a research ticket:

1. Parse each distinct action item from the developer's free-text.
2. Structure each as a proper ticket using the full schema — don't just copy-paste developer's words.
   Restructure into proper description, acceptance criteria, and dependencies.
   Dev notes are written in code vocabulary and often half-remembered. If the repo is checked out,
   resolve every class, endpoint, table, flag and script name against it before it goes in a ticket —
   this is the highest-value use of repo docs in the whole plugin. A name you can't find, and the
   developer's spelling of it, both go in the draft as a `⚠️`.
3. Run the dependency matrix. Add what the developer missed (mobile scope, edge cases, doc links).
4. Present for review before creating anything.
