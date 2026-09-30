# Dependency Matrix

This is the "must-not-forget" checklist. Every ticket is checked against this matrix
before it's shown to the requester.

> **TO THE TEAM**: This file is the highest-leverage artifact in the whole system.
> Update it whenever you discover a new "we always forget to check X" pattern.
> Each row you add here gets enforced automatically on every future ticket.

**Project-specific values come from `project-profile.md`.** Rows below that depend on your
conventions — project keys, prefixes, labels, priority names, environments, tenancy, the UI library —
say "per the profile" and point at the section. When the profile leaves a value blank, the row follows
the profile's § *How blanks are handled*: discover, else ask once, never invent.

The rows ship as **generic defaults**: each encodes a recurring miss common to most product teams.
None of them cites a ticket from your boards yet. When a row fires on real work, cite the ticket or
Confluence page next to it (see § *How to update this matrix*) — a cited row is a rule, an uncited one
is a starting point. Delete rows that don't apply to your product rather than letting them fire as
noise.

---

## 0. Project routing

Getting the project wrong is the most expensive mistake here, because restricted boards usually have
a different workflow and often no QA/release path at all.

| # | Check | Details |
|---|-------|---------|
| P1 | Correct project key | The key is one of the projects in `project-profile.md` § 2. If that table is empty, list visible projects through the connector and ask — never guess a key from the product's name. |
| P2 | Aliases are not keys | A product or team name people say as if it were a project key is routed per the alias table in profile § 2. Never create a ticket in a key that returned no project. |
| P3 | Restricted boards stay restricted | A board the profile marks as restricted (data-only, ops-only, support) does not carry code work. If the work needs code changes, QA testing or a deployment, create the ticket in the **main delivery project** and link it to the restricted-board ticket. |
| P4 | Cross-project link exists | When a ticket on one board spawns a ticket on another (P3, or any sibling project), the link must actually be created, not just mentioned in the description. |

## Universal checks (all ticket types)

