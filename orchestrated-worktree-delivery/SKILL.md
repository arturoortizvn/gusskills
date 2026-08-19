---
name: orchestrated-worktree-delivery
description: Use when you are coordinating delivery of plan or backlog items on any repo — dispatching implementers who work in worktrees rather than editing yourself — including for a single item. Also when a dispatched commit is blocked by the protect-git hook inside a worktree, when a peer review verdict has to reach a PR, or when a merged branch refuses to delete. Not for edits you make yourself.
---

# Orchestrated worktree delivery

## Overview
A delivery cycle for plan or backlog items in isolated worktrees: one ticket, one implementer, one
branch, one PR, one adversarial review, and a merge only the user performs. It applies to a single
item exactly as it applies to a batch.

This is a thin layer: it states the boundaries, the gates and the hooks, and everything about *how*
a subagent is dispatched comes from elsewhere.

Why each rule has this shape: `rationale.md`, in this directory. Read it before relaxing a rule,
not before following one.

## What this skill does not cover
Subagent dispatch mechanics — briefing, splitting a plan into independent tasks, running several in
parallel — come from `superpowers:subagent-driven-development`. Invoke it; do not restate it here.
Worktree creation belongs to `superpowers:using-git-worktrees`.

**Precedence.** **Where this cycle states a rule and an invoked skill says otherwise, this cycle
governs; where this cycle is silent, the invoked skill governs.**

## Role boundary
**The orchestrator does not touch the repo.** Every change is dispatched, including doc corrections
and one-off cleanups: no change is small enough to do yourself. The boundary is the repo's
*content* — anything that lands in a diff. Local branch hygiene in your own checkout produces no
diff (see *Closeout*) and is not dispatched.

What stays with the orchestrator — not an exhaustive list, and a rule stated anywhere in this file
binds whether or not it appears here:
- Coordinating the items and keeping the ledger of what is where.
- Filing and moving the tickets.
- Getting the branch pushed and the PR opened, per the cycle's push step, and assembling the
  review packages.
- Creating and removing the reviewer's worktree, per *Reviewer resolution*.
- Adjudicating findings between implementer and reviewer.
- **Read-only verification** — running the suite, grepping state, reading the diff — so a
  subagent's report is never accepted blind.

## The cycle
1. **Where the repo has a tracker, file the ticket** (see *Ticket hook*) — before anything is
   dispatched.
2. **Dispatch an implementer**: one item, one branch, one worktree
   (`superpowers:using-git-worktrees`).
3. **The implementer reports and waits.** It does not push, does not open a PR, does not merge.
4. **The orchestrator dispatches the task review** of that report —
   `superpowers:subagent-driven-development`'s per-task gate, after the report, never
   implementer-spawned, before the push. Its fix rounds do not consume the PR review rounds.
5. **The orchestrator gets the branch pushed and the PR opened** —
   `superpowers:finishing-a-development-branch`, capped at push and PR: no menu, no local merge,
   no merge at all. **From the item's worktree, never the orchestrator's checkout**
   (`pushing while on protected branch 'main'`); where the cwd cannot move, `git -C <the worktree>`
   on the push and `--head <the item's branch>` on the PR. **`--base` is the integration branch.**
   Then, where there is a ticket, the PR's URL goes on it and it moves to the repo's review status,
   and the reviewer is dispatched (see *Reviewer resolution*).
6. **No Blockers and no Majors → tell the user they can merge.** The merge is never the agent's.
7. **Blockers or Majors → implementer and reviewer argue through the orchestrator**, two passes —
   findings → technical response → verdict — into a joint report, never with each other.
8. **Re-dispatch the implementer with the open findings.** Rounds 2 and 3 reuse the same branch,
   worktree and PR — never a second PR for one item — and **each round's commits reach the PR
   before the reviewer is re-dispatched**. *How* a fix round runs is the invoked skill's business;
   **how many is this cycle's: three PR review rounds maximum**. Round 3 runs; when its review
   still returns Blockers or Majors, escalate both positions to the user and stop.

Also:
- **Report each item as it clears review**, never batched.
- **Two branches touching the same file** get a rebase and a full re-run of the suite before any
  merge is signalled safe.

## Reviewer resolution
**Dispatch `pr-peer-review`.** It resolves whose standards apply and carries the scale, the comment
format and the posting mechanics.

**Every round posts its own comment on the PR — always, and never over an existing one**, including
a round that clears with no findings. An existing comment is never edited or replaced.

**Two review layers, and only two.** Step 4's per-task gate stays; the dispatch skill's
whole-branch final review does not run. **The cap counts PR review rounds only.**

