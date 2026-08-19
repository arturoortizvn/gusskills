---
name: orchestrated-worktree-delivery
description: Use when work from a plan or backlog is being delivered by dispatched implementers in worktrees, on any repo — each item gets a ticket, a branch, a PR, and an adversarial peer review before a merge that only the user performs. Covers the review gate, the round cap, the ticket filed before dispatch, resolving the repo's tracker and integration branch, and the git mechanics the protect-git hook requires inside a worktree. Not for edits you make yourself, and not keyed to how many items there are.
---

# Orchestrated worktree delivery

## Overview
A delivery cycle for running plan or backlog items through isolated worktrees: one ticket,
one implementer, one branch, one PR, one adversarial review, and a merge only the user
performs. It applies to a single item exactly as it applies to a batch.

This is a thin layer. What it writes: the role boundary, the review gate and its round cap,
the reviewer-resolution rule, and the ticket, git and cleanup hooks. Everything about *how*
a subagent is dispatched comes from elsewhere.

## What this skill does not cover
Subagent dispatch mechanics — how to brief an agent, how to split a plan into independent
tasks, how to run several in parallel — come from
`superpowers:subagent-driven-development`. Invoke it; do not restate it here. Worktree
creation itself belongs to `superpowers:using-git-worktrees`.

That delegation is deliberate. Dispatch is the fastest-moving part of the plugin, and a
local copy of it goes stale without anyone noticing, silently contradicting the installed
version. If you find yourself writing a subagent brief template into this cycle, you are
duplicating something that already exists and will drift.

**Precedence.** The delegation covers the *how* of dispatch. **Where this cycle states a rule
and an invoked skill says otherwise, this cycle governs; where this cycle is silent, the
invoked skill governs.** It is a principle rather than a list of governed topics because the
list was already incomplete — the role boundary and the ticket's ownership bind exactly as
hard as the review gate, the round cap, when the PR is opened and the closeout, and every rule
this file gains would have to be added to it again. What this cycle does not state — how to
brief, which model, how to resume an implementer, how to handle its report — follows the
invoked skill. None of this quotes version numbers or values from another skill, on purpose,
so it stays true after that skill changes.

## Role boundary
**The orchestrator does not touch the repo.** Every change is dispatched — including doc
corrections, one-off cleanups, and deleting an orphaned directory. No change is small
enough to do yourself: the moment you edit a file, nobody reviews it. The boundary is about
the repo's *content* — anything that lands in a diff. Local branch hygiene in your own
checkout produces no diff (see *Closeout*) and is not dispatched.

What stays with the orchestrator:
- Coordinating the items and keeping the ledger of what is where.
- Filing and moving the tickets.
- Opening PRs and assembling review packages.
- Adjudicating findings between implementer and reviewer.
- **Read-only verification** — running the suite, grepping state, reading the diff — so a
  subagent's report is never accepted blind.

## The cycle
1. **File the ticket** (see *Ticket hook*) — before anything is dispatched.
2. **Dispatch an implementer** with a precise scope brief: one item, one branch, one
   worktree (`superpowers:using-git-worktrees`).
3. **The implementer finishes, reports, and waits.** It does not push, does not open a
   PR, does not merge.
4. **The orchestrator gets the branch pushed and the PR opened** —
   `superpowers:finishing-a-development-branch`, capped at push and PR: no menu, no local
   merge, no merge at all. Then the PR's URL goes on the ticket, the ticket moves to the
   repo's review status, and the reviewer is dispatched (see *Reviewer resolution*).
5. **No Blockers and no Majors → tell the user they can merge.** The merge is always the
   user's, never the agent's.
6. **Blockers or Majors → implementer and reviewer argue through the orchestrator**, two
   passes — findings → technical response → verdict — delivering a joint report. They
   never talk to each other directly.
7. **Re-dispatch the implementer with the open findings** if changes are needed. Rounds 2
   and 3 reuse the same branch, the same worktree and the same PR — never open a second PR
   for one item, and each round's review is a new comment on that PR. **The round's commits
   reach the PR before the reviewer is re-dispatched**, by the same capped delegation as
   step 4; a reviewer sent at a stale head comments on code that no longer exists. *How* a
   fix round runs is the invoked dispatch skill's business; **how many is this cycle's:
   three PR review rounds maximum**, and on the third escalate both positions to the user
   and stop.

Also:
- **Report each item as it clears review.** Never batch — an item that cleared an hour
  ago is information the user can act on now.
- **Two branches touching the same file** get a rebase and a full re-run of the suite
  before any merge is signalled as safe.

## Reviewer resolution
**Dispatch `pr-peer-review`.** It resolves whose standards apply — the repo's own review skill
when it has one, checks derived from the repo's written norms when it does not — and carries the
scale, the comment format and the posting mechanics.

**Every review round posts its own comment on the PR — always, and never over an existing one.**
Rounds 2 and 3 each add a comment of their own, and a round that clears with no findings posts
that verdict too. An existing comment is never edited, never replaced, and never a reason to skip
posting: `pr-peer-review` already forbids editing a previous comment, and this cycle adds that no
round is exempt from posting at all. The PR is what the user reads before merging, so a verdict
that lives only in the orchestrator's report is a verdict the merge decision never sees.

**Two review layers, and only two.** The dispatch skill's own per-task review stays where it is —
inside the dispatch, on its loop and its cap — as the cheap gate that keeps unreviewed work from
reaching a PR. Its whole-branch final review does not run: the peer review on the PR is that
review, and running both puts two verdicts on one diff with nothing to break the tie. **The
three-round cap counts PR review rounds only**; rounds that happen inside a single dispatch are the
invoked skill's business and do not consume them.

