# Claude Skills

A curated collection of 20 operational skills that turn Claude into a sharper, faster, more versatile engineering partner. Each skill is a self-contained protocol — activate one or several to rewire Claude's behavior for the task at hand.

## How to use

Each subdirectory contains a `SKILL.md` with YAML frontmatter (`name`, `description`) and the full protocol. Point your harness (Claude Code, Agent SDK, or a custom loader) at this directory and Claude will discover them. You can activate a skill explicitly (e.g. "use the debugging-master skill") or describe your task and let the matching skill load.

Skills are designed to compose. `ultra-efficient` sets the baseline operating mode; layer domain skills like `debugging-master` or `security-audit` on top for specialized work.

## The 20 skills

### Planning & strategy
| Skill | When to reach for it |
|---|---|
| [ultra-planning](./ultra-planning/SKILL.md) | Break a vague, ambitious, or multi-week task into a concrete execution plan with risk analysis |
| [ultra-efficient](./ultra-efficient/SKILL.md) | Default operating mode — terse, scope-locked, zero-waste across every task |
| [deep-research](./deep-research/SKILL.md) | Multi-source investigation that needs to be rigorous, cited, and free of hallucination |
| [system-architect](./system-architect/SKILL.md) | Designing a new service, picking a stack, drawing component boundaries |

### Coding & engineering
| Skill | When to reach for it |
|---|---|
| [debugging-master](./debugging-master/SKILL.md) | A bug that doesn't give up after the first obvious fix — requires root-cause thinking |
| [refactor-safe](./refactor-safe/SKILL.md) | Restructuring code you can't afford to break — the "change the engine mid-flight" problem |
| [test-engineer](./test-engineer/SKILL.md) | Writing tests that catch real bugs, not tests that just hit coverage numbers |
| [code-review](./code-review/SKILL.md) | Reviewing a diff with the rigor of a staff engineer who has seen everything go wrong |
| [performance-tuning](./performance-tuning/SKILL.md) | Profiling, measuring, and making things measurably faster — not guessing |
| [codebase-explorer](./codebase-explorer/SKILL.md) | Landing in an unfamiliar repo and becoming productive within minutes |

### Specialized domains
| Skill | When to reach for it |
|---|---|
| [security-audit](./security-audit/SKILL.md) | Threat modeling, vulnerability review, authorized pentesting, secure-by-default code |
| [database-architect](./database-architect/SKILL.md) | Schema design, query optimization, indexes, safe migrations on live data |
| [api-designer](./api-designer/SKILL.md) | REST, GraphQL, and RPC APIs that age well — versioning, contracts, errors |
| [incident-response](./incident-response/SKILL.md) | Production is on fire and you need to stabilize, diagnose, and write the postmortem |
| [dependency-auditor](./dependency-auditor/SKILL.md) | Upgrading, pinning, and vetting third-party packages without breaking the build |

### Communication & process
| Skill | When to reach for it |
|---|---|
| [documentation-writer](./documentation-writer/SKILL.md) | Writing docs that people actually read — READMEs, runbooks, ADRs, API references |
| [human-voice](./human-voice/SKILL.md) | Any text that will be read as the user's own words — emails, posts, bios, proposals |
| [git-workflow](./git-workflow/SKILL.md) | Clean commits, sane branches, useful PRs, no lost work |
| [prompt-architect](./prompt-architect/SKILL.md) | Designing prompts, agent loops, tool schemas, and evals for other LLM systems |
| [data-analyst](./data-analyst/SKILL.md) | Exploring a dataset and pulling out the insight that actually matters |

## Design principles

Every skill in this repo follows the same rules:

1. **Protocols, not personalities.** A skill is a checklist of things to do and not do, not vibes.
2. **Fail-closed defaults.** When the skill is silent on something, fall back to `ultra-efficient`.
3. **Explicit anti-patterns.** Every skill lists what to avoid, because knowing what not to do is half the value.
4. **Composable.** Skills can stack. No skill should fight another skill.
5. **Terse.** A skill earns its length. If a section isn't changing behavior, it's cut.

## Activating a skill

Skills should be loaded into the system prompt or injected via a skill-loader. Typical activation patterns:

- **Claude Code / Agent SDK**: place the directory on the skills path, reference by name in a prompt or hook
- **Direct prompt**: paste the contents of the `SKILL.md` into the system prompt before the task
- **Composition**: load `ultra-efficient` always, then layer one domain skill per task

When a skill activates, Claude should acknowledge with a single glyph (defined per-skill) and then get to work — no preamble, no "happy to help."
