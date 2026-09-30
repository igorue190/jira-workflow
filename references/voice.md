# Voice

Every other reference here is about **completeness** — what a ticket must contain. This one is about
**register**: how the sentences sound to whoever opens the ticket three weeks from now.

A ticket can satisfy every rule in the matrix and still read as though nobody on the team wrote it.
That costs something real. People skim a ticket that reads like boilerplate, argue with one that
reads like a form, and act on one that reads like a colleague who knows the product.

Read this when drafting any ticket body (`ticket-create` Step C, `quick-ticket` Step 2). It is short
on purpose.

---

## What this is not

**Not concealment.** `ai-draft` stays on every ticket this plugin drafts (U6), and a generated
prototype's provenance still travels with its URL. Those are facts about how the ticket was made, and
they belong on the record. What changes here is the prose, not the provenance.

**Not less rigour.** Every rule in the matrix still holds. The isolation acceptance criterion still
exists — it just stops being the same sentence forty times in a row.

---

## 1. The ticket is not the conversation

The single loudest tell, and the only one on this page that is a defect rather than a matter of
style: drafting apparatus that ends up in the Jira description. `RESEARCH`, `Not read`,
`DEPENDENCY CHECKS`, `⚠️`, `[defaulted]`, `(derived — confirm)` — all of it is the draft the
requester approves, none of it is the ticket.

The full boundary, block by block, is in `ticket-schema.md` § *What goes into Jira, and what stays in
the chat*. It is rule **U18(a)** and it is checked on every ticket.

## 2. Register

The generated-text register is recognisable and it is avoidable. Each row below is a habit, not a
banned word — the point is the sentence it produces.

| Habit | Do this instead |
|-------|-----------------|
| "Ensure that the report displays…" | State it: "The report shows…" |
| "leverage", "utilise", "facilitate" | "use", "let" |
| "in order to" | "to" |
| "comprehensive", "robust", "seamless", "streamlined", "holistic" | Cut the adjective. It survives no test |
| "This ticket aims to…", "The purpose of this ticket is to…" | Say what changes. The reader knows it's a ticket |
| "It should be noted that…", "Please note that…" | Cut, and keep the sentence after it |
| "Additionally,", "Furthermore,", "Moreover," | Start the sentence |
| Restating the summary as the description's first line | The summary is directly above it. Open with what the reader doesn't already know |
| Two or three em dashes in one sentence | One clause per sentence, or a full stop |
| Three bullets where the source gave two facts | Two bullets. The third is padding and reads as invented scope |

One more, specific to Story: **don't reach for "As a … I want … so that …" every time.** The schema
allows both, and plain prose is genuinely common on real boards (the notification example in
`examples.md` opens with "Send a final reminder to approvers who have not yet acted…"). Use the user-story
frame when the role actually matters to the requirement; use prose when it doesn't. Reaching for the
template on every ticket is how forty stories end up with identical opening sentences.

## 3. Match the size of the ticket to the size of the work

A data request can be one sentence and still be a complete, closed ticket. A column rename can carry
two requirements. Neither is underwritten (`examples.md` has both; profile § 15 may name real ones
from your boards).

A two-line change does not need six requirements. Padding is the second-loudest tell, and it costs
more than style: a reader who learns that most of a ticket is filler stops reading the part that
isn't — including the isolation check that was the reason to write it down.

The floor is still the floor. The tenant gets named, the invalid state gets named, the isolation and
regression checks stay. What varies with the work is everything you would add *beyond* what the work
needs.

## 4. The isolation and regression AC name what actually breaks

These two appear on nearly every well-written story, and that is correct — `ticket-types.md`
says not to drop them. Written the same way every time, they become a signature that also happens to
be untestable.

| Instead of | Write |
|------------|-------|
| "Change only affects the intended tenant, no cross-tenant leakage" | "Setting a custom value for Acme leaves every other tenant's Custom Labels page unchanged" |
| "Existing behaviour is unaffected" | "The other three dashboard reports still show their own column headers" |
| "No regressions are introduced" | "Order comments still notify the requester when the comment is public" |

Same check, same pass/fail, and now QA knows where to look. A regression criterion that doesn't name
what might regress is a reminder, not a test — and it is the criterion most likely to be ticked
without anyone checking anything.

## 5. Use the words the product and the requester already use

If the screen says **Custom Labels**, the ticket says Custom Labels — not "the label management
interface". If the requester said "the reminder email", don't promote it to "the notification
dispatch mechanism". If the code and the docs call it a Workspace, it is a Workspace everywhere in
the ticket.

This is U16's rule for non-English strings, generalised: a renamed thing cannot be searched for, and
a paraphrase makes the reader do a translation step on every line. It also happens to be most of what
makes a ticket sound like someone who has used the product.

## 6. Don't out-write the board

Real tickets are terse and written under time pressure, often by non-native English speakers.
Reproduction steps like *"Notice, that the private comments are not shown from this comment author"*
are normal and useful.

Uniformly polished prose is itself a tell on a board like that. The fix is **not** to imitate errors —
writing bad English on purpose is mockery and makes the ticket worse. The fix is to drop the
connective tissue: a numbered step is a step, not a sentence with a subordinate clause. Short
declaratives, product nouns, no throat-clearing.

---

## What none of this licenses

- **Vague acceptance criteria.** U9 is unchanged: an AC is a verifiable yes/no statement or it's a
  finding. "Reads naturally" is not a defence for "the report should work properly".
- **Dropping the derived marker silently.** U17 stands. The tag leaves the *description*, and the
  question it carried moves to a Jira comment or gets answered first — it does not evaporate. See
  `ticket-schema.md` § *What goes into Jira, and what stays in the chat*.
- **Dropping the isolation or regression check** because it felt formulaic. Vary the sentence, keep
  the check.
- **Dropping structure.** `REQUIREMENTS` and `ACCEPTANCE CRITERIA` stay as real headings with real
  numbered lists (U8, and the ADF rule in the schema). Human-sounding is about the sentences inside
  them.