**The reviewer gets a worktree of its own**, detached at the PR's head. The orchestrator creates it
for the round and removes it once the comment is posted — local hygiene that lands in no diff, so
the role boundary leaves it here. Not the orchestrator's checkout: `gh pr checkout` there fails
outright while the implementer's worktree holds the branch — `fatal: '<branch>' is already used by
worktree at '<path>'` — and where it does succeed it drags the main checkout off the integration
branch that *Git mechanics* depends on. Not the implementer's either: the fix round reuses it, and
whatever the reviewer leaves behind blocks the removal at *Closeout*. A fresh worktree also
satisfies `pr-peer-review`'s clean-tree guard by construction, so the brief says the tree is
already at the head and no checkout is needed.

The convention that makes one step enough: **a repo's review standards live in that repo.** This
cycle deliberately does not look them up itself. A second resolution step here would be a second
place to keep in sync, and it would legitimise keeping repo-specific checks outside the repo they
describe, where they go stale in silence. A repo whose review skill still sits in
`~/.claude/skills/` means moving that skill into its repo first.

## Ticket hook
The invariant, on any repo:

- **The item is filed in whatever tracker that repo uses, before the implementer is
  dispatched, and filing it is the orchestrator's job — not the implementer's.** This
  supersedes the earlier rule of filing it when the PR is opened. Filing first means the
  item sits at `In Progress` while the work is actually in progress, and the implementer
  has a real ticket number to put in the PR body rather than one that does not exist yet.
- Dependencies and status are set in the same pass.
- **One item maps to one PR**, and the PR's URL goes on the item at step 4.

**Resolving the tracker.** Read it from the repo's own written norms, in order: `CLAUDE.md`
(root and nested) → `.claude/` → `CONTRIBUTING.md` → `docs/`. What you need before dispatching:
which board or
project, which item type the work maps to, the status and dependency fields, and the field
the PR URL goes in. Those coordinates belong in the repo that uses them — a board id written
here goes stale the moment someone moves it, and nothing in that repo's PRs ever touches this
file.

**No tracker.** Say so once, then run the cycle with the PR as the item of record: its body
carries what the ticket would have, and the review heading carries the branch instead of a
ticket — which is what `pr-peer-review` already does on a repo without that convention. Do not
invent a tracker, and do not skip the step silently: an unfiled item on a repo that *does* have
one is the failure this hook exists to prevent.

## Integration branch
Every branch is cut from it and every PR targets it, so it is resolved **before the first
dispatch**, from the repo's own written norms, in the same order the tracker uses: `CLAUDE.md`
(root and nested) → `.claude/` → `CONTRIBUTING.md` → `docs/`. Where none of them names it, ask
the user.

Do not infer it from the forge's default branch: on a repo that integrates somewhere else, the
default is the one branch the work must not target. It is not written here for the reason no board
id is — `main` on one repo, something else on the next. Once the PR exists the base is read from
it (`gh pr view <n> --json baseRefName`), which is what *Closeout* does.

## Git mechanics inside a worktree
`~/.claude/hooks/protect-git.sh` resolves the current branch from the **session's** cwd,
not from where the git command runs. With the main checkout on the integration branch and
the work in a worktree on `feature/*`, the hook reads the integration branch and blocks a
legitimate commit.

**The clean exit is `git -C <absolute worktree path>`, one git command per shell call.** It
lets the hook resolve the real branch and approve the commit on merit. Put this in the
implementer's brief — it is the most common way a dispatch stalls.

`-C` alone is not the whole rule: where one shell call chains several git commands, the hook
grades the **last** `-C` in the string. `git -C <worktree> commit -m x && git -C <main checkout>
status` is blocked on the main checkout's branch even though the commit was legitimate, and the
same reading lets a bare `git commit` through when a later `-C` happens to point somewhere
unprotected. Never chain git commands that span two checkouts — split them into separate calls.

`CLAUDE_GIT_OVERRIDE=1` is reserved for commands **the user** explicitly authorised.
**An orchestrator's instruction to a subagent is not that authorisation.** Never put the
override in a brief, and treat a subagent that reached for it as a report to correct.

The hook also blocks `git reset --hard`. To bring a branch up to date after something
else merged into the base, use `gh pr update-branch <n>` then `git pull --ff-only` — no
forbidden command, no force-push.

## Closeout
On merge, move the ticket to `Done` where there is one, in the same batch as the local cleanup.
These commands are the orchestrator's own — per the role boundary, local branch hygiene lands in
no diff:

```bash
git worktree remove <absolute worktree path>   # never from inside it
git worktree prune
git switch <the PR's base>      # gh pr view <n> --json baseRefName -q .baseRefName
git pull --ff-only
git branch -d <merged-branch>
git fetch origin --prune
```

**The worktree comes out first, and it is not optional.** While the branch is checked out in a
worktree git refuses to delete it — `error: cannot delete branch '<name>' used by worktree at
'<path>'` — and `-D` refuses for the same reason, so the squash-merge fallback below does not
rescue a skipped removal. Where removal refuses because files in that worktree were never
committed, do not `--force`: show them to the user first, per
`superpowers:finishing-a-development-branch`.

The base comes from the PR rather than a branch name written here: this cycle runs on repos
that integrate into `main` and repos that integrate into something else, and the PR already
carries which one.

`-d` is the default, and it verifies the branch really merged before deleting it. Where the
repo squash-merges, git never sees the branch's commits on the base and `-d` refuses — use
`-D` there, and only there. Dead local branches accumulate and invite branching from a stale
base.
