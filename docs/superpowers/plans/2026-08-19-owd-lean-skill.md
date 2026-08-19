# `orchestrated-worktree-delivery` Lean Skill Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Cut `orchestrated-worktree-delivery/SKILL.md` from 2497 words to 1000-1100 by moving its
argument into a sibling `rationale.md`, and close the six items earlier designs deferred, without
losing a single operative rule.

**Architecture:** Two files in the skill directory plus one line in `README.md`. The split is one
atomic move — prose out, rules in — so it is one task rather than two: extracting the argument
without cutting it, or cutting it without extracting, both leave the skill worse than it started.
The task's own review is an invariant audit against the spec's 28-item list, which is what makes
the cut safe.

**Tech Stack:** Markdown, `python3` for exact-match edits, `grep`/`awk`/`wc` for assertions, `git`,
`gh` CLI, POSIX symlinks. No test framework applies.

**Spec:** `docs/superpowers/specs/2026-08-19-owd-lean-skill-design.md`

## Global Constraints

- **All prose in both files is English.** Conversation with the user is Spanish; file content,
  commit messages, branch names and PR titles/bodies are English (`~/.claude/CLAUDE.md`).
- **Never commit to `main`.** All work lands on `update/owd-lean-skill`, which already exists and
  holds the spec at `1885f35`. `main` is reached only through a PR **the user merges**.
- **Every git command uses `git -C /Users/arturoortiz/Proyectos/Personales/gusskills`, and one git
  command per shell call.** The hook resolves the branch from the session's cwd and grades the last
  `-C` in a chained string.
- **No `--no-verify`, no `--force`, no hard reset, no `CLAUDE_GIT_OVERRIDE`.**
- **The hook greps the command string, not its intent.** A heredoc that writes documentation
  containing a forbidden command trips it — writing the spec was blocked once because a sentence put
  `git reset` and the long-form hard flag on the same line. Split such a phrase across lines or word
  it without the flag. `CLAUDE_GIT_OVERRIDE` is never the answer to a blocked heredoc.
- **The active `gh` account must be `arturoortizvn`**, the repo owner. Verify, do not assume.
- **Nothing about the cycle's behaviour changes.** Where the produced file and the spec's invariant
  list disagree, the file is wrong.

## File Structure

| File | Responsibility | This plan |
|---|---|---|
| `orchestrated-worktree-delivery/SKILL.md` | The operative rules an agent follows | Rewritten, 1000-1100 words |
| `orchestrated-worktree-delivery/rationale.md` | Why each rule has its shape | Created |
| `README.md` | How this repo is laid out | One layout line |
| `docs/superpowers/specs/2026-08-19-owd-lean-skill-design.md` | The approved design and the invariant list | Read-only input, committed at `1885f35` |

## Verification model — read this before Task 1

A skill has no test suite, and the risk here is not a broken command but a rule that vanished with
the prose around it. So acceptance has two halves:

1. **Machine-checkable:** word counts, the absence of the three status literals, the `description`
   naming no cycle step, `rationale.md` present and referenced, `README.md` updated, both files
   resolving through the live symlink.
2. **The invariant audit:** all 28 items in the spec's *Verification* section, checked one by one
   against the produced `SKILL.md`. This is the task review's rubric, and it is the gate — a cut
   that passes every grep and drops invariant 17 has failed.

Nothing here can prove the cycle still reads well in practice. The user owns that check and has
said so.

---

### Task 1: Split the file and close the six items

**Files:**
- Modify: `orchestrated-worktree-delivery/SKILL.md`
- Create: `orchestrated-worktree-delivery/rationale.md`
- Modify: `README.md` (layout block)

**Interfaces:**
- Consumes: the spec's *The split*, *The six collected items* and *Verification* sections.
- Produces: the two files Task 2 ships, and the heading set `rationale.md` exposes for the
  reference line in `SKILL.md`.

- [ ] **Step 1: Confirm the starting state and take the baseline**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills branch --show-current
git -C /Users/arturoortiz/Proyectos/Personales/gusskills status --short
wc -w /Users/arturoortiz/Proyectos/Personales/gusskills/orchestrated-worktree-delivery/SKILL.md
```

Expected: branch `update/owd-lean-skill`, clean tree, 2497 words. A different count means the file
moved since the design was written — re-read it before cutting anything.

- [ ] **Step 2: Classify every paragraph before moving any of it**

Read `SKILL.md` whole and write a two-column list — paragraph opener, `rule` or `rationale` — to
`.superpowers/sdd/2026-08-19-owd-lean-skill/classification.md`. The test for `rationale` is: **if
this sentence disappeared, would any action change?** If no, it moves.

These paragraphs are already classified by the design; the line numbers are at `3949c19`:

| Line | Opener | Verdict |
|---|---|---|
| 13-15 | "This is a thin layer. What it writes:" | rationale, and its closed list is item 5 below — do not carry the enumeration over |
| 23-26 | "That delegation is deliberate." | rationale |
| 30-36 | "It is a principle rather than a list" | rationale — the rule in 28-30 stays |
| 104-105 | "The PR is what the user reads before merging" | rationale — the posting rule stays |
| 111-112 | "running both puts two verdicts on one diff" | rationale — "does not run" stays |
| 116-121 | "That command is written here rather than delegated" | rationale — the `--detach` command stays |
| 134-138 | "The convention that makes one step enough" | rationale |
| 154-157 | "Those coordinates belong in the repo that uses them" | rationale |
| 171-173 | "It is not written here for the reason no board id is" | rationale |
| 220-222 | "The base comes from the PR rather than a branch name" | rationale |
| 226-227 | "Dead local branches accumulate" | rationale — the `-d`/`-D` rule stays |

**Literal error messages stay in `SKILL.md`**, one line each, attached to the rule they warn about:
`fatal: '<branch>' is already used by worktree at '<path>'`, `error: cannot delete branch '<name>'
used by worktree at '<path>'`, `contains modified or untracked files`, `fatal: a branch named
'<branch>' already exists`, and `pushing while on protected branch 'main'`. How each was reproduced
goes to `rationale.md`.

- [ ] **Step 3: Write `rationale.md`**

One `##` heading per rule it explains, each naming the rule so a reader arriving from `SKILL.md`
lands in the right place. Carry the existing prose across rather than rewriting it — it was already
reviewed twice — and add only the connective sentence a heading needs. Open the file with:

