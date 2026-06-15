---
name: handoff
description: Session continuity. Orients from or writes project-state.md depending on session phase. Run at the start of a session to pick up context, or at the end to save it.
---

# Handoff

Ensures continuity between agents and sessions. Depending on when it is invoked, handoff either **picks up** (orients from existing state) or **saves** (tidies docs and writes a snapshot for the next agent). One command, two modes, detected from context.

Over time, consistent handoffs make a repository progressively better documented while preventing bloat, staleness, and duplication.

This skill works standalone. Mentions of companion skills (a full documentation audit) are optional — if no such skill is installed, recommend the equivalent work as a manual task instead.

---

## Mode Detection

Determine which mode to operate in:

**Pickup mode** (start of session):
- project-state.md exists in the memory path (or at the repo fallback, `docs/wip/project-state.md`)
- You have no significant work in this conversation yet
- The user just started a session or said something like "pick up where we left off"

**Save mode** (end of session):
- You have done work this session
- The user said "handoff", "wrap up", "save state", or similar

**If ambiguous:** ask the user. One short question: "Are you picking up from a previous session, or saving the current one?"

---

## Pickup Mode

When picking up from an existing handoff:

1. **Read project-state.md** from the memory path. Check `captured_at` against `git log --oneline -8`.

   **Which file (multi-agent):** if you were given a scope key (thread / agent / task id), read the snapshot scoped to you — `project-state.<scope>.md` — falling back to the shared `project-state.md` if you have none. A coordinator picking up after parallel work reads *all* `project-state*.md` files and treats each as scoped to its author (see Phase 4). Single-agent use just reads `project-state.md`.

   **Freshness check:**
   - If the snapshot is older than the most recent commits, treat it as **advisory only**. State has likely changed since it was written. Verify any claims before acting on them.
   - If the snapshot is recent (within the last few commits), trust it as a reliable starting point.

   **Uncommitted changes:** run `git status`. If the working tree is dirty, note it explicitly in the briefing — uncommitted edits may represent work that hasn't been documented yet and are relevant to orientation.

2. **Follow its pointers.** Read the planning/roadmap doc it references. Scan the repo orientation doc if one exists.

3. **Mark it consumed.** Append to the snapshot:
   ```
   last_read: <ISO 8601 timestamp>
   ```
   Only append to a file you own — the shared single-agent file, or your own scoped file. Never mutate another agent's scoped snapshot on read; a read should not write to a file a concurrent agent may be writing. If you need to record that you consumed a sibling's state, note it in your own snapshot instead. (Single-agent: the file is always yours, so this is unconditional.)

4. **Brief the user.** Summarize in 3-5 lines: what's active, what's next, any staleness warnings. Then ask what they want to work on, or proceed if they already said.

---

## Save Mode

### Phase 1: Orient

1. **Locate the memory directory.** Your system context contains the path to the persistent memory system. Use that exact path. Never create a `memory/` folder inside the repo.

   **If no memory path is available in system context:** write project-state.md to `docs/wip/` (creating the folder if needed; repo root only if the repo has no docs/ convention) and note in conversation that a harness-level memory directory would be better. Do not fail silently. A repo-visible snapshot has a side benefit — it travels with the repo and is shared between collaborators — but be mindful that session notes become committed history; keep them professional and secret-free.

2. **Survey what the repo already has.** Look for these *categories* of information, not specific filenames:

   | Category | What it provides |
   |----------|-----------------|
   | Agent orientation | How agents should work here |
   | Navigation / index | Where to find things |
   | Architecture | How the system is structured |
   | Decision records | Why things are the way they are |
   | Planning / next steps | What work is queued |

   Follow whatever structure exists. Note which categories are present and which are absent.

   **If the repo has extensive documentation** (many files, deep directory trees): look for entry points first. A README, an index, a docs hub. Do not read everything. Navigate from entry points inward, following links. If you cannot find a clear entry point into the docs, note that as a navigability issue for Phase 3.

   **If the repo has no documentation at all**: note the absence. Phase 3 will suggest a minimal starting point.

