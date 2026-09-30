# Prototypes: building a visual mock for a UI ticket

A prototype is one option, published at a URL and **drivable by the reviewer**, so a UI ticket stops
being argued about in prose. Read this file whenever you're building one. The workflow is in
`skills/prototype/SKILL.md`; this is the discovery rules, the interaction rules, the labeling rules
and the boundary.

The reason the labeling section below is this strict: a tenant-styled page on a forwardable URL is one
unlabelled step away from something that reads as the client's live product. Everything here keeps it
visibly, permanently, an internal draft.

---

## Style discovery in the connected repo

`repo-context.md` governs how much of a repo you may read: index filenames with `Glob` first, open two
or three files, never read a tree whole. Everything below is where to point that at.

**Where to look is configured in `project-profile.md` § 11** — the frontend framework, the component
library, where components live, where design tokens live, and whether per-tenant themes exist. The
table below is the generic fallback when the profile is blank.

**Tokens are the second thing you read, not the first.** Open the component's markup and logic before
any of this — that is mandatory before a mock (`repo-context.md` § *Reading source, not just docs*).
Styles tell you the palette; only the component tells you which controls, states and library
components already exist. Draw from stylesheets alone and you will invent chrome the product doesn't
have.

**A wrapped component delegates its chrome (UI7).** When the markup mounts a third-party library
component — a data grid, a dropdown, a dialog from the library in profile § 11 — it tells you *that*
the control is there, not what it renders. The toolbar, the sort caret, the filter row, the close
button are drawn by the library at runtime, and the checkout never spells them out. So a mock built
from the template alone gets a wrapped control's chrome wrong in both directions — inventing
affordances it doesn't have and missing ones it ships. For any region the ticket touches that is
vendor-delegated, get a **screenshot of the current screen** (`skills/prototype/SKILL.md` Step 1) or
mark that chrome **unverified** on the page rather than drawing it with confidence.

### Where the tokens live

| Look for | Pattern |
|----------|---------|
| The paths in profile § 11 | Whatever the team wrote there wins over every row below |
| Which stylesheets actually ship | The build config's style entry — `angular.json` → `"styles"`, the root CSS import in a React/Vue entry file, the layout template's `<link>` tags on a server-rendered app. The cheapest way to find the real entry point |
| Global entry stylesheet | `**/src/styles.{scss,css}`, `**/src/styles/**/*.{scss,css}`, `**/src/index.css`, `**/src/app.css` |
| Token / variable files | `**/_variables.scss`, `**/_theme*.scss`, `**/_colors.scss`, `**/*tokens*.{scss,css,json}`, `**/tailwind.config.*`, `**/theme.{ts,js}` |
| Component-library theme customisation | `**/*<library>*.{scss,css}`, plus one `Grep` of `package.json` for the library's theme package to learn which theme is the base |
| Per-tenant themes | The pattern in profile § 10–11, else `**/themes/**`, `**/tenants/**/*.{scss,css}`, `**/branding/**`, or the tenant key as a filename fragment |
| Brand colours injected at runtime | One `Grep` for `--brand`, `--primary`, `PrimaryColor`, `ThemeColor` |
| Assets the component's CSS references | `url(...)` targets in the component's SCSS/CSS — icons, sprites, background images — resolved inside the checkout |

On a multi-tenant product, tenants are usually keyed by a slug in the format profile § 10 names —
that's what appears in filenames, URLs and storage object names, so it's the string worth globbing
for.

If the tenant key appears nowhere and the `--brand`/`PrimaryColor` grep hits, **the values probably
live at runtime, not in the repo** — served from a database, blob storage or a CDN (profile § 11 says
where, if the team wrote it down). That is a `⚠️` and a fallback, not a failed search, and it is *not*
a licence to fetch from the CDN (see below).

An asset the component's CSS references and that **ships in the checkout** — a generic icon, a UI
sprite, a background texture — goes on the page: it's our own asset, so cite its path like a token and
use it. Logos need one distinction, and it is about **whose mark it is**, not about where the file sits:

- **Our own product's mark is not a client's mark.** On an internal page carrying our own byline it may
  be used, from its real path at its real box. A dashed placeholder where the product's own lockup
  ships in the checkout is a substitution, not a safeguard.
- **A third-party client's logo still never appears**, even when the repo contains it, because a page
  carrying the client's mark reads as published by the client (§ *Labeling*). That is the
  `[ CLIENT LOGO ]` placeholder's only remaining job: the repo asset is usable, the client's identity
  is not.

