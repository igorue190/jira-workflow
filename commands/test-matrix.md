---
description: Build the test matrix for a ticket or spec — which combinations need testing, and which nothing covers.
argument-hint: [ticket key, or a Confluence spec URL]
---

Use the `test-scope` skill for: $ARGUMENTS

If nothing was given, ask which ticket or spec to build the matrix for. Don't guess from recent
context.

<!-- This file is named test-matrix.md, not test-scope.md, on purpose. A command and a skill share
     one namespace: a command called `test-scope` would shadow the `test-scope` skill, and every
     hand-off to the skill would silently degrade to an echo of this file. Don't rename it. -->
