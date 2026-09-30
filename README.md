# Jira Workflow Plugin

Shared workflow for creating and reviewing Jira tickets and Confluence documentation — for BA,
dev and QA alike.

## What this plugin does

Enforces a single shared standard for ticket work across your team's Jira projects. Everything
specific to *your* projects — keys, conventions, environments, stack — lives in one file,
`references/project-profile.md`; see *Adapting it to your project* below.

- **`/ticket` — the fast path.** One-line description → pick type + project → approve → created.
  Two turns, no heavy reference reading.
- **Creates tickets** from descriptions, dev research notes, or pattern-based bulk generation
- **Splits meeting and call notes** into separate tickets — it lists the work items it found, and
  what it deliberately didn't draft, and waits for you to confirm before researching or drafting
  anything. No umbrella epic invented to bundle unrelated items
- **Reviews every new ticket automatically, on a second model** — see below
- **Reviews existing tickets** against the team's schema and dependency matrix
- **Asks whether to use Confluence**, then searches specs before drafting and creates or updates
  pages when you ask, using a fixed template
- **Reads the connected repo's docs** when you're working inside a checkout, so tickets carry the
  names the code actually uses
- **Mocks up a UI ticket when you ask** — builds a standalone **interactive** HTML/CSS/JS page styled
  with the tenant's own design tokens from the connected repo, publishes it as a Claude Code Artifact,
  and links that URL on the ticket (UI2). You click through the states rather than looking at a still.
  Offered when a UI ticket has no design and the words leave the layout ambiguous — never built
  silently, and a designer's Figma still wins
- **Offers what follows an accepted spec** — the epic, the split into stories, mocks for the UI ones,
  the spec's traceability table. A menu where every item is approved separately, not a pipeline
- **Builds the test matrix** (`/test-matrix`) — crosses the requirements, AC and mock states against
  the variation axes the dependency matrix already declares, and reports which cells nothing covers.
  It writes no test cases: rows carry one-line intent and hand off to the QA skills for steps
- **Checks a whole family for drift** (`/spec-check`) — a spec, its epic and its children read
  together, reporting where they've stopped agreeing with each other. This is the one check the
  per-ticket reviews cannot run
- **Takes sources in any language** — drafts the ticket in English, but keeps UI labels, menu paths,
  error text and quoted client wording **verbatim in the original** with the English alongside (U16).
  Those are the strings people search Jira for, and a translated one matches nothing in the product
- **Scans for blockers** and stale unanswered questions

Every full-pass ticket goes through a dependency matrix — project-specific "don't forget" rules
that catch what individuals might miss (mobile scope on UI tickets, reversal handling on
transaction tickets, etc.). The fast path keeps the naming, label, priority and assignee rules and skips the
full matrix pass, saying so in the draft.

## Adapting it to your project

The plugin ships generic. One file makes it yours: **`references/project-profile.md`**. It holds
everything that differs between teams, and every skill and the reviewer agent read it before they
draft or review anything:

| Section | What you fill in |
|---------|------------------|
| 1. Site | Jira/Confluence site, team name, product name, default spec spaces |
| 2. Projects | Keys, what belongs in each, restricted boards (e.g. data-only), aliases that aren't keys |
| 3. Summary convention | Discipline prefixes (`DEV - `, `DB - `…) and per-project variants — or none |
| 4. Issue types | Which real type each template maps to, dormant types to avoid |
| 5. Priority scheme | Your priority names and which urgency signal maps to which |
| 6. Labels | The live label vocabulary, AI-draft and not-testable markers, components |
| 7–8. Hierarchy, workflow | Whether stories need epics; statuses that mean different things |
| 9. Environments | Environment names, host patterns, setup identifiers |
| 10. Tenancy | Multi-tenant or not; what a tenant is called; tenant-gated UI variants |
| 11. Tech stack | Frontend framework, component library, where components and design tokens live |
| 12–14. Data, QA, notifications | Script locations, test-management tool, notification channels |
| 15. Calibration tickets | Real tickets from your boards that show what "good" looks like |

