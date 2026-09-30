# Test matrix: what to test, and what nothing covers

A test matrix is the **plan**, not the cases. It says which combinations of conditions a change has
to be exercised under, which of those are deliberately out of scope, and which are undefined in the
requirements. Rows carry a one-line intent; they never carry steps.

Read this file whenever you are building one. The workflow is in `skills/test-scope/SKILL.md`; this
is where the axes come from, how the grid is bounded, what the markers mean, and how it lands on a
ticket.

The reason this exists at all: the defects that recur on most product teams are **combinatorial**,
and the dependency matrix already says so in its own rules. NT3 — the notification channels "fail
independently". RP3 — a toggle's two states "diverge". UI6 — a legacy and a new UI live at once,
gated per tenant or flag. UI1 — a control that exists at one width and not the other. Each of those is
an axis. The matrix
enumerates them one by one; nothing until now crossed them.

---

## What this is not

1. **Not test cases.** Writing cases — preconditions, steps, expected results, test-management fields —
   belongs to QA, or to a test-case skill if profile § 13 names one. A row here is a *cell that needs a
   case*, and its "Expected" is one line. The moment this file starts specifying steps it becomes a
   second, diverging copy of a contract that already has an owner.
2. **Not a gap analysis.** A `⚠️` cell here is a finding; turning it into a question for the BA is a
   separate step (a question-raising skill, if profile § 13 names one, or the requester).
3. **Not a coverage report.** Searching the test management tool for what already exists is a
   separate step. This says what *should* exist. Where such a search is available its answers fill
   the `Case` column; where it isn't, the column is omitted and the omission is stated.
4. **Not a matrix pass.** It reads the dependency matrix to find axes. It does not check the ticket
   against the matrix — that's `ticket-review`, and a test matrix built over a ticket nobody reviewed
   is still a ticket nobody reviewed.
5. **Not a substitute for AC.** A `⚠️ no AC` row is a defect in the ticket, reported as one. Never
   quietly write the missing acceptance criterion into the grid instead: an AC that exists only in a
   QA table was agreed by nobody.

---

## Where the axes come from

**Derived from the dependency matrix, never invented.** Every axis below is declared by a rule that
already exists. As the team cites real tickets against those rules, add the defect to the last column
— an axis with a defect behind it is easier to defend in planning.

| Axis | Declared by | Values come from | Cited defect |
|------|-------------|------------------|--------------|
| Viewport | UI1, UI3 | breakpoints stated on the ticket, else the prototype's frames. Written as widths — `1280px` / `375px`, never bare "mobile" | — |
| UI variant | UI6 | legacy vs new, and who has which (profile § 10) | — |
| Tenant | U11, UI6 | tenants named on the ticket. Multi-tenant products only | — |
| Toggle state | RP3, RL1 | each toggle, ON and OFF | — |
| Channel | NT3 | the channels in profile § 14 — a mobile channel written `Mobile app`, never bare "mobile" | — |
| Role / permission | RL4, RP2 | roles named on the ticket | — |
| Recipient suppression | NT4 | who must **not** receive it | — |
| Boundary values | TX3, RP4, IM4 | zero, negative, invalid row | — |
| State machine | TX2 | the transaction / approval states | — |
| Language | NT6, U16 | locales named on the ticket | — |
| Environment | TE1, TE2 | the environment names in profile § 9 | — |
| Browser | UI5 | **the ticket only** | — |

### The browser axis is the worked example of the honesty rule

`UI5` is the thinnest rule in the whole dependency matrix: one sentence, no enumerated browsers. So when a ticket says nothing about browsers there is **nothing in the repo to fall back
on**, and the axis is not invented — not from what the product probably supports, not from what is
usual. It goes to `COLLAPSED` as `⚠️ UI5 unanswered`.

Same instrument as U3 on Confluence: *"we didn't look"* and *"it isn't there"* are different findings,
and an invented browser list turns the first into a confident-looking second.

### Axis values are disambiguated

**"mobile" is the known collision.** UI1's viewport and NT3's channel both want that word, and a grid
carrying both is unreadable — for a person, and worse for the next model, which will quietly treat
them as one axis and lose a whole dimension without saying so.

- Viewports are **widths**: `1280px`, `375px`.
- The notification channel is **`Mobile app`**.

The same rule applies to any other pair of axes that would otherwise share a token. Rename the value,
never the rule.

---

