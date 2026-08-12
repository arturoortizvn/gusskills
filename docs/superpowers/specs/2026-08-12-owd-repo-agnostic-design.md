# Design — `orchestrated-worktree-delivery`, repo-agnostic

**Date:** 2026-08-12
**Status:** approved (design), pending implementation plan
**Deliverable:** a rewritten `orchestrated-worktree-delivery/SKILL.md` in this repo, carrying the
same delivery cycle with nothing in it naming a repository, a board or a branch. No other file
changes.

## Why

`pr-peer-review` was generalised by keeping the transferable half of peer review and resolving the
rest from the repo under review. `orchestrated-worktree-delivery` is the same shape one step
earlier in the pipeline and never got the same treatment: the cycle is portable, but five places in
the file pin it to one repository's coordinates, branch or merge setting.

The five, as they stand at `17966ed`:

| Where | What is tied down |
|---|---|
| Ticket hook | Monday board `18425100702`, the `U-04-1040-XX` item, `OPS-NN · <title>`, group `group_mm5xd2s3`, the column names and the MCP tool names — all TaxReturnAnalyzer's |
| Reviewer resolution, step 1 | Resolves by directory name (`.claude/skills/peer-review/`) |
| Reviewer resolution | Names `idanalyzer-peer-review` as the outstanding case |
| Git mechanics and Closeout | `develop` as the integration branch, in three places |
| Closeout | `-D` justified by "The repo squash-merges", stated as if universal |

The convention this repo runs on — `README.md`, *What does not belong here* — is that a skill citing
a specific repo's coordinates belongs in that repo, because nothing in a PR that moves a board ever
touches a file here. The ticket hook is that violation, written down and shipped.

Two smaller motives. The Nit that the review of PR #1 raised and round 2 declined — step 1 keying on
the directory name while `pr-peer-review` keys on the description — is dissolved rather than patched
here: the step it lives in stops existing. And the `-D` claim was falsified by this repo three hours
after it shipped: PR #1 merged with a merge commit and `git branch -d` deleted the branch without
complaint.

## Decisions

| Decision | Choice |
|---|---|
| Repo-specific tracker coordinates | Out, entirely. The hook keeps the invariant plus a resolution order; coordinates live in the repo that uses them |
| Migrating TaxReturnAnalyzer's coordinates into that repo | Out of scope — separate repo, separate branch, separate PR |
| Reviewer resolution | Collapses from a two-step cascade to one line: dispatch `pr-peer-review`, which resolves standards itself |
| Integration branch | Read from the PR's `baseRefName`; never named in the file |
| `-d` vs `-D` | Conditional on the repo's merge method, with the reason kept |
| `idanalyzer-peer-review` | Named in `README.md` only; dropped from the skill |
| Forge | GitHub via `gh`, unchanged — `pr-peer-review` already scopes itself that way |
| Edit shape | Rewrite the file rather than patch it; the changes reach most of the body |

## The changes

The file keeps its section order and lands at roughly its current length. *Overview*, *What this
skill does not cover* (the dispatch delegation and the precedence paragraph), *Role boundary* and
*The cycle* — including the review gate and the three-round cap — are carried over unchanged: none
of them names a repo. The frontmatter `description` gains that the cycle applies on any repo, so it
fires outside TaxReturnAnalyzer.

### 1. Reviewer resolution — two steps become one

The cascade goes. Replacement:

```markdown
## Reviewer resolution
**Dispatch `pr-peer-review`.** It resolves whose standards apply — the repo's own review skill
when it has one, checks derived from the repo's written norms when it does not — and carries the
scale, the comment format and the posting mechanics.

The convention that makes one step enough: **a repo's review standards live in that repo.** This
cycle deliberately does not look them up itself. A second resolution step here would be a second
place to keep in sync, and it would legitimise keeping repo-specific checks outside the repo they
describe, where they go stale in silence. A repo whose review skill still sits in
`~/.claude/skills/` means moving that skill into its repo first.
```

Why one step is enough: `pr-peer-review`'s own standards resolution already looks for the repo's
skill first and derives the checks when there is none, and it tests the *description* rather than
the directory name, so it finds a repo skill this cycle's old step 1 would have missed.

### 2. Ticket hook — invariant, resolution order, and the case of no tracker

The section's three invariant bullets stay verbatim — filed in the repo's tracker before dispatch
and by the orchestrator rather than the implementer; dependencies and status set in the same pass;
one item mapping to one PR, whose URL goes on the item at step 4. The TaxReturnAnalyzer paragraph
that follows them is replaced by:

