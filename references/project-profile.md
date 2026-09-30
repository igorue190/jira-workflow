# Project profile

**This is the one file a team edits to adopt the plugin.** Everything project-specific — which Jira
projects exist, how summaries are prefixed, which labels are real, what the environments are called,
what the UI is built with — lives here and nowhere else. Every skill, and the `ticket-reviewer`
agent, reads this file before it drafts or reviews anything, and every other reference points back
at it by section name.

Fill in what you know. **Leave the rest blank — a blank is safe.** The plugin never invents a value to
fill one: it discovers what it can from Jira through the connector, asks the requester once for the
rest, and carries the answer for the session. The table at the end says exactly how each blank is
handled.

Values below marked `(default)` are generic Jira conventions that work out of the box. Change them
if your boards differ.

---

## 1. Site

| Setting | Value |
|---------|-------|
| Jira / Confluence site | `TODO — e.g. yourcompany.atlassian.net` |
| Team name (used as the byline on prototypes) | `TODO — e.g. Platform delivery team` |
| Product name (what users call the product) | `TODO` |
| Default Confluence space(s) for specs | `TODO — space keys, or blank to ask` |

## 2. Projects

The Jira projects tickets may be created in. **P1** in the dependency matrix checks against this
table.

| Key | Name | What belongs here | Restrictions |
|-----|------|-------------------|--------------|
| `TODO` | | | |

- **Main delivery project** — where code, QA and release work goes by default: `TODO`
- **Restricted boards** — boards that must not carry code work (data-request, ops or support boards).
  Name them in the Restrictions column, e.g. *"data changes only — code work gets a linked ticket in
  the main delivery project"*. **P3** enforces this.
- **Aliases that are not project keys** — product or team names people use as if they were a key.
  Format: `<alias> → <real key>`. **P2** enforces this.

  | Alias | Route to |
  |-------|----------|
  | | |

## 3. Summary convention

- **Discipline prefix** (U1): `none (default)` — or list them, e.g. `DEV - ` · `DB - ` · `QA - ` ·
  `UIUX - ` · `BA - ` · `Ops - `, plus a combined form such as `DEV\DB - `.
- **Per-project variants** (U2): `none (default)` — or e.g. *"ops board: environment first, then
  discipline — `PROD - DB - …`"*, *"data board: client name first"*.
- **Length target**: under ~80 characters.

## 4. Issue types

The templates in `ticket-types.md` are keyed by **kind of work**. Map each kind to the issue type your
projects actually have. Leave a row blank to fall back as shown.

| Kind (template) | Your issue type | Fallback when blank |
|-----------------|-----------------|---------------------|
| Story | `Story` (default) | — |
| Task | `Task` (default) | — |
| Bug | `Bug` (default) | — |
| Improvement | | `Story`, opening with current → desired behaviour |
| Research | | `Task` (or `Spike` if the project has it), with the Research template |
| Epic | `Epic` (default) | — |
| Sub-task | `Sub-task` (default) | — |

- **Dormant types** (U14) — types configured on the board that new work must not go into: `none`
  (default). Format: `<type> — <why, and where the work goes instead>`.
- **Extra types in use** — anything beyond the table, e.g. `Test`, `Incident`: `none`

## 5. Priority scheme

- **Values, highest first** (U10): `Highest · High · Medium · Low · Lowest` (default Jira scheme)
- **Mapping** — which value each urgency signal maps to:

  | Signal in the source | Priority |
  |----------------------|----------|
  | production down, blocking a release, data loss, a broken money path | `Highest` |
  | customer-requested, needed this sprint, a named customer waiting, a committed date | `High` |
  | no urgency language at all | `Medium` — **and mark it `[defaulted]`** |
  | "when there's time", nice-to-have, cosmetic with nobody waiting | `Low` / `Lowest` |

## 6. Labels and components

- **Label vocabulary** (U6) — labels actually in use on your boards: `TODO — e.g. feature areas
  (notifications, reports, ui, security), customer labels`
- **AI-drafted marker**: `ai-draft` (default) — added to every ticket this plugin drafts
- **Not-testable marker**: `none` (default) — e.g. `NotTestable`, if QA uses one
- **Product label**: not used (default) — the project key already carries the product
- **Components** (U7): not used (default) — or list the live components

## 7. Hierarchy

- **Stories need a parent epic** (U5): `yes` (default)
- **Epic link field**: discover per project (default) — `parent` first, the Epic Link custom field on
  company-managed projects that still expect it. See `ticket-schema.md` § *Creating via the connector*.

## 8. Workflow and closure

- **Closure semantics** (U12): `none (default)` — or describe the statuses that mean different things,
  e.g. *"`QA Passed` = needs a staging and production release; `Closed` = needs no deployment and
  drops off the board at sprint end"*.

## 9. Environments

