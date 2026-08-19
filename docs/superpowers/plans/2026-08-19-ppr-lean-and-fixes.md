# `pr-peer-review` Lean Skill and Corrections Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Move `pr-peer-review/SKILL.md`'s argument into a sibling `rationale.md`, correct the four
defects a review found, and add a rationalizations table and red flags — without losing a single
operative rule or directive.

**Architecture:** Two files in the skill directory plus one line in `README.md`. The split and the
four corrections are one task, because three of the corrections land in sentences the split is
rewriting anyway. The task's review is a two-part audit: the spec's 30 invariants, and an imperative
sweep that catches what an invariant list cannot.

**Tech Stack:** Markdown, `python3` for exact-match edits, `grep`/`wc` for assertions, `git`, `gh`
CLI, POSIX symlinks. No test framework applies.

**Spec:** `docs/superpowers/specs/2026-08-19-ppr-lean-and-fixes-design.md`

## Global Constraints

- **All prose in both files is English**; conversation with the user is Spanish (`~/.claude/CLAUDE.md`).
- **Never commit to `main`.** All work lands on `update/ppr-lean-and-fixes`, which already exists and
  holds the spec at `7043594`.
- **Every git command uses `git -C /Users/arturoortiz/Proyectos/Personales/gusskills`, one git
  command per shell call.** The hook resolves the branch from the session's cwd and grades the last
  `-C` in a chained string.
- **No `--no-verify`, no `--force`, no forbidden reset, no `CLAUDE_GIT_OVERRIDE`.**
- **The hook greps the command string, not its intent.** A heredoc writing documentation that
  contains a forbidden command trips it; this bit twice during the previous plan. Keep such tokens on
  separate lines or word around them, and never reach for the override to get a document written.
- **The active `gh` account must be `arturoortizvn`.** Verify, do not assume.
- **No rule may be deleted to hit a word count.** The target is reported, not enforced.

## File Structure

| File | Responsibility | This plan |
|---|---|---|
| `pr-peer-review/SKILL.md` | The rules a reviewer follows | Rewritten, four corrections included |
| `pr-peer-review/rationale.md` | Why each rule has its shape | Created |
| `README.md` | Repo layout | One layout line |
| `docs/superpowers/specs/2026-08-19-ppr-lean-and-fixes-design.md` | The design, the 30 invariants, the sweep | Read-only input, at `7043594` |

## Verification model — read this before Task 1

The delivery cycle's split kept all 28 of its invariants and still shipped two directives into the
companion file, where a reader following the skill never sees them. So acceptance here has three
parts, and the second exists because of that failure:

1. **The invariant list** — the spec's 30 items, one verdict each, derived from the produced file.
2. **The imperative sweep** — from `git show 5d6670c:pr-peer-review/SKILL.md`, every sentence that
   tells the reader to do or not do something must appear in the produced `SKILL.md`. A directive
   that lives only in `rationale.md` is a defect regardless of the invariant list.
3. **Machine checks** — the greps in Step 8.

---

### Task 1: Split the file and correct the four defects

**Files:**
- Modify: `pr-peer-review/SKILL.md`
- Create: `pr-peer-review/rationale.md`
- Modify: `README.md`

**Interfaces:**
- Consumes: the spec's *The split*, *The four corrections, as they land*, and *Verification*.
- Produces: the two files Task 2 ships.

- [ ] **Step 1: Confirm the starting state**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills branch --show-current
git -C /Users/arturoortiz/Proyectos/Personales/gusskills status --short
wc -w /Users/arturoortiz/Proyectos/Personales/gusskills/pr-peer-review/SKILL.md
```

Expected: branch `update/ppr-lean-and-fixes`, clean tree, 1707 words. A different count means the
file moved since the design was written — re-read it first.

- [ ] **Step 2: Build the imperative inventory before you touch anything**

This is the step that prevents the failure the previous split shipped. From the current file, write
every directive sentence — every bold instruction, every `never`, `must`, `always`, `stop`, and every
imperative verb opening a clause — to
`.superpowers/sdd/2026-08-19-ppr-lean-and-fixes/imperatives.md`, one per line, with its line number.
You will check the produced file against this list in Step 9, and the reviewer will check it too.

- [ ] **Step 3: Classify every paragraph**

Two columns — paragraph opener, `rule` or `rationale` — to `classification.md` in the same directory.
The test is: **if this sentence disappeared, would any action change?** If no, it moves. When a
sentence carries both a rule and its reason, the rule stays and only the reason moves; do not move
the whole sentence because part of it is argument.

The design already classifies these as rationale: the "ages in silence" argument after the
no-concrete-paths rule; "Stated as a principle plus examples, because a list of one repo's breaches
does not travel"; "These need no repo doc because they hold anywhere"; "Each line came from a real
finding, and this is the genuinely transferable part"; the "calibration exercise" reason for stopping
on a closed PR; "so the argument stays readable"; the explanation of why the three dispositions are
not interchangeable; and "which is its anti-staleness by construction".

- [ ] **Step 4: Write `rationale.md`**

Eight `##` headings, each naming the rule it explains, in the order those rules appear in `SKILL.md`.
Carry the existing prose across rather than rewriting it. Open with:

