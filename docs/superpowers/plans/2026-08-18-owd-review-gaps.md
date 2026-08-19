# `orchestrated-worktree-delivery` Review Gaps Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Amend `orchestrated-worktree-delivery/SKILL.md` so the five remaining gaps from the
review of `057a24a` are closed: the integration branch gets resolved, the push gets an owner, the
two review loops get divided, the precedence rule stops being a closed list, and the reviewer gets
a tree to work in.

**Architecture:** One file changes. Six edits: one new section (`## Integration branch`), two
rewritten cycle steps (4 and 7), two new paragraphs in *Reviewer resolution*, one rewritten
*Precedence* paragraph, plus two riders inside sentences already being touched. Four sections —
*Overview*, *Role boundary*, *Git mechanics inside a worktree* and *Closeout* — are carried over
byte-identical, which is the acceptance criterion that keeps the amendment from drifting.

**Tech Stack:** Markdown (skill file), `python3` for exact-match replacement, `grep`/`awk`/`diff`
for assertions, `git`, `gh` CLI, POSIX symlinks. No test framework applies.

**Spec:** `docs/superpowers/specs/2026-08-18-owd-review-gaps-design.md`

## Global Constraints

- **All prose in the file is English.** Conversation with the user is Spanish; file content, commit
  messages, branch names and PR titles/bodies are English (`~/.claude/CLAUDE.md`).
- **Never commit to `main`.** All work lands on `update/owd-review-gaps`, which already exists and
  already holds the spec at `c95f4bf`. `main` is reached only through a PR **the user merges**.
- **Every git command uses `git -C /Users/arturoortiz/Proyectos/Personales/gusskills`, and one git
  command per shell call.** The `protect-git.sh` hook resolves the branch from the session's cwd,
  and where a single shell call chains several git commands it grades the **last** `-C` in the
  string — so a chain that ends anywhere else silently changes which branch is graded. This is the
  rule the file itself gained at `057a24a`; apply it while editing it.
- **No `--no-verify`, no `--force`, no `reset --hard`, no `CLAUDE_GIT_OVERRIDE`.**
- **The active `gh` account must be `arturoortizvn`**, the repo owner. Verify, do not assume.
- **The replacement method is exact-match with an assertion.** Every edit below is applied with a
  `python3` script that asserts the old text appears exactly once before replacing it. A silent
  no-op edit is the failure mode this prevents; `sed` in-place does not catch it.

## File Structure

| File | Responsibility | This plan |
|---|---|---|
| `orchestrated-worktree-delivery/SKILL.md` | The delivery cycle | Modified — six edits, four sections carried unchanged |
| `docs/superpowers/specs/2026-08-18-owd-review-gaps-design.md` | The approved design | Read-only input, already committed at `c95f4bf` |
| `~/.claude/skills/orchestrated-worktree-delivery` | Symlink that makes the file the live skill | Verified, never written |

## Verification model — read this before Task 1

A skill has no test suite, so this plan has no red/green TDD cycle and does not pretend to have
one. Acceptance is still executable in three forms, and each has its own step below:

1. **`grep` assertions** over the produced file — each new rule present, each removed literal gone.
2. **A `diff` against `057a24a`** proving the four carried sections are byte-identical.
3. **A real scenario** (Task 2) proving the arrangement the new text prescribes actually holds:
   an implementer worktree and a detached review worktree coexisting on one head, and the
   *Closeout* sequence still closing clean behind both.

The one thing only a real run can prove — the cycle working end to end on a repo with a non-`main`
integration branch and a tracker — is declared a pending manual check in Task 3, not claimed.

---

### Task 1: Amend the skill

**Files:**
- Modify: `orchestrated-worktree-delivery/SKILL.md`

**Interfaces:**
- Consumes: the proposed text in the spec's *The changes* section, verbatim.
- Produces: the amended `SKILL.md` that Task 2 exercises and Task 3 ships.

