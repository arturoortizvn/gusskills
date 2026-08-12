---
name: pr-peer-review
description: Use when peer-reviewing someone else's pull request on any repo — resolves whose standards apply (the repo's own peer-review skill first, otherwise checks derived from that repo's written norms), grades on the shared Blocker/Major/Minor/Nit scale, and posts exactly one PR comment. Defers to a repo-local peer-review skill when one exists. Not for reviewing your own work before opening a PR — that is graph-review.
---

# Peer review — any repo

## Overview

Reviews someone else's pull request on any repository and delivers the result as **exactly one
PR comment**. It carries the half of peer review that transfers between repos — the severity
scale, the comment format, the posting mechanics, and the discipline that keeps findings
honest. The half that does not transfer, the checks that cite a repo's own paths, seams and
invariants, comes from that repo.

**This skill names no concrete repository path, ever.** That is what makes it safe to live
outside the repos it reviews: a review skill kept in the reviewer's home directory ages in
silence, because nothing in a PR that renames a service ever touches a file there. The moment
a check you want needs a real path or line number, that check belongs in the repo's own review
skill — not in this file.

## Standards resolution

Step 1 of the procedure, and it decides everything after it.

1. **The repo has a review skill of its own → grade with its checks.** Invoke it where the
   session can — a repo-local skill only reaches the skills listing when the session's cwd is
   that repo, so from anywhere else, read the file and follow it. This skill then contributes
   only what that one lacks (the `gh` mechanics, the discipline) and never competes with it.
   What counts: any skill under the repo's `.claude/skills/` whose name or description is about
   reviewing pull requests **on that repo** — `peer-review` is the
   convention, but the test is the description, not the directory name. A skill about reviewing
   your own work before opening a PR (`graph-review`, `code-review`) does not count and does
   not satisfy this step.
2. **No such skill → derive the checks** from the repo's own written norms, in this order:
   `CLAUDE.md` (root and nested) → `CONTRIBUTING.md` → `docs/` (invariants, ADRs, architecture)
   → design specs under `docs/` → the CI workflow definitions (what CI *actually* gates, not
   what the README claims) → `.env.example` → the test layout (which suites are hermetic, which
   skip themselves).
3. **Write the derived list before grading** — N checks, each with its severity and the doc
   line that backs it. An invariant you invented is not a finding. The comment states which
   norms it graded against, and says so plainly when the repo had none.

## Severity scale

Stated as a principle plus examples, because a list of one repo's breaches does not travel.

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

These need no repo doc because they hold anywhere, and they apply *on top of* the derived ones.
**Precedence: where the repo has its own review skill, its checks and its severities govern** —
these only cover what that skill is silent about.

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

Each line came from a real finding, and this is the genuinely transferable part.

- Verify the PR body's claims **against the base branch** (`git show origin/<base>:<path>`),
  never against the PR's description of itself.
- **Test guards by mutating them.** A guard with no coverage leaves the suite green while it
  guards nothing. If the PR adds one, find the test that fails when it is removed.
- Run experiments in a **clean tree**; on a dirty tree a result can be a false negative — or you
  declare it provisional.
- **Discard unsubstantiated findings explicitly, with the reason** → they go under *Checked and
  cleared*. Silence reads as "checked and fine".
- Tree-scanning checks mean nothing without the head checkout: run from the base they come back
  clean, and that is a false negative on a Blocker.

## Procedure

Scope: GitHub PRs via `gh`. No GitHub remote or no PR → say so and refer to
`superpowers:requesting-code-review`; there is nowhere to post. The comment is written in
**English**, because it is PR content.

1. **Resolve standards** (above) and write the check list.

2. **Read the PR, then check out its head.**

   ```bash
   R=<owner/repo>
   gh pr view <n> --repo $R --json number,title,body,state,headRefName,baseRefName,files,url
   gh pr diff <n> --repo $R
   gh pr checkout <n> --repo $R
   ```

   Guards: the working tree must be **clean** before the checkout — otherwise stop and tell the
   user; never clobber their work. If `state` is not `OPEN`, write the comment to a file and
   **stop**: reviewing a closed PR is a calibration exercise, and the people who already shipped
   it do not need the notification.

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
   previous comment** — a second round is a second comment, so the argument stays readable.

5. **Offer the repo-local skill** — only when checks were derived because the repo had none. One
   line at the end, **in the chat, not in the PR comment**. It is an offer, not a requirement; if
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

The three dispositions are not interchangeable, and no check is left out of all three:
*Checked and cleared* is "I ran it and there was nothing", *Could not verify* is "I could not
run it", and *Not applicable* is "it cannot apply here" — a docs-only repo has no test suite, no
env vars and no request path, so universal checks 3, 5 and 7 belong in the third, not stretched
into either of the first two. Drop a heading only when nothing landed under it.

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
repo under review. And names no concrete repository path, which is its anti-staleness by
construction.

Boundary with `graph-review`: that one is the author's, before opening the PR. This one is the
reviewer's, on someone else's PR.
