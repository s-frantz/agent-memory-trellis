---
name: doc-audit
description: Full documentation audit — reads every authoritative doc, checks it against the code, and resolves the inconsistencies found. A comprehensive pass, not a lightweight session check.
disable-model-invocation: true
---

# Doc Audit

A comprehensive pass over all repository documentation. This skill reads everything, verifies everything, and resolves what it finds — a full audit, not a lightweight session-scoped check.

This skill works standalone. If a session-state snapshot (such as a `project-state.md`) happens to exist in the environment, treat it as optional context — proceed without it; its absence is not an error.

Run when:
- Docs have visibly fallen behind an active development period
- A major refactor, rename, or architectural change just landed
- A new agent suspects the docs are significantly behind the code
- The repo feels hard to navigate and the reason isn't obvious

---

## Phase 1: Orient

1. **Locate the memory directory** from system context. Write a findings summary there when done.

   **If no memory path is available in system context:** write findings to `docs/wip/` (e.g., `docs/wip/doc-audit-findings.md`; repo root only if the repo has no docs/ convention) and note in conversation that a persistent memory directory would be better. Do not fail silently.

2. **Read project-state.md** if it exists. Check its `captured_at` field against recent git history. If stale (older than recent commits), note that the project may have diverged from what that snapshot last recorded. This is useful context for understanding whether docs have been maintained. (In a multi-agent repo the snapshot may be split per writer as `project-state*.md` — see **Multi-agent scope** for which to read.)

3. **Map what the repo provides.** Look for categories of information, not specific filenames:

   | Category | Purpose |
   |----------|---------|
   | Agent orientation | How agents should work here |
   | Navigation / index | Where to find things |
   | Architecture | How the system is structured |
   | Decision records | Why things are the way they are |
   | Planning / next steps | What work is queued |
   | Authoritative locations | Which doc owns which facts |

4. **Read the repo's own hygiene rules** if they exist (DRY constraints, maintenance rules, structural laws, doc ownership tables). If none exist, apply the general heuristics in this skill.

5. **Assess navigability.** Can you find a clear entry point into the documentation? Does it link outward to canonical locations? Or is information scattered, duplicated, or buried? Note your experience navigating the docs. If you struggled to find something, that is itself a finding.

---

## Phase 2: Read all authoritative docs

Read the full content of every doc that claims to be authoritative. Start from whatever index or hub exists; if none, enumerate the docs directory.

**If docs are numerous:** prioritize by change risk — read API contract docs, status docs, and architecture docs first, since they go stale fastest. Feature docs and infrastructure docs can follow. If findings accumulate past ~10 before you've finished reading, it's fine to stop reading and report what you've found; note explicitly which docs were not reached.

For each doc, note:
- What it claims to describe
- Staleness signals (references to files that don't exist, old API versions, architecture that doesn't match reality)
- Whether it reads as a living reference or a point-in-time snapshot

Build a complete picture before fixing anything.

---

## Phase 3: Verify against code

For each authoritative doc, check its claims against the actual codebase. This is the core of the audit.

**Structural accuracy**
- Does the described repo structure match the filesystem?
- Are there significant directories or files the docs don't mention?

**API / endpoint accuracy**
- Do documented routes match the actual route definitions?
- Endpoints added, removed, or renamed without doc updates?

**Architecture accuracy**
- Does the architecture doc reflect actual data flow, services, and dependencies?
- Any major components missing?

**Schema accuracy**
- Do field descriptions and examples match current models?
- Fields added or removed without doc updates?

**Status accuracy**
- Items described as planned that are actually complete?
- Items described as complete that are broken or removed?
- Future features written about as if current?

**Decision-record accuracy** (if the repo keeps a `DECISIONS.md` or `docs/decisions/` log)
- This skill owns the decision-log gate: the live log must hold only **in-force** decisions the code actually reflects.
- An `accepted` record with no corresponding code is a **phantom** — a planned-but-unbuilt decision that reads as current state and misleads the next agent. Verify against code; if the work landed, this is just stale status to fix; if it never landed, flag for retirement (or demote to a `docs/wip/` plan).
- A record marked `proposed` that the code now reflects should be **promoted** to `accepted`.
- A significant decision visible in the code but absent from the log is a coverage gap — suggest a record.

---

## Phase 4: Assess systemic health

Beyond individual facts, look for structural problems in the documentation as a whole.

**DRY violations**
- Same fact in more than one doc. Pick one authoritative home; replace copies with pointers.

**Broken cross-references**
- Links to renamed or deleted files
- Index entries pointing to docs that don't exist (or docs that exist but aren't indexed)

**Orphaned docs**
- Docs describing something that no longer exists in the codebase
- Old design specs or migration guides for replaced systems
- Confirm with user before deleting anything significant

**Navigability**
- Navigational docs (README, index, architecture) that have become dense or hard to follow. In a medium repo (~100K LOC or less), these work best around 300 lines. Append-only logs (decisions, changelogs) can grow without limit.
- Judge by how the doc is *used*, not its line count. A well-structured 600-line doc is fine. A confusing 200-line doc is not.