- [ ] **Step 1: Confirm the starting state**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills branch --show-current
git -C /Users/arturoortiz/Proyectos/Personales/gusskills status --short
git -C /Users/arturoortiz/Proyectos/Personales/gusskills log --oneline -2
wc -w /Users/arturoortiz/Proyectos/Personales/gusskills/orchestrated-worktree-delivery/SKILL.md
```

Expected: branch `update/owd-review-gaps`, clean tree, `c95f4bf` on top of `057a24a`, 1732 words.
A different word count means the file moved since the design was written — stop and re-read it
before editing.

- [ ] **Step 2: Invert the *Precedence* paragraph**

Replace exactly:

```markdown
**Precedence.** The delegation covers the *how* of dispatch. Where this cycle and any skill
it invokes disagree, this cycle governs four things: the review gate, the round cap, when
the PR is opened, and the closeout. Everything else — how to brief, which model, how to
resume an implementer, how to handle its report — follows the invoked skill. This is stated
without version numbers or quoted values from another skill on purpose, so it stays true
after that skill changes.
```

with:

```markdown
**Precedence.** The delegation covers the *how* of dispatch. **Where this cycle states a rule
and an invoked skill says otherwise, this cycle governs; where this cycle is silent, the
invoked skill governs.** It is a principle rather than a list of governed topics because the
list was already incomplete — the role boundary and the ticket's ownership bind exactly as
hard as the review gate, the round cap, when the PR is opened and the closeout, and every rule
this file gains would have to be added to it again. What this cycle does not state — how to
brief, which model, how to resume an implementer, how to handle its report — follows the
invoked skill. None of this quotes version numbers or values from another skill, on purpose,
so it stays true after that skill changes.
```

- [ ] **Step 3: Rewrite cycle step 4 — the push, and rider B**

Replace exactly:

```markdown
4. **The orchestrator opens the PR**, puts its URL on the ticket and moves the ticket to
   `Code Review`, then dispatches the reviewer (see *Reviewer resolution*).
```

with:

```markdown
4. **The orchestrator gets the branch pushed and the PR opened** —
   `superpowers:finishing-a-development-branch`, capped at push and PR: no menu, no local
   merge, no merge at all. Then the PR's URL goes on the ticket, the ticket moves to the
   repo's review status, and the reviewer is dispatched (see *Reviewer resolution*).
```

Two changes ride in one replacement: the delegation is named and capped, and the literal
`Code Review` becomes "the repo's review status" (rider B).

- [ ] **Step 4: Rewrite cycle step 7 — the fix round's re-push, and what the cap counts**

Replace exactly:

```markdown
7. **Re-dispatch the implementer with the open findings** if changes are needed. Rounds 2
   and 3 reuse the same branch, the same worktree and the same PR — never open a second PR
   for one item, and each round's review is a new comment on that PR. *How* a fix round
   runs is the invoked dispatch skill's business; **how many is this cycle's: three rounds
   maximum**, and on the third escalate both positions to the user and stop.
```

with:

```markdown
7. **Re-dispatch the implementer with the open findings** if changes are needed. Rounds 2
   and 3 reuse the same branch, the same worktree and the same PR — never open a second PR
   for one item, and each round's review is a new comment on that PR. **The round's commits
   reach the PR before the reviewer is re-dispatched**, by the same capped delegation as
   step 4; a reviewer sent at a stale head comments on code that no longer exists. *How* a
   fix round runs is the invoked dispatch skill's business; **how many is this cycle's:
   three PR review rounds maximum**, and on the third escalate both positions to the user
   and stop.
```

Do not touch "on the third escalate": its ambiguity is a declared out-of-scope item in the spec,
and resolving it in passing is scope the user did not approve.

- [ ] **Step 5: Add the two new paragraphs to *Reviewer resolution***

They go after the "Every review round posts its own comment" paragraph and before the paragraph
that begins "The convention that makes one step enough". Anchor the insertion on that last line —
replace exactly:

```markdown
The convention that makes one step enough:
```

with:

```markdown
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

The convention that makes one step enough:
```

- [ ] **Step 6: Insert the `## Integration branch` section**

It goes between *Ticket hook* and *Git mechanics inside a worktree*. Anchor on the heading that
follows it — replace exactly:

```markdown
## Git mechanics inside a worktree
```

with:

```markdown
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
```

- [ ] **Step 7: Rider A — make the two resolution orders identical**

Replace exactly:

```markdown
**Resolving the tracker.** Read it from the repo's own written norms, in order: `CLAUDE.md`
(root and nested) → `.claude/` → `docs/`. What you need before dispatching: which board or
```

