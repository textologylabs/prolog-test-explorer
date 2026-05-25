# prolog-test-explorer — Vision (1.0)

> **Status:** 🌱 Vision phase. No code yet. This document captures the design conversation that preceded scaffolding.
>
> **Migration:** This will move to ClickUp once we start building. Until then, this file is the single source of truth.

---

## Why this exists

The VSCode Prolog testing ecosystem is anaemic:

- `SkyDev.prologtester` — last updated Dec 2023, 1305 installs, 3.0★. Stagnant.
- `BennetKrzenzck.prolog-tester` — last meaningful source change ~1 year ago, 670 installs, 1 watcher. Its parser **fails on plunit's `test(name, [fail]) :- ...` syntax** — the standard idiom used throughout Bratko's textbook. Bus factor 1, with a quiet bus.

So the niche is real, the bar is low, and the existing tools don't even parse plunit correctly. Meanwhile, modern test runners in other languages (Jest Runner, Vitest, Wallaby, NCrunch) have leapt ahead with watch mode, code lens, inline failure messages, and continuous testing. Prolog has none of this.

**The framing:** *VSCode Test Explorer integration for SWI-Prolog that respects Prolog's semantics.*

A bolder framing: *The testing IDE for declarative programming.*

---

## Reference points (steal from these)

### "Wow" tier
- **Wallaby.js** — runs tests as you type, inline pass/fail on every line. Gold standard for fearless refactoring.
- **NCrunch** (.NET) — continuous testing, coverage gutters, performance metrics inline.

### "This just works" tier
- **Vitest extension** — clean code lens, sidebar tree, watch mode, structural diff for assertions.
- **Jest Runner** (firsttris) — minimalist "▶ Run | 🐞 Debug" above each test.
- **Rust Analyzer test integration** — elegant code lens model.
- **Mocha Test Adapter** (hbenl) — reference implementation of VSCode's Test API.

---

## Table stakes (every modern test runner has these)

1. **Test Explorer sidebar tree** — VSCode's native Test API.
2. **Code lens** above each test: `▶ Run | 🐞 Debug | 👁 Watch`.
3. **Gutter icons** (✅/❌) next to each test line.
4. **Inline failure messages** (peek-style, no need to leave the file).
5. **Output panel** with full test log.
6. **Status bar summary**: `✅ 12 ✗ 1 ⊘ 0`.
7. **Run-on-save / watch mode** (debounced — see Continuous Testing below).
8. **Debug integration** (set breakpoint, hit "Debug Test").
9. **Filter, search, re-run failures only.**
10. **Configurable swipl path** (per workspace, with sensible default).

If 1.0 lacks any of these, it will feel dated next to Vitest/Jest.

---

## Prolog-specific magic (the moat)

None of these exist in any Prolog tool today. This is where the extension stops being "Test Explorer with another grammar" and becomes genuinely worth using.

### A. Semantic-aware assertions
First-class UI rendering of plunit's semantic vocabulary:
- `solutions(Goal, N)` → "▦▦▦ 3 solutions found" in tree
- `succeeds(Goal)`, `fails(Goal)`, `deterministic(Goal)`, `nondeterministic(Goal)` — visible status icons differentiate "passed deterministically" from "passed but left choice points."

### B. Trace-viz integration — **the moat's moat**
Click a failing test → opens [`prolog-trace-viz`](../prolog-trace-viz/) showing the resolution tree for the failed goal. Step through unification, see where the proof dead-ended. Nobody else in the Prolog ecosystem has this.

### C. Warren integration — **deep debug story**
For users with the [Warren](../warren/) extension installed: click "Debug Test" → drops straight into Warren stepping through the failing goal at the WAM level. Heap cells, choice points, trail entries — all visible. Closes the loop from "test failed" to "I understand exactly why" in two clicks.

### D. Structural diff for terms
When expected `[1,2,3]` but got `[1,3,2]`, show a *term diff* highlighting "element at position 2: expected `2`, got `3`" — not a text diff. Prolog terms have structure; the UI should respect it.

### E. Counter-example replay
Test failed? One click and the failing goal opens in an embedded SWI REPL pre-loaded with the database. Poke interactively, find the actual solutions.

### F. Choice point & inference profiling
Every test run reports: "fired 47 choice points, 1,200 inferences." Tracked across commits — surfaces accidental complexity blow-ups when refactoring.