When the tenant's configured logo *is* the setting being mocked, mock the **slot** — the box at its
real dimensions, the placeholder inside it — and say on the page that the real mark would render there.

### What to extract

Two branches, and they pull in opposite directions:

- **Reproducing an existing screen** — take **every token the screen actually consumes**, no cap. If it
  reads ten colours and five radii, take ten and five. Cite each path. A cap here doesn't protect
  against invention; it caps fidelity below what the repo can supply, which is the opposite of the job.
- **Inventing a new screen with no precedent** — take **at most** 4–6 colours (brand/accent, page
  surface, raised surface, primary text, muted text, border), one border radius, one font stack with a
  base size, one spacing unit. The cap belongs here, because here the risk is fabricating a design
  system rather than reporting one.

Plus the component library's look if it's customised — button height and radius, grid header
treatment.

**Cite the file path** in the proposal and in the page footer, either way — a named path is checkable
and "per the repo styles" is not.

### Asset inventory — resolve every time

Tokens are half of what a screen is made of. The other half is binaries, and every one of these was
substituted at least once on a page whose checkout contained the real thing:

| Asset | Source | Was substituted with (do not repeat) |
|-------|--------|--------------------------------------|
| Logo | `$tenant-img-path/logo.png`, or the tenant's configured logo field — often the very setting being mocked. Whose mark it is decides whether it renders (§ *Where the tokens live*) | a dashed placeholder box, even where the real mark ships in the checkout |
| Icons | The icon font or SVG set the product ships (the component library's icon package, or the repo's own), names resolved through its manifest | Unicode lookalikes `$ ★ ▣ ⧉` |
| Type | the product's own `.woff2` files, per its `@font-face` block | the same family from a CDN — different build, different metrics |
| Page ground | whatever `body { background }` resolves to, image included | flat grey |
| Sprites / textures | any `url(...)` the component's own CSS references | omitted |

The rules that go with the table:

1. **Everything is embedded as a data URI.** Splice it in with a small re-runnable script between two
   markers, so the page stays hand-editable and no base64 ever passes through prose.
2. **A placeholder is a last resort.** Allowed only where the binary genuinely isn't in the checkout,
   and then it is **labelled on the page *and* justified in the handback**. The target is zero
   unlabelled substitutions where the binary exists.
3. **Resolve, don't remember.** The inventory is walked fresh on every build, like the tokens are.
4. **Icon names are not glyph names.** Resolve each declared name against the icon set's manifest:
   aliases point elsewhere, and an SVG package does not always contain every font-set name. It is
   easy to get a third of the icons wrong on a page built while *looking at a screenshot*.

### What not to read, and what not to carry out

1. **Read tokens, not theming code.** A `.scss` variable file is a value source. A TypeScript theme
   service, a `ThemeResolver`, a DB seed script is implementation, and reconstructing a palette out of
   it is reverse-engineering intent from source — the thing `repo-context.md` rules out.
2. **Never over the network.** No live site, no CDN, no logo URL, no font host, no `WebFetch` of the
   tenant's product. The Artifact CSP blocks it *and* it's the impersonation line — either reason alone
   is enough.
3. **Never from memory.** If you happen to know what colour a client's brand is, that knowledge does
   not enter this page. It has no source, so it is an invented value.
4. **Nothing that isn't a style value leaves the repo.** A theme file that also holds a key, a
   connection string or an internal host: reference the path, never the value.
5. **Don't cache a palette.** Read the tokens fresh every time. A palette remembered from last week is
   a palette with no source.

### The fallback ladder

| Tokens available? | Style with | Page label |
|-------------------|-----------|------------|
| Tenant tokens found | Those values | `Styled from <path> — tenant tokens as of this checkout` |
| Repo present, none for this tenant (or no tenant named) | The repo's base theme — its own tokens, or the component library's default theme | `Baseline styling — <library/theme> defaults, not <tenant> branding` |
| No repo, or nothing found | The neutral palette below | `Neutral wireframe styling — no tenant tokens available` |

**The neutral palette**, fixed so every such page looks the same and is recognisable as belonging to
nobody: a slate-biased neutral ground and text, one desaturated blue-grey accent, one warm border.
Deliberately not a brand — unmistakably a wireframe palette rather than anyone's identity.

---

