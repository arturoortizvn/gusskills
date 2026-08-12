# `pr-peer-review` Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Version the personal skills in a private repo and add `pr-peer-review`, a
repo-agnostic peer-review skill that `orchestrated-worktree-delivery` delegates to instead of
carrying its own inline fallback scale.

**Architecture:** A new private repo `arturoortizvn/gusskills` cloned to
`~/Proyectos/Personales/gusskills` holds one directory per skill; `~/.claude/skills/<name>`
becomes a relative symlink into the clone, the layout the machine already proves works for the
insforge skills. Two skills live there: `orchestrated-worktree-delivery` (moved, then edited to
delegate) and `pr-peer-review` (new). The new skill carries the transferable half of peer
review — scale, format, `gh` mechanics, discipline — and resolves the non-transferable half
from the repo under review.

**Tech Stack:** Markdown (skill files), `gh` CLI, `git`, POSIX symlinks. No test framework
applies.

**Verification model — read this before Task 1.** There is no test suite for a skill, so this
plan has no red/green TDD cycle and does not pretend to. Each task's acceptance is executable
all the same: `grep` assertions over the produced files, symlink resolution, `gh` JSON reads.
Where a check cannot be run now, the plan says so instead of asserting success. The one thing
only a real run can prove — that the skill produces a good review — is listed as a pending
manual check in Task 5, not claimed.

## Global Constraints

- **All prose in these files is English.** Conversation with the user is Spanish; file content,
  commit messages, branch names and PR titles/bodies are English (`~/.claude/CLAUDE.md`).
- **Never commit to `main`.** All work lands on one branch, `feature/gusskills-pr-peer-review`,
  and reaches `main` only through a PR **the user merges**. Do not merge.
- **Every git command uses `git -C /Users/arturoortiz/Proyectos/Personales/gusskills`.** The
  `protect-git.sh` hook resolves the current branch from the *session's* cwd, which is
  `/Users/arturoortiz/Proyectos/Viewnear/BusinessTaxReturn` on branch `dev` — not the clone.
  Without `-C` the hook grades the wrong branch and can block a legitimate commit.
- **No `--no-verify`, no `--force`, no `reset --hard`, no `CLAUDE_GIT_OVERRIDE`.**
- **The active `gh` account must stay `arturoortizvn`** (verified: it is active now). The repo is
  created under that account, private.
- **Spec is the content source.** `docs/superpowers/specs/2026-08-12-pr-peer-review-design.md`
  §"The skill" 1–7 holds the wording for the new skill. Task 3 transcribes it rather than the
  plan restating it a third time — a duplicate would drift from both.
- **`pr-peer-review` must contain zero concrete repository paths.** That property is what makes
  it safe outside the repos it reviews, and Task 3 asserts it.

---

## File Structure

| Path (relative to the clone) | Responsibility |
|---|---|
| `README.md` | What the repo is, the symlink layout, how to add a skill, the branch flow |
| `.gitignore` | macOS noise only (`.DS_Store`) |
| `orchestrated-worktree-delivery/SKILL.md` | Moved in Task 2, edited in Task 4 (delegation only) |
| `pr-peer-review/SKILL.md` | The new skill — created in Task 3 |
| `docs/superpowers/specs/2026-08-12-pr-peer-review-design.md` | The approved design |
| `docs/superpowers/plans/2026-08-12-pr-peer-review.md` | This plan |

One directory per skill at the repo root, mirroring `~/.claude/skills/` exactly, so a symlink
target is always `<clone>/<skill-name>` with no nesting to remember.

---

### Task 1: Bootstrap the repo

**Files:**
- Create (remote): `arturoortizvn/gusskills`, private
- Create: `/Users/arturoortiz/Proyectos/Personales/gusskills/README.md`
- Create: `/Users/arturoortiz/Proyectos/Personales/gusskills/.gitignore`
- Create: `/Users/arturoortiz/Proyectos/Personales/gusskills/docs/superpowers/specs/2026-08-12-pr-peer-review-design.md`
- Create: `/Users/arturoortiz/Proyectos/Personales/gusskills/docs/superpowers/plans/2026-08-12-pr-peer-review.md`

