# Design — `orchestrated-worktree-delivery`, lean skill plus rationale file

**Date:** 2026-08-19
**Status:** approved (design), pending implementation plan
**Deliverable:** `orchestrated-worktree-delivery/SKILL.md` cut to the operative rules, a new
`orchestrated-worktree-delivery/rationale.md` holding the argument for them, and `README.md`'s
layout block updated. No other file changes.

## Why

The file works and has just been through two rounds of correction, but it is 2497 words against
`writing-skills`' guidance of under 500, and roughly half of it argues *why* each rule exists
rather than stating the rule. That prose is not waste — it is what stops a future reader from
undoing a rule whose reason is invisible — but it loads into every session that touches the skill,
and its bulk is what makes the file hard to keep consistent. Two of the corrections this file has
already needed were contradictions between paragraphs far apart in it.

Six smaller items were declared out of scope by earlier designs and are collected here, because
each one is a sentence being rewritten anyway:

| # | Item | Where it stands today |
|---|---|---|
| 1 | The frontmatter `description` summarises the workflow | The failure `writing-skills` documents: the agent follows the summary instead of reading the body |
| 2 | "on the third escalate" is ambiguous | Reads two ways against "rounds 2 and 3 reuse the same branch" — does round 3 run? |
| 3 | Steps 1 and 5 read as unconditional | *Ticket hook* has a no-tracker branch the cycle's steps do not acknowledge |
| 4 | `In Progress` and `Done` are repo-specific literals | The last survivors of the class the repo-agnostic rewrite removed |
| 5 | Closed lists in *Overview* and *Role boundary* are stale | Parked in PR #4's review: they omit the push and the reviewer worktree's lifecycle |
| 6 | No rationalization table, no red flags | `writing-skills` asks for both on a file that is mostly prohibitions |

## Decisions

| Decision | Choice |
|---|---|
| Where the rationale goes | A sibling `rationale.md` in the skill directory, referenced from `SKILL.md` in one line |
| What counts as rationale | Prose whose removal changes no action: why a delegation exists, why a coordinate is not written here, how a failure was reproduced |
| What stays despite looking like prose | Literal error messages — they are recognition aids at the point of need, not argument |
| Size target | `SKILL.md` 1000-1100 words; `rationale.md` unbounded |
| The six collected items | All six, in this pass |
| `README.md` | Its layout block gains the second file; leaving it would be the same staleness this repo keeps fixing |
| Safety criterion | An invariant list extracted from the current file, asserted against the result — not "it got shorter" |

## The split

`SKILL.md` keeps: the frontmatter, the overview, the delegation and the precedence rule stated once,
the role boundary and its duty list, the eight cycle steps, the four review invariants, the ticket
hook's invariant plus resolution order plus no-tracker branch, the integration branch rule, every
git mechanic and command, the closeout sequence, and the new rationalization table and red flags.

`rationale.md` receives, each under a heading naming the rule it explains:

- Why dispatch mechanics are delegated rather than copied (drift in the fastest-moving skill).
- Why precedence is a principle and not a list of governed topics.
- Why one resolution step is enough for the reviewer, and why a repo's standards live in that repo.
- Why the detached review worktree is named inline instead of delegated, with the reproduced
  `fatal: a branch named '<branch>' already exists`.
- Why the reviewer sits in neither the orchestrator's checkout nor the implementer's worktree, with
  the reproduced `fatal: '<branch>' is already used by worktree at '<path>'`.
