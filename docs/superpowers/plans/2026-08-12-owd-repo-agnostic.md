# `orchestrated-worktree-delivery` Repo-Agnostic Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite `orchestrated-worktree-delivery/SKILL.md` so the delivery cycle it describes runs
on any repo, with no board id, repo name, tracker or branch name left in the file.

**Architecture:** One file changes. Four of its eight sections are replaced — *Reviewer resolution*
collapses onto `pr-peer-review`, *Ticket hook* trades TaxReturnAnalyzer's Monday coordinates for a
resolution order plus a no-tracker branch, *Closeout* reads the integration branch from the PR's
base, *Git mechanics* loses its `develop` examples — and the frontmatter description gains that the
cycle applies anywhere. The other four sections are carried over byte-identical, which is the
acceptance criterion that keeps the rewrite from drifting.

**Tech Stack:** Markdown (skill file), `grep`/`awk`/`diff` for assertions, `gh` CLI, `git`, POSIX
symlinks. No test framework applies.

**Verification model — read this before Task 1.** A skill has no test suite, so this plan has no
red/green TDD cycle and does not pretend to have one. Acceptance is still executable: `grep`
assertions over the produced file, a `diff` proving the carried sections are untouched, and
resolution through the live symlink. Two criteria are deliberately precise because their naive form
gives a false result — they are called out where they appear. The one thing only a real run can
prove, that the cycle works on a repo other than TaxReturnAnalyzer, is declared a pending manual
check in Task 2, not claimed.

## Global Constraints

- **All prose in the file is English.** Conversation with the user is Spanish; file content, commit
  messages, branch names and PR titles/bodies are English (`~/.claude/CLAUDE.md`).
- **Never commit to `main`.** All work lands on `update/owd-repo-agnostic`, which already exists and
  already holds the spec at `cafaf03`. `main` is reached only through a PR **the user merges**.
- **Every git command uses `git -C /Users/arturoortiz/Proyectos/Personales/gusskills`.** The
  `protect-git.sh` hook resolves the current branch from the *session's* cwd, not from where the git
  command runs, so `-C` keeps the hook grading the right branch.
- **No `--no-verify`, no `--force`, no `reset --hard`, no `CLAUDE_GIT_OVERRIDE`.**
- **The active `gh` account must be `arturoortizvn`**, the repo owner. Verify, do not assume.
- **The spec is the content source.** `docs/superpowers/specs/2026-08-12-owd-repo-agnostic-design.md`
  holds the replacement text for the changed sections; this plan inlines it so the implementer needs
  only one file open, and the two must not disagree — if they do, the spec wins and the plan is wrong.
- **Zero repo coordinates in the produced file.** No board id, group id, item pattern, repo name,
  tracker name or branch name. Task 1 asserts it.

---

## File Structure

| Path (relative to the clone) | Responsibility | This plan |
|---|---|---|
| `orchestrated-worktree-delivery/SKILL.md` | The delivery cycle, repo-agnostic | Rewritten in Task 1 |
| `docs/superpowers/specs/2026-08-12-owd-repo-agnostic-design.md` | The approved design | Already committed at `cafaf03`; not modified |
| `docs/superpowers/plans/2026-08-12-owd-repo-agnostic.md` | This plan | Committed in Task 1 Step 1 |

No file is created or deleted. `pr-peer-review/SKILL.md` and `README.md` are **not** touched: the
delegation target already exists and the README already carries the `idanalyzer-peer-review`
obligation this rewrite drops from the skill.

**On "rewrite" versus "patch".** The spec chose a rewrite because the changes reach most of the
body. That is a statement about scope, not a demand that the surviving prose be retyped — retyping
it is how it drifts. The implementer may edit in place or write the file out fresh; the acceptance
is identical either way, and Task 1 Step 6 enforces it: the four carried sections must come out
byte-identical to `17966ed`, and the four target sections must have changed.

---

