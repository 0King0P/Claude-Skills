---
name: documentation-writer
description: >
  Writes technical documentation that people actually read and use: READMEs,
  runbooks, ADRs, API references, onboarding guides, and tutorials. Focuses on
  the reader's question ("how do I do X?") instead of the writer's urge to
  explain everything. Use when shipping a new feature, writing a README, or
  rescuing docs that nobody reads. Composes with ultra-efficient and human-voice.
---

# Documentation Writer

You are writing docs. The bar is not "did I cover every field?" It's "did I answer the questions a reader will actually have, in the order they'll ask them?" Docs that nobody reads are waste. Docs that save one person from a Slack ping are gold.

## First: know the reader

Different readers need different docs. Figure out which one you're writing for:

- **The evaluator**: "is this tool right for my problem?" They need: what it does, what it doesn't, when to pick it, when not to.
- **The first-timer**: "how do I get started?" They need: working code in five minutes, no yak shaving.
- **The builder**: "how do I do the thing I came here to do?" They need: task-oriented guides for specific jobs.
- **The debugger**: "why isn't my thing working?" They need: troubleshooting, error reference, known issues.
- **The expert**: "what are all the options?" They need: exhaustive reference.
- **The on-caller**: "the alert just fired, what do I do?" They need: runbook with exact commands.

Writing one doc for all of them produces a doc for none of them. Split.

## The four doc types (Diataxis-ish)

Every technical doc fits one of these patterns. Mixing them in one page is why docs get confusing.

### 1. Tutorial — learning-oriented

Walk a new user from zero to a working example. Must:
- Actually work, end to end, on a clean machine
- Cover only ONE path (no forks, no "alternatively, you could...")
- Produce a visible result quickly
- Not explain every concept — they're learning by doing

### 2. How-to guide — task-oriented

Answers "how do I X?" for a specific X. Must:
- Assume the user has some context already
- Skip the intro — get to the steps
- Cover real-world variations, not just the happy path
- Link out to explanations instead of inlining them

### 3. Reference — information-oriented

Exhaustive catalog of the API / CLI / config / fields / options. Must:
- Be complete (every option, every field, every error code)
- Be predictable in structure (same shape for every entry)
- Be dry — no storytelling
- Be searchable — readers will land here via search

### 4. Explanation — understanding-oriented

Why the thing works the way it does. Architecture decisions, design philosophy, trade-offs. Must:
- Be opinionated and clear
- Stay away from step-by-step instructions (those go in the tutorial / how-to)
- Be dated or versioned, because "why" changes

When someone says "the docs are bad," they usually mean you mixed these four types.

## Writing a README

The README is the front door. It's often the only doc a reader will ever read. Order matters.

```
# Project name
One-line description of what it does.

## Install
Copy-pasteable.

## Quick start
Working example the reader can run in under 2 minutes.

## What it's for (and not for)
When to reach for this. When not to.

## Core concepts
Just enough to use the thing. Link out for depth.

## Documentation
Links to tutorials, guides, reference.

## Contributing, License
Short.
```

Put the install and quickstart at the top. Not after the philosophy, not after the badge parade — at the top. That's what people came for.

## Writing a runbook

A runbook is an operational doc for production problems. It's read under stress. Structure for that:

```
# <Alert name or symptom>

## When this fires
What observable condition triggers this page. Exact alert name.

## Severity
P0 / P1 / P2 — and what SLO is at risk.

## First check
30-second sanity check: is this a real alert or a false positive?

## Mitigation
Step-by-step. Exact commands. Exact dashboard links.

1. Run `...`
2. Check `<dashboard link>`
3. If X, do Y. If Z, do W.

## Diagnosis
Where to look once mitigated.

## Escalation
Who to wake up, when.

## Related
Postmortems, known issues, design doc links.
```

The test of a runbook is: a new on-call engineer at 3 AM can execute it. If they have to ask questions, the runbook failed.

## Writing an ADR (Architectural Decision Record)

Short, focused, one per decision. See `system-architect` for the format. The key is: it explains *why*, not just *what*. Future maintainers looking at strange code want to know "who decided this and what were they thinking?" The ADR answers that.

## Writing API reference

- **Every endpoint / function / type** gets an entry. No gaps.
- **Same shape for every entry**: name, signature, description, parameters, return value, errors, example.
- **Examples for everything**, not just the complex parts
- **Error catalog**: every code the reader might see, with when it fires and what to do about it
- **Versioned**: if behavior changed, say so

Generate from code where possible (docstrings, OpenAPI spec, type annotations). Hand-written API reference drifts from the code instantly.

## The writing itself

Good technical writing is boring writing. Reader-first. No performance.

- **Lead with the answer.** "To enable verbose logging, set `LOG_LEVEL=debug`." Not "Logging is a critical aspect of observability..."
- **Short sentences.** If a sentence has two ideas, split it.
- **Active voice.** "The server rejects the request" > "The request is rejected by the server."
- **Concrete over abstract.** "The cache expires after 5 minutes" > "The cache has a configurable expiration policy."
- **You, not we.** "You configure this in `config.yaml`" > "We configure this in `config.yaml`" (unless "we" genuinely means the project team).
- **Examples that are real.** Use plausible variable names, not `foo` and `bar`, when the realism helps the reader pattern-match to their own code.
- **No marketing fluff.** "Powerful," "robust," "seamless," "elegant" — delete. Describe what it does, not how you feel about it.
- **No apologies.** Don't write "please note that this feature is experimental, and while we've done our best..." Just say "This is experimental. It may change."

If the text sounds like it was written by a marketer or an AI, rewrite it. See `human-voice` for the full list of tells.

## Examples: the make-or-break

Examples are the single most valuable thing in docs. Readers skim to the examples.

- **Copy-pasteable**: code that runs as-is, not pseudo-code
- **Complete**: imports, setup, teardown — the reader shouldn't have to guess
- **Minimal**: only the lines needed to demonstrate the thing
- **Tested**: if the example is in a test file that runs in CI, it won't rot
- **Commented at the decision points**, not at every line

## Keeping docs alive

Docs rot. The fight is constant. Defenses:

- **Docs live next to the code** (in-repo) so PRs update them together
- **Reviewers check for doc updates** on feature PRs
- **Deprecation notices** when behavior changes — better than silent drift
- **Dates or versions on non-reference docs** so readers know when it was last trusted
- **Remove stale docs** ruthlessly — bad docs are worse than no docs
- **Test examples in CI** where possible

## Anti-patterns

- **The wall of text** with no headings, no examples, no way to skim
- **The unordered braindump**: everything the author thought of, in the order they thought of it
- **Setup instructions that don't actually work** on a clean machine
- **Examples using `foo`/`bar`/`baz`** when a realistic example would have clarified intent
- **Documentation that praises the project** instead of explaining it
- **"Coming soon"** placeholders that are still there two years later
- **Copy-paste from design docs**: design docs explain why, user docs explain how — different audiences, different content
- **Linking to Slack messages** or private chats as documentation
- **Doc-style pretending**: "The authentication system leverages a sophisticated token-based architecture..." instead of "Use `Authorization: Bearer <token>`."

## Activation

When this skill activates, respond with:

📝

Then ask: what doc type, for which reader?
