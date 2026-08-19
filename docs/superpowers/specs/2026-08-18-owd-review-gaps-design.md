# Design — `orchestrated-worktree-delivery`, the five gaps left by the repo-agnostic rewrite

**Date:** 2026-08-18
**Status:** approved (design), pending implementation plan
**Deliverable:** an amended `orchestrated-worktree-delivery/SKILL.md`. No other file changes.

## Why

A review of the skill at `057a24a` found seven defects. Two were falsifiable at a terminal and
shipped in PR #3: the closeout deleted a branch without removing its worktree, and
`git -C <worktree>` was documented as sufficient when the hook grades the last `-C` in the command
string. That PR also added, at the user's request, the rule that every review round posts its own
comment on the PR. The remaining five defects are the subject of this design.

| # | Gap | Where |
|---|---|---|
| 3 | The integration branch is never resolved | promised in the frontmatter, absent from the body; only `Closeout:149` reads it back from the PR |
| 4 | Nobody pushes the branch | `:54` forbids the implementer, `:56` assumes a pushed branch |
| 5 | Two review loops land on one diff | the cycle's PR review vs the dispatch skill's per-task review and whole-branch final review |
| 6 | The precedence list omits the hardest rules | `:28-33` names four topics; the role boundary and the ticket's ownership are not among them |
| 7 | Where the reviewer runs is unspecified | `pr-peer-review:104-116` checks the PR out; the cycle never says into what |

Gaps 3, 4 and 7 stop the cycle from running as written. 5 and 6 are ambiguities that resolve
differently depending on which file the agent read last.

### Evidence

Gap 7 is not a risk, it is a hard failure. With the implementer's worktree alive — and it stays
alive, since rounds 2 and 3 reuse it — the checkout `pr-peer-review` performs cannot happen in the
orchestrator's checkout:

```
$ git -C <main checkout> checkout feature/w
fatal: 'feature/w' is already used by worktree at '.../wt2'
exit=128
```

Gap 4's delegation has nowhere to land today: `subagent-driven-development` contains no mention of
`push`, `origin`, `remote` or `gh pr` — its only near-hit is line 29, where "a push to a shared
branch" is listed as something the subagent **stops and asks about**. The push exists in
`finishing-a-development-branch` (Steps 4-5), behind a three-option menu shown to the human whose
first option is a local merge, at the end of a whole branch rather than per item.

