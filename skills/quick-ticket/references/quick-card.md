# Quick Card

Everything the fast path needs, on one page. Distilled from the plugin's shared
`references/ticket-schema.md` and `references/dependency-matrix.md` — rule IDs point back there,
and those two files are the source of truth this card must follow. If a question isn't answered here,
that's the signal to escalate to `ticket-create`, not to go read the big files.

**The values come from `references/project-profile.md`** — project keys, prefixes, types, priorities,
labels. The fast path reads the profile alongside this card; the card says what to do with the values.
A blank in the profile is never filled by guessing: discover it through the connector or ask, folded
into the one question Step 1 already asks.

## Project (P1–P4)

Pick from profile § 2. Profile table empty → list visible projects through the connector and offer the
likeliest four in Step 1. An alias that isn't a real key routes per the profile's alias table (P2).

**Restricted boards are code-free** — if the work needs code, QA or a deploy, it needs a linked ticket
in the main delivery project too, which means escalate (P3, P4).

## Summary prefix (U1, U2)

Profile § 3. A prefix convention if one is defined, plus any per-project variant; otherwise a plain
summary. Under ~80 chars, readable without opening the ticket.

## Type

| Kind | Use when |
|------|----------|
| `Story` | New capability for a user |
| `Task` | Concrete work with exact values — setup, config, scripts, data |
| `Bug` | Something that already works is broken |
| `Improvement` | Change to something that exists and works — prefer over Story when Story overstates it |
| `Research` | Investigate / POC, output is documented findings |
| `Epic` | → escalate to `ticket-create` |

Map the kind to the real issue type via profile § 4 — an `Improvement` on a project with no such type
is filed as the fallback it names. Never a type the profile lists as dormant (U14).

## Priority (U10)

Profile § 5 — the scheme and the signal mapping. Top of the scheme for production down / blocking a
release / data loss; high for customer-requested / needed this sprint / a committed date; bottom for
nice-to-have. **No urgency language at all → the profile's default, marked `[defaulted]`.**

## Assignee (U13) and labels (U6, U7)

**Assignee** — the person running the conversation, resolved from the connector's current-user
lookup (`atlassianUserInfo`). Never ask for an account ID, never hard-code one; if the lookup
fails, say so and leave it unset.

**Labels** — only from profile § 6, plus **`ai-draft` on everything this path creates**. Never coin a
label; with no vocabulary, add only `ai-draft` and what the requester names. No product label unless
the profile uses one. Components per the profile — empty by default.

## Minimum research

Parent epic read · duplicate search run · **if the ticket is UI, the component file opened** (profile
§ 11 says where components live). That's the floor. Say in the draft footer what you did and didn't
read — "not read" is information, silence is a claim. Epic has Done research children? Escalate: their
findings need reading (RS5).

## Bodies

Requirements and Acceptance Criteria are always **two separate sections**, both numbered.
Requirements = what must exist. AC = testable pass/fail lines, never prose (U8, U9).

**Requirements are business behaviour, not implementation (U15).** Name the user and the rule; the dev
picks the how. Exceptions: Task config values, Bug identifiers, upload columns, notification tokens.

**Mark what the request didn't say (U17).** Scope you inferred — the other direction of a rule, a
boundary left open — goes in tagged `(derived — confirm)`, never silently. Untagged, it reads like
something the requester asked for.

**Source not in English (U16).** Draft in English, but keep UI labels, menu paths, field names, error
text and quoted customer wording **verbatim in the original**, English in parentheses:
`Кнопка "Зберегти" ("Save") does nothing`. A translated label can't be searched for and no longer
matches the product. Read urgency by meaning, not by English keyword.

**Story / Improvement**
```
[Who it's for and what they get. "As a … I want … so that …" or plain prose.
 For Improvement: current behaviour first, then desired.]

REQUIREMENTS
1. …

ACCEPTANCE CRITERIA
1. …
2. [isolation — what must stay untouched: the other tenant, the other screens]
3. [regression — which existing behaviour is unaffected, named]
```

**Task** — same shape; the Requirements carry **exact values** (`InstanceCode = ACME01`,
`Instance ID: 44`, `Run NewTenantSetup.ps1`), not descriptions of them.

**Bug** — `Preconditions` (role, toggle, data state — tenant too, if multi-tenant) / `Steps to
reproduce` (numbered, with real identifiers — user ID, GUID, order number) / `Actual result` (error
text verbatim) / `Expected result`, then AC. Name the environment **and** the host (profile § 9 host
pattern) — "on QA" is not enough (BG4).

## Voice (U18)

**The draft is not the ticket (U18a).** `⚠️`, `[defaulted]`, `(derived — confirm)`, the
`Read`/`Not read` footer and the field block stay in the chat — a description carrying check marks
tells everyone who opens it that nobody re-read the ticket. Stripping them is step 0 below.

**The prose (U18b).** Length matches the work — a data request can be one complete sentence, and six
requirements on a two-line change is padding. Isolation/regression AC name *what* breaks ("the other
three reports still show their own headers"), not "existing behaviour is unaffected". Keep the
product's and requester's own words. Drop the generated register: "ensure", "leverage",
"comprehensive", "in order to", "This ticket aims to", the summary restated as the first line, a
third bullet the source didn't give you. Full page is `../../references/voice.md` — **not read
here**; this is the fast-path version of it.

## Creating via the connector

0. **Strip the draft markers** and resolve every `(derived — confirm)`: confirmed → drop the tag;
   struck → drop the line; still open → keep the requirement plainly worded and put the question in
   a Jira comment after the create (U18a).
1. First create in a project this session → `getJiraIssueTypeMetaWithFields` to find the epic
   field (`parent` first; a company-managed project may need the Epic Link custom field instead) and
   the allowed issue types and priorities.
2. Description goes as **ADF**, not raw markdown — headings and numbered lists must be real
   structure, not literal `1.` characters in one flat paragraph.
3. Create.
4. **Read the issue back.** Confirm type, parent, priority, assignee and labels landed. Report
   the real key and URL — never a predicted one.
5. Parent didn't take? Follow-up edit. Never leave a story orphaned (U5).
6. **Spawn the `ticket-reviewer` subagent on the real key** and show its findings — a different
   model runs the schema and matrix checks this path skipped. Pass it the requester's **request
   verbatim** along with the key, plus **any line you derived and how it resolved** — the tags are
   gone from the ticket by then, so that list is the only way U17 can still run. See the SKILL for
   how to spawn it.

## Escalate to `ticket-create`

Epic or children · variant/bulk set · dev research notes · Confluence page needed ·
restricted-board item needing code, QA or deploy · 2+ unresolved `⚠️` · work under an epic with
**Done research children** — their findings need reading, and that's a full pass (RS5).
