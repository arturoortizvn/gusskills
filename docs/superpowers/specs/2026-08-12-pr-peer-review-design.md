# Design — `pr-peer-review`, a repo-agnostic peer-review skill

**Date:** 2026-08-12
**Status:** approved (design), pending implementation plan
**Deliverables:** a new private repo `arturoortizvn/gusskills` holding the skills,
`pr-peer-review/SKILL.md` (new) inside it, and a minimal edit to
`orchestrated-worktree-delivery/SKILL.md` (moved into the same repo). Both stay reachable at
`~/.claude/skills/<name>` through symlinks.

## Why

Two peer-review skills exist today and both are repo-specific:
`~/.claude/skills/idanalyzer-peer-review` (LendLogic.IDAnalyzer) and
`LendLogic.TaxReturnAnalyzer/.claude/skills/peer-review`. Comparing them shows a clean split:

- **Transferable** — the 🔴/🟠/🟡/🔵 scale, exactly one PR comment per round, the
  `Checked and cleared` / `Could not verify` pair, verifying the PR body's claims against the
  base branch, discarding unsubstantiated findings with the reason written down, the `gh`
  mechanics.
- **Not transferable** — every check that cites a path, a line number, or a named invariant.

The standing convention (memory `peer-review-skill-is-idanalyzer-only`) is that **a repo's
review standards live in that repo**, because a skill in the reviewer's home directory ages
in silence: nothing in a PR that renames a service ever touches a file under `~/.claude`.
This design does not weaken that convention — it supplies the missing half of it. Today
`orchestrated-worktree-delivery` carries a *Reviewer resolution* cascade whose step 2 falls
back to `superpowers:requesting-code-review` plus ~20 inline lines of scale and format,
annotated as a deliberate duplication "because without it, the review step cannot run at all
on a repo that has not written its standards yet." Once this skill exists, that reason
expires.

## Decisions

| Decision | Choice |
|---|---|
| Repo-specific standards | Scaffolding + delegation: defer to the repo's own review skill; derive checks from the repo's written norms when it has none |
| Scope vs. `idanalyzer-peer-review` | Out of scope — it stays where it is; its migration into IDAnalyzer remains separate pending work |
| Comment shape | Hybrid: four severities + `Checked and cleared` + `Could not verify` + `What looks good` |
| Duplication in `orchestrated-worktree-delivery` | Single source: its step 2 becomes this skill; the inline fallback scale/format is deleted |
| Where the skills live | New private repo `arturoortizvn/gusskills`, cloned to `~/Proyectos/Personales/gusskills`, symlinked into `~/.claude/skills/` |
| Which skills move in | `pr-peer-review` and `orchestrated-worktree-delivery` only |

## Where the skills live

`~/.claude/skills/` is not a git repository, so until now these skills were unversioned: no
history, no review, no way to commit the spec next to what it specifies. They move into a
private repo — `arturoortizvn/gusskills` (work account, already the active `gh` account) —
cloned to `~/Proyectos/Personales/gusskills`, with `~/.claude/skills/<name>` becoming a
symlink into the clone. That is the layout the machine already uses for the insforge skills
(`~/.claude/skills/insforge -> ../../.agents/skills/insforge`), so skill resolution is known
to work through a symlink.

Two skills move: `pr-peer-review` (born there) and `orchestrated-worktree-delivery` (moved,
then edited). Deliberately staying put:

- `idanalyzer-peer-review` — its pending destination is the IDAnalyzer repo. Moving it to a
  third place would legitimise keeping it outside the repo it describes.
- `running-brokerlos-locally`, `running-business-tax-return-locally` — untouched, out of
  scope; they can migrate later without affecting this work.

**Consequence for this document:** the spec is committed into that repo under
`docs/superpowers/specs/`, and every change to either skill lands on a `feature/*` branch with
a PR into `main` — the same rules as any other repo, which is most of the point of moving them.

## The skill

**Name** `pr-peer-review`, in `~/.claude/skills/`. Deliberately *not* `peer-review`: that
name is taken by TaxReturnAnalyzer's repo-local skill, and a home skill sharing it would make
resolution ambiguous in the one repo that already did this right.

**Frontmatter description** must fire on "review this PR" in any repo, and must say out loud
that it defers to a repo-local skill and that it is not the author-side pre-PR review
(`graph-review`).

**Single `SKILL.md`.** No reference files — both existing instances are single-file at ~200
and ~350 lines, and this one carries less because it carries no repo checks.

**It cites no repository path, ever.** That is the property that makes it safe to live in the
home directory, and the skill states it as a rule: the moment a check needs a concrete path,
that check belongs in the repo's own skill.

### 1. Standards resolution (procedure step 1)

