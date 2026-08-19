---
name: pr-peer-review
description: Use when reviewing someone else's pull request on any repo and the verdict has to land on the PR itself. Also when you need the severity scale for a review, when a repo's own review standards have to be found before grading, or when a review must be posted from the right account. Not for reviewing your own work before opening a PR — that is `superpowers:requesting-code-review`.
---

# Peer review — any repo

## Overview

Reviews someone else's pull request on any repository and delivers the result as **exactly one
PR comment**. It carries the half of peer review that transfers between repos — the severity
scale, the comment format, the posting mechanics, the discipline that keeps findings honest —
while the half that does not, the checks citing a repo's own paths, seams and invariants, comes
from that repo.

**This skill names no concrete repository path, ever.** The moment a check you want needs a real
path or line number, that check belongs in the repo's own review skill — not in this file.

Why each rule has this shape: `rationale.md`, in this directory. Read it before relaxing a rule,
not before following one.

## Standards resolution

Step 1 of the procedure, and it decides everything after it.

1. **A review skill for this repo that lives outside it stops the review.** Look before grading:
   a skill whose name or description names *this specific repo* belongs in that repo, and while
   it sits anywhere else — the reviewer's own `~/.claude/skills/` is where this happens — it ages
   in silence while nothing in that repo's PRs ever touches it. Say so, and stop: moving it into
   that repo is its own task on its own branch, and the review runs after. A skill that names no
   repo, this one included, never trips this.
2. **The repo has a review skill of its own → grade with its checks.** Invoke it where the
   session can — a repo-local skill only reaches the skills listing when the session's cwd is
   that repo, so from anywhere else, read the file and follow it. This skill then contributes
   only what that one lacks (the `gh` mechanics, the discipline) and never competes with it.
   What counts: any skill under the repo's `.claude/skills/` whose name or description is about
   reviewing pull requests **on that repo** — `peer-review` is the convention, but the test is
   the description, not the directory name.
   A skill about reviewing your own work before opening a PR (`code-review`) does not count and
   does not satisfy this step.
3. **No such skill → derive the checks** from the repo's own written norms, in this order:
   `CLAUDE.md` (root and nested) → `CONTRIBUTING.md` → `docs/` (invariants, ADRs, architecture)
   → design specs under `docs/` → the CI workflow definitions (what CI *actually* gates, not
   what the README claims) → `.env.example` → the test layout (which suites are hermetic, which
   skip themselves).
4. **Write the derived list before grading** — N checks, each with its severity and the doc
   line that backs it. An invariant you invented is not a finding. The comment states which
   norms it graded against, and says so plainly when the repo had none.

## Severity scale

- **🔴 Blocker** — merging it damages the service or makes the repo's own documentation lie:
  breaks a documented contract, seam or invariant without updating the doc that asserts it;
  leaks secrets or PII; makes the test suite non-hermetic; contradicts an approved spec.
- **🟠 Major** — must be fixed, does not by itself break the service: architecture deviations,
  new logic with no tests, a new env var with no fail-fast check or no `.env.example` entry,
  blocking work called straight from an `async` handler, unquantified new per-request cost.
- **🟡 Minor** — code smells, naming, dead branches, docs left stale by an additive change,
  error messages that send an integrator round in a loop.
- **🔵 Nit** — style, formatting, micro-optimisations. Never a reason to withhold approval.

## Universal checks

They apply *on top of* the derived ones. **Precedence: where the repo has its own review skill, its
checks and its severities govern** — these only cover what that skill is silent about.

1. **Secrets** — no key, token or connection string in code, tests, examples, docs or
   infrastructure-as-code params. 🔴
2. **PII and user data** — no extracted data in logs, error messages or fixtures, **and none in
   the review comment**. Cite `file:line`, never content. 🔴
3. **Tests** — new logic ships with tests that run in the default suite. A suite that skips
   itself (no database, no key) **is not evidence that it ran**: ask what the variable was set
   to. 🟠, and 🔴 if the PR disables or loosens a guard.
4. **Doc–code coherence** — where the repo documents its contract, response shape or security
   posture, making that doc false without updating it is 🔴; an additive change that leaves it
   stale is 🟡.
5. **Config** — a new env var needs a fail-fast check and an `.env.example` entry. 🟠
6. **Branch flow and merge method** per the repo's own `CONTRIBUTING.md` / `CLAUDE.md`; compare
   `baseRefName` against the branch type. 🔴 when it targets the wrong branch.
