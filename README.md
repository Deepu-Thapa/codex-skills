# The set of Codex skills based on Effective Java (2nd ed.).

Original content: https://github.com/HugoMatilla/Effective-JAVA-Summary

# Using the codex‑skills repository effectively

The **codex‑skills** repository contains modular skills that teach best practices for programming in Java.  Each skill is self‑contained and designed to be pulled into ChatGPT or Codex when needed.  This guide explains how the repository is organised, how to load skills, and how to write prompts that leverage both high‑level checklists and deeper reference material.

## Repository structure

- **skills/** – Each folder under `skills` holds one skill.  For example, `effective‑java‑core` and `effective‑java‑concurrency` are two skills extracted from *Effective Java*.
- **SKILL.md** – The main file in each skill.  It includes a YAML header with metadata and a concise checklist of the key items to remember.  This is the high‑level guidance that is automatically pulled into the context when you reference the skill.
- **references/** – A set of Markdown chapters that contain detailed explanations and code samples.  Each numbered file corresponds to a chapter or section of the book.  You can load these files selectively to get more depth without overloading your context.

### Paths and namespaces

When you install the skills locally they live under `.codex/skills/…`.  On GitHub they appear under `skills/…`.  The internal links in the `SKILL.md` files (for example `.codex/skills/effective‑java‑core/references/05‑generics.md`) will work correctly in a local setup.  If you browse the repository on GitHub, remember to drop the `.codex` prefix.

## Loading skills in prompts

To use a skill in a prompt, prepend its name with a `$`.  For example:

```
using $effective‑java‑core implement a value class for complex numbers
```

This instructs Codex to load the `effective‑java‑core` skill and apply its high‑level checklist.  The checklist covers core topics like minimising mutability, implementing `equals`/`hashCode`, and preferring builders for many constructor parameters [see](skills/effective-java-core/references/02-creating-and-destroying-objects.md#2-use-builders-when-faced-with-many-constructors). Only the checklist is loaded by default, so the context remains small.

If your task needs more detail, you can explicitly open one or more reference chapters.  For example:

```
using $effective‑java‑core and .codex/skills/effective‑java‑core/references/07‑methods.md,
refactor this Period class to be immutable
```

This pulls in the deep explanation of defensive copying from item 39 [see](skills/effective-java-core/references/07-methods.md#39-make-defensive-copies-when-needed) and shows how to fix a mutable `Period` implementation.

### Don’t overload the context

The skills are intentionally split into chapters because loading the entire book at once would exceed the context window.  The skill docs themselves advise you to “prefer opening the smallest relevant reference file [see](skills/effective-java-core/SKILL.md#getting-the-full-details-quickly)). Identify the chapters that apply to your problem and load only those.

## Examples of effective prompts

The following examples demonstrate how to combine skills with reference chapters to accomplish realistic tasks.

### 1. Immutable reservation period

**Prompt**

```
Using $effective‑java‑core and .codex/skills/effective‑java‑core/references/07‑methods.md,
design an immutable ReservationPeriod class that takes two Date parameters,
checks that the end does not precede the start, makes defensive copies,
and returns defensive copies from accessors.  Explain why defensive copies are necessary.
```

**What it does** – The `07‑methods.md` file explains why simply storing references to mutable `Date` objects is dangerous and shows how to make copies in the constructor and accessors [see](/skills/effective-java-core/references/07-methods.md#39-make-defensive-copies-when-needed). By loading this chapter, Codex knows to copy the incoming dates, validate them, and return new `Date` instances in `getStart()` and `getEnd()`.

### 2. Building a clean deck of cards

**Prompt**

```
Using $effective‑java‑core and .codex/skills/effective‑java‑core/references/08‑general‑programming.md,
write a method List<Card> newDeck() that constructs a 52‑card deck from Suit
and Rank enums.  Show the bug that occurs when advancing the outer iterator in the inner loop, then fix it using a nested for‑each loop.
```

**What it does** – Chapter 8 warns that using iterators incorrectly in nested loops can produce unexpected results and shows a bug in a card‑deck example [see](skills/effective-java-core/references/08-general-programming.md#46-prefer-for-each-loops-to-traditional-for-loops). It then demonstrates how a pair of nested for‑each loops avoids the problem [see](skills/effective-java-core/references/08-general-programming.md#46-prefer-for-each-loops-to-traditional-for-loops). Codex will reproduce the bug and fix using the preferred idiom.

### 3. Money calculations

**Prompt**

```
Using $effective‑java‑core and .codex/skills/effective‑java‑core/references/08‑general‑programming.md,
implement a Money class that stores euros and cents without using float or double.
Provide add and subtract methods and explain why int/long or BigDecimal are used
instead of floating‑point types.
```

**What it does** – Item 48 notes that floating‑point types are unsuitable when exact answers are required, recommending integer types or `BigDecimal` for monetary amounts. By loading this chapter, Codex chooses an exact numeric representation and documents the reasoning.

### 4. Counting set operations via composition

**Prompt**

```
Using $effective‑java‑core and skills/effective-java-core/references/04-classes-and-interfaces.md#16-favor-composition-over-inheritance,
write an InstrumentedHashSet<E> that counts how many elements have been added.
Use composition (a private HashSet field) instead of extending HashSet.  Provide
 a getAddCount() method and briefly explain the choice of composition.
```

**What it does** – The classes‑and‑interfaces chapter emphasises favouring composition over inheritance and provides an `InstrumentedSet` example. Codex will wrap a `HashSet` inside a forwarding class, increment a counter in `add()` and `addAll()`, and return the count via `getAddCount()`.  It will also explain that composition avoids the fragility of subclassing.

### 5. Refactoring a tagged class into a hierarchy

**Prompt**

```
Using $effective‑java‑core and skills/effective-java-core/references/04-classes-and-interfaces.md#20-prefer-class-hierarchies-to-tagged-classes,
refactor a tagged Figure class (which uses a Shape enum and stores fields for rectangles and circles) into a class hierarchy with Figure, Circle and Rectangle.
Include Square as a subclass of Rectangle.  Show how the area() method becomes simpler and safer.
```

**What it does** – Item 20 illustrates why tagged classes are verbose and error‑prone and shows how to replace them with a hierarchy of abstract and concrete classes. With this reference loaded, Codex will create an abstract `Figure` class with an abstract `area()` method and provide concrete `Circle`, `Rectangle` and `Square` implementations, moving the logic into the appropriate subclasses.

### 6. Concurrency patterns

**Prompt**

```
Using $effective‑java‑concurrency and .codex/skills/effective‑java‑concurrency/references/10‑concurrency.md,
write a TaskProcessor class that submits Runnable tasks to an ExecutorService,
processes them using a fixed thread pool, and shuts down the executor gracefully.
Explain why executors and concurrency utilities are preferred to manually creating threads.
```

**What it does** – The concurrency skill encourages preferring executors and tasks over manual thread management and suggests using concurrency utilities such as `ExecutorService` and `BlockingQueue`. The reference chapter provides examples of creating and shutting down a thread pool; Codex will mirror this pattern and describe the benefits.

## Additional tips

- **Be explicit about references.** Codex does not automatically load individual items based on your intent.  If you need the details of a particular chapter, include its path in your prompt.
- **Target specific guidelines.** Instead of loading many chapters at once, identify the themes (immutability, generics, concurrency, etc.) relevant to your code and load those sections.
- **Provide the code to review.** When asking Codex to refactor or review existing code, paste the code or ensure it is available in the context.  Without the code, the model cannot analyse it.
- **Leverage enumerated items.** The `SKILL.md` files list each item in order (e.g. Items 1‑22 on creating and destroying objects).  Use these numbers to pinpoint the guidance you need.
