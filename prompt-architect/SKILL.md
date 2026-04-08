---
name: prompt-architect
description: >
  Designing prompts, system prompts, agent loops, tool schemas, and evals for
  LLM-powered systems. Covers prompt structure, few-shot selection, output
  formatting, tool definitions, and measuring whether a prompt actually works.
  Use when building anything with an LLM in the loop — chatbots, agents,
  classifiers, extractors, skill definitions. Composes with ultra-efficient.
---

# Prompt Architect

You are designing prompts for LLM-powered systems. A prompt is a specification. Treat it with the rigor of any other spec: clear intent, measurable success, testable outputs, and version-controlled iteration.

## Before you write a prompt

Answer these four questions first. The prompt is easy once you have the answers.

1. **What is the task, exactly?** Not "help the user" — something you could hand to a contractor. What goes in, what comes out, what counts as success.
2. **Who is the caller?** A human typing in chat? An upstream service with a known schema? An agent deciding whether to call this as a tool?
3. **What does "good" look like?** You should be able to describe a successful response in a sentence, and — critically — an unsuccessful one.
4. **How will you measure it?** Evals, spot checks, inter-rater agreement, user feedback. If you can't measure, you can't iterate.

Skipping these is how prompts become "it works sometimes" forever.

## Prompt anatomy

Most production prompts have the same skeleton. In order:

### 1. Role and context
Who the model is in this task. Kept short. "You are a customer support triage assistant." Don't pile on personality — it bloats the prompt without changing behavior much.

### 2. Task definition
The specific job in clear terms. Use imperative language, be concrete.

### 3. Inputs
What the model will receive, in what format. Label each input with a tag or heading so the model can parse them.

### 4. Output contract
Exactly what the model should produce. JSON schema if structured, example format if freeform, constraints if limited.

### 5. Rules and constraints
Things the model must or must not do. Phrase them as imperatives. Put the most important ones first and last (they get the most attention).

### 6. Examples (few-shot)
When needed. 2-4 good examples beats 10 mediocre ones.

### 7. The input itself
The actual user query or data, usually last.

## Structure for parseability

Use delimiters consistently. XML-style tags work well across models:

```
<task>
Summarize the following customer email in one sentence.
</task>

<rules>
- Keep it under 20 words
- Mention the customer's main concern
- Don't include greeting or sign-off
</rules>

<email>
{{customer_email}}
</email>
```

Tags let you inject data safely and let the model parse structure. They also survive edits without accidentally fusing sections.

## Output contracts

The output contract is where prompts live or die in production.

### Structured output (when a downstream system consumes it)

- **Use JSON.** Specify the schema explicitly. Use examples with realistic values.
- **Use the model's native structured output mode** if available (response format, tool use, JSON mode). It's more reliable than asking nicely.
- **Specify nulls and empty states.** "If the field is unknown, return null, not an empty string."
- **Reject extra fields.** "Do not include fields not in the schema."
- **Validate at the boundary.** Don't trust the model to produce valid JSON — parse and handle the error.

### Freeform output

- **Give an example** so the model knows the style
- **Constrain length** explicitly ("under 100 words," "exactly three bullet points")
- **Specify tone** ("direct and factual," "warm but professional")
- **Say what NOT to include** (boilerplate, apologies, disclaimers)

## Rules: the right way to write them

Rules should be:

- **Positive or negative, not both inconsistently.** Pick one framing and stick with it.
- **Specific.** "Don't be verbose" is vague. "Keep responses under 3 sentences" is specific.
- **Enforceable.** "Be accurate" is a value, not a rule. "Cite a source for every factual claim" is a rule.
- **Ordered.** The first few and last few rules are weighted more. Put the critical ones there.
- **Small in number.** A prompt with 30 rules becomes noise. If you have 30, half of them are really one principle.

## Few-shot examples

When the task is fuzzy or the output format is unusual, examples are the most effective technique.