## Labeling and no-impersonation

The Artifact rules forbid publishing a page that impersonates a real organization — its name, branding,
byline or domain — or that presents fabricated records as genuine. A tenant-styled mockup of a real
client's platform sits near that line on purpose, because looking like the product is the point. What
keeps it on the safe side is that it never stops announcing what it is.

### The banner

Fixed at the top of the page, above everything, in both themes, in the **neutral** system regardless of
the tenant's tokens:

```
INTERNAL DESIGN PROTOTYPE · <team name from project-profile.md § 1> · PROJ-123
Not the live product. Not a client deliverable. Every value on this page is invented.
```

The byline is **ours, never the client's** — the team name from profile § 1; if that's blank, ask
once, and never substitute the client's name. An internal document with our name on it is not
impersonation; the same page with the client's name on it is. Not collapsible, not dismissible, not
removable on request.

### Neutral chrome, tenant content

**The frame is ours and the content area is theirs.** Banner, captions, annotations, notes block,
version line and footer all use the neutral system — neutral type, neutral ground, neutral accent — and
only the mocked UI region inside its viewport frame uses the tenant's tokens.

A viewer then sees at a glance that an internal document is *showing* a tenant screen, rather than seeing
a tenant screen. This does more work than any disclaimer, because it survives being screenshotted with
the top of the page cropped off.

### Title, favicon, captions

- `<title>`: `Prototype: <short subject> — <KEY> (internal mock)`. A shared link previews its title, so
  the marker has to be in it.
- `description`: one sentence, and it says "internal design prototype".
- `favicon`: 📐, every time, so any page from this skill is identifiable in a tab strip.
- Every viewport frame carries a caption: `Mock — Order list · desktop 1280px`.
- A version line in the footer: `v3 · added empty state`.

### What may never appear on the page

| Never | Instead | Because |
|-------|---------|---------|
| A **third-party client's** logo — the repo's asset, a recreation, or a styled text wordmark | A neutral `[ CLIENT LOGO ]` placeholder box | A page carrying the client's mark reads as published by the client. The placeholder communicates the layout identically. **Our own product's mark is not covered by this row** — see § *Where the tokens live* |
| The client's production domain as rendered chrome — a drawn address bar, a fake browser tab | The host as text in the notes block | Rendered URL chrome is what turns a mock into something that reads as a screenshot of production |
| The client's marketing trade dress — hero imagery, their site copy, their photography | Tokens only: colour, radius, type, spacing | Tokens are the *platform's* design system as configured for a tenant. Their marketing identity is theirs |
| Real people, real amounts, real comments, real IDs | The synthetic cast below | Fabricated-as-genuine records, and PII, on a forwardable URL |
| A "signed off", "approved" or "final" marker | The version line | Nobody signed anything off |

### When the requester asks for one of those

Don't do it. Say which rule in one line, offer the alternative from the table, and carry on with the
rest of the build. If they insist, **stop and don't publish** — a labelled page nobody wanted costs a
conversation; an unlabelled tenant-branded page on a shared URL cannot be recalled.

The one that comes up most: *"but the client is going to see this."* Then it isn't this skill's output.
A prototype exists to settle an internal argument about a layout. If a client-facing visual is genuinely
needed, that's a UIUX deliverable, and the honest answer is to say so rather than quietly removing a
banner.

---

## Data on the page

1. **All synthetic, all plausible in shape.** A name column holds names, an amount column holds amounts,
   a date column holds dates in the platform's format. The point is layout under realistic content
   lengths — lorem defeats that, and real data crosses the line.
2. **Use the standard cast**, so every prototype reads consistently and nobody mistakes a name for a
   colleague: **Dana Whitfield, Marcus Oyelaran, Priya Raghunathan, Tom Beckett, Lin Nakamura**, in
   units **Field Operations** and **Client Services** at the fictional employer **Northwind Trading**.
   Addresses on `example.com` (reserved by RFC 2606). IDs in an obviously fake range (`U-90001`).
3. **Never copy a value out of a real source** — not Jira, not Confluence, not a screenshot, not a test
   fixture, not the tenant's database. `repo-context.md` already blocks fixture customer data from
   reaching a *ticket*; an Artifact URL is more forwardable than a ticket, so the bar rises.
4. **A real number the ticket depends on is a citation, not a widget.** `per PROJ-123: budget 1,250`
   in the notes block is information. The same figure painted into a balance card is a
   screenshot of the client's finances waiting to be forwarded.