## The rule that stops the explosion

Six axes crossed is several hundred cells, which is not a plan — it's a way of not having one.

1. **An axis enters the grid only if** the ticket or spec states that behaviour varies along it, **or**
   a matrix rule for this ticket type demands it be stated. Neither → it goes to `COLLAPSED` carrying
   its rule ID, marked unanswered. Never silently dropped, and never invented.
2. **Independent axes are listed, not crossed** — one row each. Crossing happens **two axes at a time,
   never three**, and only for a pair on this sourced list:

   | Pair | Why it interacts | Source |
   |------|------------------|--------|
   | channel × viewport | a channel's control can exist at one width and not the other | NT3 + UI1 |
   | toggle × tenant | the toggle's two states diverge, and isolation is per tenant | RP3 + RP7 |
   | UI variant × tenant | the variant *is* tenant-gated | UI6 |
   | toggle direction × cascade-on-removal | grant and revoke are separate implementations | RL1 + RL3 |
   | transaction state × zero/negative | the boundary behaves differently per state | TX2 + TX3 |

   A pair not on this list is **two listed axes, not a grid**. A third axis never multiplies an
   existing cross. Add a pair when a real defect shows the interaction — and cite the ticket, the way
   every other row in the dependency matrix does.
3. **Hard cap of about 40 cells.** Over it, collapse to pairwise and say so on the `COLLAPSED` block.
   A grid nobody reads to the bottom is worth what no grid is worth.
4. **Every non-`●` cell carries a reason**, as a lettered footnote under the grid. A `⚠️` or `—`
   standing alone is the cell most likely to be wrong, and it reads as though somebody considered it.

---

## Cell vocabulary

`●` needs a test · `◐` partial · `—` out of scope · `⚠️` a finding (behaviour undefined, no AC, a rule
unanswered) · `~` on any row whose axis values were **inferred** rather than stated · a test-case ID,
in the format profile § 13 names, where a search of the test management tool found one.

`●` is the only marker that stands alone. `◐`, `—` and `⚠️` each carry a lettered footnote.

`~` and `⚠️` mean here exactly what they mean everywhere else in this plugin — inferred, and a check
that didn't pass. Don't invent a new marker: three markers with stable meanings are worth more than a
richer vocabulary nobody can read at a glance (README § *Three markers, three different meanings*).

---

## The two output shapes

### Chat — the scan grid

Wide, for spotting holes at a glance.

```
TEST MATRIX — PROJ-123 "Notification channel preferences"

AXES
  crossed   NT3 channel × UI1 viewport     sourced pair (NT3 + UI1)
  listed    RL1 toggle direction           independent — one row each

CROSSED — channel × viewport
                Email   In-app msg     Mobile app
  1280px          ●          ●            ⚠️ (a)
  375px           ●          ●            ⚠️ (a)

LISTED — RL1 toggle direction
  ●      channel ON   → delivered on that channel
  ●      channel OFF  → NOT delivered on that channel (NT4)
  ⚠️ (b)  re-enabled after OFF → no AC says whether the prior state returns

(a) AC 3 lets the user turn each channel on and off but names no control for
    Mobile app — undefined, not absent.
(b) RL3 asks for the cascade on removal; the ticket states only the grant path.

COLLAPSED — say the word if any should be split back out:
- browsers (UI5): nothing on the ticket, no list in the repo — ⚠️ UI5 unanswered
- tenant (U11): no per-tenant behaviour stated — ~inferred uniform, confirm
- environments (TE1): STG and DEV identical to QA by construction

NOT CHECKED
- mock states read from the ticket's Shows: line; page not opened (private to your session)
- existing test cases not searched — no test-management connector in this session
```

Four properties that block keeps, every time:

- **One cross, two axes.** The toggle is a third axis here, so it is *listed*, not multiplied in.
- **Every `⚠️`, `◐` and `—` has a lettered reason.** There is no `—` in the example because none is
  true for that ticket — an out-of-scope cell is never manufactured to demonstrate the notation.
- **No cell contradicts another.** If a control doesn't exist, neither of its directions is reachable;
  the defect belongs to the **channel**, so it sits in the channel column at both widths.
- **`AXES` lists only what is in the grid.** Anything collapsed appears once, under `COLLAPSED`.

### Jira — the long form

One row per cell that needs a test:

