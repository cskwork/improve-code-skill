# improve-code

A Claude Code skill that improves and refactors a codebase so that **one correct change needs less context** — for an AI agent and a human alike. It is goal-driven (it asks what you want improved), grounded in software fundamentals, and it *applies* the change under green tests rather than just advising.

## What it does

Given a target you point it at, the skill:

1. **Sets the goal** with you — target, payoff, constraints, definition of done.
2. **Explores and pins ground truth** — maps the code, confirms the tests are green.
3. **Diagnoses** against a smell catalog (shallow modules, hidden coupling, strong connascence, primitive obsession, inconsistent naming, stale docs …).
4. **Agrees a plan — shown as as-is → to-be** — renders a self-contained visual (current vs proposed structure, blast radius before and after, a card per fix), opens it in the browser, and walks you through it before you pick.
5. **Pins behavior with tests** — characterization tests before touching untested code.
6. **Applies surgically** — one behavior-preserving move at a time, tests green after each.
7. **Makes it navigable** — names, docstrings, docs, structure, one source of truth.
8. **Verifies and records** — runs the real checks, writes a changelog entry.

The guiding ideas are **deep modules** (small interface over rich implementation), small **blast radius** (a change stays in one place), and explicit **seams** (where tests attach and internals get swapped). Principles are applied as heuristics triggered by a **smell** and gated by **net complexity** — never as top-down law.

## When to use it

Ask for it when you want to refactor, clean up, simplify, or restructure code; improve readability, maintainability, or module and interface design; reduce coupling; make a codebase easier for an agent or a teammate to navigate; or add the tests and docs a reshape needs. It is model-invoked, so the agent can also reach for it on its own.

## File map

| File | Holds |
|------|-------|
| `SKILL.md` | The 8-step procedure (loaded every run) |
| `reference/PRINCIPLES.md` | Vocabulary, principle set, smell catalog, agent-navigability checklist, heuristics-not-laws caveats |
| `reference/VISUAL-DIFF.md` | The as-is → to-be visual for Step 4: what to draw, how to render it tool-agnostically, how to open and narrate it |
| `reference/as-is-to-be.template.html` | Self-contained, offline HTML template for the as-is → to-be visual (used when no archify-style toolchain is present) |
| `reference/SAFE-REFACTOR.md` | Characterization tests, two hats, the small-step loop, move catalog, coverage vs mutation |
| `reference/DOCS.md` | Diátaxis doc types, docstrings, README, ADR, `AGENTS.md` |
| `docs/changelog/` | Design decisions and rejected alternatives |

The reference files load on demand at the step that needs them, so the always-loaded `SKILL.md` stays legible.

## Install

Symlink the repo into your skills directory as `improve-code` (matching the skill's name):

```bash
ln -s "$(pwd)" ~/.claude/skills/improve-code
```

Or copy it if you prefer a snapshot:

```bash
cp -r "$(pwd)" ~/.claude/skills/improve-code
```

To deploy the same skill to other coding agents (Codex, opencode, …), use the `sync-skill` skill, which keeps one canonical directory and symlinks it into each agent.

## Sources

Grounded in Ousterhout (*A Philosophy of Software Design*), Parnas (information hiding), Fowler & Beck (*Refactoring*), Feathers (*Working Effectively with Legacy Code*), Dan North (CUPID), the connascence taxonomy, Diátaxis, ADRs, Archify (the as-is/to-be twin diagram as self-contained HTML), and current agent-codebase guidance. Full links live in the `## Sources` section of each reference file.
