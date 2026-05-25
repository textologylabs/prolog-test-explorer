# Prolog Test Explorer

> A VSCode test runner for SWI-Prolog that respects Prolog's semantics.

**Status:** 🌱 Vision phase. No code yet. See [VISION.md](VISION.md) for the 1.0 design.

## What this is

VSCode Test Explorer integration for SWI-Prolog's [plunit](https://www.swi-prolog.org/pldoc/man?section=plunit) — done properly. Continuous testing (Wallaby-style), debug integration, and deep integration with [Warren](https://github.com/jarecsni/warren) (WAM visualizer) and [prolog-trace-viz](https://github.com/jarecsni/prolog-trace-viz) for resolution-tree debugging when tests fail.

The existing Prolog test extensions are stagnant or broken on standard plunit syntax. This project aims to fix that — and then go further than any test runner in the Prolog ecosystem has gone.

## Why

The VSCode Prolog testing ecosystem is anaemic. Meanwhile, modern test runners in other languages (Jest Runner, Vitest, Wallaby) have leapt ahead with watch mode, code lens, inline failure messages, and continuous testing. Prolog has none of this.

This extension aims to close that gap, and to exploit Prolog's declarative semantics for things imperative test runners can't do — structural term diffs, choice-point profiling, clause coverage, and one-click drop into WAM-level debugging via Warren.

## Status

Pre-implementation. The vision is captured in [VISION.md](VISION.md); building has not started. The repository is public from day one so the design conversation is in the open.

## Roadmap

See [VISION.md](VISION.md) for the full 1.0 scope. Tracking will move to ClickUp once active work begins.

## License

MIT — see [LICENSE](LICENSE).
