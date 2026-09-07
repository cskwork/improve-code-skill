# Safe Refactoring and Tests

The discipline for Steps 5–6: change structure without changing behavior, and prove it. The rule under everything — **never refactor code that is not covered by a test that would fail if behavior changed.**

## Pin behavior first: characterization tests

When the target has no test, write a **characterization test** (a "golden master" / "pinning test"): it captures what the code *actually does today*, not what it should do. That is the net you refactor behind.

Capture trick for unknown output:

1. Assert an obviously-wrong value (`assert result == "TODO"`).
2. Run the test; read the actual value from the failure.
3. Paste that actual value in as the expected. The test now pins current behavior.

Then:

- **Inject fakes only at real boundaries** — clock, network, filesystem, DB, randomness. Do not fake pure logic; test it directly.
- **Mask non-determinism** — timestamps, random IDs, ordering, memory addresses — or the golden master produces false failures that erode trust. For large or complex output, use an approval-testing library (ApprovalTests, Verify, snapshot tests).
- **Find a seam** to inject those fakes. If none exists, make the smallest enabling refactoring to create one (Extract Interface, parameterize a constructor) — this is the way out of the legacy catch-22.

## Change under green tests: two hats

At any moment wear exactly one hat:

- **Refactoring hat** — restructure, no behavior change. Tests must stay green the whole time.
- **Adding-function hat** — new behavior, with its test written first.

Never both in one edit or commit. A red test while wearing the refactoring hat means the last step was wrong: **revert that step, do not edit the test to match.**

## The loop

For each planned fix:

1. Apply **one** named move (see catalog below), in the smallest step that stands alone.
2. Run the tests protecting the changed behavior; broaden coverage at integration when warranted.
3. Green → checkpoint (commit or note it). Red → revert this step and retry smaller.
4. Repeat.

Small reversible steps are the point. A big edit that goes red gives you nothing to bisect.

## Move catalog

The common structure-preserving moves, each triggered by a smell from PRINCIPLES.md:

| Move | Use when |
|------|----------|
| Extract Function | A block has a nameable job inside a long function |
| Inline Function/Variable | A pass-through wrapper or needless indirection adds surface, not meaning |
| Rename (via IDE/LSP tooling) | A name misleads; apply across *every* occurrence at once |
| Replace Nested Conditional with Guard Clauses | Arrow-shaped code hides the happy path |
| Replace Magic Literal with Named Constant | An unexplained literal recurs or carries meaning |
| Introduce Parameter Object | Arguments always travel together (a data clump) |
| Replace Primitive with Value Object | A domain concept rides on a raw string/int with duplicated validation |
| Extract Class / Move Method | A class does two jobs (low cohesion) or a method envies another class |
| Extract Interface (seam) | You need to inject a fake, or a real second implementation appears |

Prefer automated tooling (LSP rename, IDE extract) where reliable, then review and test its diff; generated edits can still miss dynamic references.

## Changes too big to do in place

- **Preparatory refactoring** — "make the change easy, then make the easy change." Reshape under green tests first (refactoring hat), then add the feature (function hat, tests first).
- **Sprout / Wrap** — to add behavior to risky untested code without editing it, put the new, test-driven logic in a fresh method/class (Sprout) or decorate the old behavior (Wrap).
- **Strangler fig** — for a subsystem too large to refactor in place, build the new alongside the old behind a facade, route slices across incrementally, and retire the legacy piece by piece. Never a big-bang rewrite — those "go down in flames most of the time."

## Coverage is a gap-finder, not a score

Line coverage tells you which lines *ran*, not whether a wrong result would be *caught* — 95% coverage routinely hides assertion-free tests. Read coverage as "what is definitely not tested," and confirm branches and boundaries are actually *asserted*. For critical logic under change, run **mutation testing** scoped to that code (it injects faults like `>` → `>=` and checks your tests fail); drive surviving mutants toward zero. Scope it tightly — it is slow.

## Verify with evidence

Definition of done for the refactoring phase:

- The same tests are green *before and after* the pure-refactor steps (proof behavior held).
- Repository-required checks for the affected surface pass; name any pre-existing failures and checks not run. Reuse current results for unchanged state.
- Any intended behavior change is a separate, test-first, separately-committed change.

Report what you actually ran. Do not claim success without the output.

## Sources

- Fowler & Beck, *Refactoring* (2nd ed.) — small steps, two hats: https://martinfowler.com/books/refactoring.html
- Feathers, *Working Effectively with Legacy Code* — seams, characterization tests (summary): https://understandlegacycode.com/blog/key-points-of-working-effectively-with-legacy-code/
- Strangler Fig application: https://martinfowler.com/bliki/StranglerFigApplication.html
- Mutation testing vs coverage as a vanity metric: https://about.codecov.io/blog/mutation-testing-how-to-ensure-code-coverage-isnt-a-vanity-metric/
