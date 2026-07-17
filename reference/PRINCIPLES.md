# Principles, Vocabulary, and Smell Catalog

The backbone for Step 3 (diagnose) and Step 7 (navigate). Everything here serves one aim: **reduce the context needed to make one correct change** — for an agent and a human alike.

## Vocabulary

Shared words keep every suggestion precise. Use these exactly; keep to the sharpest term rather than a vague "component," "service," or "boundary." The four headline terms — **deep module**, **shallow module**, **blast radius**, **seam** — are defined in [SKILL.md](../SKILL.md); the rest follow.

| Term | Meaning |
|------|---------|
| **Information hiding** | Draw a module boundary around a design decision *likely to change*, and hide it. Each module owns one "secret." |
| **Connascence** | Coupling taxonomy: two elements are connascent if changing one forces changing the other. Weak→strong: Name < Type < Meaning < Position < Algorithm < Execution-order < Timing < Value < Identity. Maximize it *inside* a module, minimize it *across* boundaries. |
| **Net-complexity gate** | A change must remove more for a reader to understand than it adds in files, interfaces, and indirection. If it does not, skip it. |
| **Smell** | An observable signal that a principle is being violated *here*. Fixes are triggered by smells, never applied top-down. |
| **Guard clause** | Handle preconditions and edge cases up front with an early return, so the happy path reads un-indented and last. |
| **Value object** | An immutable, self-validating typed wrapper over a primitive (`Money`, `EmailAddress`, `UserId`) that makes illegal states unrepresentable. |
| **Why-comment** | Prose that explains intent, a business rule, or a non-obvious trade-off — never one that restates what the code plainly does. |

## The principle set

Each principle is stated as something to move *toward*, with the smell that pulls you there.

1. **Reduce hidden context per change.** The test of a design: can a fresh reader find the behavior, change one bounded area, and run the smallest meaningful verification without opening the whole system? If not, the design leaks hidden context — that is the thing to fix.

2. **Build deep modules.** Before adding a class, interface, or layer, confirm it hides more than it exposes. Prefer a small interface over a rich implementation. A deep module gives you a seam to rewrite internals behind tests.

3. **Hide decisions likely to change.** Decompose by *secret*, not by processing step or technical type. If you cannot name the secret a module hides, the boundary is wrong.

4. **Pass the net-complexity gate every time.** Indirection is not free. One implementation and no real volatility behind an interface means the interface is cost with no return — remove it and note why.

5. **Decompose for a reason, never for a line count.** Extract a unit only when it hides genuine depth, removes real duplication, or isolates a separately-testable concern. A cohesive 40-line function that reads top-to-bottom beats six shallow one-call functions you must chase across the file.

6. **Keep coupling weak and local.** No module reaches into another's tables, state, or side effects. Kill hidden coupling — global state, temporal ordering, control flags (`is_preview`, `skip_validation`), scattered policy. Where coupling must exist, prefer the weakest form and keep it inside one module.

7. **Map the domain to first-class names.** A domain action named in a prompt or a conversation ("process a refund") should have a matching symbol. Pick one naming convention repo-wide; when you rename, fix every occurrence — agents replicate whatever convention they last saw.

8. **Say why, not what.** Delete comments that restate code. Reserve prose for intent and non-obvious trade-offs. When naming alone cannot make code self-explanatory, a why-comment is correct — do not shred logic into fragments just to avoid one. Guard existing why-comments through a refactor.

9. **Make verification fast and narrow.** Small hermetic tests and single-purpose checks let a change be self-corrected without a slow full build. Point to the exact command.

## Smell catalog

Diagnose by matching what you see to a row, then apply the fix. Only act where the smell is genuinely present.