**Minimum to start: nothing.** A blank value is never guessed. The plugin discovers what it can
through the connector (projects, issue types, priorities), asks the requester once for the rest, and
names every gap in the draft's `Not read` line — which also tells you which profile entries are worth
filling in first. In practice, sections 2, 3 and 6 are the ones that make drafts look like your
team's tickets from day one.

Three more files are worth a pass in the first weeks, in this order:

1. **`references/dependency-matrix.md`** — the rules ship as generic defaults. Delete the rows that
   never apply to your product, cite a real ticket next to the ones that do, and add your own
   "we always forget X" rules as new sections.
2. **`skills/ticket-create/references/examples.md`** — illustrative examples for a fictional product.
   Swap in your best real tickets as you find them.
3. **`references/prototype.md`** — only if you use `/mock`: point the token-discovery table at where
   your repo actually keeps its design tokens (or just fill in profile § 11).

## Three rules worth knowing before you read a draft

**Requirements are business behaviour, not implementation (U15).** A requirement names the user, the
rule and the states; the developer picks the mechanism. "Add a new role — Team Lead — who sees only
their own team's data; see PROJ-xxxx for the last one we added" is a requirement. "Extend
`AI.CommandLkp` with an audience column, seeded via the post-deployment script" is a design decision
smuggled into a ticket nobody reviewed it in — and it reads *more* rigorous, which is why the
second-model reviewer checks for it specifically. Four things stay exact because they're contracts
rather than design: task config values, bug identifiers, upload-file columns, and notification event
and token lists. `references/ticket-schema.md` § *Where exact values still belong*.

**Three markers, three different meanings (U17).** `[defaulted]` on a field means *nobody chose this
value*. `⚠️` means *a check didn't pass*. `(derived — confirm)` on a requirement means *the source
never said this* — it's scope the drafter inferred, tagged so you can strike it in one word. The
inference itself is wanted: the source describes approval and says nothing about rejection, and a
ticket that stays silent there ships a half-built feature. What isn't wanted is the same line
untagged, because it then reads exactly like something the client asked for and nobody downstream can
tell. Blocking questions come as **one numbered list with the draft**, not one per turn.

All three are **draft markers and none of them reaches Jira** (U18a). They're resolved at the create:
a derived line is confirmed, struck, or kept with its question moved to a Jira comment. See *Voice*
below.

**Nothing is drafted before the research is reported.** The full pass shows a `RESEARCH` block first —
epic, findings read on **Done** research siblings (RS5), Confluence, repo docs, repo source — and a
`Not read` line naming the gaps. The fast path shows the same thing shrunk to `Read:` / `Not read:`.
The gaps line is the load-bearing half: a reader who knows what was skipped reads the draft
correctly, and a reader who doesn't reads silence as coverage.

## The chain, and the drift check

Every other skill here works on **one artifact**. `spec-flow` is the only one that works on the
**set** — a spec page, its epic, that epic's children, and the mocks they link — and both of its
workflows exist because a set can be broken while every member of it passes its own review.

**The chain (`/spec`).** Once a spec is accepted, one menu: create the epic · split the requirements
into stories · mock the UI ones · build each story's test matrix · fill in the page's traceability
table. Three properties keep it honest:

- **The menu is not consent.** Say "epic" and you get the epic and nothing else. Nothing cascades.
- **The split lists before it drafts** — one line per proposed story with the requirement numbers it
  covers, plus a `Not split out` block for requirements folded into someone else's AC. You're
  reviewing our reading of *your* spec, and after the tickets exist, disagreeing costs a rewrite.
- **It's not a fast lane.** Every story still runs the full pass — research gate, schema, the whole
  matrix, second-model review. One approval turning into five unchecked stories would be worse than
  the manual path it replaces.

There's a real payoff buried in that last point. The reviewer's faithfulness check (U17) normally
degrades to *not checkable*, because a ticket drafted from a chat message has no durable source to
trace back to. A spec-chain story has one by construction: the reviewer gets the spec URL and the
**verbatim requirement text**, so "this AC covers behaviour the requirement never mentions" and
"this clause of the requirement no AC covers" both become findings instead of guesses.

