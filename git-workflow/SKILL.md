---
name: git-workflow
description: >
  Clean commits, sane branches, useful pull requests, and never losing work.
  Covers commit hygiene, branch strategy, rebase vs merge, conflict resolution,
  bisect, reflog rescue, and PR etiquette. Use whenever you're about to touch
  git — which is most of the time. Prevents the "I pushed force and now
  everything is gone" class of disaster. Composes with ultra-efficient.
---

# Git Workflow

You are operating git. Git is powerful and unforgiving: one wrong command and you lose work. This skill is the set of habits that keep the history clean, the branches tidy, and nothing irretrievably lost.

## The core principles

1. **Commits are units of thought.** One commit = one logical change. Not "end of day" and not "fix stuff."
2. **The main branch is sacred.** It should always build, test, and deploy cleanly.
3. **Never lose work.** Before any destructive command, make sure there's a way back. The reflog is your friend but only if you remember to use it.
4. **Reviewable > clever.** Small, focused PRs beat big, elegant refactors that no one can review.
5. **History is documentation.** Future you will read `git log` and `git blame` to understand why. Write commit messages for that reader.

## Commit hygiene

### Staging

- **Stage by intention, not by file.** `git add -p` is the workhorse — it lets you split even a single file into multiple commits. Learn it.
- **Never `git add .` without looking.** You'll stage secrets, debug prints, `.DS_Store`, or someone else's half-finished work sitting in your working tree.
- **Check `git diff --staged` before committing.** The staged diff is what you're actually committing. Look at it.

### Commit messages

Format:
```
type: short imperative subject line (under 72 chars)

Optional body explaining the *why*, not the *what*. The diff
already shows the what. The body explains context, trade-offs,
and anything that won't be obvious six months from now.

Refs: issue-123
```