5. **Spend the data on edge cases**, because that's what the mock reveals and the ticket text never
   does: the longest realistic name, the empty list, zero and negative (TX3, RP4), the row that fails
   validation (IM4), the label that wraps to three lines.
6. **No dates that imply a commitment.** A mock with "Releases 12 Sept" on it becomes a promise.

---

## Interaction

A picture of one option settles less than a thing a reviewer can drive. These rules are why a mock
beats prose at all — a reviewer who clicks through the states argues about the design; a reviewer
looking at a still argues about what the still is showing.

1. **Every state in the proposal's `SHOWS` list is reachable by clicking**, not merely present as a
   frame on the page. A state a reviewer can't get to is a state they won't review.
2. **The real control does the real thing.** Where the control *is* the argument — a command chip, a
   feature toggle, a tab, a filter — wire it. A separate state-switcher strip is fine for the states
   with no natural trigger (an error, an empty result, a failed upload); it is not a substitute for
   wiring the control the ticket is about.
3. **Both viewports on one page — desktop at the page's full width, mobile stacked beneath it**, each
   independently drivable. Don't collapse two frames into one resizable frame; a reviewer looking at one
   width still can't see the other, which is the exact failure UI1 exists to prevent. And don't set them
   side by side: that costs the desktop panel roughly a third of its width and forces it to scale down,
   so the panel under most scrutiny gets the least room. UI1's requirement is that **both widths are
   visible without resizing**, which stacking satisfies.
4. **Vanilla JS, inlined, no framework.** Keeps the CSP happy and keeps *"not liftable code"* true:
   no framework, no product class names, no component scaffolding. A few dozen lines of
   `addEventListener` is the whole budget.
5. **Keyboard operable.** Real `<button>`, arrow keys on a tab strip, visible focus. Accessibility is
   often verified manually (UI4), so the mock is where it gets shown rather than asserted.
6. **The current state is always labelled on screen**, so a cropped screenshot of the mock is still
   unambiguous — the same property the permanent banner protects.
7. **Deep-link each state with a URL hash** (`#state-04`). A comment on a ticket can then point at one
   state instead of describing it.
8. **No latency theatre.** If a transition is time-based, cap it low and give a way to re-trigger it.
   Nobody should have to wait out a sequence twice to check one frame.
9. **Scale frames with `zoom`, never `transform: scale`.** `transform` leaves the layout box at its
   full width, so the frame keeps a horizontal scrollbar and can drift sideways — enough to clip the
   logo off the left edge. `zoom` scales the box with the content.
10. **Wrap CSS resets in `:where()`.** `.mk button { font: inherit }` is one class plus one type, which
    outranks every component rule and silently discards its declared size and weight — nav at inherited
    14px/400 where the component says 15px/600. `:where(.mk) button { … }` has zero specificity and
    resets without winning.

---

## Theme and accessibility

1. **Three viewer states, not two.** Load the `artifact-design` skill for the full pattern. It ships with
   the Artifact capability rather than with this plugin, so it may not be present in every environment —
   **if it isn't loadable, don't skip the theming and don't guess.** This is the minimum contract, and it
   is enough to publish safely:

   ```css
   :root {                                   /* complete LIGHT palette — every token defined here */
     --surface: #ffffff;  --surface-raised: #f6f7f9;
     --text: #1a1d21;     --text-muted: #5b6472;
     --border: #d9dde3;   --accent: #3f6d9e;
   }
   :root:not([data-theme="light"]) {          /* system dark — REdefine the same tokens, nothing new */
     @media (prefers-color-scheme: dark) {
       --surface: #14171a;  --surface-raised: #1d2126;
       --text: #e8eaed;     --text-muted: #9aa4b2;
       --border: #2c3238;   --accent: #7aa9d8;
     }
   }
   :root[data-theme="dark"] { /* explicit toggle — same redefinitions again, so the toggle wins */ }
   body { background: var(--surface); color: var(--text); }
   ```

   Three rules that carry the whole thing: **every colour gets its first definition on bare `:root`**
   (a colour defined only inside a media or `[data-theme]` block is the classic unreadable-artifact
   bug); components read tokens only, never raw hex; and `body` sets an explicit token background,
   because a transparent body borrows the host page's ground and inverts on half your readers.
