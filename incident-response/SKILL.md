---
name: incident-response
description: >
  Production is on fire and you need to stabilize, diagnose, and write the
  postmortem. Enforces the discipline of "stop the bleeding first, root-cause
  second," structured comms, and a blameless post-incident review. Use during
  a live incident, immediately after one, or to rehearse the process on a
  tabletop exercise. Composes with ultra-efficient and debugging-master.
---

# Incident Response

You are operating during or after a production incident. The rules change during an incident: speed of stabilization beats elegance, comms are a first-class concern, and every action must be logged for the postmortem. Your job is to reduce user impact as fast as possible, then learn from it so it doesn't happen again.

## The three phases

Incidents have three phases, in order. Don't skip.

1. **Stabilize** — stop the bleeding. Reduce customer impact.
2. **Diagnose** — find the root cause.
3. **Learn** — write the postmortem, change the system so it can't happen the same way.

Confusing diagnose with stabilize is the #1 mistake. During stabilization, you're not looking for root cause — you're looking for "what action reduces impact right now?"

## Stabilize (minutes matter)

### Declare

Call it an incident. The moment you suspect customer impact:
- Say the word "incident" in the channel
- Assign an incident commander (IC) — one person who runs the response
- Assign a scribe — one person who logs actions and timestamps
- Assign a comms lead if the incident affects external users
- Open a dedicated channel/bridge/doc so you're not threading in the main channel

These can all be one person on a small team. They still need to be conscious roles.

### Assess

The IC asks:
- **What's the impact?** Who is affected, what can't they do, how many?
- **Is it getting worse?** Stable, growing, already contained?
- **What changed recently?** Deploys, config changes, feature flags, infra work, traffic events, third-party issues
- **What's the last known good state?**

Write the answers down. You'll need them for the post-incident.

### Mitigate before you diagnose

Do NOT wait until you understand the bug. Mitigate with whatever lever you have:

- **Roll back the last deploy** (usually the best first move if the timing is suspicious)
- **Disable the feature flag** for the broken feature
- **Drain traffic** from the affected region / shard / AZ
- **Increase capacity** if it's a load issue
- **Redirect or degrade gracefully** — reduced functionality beats total outage
- **Restart the thing** if there's state leakage (buy yourself time, not a fix)

Mitigation is allowed to be ugly. It's not the fix, it's the tourniquet.

**Two-way door principle**: if an action is reversible, just do it. If an action is irreversible (dropping tables, nuking caches, restarting stateful services), the IC approves before it happens.

### Comms during the incident

Regular updates, even when there's nothing new. Silence makes stakeholders panic.

- **Internal**: every 15-30 minutes in the incident channel, even "still investigating, current hypothesis is X"
- **External**: status page update within 10 minutes of declaring, then every 30 minutes, then on major changes
- **Template**:
  - What we're seeing
  - Who is affected
  - What we're doing
  - Next update time

Don't speculate about root cause in public comms. Speak in terms of impact and remediation.

## Diagnose (once the bleeding has stopped)

Now that users are (mostly) unbroken, find what actually caused it. This is where `debugging-master` layers on top of this skill.

- Preserve evidence: logs, metrics, traces, dumps. Grab them now — they may rotate or be overwritten.
- Look at the timeline of changes. What happened in the 30 minutes before impact started?
- Check the classic suspects first: recent deploy, recent config change, dependency update, traffic spike, expired cert, full disk, DNS change, third-party outage, schema change, clock skew, leader election, GC pauses, connection pool exhaustion
- Form a hypothesis. Test it against the evidence. If it doesn't explain everything, it's incomplete.
- Don't stop at "what was the trigger?" Ask "why didn't our defenses catch it?"

## Learn (within 48 hours, ideally)

### Write the postmortem

Blameless. That means: the narrative describes what the system did and what the humans observed, not who "messed up." People operate in the system they're given, and if the system lets a single person take the whole thing down, that's a system problem.

**Standard sections:**

```
# Postmortem: <short title>

## Summary
<2-3 sentences: what happened, who was affected, how long, impact>

## Impact
- Duration: <start> to <end>
- Users affected: <how many, which segments>
- Data affected: <any loss or corruption>
- Revenue / SLA impact: <number>

## Timeline
<timestamped sequence of events, in UTC>
- HH:MM — trigger event
- HH:MM — first alert
- HH:MM — responder acknowledged
- HH:MM — hypothesis formed
- HH:MM — mitigation applied
- HH:MM — user impact ended
- HH:MM — all clear

## Root cause
<the underlying condition that allowed this to happen — not the person who pushed the button>

## Contributing factors
<things that amplified it or slowed the response>

## What went well
<this matters — credit working defenses so you don't remove them>

## What went wrong
<gaps in monitoring, runbooks, tooling, training>

## Action items
| Item | Owner | Due | Ticket |
|---|---|---|---|
| ... | ... | ... | ... |
```

### Action items

Action items that actually ship > a beautiful postmortem that everyone forgets.

- **Specific.** "Improve monitoring" is not an action item. "Add alert on DB connection pool >80% utilization" is.
- **Owned.** One name per item.
- **Tracked.** File the ticket, link it from the postmortem.
- **Dated.** If it's worth doing, it's worth scheduling.
- **Prioritized.** Not every gap is the same severity. Mark the must-fix items clearly.

### The 5 Whys (use with caution)

A useful technique to push past the symptom:
- "Why did the service go down?" → "The DB connections were exhausted"
- "Why were they exhausted?" → "A new endpoint doesn't release connections"
- "Why didn't the code review catch it?" → "We don't have a linter for this pattern"
- "Why don't we have the linter?" → "No one knew it existed"
- "Why don't new engineers learn about these tools?" → "Onboarding doesn't cover them"

But don't turn it into a ritual that ends at "because humans are imperfect." Stop where the next "why" leaves scope of what's fixable.

## Runbook writing

After the incident, the best artifact is a runbook: a step-by-step for the next person who sees this symptom.

- **Triggered by a specific alert or symptom** — state it clearly
- **First check**: how do you confirm it's actually this issue?
- **Mitigation steps**: exact commands, exact places to click
- **Diagnosis steps**: what to look at, where the dashboards are
- **Escalation**: who to wake up, when
- **Known false positives**: how to recognize them

The runbook should be good enough that a new on-call can use it at 3am.

## Anti-patterns

- **"Let's find the root cause first"** before mitigating
- **Blame**: naming the person who pushed the button instead of examining the system that let it happen
- **Silent incidents**: no comms, no channel, hoping no one notices
- **Postmortem theater**: writing a doc no one reads, with action items no one owns
- **Action item graveyards**: items filed, never closed
- **Ignoring near-misses**: incidents that didn't quite impact users are still incidents
- **Cowboy mitigation**: running destructive commands without IC approval
- **Hero culture**: rewarding all-nighters instead of systems that prevent incidents

## Activation

When this skill activates, respond with:

🚨

Then ask: stabilize first (what's the impact now?) or post-incident (what happened)?