| Smell | Why it hurts | Fix toward |
|-------|--------------|-----------|
| Pass-through method / one-line wrapper forwarding to a near-identical signature | Interface surface with no abstraction | Inline it, or deepen it into a real module |
| Shallow module — interface almost as complex as its body; config-heavy signature pushing complexity to callers | Callers carry the complexity the module should hide | Widen the module's job; simplify the interface |
| Interface with a single implementation, or a DI container added just to pass one argument | Indirection with no volatility to justify it | Delete the abstraction; inject plainly if needed |
| Deeply nested conditionals (nesting > 3–4), arrow-shaped code | Reader must hold every condition to reach the happy path | Guard clauses; extract the middle |
| Primitive obsession — raw string/int for a domain concept, validation duplicated at call sites | Illegal states are representable; rules drift | Value object that validates once |
| Magic literals; data clumps of arguments that always travel together | Meaning is implicit; changes are error-prone | Named constant; parameter object |
| Hidden coupling — global state, temporal ordering, control flags, scattered policy | Local reasoning is impossible; blast radius is huge | Make dependencies explicit; localize policy |
| Strong connascence crossing a boundary; duplicated magic strings across distant files | One change forces edits in many places | Weaken the form, or pull it inside one module |
| Inconsistent naming (`getUserById` vs `fetch_customer` vs `loadAccount`) | Agents add a fourth convention; humans mis-search | One convention, applied everywhere |
| A domain action from prompts has no matching symbol | Agents invent one or graft onto a generic DB call | Name it as a first-class method/type |
| Junk-drawer folders (`utils/`, `helpers/`, `misc/`) and layer-first trees (`controllers/`, `services/`) | Structure hides the domain; related code scatters | Organize by feature/domain; colocate |
| Long function, feature envy, shotgun surgery | Named Fowler smells with named moves | Extract Function, Move Method, consolidate |
| Contradictory or stale docs; README install/usage pointing at flags that no longer exist | A direct reading tax; the #1 documented doc failure | One source of truth; verify commands run |
| Untested branch at the target | No safety net to refactor behind | Characterization test first (see SAFE-REFACTOR.md) |

## Agent-navigability checklist

What specifically helps an AI agent — beyond good engineering, under the one hard constraint that an agent has no project memory and a narrow view:

- A lean `AGENTS.md` / `CLAUDE.md` map at the root, and one authoritative source of truth per fact — how to author both is in [DOCS.md](DOCS.md).
- Domain concepts are first-class named symbols, so natural-language prompts map onto real code.
- Deep modules with explicit contracts, so an agent can change internals behind a stable seam.
- Concise purpose headers on modules and public functions — they double as generation-time reasoning prompts and improve retrieval on a later visit.
- Machine-enforced conventions (linter, formatter, LSP, committed `.gitignore`) over prose rules an agent will drift from (see [DOCS.md](DOCS.md)).
- Hidden coupling removed, so local reasoning is enough for a correct change.

## These are heuristics, not laws

The single most documented failure is applying any of these dogmatically. State them as trade-offs, not absolutes:

- **Tiny functions vs deep methods.** "Functions can't be too small" (Clean Code) collides with "shallow methods add complexity" (Ousterhout). Decide on depth, duplication, and testability — not line count. Martin and Ousterhout publicly converged here: a split needs a reason beyond size.
- **Comments as failure vs necessity.** Not every comment is a failure to name things. Business rules and non-obvious trade-offs earn prose.
- **Dependency inversion.** Do not invert a stable, single-implementation dependency behind an interface just to satisfy a rule — that is over-engineering.
- **"Define errors out of existence."** Redesigning an API so an error can't arise (an out-of-range delete becomes a no-op) is good; using it as license to swallow genuine errors is not.
- **Agent-friendliness is mostly just good engineering** under one hard constraint (the context window). Reach for fundamentals, not AI-specific gimmicks.

## Sources

- Ousterhout, *A Philosophy of Software Design* — deep modules, define-errors-out-of-existence: https://web.stanford.edu/~ouster/cgi-bin/aposd.php
- Ousterhout vs Martin, decomposition debate: https://github.com/johnousterhout/aposd-vs-clean-code
- Parnas, *On the Criteria To Be Used in Decomposing Systems into Modules*: http://sunnyday.mit.edu/16.355/parnas-criteria.html
- Dan North, *CUPID — for joyful coding* (properties over SOLID rules): https://dannorth.net/cupid-for-joyful-coding/
- Connascence reference: https://connascence.io/
- Agentic codebase principles (locality, blast radius, navigability): https://maintainable.software/agentic-engineering-part-2-agentic-codebase-principles/
- Anthropic, working with large codebases / best practices: https://code.claude.com/docs/en/best-practices
- `AGENTS.md` open convention: https://agents.md/
- Using linters to direct agents: https://factory.ai/news/using-linters-to-direct-agents
