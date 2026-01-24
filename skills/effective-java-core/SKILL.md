---
name: effective-java-core
description: "Apply Effective Java (2nd edition) best practices when writing or reviewing Java code: object creation/lifecycle, equality, class design, generics, enums/annotations, methods, general programming, exceptions, and serialization (excludes concurrency)."
---

# Effective Java Core (2nd ed)

Guideline checklist derived from `docs/effective_java.md` (Bloch, Effective Java 2nd edition).
If another repo skill sets stricter rules, follow the stricter one.

Use `effective-java-concurrency` for concurrency-specific guidance (Items 66-73).

## Quick workflow

1. Identify the API surface (public/protected). Minimize it and document contracts.
2. Prefer immutability and defensive copies; avoid exposing mutable internals.
3. Choose the right construction pattern (static factory / builder / singleton) and lifetime rules.
4. Make object methods correct (`equals`/`hashCode`/`toString`/`Comparable`); avoid `clone`.
5. Use generics, enums, and exceptions intentionally; avoid cleverness unless measured.
6. Treat serialization as exported API; avoid it unless you truly need it.

## Checklist by item

### Creating and destroying objects (1-7)

- 1: Prefer static factories when naming, instance control/caching, or subtype returns help.
- 2: Use builders when constructors/static factories would have many params; avoid telescoping.
- 3: Enforce singleton with `enum` (preferred) or private constructor + static field/factory.
- 4: Enforce noninstantiability for utility classes with a private constructor.
- 5: Avoid creating unnecessary objects (reuse immutables; cache expensive objects; avoid boxing).
- 6: Eliminate obsolete references (null out unused refs; watch caches/listeners/ThreadLocals).
- 7: Avoid finalizers; prefer `AutoCloseable` + try-with-resources for cleanup.

### Methods common to all objects (8-12)

- 8: `equals` must be reflexive, symmetric, transitive, consistent, and return false for null.
- 9: If overriding `equals`, always override `hashCode` (equal objects -> equal hash codes).
- 10: Override `toString` for value-ish types; include key state; document format if relied upon.
- 11: Avoid `clone`; prefer copy constructor/factory. If implementing, obey the contract.
- 12: Implement `Comparable` when a natural order exists; avoid subtraction in comparisons.

### Classes and interfaces (13-22)

- 13: Minimize accessibility of classes/members; expose the smallest surface that works.
- 14: In public classes, use accessors; do not expose mutable public fields.
- 15: Minimize mutability; make classes/fields `final` where possible; prevent representation leaks.
- 16: Favor composition over inheritance; use inheritance only for true "is-a" relationships.
- 17: Design and document for inheritance, or else prohibit it (e.g., make the class `final`).
- 18: Prefer interfaces to abstract classes; keep implementations swappable.
- 19: Use interfaces to define types; do not use "constant interfaces".
- 20: Prefer class hierarchies to tagged classes; leverage polymorphism.
- 21: Use function objects to represent strategies (pass behavior rather than branch on tags).
- 22: Favor static member classes over nonstatic to avoid implicit outer references.

### Generics (23-29)

- 23: Do not use raw types; always parameterize generic types.
- 24: Eliminate unchecked warnings; keep any `@SuppressWarnings("unchecked")` local and justified.
- 25: Prefer lists to arrays (arrays are covariant/reified; generics are invariant/erased).
- 26: Favor generic types for reusable components.
- 27: Favor generic methods for reusable algorithms.
- 28: Use bounded wildcards to increase flexibility (PECS: producer `extends`, consumer `super`).
- 29: Consider type-safe heterogeneous containers for keyed type->value storage patterns.

### Enums and annotations (30-37)

- 30: Use enums instead of `int` constants.
- 31: Use instance fields instead of ordinals; never persist/communicate `ordinal()`.
- 32: Use `EnumSet` instead of bit fields.
- 33: Use `EnumMap` instead of ordinal indexing.
- 34: Emulate extensible enums with interfaces (when clients must supply their own "enum" values).
- 35: Prefer annotations to naming patterns.
- 36: Use `@Override` consistently.
- 37: Prefer marker interfaces over marker annotations when you want a true "type" signal.

