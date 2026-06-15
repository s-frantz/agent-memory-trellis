# Contributing to agent-memory-trellis

Thanks for helping keep this suite sharp. The whole point of these commands is to fight entropy — so the bar for changes is *does this keep the suite minimal, standalone, and honest?* Smaller and clearer beats bigger and cleverer.


## What a good command looks like

Each command is a single, self-contained `commands/<name>.md` with `name` and `description` frontmatter. Hold each one to these invariants:

- **Standalone.** It works alone. References to sibling commands are *optional pointers* — if a companion isn't installed, the mention is skipped and nothing breaks.
- **Tandem-safe.** It composes with the others without overlapping their job. The division of labor (structure / planning / continuity / verification / vocabulary) stays clean; one command never edits another's domain.
- **Deferential.** On its own it leads to a specific markdown architecture, but it defers to repo- or user-defined conventions whenever those exist.
- **Manual.** Commands are invoked deliberately by a human. None should assume auto-invocation.


## The anti-entropy laws

These are the same laws `doc-library` plants in a host repo's `AGENTS.md`. This repo holds itself to them:

1. **One home per thing.** Every fact has exactly one authoritative home; everywhere else links to it.
2. **Docs move with the change.** If a change makes a doc wrong, fixing the doc is part of that change — not a follow-up.
3. **DRY.** Duplication is the primary source of rot. In particular: the `README` command table and `.claude-plugin/plugin.json` must stay in sync with `commands/`.
4. **Progressive disclosure.** Entry points stay short and link outward; detail lives one level deeper.
5. **Navigational docs stay navigational.** Keep the README tight by moving detail outward, not by omitting it.
6. **Archive, don't accumulate.** Superseded material dies or moves to history; entropy is the enemy.


## Making a change

1. Open an issue or a draft PR describing the change and *why* — especially if it touches a command's scope or the division of labor.
2. Keep diffs surgical: every changed line should trace to the stated goal. Don't refactor adjacent prose that isn't broken.
3. If you add or rename a command, update **both** the README command table and `.claude-plugin/plugin.json` in the same change.

Please be kind in reviews and issues.
