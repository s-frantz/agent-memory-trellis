---
name: doc-library
description: Recommends and scaffolds a standard documentation library — README and AGENTS at the root, everything else under docs/. Owns documentation structure; complements handoff (continuity) and doc-audit (verification).
disable-model-invocation: true
---

# Doc Library

Proposes — and on approval, builds — a standard document library for a project. The premise: **text instructions are the new code.** An agent that simultaneously maintains a well-shaped docs folder works faster, needs less supervision, and makes re-entrant passes (new sessions, new agents, post-compaction recovery) far more reliable.

This skill is a *suggestion to migrate*, not a mandate. It never edits or replaces its companion skills — the kit divides cleanly:

| Skill | Owns |
|-------|------|
| `/doc-library` | **Structure** — what docs exist and where they live |
| `/handoff` | **Continuity** — session state, picked up and saved |
| `/doc-audit` | **Verification** — do the docs match the code |
| `/grill-me` | **Planning input** — interrogate a plan to resolution; resolved plans land in `docs/wip/` |
| `/ubiquitous-language` | **Vocabulary** — a canonical domain glossary (the `UBIQUITOUS_LANGUAGE.md` doc below) |

**Standalone use:** each skill in this kit works alone. References to the others are pointers to optional companions — if a companion is not installed, skip those mentions; nothing here depends on them existing.

Run when:
- A project (or subfolder destined to become its own repo) has docs scattered at the root or none at all
- A repo split is planned and the docs need to travel cleanly
- Documentation exists but each new doc has required a naming/placement decision

---

## The Library

The target shape. Root contains only the two files every visitor and every agent reads first; everything else lives in `docs/`.

```
README.md            # root
AGENTS.md            # root
docs/
  INDEX.md
  ARCHITECTURE.md
  DECISIONS.md
  ROADMAP.md
  PITFALLS.md
  TESTING.md
  <supplementary docs as needed, all listed in INDEX.md>
```

| Doc | Purpose | Growth policy |
|-----|---------|---------------|
| `README.md` | Project purpose and setup, ~200 lines max. The front door. | Navigational — keep tight, condense aggressively |
| `AGENTS.md` | High-level guidelines the model always uses: development flow, coding style, when to run/update tests, doc-maintenance duties | Navigational — keep tight |
| `docs/INDEX.md` | Table of contents for navigating all docs | Navigational — one line per doc |
| `docs/ARCHITECTURE.md` | How things actually fit together: components, data flow, orchestration. Includes a **FEATURES table of contents** linking feature-specific .md files embedded throughout the project tree | Navigational — condense as the system stabilizes |
| `docs/DECISIONS.md` | Long doc tracking past work outcomes and major decisions, with dates and rationale | Append-only — length is fine |
| `docs/ROADMAP.md` | Planned future work, roughly sequenced | Living — prune what ships or dies |
| `docs/PITFALLS.md` | Two-directional trap log: pitfalls the *model* should avoid repeating, AND traps the model has noticed the *human prompter* falling into | Append-only — length is fine |
| `docs/TESTING.md` | How the model should validate its work; grows alongside the test suite as features are developed | Living |
| `docs/wip/` | Ephemeral working docs — see below | Expiring — every file graduates or dies |

Optional library docs, created when the need bites rather than up front:

| Doc | Create when |
|-----|-------------|
| `docs/UBIQUITOUS_LANGUAGE.md` (or `GLOSSARY.md`) | Domain terms accumulate or ambiguity causes a real mistake. Lives in `docs/`, never at root; AGENTS.md points to it as the canonical vocabulary |

Supplementary docs (a COMMANDS.md, feature specs, API references) are welcome — the blueprint is a floor, not a ceiling. Every supplementary doc must appear in INDEX.md, and feature-specific docs embedded in the source tree must appear in ARCHITECTURE.md's FEATURES table.

### Ephemeral docs: `docs/wip/`

Plans, design drafts, grilling outcomes, migration checklists — documents that are *load-bearing for days, garbage in a month*. They get a home so they stop polluting the canonical docs, with three rules:

1. **Every wip doc opens with a header:** created date, purpose, and its graduation criteria — what must happen for it to be promoted or deleted.
2. **Graduate or die.** Outcomes promote into DECISIONS.md (what was decided), ROADMAP.md (what remains), or ARCHITECTURE.md (what changed) — then the wip file is deleted. A wip doc is never linked from canonical docs as if it were one.
3. **Staleness is visible.** Audits and handoffs should flag wip docs past their use-by (roughly a month) rather than treat them as canon. Length/bloat rules don't apply here; expiry rules do.

One exception: **rolling snapshots** (e.g. a session-continuity file like `project-state.md`, when no harness-level memory directory exists) live in `docs/wip/` but are exempt from graduation — they are overwritten, not promoted, and their freshness is judged by their own timestamp.

### The public/private split (optional)

For repos meant for publication, the library supports a second, **gitignored** tier: `docs/private/`. Persistent docs that must not publish — strategy, roadmaps that name unreleased integrations, trap logs that reveal personal workflow — live there, with their own `INDEX.md`. Conventions:

- **The public INDEX carries one line** pointing at the private tier: "maintainer docs live in `docs/private/` (gitignored) — start at its INDEX.md if present."
- **Public docs must stand alone in a stranger's clone.** Refer to private docs in plain text ("the maintainer's decision log"), never as markdown links that would 404 for everyone but the maintainer.
- **Typical tier assignment when a repo goes public:** DECISIONS, ROADMAP, and PITFALLS usually go private; README, AGENTS, INDEX, ARCHITECTURE, TESTING, and operational references stay public. Decide per doc in Phase 2 — for publication-bound repos, the migration table gains a *tier* column.
- `docs/wip/` is normally gitignored too in public repos — drafts and grilling outcomes carry strategy.
- Trade-off to state when proposing: gitignored docs have no git history and don't travel with clones. Back them up accordingly.

**Docs outside the repo entirely:** some teams keep the private tier elsewhere (a second repo, a notes system) — managing that store is beyond these skills, but the same pattern applies: one pointer line in the public INDEX, and adjust the `docs/private/` paths wherever the skills mention them.

---

## Phase 1: Assess

1. Inventory every existing doc in the target scope (root-level .md files, any existing docs/ folder, feature docs embedded in the tree).
2. Map each onto the blueprint: which target doc does its content belong to? Some docs map 1:1, some dissolve into several targets, some are already supplementary docs that just need indexing.
3. Note content that exists nowhere but should — an empty DECISIONS.md is a missed opportunity if the conversation or git history holds reconstructable decisions.

## Phase 2: Propose

Present a migration table before touching anything:

| Existing | Disposition |
|----------|-------------|
| `SPEC.md` | dissolve → ARCHITECTURE.md (diagrams, schemas) + DECISIONS.md (rationale), then delete |
| `NOTES.md` | rename/merge → PITFALLS.md |
| ... | ... |

For publication-bound repos, add a *tier* column (public / private) — see "The public/private split" below.

Ask before deleting or dissolving anything significant. Creating new files and adding pointers needs no confirmation when the skill was explicitly invoked.

## Phase 3: Migrate and scaffold

- **Dissolve, don't duplicate.** When an old doc's content moves, the old doc is deleted and inbound links updated — never left as a stale copy.
- **One fact, one home.** Decisions live in DECISIONS.md; ARCHITECTURE.md describes the result and links to the decision. README links rather than restates.
- **Seed, don't stub.** Each created doc gets real content from the repo, git history, and current conversation — not placeholder headings. If a doc would genuinely be empty, create it with one line stating what belongs there and when to first write it.
- **Capture the present.** If the invoking conversation contains plans, research findings, or decisions not yet written down, landing them in ROADMAP.md / DECISIONS.md / PITFALLS.md is in scope — that is often the whole reason this skill was run.

## Phase 4: Bake maintenance in

Structure without a maintenance loop rots. The companion skills enforce hygiene *when invoked* — but most entropy arrives during ordinary coding sessions in between. So the rules must live in the repo's own entry point, where every session reads them. AGENTS.md must include (in the repo's own voice):

**The standing instruction:**

> Always revisit and update the docs at the end of each major code contribution. Condense docs and code where possible. Remove or archive what's out of date to battle bloat — especially in INDEX, README, and ARCHITECTURE.

**The anti-entropy laws** — adapt wording to the repo, keep the substance:

1. **One home per thing.** Every file type and every fact has exactly one home. If something doesn't fit, consult the entry-point doc before inventing a new location.
2. **Docs move with code, in the same change.** If a change makes a doc wrong, fixing the doc is part of the change — not a follow-up.
3. **DRY.** One fact, one authoritative home; everywhere else links to it. Duplication is the primary source of doc rot.
4. **Progressive disclosure.** Entry points stay short and link outward; detail lives one level deeper. A reader (human or agent) reaches any fact in ~two hops from the README.
5. **Navigational docs stay navigational.** README/INDEX/ARCHITECTURE/AGENTS hold ~200–300 lines by moving detail outward, never by omitting it. Append-only logs absorb history so they don't have to.
6. **Archive, don't accumulate.** Superseded material moves to an archive or dies; the archive is a record, not a library. Entropy is the enemy.

Also point AGENTS.md at the division of labor: `/handoff` for session continuity, `/doc-audit` for periodic deep verification.

## Phase 5: Report

- Migration table as executed (created / dissolved / merged / deleted)
- What was seeded from conversation context vs. reconstructed from the repo
- Anything deferred or left for the user to decide

---

## Principles

**Suggestion, not framework.** Adapt names and granularity to the repo's idiom. A tiny project may merge TESTING into AGENTS; a large one may split ARCHITECTURE. The invariants are: tight navigational entry points at the root, one home per fact, append-only logs for history, and an explicit index.

**Made for re-entrancy.** The measure of success: a fresh agent with zero session context can read README → AGENTS → INDEX and reach any fact in the repo within two hops.

**Plays well with siblings.** Never modify `/handoff` or `/doc-audit`. After a migration, recommend running `/doc-audit` once the dust settles to verify the new structure against the code.

**Battle bloat at the source.** Navigational docs (README, INDEX, ARCHITECTURE, AGENTS) are kept short by moving detail outward, not by omitting it. Append-only logs (DECISIONS, PITFALLS) absorb history so the navigational docs never have to.