| # | Check | Details |
|---|-------|---------|
| U1 | Summary prefix convention | Apply the discipline prefix from profile § 3. If the profile says `none`, write a plain summary with no prefix — don't import a convention from elsewhere. |
| U2 | Project-specific summary variant | Apply any per-project variant listed in profile § 3 (e.g. environment first on an ops board, client first on a data board). None listed → nothing to check. |
| U3 | Confluence doc linked | Every story and epic should link to a Confluence spec page. If none exists, flag it. Confluence use is asked for once per session (see `confluence.md`) — if the requester skipped it, or the connector isn't authorized, this check reports `⚠️ not checked — Confluence out of scope`. Never report an unchecked doc as a missing one: "we didn't look" and "it isn't there" are different findings, and only one of them is a reason to write a spec. |
| U4 | No duplicate exists | Search Jira before creating. Check sibling projects too — the same work often exists in the main delivery project, linked from a restricted board. |
| U5 | Parent epic assigned | When profile § 7 says stories need an epic (the default), every story belongs to one. Orphan stories are a finding. |
| U6 | Labels use the real vocabulary | Only labels from profile § 6, plus the AI-drafted marker on anything AI-drafted and the not-testable marker where QA has ruled work out of automation. **Never coin a label.** No product label unless the profile says one is used. Empty vocabulary → only `ai-draft` and labels the requester names or siblings already carry. |
| U7 | Components per the profile | Components are left empty unless profile § 6 lists live ones. Don't invent them. |
| U8 | Requirements ≠ Acceptance Criteria | Two separate sections. Requirements = what must exist ("Support both desktop and mobile layouts"). AC = testable pass/fail ("The order list renders without horizontal scroll at 375px"). Don't merge them. |
| U9 | AC is testable | Each acceptance criterion is a verifiable yes/no statement, not prose. |
| U10 | Priority set deliberately | Use the scheme and signal mapping in profile § 5. Map from the source's language — not its English keywords (U16). No urgency signal at all → the profile's default value *and say that it was defaulted* — never silently pick one. |
| U11 | Tenant named if tenant-specific | Multi-tenant products only (profile § 10). If behaviour differs per tenant, name the tenant explicitly, using the profile's word for it. "For the tenant" is not enough. Single-tenant → skip. |
| U12 | Closure semantics respected | Only when profile § 8 defines statuses that mean different things (e.g. one that requires a release and one that doesn't). Never plan to close something through a status that skips a release it still needs. |
| U13 | Assignee resolved, not asked for | Defaults to the person running the conversation, resolved from the connector's authenticated session (current-user lookup). Never ask them to paste an account ID; never invent one. On per-environment variant sets, vary the assignee with the environment rather than copying one owner across all. A person named in the description ("this ticket is for @…") belongs in the Assignee field too. |
| U14 | Dormant issue types avoided | Types profile § 4 lists as dormant are configured on the board but are not current practice. Don't draft into one unless the requester explicitly asks — route the work where the profile says. |
| U15 | Requirements are business behaviour | A requirement names the user, the rule and the states — **not** tables, columns, classes, endpoints or file paths. The developer picks the implementation. Exceptions are listed in `ticket-schema.md` § *Where exact values still belong*: task config values, bug identifiers, IM1 upload schemas, NT1 event and token lists — those four are contracts, not design. A new role, permission or setting is stated plainly with a precedent ticket ("add a Team Lead role who sees only their own team's data; see PROJ-xxxx for the last role we added"), never specified structurally. A requirement a BA or QA analyst cannot read is a finding. |
| U16 | Original-language strings preserved | A source that isn't in English is drafted in English, but every string someone will **search for or type** stays verbatim in the original with the English rendering in parentheses — UI labels, menu paths, field/column names, error text, notification subjects, quoted client wording. A translated label is unsearchable and no longer matches the product (same reasoning as BG3 and IM1). Read urgency by meaning, not by English keyword: "горить" is not "no urgency signal", and `[defaulted]` won't catch that mistake. See `ticket-schema.md` § *When the source isn't in English*. |
| U17 | Derived scope is marked, not smuggled | A requirement or AC the source doesn't state — the other direction of a one-way rule, a boundary left open, a cascade the matrix asks for — is **included and tagged `(derived — confirm)`**, and raised in the questions list. Never added silently, never quietly dropped. Scope tracing to neither the source nor cited codebase evidence is invented, and it's the finding a drafter is least able to catch in its own text: the invented line reads exactly like a real requirement. The reviewer is given the source material precisely so this check can run (`agents/ticket-reviewer.md`). See `ticket-schema.md` § *Writing Requirements* rule 5. |
| U18 | Ticket text, not drafting scaffolding | **Two parts, different severities.** **(a) The created description carries none of the drafting apparatus.** `RESEARCH`, `Not read`, `DEPENDENCY CHECKS`, `⚠️`, `[defaulted]` and `(derived — confirm)` belong to the draft the requester approves — never to the Jira description, and never as description text restating fields Jira already holds. An unresolved derived line at create time becomes a plainly-worded requirement **plus a Jira comment** carrying the question, not a tag left in the body (`ticket-schema.md` § *What goes into Jira, and what stays in the chat*). This is a real defect: a description full of check marks tells every reader the ticket was generated and never re-read. **(b) The prose reads as though a colleague wrote it** — length matched to the size of the work, isolation and regression criteria naming what actually breaks instead of repeating one sentence, the product's and requester's own words kept, and none of the generated-text register. Full guidance in **`voice.md`**. (a) is MISSING; (b) is NEEDS IMPROVEMENT and never blocks a ticket. Neither licenses vague AC (U9), dropped checks, or unmarked derived scope (U17). |

## UI/UX tickets

| # | Check | Details |
|---|-------|---------|
| UI1 | Desktop + Mobile scope | Always confirm and document both. Never assume "desktop only." Mobile has its own defect surface — a control that exists at one width and not the other is a classic miss. |
| UI2 | Prototype/design linked | Link the design. **A Figma or designer-produced mockup is the best answer and always wins** — this is a link check, not a build request. If none exists *and* the layout is ambiguous from the ticket text alone, the plugin can produce one: the `prototype` skill builds a standalone HTML/CSS mock from the product's own design tokens, publishes it as a Claude Code Artifact, and the ticket links that URL. **Offered, never built silently** (criteria in `prototype.md`; the gate is `ticket-create` Step D). A generated mock is a thing to point at during refinement — it is **not an approved design**, and the ticket must say which kind it carries. It is also not production markup: cross-check UI7 so nobody reads the mock's HTML as the implementation. Either kind goes on the ticket as a **link to a durable location, never an attached file** — DB1's reasoning. (BG6 bug evidence and NT2 templates still attach: a screenshot of a past state is a snapshot, a mockup is a living artifact.) Offer declined and still nothing linked → flag UI2 unsatisfied and say the offer was declined. |
| UI3 | Responsive breakpoints | Specify behaviour at key breakpoints if design differs. |
| UI4 | Accessibility | Note WCAG level, screen reader, keyboard nav requirements, and whether they are verified manually or by automation. |
| UI5 | Browser support | Specify if behaviour differs across browsers. |
| UI6 | Which UI, and for whom | Where profile § 10 lists UI variants (a legacy and a new UI live at once, gated per tenant or by a flag), state which variant the ticket targets and who has it. No variants listed → skip. |
| UI7 | Component library impact | Name the components from the library in profile § 11 that implement the change, and flag when it touches **shared** components — a change there, or a library version migration, usually triggers retests well beyond this screen. Library unknown and no repo to infer it from → `⚠️ component library unknown`. |

## Notification tickets

| # | Check | Details |
|---|-------|---------|
| NT1 | Event name + full token list | Every new or changed notification lists its exact event name and every token, spelled as implemented — e.g. `Order Shipped Final Reminder` with `CustomerName \| OrderNumber \| ShipDate \| TrackingLink`. A notification ticket without its token list is not ready. |
| NT2 | Template attached + default/custom behaviour | Attach the template. If admins can switch between a default and a custom template, state what happens on the switch and whether an uploaded custom file survives it. |
| NT3 | Channels in scope | Say which of the channels in profile § 14 are in scope. They fail independently. |
| NT4 | Who must *not* receive it | Suppression rules are the most common notification defect: recipients getting notifications about content they can't see, or didn't author. Specify the negative case, not just the positive one. |
| NT5 | Trigger and timing | When it fires, and any offset ("1 day before the due date"), plus the condition that suppresses it ("only if no reply was submitted yet"). |
| NT6 | Localization | Per-language template/label behaviour, and fallback when no custom value is set. |

## Role / permission / feature-toggle tickets

Anything where turning one thing on or off affects another thing. This is where "we only
specified the happy path" costs the most, because the missing direction ships as a bug.

| # | Check | Details |
|---|-------|---------|
| RL1 | Every toggle direction stated separately | For prerequisite or bundling rules between roles, permissions or fields, spell out **each** direction as its own requirement: "A ON forces B ON", "B OFF forces A OFF", "A OFF leaves B unchanged". One bullet saying the fields are "linked" is not a specification — at least one direction will be implemented wrong. |
| RL2 | The invalid state that must never persist | Name the specific combination that has to be rejected — not "validate the selection". Then carry it into AC as a negative check ("combination X can never be saved"), not just the happy-path case. |
| RL3 | Cascade on removal, not just on grant | State what happens to dependent roles/permissions/fields when the parent is revoked, and whether the dependent is reverted, retained, or blocks the revoke. |
| RL4 | Who can set it | Which admin role can grant or change this, and whether a user can hold it for themselves. For report permissions, cross-check RP2. |
| RL5 | Effect on existing users | Whether the rule applies retroactively to users who already hold the old combination, and what migrates them. |

## Data import / mapping / upload tickets

Covers user files, mapping sheets, uploads and any multi-tab spreadsheet. Distinct from the
Data/DB section below, which is about where scripts live — this is about the payload's shape.

| # | Check | Details |
|---|-------|---------|
| IM1 | Exact column/field schema | Every column named as it appears in the file, with type and whether it's required. "The user file columns" is not a schema. Reference the attached template by name. |
| IM2 | Cross-tab / cross-entity processing order | When the file has multiple tabs or references other entities, state what must exist or be validated **first** (e.g. the group must exist before the row assigning users to it). Getting the order wrong produces partial imports that look successful. |
| IM3 | Row-level example | Give one concrete row and what it should produce — *e.g. "a single row assigns the group to one user"*. This is the cheapest ambiguity killer in the whole matrix. |
| IM4 | Invalid-row behaviour | Whether a bad row fails the whole file or is skipped and reported, and where the error surfaces (upload history, email, on-screen). |
| IM5 | Duplicate and re-upload behaviour | What happens when the same file or key is uploaded twice — insert, update, or reject. |
| IM6 | Optional columns declared | Which optional/additional columns are in use for this import (and for which tenant, where that varies). |

## Report / Data Export tickets

| # | Check | Details |
|---|-------|---------|
| RP1 | Filters, columns, parameters enumerated | List every filter with its control type and every parameter value. Missing filters and wrong dropdown options are a recurring bug class. |
| RP2 | Report access rights | Say who can see, run and download the report, and whether it appears on whatever screen manages report permissions. |
| RP3 | Every data-source / toggle state | If a feature toggle or configuration switches the report between data sources or behaviours, state the expected result with the toggle **ON and OFF** — they diverge. |
| RP4 | Zero / negative / boundary filter values | Specify behaviour at 0 and negative values. Validation wrongly blocking `0` on a numeric filter is a repeat defect. |
| RP5 | Export format and exact headers | For downloadable reports, state the format and the literal column headers. Header wording changes are their own tickets. |
| RP6 | Embedded BI / dashboard specifics | For BI-tool or dashboard work also cover: the display name and where it's configured, whether export is available, and visual sizing/scroll behaviour. |
| RP7 | Cross-tenant isolation | Multi-tenant products only (profile § 10). Add an explicit AC that the change affects only the intended tenant — "no cross-tenant leakage". |

## Transactions / balances / money tickets

Anything that moves value — money, points, credits, stock, quotas — or advances a record through an
approval flow.

| # | Check | Details |
|---|-------|---------|
| TX1 | Reversal / cancellation handling | Any ticket adding or modifying a transaction type must address what happens on reversal, refund or cancellation. |
| TX2 | Full state machine | Cover every state, not just the happy path: submitted, submitted-for-review, approved, rejected, and the resulting status of each. Records stuck in an intermediate state after approval are a classic defect. |
| TX3 | Edge case: zero/negative | Specify behaviour when the amount is zero or negative. |
| TX4 | Audit trail | Confirm logging/audit requirements for financial or value-moving transactions. |
| TX5 | Rounding rules | Specify rounding behaviour if calculations are involved. |
| TX6 | Where the value lands | For anything touching balances, say which account, wallet or ledger — and what the default is when the relevant setting is off. |

## Tenant / Environment setup tickets

| # | Check | Details |
|---|-------|---------|
| TE1 | Real environment names | Use the environment names in profile § 9 exactly. "Staging" and "STG" are different strings in a search. Names blank → ask. |
| TE2 | Per-environment or single story | Each setup story is either **one story per environment** (environment-prefixed title) or **one story covering all environments**. State which — getting it wrong silently drops an environment. |
| TE3 | Specialist team required? | Flag per story whether another team (data, ops, security) must be looped in before it starts. |
| TE4 | Identifiers supplied | Every environment-specific identifier profile § 9 lists (tenant ID, database name, client ID…). Flag any not yet assigned rather than guessing — some genuinely don't exist until an earlier setup story runs. |
| TE5 | URLs per environment | Per environment, following the host pattern in profile § 9. A customer-facing production URL is often customised and must be finalised with the customer before the story starts — call that out as a blocker. |
| TE6 | Rollback plan | Describe rollback steps if setup fails. |
| TE7 | Other tenants untouched | Multi-tenant products only. Add an explicit AC that no other tenant's instance is modified. |

## Data / DB tickets

| # | Check | Details |
|---|-------|---------|
| DB1 | Scripts go in the repo, not the ticket | **Do not attach custom scripts to a Jira ticket.** They get lost once the ticket closes. |
| DB2 | Correct script location | Where profile § 12 names a location for data/migration scripts, the ticket says the script goes there, named by ticket key. |
| DB3 | Ad-hoc data fixes are audited | Data-integrity fixes use the team's ad-hoc template or audit log (profile § 12), so the change is traceable after the fact. None defined → the ticket states how the fix will be recorded. |
| DB4 | Seed/reference data in the repo | Seed, lookup and config data changes are recorded in the versioned seed/migration scripts, not applied by hand on a server. |
| DB5 | Reference IDs are immutable | Once a lookup/seed/reference ID is deployed it never changes, even if that leaves gaps. |
| DB6 | Deletion policy respected | Per profile § 12 — by default, use an inactive flag rather than deleting records. |
| DB7 | Commit message carries the ticket key | So the change can be traced from the code back to the ticket. |
| DB8 | Rolled-back release → script moves | If a release fails or is rolled back, its scripts move to the next release rather than being lost. |

## API / Backend tickets

| # | Check | Details |
|---|-------|---------|
| API1 | Endpoint specification | HTTP method, URL pattern, request/response schema — for a *published* contract. An internal endpoint is U15's business: state the behaviour, let the developer design it. |
| API2 | Auth requirements | Specify authentication/authorization needed. |
| API3 | Error handling | Define expected error responses and codes — including the 500s the ticket is meant to eliminate. |
| API4 | Performance criteria | Specify expected response time or throughput if relevant. |
| API5 | API docs updated | Flag when the change alters a published contract (OpenAPI/Swagger, SDK, partner docs). |

## Integration tickets

| # | Check | Details |
|---|-------|---------|
| INT1 | External system, version, and deprecation date | Name the system, the exact service/action, version, and protocol — plus any vendor deprecation date driving the work. |
| INT2 | Failure handling | What happens when the external system is unavailable. |
| INT3 | Data mapping | Field-by-field mapping between systems, and a documented diff where the new contract differs from the old. |
| INT4 | Credentials and sandbox access | Name what accounts/environments are needed and who provides them — this is the usual blocker. |
| INT5 | Test data available | State how test data will be generated before build starts. |

## Research tickets

| # | Check | Details |
|---|-------|---------|
| RS1 | The decision it must produce | AC describes the answer or recommendation required, not a shippable feature. |
| RS2 | Time box | Research is capped (agree a cap — 16h is a common one); a continuation needs its own Part 2 ticket rather than quietly overrunning. |
| RS3 | Anticipated impediments listed | At estimation, list what needs clearing first (accounts, environment access, vendor support). |
| RS4 | Findings documented + follow-ups created | "Document your findings" is an AC, and the resulting work becomes its own tickets. |
| RS5 | Findings of a Done research ticket are read before drafting the work it gated | A Done spike's **conclusions** are in its comments and attachments; its description is only the brief. Drafting implementation tickets from the brief re-asks questions the team already paid to answer, and contradicts decisions nobody knows were made. Check comments and attachments on **every Done sibling under the epic** before drafting, and name which ones you read (`ticket-create` Step B). |

## QA / Automation

| # | Check | Details |
|---|-------|---------|
| QA1 | Automation-coverage field is accurate | Only if profile § 13 names one. Such fields go stale constantly — check the value against reality. |
| QA2 | Automation task exists, or manual-only is justified | Either an automation task (named per profile § 13) is linked to the epic, or the epic is explicitly marked manual-only with a reason (accessibility, exploratory) — and carries the not-testable marker where the profile defines one. |
| QA3 | Test cases linked | Where profile § 13 names a test management tool, QA test-case work links the case IDs in its format. None → skip. |
| QA4 | Test-case development subtask | Stories carry the test-case subtask named in profile § 13, where the team's process expects one. None → skip. |
| QA5 | Variation axes stated | On UI, notification, report and toggle/permission tickets, the ticket says **which conditions the behaviour varies under** — viewport (UI1), tenant (U11), UI variant (UI6), toggle state (RP3/RL1), channel (NT3), role (RL4/RP2), language (NT6). Silence on an axis the type's own rules demand is a finding, not a default: NT3 says the channels "fail independently" and RP3 says the toggle's states "diverge", so an unstated axis is an untested one. `/test-matrix` builds the grid and reports which cells nothing covers. Don't invent an axis the ticket is silent about and no rule demands — that's what the `⚠️ … unanswered` line is for. |

## Bug tickets

| # | Check | Details |
|---|-------|---------|
| BG1 | Preconditions stated | Feature enabled (and for which tenant, if multi-tenant), user role, data setup. |
| BG2 | Numbered steps to reproduce | With real identifiers where relevant (user ID, record ID, GUID, order number, request ID). |
| BG3 | Actual vs Expected, separately | Both stated explicitly, error messages verbatim. |
| BG4 | Environment and URL | Which environment and which actual host, per the pattern in profile § 9. "On QA" is not enough. |
| BG5 | Tenant named | Multi-tenant products only: which tenant reproduces it. |
| BG6 | Evidence attached | Screenshots or console output. |

---

## How to update this matrix

When the team discovers a new pattern:
1. Identify the ticket type category (or create a new one)
2. Write the check as a clear, actionable statement
3. Add a row with a unique ID (category prefix + number)
4. Include enough detail that Claude can verify the check without asking
5. Cite the ticket or Confluence page it came from, so the next person can tell a real rule
   from a guess
6. If the rule depends on a value that varies by project, put the value in `project-profile.md` and
   have the row point at it — never hard-code it here

**Examples of how new rows get added:**
- "We keep forgetting to specify date format on export tickets" →
  Add row: `EXP1 | Date format specified | Always specify date format (ISO 8601, locale-specific, etc.) for any data export feature.`
- "Multi-language tickets never mention which languages" →
  Add row: `I18N1 | Languages listed | Specify which languages/locales are in scope.`
- A rule only your product needs — a domain entity with its own lifecycle, a setup checklist for a
  particular kind of instance — gets **its own section** with its own prefix, rather than being
  squeezed into a generic one.

## Grounding

The most useful thing a team can do in its first weeks with this plugin is sample its own boards:
pull twenty recent, well-regarded tickets per project, and for each row above ask "does this ever
fire here?" Cite the ones that do, delete the ones that never will, and add the misses nobody wrote
down. A matrix grounded in your own tickets catches your own defects; a generic one catches generic
ones.