```markdown
# Why peer review is shaped this way

Every rule in `SKILL.md` that is not self-evident has its argument here. Read this before relaxing a
rule, not before following one.
```

- [ ] **Step 5: Rewrite `SKILL.md`, keeping its section order**

Add, immediately after the overview:

```markdown
Why each rule has this shape: `rationale.md`, in this directory. Read it before relaxing a rule, not
before following one.
```

- [ ] **Step 6: Land the four corrections, exactly as worded here**

**Correction 1 — `graph-review` does not exist.** Three sites. The frontmatter becomes:

```yaml
description: Use when reviewing someone else's pull request on any repo and the verdict has to land on the PR itself. Also when you need the severity scale for a review, when a repo's own review standards have to be found before grading, or when a review must be posted from the right account. Not for reviewing your own work before opening a PR — that is `superpowers:requesting-code-review`.
```

In *Standards resolution*, the parenthetical keeps its point with the surviving example:

```markdown
   A skill about reviewing your own work before opening a PR (`code-review`) does not count and
   does not satisfy this step.
```

And the closing boundary becomes:

```markdown
Boundary with `superpowers:requesting-code-review`: that one is the author's, before opening the PR.
This one is the reviewer's, on someone else's PR.
```

**Correction 2 — the checkout is conditional.** Procedure step 2's guard paragraph becomes:

```markdown
   Guards: **check the head out only if the tree is not already at it.** A reviewer dispatched into a
   worktree already detached at the PR's head must skip the checkout — run there it dies with
   `fatal: '<branch>' is already used by worktree at '<path>'`, because another worktree holds that
   branch. Where you do check out, the working tree must be **clean** first — otherwise stop and tell
   the user; never clobber their work. If `state` is not `OPEN`, write the comment to a file and
   **stop**.
```

**Correction 3 and 4 — the relocation is required.** *Standards resolution* gains this as its first
numbered item, renumbering the rest:

```markdown
1. **A review skill for this repo that lives outside it stops the review.** Look before grading: a
   skill whose name or description names *this specific repo* belongs in that repo, and while it sits
   anywhere else — the reviewer's own `~/.claude/skills/` is where this happens — it ages in silence
   while nothing in that repo's PRs ever touches it. Say so, and stop: moving it into that repo is
   its own task on its own branch, and the review runs after. A skill that names no repo, this one
   included, never trips this.
```

And step 5 of the procedure keeps only the create case, losing "It is an offer, not a requirement":

```markdown
5. **Offer to write the repo's own review skill** — only when checks were derived because the repo
   had none anywhere. One line at the end, **in the chat, not in the PR comment**; if accepted, that
   work is its own task on its own branch.
```

- [ ] **Step 7: Append the two new sections, verbatim**

```markdown
## Rationalizations

Every row is a real failure of a real review.

| Excuse | Reality |
|--------|---------|
| "The PR body says the guard is covered" | The body is a claim. Verify against the base branch. |
| "The suite passed, so the tests ran" | A suite that skips itself proves nothing. Ask what the variable was set to. |
| "I found nothing on that check, so I left it out" | Silence reads as "checked and fine". Discard it explicitly, with the reason. |
| "The tree scan came back clean" | From the base it always does. Without the head checkout that is a false negative on a Blocker. |
| "I'll add round 2's findings to my first comment" | A second round is a second comment. Never edit or re-post over one. |
| "The repo documents nothing, so I'll grade against good practice" | An invented invariant is not a finding. Derive the checks or say the repo had none. |
| "The PR is closed but my review is written" | Write it to a file and stop. The people who shipped it do not need the notification. |
| "It looks fine, I'll approve it on GitHub" | The verdict is text. Approving through the API and merging are both the user's. |

## Red flags — stop

- You are about to edit or re-post over a previous review comment.
- You are about to quote extracted data or user content instead of citing `file:line`.
- You are about to report a finding you could not substantiate, or drop one silently.
- You are about to run a tree-scanning check without the head checked out.
- You are about to grade against a rule no doc in that repo states.
- You are about to review a repo whose own review skill still lives outside it.
- You are about to approve through `gh pr review --approve`, or to merge.

**Every one of these means: stop and re-read the rule it breaks.**
```

- [ ] **Step 8: Update `README.md`'s layout block**

Replace exactly:

```
  pr-peer-review/SKILL.md
```

with:

```
  pr-peer-review/{SKILL.md,rationale.md}
```

- [ ] **Step 9: Verify — the sweep first, then the machine checks**

Walk the Step 2 inventory line by line against the produced `SKILL.md` and record any directive that
is missing or that now lives only in `rationale.md`. Fix each before moving on; then:

