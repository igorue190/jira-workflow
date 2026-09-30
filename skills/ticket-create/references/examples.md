# Ticket Examples

These are reference examples showing what "good" looks like. Use them as a calibration
tool — your drafts should match this level of clarity and completeness.

**Every example here is illustrative**, written for a fictional multi-tenant order-management
product. The keys (`PROJ-…`, `DATA-…`, `OPS-…`) are placeholders, not real tickets — never cite one
as if it existed. Each example is shaped after a pattern that recurs on real boards, and each
"why this is good" note names the matrix rule it demonstrates.

**Replace them with your own.** A team's best tickets are better calibration than any generic set:
list them in `project-profile.md` § 15, and swap examples here for real ones as you find them (see
§ *Refresh policy*). Where your product is single-tenant, read the tenant lines as "the other
customers / screens / roles that must stay untouched".

The last section, **§ A failure mode, before and after**, is deliberately a *bad* draft next to its
fix: a file of nothing but good examples gives you nothing to calibrate *wrong* against.

---

## Story — custom report name

`DEV - Custom label for the Sales Summary report name`
Parent: PROJ-500 (Reporting for enterprise tenants) · Priority: Medium

```
As a tenant admin managing labels on the Custom Labels page, I want a new custom label
entry that controls the displayed name of the Sales Summary report, so each tenant can
rename it to something meaningful for their organization without a developer changing it.

REQUIREMENTS

1. Add a new row to the Custom Labels page (Home / Custom labels) for the Sales Summary
   report name
2. Include English value, Default language value, and an editable Custom value field,
   consistent with existing rows (e.g. the "Inbox" page title)
3. Available on the Custom Labels page for all tenants, not just the one that asked
4. Saving a custom value updates the displayed report name wherever it's shown to end
   users (e.g. navigation tile, report page title)
5. If no custom value is set, the report falls back to the Default language value,
   consistent with other labels on this page
6. Supports the same per-language custom values as other labels on the page

ACCEPTANCE CRITERIA

1. A new label row for the report name appears on the Custom Labels page for every tenant
2. Entering a custom value and saving updates the report's displayed name for end users on
   that tenant
3. Leaving the custom value blank falls back to the default/English value
4. Setting a custom value for one tenant leaves every other tenant's report name unchanged
5. Existing custom labels on the page keep their saved values after this row is added
```

**Why this is good:**
- Requirements and AC are genuinely different content, not the same list twice
- Req 3 catches the classic trap: work raised by one tenant but built for all of them
- AC 3 covers the fallback, AC 4 the isolation (RP7), AC 5 the regression — the three most often
  missing — and AC 4 and 5 each name *what* stays unchanged (U18b)
- Every AC is a yes/no check QA can run without asking a question

## Story (notifications) — final reminder

`DEV - Add final reminder notifications for unanswered approval requests`
Parent: PROJ-520 (Approval reminders) · Priority: Medium

```
Send a final reminder to approvers who have not yet acted on a pending approval request.
The reminder goes out 1 day before the request's due date, and only if the approver has
not already approved or rejected it.

Requirements

1. Add support for these new notification events:
   1. Approval Request Final Reminder — Manager.
      Tokens: ApproverFirstName | ApproverLastName | RequesterName | RequestTitle |
      RequestLink | DueDate | ShortUrl | PlatformName
   2. Approval Request Final Reminder — Delegate.
      Tokens: DelegateFirstName | RequesterName | RequestTitle | RequestLink | DueDate
2. Trigger the notification 1 day before the request's due date.
3. Only send it if the recipient has not yet approved or rejected the request.
4. Never send a second final reminder for the same request.
5. Respect the existing setting 'Send final reminder for pending approvals'.

Acceptance criteria

1. An approver with a pending request receives the final reminder exactly one day before
   the due date.
2. A delegate with a pending request receives the delegate version of the reminder.
3. An approver who has already acted on the request does not receive it.
4. When the setting is disabled, no final reminders are sent.
5. The existing first reminder still sends 3 days before the due date.
```

**Why this is good:**
- Every event is named exactly as implemented, with its complete token list (NT1). This is the single
  biggest difference between a notification ticket dev can build and one they come back to ask about
