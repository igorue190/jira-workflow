---
name: prototype
description: >
  Build a standalone interactive HTML/CSS/JS mockup of a UI ticket, styled with the product's own design
  tokens (the tenant's, on a multi-tenant product) read out of the connected repo, and publish it as a private Claude Code Artifact whose URL
  goes on the ticket to satisfy UI2 (prototype/design linked). Use when the user runs `/mock`,
  or says "mock this up", "mock up the screen", "what would this look like", "show me the layout",
  "build a prototype", "wireframe this", "I can't picture this", "draft a design for this ticket",
  "visualise this story", or asks for a visual before writing the acceptance criteria. Also use when
  a UI-facing ticket has no design link and the requester accepts an offer to mock it up.
  Do NOT use this skill to write production frontend code, or to scaffold components —
  the output is a picture, not code a developer copies into the product. Do NOT use it for backend,
  API, DB, data-import or report-data tickets with no visual change; for recreating the live product
  to document a bug (a real screenshot is the honest instrument there, BG6); for anything that will
  be sent to a client or used as a client deliverable; or to create the ticket itself — that's
  `quick-ticket` or `ticket-create`.
---

# Prototype (visual mock for a UI ticket)

A ticket that says "the order list should be usable on mobile" can be built four different ways, and
every one of them passes the acceptance criteria as written. This skill draws one of them, so the
argument happens in refinement instead of in code review.

The output is a layout on a private URL that the reviewer can **drive** — click the control, watch the
state change, go back. Not code anyone lifts, not a design UIUX signed off on, and not a spec. `UI2 | Prototype/design linked` in `../../references/dependency-matrix.md` is
the rule it satisfies, and it satisfies that one only.

## What stays true here

1. **Nothing is built or published without explicit approval.** The proposal block stops and waits for
   `build`. Publishing does **not** imply putting the URL on the ticket — that is a second, separate
   approval.
2. **No invented values, and no invented brand.** A style token you can't find in the repo is a `⚠️`
   line and a named fallback, never a colour you remember the client using. Your memory of a client's
   palette is the most dangerous input to this skill, because it produces a page that looks
   authoritative and has no source.
3. **The page is labelled an internal prototype, permanently.** Banner, title marker, synthetic-data
   notice, version line. Not negotiable, including when the requester asks for them gone. Rules in
   `../../references/prototype.md`.
4. **Every value on the page is invented.** Nothing copied from Jira, Confluence, a fixture or a
   screenshot. An Artifact URL is more forwardable than a Jira ticket, so the bar rises, not falls.

## Entry paths

- **Asked for** — `/mock`, "mock this up", "what would this look like". Start at Step 1.
- **Offered and accepted** — `ticket-create` or `ticket-review` hit the offer criteria and the
  requester said yes. Still start at Step 1: the offering skill establishes *that* a mock is wanted,
  not *what* it shows.

**Re-check that this is UI work even when you were asked directly.** "What would this look like" also
fires on a report's column list and on a DB migration. If the ticket isn't UI-facing, say so in one
line and offer what actually helps — a `⚠️ UI3` breakpoint question, or a real screenshot for a bug —
instead of drawing a picture of a table nobody was confused about.

---

## Step 1 — Ask for the screenshots, then identify the ticket, the tenant and the UI variant

**If the ticket touches any existing screen — not only when it modifies one — ask for screenshots
before you do anything else.** This is a gate, not a courtesy: it happens before you propose, before
you read tokens, before you decide what the mock shows. One message, asked once, folded into any other
question you owe them, naming exactly what you need:

- **Every screen the ticket touches**, not just the region that changes.
- **Each vendor control in its open state** — the library's data grid with its filter menu expanded, the upload
  dialog open, the dropdown open, the menu hovered. A closed control tells you almost nothing.
- **The tenant**, on a multi-tenant product, so the screenshots and the tokens are describing the same
  product.
- **Any screen that routes into** the changed one — the tile, the card, the nav item people arrive by.

Why it blocks rather than waits: a third-party library control appears in the checkout only as a tag. Its toolbar,
filter row, sort caret and close button are drawn at runtime and exist nowhere in source, so building
them from stylesheets **invents affordances the product doesn't have** and misses ones it ships
(`../../references/prototype.md` § *Style discovery*).

**"No screenshot available" is still a valid answer.** Build that region from the component template
and **mark it unverified on the page** — never draw the missing part with confidence. Apply the hedge
**by category, not by whichever part you happened to doubt**: if the grid and the upload dialog are
marked unverified, the page header and the nav are the same category of vendor-rendered chrome and get
the same mark. A page that hedges half its guesses reads as confident about the other half, which is
worse than hedging none of them.

| Need | Where it comes from | If absent |
|------|--------------------|-----------|
| Ticket key | The requester, or a ticket created earlier this session | No key is fine — build the mock and hand back the URL. Offer `quick-ticket` so the link lands in a draft before create, which beats an edit afterwards |
| Tenant | Multi-tenant products only (profile § 10). The requester's words · the ticket's U11 tenant statement, customer label or client-first summary prefix · the linked Confluence page if Confluence is in scope | **Ask once**, folded into any other question you owe them. "Don't know" or "doesn't matter" is a fine answer: build on the neutral baseline and say so. Single-tenant → skip the row |
| UI variant (UI6) | The ticket, checked against the variants profile § 10 lists — some products run a legacy and a new UI side by side, gated per tenant or by a flag | `⚠️ which UI variant — legacy or new — not stated (UI6)` in the proposal. Draw one, name which, and say the other exists. No variants listed → skip the row |
| What the ticket actually changes | The description and AC | If you can't tell which region of which screen moves, that's the finding. Ask rather than mocking a whole page |

**Never infer the tenant** from the accent colour in a screenshot, from the most recently modified
theme file, or from whichever client came up ten minutes ago. Getting the tenant wrong produces a
confident page about somebody else's product — the most expensive failure this skill has.

## Step 2 — Read the component, then find the style tokens

**Read `../../references/project-profile.md` § 10–11 first** — the frontend framework, the component
library, where components and tokens live, and whether the product is multi-tenant. Blank → infer
from the checkout (`package.json`, project files) and say you inferred.

**Open the component first — its markup and its logic, not just its stylesheet.** This is mandatory
before a mock (`../../references/repo-context.md` § *Reading source, not just docs*) and it is the
cheapest correction available: a stylesheet gives you the palette and tells you nothing about which
controls exist, which states are already handled, or which library components are already in use. A
mock built from SCSS alone invents chrome the product doesn't have — a `▾` where the real screen has
the library's close button — and then confidently annotates a feature as *"no library equivalent"*
that the library has shipped all along. Both failures look like design opinions and neither is one.

### Read the component twice, with different questions

One pass answers one kind of question, and the pass people skip is the second one.

1. **Content pass** — fields, labels, tooltip copy, control types, validation bounds.
2. **Structure pass** — every grid class copied **verbatim**, in order, plus the row spacing utilities
   (`mb-4`, `mt-4`, `align-items-center`). Write them into the proposal's source manifest before you
   write any markup.

> **Geometry comes from the component template. Never from the ticket's mockup CSS.**
> When a ticket ships its own HTML/CSS reference, that reference is authoritative for fields, copy,
> thresholds and interaction sequence — and for nothing else. If the mockup and the template disagree
> about layout, **the template wins**, and the disagreement is a finding worth reporting on the ticket.

The abstract rule doesn't land on its own, so here is what it looks like:

```
ticket mockup                     real component
.switch-row { display:flex;       <div class="row align-items-center mb-4">
  gap:18px; }                       <div class="col-md-3 col-6"><ui-label …/></div>
.switch-label { width:210px; }      <div class="col-md-3 col-6"><ui-switch …/></div>
→ control flush right             </div>
                                  → label 25%, control 25%, right half empty
                                  → below md: 50% + 50%, still side by side
```

- **Check the below-`md` case separately.** `col-6 col-6` keeps a label and its control on one line at
  375px. A naive mobile rule stacks them and silently changes what the reviewer is judging.
- **Never drop a field the template has without saying where it went.** A field omitted in silence
  reads as a design decision nobody made, and it survives every revision because nobody can see it.
- **Enumerate every route into each region you mock.** A home-page tile grid dismissed as "chrome" is
  a live second route into the pages you're drawing; leaving it out means the reviewer never sees how
  people actually arrive.

Then the tokens, and the assets. **Read `../../references/prototype.md` here, once and whole** — the
search table, the extraction rule, § *Asset inventory*, and the labeling, data and accessibility
rules Step 5 applies. Every logo, icon font, typeface and background image that ships in the
checkout goes on the page as a data URI, never as a placeholder or a lookalike. Later steps cite its
sections rather than asking you to open it again. Read
`../../references/repo-context.md` for how much of a repo you may read at all — index filenames with
`Glob` first, open two or three files, never read a tree whole.

Exactly three outcomes, each with its own label on the page:

| Outcome | Style with | The page says |
|---------|-----------|---------------|
| Tenant (or product) tokens found | Those values, path cited | `Styled from <path> — tokens as of this checkout` |
| Repo present, no tenant tokens (or no tenant named) | The repo's base theme — its own tokens, or the component library's default theme (one `Grep` of `package.json` for the library's theme package tells you which) | `Baseline styling — <library/theme> defaults, not <tenant> branding` |
| No repo, or nothing found | The fixed neutral wireframe palette in `prototype.md`, so every such page looks identical and obviously belongs to nobody | `Neutral wireframe styling — no tenant tokens available` |

**There is no fourth outcome.** Don't derive a palette from the client's public website, their logo, a
marketing PDF, or your own knowledge of their brand. That is the invented-values failure and the
impersonation failure at once. `⚠️ no tenant tokens found — neutral styling` is a better page than a
confident guess, and it's an honest finding for the ticket.

**Never fetch anything from the tenant's live site or CDN** — not a stylesheet, not a logo, not a font.
Two independent reasons, either sufficient: the Artifact CSP blocks every external host, so a remote
reference renders as silent unstyled fallback; and pulling a client's live brand assets into a page you
then publish is the impersonation line.

## Step 3 — Decide what the mock shows

- **Both viewports, always — desktop at the page's full width, mobile stacked beneath it**, each
  framed and independently drivable, not one responsive page the reviewer resizes. Side by side costs
  the desktop panel roughly a third of its width and forces it to scale down, so the panel under most
  scrutiny gets the least room. `UI1` asks that **both widths are visible without resizing**, and
  stacking satisfies that while letting desktop render near 1:1 and mobile keep its native 375px.
- **The states the text skips** — populated, empty, and one overflow case: the 42-character product name,
  the zero balance, the label that wraps to three lines. This is the whole value of a mock over prose.
- **Only the region the ticket changes**, plus enough surrounding chrome to place it. A mock of the
  whole application is a redesign proposal nobody asked for.
- **Component-library feasibility (UI7)** — everything drawn must be achievable with a component from
  the library in profile § 11. Anything that isn't gets annotated on the page: `no <library> equivalent
  — needs UIUX/dev feasibility`. Inventing a control the library doesn't have costs a sprint of
  argument. No component library (plain CSS, server-rendered theme) → name the theme and skip UI7.
- **Drivable, not just visible.** List what the reviewer will be able to click, and make sure every
  state in `SHOWS` has a way in. A state that exists only as a frame is a state nobody reviews, and
  the control the ticket is *about* is the one that has to actually work. Rules in
  `../../references/prototype.md` § *Interaction*.
- **Charts** — load the `dataviz` skill before drawing one (a BI/dashboard report layout, RP6).

## Step 4 — Show the proposal and stop

```
PROTOTYPE PROPOSAL — reply "build" to make it, or tell me what to change
Ticket:   PROJ-123 · UIUX - Order list overflows the container on mobile
Tenant:   Acme         UI variant: new checkout — UI6
─────────────────────────────────────────────
SHOWS
1. Order list, desktop 1280px — populated, 6 rows
2. Order list, mobile 375px — same data, breakpoint 768px
   (src/styles/_breakpoints.scss)
3. Empty state, and one 42-character product name — the overflow this ticket is about

SOURCES — read before any markup
order-list.component.html — row 1 `col-md-3 col-6` / `col-md-3 col-6`, `mb-4`
                            row 2 `col-md-6 col-12`, `align-items-center`
                            9 fields, verbatim; none dropped
Screenshots: grid + filter menu open, mobile nav open (supplied)
⚠️ upload dialog — no screenshot, built from template, marked unverified on the page
   (all vendor-rendered chrome carries the same mark, not just this one)

STYLING
src/styles/themes/_acme.scss — 10 colours, 5 radii, all cited; 14px base
Assets inlined: assets/img/product-logo.svg (our own mark) · icon font .ttf
(6 glyphs, names resolved) · OpenSans-{Regular,SemiBold}.woff2 · body texture.png
[ CLIENT LOGO ] placeholder in the tenant logo slot — Acme's mark never renders
⚠️ no tenant token for the mobile nav — library default used, marked on the page

INTERACTION
Click an order row → its detail panel opens; the close button and Esc return to the list.
That row control is what this ticket is about, so it is wired rather than drawn.
State strip switches populated / empty / 42-char name — those have no natural trigger.
Desktop full width, mobile stacked beneath — both drivable independently.
Keyboard: real buttons, tab order, visible focus.
Each state deep-links (#state-02), so a ticket comment can point at one.

DATA
Synthetic, standard cast. Nothing copied from the ticket, Jira or a fixture.

DOES NOT SHOW
- Filter panel behaviour — not in this ticket's scope
- The admin side — no requirement stated
- ⚠️ Which breakpoints the design actually changes at — UI3 unanswered on the ticket

PUBLISH AS
Private Artifact page · banner "INTERNAL DESIGN PROTOTYPE" · favicon 📐
File: <scratchpad>/prototype-PROJ-123.html — re-publishing updates the same URL
─────────────────────────────────────────────
Nothing is built or published until you reply "build".
The URL goes on the ticket only after you approve that separately.
```

`DOES NOT SHOW` is the half that gets skipped and shouldn't. A reviewer who knows what the page omits
reads it as one option; a reviewer who doesn't reads it as the design. It's also where the matrix gaps
surface — a `⚠️ UI3 unanswered` there is worth more to the ticket than the picture is.

Anything other than `build` — a correction, a question, another region — is an edit to the proposal,
not approval. Redraft and stop again.

## Step 5 — Build the page

1. **Load the `artifact-design` skill before writing a line of the page** — it carries the three-state
   theme token pattern and the CSP constraints, and a page written without either renders one theme's
   text on the other theme's ground. It ships with the Artifact capability, not with this plugin, so it
   isn't guaranteed present: **if it won't load, don't skip the theming and don't improvise it** — use
   the minimum token contract written out in `../../references/prototype.md` and say in your handback
   that you styled from the fallback.
2. **Apply the labeling block, the data cast and the accessibility requirements** from
   `prototype.md` § *Labeling and no-impersonation*, § *Data on the page* and § *Theme and
   accessibility* — read at Step 2. Those are what make the page safe to publish and none of them
   are in `artifact-design`.
3. **Write to the session scratchpad** at a stable path — `prototype-<KEY>.html`, or
   `prototype-<slug>.html` with no key. **Never inside the connected repo's working tree**: it shows up
   in `git status`, someone commits it, and the product repo now carries a mock nobody maintains.
   Re-publishing that same path updates the same URL, which is the only reason a link on a ticket
   survives revision — don't change the path casually.
4. **Everything inlined, and everything real.** No CDN script, no remote font, no remote image. Fonts
   are `@font-face` data URIs built from the repo's own `.woff2`/`.ttf` files; logos, icon fonts,
   sprites and background images are data URIs from their real paths — `../../references/prototype.md`
   § *Asset inventory* is the list to resolve, and it is resolved every time, not remembered. **A drawn
   placeholder is a last resort**: allowed only where the binary genuinely isn't in the checkout, and
   then labelled on the page *and* justified in the handback.
5. **Wire the interaction with inlined vanilla JS** — `../../references/prototype.md` § *Interaction*
   is the contract. No framework, no build step, nothing a developer could lift. Then **publish first
   and verify on the published page**, in this order:
   1. Write the file to its **stable path** (item 3), publish it, *then* check what you can.
   2. **Do not open a `file://` preview and do not stand up a local server.** A local file outside the
      project folder renders as a static snapshot with scripts disabled, so a drivable page looks
      dead — and working around that with a throwaway static server burns cycles and produces nothing
      the reviewer ever sees. On the published page, click every state in `SHOWS` and tab to every
      control; a state you listed and never reached is the failure this step exists to catch, and it
      is invisible in the markup.
   3. If the published page auth-walls the session, **say so plainly**, hand back the file path
      alongside the URL, and let the requester do the visual check (Step 6). Never write "verified
      both themes render" about a page you got an auth wall from.
6. **Run the mechanical checks** in `../../references/prototype.md` § *Mechanical checks before
   publishing* — eight measurements, not judgements, each one a defect that has shipped before.
   **Check 3** catches the classic unreadable-artifact bug: a colour declared only inside a `@media`
   or `[data-theme]` block. On this page that reads as "the design is broken" rather than "the page is
   broken".

## Step 6 — Publish, read it back, say who can open it

Publish with the `Artifact` tool: `<title>` beginning `Prototype:` and ending `(internal mock)`, a
one-sentence `description` that says internal design prototype, `favicon` 📐.

**Then open the published URL and check it.** The banner is present, both themes resolve, the states
click through, nothing tried to reach an external host. Report the **real URL** — never a predicted
one. Same rule as reading a created issue back, for the same reason: the failure is silent.

**If you cannot open the published page, say so plainly.** The in-app browser is not signed in to
claude.ai, so the page you just published can auth-wall the session that made it. That is a normal
outcome and not a reason to skip the step or to soften it: say the rendering was not verified, hand
back the **local file path** as well as the URL so the requester can open the HTML directly, and let
them do the check. Never write "verified both themes render" about a page you got an auth wall from —
an unverified page honestly labelled costs one sentence; a false verification costs the requester's
trust in every other line of your handback.

Then say plainly, in the requester's terms: **the page is private. Sharing it with the team is yours to
do, and the link is only worth putting on a ticket once you have** — otherwise UI2 is satisfied by a URL
that auth-walls for every developer who opens it, which is worse than an honest `⚠️`. The plugin never
shares an artifact on their behalf.

**If artifact publishing isn't available at all** — a plain terminal session, a headless or scheduled
run, an environment where the `Artifact` tool isn't present — this skill has no URL to produce, and that
is a fact to report rather than route around. Say publishing is unavailable, hand back the **file path**
of the HTML you wrote, and stop. Do **not** put a local path on the ticket in place of a URL: UI2 asks
for a link the team can open, and a path on one person's machine isn't one — the ticket keeps its honest
`⚠️ UI2 unsatisfied`. Never paste the page's markup into a ticket or a comment as a substitute.

## Step 7 — Get the URL onto the ticket

Separate approval. Approval of one thing is not approval of the next.

```
TICKET EDIT — PROJ-123 — reply "apply" to update the ticket
─────────────────────────────────────────────
Add under LINKED DOCUMENTATION:
  Prototype: <url> — internal mock, not a signed-off design
  Shows desktop 1280 / mobile 375, populated + empty states. Synthetic data.
  Tokens from src/styles/themes/_acme.scss
UI2 then reads: ✅ prototype linked — generated mock, not Figma
─────────────────────────────────────────────
```

1. **Say it's a mock, in the link text.** UI2 asks for "Figma or design mockup"; this is the second
   one. A bare `Design: <url>` invites a developer to build to it as though UIUX approved it. That one
   clause is the difference between satisfying a rule and creating a misunderstanding.
2. **Field mechanics checked, not guessed.** `getJiraIssueTypeMetaWithFields` before the edit if the
   project exposes a Documentation field, same discipline as the epic-link field. Read the issue back
   and confirm the link landed; say plainly if it didn't.
3. **Do not invent a label.** No `prototype`, no `mockup`, no `has-design`. U6 names the live
   vocabulary and forbids adding to it; a label the board has never seen is noise in every future JQL
   query. The URL in `LINKED DOCUMENTATION` is the record.
4. **A comment is fine if they'd rather** — `addCommentToJiraIssue`, comment text approved first,
   because a comment notifies watchers.
5. **Never edit a ticket you weren't handed.** No key, or a key you inferred, means you hand back the
   URL and stop.
6. **UI2 satisfied is UI2 only.** Don't report the UI section as clear. UI1, UI3, UI4, UI5, UI6 and UI7
   are untouched by a mock existing, and a ticket that now *looks* designed with UI3 still blank is
   worse off than before.

## Step 8 — Revisions

- A material change — a new region, a different flow, a different tenant — re-runs the Step 4 proposal
  before rebuilding. The requester approves content, not intention.
- **Re-publish the same file path.** The URL on the ticket stays valid; that's what the stable path is
  for.
- **Add a version line on the page** — `v3 · added empty state`. Re-publishing silently replaces a page
  someone may already have reviewed, and a one-line log is the cheapest honest fix.
- If the URL is already on the ticket and the mock changed materially, offer to update the one-line
  `Shows:` summary. The URL itself doesn't change.
- **Two options go on one page as two panels, not two URLs.** One link per ticket keeps UI2
  unambiguous.
- **Never re-publish to remove the banner, the synthetic-data notice or the version line.**

---

## When the request crosses the line

Some requests turn a labelled internal prototype into something that reads as the client's live
product. Full rules in `../../references/prototype.md`; the behaviour is the same every time — **don't
do it, say why in one line, offer the honest alternative.**

| They ask for | Say | Offer instead |
|--------------|-----|---------------|
| A **third-party client's** logo on it | A page carrying the client's mark reads as published by the client | The `[ CLIENT LOGO ]` placeholder — it conveys the layout identically |
| Our **own product's** mark left off | Nothing — this one is allowed, from its real path at its real box, on an internal page carrying our own byline | Use it. A dashed placeholder where the product's own lockup ships in the checkout is a substitution, not a safeguard |
| The customer's production host in a browser frame | Rendered URL chrome makes a mock look like a screenshot of production | The host as text in the notes block, and on the ticket where BG4/TE5 want it |
| Real users, real amounts, real comments | Fabricated-as-genuine records, and PII, on a forwardable URL | The synthetic cast; a real figure quoted as `per PROJ-123: 1,250` in the notes |
| A pixel copy of the live page | A hand-built replica of production is not evidence of production | They take a screenshot and attach it — that *is* the honest instrument (BG6) |
| The banner removed, "just for the client demo" | Not available | A screenshot of the page in their own deck, where their own framing carries it |

If they press, **stop building.** An unlabelled tenant-branded page on a shared URL cannot be recalled.

## Related skills

- **`artifact-design`** — load before writing the page, every time it's available; the fallback token
  contract in `prototype.md` covers the environments where it isn't. **`dataviz`** too if the mock has a
  chart.
- **`quick-ticket` / `ticket-create`** — where the ticket comes from. They offer this skill; they don't
  run it.
- **`ticket-review`** — where UI2 gets audited on tickets that already exist.
- **`ticket-reviewer`** (subagent) — it checks that the URL landed and that the AC cover what the mock
  is *said* to show. It never opens the page and never judges the visual; that's a call the requester
  makes in one glance.

## Reference files

| File | When to read |
|------|-------------|
| `../../references/project-profile.md` | § 1 (team name for the banner), § 10 (tenancy, UI variants), § 11 (framework, library, component and token paths) — every prototype |
| `../../references/prototype.md` | Every prototype — style discovery, interaction, labeling, data, accessibility |
| `../../references/repo-context.md` | Whenever a repo is checked out, before any `Glob` — and § *Reading source, not just docs* before every mock, since the component read is mandatory |
| `../../references/dependency-matrix.md` | The UI section only (UI1–UI7). A prototype is not a matrix pass — don't read the whole thing |
| `../../references/ticket-schema.md` | Only for Step 7's field mechanics, if the link needs the Documentation field |