with:

```markdown
**Resolving the tracker.** Read it from the repo's own written norms, in order: `CLAUDE.md`
(root and nested) → `.claude/` → `CONTRIBUTING.md` → `docs/`. What you need before dispatching:
which board or
```

- [ ] **Step 8: Apply steps 2-7 as one asserted script**

Write the six replacements into a single script so a silent no-op cannot pass. Each pair is the
verbatim old and new text from the steps above.

```bash
cd /Users/arturoortiz/Proyectos/Personales/gusskills && python3 - <<'PY'
p = 'orchestrated-worktree-delivery/SKILL.md'
s = open(p).read()

def sub(old, new):
    global s
    assert s.count(old) == 1, f"expected 1 match, got {s.count(old)} for: {old[:70]!r}"
    s = s.replace(old, new)

# ... one sub(old, new) call per edit from Steps 2-7, in that order ...

open(p, 'w').write(s)
print("all six edits applied")
PY
```

Expected: `all six edits applied`. An `AssertionError` means the anchor text does not match the
file byte for byte — re-read the file at that spot and fix the anchor, never loosen the assertion.

- [ ] **Step 9: Assert the properties the spec requires**

```bash
cd /Users/arturoortiz/Proyectos/Personales/gusskills/orchestrated-worktree-delivery
grep -c '^## Integration branch' SKILL.md          # expect 1
grep -n 'Code Review' SKILL.md                     # expect no output, exit 1
grep -c 'three PR review rounds maximum' SKILL.md  # expect 1
grep -c 'where this cycle is silent' SKILL.md      # expect 1
grep -c "The round's commits" SKILL.md             # expect 1
grep -c 'finishing-a-development-branch' SKILL.md  # expect 2 (step 4 and Closeout)
grep -c 'Two review layers, and only two' SKILL.md # expect 1
grep -c 'gets a worktree of its own' SKILL.md      # expect 1
tr '\n' ' ' < SKILL.md | grep -oE '`CLAUDE\.md` \(root and nested\) → `\.claude/` → `CONTRIBUTING\.md` → `docs/`' | wc -l   # expect 2
grep -nE '18425100702|TaxReturnAnalyzer|idanalyzer|Monday|\b(develop|master)\b' SKILL.md  # expect none
head -2 SKILL.md | tail -1                         # expect: name: orchestrated-worktree-delivery
wc -w SKILL.md                                     # expect ~2100
```

The `tr | grep -o | wc -l` line is the rider-A check, and it is written that way for two reasons
that were verified before this plan shipped: each order wraps across a newline, so the file has to
be flattened first, and a looser character class cannot be used because `.claude/` contains the
dot it would have to cross. One literal pattern matching **twice** is the proof that both orders
are identical — anything other than `2` means they drifted apart again, which is the discrepancy
the rider exists to remove.

- [ ] **Step 10: Prove the carried sections did not drift**

```bash
cd /Users/arturoortiz/Proyectos/Personales/gusskills
git -C . show 057a24a:orchestrated-worktree-delivery/SKILL.md > "${TMPDIR:-/tmp}/owd-base.md"
sec() { awk -v h="$2" '$0 == h {p=1; next} p && /^## / {exit} p' "$1"; }
for h in '## Overview' '## Role boundary' '## Git mechanics inside a worktree' '## Closeout'; do
  if diff -q <(sec "${TMPDIR:-/tmp}/owd-base.md" "$h") <(sec orchestrated-worktree-delivery/SKILL.md "$h") >/dev/null; then
    echo "unchanged: $h"
  else
    echo "DRIFTED: $h"; diff <(sec "${TMPDIR:-/tmp}/owd-base.md" "$h") <(sec orchestrated-worktree-delivery/SKILL.md "$h")
  fi
done
```

Expected: four `unchanged:` lines and nothing else. Any `DRIFTED:` line is a real failure — this
amendment is not allowed to touch those four sections, and a stray edit there is exactly what the
check exists to catch. Note `## Git mechanics inside a worktree` is compared as a section: its
heading is also the anchor Step 6 used for the insertion, and the insertion must land *above* it.

- [ ] **Step 11: Verify the live skill still resolves and serves the new text**