```markdown
# Why the delivery cycle is shaped this way

Every rule in `SKILL.md` that is not self-evident has its argument here. Read this before relaxing
a rule, not before following one.
```

The nine headings the spec names, in the order the rules appear in `SKILL.md`: the delegation of
dispatch mechanics; precedence as a principle; one resolution step for the reviewer and where a
repo's standards live; the detached review worktree named inline; the reviewer in neither existing
tree; the push delegation running from the worktree; tracker coordinates and the integration branch
kept out of the skill; the worktree before `git branch -d`, and `-D`; `CLAUDE_GIT_OVERRIDE` never in
a brief.

- [ ] **Step 4: Rewrite `SKILL.md`**

Keep the section order. Every rule survives; only argument leaves. Add, immediately after the
overview:

```markdown
Why each rule has this shape: `rationale.md`, in this directory. Read it before relaxing a rule,
not before following one.
```

- [ ] **Step 5: Close the six collected items, exactly as worded here**

Item 1 — the frontmatter `description` becomes:

```yaml
description: Use when you are coordinating delivery of plan or backlog items on any repo — dispatching implementers who work in worktrees rather than editing yourself — including for a single item. Also when a dispatched commit is blocked by the protect-git hook inside a worktree, when a peer review verdict has to reach a PR, or when a merged branch refuses to delete. Not for edits you make yourself.
```

Item 2 — the cap sentence in the last cycle step becomes:

```markdown
**three PR review rounds maximum**. Round 3 runs; when its review still returns Blockers or
Majors, escalate both positions to the user and stop.
```

Item 3 — the ticket steps gain the predicate. Cycle step 1 becomes:

```markdown
1. **Where the repo has a tracker, file the ticket** (see *Ticket hook*) — before anything is
   dispatched.
```

and the ticket clause in the push step becomes:

```markdown
Then, where there is a ticket, the PR's URL goes on it and it moves to the repo's review status,
and the reviewer is dispatched (see *Reviewer resolution*).
```

Item 4 — `In Progress` becomes "the repo's in-progress status" and `Done` becomes "the repo's done
status", in the *Ticket hook* bullet and the *Closeout* opener respectively. No backticks: they are
no longer literals.

Item 5 — the two closed lists. The overview's "This is a thin layer. What it writes: …" enumeration
does not come back at all; the sentence that replaces it is:

```markdown
This is a thin layer: it states the boundaries, the gates and the hooks, and everything about *how*
a subagent is dispatched comes from elsewhere.
```

And *Role boundary*'s duty list gains two bullets and a non-exhaustive marker:

```markdown
What stays with the orchestrator — not an exhaustive list, and a rule stated anywhere in this file
binds whether or not it appears here:
- Coordinating the items and keeping the ledger of what is where.
- Filing and moving the tickets.
- Getting the branch pushed and the PR opened, per the cycle's push step.
- Creating and removing the reviewer's worktree, per *Reviewer resolution*.
- Opening PRs and assembling review packages.
- Adjudicating findings between implementer and reviewer.
- **Read-only verification** — running the suite, grepping state, reading the diff — so a
  subagent's report is never accepted blind.
```

Item 6 — append these two sections at the end of `SKILL.md`, verbatim:

```markdown
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
```

- [ ] **Step 6: Update `README.md`'s layout block**

Replace exactly:

```
  orchestrated-worktree-delivery/SKILL.md
```

with:

```
  orchestrated-worktree-delivery/{SKILL.md,rationale.md}
```

- [ ] **Step 7: Assert the machine-checkable half**