**Interfaces:**
- Consumes: nothing.
- Produces: the clone at `/Users/arturoortiz/Proyectos/Personales/gusskills` on branch
  `feature/gusskills-pr-peer-review`, with `main` existing on the remote so a PR can target it.
  Every later task commits into this clone with `git -C`.

- [ ] **Step 1: Confirm the account and that the name is still free**

```bash
gh auth status 2>&1 | grep -A1 'Active account: true'
gh repo view arturoortizvn/gusskills 2>&1 | head -2
```

Expected: the active account block names `arturoortizvn`; the second command fails with
`Could not resolve to a Repository`. If the repo already exists, stop and report — do not
touch a repo this plan did not create.

- [ ] **Step 2: Create the private repo with a server-side initial commit**

```bash
gh repo create arturoortizvn/gusskills --private --add-readme \
  --description "Personal Claude Code skills, versioned"
```

`--add-readme` matters: it creates `main` with one commit **on the server**, so `main` exists
as a PR base and this session never commits to it locally.

- [ ] **Step 3: Clone it and branch immediately**

```bash
git clone https://github.com/arturoortizvn/gusskills.git /Users/arturoortiz/Proyectos/Personales/gusskills
git -C /Users/arturoortiz/Proyectos/Personales/gusskills switch -c feature/gusskills-pr-peer-review
git -C /Users/arturoortiz/Proyectos/Personales/gusskills branch --show-current
```

Expected: `feature/gusskills-pr-peer-review`. Do not run another command in the clone until
this prints the feature branch.

- [ ] **Step 4: Write `README.md`** (overwrites the stub GitHub created)

The block below is fenced with four backticks because the README's own layout block uses
three; write the README with three, as shown inside.

````markdown
# gusskills

Personal Claude Code skills, versioned. Each top-level directory is one skill and holds a
`SKILL.md`; `~/.claude/skills/<name>` is a symlink into this clone, so editing a file here is
editing the live skill.

## Layout

```
gusskills/
  pr-peer-review/SKILL.md
  orchestrated-worktree-delivery/SKILL.md
  docs/superpowers/{specs,plans}/
```

`~/.claude/skills/pr-peer-review -> ../../Proyectos/Personales/gusskills/pr-peer-review`

The relative form is deliberate: `~/.claude/skills/../../` resolves to `~/`, and it is the
same shape the insforge skills already use, which is how we know skill resolution follows a
symlink.

## Adding a skill

1. `mkdir <name>` here and write `<name>/SKILL.md` with `name` and `description` frontmatter.
   `name` must equal the directory name.
2. Symlink it: `ln -s ../../Proyectos/Personales/gusskills/<name> ~/.claude/skills/<name>`
3. Verify it reads through the symlink: `cat ~/.claude/skills/<name>/SKILL.md | head -5`
4. A brand-new skill may only appear in the skills listing in a fresh session.

## What does not belong here

Skills whose checks cite a specific repo's paths, line numbers or invariants. Those live in
that repo, at `.claude/skills/peer-review/`, because a skill outside the repo it describes
goes stale in silence — nothing in a PR that renames a service ever touches a file here.
`idanalyzer-peer-review` is the outstanding case: its destination is the IDAnalyzer repo, not
this one.

## Branch flow

Work on `feature/*`, `fix/*` or `update/*`; PR into `main`; the merge is the owner's, never an
agent's.
````

- [ ] **Step 5: Write `.gitignore`**

```gitignore
.DS_Store
```

- [ ] **Step 6: Copy the spec and this plan into the repo**

```bash
S=/private/tmp/claude-501/-Users-arturoortiz-Proyectos-Viewnear-BusinessTaxReturn/ac654f16-247f-4d86-9f2d-2e0ed22dc5ab/scratchpad
D=/Users/arturoortiz/Proyectos/Personales/gusskills
mkdir -p $D/docs/superpowers/specs $D/docs/superpowers/plans
cp $S/2026-08-12-pr-peer-review-design.md $D/docs/superpowers/specs/
cp $S/2026-08-12-pr-peer-review-plan.md   $D/docs/superpowers/plans/2026-08-12-pr-peer-review.md
```