**The drift check (`/spec-check`).** Give it a spec page or an epic key and it reads the whole family
at once: requirements whose AC now say something different (`CONTRADICTS`), requirements no story
implements (`UNBUILT`), children tracing to no requirement (`UNSPECCED`), two siblings stating one
rule two ways (`SIBLINGS DISAGREE`), a mock published before the requirement changed under it. Then
one proposed edit per artifact, approved individually — there is no "fix all", because the point of
the pass is that *you* decide which side of each disagreement moves.

Two things it will not do. **It never resolves a disagreement**: both sides are quoted, both sources
named, and the timestamps go in as context — Confluence gives one modified date for a whole page and
Jira one per issue, so "the page is newer" is a hint and never a verdict. And **it always says what
it didn't check**, on a mandatory `NOT CHECKED` line. A drift report with no scope line reads as full
coverage of the set, and the only thing worse than finding drift late is believing you'd looked.

**Why there's no git trigger.** The workflow this was modelled on treated *pushing the spec* as the
acceptance signal. We didn't build that, deliberately. Claude Code has no daemon watching your
remote, so the closest honest thing is a session-start diff — which fires whenever you next open a
session rather than when you push, and would load in every repo every dev opens. That is precisely
the mistake the `PreToolUse` hook made before 0.5.0 removed it (see *Ticket work doesn't change the
repo*). Saying "the spec is accepted" gets the same behaviour with no false positives, works
identically with no repo checked out, and specs stay where the team already reads them: Confluence.

## Two models, not one

A model reviewing a ticket it wrote thirty seconds ago is grading its own assumptions. So the
review is handed to someone else: **every ticket created by `/ticket` or `ticket-create` is
immediately reviewed by the `ticket-reviewer` subagent, running on a different model.**

It fetches the ticket from Jira rather than reading the draft, which is what catches the silent
failures — an epic link that didn't take, a description that flattened into one paragraph, AC that
only reads as testable to whoever wrote it. It is read-only: it proposes rewrites, and nothing is
edited until you approve the specific rewrite. A "needs fixes" verdict never rolls a ticket back —
the ticket exists, and the fix is your call.

It also gets **the source material** — your message, the notes, the dev write-up, verbatim — because
the one thing no reviewer can reconstruct from Jira is whether a requirement traces back to something
a person actually said. That's the check for invented scope (U17), and it runs in both directions: a
line in the ticket that traces to nothing, and a line in your notes that never made it in. On notes
you split into several tickets, the reviewer is judged against the item list *you confirmed*, so
something you dropped isn't reported as missing. If no source is passed it says so on a `SCOPE` line
rather than letting "ready for dev" imply a check it couldn't run.

It can't ask you anything — there's nobody on the other end of a subagent's context — so it's *told*
instead: your Confluence answer is passed down with the ticket key. No answer passed means it makes no
Confluence calls, and says so in its findings rather than guessing.

**This costs wall-clock, including on the fast path.** `/ticket` promises **two user turns**, not a fast
clock — after you say `go` there's a second model reading the schema and the matrix before you see the
result. That's the trade on purpose: the fast path skips those checks while drafting, so the only thing
standing between a two-turn ticket and an unchecked one is a review that actually runs before you move
on. A review delivered after you've closed the tab is a review nobody reads. If you want the ticket and
nothing else, say so and the review is skipped — but then the footer's coverage claims go with it.

`/review-ticket` on an existing ticket still runs in your own session. There's no self-review
problem there, and no reason to pay for a subagent.

## Components