```bash
head -2 ~/.claude/skills/orchestrated-worktree-delivery/SKILL.md | tail -1
grep -c '^## Integration branch' ~/.claude/skills/orchestrated-worktree-delivery/SKILL.md
```

Expected: `name: orchestrated-worktree-delivery`, then `1`. This is the one check that can break a
skill in use today, so it runs against the live path, not the repo path.

- [ ] **Step 12: Commit**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills add orchestrated-worktree-delivery/SKILL.md
```

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills commit -m "fix: close the five gaps the repo-agnostic rewrite left in the cycle

The integration branch is now resolved from the repo's norms before the first
dispatch instead of being promised in the frontmatter and never defined. The
push is delegated to finishing-a-development-branch, named and capped at push
plus PR, and each fix round re-pushes before the reviewer is re-dispatched. The
dispatch skill's per-task review stays and its whole-branch final review does
not run, with the three-round cap counting PR rounds only. Precedence becomes a
principle rather than a list that was already missing the role boundary and the
ticket's ownership. And the reviewer gets its own detached worktree, because a
checkout in the orchestrator's tree fails outright while the implementer's
worktree holds the branch."
```

Two separate shell calls: one git command per call, per the global constraints.

---

### Task 2: Prove the arrangement the new text prescribes

**Files:** none — the deliverable is a transcript, and nothing here is committed.

**Interfaces:**
- Consumes: the amended `SKILL.md` from Task 1.
- Produces: evidence that the reviewer worktree, the implementer worktree and the *Closeout*
  sequence coexist, to be quoted in Task 3's PR body.

- [ ] **Step 1: Build the scenario**

A throwaway repo standing in for a real one: an integration branch, an implementer worktree on a
feature branch, one commit in it. Use a `feature/*` name for the integration stand-in — the hook
blocks commits on `main`, `master` and `develop`, and this repo is the wrong place to argue with
it.

```bash
SB="${TMPDIR:-/tmp}/owd-scenario"; rm -rf "$SB"; mkdir -p "$SB"
git init -q -b feature/base "$SB/base"
git -C "$SB/base" config user.email t@t
git -C "$SB/base" config user.name t
echo a > "$SB/base/a.txt"
git -C "$SB/base" add -A
git -C "$SB/base" commit -qm init
git -C "$SB/base" branch feature/item
git -C "$SB/base" worktree add -q "$SB/impl" feature/item
echo work >> "$SB/impl/a.txt"
git -C "$SB/impl" commit -qam work
```

- [ ] **Step 2: Reproduce the failure the new paragraph cites**

```bash
SB="${TMPDIR:-/tmp}/owd-scenario"
git -C "$SB/base" checkout feature/item; echo "exit=$?"
```

Expected: `fatal: 'feature/item' is already used by worktree at '<...>/impl'`, `exit=128`. This is
the sentence quoted in *Reviewer resolution*; if it does not reproduce, the paragraph is wrong and
Task 1 must be corrected rather than the check relaxed.

- [ ] **Step 3: Show the detached review worktree succeeds where the checkout failed**

```bash
SB="${TMPDIR:-/tmp}/owd-scenario"
git -C "$SB/base" worktree add -q --detach "$SB/review" feature/item; echo "exit=$?"
git -C "$SB/base" worktree list
```

Expected: `exit=0`, and three worktrees listed — base on `feature/base`, `impl` on
`feature/item`, `review` detached at the same commit as `impl`.

- [ ] **Step 4: Show the reviewer's mess stays in the reviewer's tree**

```bash
SB="${TMPDIR:-/tmp}/owd-scenario"
echo "test output" > "$SB/review/scratch.log"
git -C "$SB/impl" status --porcelain -uall
```

Expected: empty output. The implementer's worktree is untouched by anything the reviewer does,
which is the reason the design refused to seat the reviewer there.

- [ ] **Step 5: Remove the review worktree, then run the *Closeout* sequence**

```bash
SB="${TMPDIR:-/tmp}/owd-scenario"
git -C "$SB/base" worktree remove --force "$SB/review"; echo "review removed exit=$?"
git -C "$SB/base" merge -q --no-ff feature/item -m "merge (stands in for the PR merge)"
git -C "$SB/base" worktree remove "$SB/impl"; echo "impl removed exit=$?"
git -C "$SB/base" worktree prune
git -C "$SB/base" branch -d feature/item; echo "branch -d exit=$?"
git -C "$SB/base" worktree list
```