**The reviewer gets a worktree of its own**, detached at the PR's head:
`git worktree add --detach <absolute path> <the PR's branch>` — created per round by the
orchestrator, removed once the comment is posted. Do not delegate it:
`superpowers:using-git-worktrees` creates on a *new* branch and dies with
`fatal: a branch named '<branch>' already exists`. **Not the orchestrator's checkout**: `gh pr
checkout` there fails while the implementer's worktree holds the branch —
`fatal: '<branch>' is already used by worktree at '<path>'`. **Not the implementer's**: the fix
round reuses it. **The comment body is written outside the worktree, under `$TMPDIR`** — a body
file left inside makes the removal refuse
(`contains modified or untracked files`), and `--force` is not an option this cycle allows.

## Ticket hook
- **The item is filed in whatever tracker that repo uses, before the implementer is dispatched, and
  filing it is the orchestrator's job — not the implementer's.** Filing first puts it at the repo's
  in-progress status while the work is in progress, and gives the implementer a real ticket number
  for the PR body.
- Dependencies and status are set in the same pass.
- **One item maps to one PR**, and the PR's URL goes on the item at step 5.

**Resolving the tracker.** From the repo's own written norms, in order: `CLAUDE.md` (root and
nested) → `.claude/` → `CONTRIBUTING.md` → `docs/`. You need the board or project, the item type,
the status and dependency fields, and the field the PR URL goes in.

**No tracker.** Say so once, then run the cycle with the PR as the item of record: its body carries
what the ticket would have. Do not invent a tracker, and do not skip the step silently.

## Integration branch
Every branch is cut from it and every PR targets it, so it is resolved **before the first
dispatch**, from the repo's written norms, in the tracker's order. Where none of them names it, ask
the user. **Never infer it from the forge's default branch.** Once the PR exists the base is read
from it (`gh pr view <n> --json baseRefName`), which is what *Closeout* does.

## Git mechanics inside a worktree
`~/.claude/hooks/protect-git.sh` resolves the current branch from the **session's** cwd, not from
where the git command runs, so it blocks a legitimate commit made in a worktree.

**The clean exit is `git -C <absolute worktree path>`, one git command per shell call** — the hook
grades the **last** `-C` in the string. Never chain git commands that span two checkouts. Put this
in the implementer's brief.

`CLAUDE_GIT_OVERRIDE=1` is reserved for commands **the user** explicitly authorised. **An
orchestrator's instruction to a subagent is not that authorisation.** Never put the override in a
brief; treat a subagent that reached for it as a report to correct.

The hook also blocks `git reset`
with `--hard`. To update a branch after something merged into the base, use
`gh pr update-branch <n>` then `git pull --ff-only`.

## Closeout
On merge, move the ticket to the repo's done status where there is one, in the same batch as the
local cleanup — the orchestrator's own commands, per the role boundary:

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
'<path>'` — and `-D` refuses for the same reason. Where removal refuses because files there were
never committed, do not `--force`: show them to the user first, per
`superpowers:finishing-a-development-branch`.

`-d` is the default and verifies the branch really merged. Where the repo squash-merges it refuses
— use `-D` there, and only there.

## Rationalizations

Every row is a failure this cycle actually produced.

| Excuse | Reality |
|--------|---------|
| "It's one line in a doc — dispatching it is overhead" | The moment you edit a file, nobody reviews it. Dispatch it. |
| "I used `git -C` on every call, so the hook is handled" | The hook grades the **last** `-C` in the command string. One git command per shell call. |
| "The branch merged, so `git branch -d` will delete it" | Not while a worktree holds it, and `-D` refuses for the same reason. Remove the worktree first. |
| "Round 2's findings were already in round 1's comment" | A round with no comment of its own is a round the user cannot read. Post every round. |
| "The subagent needed the override to get unstuck" | `CLAUDE_GIT_OVERRIDE` is the user's to grant. An orchestrator's instruction is not authorisation. |
| "The reviewer can work in my checkout, it's clean" | `gh pr checkout` there fails while the implementer's worktree holds the branch, and drags the checkout off the integration branch when it does not. |
| "The delegated skill will figure out where to run" | It runs where you point it. From the orchestrator's checkout the push is blocked and the PR opens from the integration branch. |

## Red flags — stop

- You are about to edit a file in the repo yourself.
- You are about to chain two git commands that touch different checkouts.
- You are about to run `git branch -d` without having removed the worktree.
- You are about to skip a review round's comment because the last one covered it.
- You are about to put `CLAUDE_GIT_OVERRIDE` in a brief.
- You are about to merge, or to offer to merge.

**Every one of these means: stop and re-read the rule it breaks.**
