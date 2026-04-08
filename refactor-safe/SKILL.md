---
name: refactor-safe
description: >
  A discipline for restructuring code you can't afford to break. Enforces the
  "behavior-preserving" definition of refactoring: characterization tests first,
  small atomic changes, and continuous verification. Use when touching code that
  has real users, unclear behavior, weak test coverage, or historical scar tissue.
  Distinguishes refactoring from rewriting and prevents the "while I'm here"
  scope creep that turns a 50-line diff into a 500-line minefield. Composes with
  ultra-efficient.
---

# Refactor Safe

You are refactoring. That word has a strict definition here: **changing the structure of code without changing its externally observable behavior.** If behavior changes, it's not a refactor. It's a rewrite, a feature change, or a bug fix — and it needs a different process.

## Before the first edit

### 1. Define the target

In one sentence: what structural improvement are you making, and why?
- "Extract this 200-line function into three smaller ones so it's readable"
- "Move the email sending out of the request handler so we can test it"
- "Consolidate the three copies of the validation logic into one"

If the sentence is "clean this up" or "make it better," stop. That's not a target — that's a mood. Get specific or don't refactor.

### 2. Define the contract

What is the observable behavior that MUST be preserved? Be concrete:
- Public function signatures
- API responses
- Database writes
- Emitted events
- Side effects (files written, messages queued, logs for alerts)
- Performance envelopes (if load-bearing)

Anything not in this list is fair game. Anything in this list must be identical before and after.

### 3. Characterization tests

This is the step people skip, and it's the step that saves you.

Before changing anything, pin down the current behavior with tests — even if the behavior is weird or wrong. These are **characterization tests**: they document what the code does today, not what it should do.

- Write a test that captures each behavior in the contract
- Include edge cases that the code handles today (even accidentally)
- If there are existing tests, run them and confirm they pass
- If coverage is thin, write more tests before touching the code

If the code is too entangled to test, that tells you something: the refactor's first job is to break the entanglement enough to add tests, then refactor for real.

## The refactoring loop

Each step should be small enough to hold in your head and small enough to revert.

```
1. Pick the smallest meaningful improvement
2. Make the change
3. Run the tests
4. If green: commit. If red: revert, not debug.
```

That last rule is the discipline. If a refactor breaks tests and you can't immediately see why, **revert and retry smaller**. Don't debug in the middle of a refactor — you're conflating two problems.

## Safe moves

These moves preserve behavior by construction when done carefully:

- **Extract function** — pull lines into a new function with the same inputs/outputs
- **Extract variable** — name a sub-expression
- **Inline** — the inverse of the above
- **Rename** — use tooling, not regex, to avoid collisions
- **Move** — relocate a function/class to a different file or module
- **Introduce parameter** — add an unused parameter first, then use it in a subsequent commit
- **Replace conditional with polymorphism** — when the condition is stable
- **Split loop** — if two concerns share a loop, separate them

Any single move that changes behavior is not a refactor — it's a change. Label it as such.

## Unsafe moves disguised as refactors

These change behavior and should NOT be called refactors:

- "Cleaning up" error handling (changes what errors propagate)
- "Simplifying" a condition (usually changes edge cases)
- "Removing dead code" (prove it's dead first — with a test, a log, or a metric)
- "Updating the dependency while I'm here"
- "Fixing the bug I noticed"
- "Improving the types" when the new types reject inputs the old types accepted

Each of these can be legitimate — just not under the refactor label. Do them in separate commits with their own justification.

## Commit discipline

- **One refactoring move per commit.** Extract function, commit. Rename, commit. Move file, commit.
- **Green at every commit.** Each commit must leave the tree in a working state.
- **Descriptive messages.** "refactor: extract `validateUser` from `createUser`" — not "refactor."
- **Never mix refactor and behavior change** in the same commit. It makes review impossible and bisect useless.

If you're doing a big refactor, the PR might have 20 commits. That's fine. Small commits are a feature, not noise.

## Scope discipline

The #1 way refactors go wrong is scope creep. Rules:

- **Draw the box before you start.** Write down what's in scope and what's not.
- **"While I'm here" is a trap.** Note the thing you want to fix, but don't fix it in this PR.
- **No opportunistic dependency updates.** Pinned deps for the duration of the refactor.
- **No style-only changes mixed in.** If the linter changes are separate, keep them separate.
- **No new features.** If you notice you're adding capability, you've drifted. Stop and split.

The larger the refactor, the stricter the scope discipline. Big refactors only survive if they're ruthlessly focused.

## Verification checklist

Before declaring a refactor done:

- [ ] All pre-existing tests pass
- [ ] New characterization tests pass
- [ ] No public API surface changes (or they're called out explicitly)
- [ ] Git diff reviewed for accidental behavior changes
- [ ] Run the app, hit the affected paths manually if tests are thin
- [ ] Benchmarks unchanged (if performance is part of the contract)
- [ ] Logs/metrics/events unchanged (unless explicitly scoped in)

## When a refactor reveals a bug

You'll find bugs while refactoring. Resist the urge to fix them in place.

1. Note the bug
2. Finish the refactor preserving the buggy behavior (yes, really)
3. Land the refactor
4. Fix the bug in a separate commit/PR where it's visible and reviewable

This sounds backwards but it's the only way to keep the refactor's diff reviewable and the bug fix's diff testable.

## Anti-patterns

- **Big bang rewrites** labeled as "refactors"
- **Refactor-and-feature sandwich**: mixing structural change with new behavior
- **Refactoring without tests**: hoping for the best
- **Fixing bugs inside a refactor**
- **Changing public APIs without a migration plan**
- **Abstracting for imagined future use cases**: YAGNI still applies

## Activation

When this skill activates, respond with:

🔧

Then start with the target sentence and the contract.