Expected: both removals `exit=0`, `Deleted branch feature/item`, and one worktree left. `--force`
is correct on the review worktree and only there: its scratch file is throwaway by construction,
which is precisely what *Closeout*'s guard says is **not** true of the implementer's tree — that
one is removed without `--force`, and a refusal there goes to the user.

- [ ] **Step 6: Clean up and report**

```bash
rm -rf "${TMPDIR:-/tmp}/owd-scenario"
```

Report the transcript of Steps 2, 3 and 5 verbatim. Nothing is committed in this task.

---

### Task 3: Push, open the PR, hand off

**Files:** none in the repo — the deliverable is a PR.

**Interfaces:**
- Consumes: the commit from Task 1 and the transcript from Task 2.
- Produces: an open PR into `main` for the user to merge.

- [ ] **Step 1: Review the whole diff before pushing**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills diff 057a24a..HEAD -- orchestrated-worktree-delivery/SKILL.md
```

Read it. Expected: six edits and nothing else in the skill file. An unexplained hunk means Step 10
of Task 1 missed something.

- [ ] **Step 2: Confirm the account, then push**

```bash
gh auth status
```

Expected: `Active account: true` under `arturoortizvn`. On a mismatch, stop and ask the user to
switch (`gh auth switch`) — never push from the wrong account.

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills push -u origin update/owd-review-gaps
```

- [ ] **Step 3: Open the PR into `main`**

```bash
cd /Users/arturoortiz/Proyectos/Personales/gusskills && gh pr create --base main --head update/owd-review-gaps --title "fix: close the five gaps the repo-agnostic rewrite left in the delivery cycle" --body-file -
```

The body states, one section each: what each of the five gaps was and how it is closed; the
`fatal: ... is already used by worktree at ...` transcript from Task 2 as the evidence for the
reviewer-worktree paragraph; the two riders and why they rode along; what was verified and what was
not; and the line **"Review this from a second clone"** — `gh pr checkout` in this working tree
rewrites the live skills through the symlinks (`README.md`). Close with the session URL.

- [ ] **Step 4: Report and stop**

State plainly: the PR URL, the assertions that passed with their output, and the pending manual
check that was **not** run — a full cycle on a repo with a non-`main` integration branch and a
tracker. Do not merge. Do not offer to merge. The merge is the user's, and on this repo it is the
owner's alone.

---

## Self-Review

**Spec coverage.** Each numbered change in the spec's *The changes* maps to a step: change 1 →
Task 1 Step 6; change 2 → Step 3; change 3 → Step 4; change 4 and 5 → Step 5; change 6 → Step 2;
riders → Steps 3 and 7. The spec's *Verification* maps to Task 1 Steps 9-11 (greps, drift diff,
live symlink) and Task 2 (the scenario). The spec's pending manual check is carried into Task 3
Step 4 as an explicit non-claim.

**Placeholders.** Step 8 of Task 1 deliberately leaves the body of the script as a pointer back to
Steps 2-7 rather than repeating six pairs of text a third time; every pair is written verbatim in
the step that introduces it, in the order the script must apply them. Nothing else in the plan
defers content.

**Commands verified before shipping this plan.** Every anchor in Steps 2-7 was matched against
`SKILL.md` at `c95f4bf` and each appears exactly once. The drift check in Step 10 was run as
written and reports four `unchanged:` lines with non-empty section bodies (8, 14, 23 and 28
lines), so it cannot pass vacuously. The rider-A assertion was tested against a sample of the
post-change text: the first form drafted returned `0` and was replaced with the flattened one,
which returns `2`.

**Consistency.** The anchor strings in Steps 2-7 were taken from the file at `057a24a` and their
uniqueness is what Step 8's assertion enforces at run time. Step 5 and Step 6 both work by
replacing a following anchor rather than appending to a preceding one, because a trailing paragraph
has no stable end-of-section marker. Step 10 compares `## Git mechanics inside a worktree` as a
section after Step 6 used its heading as an insertion anchor — the insertion goes above the
heading, so the section's own body is unaffected and the comparison stays valid.