- Trigger timing is specific: "1 day before", not "shortly before" (NT5)
- AC 3 and AC 4 are both negative cases — who does *not* get it, and when nothing sends (NT4)
- Req 5 ties the behaviour to an existing setting rather than inventing a new one

## Bug — missing report filters

`DEV\DB - Audit Log report - Add missing filters and fix the access-level options`

```
Pre-conditions:
Audit Log report is enabled for the tenant

Steps to reproduce:
1. Log in to the admin site as a platform admin
2. Open the tenant's admin site → Reports / Run reports
3. Select the Audit Log report
4. Observe the filters

Actual result:
1. These filters are missing:
   * Session created from: datepicker
   * Session created to: datepicker
   * Session status: dropdown Active/Inactive
2. The Access level filter offers Full/None/Read only
   instead of Full/Limited/Both

Expected result:
1. All filters are displayed
2. Access level offers Full/Limited/Both (instead of Full/None/Read only)

Environment: QA — acme.qa.example.com · Tenant: Acme
```

**Why this is good:**
- The precondition states the feature gate — this report doesn't exist for every tenant
- Every missing filter is named with its control type, and the wrong dropdown values are given both
  as-is and as-expected (RP1)
- Actual and Expected are numbered in parallel, so each defect maps to its fix
- Environment, host and tenant are named (BG4, BG5) — a dev can start without a follow-up question

## Bug (suppression) — notification for content the user can't see

`DEV - Requester receives reaction notifications for a private comment they didn't write`

```
Preconditions
An order is visible on the Activity feed. "Comment reaction" notifications are enabled
for the requester.

Steps to reproduce
1. Log in as a regular user (not the requester or the approver)
2. Add a private comment on the order (optionally, a private reply)
3. Add a reaction to that private comment
4. Log in as the requester
5. Open Inbox > Orders
6. Notice the Comment reaction notification from the regular user has arrived
7. Notice the private comment itself is not shown to the requester

Actual result
The requester receives Comment reaction notifications for a private comment they did
not author and cannot see.

Expected result
The requester does not receive reaction notifications for private comments they did not
author.
```

**Why this is good:** the steps need three different roles and the bug only appears in the
interaction between them. Steps 6 and 7 point out the contradiction — the notification arrives for
content the user can't see. That is the actual defect (NT4), and spelling it out saves the dev from
"working as designed."

## Improvement — label-only rename

`DB - Rename "Amount Granted" column to "Initial Amount"` · Labels: `reports`, `ai-draft`

```
On the "Budget Activity by Team Lead" report, the column currently labelled
"Amount Granted" should be renamed to "Initial Amount" to better reflect what the value
represents.

REQUIREMENTS

1. Rename the "Amount Granted" column header to "Initial Amount" on the Budget Activity
   by Team Lead report
2. Underlying data/values in the column remain unchanged — this is a label-only change

ACCEPTANCE CRITERIA

1. The column header reads "Initial Amount" instead of "Amount Granted"
2. Column values are identical before and after the rename
3. No other column headers or layout on the report change
```

**Why this is good:** a two-line change, but Req 2 and AC 2 exist specifically to stop anyone treating
it as a data fix, and AC 3 bounds the blast radius. Small tickets still get scope boundaries — and
they stay small (U18b). Note the labels: a feature area from the vocabulary, and `ai-draft` because it
was AI-drafted.

## Task — instance setup with exact values

`DB - DEV,QA - Acme archived-users instance setup`

```
Archived Acme users need a standalone instance they can access for up to 180 days to
redeem their remaining balance.

Requirements

Please create the following instance for Acme archived users in DEV and QA:

* Peer transfers: off
* Transfer limit: N/A
* Budgets: off
* Manager approval: off
* Optional columns in use:
    * Ship Country Code
    * Termination Date
    * Global User ID
* InstanceCode = ACME-ARCH
* Instance Name = Acme Archived Users
* Catalog = All Acme catalogs
* Emails needed: Welcome, Password reset (templates attached)

Acceptance Criteria
Verify the instance is set up with:
* Instance ID recorded on this ticket
* Budgets off, peer transfers off, manager approval off
* InstanceCode, Instance Name and Catalog as specified
```

**Why this is good:**
- Every feature flag is stated explicitly, including the off ones. "Budgets: off" is information;
  omitting budgets entirely is not