1. The repo has a review skill of its own → **invoke it and grade with its checks.** This
   skill then contributes only what that one lacks (`gh` mechanics, discipline) and never
   competes with it. What counts: any skill under the repo's `.claude/skills/` whose name or
   description is about reviewing pull requests **on that repo** — `peer-review` is the
   convention, but the test is the description, not the directory name. A skill about
   reviewing your own work before opening a PR (`graph-review`, `code-review`) does not count
   and does not satisfy step 1.
2. No such skill → **derive** the checks from the repo's written norms, in order:
   `CLAUDE.md` (root and nested) → `CONTRIBUTING.md` → `docs/` (invariants, ADRs,
   architecture) → `docs/superpowers/specs/` → `.github/workflows/` (what CI actually gates,
   not what the README claims) → `.env.example` → the `tests/` layout (which suites are
   hermetic, which skip themselves).
3. **Write the derived list before grading** — N checks, each with its severity and the doc
   line that backs it. An invariant you invented is not a finding. The comment states which
   norms it graded against, and says so plainly when the repo had none.

### 2. Severity scale

Stated as a principle plus examples, not as a list of one repo's breaches — that is what
makes it portable.

- **🔴 Blocker** — merging it damages the service or makes the repo's own documentation lie:
  breaks a documented contract, seam or invariant without updating the doc that asserts it;
  leaks secrets or PII; makes the test suite non-hermetic; contradicts an approved spec.
- **🟠 Major** — must be fixed, does not by itself break the service: architecture
  deviations, new logic with no tests, a new env var with no fail-fast check or no
  `.env.example` entry, blocking work called straight from an `async` handler, unquantified
  new per-request cost.
- **🟡 Minor** — code smells, naming, dead branches, docs left stale by an additive change,
  error messages that send an integrator round in a loop.
- **🔵 Nit** — style, formatting, micro-optimisations. Never a reason to withhold approval.

### 3. Eight universal checks

These need no repo doc because they hold anywhere. They apply *on top of* the derived ones.
**Precedence: where the repo has its own review skill, its checks and its severities govern**
— the universal ones only cover what that skill is silent about.

1. **Secrets** — no key, token or connection string in code, tests, examples, docs or IaC
   params. 🔴
2. **PII and user data** — no extracted data in logs, error messages or fixtures, **and none
   in the review comment**. Cite `file:line`, never content. 🔴
3. **Tests** — new logic ships with tests that run in the default suite. A suite that skips
   itself (no DB, no key) **is not evidence that it ran**: ask what the variable was set to.
   🟠, and 🔴 if the PR disables or loosens a guard.
4. **Doc–code coherence** — where the repo documents its contract, response shape or security
   posture, making that doc false without updating it is 🔴; an additive change that leaves it
   stale is 🟡.
5. **Config** — a new env var needs a fail-fast check and an `.env.example` entry. 🟠
6. **Branch flow and merge method** per the repo's own `CONTRIBUTING.md` / `CLAUDE.md`; compare
   `baseRefName` against the branch type. 🔴 when it targets the wrong branch.
7. **Cost and latency** of new calls in the request path; an unbounded retry loop is 🔴.
8. **Ticket** in the PR body where the repo has that convention; if absent, write `no ticket`
   in the heading and raise it as 🟡.

### 4. Discipline

Each line came from a real finding, and this is the genuinely transferable part.

- Verify the PR body's claims **against the base branch** (`git show origin/<base>:<path>`),
  never against the PR's description of itself.
- **Test guards by mutating them.** A guard with no coverage leaves the suite green while it
  guards nothing.
- Run experiments in a **clean tree**; on a dirty tree a result can be a false negative — or
  you declare it provisional.
- **Discard unsubstantiated findings explicitly, with the reason** → they go under
  `Checked and cleared`. Silence reads as "checked and fine".
- Tree-scanning checks mean nothing without the head checkout: run from the base they come
  back clean, and that is a false negative on a Blocker.

### 5. Procedure

Scope: GitHub PRs via `gh`. No GitHub remote or no PR → say so and refer to
`superpowers:requesting-code-review`; there is nowhere to post. The comment is written in
**English** (it is PR content).

1. **Resolve standards** (§1) and write the check list.
2. **Read the PR, then check out its head.**
   ```bash
   R=<owner/repo>
   gh pr view <n> --repo $R --json number,title,body,state,headRefName,baseRefName,files,url
   gh pr diff <n> --repo $R
   gh pr checkout <n> --repo $R
   ```
   Guards: the working tree must be **clean** before the checkout — otherwise stop and tell
   the user; never clobber their work. If `state` is not `OPEN`, write the comment to a file
   and **stop**: reviewing a closed PR is a calibration exercise, and the people who already
   shipped it do not need the notification.
3. **Grade** against the derived checks plus the universal ones. An optional first pass with
   `/code-review <n>` is allowed for a correctness sweep, but **its output is never posted
   raw** — its findings go through the scale and the substantiation rule like any other.