```bash
cd /Users/arturoortiz/Proyectos/Personales/gusskills/pr-peer-review
grep -rn 'graph-review' . ; echo "expect: no output"
grep -c 'superpowers:requesting-code-review' SKILL.md          # expect >= 2
grep -c 'rationale.md' SKILL.md                                # expect >= 1
grep -c '^## ' rationale.md                                    # expect 8
grep -c '^## Rationalizations' SKILL.md                        # expect 1
grep -c '^## Red flags' SKILL.md                               # expect 1
grep -cF "fatal: '<branch>' is already used by worktree at '<path>'" SKILL.md   # expect 1
grep -c 'offer, not a requirement' SKILL.md                    # expect 0
grep -c 'stops the review' SKILL.md                            # expect 1
grep -nE '^description:.*(grades on|posts exactly one|resolves whose)' SKILL.md ; echo "expect: no output"
awk '/^description:/{print length($0)-13}' SKILL.md            # report it
wc -w SKILL.md rationale.md                                    # report both
grep -n 'pr-peer-review/{SKILL.md,rationale.md}' ../README.md  # expect 1 hit
```

The `fatal:` check uses `grep -F` on the complete string on purpose: a bounded pattern is what let a
line-wrapped literal pass this same check on the previous skill.

- [ ] **Step 10: Verify the live skill serves both files**

```bash
head -2 ~/.claude/skills/pr-peer-review/SKILL.md | tail -1
head -1 ~/.claude/skills/pr-peer-review/rationale.md
```

Expected: `name: pr-peer-review`, then the rationale's title line.

- [ ] **Step 11: Commit**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills add pr-peer-review/SKILL.md pr-peer-review/rationale.md README.md
```

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills commit -m "refactor: split peer review into rules and rationale, and correct four defects

The argument for each rule moves to rationale.md and the skill keeps the rules,
the commands and the literal error messages a reviewer meets at the point of
failure.

Four corrections ride with it. graph-review, referenced three times including
the frontmatter's negative trigger, exists nowhere on this machine; it becomes
superpowers:requesting-code-review. The mandated checkout is now conditional,
because a reviewer dispatched into a worktree already at the PR's head hits
'fatal: already used by worktree' when it runs. And the relocation knot is
untied by requiring the move: a review skill naming this specific repo while
living outside it stops the review, which is what makes this skill and the
delivery cycle agree, and what makes a bespoke skill reachable instead of
invisible."
```

**Note for the controller:** this task's review is the two-part audit — the spec's 30 invariants and
the imperative sweep. Require a verdict per invariant and a verdict on the sweep, not a summary.

---

### Task 2: Push, open the PR, hand off

**Files:** none in the repo — the deliverable is a PR.

- [ ] **Step 1: Read the whole diff before pushing**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills diff 5d6670c..HEAD
```

Expected: three files plus the spec and this plan.

- [ ] **Step 2: Confirm the account, then push**

```bash
gh auth status
```

Expected `Active account: true` under `arturoortizvn`; on a mismatch stop and ask for the switch.

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills push -u origin update/ppr-lean-and-fixes
```

- [ ] **Step 3: Open the PR into `main`**

```bash
gh pr create --base main --head update/ppr-lean-and-fixes --title "refactor: split peer review into rules and rationale, and correct four defects" --body-file -
```

The body states: the four defects with their evidence, including that `graph-review` was verified
absent from `~/.claude/skills/`, the plugin cache and both instruction files; that the relocation is
now required and what that costs (an IDAnalyzer PR cannot be reviewed with its bespoke checks until
that skill moves); both word counts; that acceptance was the invariant list **plus** an imperative
sweep, and why the sweep exists; the line **"Review this from a second clone"**; and the pending
manual check the user owns. Close with the session URL.

- [ ] **Step 4: Report and stop**

State the PR URL, the audit result, both word counts, and what was not verified. Do not merge. Do not
offer to merge.

---

## Self-Review

**Spec coverage.** *The split* → Steps 3-5; *The four corrections* → Step 6, each with exact text;
the two new sections → Step 7; the `README.md` decision → Step 8; the invariant list and the sweep →
Step 9 plus the controller note at Step 11; the machine checks → Step 9; the live-symlink check →
Step 10. The spec's out-of-scope items appear nowhere in the tasks, which is correct.

**Placeholders.** Steps 4 and 5 describe a prose rewrite, which no plan can pre-write without doing
the task; every rule that must survive is enumerated in the spec's 30 invariants, and every piece of
*new* text — the description, the checkout guard, the relocation gate, step 5, the table, the red
flags, the rationale opener — is written verbatim here.

**Consistency.** Step 6's relocation gate becomes item 1 of *Standards resolution*, so the existing
items renumber to 2, 3 and 4; the invariant list refers to them by content, not by number, so the
renumbering breaks nothing. Step 9's `stops the review` grep is the gate's own marker phrase, chosen
because it appears in no other rule.

## Reversal

**2026-08-19, during implementation.** The split was dropped. Measurement put the companion at 769
words of newly written prose against only about 118 carried out of `SKILL.md`, which falsified the
design's premise that roughly half the file was argument; the user chose to drop the split rather
than force it to fit. The four corrections and the two new sections shipped in a single
`pr-peer-review/SKILL.md` at 2209 words, with no companion file and `README.md` untouched.

Historical only, kept as the dated record of what was approved: Steps 3, 4 and 8, the `rationale.md`
pointer in Step 5, and Step 9's `rationale.md` assertions. Everything else still binds — the four
corrections as worded, the imperative inventory of Step 2, and the two-part verification.
