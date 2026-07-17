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

---

# Enhancement — visual as-is → to-be at Step 4

Step 4 now *shows* the plan instead of only listing it: it renders a self-contained **as-is → to-be** HTML (current vs proposed structure, blast radius before and after, one card per fix), opens it in the browser, and narrates it before the user picks. Added `reference/VISUAL-DIFF.md` (procedure) and `reference/as-is-to-be.template.html` (offline fallback + worked example). Touched `SKILL.md` (Step 4 body + Done-when, on-demand disclosure list) and `README.md` (What-it-does step 4, file map, Sources).

## Decisions

- **Insert at Step 4, not a new step.** Step 4 (Agree the plan) is the user's decision point, so the visual serves that decision directly. Rejected a new standalone step (renumbers 5–8 and ripples every "Step N" cross-reference) and rejected Step 3 (diagnosis is internal analysis; the user is not choosing yet).

- **The archify approach, brought in tool-agnostically — not a hard dependency.** The as-is/to-be *twin diagram as self-contained HTML* is the archify technique. `improve-code` adopts the approach and prefers an archify-style toolchain when the environment provides one, else falls back to a bundled offline template. Rejected hard-wiring the `supergoal` skill's vendored `templates/archify/` path: it would break this skill's standing "self-contained; names no other skill in its instructions" decision (recorded above) and would fail everywhere the toolchain is absent. `supergoal` / archify is named only in this changelog and the README Sources line — never in `SKILL.md` or `reference/*` instructions — consistent with the tool-agnostic Step 2.

- **One template file that is also the worked example.** `as-is-to-be.template.html` ships filled with a realistic order-pricing refactor (shallow → deep module, primitive obsession → `Money` value object, global singleton → injected `TaxPolicy` seam) and marked with `<!-- SLOT -->` replace points. Rejected generating HTML from scratch each run (variance, tokens, drift) and rejected a blank template plus a separate example — one openable file serves both, matching archify's `examples/` philosophy.

- **Offline by construction.** No CDN or network: inline CSS, inline SVG node/edge drawing, a small inline theme-toggle script, dark/light using the project's own landing-page color tokens. Opens on a double-click and survives with the repo — mirroring archify's no-network guarantee.

- **Draw-by-default, prose fallback.** The visual is the default at Step 4 but is skipped for a trivial one-symbol rename or when the user declines — a diagram that would cost more than the fix fails the same net-complexity gate the skill applies to code.

## Verified

- archify toolchain `doctor` passes in this workspace (confirms the preferred render path is real; Node ≥18).
- Template carries zero external URLs (offline confirmed) with 8 fill slots.
- All pointers resolve: `SKILL.md` → `VISUAL-DIFF.md` (×2) → `as-is-to-be.template.html`; README file-map links intact.
- Opened `as-is-to-be.template.html` in the browser — renders the two-panel as-is → to-be with the blast-radius meter, legend, and fix cards; theme toggle works.