```markdown
**Resolving the tracker.** Read it from the repo's own written norms, in order: `CLAUDE.md` (root
and nested) → `.claude/` → `docs/`. What you need before dispatching: which board or project,
which item type the work maps to, the status and dependency fields, and the field the PR URL goes
in. Those coordinates belong in the repo that uses them — a board id written here goes stale the
moment someone moves it, and nothing in that repo's PRs ever touches this file.

**No tracker.** Say so once, then run the cycle with the PR as the item of record: its body
carries what the ticket would have, and the review heading carries the branch instead of a ticket
— which is what `pr-peer-review` already does on a repo without that convention. Do not invent a
tracker, and do not skip the step silently: an unfiled item on a repo that *does* have one is the
failure this hook exists to prevent.
```

The no-tracker branch is new, and it is not hypothetical: this repo has no tracker, so the cycle
cannot run here at all without it.

**[corrected in review]** An earlier draft of that paragraph said `no ticket` goes in the review
heading. It does not: `pr-peer-review` puts the branch in the heading when the repo has no ticket
convention, and reserves the literal `no ticket` for a repo that has the convention but a PR that
names none. The wording above is the corrected one.

### 3. Closeout — base from the PR, `-d` by default

```bash
git switch <the PR's base>      # gh pr view <n> --json baseRefName -q .baseRefName
git pull --ff-only
git branch -d <merged-branch>
git fetch origin --prune
```

The note under it keeps the reasoning and makes the condition explicit: `-d` normally; where the
repo squash-merges, git never sees the branch's commits on the base and `-d` refuses, so `-D` is
correct there and only there. Dead local branches accumulate and invite branching from a stale
base. These commands remain the orchestrator's own — per the role boundary, local branch hygiene
lands in no diff.

### 4. Git mechanics — same content, no branch names

The `protect-git.sh` explanation is unchanged in substance; only its example is de-named: "with the
main checkout on the integration branch and the work in a worktree on `feature/*`, the hook reads
the integration branch and blocks a legitimate commit". The `gh pr update-branch` line becomes
"after something else merged into the base". `git -C <absolute worktree path>` on every call, and
the `CLAUDE_GIT_OVERRIDE` prohibition, are carried over untouched.

## Verification

A skill has no test suite. What is executable:

- The file names no repository, board, group or branch: grep for `18425100702`, `group_mm5xd2s3`,
  `U-04-1040`, `TaxReturnAnalyzer`, `idanalyzer` and `Monday` returns nothing. The branch check
  must be `grep -nE '\bdevelop\b'`, **not** a bare `develop` — the surviving reference to
  `superpowers:subagent-driven-development` contains the substring, so a bare grep reports a
  failure that is not one. Today the word boundaries isolate the five branch mentions
  (`:112`, `:113`, `:123`, `:131`, `:138`) from that one, and all five must be gone.
- `pr-peer-review` appears exactly once, in *Reviewer resolution*.
- The review gate survives the rewrite: its references to Blockers and Majors are load-bearing —
  the gate is *stated* in those terms — so grep hits there are correct, not leftovers.
- The invariant bullets of the ticket hook and the round cap are present, since the rewrite is
  meant to preserve them.
- Frontmatter `name` is still `orchestrated-worktree-delivery`, the directory name is unchanged,
  and the skill reads through `~/.claude/skills/orchestrated-worktree-delivery/SKILL.md`. This is
  the one check that can break a skill in use today, so it runs against the live path.

Real evidence is a cycle run on a repo that is not TaxReturnAnalyzer — the case the rewrite exists
to serve. There is none to run during this work, so it is declared a pending manual check rather
than claimed, per `superpowers:verification-before-completion`.

## Out of scope / follow-up

- **Migrating TaxReturnAnalyzer's tracker coordinates into that repo.** Separate repo, separate
  branch, separate PR. They are not lost by this rewrite: the Ticket hook section of
  `orchestrated-worktree-delivery/SKILL.md` at `17966ed` holds them, so the migration has a source
  to copy from afterwards. Until it happens, a cycle run in TaxReturnAnalyzer resolves the tracker
  from that repo's norms or asks.
- **Migrating `idanalyzer-peer-review` into IDAnalyzer.** Unchanged and still outstanding; the
  obligation now sits in `README.md` alone.
- **Non-GitHub forges.** No repo needs it.