- **Environment names** (TE1): `TODO — e.g. DEV · QA · STAGING · PROD` — use exactly the names your
  team uses; "Staging" and "STG" are different strings in a search
- **Host pattern per environment** (BG4, TE5): `TODO — e.g. <tenant>.qa.example.com`
- **Environment-specific identifiers a setup ticket needs** (TE4): `none` — e.g. tenant ID, database
  name, client ID

## 10. Tenancy

- **Multi-tenant product**: `TODO — yes / no`. When `no`, every tenant rule (U11, RP7, TE7, UI6's
  per-tenant half, the tenant axis in `test-matrix.md`) is skipped rather than flagged.
- **What a tenant is called**: `tenant` (default) — or `client`, `customer`, `workspace`, `org`
- **Tenant key format** (how tenants appear in filenames, URLs, theme folders): `TODO — e.g. lowercase
  slug`
- **Known tenants** (optional, helps U11): `none`
- **UI variants gated per tenant or flag** (UI6): `none` — e.g. *"legacy and new checkout flow; new is
  on for tenants A and B only"*

## 11. Tech stack

- **Backend**: `TODO — e.g. .NET, Java/Spring, Node, Python`
- **Frontend framework**: `TODO — e.g. Angular, React, Vue, server-rendered`
- **UI component library** (UI7): `TODO — e.g. Kendo UI, Material, Ant Design, none`
- **Where the component for a screen lives** (repo glob, used by the mandatory component read):
  `TODO — e.g. **/<feature>*.component.{html,ts}` or `**/<Feature>*.{tsx,jsx}`
- **Where the service behind a backend change lives**: `TODO — e.g. **/<Feature>*Service.*`
- **Where design tokens live** (for `/mock`): `TODO — e.g. src/styles/_variables.scss,
  tailwind.config.*, tokens.json`
- **Per-tenant themes** (for `/mock`): `none` — or the path pattern, or *"served at runtime from
  <place> — not in the repo"*
- **Our own product's mark** (may appear on internal prototypes): `none` — or its repo path

## 12. Data and database conventions

- **Where data/migration scripts go** (DB2): `TODO — repo and folder`
- **Ad-hoc data-fix template / audit log** (DB3): `none`
- **Deletion policy** (DB6): `soft delete (default)` — or *"hard deletes allowed"*

## 13. QA

- **Test management tool** (QA3): `none` (default) — e.g. TestRail, Xray, Zephyr; and its case-ID
  format, e.g. `C12345`
- **Automation-coverage field on epics** (QA1): `none` (default)
- **Automation task naming** (QA2): `none` (default) — e.g. an `AQA - ` prefix
- **Test-case development subtask** (QA4): `none` (default) — e.g. `QA - TestCaseDevelopment - <area>`
- **Optional QA hand-off skills** (`test-matrix.md` § *Hand-offs*): `none` — name any installed skill
  that writes test cases, searches existing cases, or turns findings into questions

## 14. Notifications

- **Channels** (NT3): `TODO — e.g. Email · In-app message · Mobile push`. Write a mobile channel as
  `Mobile app`, never bare "mobile" — see `test-matrix.md` § *Axis values are disambiguated*.

## 15. Calibration tickets

Real tickets from your own boards that show what "good" looks like, per kind. `examples.md` ships with
illustrative examples; replace or supplement them with these as you find them.

| Kind | Key(s) |
|------|--------|
| Story | |
| Bug | |
| Task | |
| Epic | |
| A short, complete ticket (calibration for U18b padding) | |

---

## How blanks are handled

The rule is the same everywhere: **discover, else ask once, never invent.**

| Blank | What the plugin does |
|-------|---------------------|
| Projects (§ 2) | Lists visible projects through the connector and asks which one, as part of the first question it already owes the requester. Remembers the answer for the session |
| Issue types (§ 4) | Reads the project's issue types from the connector's create metadata and maps them to the templates by name |
| Priority scheme (§ 5) | Uses the allowed values in the create metadata's priority field, mapped by rank to the signal table |
| Label vocabulary (§ 6) | Adds only `ai-draft`, plus labels the requester names or that sibling tickets under the same epic already carry. Never coins a label |
| Summary prefix (§ 3) | No prefix. Mirrors the style of sibling tickets only if the requester confirms it |
| Environments, hosts (§ 9) | Asks when a ticket needs them (bugs, setup work). Never guesses a hostname |
| Tenancy (§ 10) | Treats the product as single-tenant for the session unless the requester or the ticket says otherwise |
| Tech stack (§ 11) | Infers from the checked-out repo (`package.json`, project files) and says it inferred; with no repo, UI7 reports `⚠️ component library unknown` |
| QA tooling (§ 13) | Skips the rule, and says it was skipped rather than passed |

**A blank that mattered goes in the draft.** If the requester had to be asked, or a rule was skipped
because this file didn't say, the draft's `RESEARCH` block names it under `Not read` — so the team
can see which entries in this file are worth filling in.
