# Visual as-is → to-be — show the plan before you edit

Step 4 is where the user decides. A prioritized fix list reads as a wall of text; the payoff of a
refactor — a smaller **blast radius**, a **deep module** where a shallow one was, a **seam** where a
global was — is spatial, so draw it. Render a **self-contained HTML** that puts the current structure and
the proposed structure side by side, open it, and narrate it while the user picks.

This is the *archify approach*: an **as-is / to-be twin** — two diagrams of the same subject, one before
and one after — plus fix cards that map each smell to the move that removes it. It is a **proposal
artifact only**; no code changes until the user has chosen (Step 4's Done-when).

## What to draw

Draw only nodes that carry a smell or a fix — not the whole system. Keep each panel to ~4–7 nodes.

- **As-is** — the current structure with the smells marked: a shallow/god module, primitive-obsession
  types, hidden-coupling edges (a global read from many places), the nodes a target change must touch.
- **To-be** — the proposed structure: the deep module behind one interface, the value object, the seam
  drawn as an injectable (dashed) edge, and the shrunken set of nodes a change now touches.
- **Blast-radius meter** — the headline number: files a representative change touches, before → after.
- **Fix cards** — one per chosen fix: its smell, the named move, and the **net-complexity gate** result
  (what a reader no longer has to understand, minus what the change adds). This is the same list Step 3
  produced, now anchored to the picture.

## How to render — pick what the environment offers

Tool-agnostic, same as Step 2's exploration: use the best renderer present, assume none by default.

1. **An archify-style diagram toolchain, if one is installed.** These take a typed JSON IR
   (`architecture` type fits module/component structure) and emit one validated, offline HTML per
   diagram. Author two IR files — `<target>.as-is.json` and `<target>.to-be.json` — render and run the
   toolchain's own check, fixing the *JSON* on any layout error (never the renderer), then inline each
   produced `<svg>…</svg>` into one host page beside the fix cards. Keep the `.json` next to the HTML so a
   later edit re-renders instead of redrawing. Consult the toolchain's `examples/` and `schemas/` for the
   exact IR shape.
2. **Otherwise, the bundled template** [`as-is-to-be.template.html`](as-is-to-be.template.html). Copy it
   next to the target (e.g. `docs/refactor/<target>-as-is-to-be.html`), replace each `<!-- SLOT -->`
   block, and hand-draw the two panels with the template's inline-SVG node styles (`.n-box`,
   `.n-box.smell`, `.n-box.good`, `.edge.seam`). It ships filled with a worked example to imitate.

Either way the file must be **offline**: no `<script src>` / `<link href>` to any URL, everything inline —
so it opens with a double-click and survives with the repo.

## Open it, then narrate

Open the file with the platform opener — macOS `open <file>`, Linux `xdg-open <file>`, Windows
`start "" <file>`. If the environment is headless or has no opener, print the absolute path and say it is
ready to open.

Then walk it, don't just link it: name the one smell that hurts most today, trace what a change to it
touches in the **as-is** panel, show the move that collapses that in **to-be**, and land on the
blast-radius number. **Explain plainly by default** — the person deciding may not be an engineer, so keep
the precise term but gloss it the first time it appears: *blast radius* = how many files one change forces
you to touch; *deep module* = a small, simple door over a big, complex room; *seam* = a spot where you can
slot in a stand-in so a test can run. Short words, everyday images, no wall of jargon. Close by asking
which fixes to apply — the visual serves the decision, it does not replace it.

## When to skip

Default to drawing it. Skip only when the change is a single-symbol rename or a one-file tidy where a
diagram would add nothing, or when the user declines the visual — say so and present the list in prose
instead. A diagram that would take longer to draw than the fix takes to apply is itself a failed
net-complexity gate.

## Sources

Archify (as-is/to-be twin diagrams as self-contained HTML; upstream `tt-a1i/archify`, MIT). Ousterhout,
*A Philosophy of Software Design* — deep vs shallow modules. Blast radius / locality of change and the
seam concept (Feathers, *Working Effectively with Legacy Code*) carry the same weight here as in
[PRINCIPLES.md](PRINCIPLES.md) and [SAFE-REFACTOR.md](SAFE-REFACTOR.md).