| Component | Purpose |
|-----------|---------|
| `/ticket` | Slash command → fast ticket creation |
| `/review-ticket` | Slash command → review a ticket or epic |
| `/mock` | Slash command → build and publish an interactive mockup for a UI ticket |
| `/spec` | Slash command → a spec was accepted: offer the epic, the split, the mocks, the traceability table |
| `/spec-check` | Slash command → report where a spec, its epic and its children have drifted apart |
| `/test-matrix` | Slash command → build the test matrix for a ticket or spec: which combinations need testing, and which nothing covers |
| `skills/quick-ticket` | Fast path: one-liner → type/project pick → compact draft → create → auto-review |
| `skills/ticket-create` | Full pass: tickets, variants, dev notes, Confluence pages |
| `skills/ticket-review` | Review tickets, batch audit, check the linked spec, scan for blockers |
| `skills/spec-flow` | The **set**, not one ticket: the post-acceptance chain, and the drift check across a spec and its children |
| `skills/prototype` | Reads the connected repo's design tokens, builds a standalone interactive HTML/CSS/JS mockup, publishes it as an artifact, hands back the URL for the ticket |
| `skills/test-scope` | The **test matrix**: crosses the requirements, AC and mock states against the variation axes the dependency matrix declares, and reports the cells nothing covers |
| `agents/ticket-reviewer` | Read-only post-create review, pinned to a **different model** |
| `references/project-profile.md` | **Your project's settings** — the one file to edit when adopting the plugin |
| `references/` | The shared standard — ticket schema, per-type templates, voice, dependency matrix, the reviewer spawn contract, Confluence rules + page template, repo-docs rules, prototype rules, test-matrix rules. One copy each, read by the skills *and* the reviewer agent |
| `.mcp.json` | Atlassian Rovo connector (Jira + Confluence) |

**Commands are never named after a skill, deliberately.** `/mock` invokes the `prototype` skill,
`/spec` and `/spec-check` both invoke `spec-flow`, `/test-matrix` invokes `test-scope`, and
`/review-ticket` invokes `ticket-review`, and the mismatch is the fix rather than an oversight:
commands and skills share one namespace, so a command called `prototype` **shadows** the skill of the
same name. Until 0.6.0 both did, which meant invoking `prototype` returned the command's one-line body
— `Use the prototype skill for: …` — and the skill never loaded. Every hand-off degraded to an echo,
silently. If you add a command, check its filename against `skills/*/` first.

## Setup

Put the plugin in a marketplace your team can reach — a git repo with a `marketplace.json`, or a local
directory — then enable it in **your own** `~/.claude/settings.json`:

```
/plugin marketplace add <your-org>/<your-toolkit-repo>     # or a local path: /plugin marketplace add .
/plugin install jira-workflow
```

Two things worth knowing:

- **`enabledPlugins` goes in user settings, never a project's `.claude/settings.json`.** Settings
  precedence is user < project, so a project entry creates a *second, project-scoped install* that
  pins to an old commit and never updates — duplicated skills and commands, and a repo quietly running
  an old copy of this plugin.
- **Updates need a marketplace refresh and a restart.** `autoUpdate` refreshes the marketplace clone on
  Claude Code's own schedule — roughly hourly — so a version merged to `main` can sit unseen until the
  next refresh. A `SessionStart` hook that runs the refresh forces it at every session start. A merge
  never reaches a session that is already running, and hooks load at session start, so a **restart**
  is always needed to pick up a new version. Without auto-update, run
  `/plugin marketplace update <marketplace-name>` by hand.

Then connect your Atlassian account via Claude connectors settings, fill in as much of
`references/project-profile.md` as you know, and start with `/ticket <what you need>`,
"Create a story for...", or "Review PROJ-123".

### What the host has to provide

The plugin ships its own skills, commands, agent and connector, but two things it depends on come from
the environment rather than from this repo:

| Needed by | What | If it's absent |
|-----------|------|----------------|
| everything | The **Atlassian Rovo** connector, authorized | No Jira or Confluence calls work at all |
| `/mock` | The **Artifact** capability — the publishing tool plus its bundled `artifact-design` skill | `/mock` degrades rather than failing: it styles from the fallback token contract written into `references/prototype.md`, and if publishing itself is unavailable it hands back the local HTML file path and leaves UI2 honestly unsatisfied. It never puts a local path on a ticket in place of a URL |
| `/test-matrix` | Optional QA hand-off skills (test-case writing, case search, question raising) and a **test-management MCP** — whichever the team names in profile § 13 | Each hand-off is optional and named. Without a test-case skill the rows are handed back as they are; without a test-management connector the `Case` column is **omitted** and `NOT CHECKED` says existing cases weren't searched. The matrix itself needs none of them |

`artifact-design` is not a plugin dependency and can't be declared in `plugin.json` — a plugin manifest
lists what a plugin *provides*, not what the host must supply. That's why the fallback lives in the
reference file instead.

## Ticket work doesn't change the repo