3. **Adopt the repo's own rules.** If the repo defines doc hygiene standards (DRY constraints, maintenance rules, structural laws), apply them. If it defines nothing, default to conservative: fix only what is verifiably wrong; flag the rest.

### Phase 2: Assess

1. Run `git status` and `git log --oneline -8` to understand what changed recently.

2. Read the existing `project-state.md` from the memory path if it exists.

   **If it does not exist:** this is a first handoff. Skip to Phase 3. The write phase will create it from scratch.

   **If it exists:** check `captured_at` against git history. If stale (older than recent commits), note what may have changed between then and now. Do not blindly trust its claims.

3. Find the planning/roadmap file (or equivalent). If none exists, note the gap.

4. Confirm your internal model of the codebase: what it does, what's active, what the docs claim.

### Phase 3: Tidy

This phase scales proportionally with the session's impact. A trivial bugfix session produces a near-empty Phase 3. A session that restructured architecture, renamed modules, or made undocumented decisions warrants a thorough pass. The skill is allowed to run long when the mess is real.

**Detecting what needs attention:**

Compare what happened this session (and what the code currently shows) against what the documentation says. The trigger is **surprise**: if the docs assert something the code contradicts, investigate.

Do not act on your conversational memory alone. Every correction must be verified against the actual code or filesystem.

**Escalation model:**