4. **Post exactly one comment.**
   ```bash
   gh auth status                                  # active account must match the repo owner
   gh pr comment <n> --repo $R --body-file <file>
   ```
   Identity per `~/.claude/personal-projects.md`: work repos → the work account; personal
   repos → `aiyangar`. On a mismatch, **stop and ask for the switch** (`gh auth switch`); never
   post from the wrong account. No issues, no email, no Slack. **Never edit or re-post over a
   previous comment** — a second round is a second comment, so the argument stays readable.
5. **Offer the repo-local skill** — only when checks were derived because the repo had none.
   One line at the end, **in the chat, not in the PR comment**. It is an offer, not a
   requirement; if accepted, that work is its own task on its own branch.

### 6. Output

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

### ✅ What looks good
- <1–3 specific things>

**Verdict:** Approve | Comment | Request changes — <one line>
```

The heading carries the ticket where the repo has a ticket convention, the branch
(`headRefName`) where it has none, and the literal `no ticket` where the convention exists but
the PR body names none — which is also the 🟡 of universal check 8.

Rules: all four severity headers always present, `_None._` under the empty ones, so a reader
can tell "no blockers" from "blockers not considered". Every finding cites `file:line`.
`Checked and cleared` ("I ran it and there was nothing") and `Could not verify` ("I could not
run it") are not interchangeable, and no check is left out of both. Verdict: any 🔴 → Request
changes; 🟠 without 🔴 → Comment, with the decision spelled out in the summary; only 🟡/🔵 or
none → Approve. A pure-formatting diff gets only `Auto-formatting only — no review needed.`
and stops.

### 7. Non-goals

Does not merge, does not approve through `gh pr review --approve` (the verdict is text; the
merge is the user's), does not open issues, does not commit, does not push, does not modify
the repo under review — and cites no concrete repo path, which is its anti-staleness by
construction.

Boundary with `graph-review`: that one is the author's, before opening the PR; this one is the
reviewer's, on someone else's PR.

## Second deliverable — edit to `orchestrated-worktree-delivery`

Minimal and surgical:

- *Reviewer resolution* step 2 becomes `pr-peer-review` instead of
  `superpowers:requesting-code-review` + inline scale.
- The `### Fallback severity scale and format` section is deleted, along with the sentence
  justifying the duplication.
- The paragraph explaining the convention (no intermediate "global skill named after the
  repo" step, and `idanalyzer-peer-review` must move into its repo first) **stays** — it is
  still true, and this skill is not that discarded step: it looks for the repo's own skill,
  not for one named after the repo.

Nothing else in that skill is touched — the role boundary, the round cap, the ticket hook and
the git mechanics are out of scope here.

## Verification

A skill has no test suite. The evidence available before use is:

- Both files parse as skills and appear in the skills listing with the intended descriptions.
  A brand-new skill may need a fresh session before it is listed — an absent entry right after
  writing it is not yet a failure.
- Both symlinks resolve (`ls -l ~/.claude/skills/` shows them pointing into the clone, and
  `cat ~/.claude/skills/<name>/SKILL.md` reads the file). This is the one step that can break
  a skill that was working before, so it is checked for
  `orchestrated-worktree-delivery` specifically — it is in use today.
- The repo is **private** (`gh repo view arturoortizvn/gusskills --json visibility`) and holds
  no secrets: `git ls-files | grep -E '^\.env'` comes back empty.
- The new skill contains **zero** concrete repo paths (grep for `server_py/`, `app/`,
  `services/`, `tests/` as path prefixes).
- `orchestrated-worktree-delivery` no longer *defines* the scale: `Fallback severity scale`
  and the `🔴 **Blocker** —` definition line are gone. **The cycle's own references to
  Blockers and Majors must survive** — its review gate and its argue-through-the-orchestrator
  step are stated in those terms, so a grep for the words returning hits there is correct,
  not leftover text to delete.
- The convention paragraph survives the edit verbatim.

Real evidence is a run on an actual open PR. There is none open in this repo right now, so
that is stated as a pending manual check rather than claimed — per
`superpowers:verification-before-completion`.

## Out of scope / follow-up

- **Migrating `idanalyzer-peer-review`** into `LendLogic.IDAnalyzer/.claude/skills/peer-review/`
  — separate task, separate repo, separate branch. `CONTRIBUTING.md` there already puts that
  obligation on the PR author.
- **Generalising `orchestrated-worktree-delivery`** the same way this skill generalises peer
  review: it currently hardcodes TaxReturnAnalyzer's Monday board coordinates in its ticket
  hook. The user intends to take that on next; it is its own task with its own spec, and this
  design deliberately leaves the seam for it (step 2 already delegating outward).