Ticket work reads the codebase; it never changes it. That's rule 7 in
`references/repo-context.md`, which every skill reads when it's working inside a checkout: no edits,
no commits, no installs, not even a "while I'm here" fix. What needs changing goes *in the ticket*.

It's a rule Claude follows, not something the plugin enforces. **0.4.0 shipped a `PreToolUse` hook
that blocked writes; 0.5.0 removes it.** Two reasons, both learned the hard way:

- **It policed everybody to protect one flow.** The plugin is enabled at user scope, so a guard
  written for BA ticket work loaded in every repo every dev opened, and blocked them from coding in
  it. That's the plugin picking a fight it has no business picking.
- **Guessing intent from shell strings can't be made right.** The pattern list denied `mkdir`, `cp`,
  `touch`, `git add` and anything containing `>` — which meant it also denied `git branch --contains
  <sha> 2>/dev/null`, a pure inspection command, because `2>/dev/null` looks like a file write. That
  false-positive surface has no bottom, and every miss cost somebody a restart.

If the words alone aren't enough for your machine, enforce it with the harness instead of a plugin —
in **your own** `~/.claude/settings.json`:

```json
{
  "permissions": {
    "deny": ["Edit", "Write", "NotebookEdit", "Bash(git commit:*)", "Bash(git push:*)"]
  }
}
```

Better than the hook on every axis that mattered: Claude Code enforces it rather than a shell script,
it needs no Git Bash, it doesn't guess at command strings, and it's set by the person it applies to.
If you only ever open a product repo to read context for a ticket, this is the line worth having.
Note that it stops `/mock` from writing its mockup HTML too — add an `allow` for your temp
directory if you use it.

## Customization

The ticket schema and the dependency matrix are the shared standard, so they live once at the
**plugin root** in `references/` — not inside any one skill. `ticket-create`, `ticket-review` and
`spec-flow` read those same files (`../../references/…` from their `SKILL.md`), which is what stops a
review from being run against a different version of the standard than the draft was written to.
Edit them in one place; there is no copy to sync.

### Project profile
`references/project-profile.md` is the only file where project-specific **values** live — keys,
prefixes, types, priorities, labels, environments, tenancy, stack. The schema, the matrix and the
skills say *what to do* with those values and point at the profile's sections by number, so a new team
edits the profile and leaves the standard alone. If you find yourself hard-coding a key, a label or a
hostname into any other file, it belongs in the profile instead, with the rule pointing at it.

### How the files are sized, and why it matters

A `SKILL.md` is loaded **whole** the moment its skill triggers; a `references/` file is read **on
demand**. That single mechanic decides where a rule belongs, and it's the reason these files are
split the way they are rather than by topic or by length:

- **A rule that must apply on every run belongs in `SKILL.md`.** Move it into a reference file and it
  silently stops applying on the runs that don't open that file. That's a correctness bug wearing the
  costume of tidying up, and it leaves no trace when it fires.
- **Content you need for *one* case belongs in a reference.** The per-type body templates, the
  mixed-source and variant workflows, the two `spec-flow` workflows — you use one at a time, so
  carrying all of them through every invocation is pure waste.
- **Two skills needing the same procedure means one file, not two copies.** `references/auto-review.md`
  exists because `quick-ticket`, `ticket-create` and `spec-flow` each used to carry their own copy of
  how to spawn the reviewer. Three copies of a contract drift, and the one that drifts is the one
  nobody re-reads. `references/test-matrix.md` sits at the root for the same reason: `test-scope`
  builds the grid, and `spec-flow`'s chain counts axes off the same table.

Target 200–300 lines per file. `ticket-create/SKILL.md` sits above that on purpose: what's left is
Steps A–G, which every ticket needs. Cutting *that* to hit a number would trade a real check for a
tidier file.

### Dependency Matrix
Edit `references/dependency-matrix.md` to add project-specific rules. Each new row becomes a check
Claude applies automatically on every full-pass ticket. It stays one file deliberately — the rule is
"check every applicable row", and a split matrix invites a half-checked one.

### Ticket Schema
`references/ticket-schema.md` holds what applies to **every** ticket: the draft preview format, the
summary line, priority, assignee, labels, how to write requirements, field rules, connector rules.