| Situation | Action |
|-----------|--------|
| Small, verified mismatch (renamed file still referenced by old name, completed item marked pending) | Fix it silently |
| Fixes accumulating beyond a few | Pause, summarize what you've found, ask the user before continuing |
| Major structural divergence (architecture doc doesn't match reality) | Ask the user |
| Cleanup needed is comprehensive and beyond session scope | Suggest a dedicated, full documentation audit as a separate task |

**What to look for:**

- **Code vs. docs** - Do the docs reflect what the code actually does?
- **DRY violations** - Same fact stated in multiple places that could drift apart. Flag it.
- **Navigability** - Fundamental docs that have become unwieldy or hard to follow. For medium repos (~100K LOC or less), navigational docs work best around 300 lines; append-only logs (decisions, changelogs) can grow without limit. Judge by how the doc is *used*.
- **Coverage gaps** - Significant code with no documentation anywhere. Raise specifically, not generically. Before suggesting a new doc, ask: who maintains it? How does it stay accurate?
- **Redundancy risk** - Information that exists in a form likely to drift from its source of truth.

When raising gaps, be specific and realistic. Do not prescribe filenames or rigid structure. Frame suggestions around the *type* of information that's missing and where it might naturally live given the repo's existing conventions.

**First handoff in an undocumented repo:** If the repo has essentially no documentation, suggest a minimal starting structure. Typically this means: (1) a README explaining what the project does and how to run it, and (2) a single navigational doc or section that points to where different types of information live. Frame this as a suggestion, not a mandate. Keep it small enough that the user could realistically maintain it.

### Phase 4: Write project-state.md

Write to the memory path from Phase 1. Overwrite completely. This is a rolling snapshot, not a log.

**Required fields** (every handoff, regardless of session size):

```
captured_at: <ISO 8601 timestamp>
repo_purpose: <one sentence: what is this repository for?>
active_work: <what is currently in progress or was just completed>
pointers:
  - <where to find next steps / planning>
  - <where to find architecture / orientation>
session_notes: |
  <anything the next agent needs that isn't captured in repo docs>
```

**Optional — multi-agent / concurrent handoffs.** Single-agent use ignores this section entirely; behaviour is exactly as above, and nothing else in the skill depends on it.

The hazard in a multi-agent system is *this phase*: "overwrite completely" means two agents saving to the same `project-state.md` is last-writer-wins, and one snapshot is silently lost. Avoid it by **scoping per writer rather than locking** — each agent writes its own file:

```
project-state.md             # single-agent, or the shared / coordinator snapshot
project-state.<scope>.md     # one per concurrent agent; <scope> = thread / agent / task id from harness context
```

Conventions (offer these only when a scope key is actually available):

- **One writer per file.** An agent only ever overwrites the file scoped to itself, so concurrent saves never collide and no lock is required. Derive `<scope>` from a stable id *if your harness exposes one* (thread / agent / task id); slugify it for the filename. If no such id is reachable at runtime, there is no scope key — fall back to the single shared `project-state.md`.
- **Record the scope inside, too.** Add `thread_id: <scope>` to the metadata so a reader can attribute a snapshot without parsing the filename.
- **Atomic publish if available.** If the harness can write-then-rename, write to a temp file and rename into place so a concurrent reader never sees a half-written snapshot.
- **A coordinator rolls up.** An orchestrator (or the next solo agent) reads *all* `project-state*.md`, treats each sibling as advisory and scoped to that agent's work, and may synthesize them into the shared `project-state.md` once the parallel agents have finished. Stale scoped files from agents that died are handled by the normal freshness check — old `captured_at`, treat as advisory.

The `repo_purpose` field anchors the snapshot. It helps future agents (and multi-agent systems) quickly determine whether they're looking at the right context. It should be stable across handoffs unless the project's direction changes.

The `active_work` field captures what was happening at the moment of save. This is the most volatile field and the primary value of the snapshot.

The `session_notes` field is free-form. Include:
- Context from this session that isn't captured in any repo doc
- Anomalies or surprises discovered during this handoff
- Anything non-obvious about the current state

**Exclude from session_notes:**
- Copies of information from canonical docs (DRY violation, will drift)
- Process instructions (this skill file covers process)
- Prescriptive step-by-step playbooks (the next agent is capable; point, don't hand-hold)

### Phase 5: Report

Summarize briefly in conversation:
- What was captured in project-state.md
- Any docs fixed inline
- Any issues flagged for follow-up
- Whether a more comprehensive documentation audit is warranted

Scale the report to the session. A clean session gets one or two sentences.

---

## Principles

**Unified command.** Pickup and save are the same skill. The agent detects which mode applies. If unclear, ask.

**Memory path.** Always from system context. If unavailable, fall back to `docs/wip/project-state.md` with a note to the user. Never fail silently.

**Rolling snapshot.** Always overwrite (in multi-agent use, overwrite *your own* file — never a sibling's). Git provides history for repo files; the memory system may not have version history, but staleness is handled by the freshness check.

**Freshness over trust.** A snapshot's value decays with time and commits. Old snapshots are advisory. Recent snapshots are reliable. The `captured_at` timestamp is the signal.

**Repo-agnostic.** Works in any repository. Follows existing structure. Degrades gracefully when structure is minimal. Suggests improvements without prescribing.

**Manual trigger.** The user runs `/handoff` deliberately. No auto-invocation.

**Re-entrancy.** Every action taken during handoff should leave the repo easier to hand off *next* time. Avoid creating artifacts with no realistic maintenance path.

**DRY.** project-state.md points; it does not duplicate. Canonical docs hold canonical information.

**Entry points and navigation.** Well-documented repos have simple entry points (README, index, agent orientation) that link to where information is canonically stored. Each doc covers one concern; cross-references replace duplication. When this pattern is in place, handoff is fast and reliable. When it is absent or broken (docs that try to do too much, circular references, no clear starting point), handoff should note the issue and suggest improvements toward this pattern.

**Multi-agent awareness.** The structured fields in project-state.md are designed to be parseable by other agents or orchestration systems. The skill does not assume it is the only agent operating, but it also does not require multi-agent infrastructure. It writes a clean, self-contained artifact that any agent can read. When agents run in parallel, each writes its own scoped file (see Phase 4) — one writer per file, so concurrent handoffs never collide and no locking is required. Reads stay non-mutating across scopes (Pickup step 3). The mechanism is opt-in: with no scope key, the skill writes a single self-contained `project-state.md` that any agent can read.