```
| Req | AC | Condition | Expected | Status | Case |
|-----|----|-----------|----------|--------|------|
| 2 | 3 | Email · 1280px | preference saves and persists | needed | — |
| 2 | 3 | Mobile app · any width | ⚠️ no control named in AC 3 | finding | — |
| 2 | — | re-enabled after OFF | ⚠️ prior state undefined (RL3) | finding | — |
```

The wide grid does not survive a Jira comment — a table that flattens into one paragraph is a failure
this plugin already knows about — and the long form gives each row somewhere to carry a test-case ID
(QA3) and a status. Same data, one row per cell.

**The `—` and `⚠️` rows still appear.** That is the Traceability doctrine applied here: a table listing
only what exists cannot tell you what is missing, which is half of why the table is worth writing.

---

## Where the prototype half comes from

**Never open the artifact page.** Those pages are private to the requester's session, so anything
fetched back is an auth wall or raw markup, and a row built on it is invented. Same rule `spec-flow`
and `ticket-reviewer` both carry — and `prototype.md` notes that even the session that published a
mock often cannot read it back.

State rows come from, in this order:

1. the `PROTOTYPE PROPOSAL` block's **`SHOWS`** list, when the mock was built in this session;
2. otherwise the **`Prototype: <url>` link and its `Shows:` one-liner**, which `prototype` Step 7
   already writes under `LINKED DOCUMENTATION` on the ticket;
3. otherwise **ask the requester** — they can open it and you cannot.

None of the three → the state rows are simply absent, and `NOT CHECKED` says the mock wasn't read.
Never write a row describing a screen you have not been told about.

**`DOES NOT SHOW` feeds `COLLAPSED` directly.** That block already lists the matrix rules the mock
left unanswered, in exactly the form this grid needs.

---

## Writing it to Jira

The matrix lands on the ticket. Confluence is read when the source is a spec, and never written by
this pass.

1. **A comment, on approval of the specific content.** `addCommentToJiraIssue` with the long form,
   then read the issue back and report what actually landed — never a predicted result.
2. **The QA4 subtask is offered separately.** Where profile § 13 names a test-case development
   subtask, offer to create or update it, as its own approval. U1 governs its summary; U13 its
   assignee. None named → don't offer one.
3. **Never as an attachment.** DB1's reasoning, the same as UI2's: a file dies with the ticket.
4. **No new label.** U6 limits labels to the profile's vocabulary. Unless `test-matrix`, `qa-matrix`
   or `coverage` is in it, don't invent one.
5. **One approval is one action.** The comment and the subtask are two approvals. Approving the matrix
   in chat is not approval to write it anywhere.
6. **Never edit a ticket you weren't handed**, and never edit AC to close a `⚠️` — propose the AC
   change and let the requester take it through `ticket-review`.

---

## Hand-offs, and honest degradation

QA hand-off skills and a test-management connector are not part of this plugin — the team names any
it has in profile § 13, and no test-management MCP is declared in `.mcp.json`. So none of them can be
depended on — name the ones the profile lists, offer the hand-off, and say plainly when one wasn't
available. Same arrangement as `artifact-design` for `/mock`.

| Wanted | Needs | Absent → |
|--------|-------|----------|
| Steps for the approved rows | a test-case-writing skill (profile § 13) | hand the rows back as they are, and say no cases were drafted |
| Existing coverage in the `Case` column | a case-search skill + a test-management connector | omit the column; `NOT CHECKED` says existing cases weren't searched |
| Questions for the `⚠️` rows | a question-raising skill (profile § 13) | list the findings; don't post a question comment |

An omitted column is honest. An empty column reads as "we looked and found nothing", which is a
different and false claim.

---

## Rules that don't bend

1. **Axes are derived, never invented.** Every one traces to a dependency-matrix rule. An axis the
   ticket is silent about and no rule demands is not an axis — it's a guess.
2. **Two axes crossed, never three**, and only a pair on the sourced list.
3. **Every non-`●` cell carries its reason.**
4. **Inferred is marked `~`**, and every finding resting on an inferred row says so.
5. **Never open an artifact page.**
6. **`NOT CHECKED` is mandatory**, on every matrix. A grid with no scope line reads as full coverage,
   and believing you looked is worse than not having looked.
7. **No test case ever leaves this skill.** Rows carry one-line intent; steps belong to QA or to the
   test-case skill profile § 13 names.
8. **Read, never write, in the repo.** Unchanged from `repo-context.md` rule 7.