`references/ticket-types.md` holds the **per-type body templates** — Story, Task, Bug, Improvement,
Research, Epic — and you read one section, for the type you're drafting. They're separate because the
schema is read on every ticket and six templates meant reading five you didn't need.

If you add a type, add it to *both*: the type list in the schema and a template section here.

### Voice
`references/voice.md` is the third file in that set, and the only one about *how the ticket reads*
rather than what it contains. It carries rule **U18**, which has two halves that behave very
differently:

- **U18a — the ticket is not the conversation.** `RESEARCH`, `DEPENDENCY CHECKS`, `⚠️`,
  `[defaulted]` and `(derived — confirm)` are decision aids for the requester approving a draft. They
  are stripped before the create, and a description carrying them tells everyone who opens the ticket
  that nobody re-read it. This is a defect and the reviewer reports it as MISSING. The block-by-block
  boundary lives in `ticket-schema.md` § *What goes into Jira, and what stays in the chat*.
- **U18b — register.** Length matched to the work, isolation and regression criteria that name what
  actually breaks, the product's own words kept, and the generated-text phrasing avoided. Capped at
  **two findings per review** and never blocking — style findings that crowd out correctness ones are
  a worse outcome than a stiffly-written ticket.

Stripping the `(derived — confirm)` tags means the live ticket no longer shows which requirements
were inferred, so every create path now passes **the derived lines and how each resolved** to the
reviewer (`auto-review.md` item 5). That list is the only place the distinction survives the create;
without it U17 cannot run. If you add a create path, pass it.

`ai-draft` is unaffected and stays on every drafted ticket (U6). U18 is about prose, not provenance.

### Reviewer spawn contract
`references/auto-review.md` is what every create path passes to the `ticket-reviewer` subagent.
Change what the reviewer receives here, once, and all three paths change together. If you add a
create path, give it a short "what's specific to this path" block and point it at this file — don't
copy the procedure in.

### Quick Card
`skills/quick-ticket/references/quick-card.md` is the one-page distillation the fast path reads.
When a schema or matrix rule changes and it's one the fast path must honour, update the card too —
and keep it under ~100 lines, or the fast path stops being fast.

### Reviewer model
`agents/ticket-reviewer.md` sets `model: sonnet` in its frontmatter — that one line is the knob.
Change it if you want the review on a different model. If your own session is already running the
same model the agent is pinned to, the skills pass an override so the review is still a genuinely
different model; you don't have to manage that.

### Repo docs
`references/repo-context.md` controls what happens when the plugin runs inside a checkout: where it
looks (`docs/`, `Docs/`, ADRs, per-module `README`s, root `*.md`), what it trusts them for, and what
never leaves the repo.

**It reads source too, deliberately.** "Don't infer requirements from source" used to read as a
blanket ban and discouraged opening a component at all — which is how a mock ends up inventing UI the
product doesn't have. It's now split: what the product *should do next* comes from Confluence, the epic
and you; what it *does today* comes from the code, because a doc can lag it by months. There's a
per-ticket-kind table of what to open (UI → the component template and TS, mandatory before a mock;
Bug/Improvement → the path behind the "current behaviour" claim), capped at about five files and named
in the draft's `RESEARCH` block.

The precedence rule is the part worth knowing. **Confluence is authoritative for what should be
built; the repo is authoritative for how the system behaves today.** When they disagree on the same
fact, that disagreement goes in the draft as a `⚠️` with both sides quoted — it's never resolved
silently, because a spec the code has diverged from is exactly what the team needs told about.

The fast `/ticket` path still skips the `docs/` sweep to stay fast, and the second-model reviewer picks
that up after the ticket is created. It keeps one repo read: **on a UI ticket it opens the component**,
because a ticket that contradicts the controls the product actually has is wrong in a way no reviewer
can catch from Jira alone. No repo, or no doc covering the feature, is a normal state — it's reported
in the footer, never treated as a gap.

### Prototypes
`references/prototype.md` holds the mockup rules: how the product's (or tenant's) design tokens are
found in the connected repo (theme/SCSS variables, CSS custom properties, token files, component-library
theme overrides), what the generated page may contain, the interaction contract, and the
publish-and-link rules. **Tell it where your repo keeps tokens and components in profile § 11** — that's
the one genuinely per-repo part of the mock workflow.