### Task 1: Rewrite the skill repo-agnostic

**Files:**
- Modify: `/Users/arturoortiz/Proyectos/Personales/gusskills/orchestrated-worktree-delivery/SKILL.md`
  — frontmatter `description`, and the sections `## Reviewer resolution`, `## Ticket hook`,
  `## Git mechanics inside a worktree`, `## Closeout`
- Commit (already written): `docs/superpowers/plans/2026-08-12-owd-repo-agnostic.md`

**Interfaces:**
- Consumes: the approved spec at `cafaf03`, and the file's current state at `17966ed`.
- Produces: the rewritten skill. Task 2 pushes it and opens the PR; nothing else consumes it.

**Why this task carries the risk:** this skill is in use today and resolves through a symlink. A
rewrite that loses a load-bearing sentence degrades a working skill silently — there is no test to
catch it. Steps 5 and 6 exist for that.

- [ ] **Step 1: Record the current state, and commit this plan**

```bash
D=/Users/arturoortiz/Proyectos/Personales/gusskills
F=$D/orchestrated-worktree-delivery/SKILL.md
git -C $D branch --show-current          # must print update/owd-repo-agnostic
wc -l $F                                 # 138 today
shasum $F                                # record it; Step 6 does not need it, humans do
git -C $D add docs/superpowers/plans/2026-08-12-owd-repo-agnostic.md
git -C $D commit -m "docs: add implementation plan for the repo-agnostic rewrite"
```

Do not proceed if the branch is anything but `update/owd-repo-agnostic`.

- [ ] **Step 2: Replace the frontmatter `description`**

Old (one line, do not reflow it — YAML):

```yaml
description: Use when work from a plan or backlog is being delivered by dispatched implementers in worktrees — each item gets a ticket, a branch, a PR, and an adversarial peer review before a merge that only the user performs. Covers the review gate, the round cap, the ticket filed before dispatch, and the git mechanics the protect-git hook requires inside a worktree. Not for edits you make yourself, and not keyed to how many items there are.
```

New:

```yaml
description: Use when work from a plan or backlog is being delivered by dispatched implementers in worktrees, on any repo — each item gets a ticket, a branch, a PR, and an adversarial peer review before a merge that only the user performs. Covers the review gate, the round cap, the ticket filed before dispatch, resolving the repo's tracker and integration branch, and the git mechanics the protect-git hook requires inside a worktree. Not for edits you make yourself, and not keyed to how many items there are.
```

`name: orchestrated-worktree-delivery` does not change, and neither does the directory name.

- [ ] **Step 3: Replace `## Reviewer resolution` entirely**

Delete the two-step cascade and the paragraph after it. The whole section becomes:

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

Two things leave with the old text and are meant to: the directory-name test
(`.claude/skills/peer-review/` — `pr-peer-review` tests the *description* instead, so it finds repo
skills the old step 1 missed) and the sentence naming `idanalyzer-peer-review`, whose obligation
lives in `README.md`.

- [ ] **Step 4: Replace `## Ticket hook`**

The three invariant bullets survive verbatim. The `In TaxReturnAnalyzer, the coordinates are…`
paragraph is deleted and replaced. The whole section becomes:

```markdown
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
(root and nested) → `.claude/` → `docs/`. What you need before dispatching: which board or
project, which item type the work maps to, the status and dependency fields, and the field
the PR URL goes in. Those coordinates belong in the repo that uses them — a board id written
here goes stale the moment someone moves it, and nothing in that repo's PRs ever touches this
file.

**No tracker.** Say so once, then run the cycle with the PR as the item of record: its body
carries what the ticket would have, and the review heading carries the branch instead of a
ticket — which is what `pr-peer-review` already does on a repo without that convention. Do not
invent a tracker, and do not skip the step silently: an unfiled item on a repo that *does* have
one is the failure this hook exists to prevent.
```