- Exact codes, not descriptions of codes: `InstanceCode = ACME-ARCH` (U15's first exception)
- The access window is in the opening line
- The AC mirror the requirements as a verification checklist, which is exactly what whoever runs the
  setup needs

## Pattern / variant set — one story per environment

One story per environment, same body, environment-prefixed title, different assignees:

```
PROD - DB - Run tenant provisioning scripts for Acme   → <engineer A>
QA   - DB - Run tenant provisioning scripts for Acme   → <engineer B>
STG  - DB - Run tenant provisioning scripts for Acme   → <engineer A>
```

All three share a parent epic ("Tenant setup for Acme") and the same description listing the scripts
to run and what they generate.

**Why this is the right pattern:** these environments get worked at different times by different
people and can fail independently. One story saying "run in all environments" hides which platform is
actually done. When you generate variants, this is the shape — vary the environment prefix and the
assignee, keep the body identical (TE2, U13). Use the environment names from profile § 9, exactly.

## Per-environment verification story — the checklist is the spec

`PROD - QA - Verify tenant setup for Acme`

```
As a QA analyst, I want to verify the following options are available for the new tenant.

Budget
* Public budget
* Private budget
* Budget file upload

Users
* User file processing rules
* User groups
* Role assignments
* User file upload

Content
* Welcome message
* Document library

Administration
* General settings
* Reports
* File upload history
* Notifications

ACCEPTANCE CRITERIA
* Every page above opens with no error
* A change made on the admin site is reflected on the user site
```

**Why this is good:** the checklist is the specification. Nobody has to remember what "verify tenant
setup" covers. A team with a real version of this list should keep it verbatim and reuse it.

## Research — vendor API migration

`DEV - Research / POC: shipping-rates API migration (Part 1)` / `(Part 2)`

```
The vendor's shipping-rates service we call is deprecated as of 30 June. This ticket
evaluates what the migration needs.

| API name       | Action / URI         | Version | Type | Deprecation date |
| Get Rates      | RatesRequest v1      | 1.x     | SOAP | 30 JUN           |

Checkout depends on this call, so it is critical to the site.

Notes:
* Suggest assigning to <dev>, who worked on the legacy shipping integration.
* When estimating, please list the impediments you expect to need cleared before work can
  start (e.g. a new vendor sandbox account, access to the integration environment).

REQUIREMENTS:
* Find the replacement for "Get Rates" (RatesRequest v1.x)
    * Document your findings
* Review the current calls and responses to identify which integration points change
    * Document your findings
* Call the replacement API manually and identify the sequence of calls needed
    * Document your findings
* Summarise the work needed to integrate it, answering:
    * Does this change what the checkout integration is responsible for?
    * Is it as simple as pointing the same integration at a new endpoint?
```

Part 2 opens with: *"Continuation of the research started in Part 1, which exceeded the time cap."*

**Why this is good:**
- The external system is pinned down completely — action, version, protocol, deprecation date (INT1)
- Asking the estimator to list anticipated impediments up front (INT4, RS3) is the single most useful
  line in the ticket
- "Document your findings" as an explicit requirement, and the closing questions define the decision
  the research must produce (RS1)
- Overrun handled by opening a Part 2 rather than letting one ticket run indefinitely (RS2)

## Epic — accountable people first

`[Supplier Portal Phase 1] Automated order fulfilment`

```
BA - <name> (Responsible)
Devs - <name>
Database - <name>
DevOps - <name>
QA - <name>, <name>
Stakeholder - <name> (Accountable), <name> (Consulted)
Design - not yet provided

## Context
The storefront lets users spend their balance on products from several suppliers. Orders
for supplier products are currently placed with the supplier by hand.

## Business Problem
Phase 1 (PROJ-480) imported the supplier catalogue, but a purchase still needs someone
to place the order with the supplier manually.

## Business Goal
Place supplier orders automatically through the supplier's API when a user checks out.

## Business Value
1. Orders reach the supplier in minutes instead of a working day
2. No manual effort placing supplier orders
3. Every fulfilment step is traceable

### Links and references
<Confluence: Supplier order fulfilment>
```

**Why this is good:**
- The accountability block names a person per discipline before anything else, and honestly records
  `Design - not yet provided` instead of leaving a gap
- Context is written for someone who doesn't already know the area
- Business Value is three concrete outcomes, not a mission statement
- Links to both the parent phase epic and the Confluence spec (U3)

## A short request on a restricted board

`ACME - Archived users - July`

```
Please load the attached user file for the Acme archived-users instance and zero out
their balances.
```

**Why this is fine:** one sentence, closed, no drama. A data-request board handles ad-hoc requests,
and inflating them into full stories wastes everyone's time. What it does carry: the customer in the
summary, the specific instance, and both actions required.

**What the matrix still adds:** because this changes data, the script goes in the repo location the
profile names, not attached to the ticket (DB1, DB2), and a fix like zeroing balances goes through the
team's ad-hoc template or audit log so it can be traced later (DB3).

---

## A failure mode, before and after — Requirements at the wrong altitude (U15)

Every example above is a good ticket, which makes them useless for calibrating *wrongness*. This one
is a drafting failure, kept because it is the easiest mistake to make and the hardest to see in your
own text: the requirements are specific, internally consistent, confident — and they specify an
implementation nobody reviewed.

The work: add a set of clickable commands to the AI chatbot, some visible only to managers, each
switchable per client.

**Before — what was drafted**

```
REQUIREMENTS

1. Extend AI.CommandLkp with display label, display order and audience columns.
   Seeded via the post-deployment seed script (DB4), IDs immutable (DB5).
2. New POST /api/ai/command endpoint returning an envelope with text plus an
   optional chart dataset.
3. Register the command→flag mapping in AICommandFeatures.
4. Generalise ActivityStreamSummaryService into a command dispatch service.
```

**Why this is wrong**, even though every line is precise:

- Nobody on the ticket chose `AI.CommandLkp`, a `POST` envelope, or a dispatch service. The drafter
  did, from outside the codebase, and the citations (`DB4`, `DB5`) make a guess look ratified.
- The BA who owns the feature cannot review it. Neither can QA, and QA has to write tests from it.
- The four items describe one design. If the developer finds a better one, the ticket is now wrong
  and someone has to decide whether the ticket or the code is authoritative — a conversation the
  ticket exists to prevent.
- What it *doesn't* say is the part that mattered: who sees which commands, what happens when a
  command is off, what a manager sees that an employee doesn't.

**After — the same work, at the right altitude**

```
REQUIREMENTS

1. The chatbot shows a set of commands the user can click instead of typing.
2. Clicking a command shows an answer in the chat. Some answers include a
   simple chart.
3. Each command can be switched on or off per client. A command that is off
   is not shown and cannot be invoked.
4. Add a new role — Team Lead — who sees the team-level commands; an employee
   sees only their own. See PROJ-xxxx for the last role we added.
5. Every command follows the same path, so adding the next one is cheap.

ACCEPTANCE CRITERIA

1. A user with no commands enabled sees the chatbot with no command chips and
   can still type a question.
2. An employee does not see a team-level command, and cannot invoke one by any
   route.
3. Turning a command off for one client leaves it available for every other
   client (isolation).
4. Typing a free-text question still returns an answer as before (regression).
```

**Why this is right:**

- Req 4 states the new role plainly and points at precedent, rather than specifying how roles are
  stored. That is the whole U15 move: the *decision* stays in the ticket, the *mechanism* goes to
  the developer.
- AC 2 is the invalid state named explicitly (rule 2 of § Writing Requirements) — "cannot invoke
  one by any route" is what stops it shipping as a hidden-but-callable command.
- Req 5 preserves the one genuinely architectural intent from the "before" version — extensibility —
  without naming the class that delivers it.
- Nothing here stops a developer choosing exactly the design in the "before" block. It stops the
  ticket from *pre-deciding* it.

**The exceptions still apply.** If this had been a setup Task, `InstanceCode = ACME-ARCH` would stay
verbatim; if it had been a Bug, the user ID and the host would. See `ticket-schema.md` § *Where exact
values still belong* — those four cases are contracts with someone outside the dev team, which is
exactly what a table name is not.

---

## Refresh policy

Replace an example when your team has a real ticket that demonstrates the same point better — which,
for a generic set like this one, is almost always. When you swap one in, cite the real key and the date
pulled, and add the key to `project-profile.md` § 15. Keep the "why this is good" notes and their rule
IDs: the note is what makes an example calibration rather than decoration.
