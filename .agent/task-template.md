# Task Template — fill this out before implementation begins

Copy this block into your Antigravity task/planning artifact. Do not skip fields — vague
fields produce vague code. Every path and symbol name below must be verified against the
actual repo (grep/read the file), not guessed.

## Repository / Module
(e.g. src/backend/cpp — native math engine)

## Description
One or two sentences: what should change and why.

## Files to Modify
- `path/to/File.java` — <what changes here, one line>
- `path/to/file.cpp` — <what changes here, one line>

(List only files you have actually opened and confirmed exist / contain the relevant symbol.)

## Implementation Notes
- Follow the existing pattern in `<real function/class name>` at `<real path>`.
- Reuse `<real type/class>` from `<real path>` instead of introducing a new one.
- Any math/algorithm assumptions being made (state them explicitly for billiard-code-sequence
  or unfolding logic — these are easy to get subtly wrong).

## Acceptance Criteria
- [ ] <concrete, testable outcome>
- [ ] <concrete, testable outcome>
- [ ] Existing behavior <X> still works (regression guard)

## Test / Verification Requirements
- [ ] `./gradlew jar` builds clean
- [ ] `./gradlew benchBackend` run before AND after if touching src/backend/cpp or metal/
- [ ] Manual smoke test in `./run_mac.sh` if touching UI/viewer code

## Session Log Entry (fill after completion)
Append to `.agent/session-log.md` using the standard format once this task is done.