2. **Why it matters here specifically:** this page gets opened by a BA, a dev and a QA person in three
   different theme settings, and a mock that renders white-on-white gets read as "the design is
   broken", not "the page is broken". The design gets rejected for a CSS bug.
3. **Accessibility is usually verified manually (UI4), so the mock is where it can be shown.**
   State the accent-on-surface contrast ratio numerically in the notes block. Give every interactive
   element a visible focus ring. Use real semantics — `<button>`, `<table>` with `<th scope>`, labels
   tied to inputs — so keyboard order matches visual order without effort.
4. **A tenant token that fails contrast is a finding, not something to quietly fix.** If the tenant's
   accent doesn't clear 4.5:1 on its own surface, keep the real value, mark it on the page, and report
   it as a `⚠️` and a candidate acceptance criterion. Silently darkening a client's brand colour to make
   your mock look good hides a live accessibility defect — same reasoning as a source conflict in
   `repo-context.md`: the conflict *is* the finding.
5. **Both viewports on one page (UI1).** Name the breakpoint and where it came from — a repo
   `$breakpoint-*` value, or explicitly labelled an assumption (UI3).
6. **Interactive, barely animated.** These pull in opposite directions and both are required. The
   reviewer must be able to drive the mock — click the control, watch the state change, go back;
   that is the whole reason a mock beats prose, and § *Interaction* above is the contract.
   **Animation is the opposite**: keep transitions at or near zero, respect
   `prefers-reduced-motion`, and never make a reviewer wait out a timed sequence to see a state.
   Interaction reveals; animation makes a mock read as a build.

---

## Mechanical checks before publishing

Eight checks, each one a measurement rather than a judgement, and each one a defect that has shipped
on a real prototype. Run them on the published page (`skills/prototype/SKILL.md` Step 5).

1. **Declared type survives the cascade** — computed `font-size` and `font-weight` match what each
   component rule declares. A specificity loss looks identical to a design choice, which is why nobody
   catches it by eye.
2. **Column fractions match the template**, row by row, at a wide width **and** below `md`.
3. **No stranded colour tokens** — every custom property gets its first definition on bare `:root`, all
   three theme states resolve, and `body` paints an explicit token background.
4. **No horizontal overflow** — for the page body and every frame, `scrollWidth === clientWidth`.
5. **Fonts and icon glyphs actually loaded** — `document.fonts` reports each embedded face as *loaded*,
   not merely declared. A failed data URI falls back silently and the page still looks deliberate.
6. **Zero unlabelled placeholders.**
7. **Every state in `SHOWS` reachable by clicking.**
8. **No external host** beyond the allowed font hosts.

---

## When to offer a prototype

This is the canonical copy of the criteria. `ticket-create` and `ticket-review` carry a short
restatement inline rather than reading this file — same arrangement as the matrix: one authority, cheap
local copy. The command is **`/mock`**, not `/prototype`: a command named after a skill shadows it
(see the plugin README).

**All four must hold**, or it's a plain `⚠️ UI2` and nothing else:

1. **UI-facing** — the change alters layout, control placement, states, or adds a screen or region. A
   copy change, or a behaviour fix with no visual consequence, is not UI-facing merely because a screen
   is involved.
2. **UI2 genuinely unsatisfied** — no Figma or design link on the ticket, and none on the linked
   Confluence page *when Confluence was in scope*. An **unchecked** UI2 is not grounds to offer: "we
   didn't look" and "it isn't there" are different findings, and only one is a reason to draw something.
3. **Visually ambiguous from the text** — two materially different layouts would both pass the
   acceptance criteria as written. If exactly one thing can be drawn from the description, a mock adds
   nothing and costs a review cycle.
4. **There is something to draw** — a described region or screen. A one-line behaviour tweak inside an
   existing control is not a picture.

**Don't offer** when:

- The ticket is a **Bug with screenshots attached** (BG6). A mock of what's broken is worse than the
  real screenshot the ticket already has, and a mock of the fix is a design decision the bug didn't ask
  for.
- **A design link already exists**, even a stale one. That's a currency question, not a missing
  prototype.
- The requester **already declined for this ticket**, or twice this session. Offer at most once per
  ticket.
- It's a **variant in a set already mocked**, or a per-environment variant (TE2) where the visual is
  identical by construction.
- It's a **batch audit**. An epic sweep would fire the offer twelve times — use one aggregate line
  instead: *4 of 6 stories have no design link (UI2)*.