**The mock is drivable, and that's the point.** Every state listed in the proposal is reachable by
clicking; the control the ticket is actually about is wired rather than drawn; desktop renders at the
page's full width with mobile stacked beneath it, each working on its own; it's keyboard operable,
because accessibility is manually verified here (UI4) and the mock is where that gets shown. Inlined
vanilla JS only — no framework, and nothing a developer could lift. Animation is the opposite of
interaction and stays near zero: a page that animates gets mistaken for a build.

**Screenshots are asked for first, and the real assets go on the page.** If the ticket touches an
existing screen, the skill asks for screenshots — including each vendor control in its *open* state —
before it proposes anything, because a third-party library control exists in the checkout only as a tag
and its toolbar, filter row and close button are drawn at runtime. Geometry then comes from the component
template, verbatim grid classes and all, never from a mockup the ticket happens to ship. Logos, icon
fonts, typefaces and background images are inlined from their real paths as data URIs; a placeholder is
a last resort that gets labelled on the page and justified in the handback.

**Tokens are the second thing it reads.** The component's markup and logic come first, and that's
mandatory. A stylesheet gives you the palette and tells you nothing about which controls exist or
which library components are already in use, so a mock built from SCSS alone invents chrome the product
doesn't have — and then annotates as "no library equivalent" something the library has shipped for
years.

Two things it deliberately does not do. It **never builds a mockup unprompted**: one is offered only when
a UI ticket has no design, the layout is ambiguous from the words alone, and UI2 would otherwise ship as a
flag — and you have to say yes. And it **never replaces a designer**: a Figma link still wins, and a
generated mockup is something to point at during refinement, not an approved design. The ticket says which
kind it carries, every time.

The URL goes on the ticket **as a link, never as an attachment** — the same reasoning DB1 uses to keep
scripts out of Jira. The page itself is private when published; sharing it with the team is your action,
and the link isn't worth putting on a ticket until you have, or every developer who clicks it hits an auth
wall.

The second-model reviewer checks that the URL actually landed on the ticket and that the acceptance
criteria cover what the mockup is *said* to show and *said* to do — but it does **not** open the artifact
page, and it says so on a `SCOPE` line. Those pages are private to your session, so a subagent claiming
to have looked at one would be making it up. Opening the mock, clicking through it and sanity-checking
the AC against it is yours.

One honest failure worth knowing about: **the session that publishes a mock often can't open it.** The
in-app browser isn't signed in to claude.ai, so the read-back can hit an auth wall. When that happens
the skill says the rendering wasn't verified and hands you the local file path alongside the URL,
rather than claiming a check it didn't run.

### Test matrix
`references/test-matrix.md` holds the axis table, the pruning rules, the marker vocabulary and the
Jira write rules. **The axes are derived from the dependency matrix, never invented** — UI1 viewport,
U11 tenant, UI6 variant, RP3/RL1 toggle, NT3 channel, RL4/RP2 role, NT6 language, TE1 environment,
TX3/RP4/IM4 boundaries, UI5 browser. Add an axis by adding the rule to the matrix first; a row here
with no rule behind it is a guess wearing a rule's clothes.

**The empty cell is the deliverable.** Same doctrine as the Traceability table: a grid listing only
what will be tested cannot tell you what was missed. So a combination nobody specified appears as
`⚠️`, with its rule ID and a reason, rather than being quietly left out.

Three rules keep it from becoming a second test-case writer or an unreadable cross-product:

- **It never writes a test case.** A row carries a one-line intent; steps, preconditions and
  test-management fields belong to QA, or to a test-case skill named in profile § 13, which this skill
  hands off to. Two implementations of that
  contract would drift, and the one that drifts is the one nobody re-reads.
- **Two axes crossed, never three**, and only for a pair on the sourced interaction list — each with
  the real defect that proved the interaction. Everything else is listed one row at a time. Six axes
  multiplied is several hundred cells, which is a way of not having a plan.
