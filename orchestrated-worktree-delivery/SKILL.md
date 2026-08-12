---
name: orchestrated-worktree-delivery
description: Use when work from a plan or backlog is being delivered by dispatched implementers in worktrees — each item gets a ticket, a branch, a PR, and an adversarial peer review before a merge that only the user performs. Covers the review gate, the round cap, the ticket filed before dispatch, and the git mechanics the protect-git hook requires inside a worktree. Not for edits you make yourself, and not keyed to how many items there are.
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

**Precedence.** The delegation covers the *how* of dispatch. Where this cycle and any skill
it invokes disagree, this cycle governs four things: the review gate, the round cap, when
the PR is opened, and the closeout. Everything else — how to brief, which model, how to
resume an implementer, how to handle its report — follows the invoked skill. This is stated
without version numbers or quoted values from another skill on purpose, so it stays true
after that skill changes.

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
4. **The orchestrator opens the PR**, puts its URL on the ticket and moves the ticket to
   `Code Review`, then dispatches the reviewer (see *Reviewer resolution*).
5. **No Blockers and no Majors → tell the user they can merge.** The merge is always the
   user's, never the agent's.
6. **Blockers or Majors → implementer and reviewer argue through the orchestrator**, two
   passes — findings → technical response → verdict — delivering a joint report. They
   never talk to each other directly.
7. **Re-dispatch the implementer with the open findings** if changes are needed. Rounds 2
   and 3 reuse the same branch, the same worktree and the same PR — never open a second PR
   for one item. *How* a fix round runs is the invoked dispatch skill's business; **how
   many is this cycle's: three rounds maximum**, and on the third escalate both positions
   to the user and stop.

Also:
- **Report each item as it clears review.** Never batch — an item that cleared an hour
  ago is information the user can act on now.
- **Two branches touching the same file** get a rebase and a full re-run of the suite
  before any merge is signalled as safe.

## Reviewer resolution
Two steps, in order:

1. `.claude/skills/peer-review/` exists in the repo → **use it.**
2. Otherwise → `superpowers:requesting-code-review`, with the fallback scale and format
   below.

The convention this encodes: **a repo's review standards live in that repo.** There is
deliberately no intermediate "look for a global skill named after the repo" step — it
would make the lookup depend on a naming convention someone has to remember, and it would
legitimise keeping repo-specific checks outside the repo they describe, where they go
stale in silence. Running this cycle on a repo whose review skill still sits in
`~/.claude/skills/` means moving that skill into its repo first — today that is
`idanalyzer-peer-review`, which must move into IDAnalyzer before the cycle runs there.

### Fallback severity scale and format
Used only when the repo has no `peer-review` skill of its own. This duplicates ~20 lines
against a repo's own `peer-review` skill on purpose: without it, the review step cannot run
at all on a repo that has not written its standards yet.

- 🔴 **Blocker** — must fix before merge. Breaks a published contract, leaks secrets or
  PII, makes the test suite non-hermetic, or contradicts an approved spec.
- 🟠 **Major** — fix before merge unless explicitly waived. Architecture deviations, new
  logic with no tests, unquantified new cost in the request path.
- 🟡 **Minor** — fix here or open a follow-up. Code smells, naming, dead branches.
- 🔵 **Nit** — optional polish. Style, formatting, micro-optimisations.

**Exactly one** PR comment (`gh pr comment <n> --body-file <file>`), never edited and
never re-posted over. All four severity headers always present, `_None._` under the empty
ones. Every finding cites `file:line`. Checks that could not be run go under a "Could not
verify" heading instead of being guessed. Verdict: any 🔴 → Request changes; only 🟡/🔵 →
Approve; 🟠 without 🔴 → Comment, with the decision to be made spelled out in the summary.

## Ticket hook
The invariant, on any repo:

- **The item is filed in whatever tracker that repo uses, before the implementer is
  dispatched, and filing it is the orchestrator's job — not the implementer's.** This
  supersedes the earlier rule of filing it when the PR is opened. Filing first means the
  item sits at `In Progress` while the work is actually in progress, and the implementer
  has a real ticket number to put in the PR body rather than one that does not exist yet.
- Dependencies and status are set in the same pass.
- **One item maps to one PR**, and the PR's URL goes on the item at step 4.

In TaxReturnAnalyzer, the coordinates are Monday board `18425100702`: product work maps to
its existing `U-04-1040-XX` item; everything else — infrastructure, tooling, docs — gets a
new `OPS-NN · <title>` item in *Operations and Quality* (`group_mm5xd2s3`). `Depends On`
and `Status` are the columns; `Pull Request` is a link column holding a single URL, which
is where the one-item-one-PR rule comes from. MCP tools: `create_items`,
`change_item_column_values`. On another repo, keep the invariant and use that repo's
tracker.

## Git mechanics inside a worktree
`~/.claude/hooks/protect-git.sh` resolves the current branch from the **session's** cwd,
not from where the git command runs. With the main checkout on `develop` and the work in
a worktree on `feature/*`, the hook reads `develop` and blocks a legitimate commit.

**The clean exit is `git -C <absolute worktree path>` on every call.** It lets the hook
resolve the real branch and approve the commit on merit. Put this in the implementer's
brief — it is the most common way a dispatch stalls.

`CLAUDE_GIT_OVERRIDE=1` is reserved for commands **the user** explicitly authorised.
**An orchestrator's instruction to a subagent is not that authorisation.** Never put the
override in a brief, and treat a subagent that reached for it as a report to correct.

The hook also blocks `git reset --hard`. To bring a branch up to date after something
else merged into `develop`, use `gh pr update-branch <n>` then `git pull --ff-only` — no
forbidden command, no force-push.

## Closeout
On merge, move the ticket to `Done` in the same batch as the local cleanup. These commands
are the orchestrator's own — per the role boundary, local branch hygiene lands in no diff:

```bash
git switch develop && git pull --ff-only
git branch -D <merged-branch>   # -D, not -d: squash merges do not register as merged
git fetch origin --prune
```

`-D` is not carelessness. The repo squash-merges, so git never sees the branch's commits
on `develop` and `-d` refuses to delete it. Dead local branches accumulate and invite
branching from a stale base.