Types (Conventional Commits or your team's convention): `feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `build`, `ci`, `chore`.

Rules:
- **Imperative mood**: "add user login" not "added user login" or "adds user login"
- **Subject line under 72 chars**, ideally 50
- **Blank line between subject and body**
- **Wrap body at ~72 chars** (or let your editor do it)
- **Explain WHY**, not what. What is in the diff. Why isn't.
- **Link the issue** if there is one
- **No "misc changes"**, no "wip", no "stuff" — take the extra 30 seconds

### One thing per commit

If a commit does two things, split it. Tools:
- `git add -p` to stage partially
- `git reset HEAD <file>` to unstage
- `git commit --patch` to cherry-pick hunks into a commit
- `git stash` to park the other changes while you commit one

The test: can you describe this commit in one sentence without using "and"?

## Branch strategy

Pick one and be consistent. For most teams:

- **main**: always deployable
- **feature branches**: short-lived, branched from main, merged back via PR
- **release branches** (only for teams with formal releases)
- **hotfix branches** (for production emergencies)

Long-lived feature branches are a warning sign. If a branch is living more than a week or two, either merge incrementally behind a flag or the scope is too big.

### Naming
- `feat/<short-desc>`, `fix/<issue-num>-<short-desc>`, `chore/<what>`
- No personal names in branches that will outlive you
- No `tmp`, `test`, `asdf`, `new-stuff`

## Keeping your branch up to date

Two options, both valid:

### Rebase onto main (for a clean linear history)
```
git fetch origin
git rebase origin/main
```

Pros: linear history, easier to bisect
Cons: rewrites commits, don't do it on branches others have checked out

### Merge main in (for a safer workflow)
```
git fetch origin
git merge origin/main
```

Pros: preserves history exactly, no rewriting
Cons: merge commits clutter the log

**Rule**: rebase your own private branch, merge anything that's been shared. Never rebase a branch someone else has based their work on.

## Conflicts

Conflicts are a normal part of collaborating. Approach:

1. **Understand both sides before resolving.** Read the `<<<<<<<` and `=======` and `>>>>>>>` blocks carefully. Don't just pick one.
2. **Resolve in the smallest unit possible.** If both sides changed a function, sometimes the answer is a new function that combines their changes.
3. **Test after resolving.** A conflict resolution that compiles can still be semantically wrong.
4. **Never blindly accept theirs or ours.** That's not resolution, that's deletion.

If a conflict is big and confusing, abort (`git rebase --abort` or `git merge --abort`) and get help. A bad resolution is much worse than a retry.

## Dangerous commands

These can destroy work. Use them only when you're sure, and know the recovery path.

### `git reset --hard`
- Wipes working tree and staged changes
- Commits you "lost" are still in the reflog for ~90 days. Find them with `git reflog` and recover with `git reset --hard <sha>` or `git checkout <sha>`

### `git push --force`
- Overwrites the remote branch
- **Never on main or shared branches**
- On your own feature branch: use `git push --force-with-lease` instead — it fails if someone else pushed meanwhile, preventing you from blowing away their work

### `git clean -fd`
- Deletes untracked files
- Check first with `git clean -n` (dry run)

### `git checkout --` / `git restore`
- Throws away uncommitted working-tree changes, unrecoverably
- There is no reflog for working-tree changes you never committed. Lost is lost.

### `git rebase -i`
- Rewrites history
- Fine on your own branch, dangerous on shared branches
- If you mess it up, `git reflog` to find the pre-rebase state

## Recovery patterns

Git very rarely loses data permanently. What it does is make old data unreachable from branches. The reflog remembers.

- **"I reset --hard and lost my commit"**: `git reflog`, find the SHA, `git reset --hard <sha>` or `git branch <name> <sha>`
- **"I deleted a branch"**: `git reflog` to find the last tip, `git branch <name> <sha>`
- **"I force-pushed over someone else's work"**: check `git reflog` on the remote if you have server access, or the other person's local reflog
- **"I committed to the wrong branch"**: `git reset HEAD~` on the wrong branch, then move to the right branch and `git cherry-pick` the SHA
- **"I made a typo in the last commit message"**: `git commit --amend` (if you haven't pushed)
- **"I want to undo a pushed commit without destroying history"**: `git revert <sha>`

## Interactive rebase — the tidy-up

Before opening a PR, clean up your commits:
```
git rebase -i origin/main
```

You can:
- **reword** — fix a commit message
- **squash / fixup** — combine commits
- **reorder** — move commits around
- **drop** — delete a commit

Goal: a sequence of commits that tells a story. Each one compiles. Each one does one thing. The PR reviewer can follow it.

Do this on your own branch before pushing or before others depend on your commits.

## Pull requests

### Size

- **Under 400 lines changed** is the sweet spot for effective review
- If it's bigger, split: multiple PRs stacked, or mechanical changes separate from logic changes
- "It's all one feature" isn't a reason to make a 3000-line PR

### Description

Include:
- **What**: one line summarizing the change
- **Why**: the problem this solves, linked to an issue
- **How**: the high-level approach, if non-obvious
- **Test plan**: what you tested, what you didn't, how a reviewer can verify
- **Screenshots / recordings** for UI changes
- **Deploy notes** if there's anything special (migrations, flag flips, env changes)

A PR without a description is a gift to no one.

### Self-review first

Before requesting review:
- Read your own diff. You'll find issues.
- Remove debug code, commented-out lines, TODO comments for things you already did
- Verify CI is green
- Confirm the commits tell a clean story

### Responding to review

- Take comments seriously, even the ones you disagree with
- If you push back, explain why with a real reason, not ego
- Small changes: amend or add a fixup commit
- After addressing comments, re-request review — don't assume the reviewer is watching

## Bisect — finding the commit that broke it

When did this bug get introduced? Use `git bisect`:

```
git bisect start
git bisect bad              # current version is broken
git bisect good <old-sha>   # this older version worked
# git checks out a middle commit
# test it, then:
git bisect good   # or: git bisect bad
# repeat until git narrows it to one commit
git bisect reset
```

Prerequisites: a reliable way to test "good" vs "bad" (even a manual one). With a script, `git bisect run <script>` automates the whole thing.

## Anti-patterns

- **Committing `node_modules`, `.env`, build artifacts, lockfile conflicts left unresolved**
- **"Final", "final2", "actually-final"** in commit messages or branch names
- **Giant "wip" commits** that nobody can review
- **Force-pushing shared branches**
- **`git push --force` on main**
- **Rebasing a branch others have branched off**
- **Merging without reading the diff**
- **"Resolved conflicts"** as the commit message — say what you resolved
- **Pushing to master/main directly** on a team that requires PRs
- **Using `git reset --hard` as a way to "start fresh"** without first saving your work to a branch

## Activation

When this skill activates, respond with:

🌿

Then ask what git operation you're helping with.
