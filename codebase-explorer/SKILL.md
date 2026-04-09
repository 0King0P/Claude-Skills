---
name: codebase-explorer
description: >
  A protocol for landing in an unfamiliar repository and becoming productive
  within minutes, not days. Teaches how to map a codebase fast, identify the
  load-bearing modules, understand the conventions, and find the right place
  to make a change — without reading every file. Use when you're dropped into
  a new project, joining a codebase mid-stream, or investigating a question in
  code you didn't write. Composes with ultra-efficient.
---

# Codebase Explorer

You are new to this code. Your job is to become useful fast without pretending to understand more than you do. You are NOT here to re-read the whole repo.

## The orientation pass (first 5 minutes)

Do these in order. They're cheap and they prune your search space.

### 1. Top-level layout

Look at the repo root. You're trying to answer:
- What language(s) and major frameworks?
- Is this a monorepo? If so, how is it split?
- Where does code live vs tests vs docs vs config vs build?
- What's the entry point?

Files to check: `README.md`, `package.json` / `pyproject.toml` / `go.mod` / `Cargo.toml` / `pom.xml`, top-level directory names, `Makefile` / `justfile` / `Taskfile`.

### 2. The docs that actually matter

Most docs are stale or aspirational. The ones worth reading:
- `README.md` — usually at least tells you the project's purpose
- `CONTRIBUTING.md` — conventions, required tooling
- `ARCHITECTURE.md` / `docs/architecture.md` — if it exists, read it
- `ADR/` or `docs/decisions/` — why things are the way they are
- `CHANGELOG.md` — what's changed recently, direction of travel

Skip: auto-generated API docs, marketing material, blog posts linked from the README.

### 3. Build and run

If you can't build and run it, you can't test your changes. Before digging in:
- Find the build command
- Find the test command
- Find how to run a local instance (if applicable)
- Actually run the tests to confirm they pass on the current HEAD

If any of these fail, that's diagnostic. Maybe the branch is broken, maybe there are undocumented prerequisites. Ask the user rather than guessing.

### 4. Entry points

Every app has entry points. Find them:
- `main.go`, `main.py`, `index.ts`, `app.py`, `Program.cs`
- Server bootstrap files
- CLI definitions
- Lambda / function handlers
- Test entry points

Skim the top of each entry point to understand what gets wired up at startup. The wiring tells you the top-level architecture.

## The map, not the territory

You don't need to read every file. You need a mental map with three kinds of nodes:

1. **Core domain**: where the business logic lives
2. **Adapters**: code that talks to the outside world (DB, HTTP, queues, filesystem, third parties)
3. **Glue**: config, wiring, scaffolding, utilities

Most bugs and features live in core domain or adapters. Glue is noise unless the bug is in the wiring.

For each area, note the directory/package and one sentence about what it does. That's your map. It should fit on a napkin.

## The grep playbook

Precision beats breadth. Good searches look like this:

- **Function names you know from the problem**: `grep "createUser"` → find the implementation and all call sites
- **Error messages**: if the bug produces a message, grep the literal string
- **Config keys**: if you suspect a config is involved, grep the key name
- **Route definitions**: grep the URL path
- **Database table or column names**: finds the ORM code
- **TODO / FIXME / XXX**: tells you where the known weak spots are
- **Dates in comments**: finds code that's been untouched for years (either stable or abandoned)

Don't grep broadly. Don't "just read everything in the auth module." Search for the specific thing you need.

## Task-driven exploration

Exploration without a task is a time sink. Always have a question:

- "Where does a request to `/users` get handled?"
- "How does the session token get validated?"
- "What happens when the payment webhook fires?"
- "Where is the `email_verified` column read?"

For each question, find the answer with the minimum reading:
1. Grep for the obvious keyword
2. Follow one or two hops through the call graph
3. Stop the moment you have the answer

Don't "understand everything first." That's impossible and wasteful. Understand the thing you need to touch, then expand outward as needed.

## Reading code efficiently

When you do read a file:

- **Read the top first** — imports and top-level declarations tell you the file's vocabulary
- **Read signatures before bodies** — know what each function takes and returns, then decide which bodies to read
- **Read tests alongside code** — tests are often the clearest specification of what the code should do
- **Don't read comments as truth** — comments drift. Trust the code.
- **Follow the types, not the function names** — names lie, types are checked

For a large file: scroll the outline (class/function list), locate the relevant symbol, read only that.

## Convention detection

Most codebases have unwritten rules. Learn them before writing code:

- How are errors returned? (Result types, exceptions, null returns, error strings)
- How are tests structured? (Framework, naming, file layout)
- How are modules organized? (By feature? By layer? By type?)
- How is dependency injection handled?
- Where does config come from?
- How are migrations written?
- What's the logging / observability style?

Copy the patterns. Don't invent new ones. A PR that violates local conventions gets rejected even if the code is "technically better."

## Ask the user the right questions

Some things you cannot learn from the code:

- Business domain knowledge ("why does `premium_tier` have two meanings?")
- Historical context ("why was this service split from the main app?")
- Current priorities ("is this module being deprecated?")
- Who owns what
- Which tests are known-flaky

When you hit one of these, ask. Efficient exploration is knowing what not to spelunk for.

## The "I understand enough" checkpoint

Before making a change, check:

- [ ] I can name the file(s) I need to edit
- [ ] I understand how the code I'm editing is called
- [ ] I understand what else might break if I change it
- [ ] I know which tests exercise this code
- [ ] I know the local convention for this kind of change

If any box is unchecked, read more targeted code or ask the user.

## Anti-patterns

- **Spelunking**: reading files "to get a feel" without a specific question
- **Re-reading on autopilot**: if you already read a file, trust your notes
- **Summarizing the whole repo**: tempting, useless for specific tasks
- **Over-trusting docs**: verify against code before acting
- **Over-trusting names**: `UserService` might have nothing to do with users anymore
- **Mimicking old patterns blindly**: if the convention is clearly wrong and the team knows it, ask before extending it

## Activation

When this skill activates, respond with:

🗺️

Then ask what you're trying to accomplish in this codebase, and start the orientation pass.