- [ ] **Step 7: Verify the repo is private and carries no secrets**

```bash
gh repo view arturoortizvn/gusskills --json isPrivate,visibility
git -C /Users/arturoortiz/Proyectos/Personales/gusskills add -A
git -C /Users/arturoortiz/Proyectos/Personales/gusskills diff --cached --name-only | grep -E '^\.env' || echo "no env files: OK"
```

Expected: `"isPrivate": true`, `"visibility": "PRIVATE"`, and `no env files: OK`. If
`isPrivate` is false, stop and report — do not push to a public repo.

- [ ] **Step 8: Commit**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills commit -m "chore: bootstrap skills repo with the approved design and plan"
```

---

### Task 2: Move `orchestrated-worktree-delivery` into the repo

**Files:**
- Move: `/Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/` →
  `/Users/arturoortiz/Proyectos/Personales/gusskills/orchestrated-worktree-delivery/`
- Create: symlink `/Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery`

**Interfaces:**
- Consumes: the clone and feature branch from Task 1.
- Produces: `orchestrated-worktree-delivery/SKILL.md` inside the clone, reachable at its old
  path. Task 4 edits it **through the clone path**, never through the symlink.

**Why this task is the risky one:** this skill is in use today. Every other task adds files;
this one relocates a working file, and a broken symlink silently removes a skill. The
verification below exists for that.

- [ ] **Step 1: Record what the skill looks like now, to compare after the move**

```bash
wc -l /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/SKILL.md
shasum /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/SKILL.md
ls -la /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/
```

Write the line count and hash into your notes — Step 4 compares against them. Note whether
the directory holds anything besides `SKILL.md`; if it does, the move must carry it too.

- [ ] **Step 2: Move the directory, do not copy it**

```bash
mv /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery \
   /Users/arturoortiz/Proyectos/Personales/gusskills/orchestrated-worktree-delivery
```

A copy would leave two divergent originals, and the one still in `~/.claude/skills/` would win
resolution — the exact failure this repo exists to prevent.

- [ ] **Step 3: Symlink it back, relative, from inside `~/.claude/skills`**

```bash
cd /Users/arturoortiz/.claude/skills
ln -s ../../Proyectos/Personales/gusskills/orchestrated-worktree-delivery orchestrated-worktree-delivery
```

The `cd` is required: `ln -s` stores the target string verbatim, so a relative target is
resolved against the link's own directory.

- [ ] **Step 4: Verify the old path still reads the same file**

```bash
readlink /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery
shasum /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/SKILL.md
head -3 /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/SKILL.md
ls -l /Users/arturoortiz/.claude/skills/ | grep orchestrated
```

Expected: the hash matches Step 1's exactly; `head` prints the `---` / `name:
orchestrated-worktree-delivery` frontmatter. A "No such file or directory" here means the
relative target is wrong — fix the link before committing, and do not proceed with a skill
that has gone missing.

- [ ] **Step 5: Commit the move**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills add orchestrated-worktree-delivery
git -C /Users/arturoortiz/Proyectos/Personales/gusskills commit -m "chore: move orchestrated-worktree-delivery into version control"
```

Committing the move before editing it keeps Task 4's diff readable as a content change rather
than a whole-file addition.

---

### Task 3: Write the `pr-peer-review` skill

**Files:**
- Create: `/Users/arturoortiz/Proyectos/Personales/gusskills/pr-peer-review/SKILL.md`
- Create: symlink `/Users/arturoortiz/.claude/skills/pr-peer-review`
- Source of content: `docs/superpowers/specs/2026-08-12-pr-peer-review-design.md` in the clone
  (copied there in Task 1)

**Interfaces:**
- Consumes: the clone and feature branch from Task 1.
- Produces: a skill invocable as `pr-peer-review`. Task 4 references that exact name in
  `orchestrated-worktree-delivery`, so the frontmatter `name` and the directory name must both
  be `pr-peer-review`, spelled that way.