```bash
cd /Users/arturoortiz/Proyectos/Personales/gusskills/orchestrated-worktree-delivery
wc -w SKILL.md                                    # expect 950-1150
wc -w rationale.md                                # expect > 700
grep -c 'rationale.md' SKILL.md                   # expect >= 1
grep -nE 'In Progress|`Done`|Code Review' SKILL.md            # expect no output
grep -n 'three PR review rounds maximum' SKILL.md             # expect 1 hit
grep -n 'Round 3 runs' SKILL.md                               # expect 1 hit
grep -c 'not an exhaustive list' SKILL.md                     # expect 1
grep -c '^## Rationalizations' SKILL.md                       # expect 1
grep -c '^## Red flags' SKILL.md                              # expect 1
grep -c '^## ' rationale.md                                   # expect 9
awk '/^description:/{print length($0)-13}' SKILL.md           # expect < 600
grep -nE '^description:.*(step [0-9]|round cap|review gate)' SKILL.md   # expect no output
grep -n 'orchestrated-worktree-delivery/{SKILL.md,rationale.md}' ../README.md   # expect 1 hit
grep -oE 'fatal: .{0,40}|error: cannot delete branch.{0,30}|contains modified or untracked files|pushing while on protected branch' SKILL.md | sort -u
```

The last line prints the literal error messages that survived; all five named in Step 2 must appear.
The `description` grep is item 1's real test: the old one enumerated the cycle, and a description
that still names a step or a gate has not been fixed.

- [ ] **Step 8: Verify the live skill serves both files**

```bash
head -2 ~/.claude/skills/orchestrated-worktree-delivery/SKILL.md | tail -1
head -1 ~/.claude/skills/orchestrated-worktree-delivery/rationale.md
```

Expected: `name: orchestrated-worktree-delivery`, then the rationale's title line. This is the one
check that can break a skill in use today.

- [ ] **Step 9: Commit**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills add orchestrated-worktree-delivery/SKILL.md orchestrated-worktree-delivery/rationale.md README.md
```

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills commit -m "refactor: split the cycle into its rules and its rationale

The skill carried 2497 words, half of them arguing why each rule exists rather
than stating it, which is what made two of its corrections contradictions
between paragraphs far apart in it. The argument moves to rationale.md and the
skill keeps the rules, the commands and the literal error messages a reader
needs at the point of failure.

Six items earlier designs deferred close with it: the description no longer
summarises the workflow, the round cap says whether round 3 runs, the ticket
steps are keyed to whether the repo has a tracker, the last two repo-specific
status literals are gone, the two closed lists are complete and marked
non-exhaustive, and the file gains a rationalization table and red flags built
from the failures it actually produced.

Claude-Session: https://claude.ai/code/session_012XQbo1K6hpv2emRTMyQ5m4"
```

**Note for the controller:** this task's review is the invariant audit. Hand the reviewer the spec's
*Verification* list and require a verdict per item, not a summary.

---

### Task 2: Push, open the PR, hand off

**Files:** none in the repo — the deliverable is a PR.

**Interfaces:**
- Consumes: Task 1's commit and its invariant audit.
- Produces: an open PR into `main` for the user to merge.

- [ ] **Step 1: Read the whole diff before pushing**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills diff 3949c19..HEAD
```

Expected: three files touched plus the spec and this plan. An unexplained hunk means Step 7 of
Task 1 missed something.

- [ ] **Step 2: Confirm the account, then push**

```bash
gh auth status
```

Expected: `Active account: true` under `arturoortizvn`. On a mismatch, stop and ask the user to
switch; never push from the wrong account.

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills push -u origin update/owd-lean-skill
```

- [ ] **Step 3: Open the PR into `main`**

```bash
cd /Users/arturoortiz/Proyectos/Personales/gusskills && gh pr create --base main --head update/owd-lean-skill --title "refactor: split the delivery cycle into its rules and its rationale" --body-file -
```

The body states: the before and after word counts; that acceptance was an invariant audit rather
than a word count, and its result; each of the six collected items in one line; that no rule
changed; the line **"Review this from a second clone"** — `gh pr checkout` in this working tree
rewrites the live skills through the symlinks (`README.md`); and the pending manual check the user
owns. Close with the session URL.

- [ ] **Step 4: Report and stop**

State the PR URL, the invariant audit's result, the two word counts, and the pending manual check
that was **not** run. Do not merge. Do not offer to merge.

---

## Self-Review

**Spec coverage.** *The split* → Task 1 Steps 2-4; *The six collected items* → Step 5, each with its
exact replacement text; the `README.md` decision → Step 6; the machine-checkable half of
*Verification* → Step 7 and Step 8; the invariant audit → the controller note under Step 9, which
routes it to the task review where the gate belongs; the pending manual check → Task 2 Step 4 as an
explicit non-claim.

**Placeholders.** Step 3 and Step 4 describe a prose rewrite rather than quoting a finished 1000-word
file, which no plan can pre-write without doing the task; every rule that must survive it is
enumerated in the spec's 28-item list, and every piece of *new* text — the description, the cap, the
predicates, the replacement overview sentence, the duty list, the table and the red flags — is
written out verbatim in Step 5. Nothing else is deferred.

**Consistency.** The line numbers in Step 2's table are at `3949c19`, the commit `main` points at, and
Step 1's word count is the guard that they still apply. Step 7's `description` assertion tests item
1 by what the old description did wrong — naming steps and gates — rather than by length alone,
because a short description can still summarise a workflow.
