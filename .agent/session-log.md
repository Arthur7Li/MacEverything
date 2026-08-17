# Session Log — MacEverything

Append-only. Every agent thread reads this FIRST, then appends one entry when done.
Do not delete or rewrite prior entries; if a decision is superseded, add a new entry that says so.

## Format

```
## YYYY-MM-DD — <short title> (<agent/tool used>)
- Task: <what was asked>
- Files touched: <paths>
- Decisions made: <anything a future thread must know to avoid re-deciding or reverting>
- Open questions / follow-ups: <unresolved items, or "none">
- Verification: <what was run to confirm correctness, e.g. ./gradlew benchBackend output>
```

## Example (delete once real entries exist)

## 2026-08-09 — Initial harness setup (Antigravity)
- Task: Set up AGENTS.md, session-log.md, task-template.md for cross-thread context.
- Files touched: AGENTS.md, .agent/session-log.md, .agent/task-template.md
- Decisions made: Adopted AGENTS.md as the single instruction file (not a proprietary
  per-tool format) so it works across Antigravity and any other agent tooling.
- Open questions / follow-ups: Need per-module AGENTS.md for src/backend and src/java.
- Verification: N/A (documentation only)

## 2026-08-09 — Onboarding pass (arthur, Antigravity)
- Task: Read instructions, explore repo structure, review profiling docs, and establish a benchmark baseline.
- Files touched: gradle/wrapper/gradle-wrapper.properties (attempted update)
- Decisions made: Documented that `wrapper.hpp` resides in `src/backend/headers/` (contrary to AGENTS.md claiming it is in `src/backend/cpp/`).
- Open questions / follow-ups: The machine runs Java 26 (class version 70) which breaks Gradle builds (even Gradle 8.10). A compatible JDK (e.g., 21 or 17) needs to be used to build and benchmark.
- Verification: Ran `./gradlew jar` and `./gradlew benchBackend` (failed due to Java version 26 incompatibility).

## 2026-08-09 — Resolve Java incompatibility and verify build (arthur, Antigravity)
- Task: Fix the Java environment so the project builds, get the baseline benchmark, and verify the app runs.
- Files touched: `AGENTS.md` (corrected `wrapper.hpp` path), `gradle.properties` (set Java 17 path).
- Decisions made: Reverted to using Java 17 (via Homebrew) to ensure JNI compatibility and avoid breaking the C++ integration, instead of trying to upgrade all dependencies to support Java 26.
- Open questions / follow-ups: none
- Verification: `./gradlew benchBackend` returned valid baseline metrics. User verified `./gradlew run` successfully built and launched the UI without native link errors.

## 2026-08-17 — Ignore runtime generated files (arthur, Antigravity)
- Task: Check local changes in cover/ and tmp/, ignore user/app-generated data so it is not pushed to remote.
- Files touched: `.gitignore`, `cover/*`, `tmp/*`, `.agent/session-log.md`
- Decisions made: Added `cover/`, `small_cover/`, and `tmp/` to `.gitignore` and untracked them from git (`git rm --cached`) so runtime session dumps and calculations don't dirty working tree.
- Open questions / follow-ups: none
- Verification: `./gradlew jar` and `./gradlew benchBackend` both completed successfully with zero errors.
