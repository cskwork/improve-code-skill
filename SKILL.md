---
name: improve-code
description: Improve and refactor a codebase for both AI-agent navigability and human readability, applying the change under green tests. Use when the user wants to refactor or restructure code; improve readability, maintainability, or module and interface design (SOLID, coupling, cohesion); make a codebase easier for an agent or teammate to navigate or onboard; or add the tests and docs a reshape needs.
---

# Improve Code

Reshape a target so that making one correct change needs less context. The bar is dual and non-negotiable: an **AI agent** and a **human** must both be able to find the behavior, change one bounded place, and verify it — without reading the whole system.

Work only where a **smell** is present and the fix passes the **net-complexity gate** (it removes more to understand than it adds). Absent a smell and a named payoff, leave the code alone — a patch that fights the existing structure is a defect, not an improvement.

Three leading ideas carry the whole skill:

- **Deep module** — a small, stable interface hiding a large implementation. The opposite, a **shallow module** (interface nearly as complex as the code behind it: thin wrappers, pass-through methods), adds cost without hiding complexity. Deepen shallow things; do not shred deep things into fragments.
- **Blast radius** — how many places one change must touch. Good structure keeps it to one place (high **locality**).
- **Seam** — a point where behavior can be substituted (inject a fake for the clock, network, filesystem, or DB). Seams are where tests attach and where internals get rewritten safely.

Reference material is disclosed on demand:

- Diagnosing (Step 3) and naming (Step 7): the full smell catalog, vocabulary, and principle set live in [PRINCIPLES.md](reference/PRINCIPLES.md).
- Proposing (Step 4): how to render and open the as-is → to-be visual lives in [VISUAL-DIFF.md](reference/VISUAL-DIFF.md).
- Testing and changing safely (Steps 5–6): the discipline and the move catalog live in [SAFE-REFACTOR.md](reference/SAFE-REFACTOR.md).
- Documenting (Step 7): which doc to write, and how much, lives in [DOCS.md](reference/DOCS.md).

## Steps

Run them in order. Each ends on a checkable **Done when** line; finish it before starting the next.

### 1. Set the goal

Ask the user what to improve and why, unless the conversation has already made it explicit. Draw out four things and nothing more:

- **Target** — the file, module, or flow in scope.
- **Payoff** — the pain to relieve: hard to read, hard to change, hard to test, or hard for an agent to navigate.
- **Constraints** — must behavior and the public interface stay identical? Any area that is off-limits?
- **Definition of done** — the observable state that ends the task.

Do not proceed on a guessed goal. If the request is broad ("clean this up"), narrow it to one target with the user first.

**Done when:** you can state the goal back in one short paragraph (target + payoff + constraints) and the user confirms it.

### 2. Explore and pin the ground truth

Read before proposing. Map the target — what it calls, what depends on it — using whatever code-navigation tools this environment offers: a codebase knowledge graph or language server when present, otherwise plain search and read. Choose the tools yourself; do not assume any particular one exists. When the exploration is broad, hand it to a read-only subagent so the main context keeps the picture, not the raw file dumps. State, in plain language, what the target does today.

Then run the existing test suite. Confirm it is **green** and fast. If it is red or flaky, stop and stabilize first — you cannot tell a regression from pre-existing noise otherwise.

**Done when:** you can describe the target's current behavior in plain language, and you know its test state (green, red, or absent).

### 3. Diagnose against the principle set

Walk the target for **smells** from [PRINCIPLES.md](reference/PRINCIPLES.md) — shallow modules and pass-through methods, hidden coupling (global state, temporal ordering, control flags), strong **connascence** crossing a boundary, primitive obsession, deep nesting, junk-drawer or layer-first structure, inconsistent naming, and stale or contradictory docs.

For every candidate fix, apply the **net-complexity gate**: does it remove more for a reader to understand than it adds in new files, interfaces, or indirection? An abstraction layer with one implementation and no real volatility fails the gate — drop it and say why. Treat SOLID, connascence, and Clean-Code rules as heuristics that a smell triggers, never as laws to apply top-down.

**Done when:** you have a prioritized list where each item names its smell, the principle behind it, the concrete fix, and the payoff — and every item has passed the gate.

### 4. Agree the plan

Present the prioritized fixes — and *show* them, don't just list them. Render a self-contained **as-is → to-be** visual: the current structure and the proposed one side by side, the blast radius before and after, and one card per fix mapping a smell to its move. Open it in the browser and walk the user through it. Building and opening it tool-agnostically — an archify-style diagram toolchain if the environment has one, otherwise the bundled template — is covered in [VISUAL-DIFF.md](reference/VISUAL-DIFF.md). Draw it by default; drop to prose only for a trivial one-symbol change or when the user declines.

Recommend the smallest high-leverage set rather than the whole list. Surface risk, and flag anything that contradicts an existing ADR (see [DOCS.md](reference/DOCS.md)) so the user can decide whether to reopen it. Let the user pick before you edit anything.

**Done when:** the user has seen the as-is → to-be visual (or agreed to skip it) and has chosen which fixes to apply, or approved your recommendation.

### 5. Pin behavior with tests

Refactoring rests on tests that fail if behavior changes. For any area you will touch that lacks such a test, add a **characterization test** first — one that pins what the code *actually* does now, not what it should do. Inject fakes only at real boundaries (clock, network, filesystem, DB) and mask non-deterministic output (timestamps, random IDs, ordering). The how-to, and the rule this rests on, are in [SAFE-REFACTOR.md](reference/SAFE-REFACTOR.md).

**Done when:** every area the plan will touch is covered by a green test that would fail if its behavior changed.

### 6. Apply surgically — one hat, small steps

Wear the **refactoring hat**: change structure, not behavior. Apply one named move at a time from [SAFE-REFACTOR.md](reference/SAFE-REFACTOR.md) — deepen a shallow module, replace a nested conditional with **guard clauses**, introduce a parameter object or value object, rename a concept across every occurrence, inline a pass-through wrapper. Run the tests after each move: green means checkpoint it; red means revert that one step and retry smaller — do not debug forward.

Match the surrounding style. Touch only what the goal requires. Preserve existing **why-comments**. Any intended behavior change waits for a separate, test-first commit under the adding-function hat.

**Done when:** every chosen fix is applied, the suite is green after each step, and the diff is scoped to the target with unrelated code untouched.

### 7. Make it navigable — names, docs, structure

Close the loop for the next reader, human or agent:

- **Names** — domain nouns and verbs map to first-class symbols (a `processRefund` method, a `Refund` type), one naming convention repo-wide; a rename fixes *every* occurrence.
- **Docs** — add a concise purpose header or docstring (the *why* and the contract) wherever a signature does not speak for itself. Update every doc the change touched *in the same diff* — README snippet, docstring, reference, ADR pointer — keeping one source of truth with no contradictions, and verify any install or usage command still runs. [DOCS.md](reference/DOCS.md) decides which doc type and how much.
- **Structure** — where it helped the goal, related code is colocated and hidden coupling is gone, so local reasoning is enough for a correct change.

**Done when:** every renamed concept is updated at all occurrences; every doc the change touched is updated in the same diff with its commands verified to run; and no contradictory source of truth remains.

### 8. Verify and record

Run the exact tests, linter, type checker, and build for the target. Report precisely what ran and what passed — quote the evidence; do not claim success without it. Then record the decision where the repo already keeps a trail — a changelog entry, an ADR, or the PR description — capturing the smell you fixed and the alternative you rejected and why.

**Done when:** the named checks are green with quoted evidence, and the decision is recorded.
