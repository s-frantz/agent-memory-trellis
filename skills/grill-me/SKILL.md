---
name: grill-me
description: Interviews you relentlessly about a plan or design until you reach shared understanding, resolving each branch of the decision tree. For each question it offers its recommended answer.
disable-model-invocation: true
---

Interview me relentlessly about every aspect of this plan until we reach a shared understanding. Walk down each branch of the design tree, resolving dependencies between decisions one-by-one. For each question, provide your recommended answer.

Ask the questions one at a time.

If a question can be answered by exploring the codebase, explore the codebase instead.

Stop when the branches are resolved — when further questions would re-litigate settled decisions rather than uncover new ones. Say so explicitly.

## Outputs

A grilling produces decisions, and decisions want to be written down — but this skill never silently edits canonical docs mid-interview. When the grilling concludes:

1. Offer to write the resolved plan as an ephemeral working doc (e.g. `docs/wip/PLAN-<topic>.md` if the repo keeps a wip space, otherwise wherever drafts live), with date, purpose, and what would graduate it.
2. Propose — don't apply — any updates the resolved plan implies for roadmap or decision docs. I, the user, must accept or decline each.

Be aware that invoking this skill can therefore result in new temporary files and doc updates. This is intended, but it happens with consent, at the end, never as a silent side effect.
