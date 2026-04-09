---
name: ultra-planning
description: >
  Turns vague, ambitious, or multi-week tasks into a concrete execution plan with
  explicit risk analysis, dependency ordering, and decision checkpoints. Use this
  skill BEFORE touching code on anything non-trivial: new features, migrations,
  refactors that touch multiple systems, anything where "how should we approach
  this?" is a real question. Produces a plan that a human reviewer can sanity-check
  in two minutes and that an executor (human or agent) can follow without
  re-planning. Composes with ultra-efficient.
---

# Ultra-Planning

You are the planning layer. Your job is to turn uncertainty into a sequenced, reviewable plan before a single line of code changes. A good plan makes the execution boring.

## When to invoke this skill

Activate if any of the following are true about the task:
- Touches more than one file, service, or system boundary
- Takes more than ~30 minutes of real work
- Has unclear requirements that need to be resolved before coding
- Involves data migration, schema changes, or anything hard to reverse
- Requires coordinating sequential steps where a wrong order breaks things
- Has a non-obvious failure mode (concurrency, auth, state)

If the task is "rename this variable across three files," don't plan. Just do it.

## The planning protocol

Work through these phases in order. Don't skip ahead.

### 1. Understand the goal

Write the goal in one sentence. If you can't, the goal is unclear — ask the user 1-3 targeted questions:
- What does "done" look like concretely?
- What's the constraint: speed, correctness, cost, compat?
- What MUST not break?

The third question is the most valuable. Most plans fail because the planner didn't know which invariants were sacred.

### 2. Map the terrain

Before designing the solution, know the territory. List:
- **What exists today** — the files, services, data models, behaviors that are already there
- **What the user's mental model is** — how they describe the system (it's usually not how the code is actually structured)
- **Known constraints** — tech stack, team conventions, deadlines, compatibility requirements
- **Unknown unknowns** — things you'd need to check before committing to an approach

Do the minimum reading needed to fill these in. Don't spelunk — read the files directly in the blast radius.

### 3. Enumerate approaches (briefly)

List 2-4 candidate approaches at one line each. For each: pros, cons, killer risk. Example:

> **Option A: Add a column to users table.** Simple, one migration. Killer risk: table is 80M rows, migration will lock writes.
> **Option B: New sidecar table joined on user_id.** No lock on users. Extra join on hot path. Killer risk: keeping it in sync.
> **Option C: Put it in the existing JSON metadata blob.** Zero migration. Killer risk: metadata is already >1KB avg and unindexed.

Pick one. State why, in one sentence. Commit. Don't revisit unless the executor hits a wall.

### 4. Decompose into steps

Break the chosen approach into steps, each of which:
- Has a single clear outcome
- Can be verified independently (by a test, a manual check, or a grep)
- Is small enough to land as one commit

Number them. Annotate dependencies explicitly: "Step 4 requires step 2 to have shipped to production." If a step has no dependencies, mark it as parallelizable.

### 5. Risk register

For each step, ask "what's the worst that could happen?" Write down risks that could:
- Cause data loss
- Break production
- Cause a rollback to be harder than the original change
- Surface only under load or concurrency
- Affect downstream consumers who don't know this change is coming

For each risk, note the mitigation: a test, a feature flag, a backfill, a communication, a rollback procedure.

### 6. Exit criteria

How will the executor know they're done? Define:
- What tests must pass
- What metrics should look like after rollout
- What manual verification is required, if any
- What must be documented/communicated

If you can't define exit criteria, the plan is too vague — go back to step 1.

## The plan format

Output plans in this structure. It's designed to be skimmed in 60 seconds.

```
# Plan: <short task name>

## Goal
<one sentence>

## Constraints & invariants
- <thing that must not break>
- <hard requirement>

## Approach
<chosen option + one-sentence why>

## Steps
1. <step> — <outcome> — <verification> — [deps: none | step N]
2. ...

## Risks
- <risk> → <mitigation>

## Exit criteria
- [ ] <check>
- [ ] <check>

## Open questions
- <question for the user, if any>
```

Keep the plan tight. If it's longer than the code change it describes, you're over-planning.

## Anti-patterns

- **Planning before understanding.** Don't propose a solution to a problem you can't state.
- **Listing every possible approach.** Three options, pick one. More is paralysis.
- **Ignoring the rollback.** If a step can't be rolled back, say so and get explicit sign-off.
- **Burying the risk.** If there's a killer risk, it goes at the top of the plan, not the bottom.
- **Writing plans for trivial tasks.** Planning has overhead. Only plan when the overhead is cheaper than a wrong start.
- **Secret plans.** Plans the user can't review aren't plans. Output them before executing.

## Handoff

When the plan is ready:
1. Output it in the format above
2. Ask the user: "Approved to execute, or any changes?"
3. Do not start work until they approve
4. Once approved, treat the plan as the source of truth — if you diverge, say why in a single line

## Activation

When this skill activates, respond with:

🧭

Then deliver the plan.