- Why the push delegation must run from the worktree (the hook's branch resolution and
  `gh pr create`'s `--head` default).
- Why tracker coordinates and the integration branch are not written in the skill.
- Why the worktree comes out before `git branch -d`, and why `-D` is not the fallback it looks like.
- Why `CLAUDE_GIT_OVERRIDE` is never in a brief.

## The six collected items

1. **`description`** becomes triggering conditions and symptoms only, with no enumeration of what
   the cycle does: coordinating delivery rather than editing, including for one item; a dispatched
   commit blocked by the protect-git hook inside a worktree; a peer review verdict that has to
   reach a PR; a merged branch that refuses to delete. Plus the negative trigger it already has.
2. **The cap** becomes: when the third round's review still returns Blockers or Majors, escalate
   both positions to the user and stop. Round 3 runs.
3. **Steps 1 and 5** are keyed to an observable predicate — "where the repo has a tracker" — so the
   no-tracker branch in *Ticket hook* is reachable from the cycle rather than only from the hook.
4. **`In Progress` and `Done`** become "the repo's in-progress status" and "the repo's done status",
   matching what step 5 already says for the review status.
5. **The two closed lists** gain the push and the reviewer worktree's lifecycle, and are marked
   non-exhaustive so the next rule this file gains does not have to be added to them.
6. **A rationalization table and a red-flags list**, built from the failures this file actually
   produced rather than invented ones: `-C` on every call being treated as sufficient, `git branch
   -d` run with the worktree still alive, a fix round reusing the previous round's comment, an
   orchestrator editing one small file itself, a subagent reaching for `CLAUDE_GIT_OVERRIDE`, and a
   review round skipped because the previous comment "already covered it".

## Verification

The risk in a cut this size is losing a rule along with the prose around it, so acceptance is an
invariant list, not a word count. Every item below is asserted against the produced `SKILL.md`;
each is a rule that exists in the file at `3949c19` and must still be there after the cut.

1. One item maps to one ticket, one implementer, one branch, one PR, one review, and a merge only
   the user performs; a single item is treated as a batch of one.
2. Dispatch mechanics come from `superpowers:subagent-driven-development`; worktree creation from
   `superpowers:using-git-worktrees`.
3. Precedence: where this cycle states a rule it governs; where it is silent the invoked skill does.
4. The orchestrator does not touch the repo; the boundary is anything that lands in a diff; local
   branch hygiene is excluded.
5. The orchestrator's duties include read-only verification, so no subagent report is accepted blind.
6. Where the repo has a tracker, the ticket is filed before anything is dispatched, by the
   orchestrator, not the implementer.
7. The implementer does not push, does not open a PR, does not merge.
8. The task review is dispatched by the orchestrator, after the report, before the push.
9. The push and PR are delegated to `superpowers:finishing-a-development-branch` capped at push and
   PR — no menu, no local merge, no merge at all.
10. That delegation runs from the item's worktree, with `--head` the item's branch and `--base` the
    integration branch.
11. No Blockers and no Majors → the user is told they can merge; the merge is never the agent's.
12. Blockers or Majors → implementer and reviewer argue through the orchestrator, two passes, joint
    report, never directly with each other.
13. Rounds 2 and 3 reuse the same branch, worktree and PR; never a second PR for one item.
14. Each round's commits reach the PR before the reviewer is re-dispatched.
15. Three PR review rounds maximum, and the escalation is stated unambiguously.
16. Each item is reported as it clears review, never batched.
17. Two branches touching the same file get a rebase and a full suite re-run before any merge is
    signalled safe.
18. `pr-peer-review` is what resolves whose standards apply.
19. Every review round posts its own comment on the PR, never over an existing one, including a
    round that clears with no findings.
20. The dispatch skill's whole-branch final review does not run; the cap counts PR rounds only.
21. The reviewer gets its own detached worktree via `git worktree add --detach`, created and removed
    by the orchestrator per round, with the comment body written outside it.
22. The tracker resolution order is `CLAUDE.md` (root and nested) → `.claude/` → `CONTRIBUTING.md`
    → `docs/`, and the integration branch uses the same order.
23. A repo with no tracker runs the cycle with the PR as the item of record; no tracker is invented
    and the step is not skipped silently.
24. The integration branch is resolved before the first dispatch and never inferred from the forge's
    default branch.
25. `git -C <absolute worktree path>`, one git command per shell call, because the hook grades the
    last `-C` in the string.
26. `CLAUDE_GIT_OVERRIDE` is never put in a brief, and the hard reset the hook blocks is replaced
    by `gh pr update-branch` plus `git pull --ff-only`.
27. The closeout removes the worktree and prunes before `git branch -d`, reads the base from the
    PR, and treats `-D` as squash-merge-only.
28. A removal that refuses because of uncommitted files goes to the user, never to `--force`.

Also executable: `SKILL.md` between 950 and 1150 words; `rationale.md` present and referenced from
`SKILL.md`; no `In Progress`, `Done` or `Code Review` literal; the `description` naming no cycle
step; `README.md`'s layout block listing `rationale.md`; and both files resolving through
`~/.claude/skills/orchestrated-worktree-delivery/`.

The pending manual check the user already owns — a full cycle run on a repo with a non-`main`
integration branch and a tracker — is unchanged by this work and still theirs.

## A note the implementer needs

The protect-git hook greps the *command string* it is about to run, so a heredoc that writes
documentation containing a forbidden command trips it: writing this spec was blocked once because a
sentence put `git reset` and the long-form hard flag on one line. Split such a phrase across lines
or word it without the flag. Never reach for `CLAUDE_GIT_OVERRIDE` to get a document written — that
escape hatch is for commands the user authorised, and a blocked heredoc is not one.

## Out of scope

- **`pr-peer-review`.** It has the same shape of problem (203 lines, much of it argument) and is not
  touched here. Its own pass, its own branch.
- **The cycle's behaviour.** Nothing about what the cycle does changes; if the invariant list and
  the produced file disagree, the file is wrong.