Gap 6 is latent rather than live: `subagent-driven-development:408` ("Never fix findings yourself
in the controller session") agrees with the role boundary today, and it never mentions tickets. The
precedence rule as written would hand both away the moment that changes — which is exactly the
drift the skill's own delegation paragraph warns about.

## Decisions

| Decision | Choice |
|---|---|
| Integration branch | Resolved from the repo's written norms before the first dispatch; ask the user where they are silent; never inferred from the forge's default branch |
| Who pushes | Delegated to `superpowers:finishing-a-development-branch`, named and capped at push + PR |
| Fix-round re-push | Written here — neither invoked skill covers it |
| Review layers | Dispatch skill's per-task review stays; its whole-branch final review does not run; the three-round cap counts PR rounds only |
| Precedence | Inverted from a closed list of topics to a principle |
| Reviewer's tree | Its own detached worktree at the PR head, created per round and removed when the comment is posted |
| Rider A | `CONTRIBUTING.md` added to the *Ticket hook* resolution order, so both orders in the file are identical |
| Rider B | The literal `Code Review` in step 4 becomes "the repo's review status" |
| Edit shape | Amend in place — one new section, two rewritten steps, two new paragraphs in *Reviewer resolution*, one rewritten paragraph |

## The changes

Section order is unchanged except for the one insertion. Every other section is carried over
byte-identical, which is the acceptance criterion that keeps the amendment from drifting.

### 1. New section `## Integration branch`

Placed between *Ticket hook* and *Git mechanics*, which already uses the term without defining it.

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
```

### 2. Step 4 — the push, delegated and capped

```markdown
4. **The orchestrator gets the branch pushed and the PR opened** —
   `superpowers:finishing-a-development-branch`, capped at push and PR: no menu, no local
   merge, no merge at all. Then the PR's URL goes on the ticket, the ticket moves to the
   repo's review status, and the reviewer is dispatched (see *Reviewer resolution*).
```

The cap is the one already written in `~/.claude/CLAUDE.md`. Naming the skill matters: the generic
"the dispatch skill handles it" resolves to a skill that says nothing about remotes.

### 3. Step 7 — the fix round's re-push, and what the cap counts

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

### 4. *Reviewer resolution* — two layers, and only two

```markdown
**Two review layers, and only two.** The dispatch skill's own per-task review stays where it is —
inside the dispatch, on its loop and its cap — as the cheap gate that keeps unreviewed work from
reaching a PR. Its whole-branch final review does not run: the peer review on the PR is that
review, and running both puts two verdicts on one diff with nothing to break the tie. **The
three-round cap counts PR review rounds only**; rounds that happen inside a single dispatch are the
invoked skill's business and do not consume them.
```

### 5. *Reviewer resolution* — the reviewer's worktree

```markdown
**The reviewer gets a worktree of its own**, detached at the PR's head. The orchestrator creates
it for the round and removes it once the comment is posted — local hygiene that lands in no diff,
so the role boundary leaves it here. Not the orchestrator's checkout: `gh pr checkout` there fails
outright while the implementer's worktree holds the branch — `fatal: '<branch>' is already used by
worktree at '<path>'` — and where it does succeed it drags the main checkout off the integration
branch that *Git mechanics* depends on. Not the implementer's either: the fix round reuses it, and
whatever the reviewer leaves behind blocks the removal at *Closeout*. A fresh worktree also
satisfies `pr-peer-review`'s clean-tree guard by construction, so the brief says the tree is
already at the head and no checkout is needed.
```

### 6. Precedence, inverted

```markdown
**Precedence.** The delegation covers the *how* of dispatch. **Where this cycle states a rule and
an invoked skill says otherwise, this cycle governs; where this cycle is silent, the invoked skill
governs.** It is a principle rather than a list of governed topics because the list was already
incomplete — the role boundary and the ticket's ownership bind exactly as hard as the review gate,
the round cap, when the PR is opened and the closeout, and every rule this file gains would have to
be added to it again. What this cycle does not state — how to brief, which model, how to resume an
implementer, how to handle its report — follows the invoked skill. None of this quotes version
numbers or values from another skill, on purpose, so it stays true after that skill changes.
```

### 7. Riders

- *Ticket hook*: the resolution order gains `CONTRIBUTING.md`, becoming `CLAUDE.md` (root and
  nested) → `.claude/` → `CONTRIBUTING.md` → `docs/` — identical to the one in *Integration
  branch*. Two different orders in one file is a discrepancy a reader has to adjudicate.
- Step 4: the literal `Code Review` goes. It is the same class of repo-specific leak the
  repo-agnostic rewrite removed from the rest of the file; `In Progress` and `Done` survive
  elsewhere and are out of scope here.

## Verification

A skill has no test suite. What is executable:

- **The gap-7 scenario, end to end.** Build a repo with an integration branch and an implementer
  worktree on a feature branch; add a detached review worktree at the same head; confirm the two
  coexist, that dirt in the review worktree does not touch the implementer's, and that removing
  the review worktree followed by the *Closeout* sequence still closes clean. The failure this
  replaces is already recorded above.
- **Greps.** `## Integration branch` present; no `Code Review` literal; the two resolution orders
  are byte-identical; `three PR review rounds` present; the precedence sentence present; the
  frontmatter `name` unchanged.
- **Carried sections byte-identical** by `diff` against `057a24a` for every section not listed in
  *The changes*.
- **Live resolution** through `~/.claude/skills/orchestrated-worktree-delivery/SKILL.md`, the one
  check that can break a skill in use today.
- **Word count recorded**, expected around 2100.

Real evidence is a cycle run on a repo with both an integration branch that is not `main` and a
tracker. There is none to run during this work, so it is a declared pending manual check rather
than a claim, per `superpowers:verification-before-completion`.

## Out of scope / follow-up

- **The word count.** The file lands near 2100 words against `writing-skills`' guidance of under
  500, and a large share is justificatory prose. A trim is its own task on its own branch.
- **The round-cap ambiguity.** "On the third escalate" still reads two ways against "rounds 2 and 3
  reuse the same branch" — does round 3 run, or is it replaced by the escalation? Step 7 is
  rewritten here for the re-push and the cap's unit, and this ambiguity is deliberately left
  untouched rather than resolved in passing.
- **The tracker-shaped steps.** Steps 1 and 4 read as unconditional even though *Ticket hook* has a
  no-tracker branch; keying them to an observable predicate is a separate change.
- **The frontmatter description**, which summarises the workflow — the failure mode
  `writing-skills` documents, where the agent follows the description instead of reading the body.
- **No rationalization table and no red-flags list**, on a file that is mostly prohibitions.
- **`In Progress` and `Done`**, the two surviving repo-specific status literals.
