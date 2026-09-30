---
description: Review a Jira ticket against the team schema and dependency matrix.
argument-hint: [ticket key, e.g. PROJ-123]
---

Use the `ticket-review` skill to review: $ARGUMENTS

If no ticket key was given, ask which ticket (or epic, for a batch audit) to review.

<!-- This file is named review-ticket.md, not ticket-review.md, on purpose. A command and a skill
     share one namespace: a command called `ticket-review` shadows the `ticket-review` skill, and
     every hand-off to the skill silently degrades to an echo of this file. Don't rename it back. -->
