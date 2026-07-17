# Changelog — 2026-07-17

Initial creation of the `improve-code` skill. This entry records the decisions and the alternatives rejected, so a future editor can rebuild the reasoning.

## Goal

A skill that helps an agent improve/refactor a codebase for both agent-navigability and human readability — grounded in software fundamentals and SOLID, with matching docs and tests — and that **applies** the change (diagnose + fix + verify), not just advises. It asks the user for the goal on invocation.

## Decisions

- **Model-invoked (not user-invoked).** The skill carries a trigger-rich `description` so the agent can fire it autonomously and the user can type it. Rationale: code improvement is a broad, frequently-wanted capability; the context-load cost of an always-loaded description is worth the autonomous reach. Rejected: `disable-model-invocation: true` (zero context load) — that would hide a generally useful skill behind memory.

- **Self-contained; no cross-skill dependencies.** The sibling `improve-codebase-architecture` skill references `/codebase-design`, `/grilling`, and `/domain-modeling`, none of which are installed locally. To avoid dead pointers, `improve-code` inlines its own vocabulary and refactoring discipline. For distribution it names no other skill in its instructions — the soft `improve-codebase-architecture` "see also" was removed from both `SKILL.md` and the README so the package stands alone. `sync-skill` remains only in the human README as an optional install convenience.

- **Progressive disclosure into `reference/`.** `SKILL.md` holds only the 8-step spine (needed every run). The principle catalog, refactoring discipline, and doc guidance live in `reference/PRINCIPLES.md`, `reference/SAFE-REFACTOR.md`, and `reference/DOCS.md`, each pointed to from the step that needs it. Rationale: keep the always-loaded file legible (writing-great-skills: progressive disclosure, avoid sprawl). Rejected: one long `SKILL.md` (sprawl, higher per-load token cost); flat sibling files at the root (works, but the dominant convention in this workspace is a `reference/` subdir — 13 skills use `reference/` vs 3 `references/`, and every `super*` skill uses it).

- **Principles as heuristics, gated by net complexity — not laws.** The strongest signal from the research was that dogmatic SOLID / Clean-Code application is the #1 documented failure. So every principle is stated as a smell that triggers a fix, and a `net-complexity gate` drops fixes that add more than they remove. A dedicated "heuristics, not laws" section captures the live debates (tiny functions vs deep methods; comments; DIP over-engineering; define-errors-out-of-existence; agent-friendliness as ordinary good engineering).

- **Leading words for predictability.** deep/shallow module, blast radius, seam, connascence, smell, guard clause, two hats, characterization test, why-comment — repeated across files so the agent anchors the same behavior each run (writing-great-skills: leading words).

- **Checkable completion criteria per step.** Each step ends on a "Done when" gate, several made exhaustive ("every area under a green test", "every occurrence renamed", "diff scoped to the target") to prevent premature completion.

- **Repo layout follows the local ecosystem convention.** `SKILL.md` sits at the repo root and the repo (`improve-code-skill/`) is symlinked as `~/.claude/skills/improve-code`, matching the existing `super*`-skill symlinks in this workspace.

- **Tool-agnostic exploration.** Step 2 names no environment-specific tool. The skill is distributable and must run where the codebase-memory MCP graph (or any particular navigator) is absent, so it directs the agent to use whatever navigation tools the environment offers and choose them itself — degrading from a knowledge graph or language server down to plain search and read. Rejected: hardcoding `search_graph`/`trace_path`/etc., which would silently fail outside this workspace.

## How it was researched

A focused background research workflow ran 5 parallel web-research agents (agent-navigability, design fundamentals, readability/maintainability, docs, tests+safe-refactoring) into one synthesized, cited design brief. Verification was deliberately light: the topics are well-established and the authoritative skill-craft sources (`write-a-skill`, `writing-great-skills`, `refine-skill-terse`) are local, so a full adversarial fact-check pass was not warranted. Rejected: the full 5-angle deep-research *report* workflow — it produces a cited report, not a skill, and the deliverable here is the skill.

## Not done (possible follow-ups)

- No executable validation script yet (e.g. a frontmatter/line-count linter). Add one if the skill is distributed widely.
- Published: `git init` done, public repo `cskwork/improve-code-skill` with a GitHub Pages landing page (`index.html`) and an easy-to-read guide (`guide.html`, linked from the landing hero), and installed for Claude Code + Codex via a single `~/.agents/skills/improve-code` symlink.