- **`UI5` is the honesty test.** It's the thinnest rule in the matrix — one sentence, no browsers
  enumerated — so when a ticket says nothing about browsers there is nothing to fall
  back on, and the axis is reported `⚠️ UI5 unanswered` rather than filled in with what the product
  probably supports.

One collision worth knowing if you edit the axis table: **UI1's viewport and NT3's channel both want
the word "mobile"**, and a grid carrying both loses a dimension silently. Viewports are written as
widths (`1280px`, `375px`); the channel is `Mobile app`.

The matrix goes on the **ticket**, not the spec page — a comment, plus an optionally offered
test-case subtask (QA4, when profile § 13 names one), each its own approval. It never opens a prototype artifact:
those pages are private to your session, so the mock's states come from its `SHOWS` list, from the
`Shows:` line `/mock` already wrote on the ticket, or from you.

### Confluence
**You're asked once per session whether to use Confluence at all** — *search for context* (read-only),
*search and write*, or *skip*. Nothing touches Confluence before that answer, including the
linked-doc check, and the answer carries for the rest of the session rather than being asked per
ticket. On the `/ticket` fast path the question rides along in the same prompt as type and project, so
it still costs one turn. If the connector isn't authorized you aren't asked at all — the plugin says
Confluence is unavailable and carries on.

Skipping changes what the plugin is allowed to *claim*: U3 reports `⚠️ not checked — Confluence out of
scope`, never "no page exists". "We didn't look" and "it isn't there" are different findings, and only
one of them is a reason to go write a spec.

`references/confluence.md` holds the read/write mechanics and the guardrails — search before creating,
confirm space and parent, read-and-version before updating, approval per publish.
`references/doc-template.md` holds the page shape. Both live at the plugin root because
`ticket-create`, `ticket-review` and `spec-flow` all write pages now, so there is one copy of each.

Writing stays doubly gated: it needs the *search and write* answer **and** a request for that specific
page. "Search and write" is a permission, not an instruction — every publish and every update still
stops for approval on the actual content.

A skip bites harder on `spec-flow` than anywhere else: it removes the spec, which is the thing both
of that skill's workflows are *about*. The chain has nothing to chain from, and `/spec-check` reports
that it didn't run. What it must never do is reconstruct the spec from the tickets and grade the
tickets against that — it would agree with itself by construction and return a clean report having
checked nothing.

### Traceability
`references/doc-template.md` gained a **Traceability** section — `| Req # | Story | Status |` — and
that table is what `/spec-check` reads to know which story came from which requirement. Two design
notes if you're editing the template:

- **A requirement nothing implements gets a row too**, with `—` and a reason. A table listing only
  what exists cannot tell you what's missing, which is half of why the table is there.
- **A page without the section is normal, not a gap.** Every spec written before it existed lacks
  one. `/spec-check` infers the mapping from the text instead, marks every inferred row `~`, and
  offers to add the table. Nothing retrofits it silently, and an inferred mapping is never reported
  as a confirmed one — same instrument as `(derived — confirm)` on a requirement.

### Examples
`skills/ticket-create/references/examples.md` ships with illustrative examples for a fictional
product. Replace them with real tickets from your projects as you find them, and list the keys in
profile § 15 — the reviewer uses those as its calibration for how short a complete ticket can be.

## Usage examples

- `/ticket modal overflows on mobile on the orders page`
- `/review-ticket PROJ-123`
- `/mock PROJ-124` — or "mock this up before we write the AC"
- `/spec` — or "the spec is accepted, what's next?"
- `/spec-check PROJ-300` — or "does the spec still match the tickets?"
- `/test-matrix PROJ-124` — or "what should I test here?"
- "Did we cover the toggle-off case and the other tenant on PROJ-125?"
- "We changed the AC on PROJ-302 in grooming — what else needs updating?"
- "Create a user story for filtering orders by date in PROJ"
- "Generate tenant setup tickets for Acme Corp across Dev, Staging, and Prod"
- "Here are the dev's research notes, turn them into stories"
- "Notes from this morning's call — turn them into tickets"
- "Review PROJ-456 and all its child stories"
- "Check for blocked tickets in PROJ with no activity in 3 days"
- "Draft a Confluence page for the new reporting feature"

## License

[MIT](LICENSE) © 2026 Ihor Nesterenko