- The work is **a restricted-board request, DB, API, integration or data-import**. No visual surface.

Note the deliberate asymmetry with Confluence: the Confluence answer is asked once and carries for the
**session**; the prototype offer is **per ticket**, because a mock is a property of one ticket's
ambiguity, not a standing permission. Don't harmonise those into a session-wide decline.

---

## What a prototype is not

1. **Not a spec.** Requirements and AC live in the ticket. If the page shows behaviour the AC don't
   cover, either the AC are incomplete — say so, and offer the edit — or the mock overreached. Both are
   worth catching, and neither is fixed by leaving the page as the source of truth.
2. **Not an AC substitute, and not a matrix pass.** It satisfies UI2. UI1, UI3, UI4, UI5, UI6 and UI7
   are exactly as answered as they were before. A ticket that *looks* designed with UI3 still blank is
   worse off than one that obviously wasn't.
3. **Not a promise of implementation.** The page is HTML and CSS; the product is built with the
   framework and component library in profile § 11. Anything drawn must map to a component that
   library has, or be annotated `no <library> equivalent — needs feasibility` (UI7). Inventing a
   control the library doesn't have moves the argument from the ticket into the sprint. No library
   named and none inferable → annotate custom controls `feasibility unverified`.
4. **Not liftable code.** Say so on the page. No framework scaffolding, no class names mimicking the
   product's, no component naming that invites a copy-paste. The page has real behaviour — it has to,
   or a reviewer can't drive it — but that behaviour is *demonstration*, not an implementation: plain
   inlined vanilla JS with no state management, no data layer and no structure worth copying. A
   developer reading it should find nothing to lift; a reviewer clicking it should find everything to
   review.
5. **Not a design review, and not UIUX's sign-off.** Where a designer is on the work, the mock is input
   to them.
6. **Not client-facing, not final, not evidence.** A screenshot of the real product taken by a person
   documents the real product (BG6); this documents an idea.

---

## Rules that don't bend

1. **Approval twice.** Once to build and publish, once for the ticket edit. Neither implies the other,
   and approval of one version is not approval of the next.
2. **The banner, the title marker, the synthetic-data notice and the version line are permanent.** No
   request removes them. A page without them doesn't get published.
3. **No invented brand.** Tokens come from the repo, or you use a named fallback and say which. Never
   the client's live site, never your memory of their palette.
4. **No real data, ever.** Not a name, not an amount, not an email, not an ID.
5. **Read the published page back.** Confirm the banner and both themes rendered, and report the real
   URL — never a predicted one.
6. **The page is private and stays that way.** Sharing is the requester's decision and the requester's
   action, and the link isn't worth putting on a ticket until they've done it.
7. **A mock nobody can click is not finished.** Every `SHOWS` state reachable, every control that
   carries the argument wired, keyboard included. See § *Interaction*.

---

## Fidelity ledger

Ten defects that recur on generated prototypes, all answerable from information already in the
connected checkout. Run this list before handing a prototype over —
it's a checklist, not a history.

| # | Defect | Caught by |
|---|--------|-----------|
| 1 | Geometry taken from the ticket's mockup CSS, not the template's `col-md-3 col-6` | § *Style discovery* / SKILL Step 2 |
| 2 | Mobile stacked a label and control that `col-6 col-6` keeps side by side | SKILL Step 2 |
| 3 | A template field omitted in silence | SKILL Step 2 |
| 4 | Row spacing invented — 14px where the template says `mb-4` | SKILL Step 2 |
| 5 | A dashed placeholder where the product's own lockup ships in the repo | § *Asset inventory* · § *Where the tokens live* |
| 6 | Unicode lookalikes instead of the product's icon font | § *Asset inventory* |
| 7 | A CDN typeface instead of the repo's own binary | § *Asset inventory* |
| 8 | An entry-point screen omitted — a home tile grid routing into three mocked pages | SKILL Step 2 |
| 9 | Vendor chrome drawn without a screenshot; hover styled against a rule that said otherwise | SKILL Step 1 |
| 10 | `font: inherit` specificity collision; `transform: scale` overflow | § *Interaction* · § *Mechanical checks before publishing* |

The two that get written down and still not followed are 1 and 3, because both ask for work **before**
the drawing starts. A source manifest in the proposal — verbatim grid classes, field list, assets
resolved — is what makes them visible to the requester instead of invisible in the markup.
