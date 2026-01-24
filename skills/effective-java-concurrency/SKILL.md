---
name: effective-java-concurrency
description: "Apply Effective Java (2nd edition) concurrency best practices when writing or reviewing multi-threaded Java code: synchronization/visibility, avoiding excessive locking, executors, concurrency utilities, thread-safety docs, lazy init, scheduler independence, and avoiding thread groups."
---

# Effective Java Concurrency (2nd ed)

Guideline checklist derived from `docs/effective_java.md` (Bloch, Effective Java 2nd edition, Items 66-73).
If another repo skill sets stricter rules, follow the stricter one.

## Quick workflow

1. Identify all shared mutable state and define the locking/visibility policy.
2. Prefer not sharing mutable state at all (immutability, confinement, or safe publication).
3. Prefer high-level concurrency utilities (executors, concurrent collections, synchronizers).
4. Keep synchronized regions small; avoid calling client/overridable code while holding locks.
5. Document the thread-safety guarantees and any required external locking.

## Checklist by item (66-73)

- 66: Synchronize access to shared mutable data (mutual exclusion + visibility); use `volatile` only for visibility; use atomics for atomic updates.
- 67: Avoid excessive synchronization; do as little work as possible under lock; use open calls (move alien method calls outside locks).
- 68: Prefer executors and tasks to threads; use `ExecutorService` and thread pools; shut down executors cleanly.
- 69: Prefer concurrency utilities to `wait`/`notify`; use concurrent collections and synchronizers; if you must, use the wait-loop idiom and prefer `notifyAll`.
- 70: Document thread safety (immutable / unconditionally thread-safe / conditionally thread-safe / not thread-safe / thread-hostile); document which lock guards which state.
- 71: Use lazy initialization judiciously; prefer eager init; otherwise use holder idiom (static), synchronized accessors, or double-check idiom with `volatile`.
- 72: Do not depend on the thread scheduler for correctness or performance; avoid busy-wait and oversubscription.
- 73: Avoid thread groups (obsolete).

## Common red flags

- Unsynchronized read/write of shared mutable fields (especially "write without synchronization, read with synchronization" or vice versa).
- `volatile` used for non-atomic compound actions (e.g., `++`).
- Calling callbacks / overridable methods while holding a lock.
- Unbounded thread creation or "new Thread(...)" in library code instead of an executor.
- `wait()` or `notify()` used outside the standard wait-loop idiom.

## Getting the full details quickly

Prefer opening the smallest relevant reference file (same original structure + code blocks):

- `.codex/skills/effective-java-concurrency/references/10-concurrency.md`

Legacy (large, avoid loading unless you truly need everything at once):

- `.codex/skills/effective-java-concurrency/references/effective_java.md`

You can also search the repo source summary:

- File: `docs/effective_java.md`
- Find a specific item: `rg -n '^## 66\\.' docs/effective_java.md`
