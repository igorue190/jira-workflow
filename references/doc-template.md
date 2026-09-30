# Confluence Documentation Template

Every Confluence spec page must follow this section order. No section should be omitted —
if a section is not applicable, include it with "N/A" and a brief reason why.

---

## Page structure

```
# [Feature / Epic Name]

## Overview
[2-4 sentences. What this feature does, why it exists, who it's for.
Link to the parent epic in Jira.]

## Scenarios (Examples)
[Concrete user scenarios that illustrate the feature.
Use numbered scenarios with specific data, not abstract descriptions.]

### Scenario 1: [Name]
**Given** [context]
**When** [action]
**Then** [expected result]

### Scenario 2: [Name]
...

## Requirements
[Detailed functional requirements. Use a numbered list.
Each requirement should be specific enough to write acceptance criteria from.]

1. [Requirement]
2. [Requirement]
...

### Non-functional requirements
- Performance: [if applicable]
- Security: [if applicable]
- Accessibility: [if applicable]

## Traceability
[Which requirement is implemented by which ticket. One row per requirement — including the
ones nothing implements yet, which is the half of this table that earns it.]

| Req # | Story | Status |
|-------|-------|--------|
| 1 | PROJ-211 | In Progress |
| 2 | PROJ-211 | In Progress |
| 3 | — | not yet split out |
| 4 | PROJ-214 | Done |

## Solution Concept
[High-level approach. How will this be built?
Architecture decisions, component overview, data flow.
This section is often co-authored with developers.]

## Solution Design
[Detailed technical design. API contracts, database changes,
component structure. Can link to separate technical docs if extensive.]

## Out of Scope
[What this feature explicitly does NOT cover.
Prevents scope creep and sets clear boundaries for dev and QA.]

- [Excluded item 1 — and why]
- [Excluded item 2 — and why]

## Open Questions
[Unresolved items. Each with an owner and a date raised.
Remove or move to a "Resolved" section once answered.]

| # | Question | Owner | Date Raised | Status |
|---|----------|-------|-------------|--------|
| 1 | [Question] | [Name] | [Date] | Open |

## Change Log
| Date | Author | Change |
|------|--------|--------|
| [Date] | [Name] | Initial draft |
```

---

## Rules

1. **Section order is fixed.** Don't rearrange sections. Consistency across all pages
   makes it possible for devs and QA to jump to the section they need.
2. **Scenarios use Given/When/Then.** Not prose. This makes them directly reusable in QA.
3. **Requirements are numbered.** So they can be referenced in tickets and conversations
   (e.g., "see Requirement 3 in the spec").
4. **Open Questions have owners.** An unowned question is nobody's responsibility.
5. **Link to Jira tickets.** The Overview must link to the epic. Individual requirements
   should reference their implementing stories where possible.
6. **Traceability is the machine-readable half of rule 5**, and it's what `/spec-check` reads to
   know which story came from which requirement. Two things keep it useful:
   - **A requirement nothing implements gets a row too**, with `—` and a reason. That row is the
     whole point: a table listing only what exists can't tell you what's missing.
   - **It is written once per piece of work, not once per ticket.** The `spec-flow` chain fills it in
     when the chain finishes, so five stories cost one page version and one Change Log row.
7. **A page with no Traceability section is normal, not a gap.** Every spec written before this
   section existed lacks one, and nothing retrofits it silently. `/spec-check` infers the mapping
   from the text instead, marks every inferred row `~`, and offers to add the table. Don't report
   its absence as a finding, and don't treat an inferred mapping as a confirmed one.