### G. Clause coverage
Multi-clause predicates show which clauses fired during each test. Gutter colour next to each clause head:
- 🟢 entered and succeeded
- 🟡 entered but never succeeded
- ⚪ never tried

### H. Property-based testing
First-class integration with quickcheck-style tools. Generators per type, shrinking, CLP(FD) where applicable.

### I. Mode-aware test execution
Mark tests with intended modes (`p(+,-)` vs `p(-,+)`), verify behaviour in both directions. Flag when a predicate quietly stops working in `-` mode after a refactor.

### J. Declarative reading view
Adjacent to each test, show the English reading:
`solutions(employed_child(C), 4)` → *"There should be exactly 4 employed children."*
Doubles as documentation.

### K. Snapshot/golden tests for term outputs
For predicates that produce structured terms — save expected output, diff on changes. Great for parsers, compilers, anything generative.

---

## Continuous testing (Wallaby-style) — explicit captain pick

**Run tests in the background, debounced after the user stops typing.** Never block the editor. Inline pass/fail decorations update as results arrive.

Design notes:
- **Debounce window**: ~500–800ms after last keystroke. Tunable per workspace.
- **Background process**: long-lived `swipl` REPL per workspace, kept warm to avoid startup cost. Re-consult changed files; don't restart the engine.
- **Cancellation**: new edit during a run cancels the in-flight run. Latest wins.
- **Visual feedback**: subtle "running…" indicator, ✅/❌ replaces it on completion. No popups, no toast spam.
- **Opt-in toggle** in status bar: ⚡ continuous mode on/off.
- **Resource budget**: cap CPU/memory; if a test takes >N seconds in continuous mode, skip it (mark as "manual-only").

---

## Debug integration — explicit captain pick

**"Debug Test" command per test.**
- Drops into SWI's interactive debugger with `trace.` enabled on the test's goal.
- Output streams to a dedicated debug console.
- Breakpoints set in `.pl` files honoured (via spy points).
- For users with Warren installed → option to "Debug in Warren" instead, opening the WAM stepper on the failing goal.

---

## Beyond 1.0 — aspirational

- **Mutation testing**: auto-perturb predicate clauses, check which mutations the suite catches.
- **Test-driven Prolog tutorials**: hook into Bratko-style exercises with built-in checks. Could ship as a companion skill.
- **CI mode**: `prolog-test-explorer ci` standalone binary, JUnit-XML output, GitHub Action.
- **Cross-dialect support**: SICStus, Scryer, GNU Prolog. SWI-first, others modular.

---

## Captain's picks (priority for 1.0)

From this morning's design session, these were called out as high-value:

1. **Continuous testing (Wallaby-style)** — debounced, background, non-blocking. *Captain: "obviously would do it with a bit of delay, when the user stops typing intensively, and would run it in background, so we dont block the user."*
2. **Debug integration.** *Captain: "debugging sounds fun!"*
3. **Warren integration for deeper debugging.** *Captain: "who knows maybe if the user has the Warren plugin too they can use that for deeper debugging :D"*

These three define the soul of the product. Everything else — table stakes (1–10) and the rest of the moat features (A, D, E, F, G, H, I, J, K) — is scoped around them.

---

## Naming

**Extension ID:** `prolog-test-explorer` (under publisher `TextologyLabs`).
**Display name:** *Prolog Test Explorer*.
**Marketplace ID:** `TextologyLabs.prolog-test-explorer`.

Publisher `TextologyLabs` is registered. Extension name is free on the Marketplace as of [today].

---

## Open questions

- **Bundling strategy**: ship as one extension, or split (`prolog-test-runner` core + `prolog-test-warren-integration` add-on)? Probably one extension with optional integrations triggered by Warren's presence.
- **Logo / icon**: TBD. Should pair visually with Warren and prolog-trace-viz.
- **Pricing**: free, MIT. Pro features (CI, mutation testing) could be paid much later — orthogonal decision.
- **Sponsorship / OSS funding**: GitHub Sponsors button on the repo from day one.

---

## Next steps (when we start building)

1. Migrate this vision to ClickUp (epic per moat feature).
2. Yeoman-scaffold the extension under `~/dev/products/prolog/prolog-test-explorer/`.
3. Walking skeleton: parse one `.plt`, register one test with VSCode Test API, run via `swipl -g run_tests -t halt`, surface ✅/❌. Validate the pipeline end-to-end before adding features.
4. Iterate per captain's-picks priority order.
