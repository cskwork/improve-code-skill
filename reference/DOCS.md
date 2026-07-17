# Documentation

For Step 7. Great docs are not more words — they are the *right type* in *one* place, close to the code, verified to still be true.

## Pick the doc type by reader need (Diátaxis)

Four types, each serving a different need. **Never mix two types in one document** — that is what makes docs confusing.

| Type | Reader's need | Form |
|------|---------------|------|
| **Tutorial** | Learn by doing, first time | A guided lesson that is guaranteed to work start to finish |
| **How-to** | Accomplish one specific task | Numbered steps for a goal the reader already has |
| **Reference** | Look up a precise fact | Dry, complete, structured — API signatures, flags, config |
| **Explanation** | Understand why it is this way | Discussion of trade-offs, background, design rationale |

Deciding what to write: a new contributor onboarding → tutorial; "how do I add a provider?" → how-to; "what does this function return?" → reference (or a docstring); "why is it split this way?" → explanation (often an ADR).

## Docstrings and purpose headers

- **Scale depth to significance.** A one-line summary when the signature already speaks; full contract (arguments, returns, raises, and the *why*) for public APIs and non-obvious helpers.
- **Say why and what-contract, not how.** The body shows how. The docstring states intent and the contract a caller relies on.
- **One style repo-wide.** Pick a docstring convention (Google-style is a safe default) and record it in the linter config so it is enforced, not remembered.
- **Write the docstring before the implementation** as a design check — if the contract is hard to state, the interface is wrong.
- A purpose header on a module or public function doubles as a signpost for an agent that arrives with no project memory.

## README

The README is the front door. Its most common failure is **stale install and usage commands** — a command pointing at a flag or script that no longer exists. So:

- Keep setup, build, test, and run commands current, and *verify they actually run* as part of the change that touched them.
- Cover: what this is, how to install/run, how to test, and where to look next. Link deeper docs rather than inlining them.

## ADR — Architecture Decision Record

For a decision that is architecturally significant and would otherwise be re-litigated: one short Markdown file per decision, in-repo (`docs/adr/`), with **Title, Status, Context, Decision, Consequences**.

- Accepted ADRs are **immutable** — when a decision changes, add a new ADR that *supersedes* the old one; never edit history.
- In this skill, offer an ADR when the user rejects a fix for a load-bearing reason a future explorer would need — so the next run does not re-suggest the same thing. Skip ephemeral reasons ("not worth it now") and self-evident ones.

## `AGENTS.md` / `CLAUDE.md` — the map for agents

A "README for agents," separate from the human README:

- Exact build/test/lint commands, conventions that differ from defaults, and never-touch boundaries.
- Keep it **lean** — a map with pointers and critical gotchas, not the territory. Agents reliably follow only a limited number of instructions, so spend them on what cannot be inferred.
- Encode the *what* (formatting, import order, naming) as deterministic tooling — linters, formatters, LSP, committed `.gitignore` — and reserve prose for the *why*. Prose conventions drift as agents generate more code.

## One source of truth

Every fact lives in exactly one canonical place. Delete or redirect contradictory copies across READMEs, comments, and agent-config files. Prefer a symlink or an `@import` over a duplicated paragraph. Scattered, contradictory truth taxes every reader, human or agent, equally.

## Sources

- Diátaxis framework: https://diataxis.fr/
- Architecture Decision Records: https://adr.github.io/
- `AGENTS.md` convention: https://agents.md/
- Anthropic best practices (lean agent instruction files): https://code.claude.com/docs/en/best-practices