- [ ] **Step 1: Create the directory and write the frontmatter verbatim**

```yaml
---
name: pr-peer-review
description: Use when peer-reviewing someone else's pull request on any repo — resolves whose standards apply (the repo's own peer-review skill first, otherwise checks derived from that repo's written norms), grades on the shared Blocker/Major/Minor/Nit scale, and posts exactly one PR comment. Defers to a repo-local peer-review skill when one exists. Not for reviewing your own work before opening a PR — that is graph-review.
---
```

The description is deliberately plain-worded rather than emoji-keyed: it is matched as text.

- [ ] **Step 2: Write the body, transcribing the spec's sections in this order**

Read `docs/superpowers/specs/2026-08-12-pr-peer-review-design.md` and transcribe, preserving
its wording — the spec was reviewed and approved as the skill's text, so paraphrasing it here
would ship unreviewed prose:

| Heading in `SKILL.md` | Source in the spec |
|---|---|
| `# Peer review — any repo` + `## Overview` | "Why" (condensed to 3–4 lines) plus the no-repo-paths rule from "The skill" |
| `## Standards resolution` | §1, all three numbered steps including what counts as a repo review skill |
| `## Severity scale` | §2, all four levels |
| `## Universal checks` | §3, all eight, with the precedence sentence |
| `## Discipline` | §4, all five bullets |
| `## Procedure` | §5, all five steps with the `gh` blocks verbatim |
| `## Output` | §6, the template fenced as ```markdown, plus every rule below it |
| `## What this skill does not do` | §7, including the `graph-review` boundary |

Two rules while transcribing: keep the code fences intact so the `gh` commands stay
copy-pasteable, and **introduce no repository path** — not as an example, not in a
parenthesis. Where the spec names a repo file generically (`CLAUDE.md`, `CONTRIBUTING.md`,
`.env.example`), that is a filename any repo can have, and it stays.

- [ ] **Step 3: Symlink it into `~/.claude/skills`**

```bash
cd /Users/arturoortiz/.claude/skills
ln -s ../../Proyectos/Personales/gusskills/pr-peer-review pr-peer-review
cat /Users/arturoortiz/.claude/skills/pr-peer-review/SKILL.md | head -5
```

Expected: the frontmatter prints through the symlink.

- [ ] **Step 4: Assert the properties the spec requires**

```bash
F=/Users/arturoortiz/Proyectos/Personales/gusskills/pr-peer-review/SKILL.md
grep -n '^## ' $F                                    # expect 9 hits: 8 sections + 1 inside the fence
grep -n 'name: pr-peer-review' $F                    # frontmatter name matches the directory
grep -nE '(server_py|app|client|shared|infra)/' $F || echo "no repo paths: OK"
grep -nE 'Blockers|Major|Minor|Nits' $F | head -8    # all four severities present
grep -n 'Checked and cleared' $F && grep -n 'Could not verify' $F
grep -n 'gh pr comment' $F
```

Expected: **9** heading hits — the 8 real sections plus the `## Peer review — …` line inside the
fenced output template, which `grep` counts because it cannot see fences. The name line present;
`no repo paths: OK`; all four severity words; both result sections; the posting command. A hit on
the repo-path grep is a failure to fix, not a finding to note — that property is the whole reason
this skill may live outside a repo.

- [ ] **Step 5: Commit**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills add pr-peer-review
git -C /Users/arturoortiz/Proyectos/Personales/gusskills commit -m "feat: add pr-peer-review, a repo-agnostic peer review skill"
```

---

### Task 4: Make `orchestrated-worktree-delivery` delegate

**Files:**
- Modify: `/Users/arturoortiz/Proyectos/Personales/gusskills/orchestrated-worktree-delivery/SKILL.md`
  — section `## Reviewer resolution`, step 2 and the `### Fallback severity scale and format`
  subsection that follows it

**Interfaces:**
- Consumes: the skill name `pr-peer-review` produced by Task 3. Do not run this task first —
  it would point at a skill that does not exist.
