# Ticket types: the per-type templates

The body shape for each **kind** of ticket — what goes in the description and which sections.
`project-profile.md` § 4 maps each kind to the issue type your projects actually use, and § 15 lists
real tickets from your boards to calibrate against, where the team has named some.

Read **one section**, for the type you are drafting. That's why these live here rather than in
`ticket-schema.md`: the schema is read on every ticket, and carrying six templates through it meant
reading five you didn't need. Everything that applies to *every* ticket — the draft preview format,
summary line, priority, assignee, labels, how to write requirements, the field rules and the
connector rules — stayed there.

Which kind to pick, and the dormant types to avoid, are in `ticket-schema.md` § *Issue types in use*.

The `Summary:` lines below show a discipline prefix (`DEV - `, `DB - `) as a placeholder. Use the
convention in profile § 3 — which may be no prefix at all.

---

## Story

Requirements and Acceptance Criteria are **two separate sections**. Requirements describe what
must exist; AC are the pass/fail checks. Both are numbered lists.

```
Summary: DEV - [Clear description of the change]

Description:
  [Opening paragraph: who this is for and what they get. Often written as
   "As a <role>, I want <capability>, so that <benefit>" but plain prose is
   equally common and fine.]

  REQUIREMENTS

  1. [Scope item — what must exist or be built]
  2. [Scope item]
  3. [Include the "not just for X" clauses: "Available for all tenants, not just the one that asked"]

  ACCEPTANCE CRITERIA

  1. [Testable pass/fail statement]
  2. [Testable pass/fail statement]
  3. [Fallback / default behaviour]
  4. [Isolation: change only affects the intended tenant, no cross-tenant leakage]
  5. [Regression: existing behaviour is unaffected]

Priority: [per the profile's scheme]
Labels: [from the profile's vocabulary, ai-draft if applicable]
Parent: [epic key]
```

Given/When/Then is also accepted for AC where it reads better:
`GIVEN a created survey WHEN the report is downloaded THEN the table headers contain 'Correct answer…'`

The last two AC slots — isolation and regression — appear on most well-written stories. Don't drop
them. On a single-tenant product, the isolation slot names whatever else must stay untouched — the
other reports, the other roles, the other screens that share the component.

## Task

```
Summary: [DB / Ops / Dev] - [Verb-first action]

Description:
  [Overview: what this accomplishes and why. 1-3 sentences.
   Name the person it's for if it's assigned work: "Note: This ticket is for @name"]

  REQUIREMENTS

  1. [Concrete step or configuration value]
  2. [Include exact values: IDs, codes, URLs, script names — not descriptions of them]

  ACCEPTANCE CRITERIA

  1. [Verifiable completion criteria]
  2. [Explicit "nothing else was touched" check where relevant]

Priority: [...]
Labels: [...]
Parent: [epic key]
```

For setup tasks, exact values are the whole point. `InstanceCode = ACME01`, `Instance ID: 44`,
`URL for QA - acme.qa.example.com`, `Run NewTenantSetup.ps1` — not "configure the instance code" or
"set up the URLs".

## Bug

```
Summary: [DEV / DB / UIUX] - [Component] - [What actually happens]

Description:
  Preconditions
  [Feature enabled, user role, data state — and which tenant, on a multi-tenant product]

  Steps to reproduce
  1. [Step, with real identifiers — user ID, record ID, GUID, order number, request ID]
  2. [Step]
  3. [Step]

  Actual result
  [What happens. Error messages verbatim.]

  Expected result
  [What should happen]

  [Screenshots / console output attached]

Priority: [...]
Labels: [...]
```

Name the environment and the actual host where it reproduces, per the host pattern in profile § 9 —
`acme.qa.example.com`, not "on QA" — and, on a multi-tenant product, the tenant. "On QA" is not
enough when behaviour is tenant-gated (BG4, BG5).

## Improvement

Same shape as Story, but the opening paragraph states the current behaviour first, then the
desired behaviour. Keep the scope tight — a label-only change should say so explicitly
("Underlying data/values in the column remain unchanged, this is a label-only change") so QA
doesn't go looking for a data fix.

If the project has no `Improvement` type, file it as the fallback profile § 4 names and keep this
shape.

## Research

```
Summary: [DEV / DB] - Research / POC [subject]

Description:
  [What needs evaluating and why it matters now — vendor deprecation date,
   blocked downstream work, etc.]

  Notes:
  * [Who should take this and why, if it matters]
  * [When reviewing and estimating, leave a list of impediments you anticipate
     needing cleared before work can start]

  REQUIREMENTS

  * [Investigate X — Document your findings]
  * [Attempt Y — Document your findings]
  * [Provide a summary of the work needed, answering: <the specific decision>]
```

Research is time-boxed — agree a cap (16h is a common one), and a continuation gets its own
"(Part 2)" ticket rather than quietly overrunning. AC describe the answer the research must
produce, not a shippable feature.

## Epic

A good epic leads with **who's accountable**, then the business case:

```
Summary: [Initiative name] — often bracketed with the phase, e.g.
         "[Supplier Portal Phase 1] Automated order fulfilment"

Description:
  BA - @name (Responsible)
  Devs - @name
  Database - @name
  DevOps - @name
  QA - @name, @name
  Stakeholder - @name (Accountable), @name (Consulted)
  Design - @name, or "not yet provided"

  ## Context
  [What the system does today, for someone who doesn't know this area]

  ## Business Problem
  [What doesn't work or isn't possible right now]

  ## Business Goal
  [What implementing this achieves]

  ## Business Value
  1. [Concrete benefit]
  2. [Concrete benefit]
  3. [Concrete benefit]

  ### Links and references
  [Confluence spec, related epics, design/prototype URL if the epic has one]

Priority: [...]
Parent: [none — epics are top level]
```

Adjust the role lines to the disciplines your team actually has. If the profile names an
automation-coverage field or an automation-task convention (§ 13), also confirm the field is set
truthfully and the task is linked — or that the epic is explicitly manual-only with a reason (QA1,
QA2). Those fields go stale constantly.

---

## A note on restricted boards

Requests on a data-request or ops board (profile § 2) are frequently one or two sentences — *"Please
load the attached user file for the Acme instance and zero out their point balances"* is a complete,
closable ticket. That's normal for ad-hoc requests and you shouldn't inflate them into full stories.

What they *do* still need: the customer or tenant named, the exact records or identifiers involved,
and — where data is being changed — the scripting and audit requirements from the Data section of the
dependency matrix (DB1–DB3).

Setup work on those boards is the exception: a tenant or environment setup is fully specified, often
one ticket per environment (TE2).