### Methods (38-44)

- 38: Validate parameters; document constraints; fail fast with the right exception.
- 39: Make defensive copies when needed (especially of mutable inputs/outputs).
- 40: Design signatures carefully (clarity > brevity; avoid long param lists; prefer enums).
- 41: Use overloading judiciously; prefer distinct method names when overloads can confuse.
- 42: Use varargs judiciously; avoid varargs in performance-critical paths.
- 43: Return empty arrays/collections, not nulls.
- 44: Write doc comments for every exposed API element; describe contracts and edge cases.

### General programming (45-56)

- 45: Minimize scope of local variables; initialize at declaration where possible.
- 46: Prefer for-each loops to indexed loops unless you need indices or mutation by index.
- 47: Know and use libraries; do not re-invent standard utilities.
- 48: Avoid `float`/`double` for exact values; use `BigDecimal`, `int`, or `long`.
- 49: Prefer primitives to boxed primitives; avoid accidental boxing and `NullPointerException`s.
- 50: Avoid strings when other types are more appropriate (use proper types for IDs, money, etc.).
- 51: Beware string concatenation performance in loops; use `StringBuilder`.
- 52: Refer to objects by their interface type.
- 53: Prefer interfaces to reflection; avoid reflective access in normal code paths.
- 54: Use native methods judiciously; avoid unless you need platform-specific features/perf.
- 55: Optimize judiciously (measure; design for performance without premature micro-optimizations).
- 56: Adhere to generally accepted naming conventions for readability and consistency.

### Exceptions (57-65)

- 57: Use exceptions only for exceptional conditions; do not use them for control flow.
- 58: Use checked exceptions for recoverable conditions; runtime exceptions for programming errors.
- 59: Avoid unnecessary checked exceptions; prefer returning optional results where appropriate.
- 60: Favor standard exceptions (e.g., `IllegalArgumentException`, `IllegalStateException`).
- 61: Throw exceptions appropriate to the abstraction; do not leak lower-level details.
- 62: Document all exceptions thrown by each method (especially in public APIs).
- 63: Include failure-capture information in detail messages for debugging.
- 64: Strive for failure atomicity (on failure, leave objects unchanged or in a valid state).
- 65: Do not ignore exceptions; handle, propagate, or explicitly justify suppression.

### Serialization (74-78)

- 74: Implement `Serializable` judiciously; it becomes part of your exported API.
- 75: Consider a custom serialized form; declare `serialVersionUID`; mark derived fields `transient`.
- 76: Write `readObject` defensively (defensive copies + invariant checks); avoid overridable calls.
- 77: For instance control, prefer `enum` singletons to `readResolve`; beware `readResolve` attacks.
- 78: Prefer serialization proxies when you must serialize complex invariants.

## Getting the full details quickly

Prefer opening the smallest relevant reference file (same original structure + code blocks):

- Items 1-7: `.codex/skills/effective-java-core/references/02-creating-and-destroying-objects.md`
- Items 8-12: `.codex/skills/effective-java-core/references/03-methods-common-to-all-objects.md`
- Items 13-22: `.codex/skills/effective-java-core/references/04-classes-and-interfaces.md`
- Items 23-29: `.codex/skills/effective-java-core/references/05-generics.md`
- Items 30-37: `.codex/skills/effective-java-core/references/06-enums-and-annotations.md`
- Items 38-44: `.codex/skills/effective-java-core/references/07-methods.md`
- Items 45-56: `.codex/skills/effective-java-core/references/08-general-programming.md`
- Items 57-65: `.codex/skills/effective-java-core/references/09-exceptions.md`
- Items 74-78: `.codex/skills/effective-java-core/references/11-serialization.md`

For navigation:

- Table of contents: `.codex/skills/effective-java-core/references/01-table-of-contents.md`

Legacy (large, avoid loading unless you truly need everything at once):

- `.codex/skills/effective-java-core/references/effective_java.md`

You can also search the repo source summary:

- File: `docs/effective_java.md`
- Find a specific item: `rg -n '^## 12\\.' docs/effective_java.md`