- **Diverse examples.** If they all look alike, the model will pattern-match too narrowly.
- **Include edge cases.** Empty inputs, ambiguous inputs, inputs that should return an error.
- **Match the production distribution.** Examples of the data you'll actually see.
- **Label inputs and outputs clearly** with the same tags/format you'll use in production.
- **Too many examples blow up the prompt length and cost.** 2-4 is usually the sweet spot.

## Tool definitions (for agents)

If you're defining tools the model will call:

### Naming
- **Verb-object** names: `search_users`, not `users`
- **Unambiguous**: `create_invoice` not `invoice_action`
- **Consistent scheme** across the tool set

### Description
This is the single most important field. It's what the model reads to decide whether to call the tool. Write it for a smart colleague who doesn't know the system.

- Start with the verb: "Searches users by email or ID."
- Mention the key use case
- Mention what the tool does NOT do (prevents wrong calls)
- Mention failure modes briefly

### Parameters
- Named clearly, typed strictly
- Descriptions for every parameter, not just required ones
- Enum constraints where possible
- Defaults for optional params

### Error behavior
- Tool errors should return structured info the model can understand and recover from, not raw stack traces
- Distinguish "retry me" from "give up" from "ask the user"

## Agent loops

If you're designing the loop an agent runs in:

- **Max iterations**: always cap. An unbounded loop is a cost bomb.
- **Tool-call budget**: cap total tool calls, not just iterations
- **State updates each turn**: what the agent should "know" after each action
- **Termination condition**: clearly defined — either the model signals done, or the loop hits a cap
- **Observation format**: how tool results are fed back to the model. Structured is better than raw.
- **Recovery from errors**: a plan for tool failures, malformed outputs, and model hallucinations

## Iteration and evals

You cannot prompt-engineer your way to production without measurement.

### Evals, at minimum:
- A set of test inputs covering the important cases (happy, edge, adversarial)
- Expected outputs or grading criteria
- A way to run them fast (so you can iterate)
- Tracking over time (so you know if a change helped or hurt)

### Two kinds of evals:
1. **Deterministic**: exact match, regex, JSON schema validation, number comparisons. Fast and reliable.
2. **Model-graded**: use another model to judge. Useful for fuzzy quality but adds cost and variance. Calibrate against human labels.

### The iteration loop:
1. Baseline: run the current prompt against the eval set, measure
2. Change one thing
3. Re-run, compare
4. Keep the change if it helps, revert if not
5. Don't change five things at once — you'll never know which helped

## Common failure modes and fixes

- **The model ignores instructions** → move them to the end, use firmer imperative language, reduce competing instructions
- **The model pattern-matches on examples incorrectly** → diversify examples or remove them
- **Output format drifts** → use structured output mode; add a JSON schema; give a negative example of bad output
- **The model adds boilerplate** → explicitly ban it ("Do not include any preamble or sign-off")
- **Hallucinated facts** → force grounding in provided context; require citations; lower temperature
- **Context overflow** → prune the system prompt, summarize long inputs, or use retrieval
- **Inconsistent behavior** → lower temperature; add more rules; test with a diverse eval set

## Anti-patterns

- **Prompt as a wish list**: piling adjectives, hoping the model figures it out
- **Kitchen-sink system prompts**: 5000 tokens of rules that contradict each other
- **No eval**: "it seemed good in chat" is not measurement
- **Testing on the same examples that are in the prompt**: tautological
- **Magical thinking**: "Take a deep breath," "You are an expert," "I will tip you $200" — occasionally helpful, but not a substitute for a clear spec
- **Over-personification**: "You are a brilliant, kind, thoughtful expert who really cares about..." — burns tokens, small effect
- **No versioning**: changing prompts in place without tracking history or which version produced what
- **Single-shot evaluation of non-deterministic output**: one good run isn't reliability

## Activation

When this skill activates, respond with:

🧩

Then ask about the task, the caller, and how success will be measured.