- Produces: nothing later tasks consume.

- [ ] **Step 1: Read the section as it stands**

```bash
grep -n -A 35 '^## Reviewer resolution' \
  /Users/arturoortiz/Proyectos/Personales/gusskills/orchestrated-worktree-delivery/SKILL.md
```

Confirm two things before editing: step 2 currently reads
"Otherwise → `superpowers:requesting-code-review`, with the fallback scale and format below.",
and the `### Fallback severity scale and format` subsection ends with the verdict sentence
("… spelled out in the summary.") immediately before `## Ticket hook`.

- [ ] **Step 2: Replace step 2 of the cascade**

Old:

```markdown
2. Otherwise → `superpowers:requesting-code-review`, with the fallback scale and format
   below.
```

New:

```markdown
2. Otherwise → `pr-peer-review`, which resolves the checks from that repo's own written norms
   and carries the scale, the comment format and the posting mechanics.
```

- [ ] **Step 3: Delete the whole `### Fallback severity scale and format` subsection**

Delete from the `### Fallback severity scale and format` heading through the verdict sentence
that ends it — heading, the three-line justification, the four severity bullets and the
one-comment paragraph. `## Ticket hook` becomes the next thing after the convention paragraph.

Its justification ("without it, the review step cannot run at all on a repo that has not
written its standards yet") is what expires: Task 3 shipped the skill that runs it.

- [ ] **Step 4: Leave the convention paragraph untouched**

The paragraph beginning "The convention this encodes: **a repo's review standards live in that
repo.**" stays **verbatim**, including the sentence about `idanalyzer-peer-review` having to
move into IDAnalyzer. It remains true: `pr-peer-review` is not a global skill named after a
repo, it carries no repo-specific checks, and it is precisely the step-2 the paragraph leaves
open. Do not soften or annotate it.

- [ ] **Step 5: Verify the edit removed the definition and kept the gate**

```bash
F=/Users/arturoortiz/Proyectos/Personales/gusskills/orchestrated-worktree-delivery/SKILL.md
grep -n 'Fallback severity scale' $F || echo "scale definition gone: OK"
grep -n 'pr-peer-review' $F                          # expect exactly one hit, in step 2
grep -n 'a repo.s review standards live in that repo' $F   # convention paragraph survived
grep -cn 'Blockers\|Majors' $F                       # expect 2 — the gate steps, which must stay
grep -n -A 3 '^## Reviewer resolution' $F
```

Expected: `scale definition gone: OK`; one `pr-peer-review` hit; the convention line present;
**two** hits for Blockers/Majors — those are the cycle's review gate ("No Blockers and no
Majors → tell the user they can merge") and its argue step. Those two are load-bearing: the
gate is *stated* in those terms. Deleting them would break the cycle, so a grep hit there is
correct, not leftover text.

- [ ] **Step 6: Verify the skill still reads through its symlink**

```bash
head -3 /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/SKILL.md
grep -c 'pr-peer-review' /Users/arturoortiz/.claude/skills/orchestrated-worktree-delivery/SKILL.md
```

Expected: frontmatter prints, and the count is 1 — proof the edit landed on the file the live
path resolves to.

- [ ] **Step 7: Commit**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills add orchestrated-worktree-delivery
git -C /Users/arturoortiz/Proyectos/Personales/gusskills commit -m "refactor: delegate reviewer resolution step 2 to pr-peer-review"
```

---

### Task 5: Push, open the PR, hand off

**Files:**
- No file changes. Remote branch and PR only.

**Interfaces:**
- Consumes: the four commits from Tasks 1–4.
- Produces: a PR into `main` for the user to merge. **Do not merge it.**

- [ ] **Step 1: Review the full diff before pushing**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills log --oneline main..HEAD
git -C /Users/arturoortiz/Proyectos/Personales/gusskills diff main..HEAD --stat
git -C /Users/arturoortiz/Proyectos/Personales/gusskills status --short
```

Expected: four commits; the stat lists `README.md`, `.gitignore`, both `SKILL.md` files, the
spec and the plan; `status` is clean. An untracked file here means something was written
outside the plan — report it rather than sweeping it into the commit.

- [ ] **Step 2: Push the branch**

```bash
git -C /Users/arturoortiz/Proyectos/Personales/gusskills push -u origin feature/gusskills-pr-peer-review
```

- [ ] **Step 3: Open the PR into `main`**

```bash
cd /Users/arturoortiz/Proyectos/Personales/gusskills
gh pr create --base main --head feature/gusskills-pr-peer-review \
  --title "Bootstrap gusskills and add pr-peer-review" \
  --body "$(cat <<'EOF'
Puts the personal skills under version control and adds `pr-peer-review`.

**What is here**
- `pr-peer-review/` — new. Repo-agnostic peer review: resolves whose standards apply (the
  repo's own peer-review skill first, otherwise checks derived from that repo's written
  norms), grades on the Blocker/Major/Minor/Nit scale, posts exactly one PR comment. It cites
  no concrete repository path, which is what makes it safe to live outside the repos it
  reviews.
- `orchestrated-worktree-delivery/` — moved into version control unchanged, then edited in one
  place: *Reviewer resolution* step 2 now delegates to `pr-peer-review`, and the ~20-line
  inline fallback scale is deleted. Its justification ("without it, the review step cannot run
  at all on a repo that has not written its standards yet") expired when that skill shipped.
  The convention paragraph is untouched.
- `docs/superpowers/` — the approved design and this work's plan.

**Not here, on purpose:** `idanalyzer-peer-review` stays out — its destination is the
IDAnalyzer repo, and a third location would legitimise keeping it outside the repo it
describes. The two `running-*-locally` skills are untouched.

**Verified**
- Both skills read through their `~/.claude/skills/` symlinks; the moved file's checksum is
  unchanged by the move.
- `pr-peer-review` contains zero repository paths.
- `orchestrated-worktree-delivery` no longer defines a severity scale, while the review gate's
  own references to Blockers and Majors survive — the gate is stated in those terms.
- Repo is private; no `.env*` tracked.

**Pending manual check:** a real run. The only evidence that the skill produces a good review
is using it on an open PR, and there was none available while writing it.
EOF
)"
```

- [ ] **Step 4: Report and stop**

Give the user the PR URL, the pending manual check, and this note: the new skill may only
appear in the skills listing in a fresh session. **The merge is the user's** — do not merge,
do not switch `main`, do not delete the branch.

---

## Self-Review

**Spec coverage.** Every section maps to a task: "Where the skills live" → Tasks 1–3
(repo, symlinks, which skills move); "The skill" §1–§7 → Task 3 Step 2's transcription table;
"Second deliverable" → Task 4 (all three bullets: step 2 rewritten, fallback deleted,
convention kept); "Verification" → Task 1 Step 7 (private, no env), Task 2 Step 4 and Task 3
Step 3 (symlinks), Task 3 Step 4 (no repo paths), Task 4 Step 5 (scale gone, gate kept, listing
caveat carried into Task 5 Step 4), and the real-run gap declared in Task 5 Step 3's PR body.
"Out of scope / follow-up" → stated in the README and the PR body; no task touches
`idanalyzer-peer-review` or the `running-*` skills.

**Placeholder scan.** No TBD/TODO. Every command is literal and absolute; the one piece of
content not inlined is the new skill's body, which is a transcription of an approved spec, with
the section-to-source mapping given explicitly and the deviation from "inline everything"
stated in the Global Constraints with its reason.

**Name consistency.** `pr-peer-review` is spelled identically in the directory name, the
frontmatter, Task 4's replacement text and every grep. The branch is
`feature/gusskills-pr-peer-review` in Tasks 1, 5 and nowhere else. The clone path
`/Users/arturoortiz/Proyectos/Personales/gusskills` is absolute and identical in every command.
The symlink target string `../../Proyectos/Personales/gusskills/<name>` is the same relative
shape in both Task 2 and Task 3.

**Ordering constraint worth restating:** Task 4 must not run before Task 3 — it would point the
cascade at a skill that does not exist yet.