**Coverage gaps**
- Significant code with no documentation anywhere
- Entire categories of information absent from the repo
- For each gap: suggest specifically, and consider whether the suggested doc has a realistic maintenance path. If not, say so.

**Re-entrancy concerns**
- Docs that can only stay accurate through manual vigilance with no clear trigger
- Point-in-time snapshots that aren't marked as such
- Flag: "This reads as a snapshot, not a living reference. Consider converting to a pointer or adding a maintenance note."

**Publication boundary** (repos with a public/private docs split — gitignored `docs/private/` or `docs/wip/`)
- Establish the boundary first: `git check-ignore` (or read the .gitignore) tells you which docs publish and which don't. Audit both tiers, but hold them to different rules.
- **Leak check:** public docs must contain no private-tier material — strategy, unreleased product/integration names, personal info, or machine-specific absolute paths. Treat a leak as a fix-immediately finding.
- **Standalone check:** public docs must work in a stranger's clone where the private tier is absent — references to private docs are plain-text mentions, never markdown links (they'd 404 for everyone but the maintainer).
- **Pointer check:** the public index should carry the one-line pointer to the private tier ("if present"), and the private tier should have its own index when it holds more than a file or two.

---

## Phase 5: Fix or escalate

Broader license here than during a routine session check, since this skill was explicitly invoked:

| Situation | Action |
|-----------|--------|
| Verified, unambiguous mismatch (stale status, broken link, renamed file) | Fix it |
| DRY violation with clear resolution (copy → pointer) | Fix it |
| Structural rewrite, doc splitting, or deletion | Ask the user with a specific recommendation |
| Many fixes accumulating across many docs | Pause, summarize findings, confirm before proceeding |

When asking, be specific: "ARCHITECTURE.md describes X. The code shows Y. Should I update the architecture section to reflect Y?"

**External-state discrepancies:** some mismatches can only be verified against external systems (a third-party dashboard, a live service, a Stripe config). Do not guess — flag these explicitly, state what needs to be verified manually, and move on. Fixing the wrong side of an externally-dependent discrepancy can introduce a real bug.

---

## Phase 6: Report

Summarize what was found and done:

- Every doc checked — if the audit was partial, list which docs were not reached and why
- For each doc checked: what was fixed, flagged, or left alone (and why)
- Systemic issues found (DRY violations, coverage gaps, re-entrancy concerns)
- Recommended follow-up, if any

A clean audit is useful information. Say so clearly if nothing needs fixing. A partial audit is also useful — be explicit about coverage so the next agent knows where to continue.

---

## Principles

**Comprehensive by design.** This skill reads and verifies everything. Token cost is acceptable.

**Verify before acting.** Every fix must be confirmed against the actual code or filesystem.

**Respect existing structure.** Follow the repo's own conventions. Suggest improvements within its idiom, not by imposing a new framework.

**Re-entrancy.** Every action should leave the repo easier to audit next time. Avoid creating maintenance burden.

**DRY.** Each fact lives in one place. Other docs point to it. Redundancy is the primary source of doc rot.

**Entry points matter.** A well-documented repo has simple entry points that link to canonical locations. Each doc covers one concern. Cross-references replace duplication. When auditing, evaluate whether this pattern holds. When suggesting improvements, guide toward it.

**Session context.** If project-state.md exists in the memory path, read it for context on what work has been active and what the last agent found surprising. If it is stale or missing, note it — the repo is operating without cross-session continuity, which is itself a finding.

**Multi-agent scope.** If multiple agents maintain this repo concurrently, an audit can mistake another agent's in-flight work for inconsistency. When a scope key like `thread_id` is in use, scope the audit to the current thread rather than flagging cross-thread divergence as doc rot. Session snapshots may follow a per-writer convention (`project-state.<scope>.md`, one file per agent): read the one scoped to you, or read all `project-state*.md` only when you are deliberately auditing across threads. A single `project-state.md` (the common case) is just read directly.

## Heuristics reference

These are the heuristics baked into this skill. Apply judgment — they are guides, not rules.

| Signal | What it means |
|---|---|
| Same fact in 2+ docs | DRY violation — pick one home, replace copies with pointers |
| Doc mentions file/endpoint that doesn't exist | Stale — verify and fix |
| "Planned" item that's clearly shipped | Status stale — update |
| `accepted` decision record with no matching code | Phantom — fix status if it shipped, else flag for retirement/demote to wip |
| `proposed` decision record the code now reflects | Promote to `accepted` |
| Navigational doc >~300 lines and hard to follow | Suggest splitting or linking out |
| Append-only log (decisions, pitfalls, changelog) | Length is fine — don't flag |
| Ephemeral/working doc (e.g. in a `docs/wip/` space) | Exempt from bloat rules, but flag if clearly expired (~a month old, or its stated graduation criteria already happened) |
| Public doc contains private-tier material (strategy, personal paths, unreleased names) | Leak — fix immediately, move content behind the boundary |
| Public doc markdown-links into a gitignored path | Breaks in a stranger's clone — convert to a plain-text mention |
| Doc with no clear update trigger | Re-entrancy concern — flag |
| Significant code with zero doc coverage | Missing info type — raise with specific suggestion |
| Doc references architecture that doesn't match code | Verify against code, then fix or escalate |