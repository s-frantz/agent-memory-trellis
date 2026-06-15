<p align="center">
  <img src="assets/trellis.png" width="150" alt="A green vine growing up a wooden trellis">
</p>

<h1 align="center">agent-memory-trellis</h1>

<p align="center">
  The fundamental skill suite for agentic engineering with persisted memory.
</p>

<p align="center">
  <a href="LICENSE"><img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License"></a>
  <img src="https://img.shields.io/badge/Claude%20Code-plugin-d97757.svg" alt="Claude Code plugin">
</p>

---
Play to your agents' strengths, and your own, by growing your project along clean documentation lattices. Give new agents only new instructions, and see them pick up instantly to make useful contributions.

The skills maintained here bear the brunt of the repo-hygiene / anti-entropy / anti-bloat work needed to support this. They are — in this spirit — deliberately minimal, project-agnostic, and respectful of existing conventions.  Feel free to use them as-is, or build them out to your preferred specs.

Contributions are welcome — see [guidelines](CONTRIBUTING.md).


## The skills

| Command | Responsible for | Invoke when |
|---------|------|-------------|
| `doc-library` | **Structure** — what docs exist and where they live | Creating docs for the first time, or seeking guidance on doc responsibilities |
| `grill-me` | **Planning** — make details explicit, and interrogate human <> agent gaps | Planning new features (an extension of the famous Matt Pocock skill, with nuanced doc guidance) |
| `handoff` | **Continuity** — session state, saved and picked up | Ending a working session; or starting the next one |
| `doc-audit` | **Verification** — do the docs match the code | Major code changes or architecture planning have occurred, or if docs feel stale |
| `ubiquitous-language` | **Vocabulary** — a canonical domain glossary | Shorthanding complex ideas, or resolving ambiguous terminology |

Multi-agent runs are supported out of the box: under `handoff`, parallel agents each write their own scoped snapshot — without locks — and a coordinator rolls them up. Single-agent use ignores this entirely.


## Rationale

Our skills address LLM tendencies that are likely to persist through jumps in intelligence and context capacity. Our fundamental assumptions:

1. **LLMs are inferencing machines.** Modern LLMs, built on the transformer model, are fundamentally better at pattern-matching and distilling information, than they are at foreseeing the long-term consequences of their own actions. Set loose on "vibes" alone, they are liable to losing the plot to an inundation of their own making.
2. **Context should be concise.** We experience this as a context-size tipping point, beyond which model performance degrades. They begin to struggle with prioritization, or need frequent reminders of earlier details or guidelines. Just like you and me, models need some free overhead space to mull ideas and grow into.
3. **Context should be maximized.** Nonetheless, agents do their best work if they have maximal access to *relevant* context for the specific problem they are actively working on.
4. **We should prioritize clean reentrancy.** To support this, it helps to give agents headroom for new tasks, leaving a clear path for retrieving old, relevant details and resolving inconsistencies. In the limit, the clarity / navigability of this path can be measured by new agents seeing the repo for the first time.
5. **We should use progressive disclosure.** To avoid inundating context right at our entry points, we should aim to store only essential information and indices, pointing to progressively more complete artifacts of existing code, docs, discussion etc.
6. **We should aim to self-right.** Finally, we must accept that new work will produce new noise and entropy, and even agents tasked with cleanup will miss things. As long as our systems are self-righting — front-and-center docs nudge agents back towards order — our projects can stay upright even through rapid growth.


## Information privacy

For public repos, a gitignored `docs/private/` tier may be kept with its own INDEX; the public INDEX points to it in one line, and public docs reference it in plain text so they stand alone in any clone. `doc-library` defines the possible split; `doc-audit` polices it to prevent leaks and broken boundary links.


## Install

Two paths — depending on your goals:

### Consume (stay current)

Install as a Claude Code plugin and get auto-updatable, namespaced commands (`/trellis:*`):

```
/plugin marketplace add s-frantz/agent-memory-trellis
/plugin install trellis@agent-memory-trellis
```

Commands live outside your repo, in Claude Code's plugin cache; unless you add them to your project's `.claude/settings.json`.

### Own (customize)

Copy the command files into your project — bare `/command` invocation, helpful if you want to customize the skills:

```bash
# Claude Code, per-project
cp commands/*.md {your-repo}/.claude/commands/

# Claude Code, global (all projects)
cp commands/*.md ~/.claude/commands/
```

Other agent harnesses that accept markdown instructions can use `commands/*.md` as-is; only the `/slash-command` invocation is Claude Code-specific.


## License

[MIT](LICENSE)
