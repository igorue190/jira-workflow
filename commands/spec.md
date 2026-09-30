---
description: A spec was accepted — offer the epic, the story split, the mocks and the traceability table.
argument-hint: [Confluence page URL, epic key, or the topic]
---

Use the `spec-flow` skill — the CHAIN workflow — for: $ARGUMENTS

If no page or epic was given, ask which spec. If the spec doesn't exist yet, say so and hand off to
`ticket-create` to draft it: this command is for what happens *after* a spec is accepted, not for
writing one.

Every item in the chain is approved separately. The menu is not consent.

<!-- This file is named spec.md, not spec-flow.md, on purpose. A command and a skill share one
     namespace: a command called `spec-flow` shadows the `spec-flow` skill, and every hand-off to
     the skill silently degrades to an echo of this file. Don't rename it to match. -->