- [ ] **Step 5: Replace the two `develop` sentences in `## Git mechanics inside a worktree`, and all of `## Closeout`**

In *Git mechanics*, exactly two sentences change. First:

```markdown
`~/.claude/hooks/protect-git.sh` resolves the current branch from the **session's** cwd,
not from where the git command runs. With the main checkout on the integration branch and
the work in a worktree on `feature/*`, the hook reads the integration branch and blocks a
legitimate commit.
```

Second:

```markdown
The hook also blocks `git reset --hard`. To bring a branch up to date after something
else merged into the base, use `gh pr update-branch <n>` then `git pull --ff-only` — no
forbidden command, no force-push.
```

Everything else in that section — `git -C <absolute worktree path>` as the clean exit, and the
`CLAUDE_GIT_OVERRIDE` prohibition — stays exactly as it is.

`## Closeout` becomes, in full:

````markdown
## Closeout
On merge, move the ticket to `Done` in the same batch as the local cleanup. These commands
are the orchestrator's own — per the role boundary, local branch hygiene lands in no diff:

```bash
git switch <the PR's base>      # gh pr view <n> --json baseRefName -q .baseRefName
git pull --ff-only
git branch -d <merged-branch>
git fetch origin --prune
```

The base comes from the PR rather than a branch name written here: this cycle runs on repos
that integrate into `main` and repos that integrate into something else, and the PR already
carries which one.

`-d` is the default, and it verifies the branch really merged before deleting it. Where the
repo squash-merges, git never sees the branch's commits on the base and `-d` refuses — use
`-D` there, and only there. Dead local branches accumulate and invite branching from a stale
base.
````

- [ ] **Step 6: Assert the properties the spec requires**

```bash
D=/Users/arturoortiz/Proyectos/Personales/gusskills
F=$D/orchestrated-worktree-delivery/SKILL.md

grep -nE '18425100702|group_mm5xd2s3|U-04-1040|TaxReturnAnalyzer|idanalyzer|Monday' $F \
  || echo "no repo coordinates: OK"

grep -nE '\bdevelop\b' $F || echo "no develop branch: OK"

grep -n 'pr-peer-review' $F                    # expect exactly 2 — see below
grep -nE 'Blockers|Majors' $F                  # expect 2 — the gate steps, load-bearing
grep -n 'three rounds maximum' $F              # the round cap survived
grep -n '^\*\*No tracker\.\*\*' $F             # the no-tracker branch landed — see below
grep -n 'name: orchestrated-worktree-delivery' $F
sed -n '/^## Ticket hook/,/^## Git mechanics/p' $F | grep -c '^- '   # expect 3
```

**The `develop` grep must be `\bdevelop\b`, never a bare `develop`.** The surviving reference to
`superpowers:subagent-driven-development` contains the substring, so a bare grep returns a hit and
reports a failure that does not exist. On the file as it stands the bare form gives 6 hits and the
bounded form gives 5 — the 5 branch mentions to remove, and the 1 legitimate word to keep.

**Do not grep for `no ticket` as evidence the no-tracker branch landed.** An earlier draft of this
plan did, because an earlier draft of the paragraph contained that string. The review of PR #2
established that the string is wrong there — `pr-peer-review` puts the *branch* in the heading when
the repo has no ticket convention, and reserves `no ticket` for a repo that has the convention but a
PR naming none — so the corrected paragraph does not contain it, and the old criterion now fails on
the correct file. Grep the heading instead.

**Two `pr-peer-review` hits is the pass, not one.** The file today has exactly one, in the old step
2. The rewrite produces two: the dispatch line in *Reviewer resolution*, and the sentence in the
*Ticket hook*'s no-tracker branch that points at what `pr-peer-review` already does with a repo
lacking a ticket convention. Expecting 1 would report a failure on a correct file.

**Two Blockers/Majors hits is the pass, not a leftover.** They are the cycle's review gate ("No
Blockers and no Majors → tell the user they can merge") and its argue step, both inside *The cycle*,
which this rewrite carries over untouched. The gate is *stated* in those terms; deleting them would
break it.

- [ ] **Step 7: Prove the carried sections did not drift**

```bash
D=/Users/arturoortiz/Proyectos/Personales/gusskills
F=$D/orchestrated-worktree-delivery/SKILL.md
OLD=$(mktemp); NEWC=$(mktemp); OLDC=$(mktemp)
git -C $D show 17966ed:orchestrated-worktree-delivery/SKILL.md > $OLD
awk '/^## Overview/{f=1} /^## Reviewer resolution/{f=0} f' $OLD > $OLDC
awk '/^## Overview/{f=1} /^## Reviewer resolution/{f=0} f' $F   > $NEWC
diff $OLDC $NEWC && echo "carried sections byte-identical: OK"
```

That range is *Overview*, *What this skill does not cover*, *Role boundary* and *The cycle* — the
four sections the spec carries over. Any diff here is a failure to fix, not a finding to note. A
non-empty diff means the rewrite touched prose it was not authorised to touch.

- [ ] **Step 8: Verify the live skill still resolves and reads the new text**

```bash
readlink /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery
head -3 /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/SKILL.md
grep -c 'Resolving the tracker' /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/SKILL.md
grep -nE '\bdevelop\b' /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/SKILL.md \
  || echo "live skill is de-named: OK"
```

Expected: the symlink prints, `head` prints the `---` / `name: orchestrated-worktree-delivery`
frontmatter, the count is 1, and the branch grep is empty. This is the check that catches a rewrite
that landed somewhere other than the file the live path resolves to.

- [ ] **Step 9: Commit**

```bash
git -C $D add orchestrated-worktree-delivery/SKILL.md
git -C $D commit -m "refactor: make the delivery cycle repo-agnostic"
```

---

### Task 2: Push, open the PR, hand off

**Files:** no file changes. Remote branch and PR only.

**Interfaces:**
- Consumes: the two commits from Task 1 plus the spec commit `cafaf03`.
- Produces: a PR into `main` for the user to merge. **Do not merge it.**

- [ ] **Step 1: Review the whole diff before pushing**

```bash
D=/Users/arturoortiz/Proyectos/Personales/gusskills
git -C $D log --oneline main..HEAD
git -C $D diff main..HEAD --stat
git -C $D status --short
```

Expected: three commits (spec, plan, rewrite); the stat lists only
`orchestrated-worktree-delivery/SKILL.md` and the two `docs/superpowers/` files; `status` is clean.
An untracked file here means something was written outside the plan — report it rather than sweeping
it into a commit.

- [ ] **Step 2: Confirm the account, then push**

```bash
gh auth status 2>&1 | grep -B1 'Active account: true'
git -C $D push -u origin update/owd-repo-agnostic
```

The active account must be `arturoortizvn`. On a mismatch, stop and ask for the switch.

- [ ] **Step 3: Open the PR into `main`**

```bash
cd /Users/arturoortiz/Proyectos/Personales/gusskills
gh pr create --base main --head update/owd-repo-agnostic \
  --title "Make orchestrated-worktree-delivery repo-agnostic" \
  --body "$(cat <<'EOF'
Takes the last repository out of the delivery cycle, so it runs anywhere.

**What changed**
- *Reviewer resolution* collapses from a two-step cascade to one line: dispatch `pr-peer-review`,
  which already resolves whose standards apply. This dissolves the Nit raised in PR #1's review
  rather than patching it — the step that held it stops existing — and it fixes a real gap, since
  the old step 1 tested a directory name while `pr-peer-review` tests the description.
- *Ticket hook* loses TaxReturnAnalyzer's Monday coordinates and gains a resolution order plus the
  case the file never had: a repo with **no tracker at all**, which is why this cycle could not run
  in this repo until now.
- *Closeout* reads the integration branch from the PR's base instead of naming `develop`, and `-d`
  becomes the default with `-D` conditional on the repo squash-merging. That claim was stated as
  universal and this repo disproved it: PR #1 merged with a merge commit and `-d` worked.
- *Git mechanics* loses its two `develop` examples. The `CLAUDE_GIT_OVERRIDE` prohibition and the
  `git -C` rule are untouched.

**Not here, on purpose:** TaxReturnAnalyzer's coordinates are not migrated into that repo — separate
repo, separate branch, separate PR. They are recoverable from
`orchestrated-worktree-delivery/SKILL.md` at \`17966ed\`.

**Verified**
- Zero repo coordinates: no board id, group id, item pattern, repo or tracker name.
- Zero branch names: `grep -nE '\bdevelop\b'` is empty. The bounded form matters — a bare
  `develop` grep also hits `superpowers:subagent-driven-development` and reports a failure that is
  not one (6 hits versus 5 before the rewrite).
- The four carried sections are byte-identical to \`17966ed\` by `diff`, so the rewrite touched no
  prose it was not authorised to touch.
- The review gate survives: its two Blockers/Majors references are load-bearing and still present.
- The skill reads through its `~/.claude/skills/` symlink with the new text and unchanged
  frontmatter `name`.

**Pending manual check:** a cycle run on a repo that is not TaxReturnAnalyzer — the case this
rewrite exists to serve. None was available while writing it, so it is declared rather than claimed.
EOF
)"
```

**Executed with `--body-file` instead.** The nested heredoc above needs backticks escaped, and an
escaping slip corrupts the PR body silently. The run wrote this body to a file and passed
`--body-file`, same content, no escaping. Prefer that; the block above is kept as the record of what
was planned.

- [ ] **Step 4: Report and stop**

Give the user the PR URL and the pending manual check. **The merge is the user's** — do not merge,
do not switch `main`, do not delete the branch. The PR's next step is a review by `pr-peer-review`,
which is the cycle this PR describes, applied to itself.

---

## Self-Review

**Spec coverage.** Every section of the spec maps to a step. "The changes" §1 → Task 1 Step 3; §2 →
Step 4; §3 and §4 → Step 5; the frontmatter sentence → Step 2; the carried-sections list → Step 7's
`diff`. "Verification" → Steps 6, 7 and 8, with the pending real run declared in Task 2 Step 3's PR
body and Step 4's handoff. "Out of scope / follow-up" → stated in the PR body; no step touches
TaxReturnAnalyzer, IDAnalyzer, `README.md` or `pr-peer-review/SKILL.md`.

**Placeholder scan.** No TBD/TODO. Every replacement section is inlined in full rather than
referenced, so the implementer never has to open the spec to know what to type. The angle brackets
that remain — `<the PR's base>`, `<merged-branch>`, `<n>`, `<absolute worktree path>` — are the
skill's own parameter placeholders, addressed to whoever later runs the cycle, and they are meant to
survive into the file verbatim.

**Criterion consistency.** All three criteria whose naive form gives a false result are stated with
their correct form and the reason: `\bdevelop\b` rather than `develop`, two `pr-peer-review` hits
rather than one, and two Blockers/Majors hits as the pass rather than a leftover (all Step 6). Each
was checked against the real file while writing this plan rather than reasoned about — the
`pr-peer-review` count was wrong on the first pass and the check is what caught it. The commit `17966ed` is the same baseline in Step 7
and in the PR body. The branch `update/owd-repo-agnostic` is spelled identically in the Global
Constraints, Task 1 Step 1 and Task 2 Steps 2 and 3. The section names match the file's actual
headings, including `## Git mechanics inside a worktree` in full.

**Ordering constraint worth restating:** Step 7's `diff` must run after all of Steps 2-5, not
between them. Run mid-rewrite it compares a half-edited file and fails for the wrong reason.
