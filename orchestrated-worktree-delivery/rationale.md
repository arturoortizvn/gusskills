# Why the delivery cycle is shaped this way

Every rule in `SKILL.md` that is not self-evident has its argument here. Read this before
relaxing a rule, not before following one.

## Why dispatch mechanics are delegated rather than copied

That delegation is deliberate. Dispatch is the fastest-moving part of the plugin, and a local
copy of it goes stale without anyone noticing, silently contradicting the installed version. If
you find yourself writing a subagent brief template into this cycle, you are duplicating
something that already exists and will drift.

## Why precedence is a principle and not a list of governed topics

It is a principle rather than a list of governed topics because the list was already incomplete
— the role boundary and the ticket's ownership bind exactly as hard as the review gate, the
round cap, when the PR is opened and the closeout, and every rule this file gains would have to
be added to it again. What this cycle does not state — how to brief, which model, how to resume
an implementer, how to handle its report — follows the invoked skill. None of this quotes
version numbers or values from another skill, on purpose, so it stays true after that skill
changes.

## Why one resolution step is enough for the reviewer, and where a repo's standards live

The convention that makes one step enough: **a repo's review standards live in that repo.**
This cycle deliberately does not look them up itself. A second resolution step here would be a
second place to keep in sync, and it would legitimise keeping repo-specific checks outside the
repo they describe, where they go stale in silence.

The posting rule has the same root. `pr-peer-review` already forbids editing a previous
comment; what this cycle adds is that no round is exempt from posting at all, because the PR is
what the user reads before merging — a verdict that lives only in the orchestrator's report is
a verdict the merge decision never sees.

And the dispatch skill's whole-branch final review does not run because the peer review on the
PR is that review: running both puts two verdicts on one diff with nothing to break the tie.

## Why the detached review worktree is named inline instead of delegated

That command is written here rather than delegated because `superpowers:using-git-worktrees`
only creates a worktree on a *new* branch — `git worktree add "$path" -b "$BRANCH_NAME"`, which
against the branch the PR already has dies with
`fatal: a branch named '<branch>' already exists`, and its preferred native tool creates branches
too. Detached, the worktree coexists with the implementer's on that same branch.

## Why the reviewer sits in neither existing tree

Not the orchestrator's checkout: `gh pr checkout` there fails outright while the implementer's
worktree holds the branch — `fatal: '<branch>' is already used by worktree at '<path>'` — and
where it does succeed it drags the main checkout off the integration branch that *Git
mechanics* depends on. Not the implementer's either: the fix round reuses it, and whatever the
reviewer leaves behind blocks the removal at *Closeout*.

A fresh worktree also satisfies `pr-peer-review`'s clean-tree guard by construction, which is
why the brief can say the tree is already at the head and no checkout is needed.
`pr-peer-review` posts with `gh pr comment --body-file <file>`, and a body file left in the
review tree is enough to make the removal refuse — `contains modified or untracked files` —
where forcing the removal is not an option this cycle allows.

## Why the push delegation runs from the worktree

Run from the orchestrator's checkout, the hook grades the push against the integration branch
that checkout sits on (`pushing while on protected branch 'main'`), and `gh pr create` defaults
`--head` to that same branch, opening the PR *from* the integration branch. Naming the worktree
and the head explicitly is the fallback for a session whose cwd cannot move, not an alternative
to it.

Omitting `--base` fails the same way, quietly rather than loudly: `gh pr create` then defaults it
to the forge's repository default branch, and *Integration branch* forbids inferring the
integration branch from that default. On a repo where the two differ, the PR would open against
the wrong branch with no error to catch it.

## Why tracker coordinates and the integration branch are not written in the skill

Those coordinates belong in the repo that uses them — a board id written here goes stale the
moment someone moves it, and nothing in that repo's PRs ever touches this file. The integration
branch is left out for the reason no board id is written down: `main` on one repo, something
else on the next, and on a repo that integrates somewhere else the forge's default is the one
branch the work must not target.

The base at closeout comes from the PR for the same reason rather than from a branch name
written here: this cycle runs on repos that integrate into `main` and repos that integrate into
something else, and the PR already carries which one.

## Why the worktree comes out before `git branch -d`, and why `-D` is not the fallback it looks like

While the branch is checked out in a worktree git refuses to delete it, and `-D` refuses for
the same reason — the squash-merge fallback does not rescue a skipped removal, because the
objection is the worktree, not the merge status. `-d` verifies the branch really merged before
deleting it, which is the check worth keeping: dead local branches accumulate and invite
branching from a stale base.

## Why `CLAUDE_GIT_OVERRIDE` is never in a brief

The override exists for commands **the user** explicitly authorised, and an orchestrator is not
the user. An instruction it writes into a brief is not authorisation, so a subagent that
reached for the override on that basis is a report to correct rather than a dispatch that got
unstuck.
