# AGENTS.md — MacEverything (Billiards Everything, macOS fork)

This file is the operational contract for any AI agent (Antigravity, Copilot, Claude Code, etc.)
working in this repository, across any thread or session. Read this before making changes.
Nested `AGENTS.md` files in `src/backend/` and `src/java/` add module-specific rules and take
precedence over this file for their subtree.

## 1. Project Overview

MacEverything is a macOS-native fork of "Billiards Everything," a research tool for finding
periodic billiard paths in polygons (an open problem since 1775). It ports the original
Java-based tool with a GPU-accelerated native backend for Apple Silicon.

Architecture (do not restructure without an explicit human decision):
- `src/java/billiards/viewer/` — Swing UI. Entry point: `Main.java`. Long-running compute is
  dispatched via `PolyVaryTask`, `CycleVaryTask`, `VaryLTask` (implement `Callable`, report
  through `Progress`/`ProgressWithStatus`).
- `src/java/billiards/wrapper/Wrapper.java` — JNI boundary to the native library. Any signature
  change here requires a matching change in `src/backend/cpp/wrapper.cpp` and `wrapper.hpp`.
- `src/backend/cpp/` — core math: `code_sequence.cpp`, `bounding_inequalities.cpp`,
  `unfolding.cpp`, `triangle_billiard.cpp`, `vary3.cpp`/`vary4.cpp`. This is the actual research
  algorithm; treat changes here as math-correctness-critical, not routine refactors.
- `src/backend/metal/` — Metal GPU kernels (`eliminate.metal`) for the heaviest compute paths.
- `docs/algorithmic-optimization-opportunities.md` and `docs/performance-changes.md` — read
  before proposing any performance work; they contain prior profiling findings.

## 2. Build, Test, and Package Commands

- Build native + Java: `./gradlew jar`
- Deterministic backend benchmark (use for before/after perf claims, ~0.1% variance):
  `./gradlew benchBackend`
- macOS packaging (rebuilds jar first, bundles dylibs recursively via otool -L, ad-hoc signs):
  `./package-mac.sh`
- Run locally without packaging: `./run_mac.sh`
- Do NOT hand-edit the dylib bundling list in `package-mac.sh`; it is a recursive resolver on
  purpose (fixed a real launch crash from missing transitive Boost dylibs).

## 3. Development Workflow

- Before editing anything in `src/backend/cpp/` or `src/backend/metal/`, write a short
  implementation plan (what changes, why, expected effect on `benchBackend` numbers) and get it
  reviewed before executing.
- Before starting a task, read `.agent/session-log.md` for context from other threads/sessions.
- After finishing a task, append an entry to `.agent/session-log.md` (what changed, files
  touched, open questions).
- Use `.agent/task-template.md` to scope any non-trivial task before implementation.
- Preserve `// <Author>, <Year>` authorship markers above changed regions; add your own for new
  regions instead of removing existing ones.

## 4. Common Pitfalls / Prohibited Patterns

- The native cancel flag (`backend_cancel`) is a process-wide atomic. Reset it at the START of
  each vary task's `call()`, never in a UI cancel handler — resetting it while C++ threads are
  still running can make cancellation silently ineffective (this bug shipped once already).
- Do not reintroduce nested `boost::asio` thread pools for curve generation; use
  `tbb::parallel_for` (already migrated in `unfolding.cpp` to fix CPU oversubscription).
- Packaged builds must use `-mcpu=apple-m1` (via the `-Pportable` Gradle flag), not
  `-march=native`, or the .dmg will crash on non-build-machine Apple Silicon Macs.
- Do not commit large binaries; `bfg-1.15.0.jar` was already used once to scrub history — use it
  again rather than letting a large artifact merge to a shared branch.
- Empty/malformed polygon input must show a friendly alert (see `createConvexPolygon`), not throw
  an uncaught `NumberFormatException`.

## 5. Terminology Cheat Sheet (for consistent agent understanding)

- "Vary" = iteratively perturbing a polygon's code sequence to search for periodic paths.
- "Code sequence" = the combinatorial encoding of a billiard trajectory's bounces.
- "Unfolding" = reflecting the polygon across its edges to linearize the billiard path.
- "Cover" = a diagram/window showing coverage of known periodic path patterns.