7. **Cost and latency** of new calls in the request path; an unbounded retry loop is 🔴.
8. **Ticket** in the PR body where the repo has that convention; if absent, write `no ticket` in
   the heading and raise it as 🟡.

## Discipline

- Verify the PR body's claims **against the base branch** (`git show origin/<base>:<path>`),
  never against the PR's description of itself.
- **Test guards by mutating them.** If the PR adds one, find the test that fails when it is
  removed.
- Run experiments in a **clean tree**, or declare the result provisional.
- **Discard unsubstantiated findings explicitly, with the reason** → they go under *Checked and
  cleared*.
- Tree-scanning checks mean nothing without the head checkout.

## Procedure

Scope: GitHub PRs via `gh`. No GitHub remote or no PR → say so and refer to
`superpowers:requesting-code-review`; there is nowhere to post. The comment is written in
**English**, because it is PR content.

1. **Resolve standards** (above) and write the check list.

2. **Read the PR, then work from its head.**

   ```bash
   R=<owner/repo>
   gh pr view <n> --repo $R --json number,title,body,state,headRefName,baseRefName,files,url
   gh pr diff <n> --repo $R
   gh pr checkout <n> --repo $R      # only where the tree is not already at the head
   ```

   Guards: **check the head out only if the tree is not already at it.** A reviewer dispatched
   into a worktree already detached at the PR's head must skip the checkout — run there it dies
   with `fatal: '<branch>' is already used by worktree at '<path>'`, because another worktree
   holds that branch. Where you do check out, the working tree must be **clean** first —
   otherwise stop and tell the user; never clobber their work. If `state` is not `OPEN`, write
   the comment to a file and **stop**.

3. **Grade** against the derived checks plus the universal ones. An optional first pass with
   `/code-review <n>` is allowed for a correctness sweep, but **its output is never posted raw**
   — its findings go through the scale and the substantiation rule like any other.

4. **Post exactly one comment.**

   ```bash
   gh auth status                                  # active account must match the repo owner
   gh pr comment <n> --repo $R --body-file <file>
   ```

   Identity per `~/.claude/personal-projects.md`: work repos → the work account; personal repos
   → the personal one. On a mismatch, **stop and ask for the switch** (`gh auth switch`); never
   post from the wrong account. No issues, no email, no Slack. **Never edit or re-post over a
   previous comment** — a second round is a second comment.

5. **Offer to write the repo's own review skill** — only when checks were derived because the
   repo had none anywhere. One line at the end, **in the chat, not in the PR comment**; if
   accepted, that work is its own task on its own branch.

## Output

```markdown
## Peer review — <ticket | branch>, <short title>

<2–4 lines: what the PR does, and the verdict with its reason>

### 🔴 Blockers
_None._

### 🟠 Major
- `path/file.py:NN` — <finding>. <one-line fix>

### 🟡 Minor
_None._

### 🔵 Nits
- `path/file.py:NN` — <finding>

### Checked and cleared
- <check> — <what was run, and what it showed>

### Could not verify
- <check> — <why it could not be run>

### Not applicable
- <check> — <why it cannot apply to this repo>

### ✅ What looks good
- <1–3 specific things>

**Verdict:** Approve | Comment | Request changes — <one line>
```

The heading carries the ticket where the repo has a ticket convention, the branch
(`headRefName`) where it has none, and the literal `no ticket` where the convention exists but
the PR body names none — which is also the 🟡 of universal check 8.

All four severity headers are always present, `_None._` under the empty ones, so a reader can
tell "no blockers" from "blockers not considered". Every finding cites `file:line`.

The three dispositions are not interchangeable, and no check is left out of all three: *Checked and
cleared* is "I ran it and there was nothing", *Could not verify* is "I could not run it", and *Not
applicable* is "it cannot apply here". A docs-only repo has no test suite, no env vars and no
request path, so its universal checks 3, 5 and 7 go under *Not applicable*, never stretched into
either of the first two. Drop a heading only when nothing landed under it.

**Verdict logic:**

| Findings | Verdict |
|---|---|
| any 🔴 | Request changes |
| 🟠 without 🔴 | Comment — spell the decision out in the summary |
| only 🟡 / 🔵, or none | Approve |

A pure-formatting diff gets only `Auto-formatting only — no review needed.` and stops.

## What this skill does not do

Does not merge. Does not approve through `gh pr review --approve` — the verdict is text, and the
merge is the user's. Does not open issues, does not commit, does not push, does not modify the
repo under review. And names no concrete repository path.

Boundary with `superpowers:requesting-code-review`: that one is the author's, before opening the
PR. This one is the reviewer's, on someone else's PR.

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
