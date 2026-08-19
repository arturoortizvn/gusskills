# Design — `pr-peer-review`, lean skill plus four corrections

**Date:** 2026-08-19
**Status:** approved (design); partly superseded during implementation — see *Reversal* below
**Deliverable:** `pr-peer-review/SKILL.md` cut to its operative rules with four defects corrected, a
new `pr-peer-review/rationale.md`, and `README.md`'s layout block updated. No other file changes.

## Why

`pr-peer-review` is 1707 words over 203 lines, and like the delivery cycle before it, a large share
argues *why* each rule exists rather than stating it. The same split applies for the same reason:
the argument is what stops someone undoing a rule whose reason is invisible, but it loads into every
session and its bulk is what lets contradictions hide.

A review of the file found four defects that are not stylistic:

| # | Defect | Evidence |
|---|---|---|
| 1 | `graph-review` does not exist | Referenced at `:3` (the frontmatter's negative trigger), `:33` and `:202`. Not in `~/.claude/skills/` (eleven skills, none of them that), not in the plugin cache, not named in `CLAUDE.md` or `personal-projects.md` |
| 2 | The mandated `gh pr checkout` collides with the delivery cycle | Procedure step 2 states the checkout unconditionally. `orchestrated-worktree-delivery` puts the reviewer in a detached worktree already at the PR head, where that command dies with `fatal: '<branch>' is already used by worktree at '<path>'`. Today it works only because the cycle overrides this file from outside |
| 3 | Relocation is an offer here and a prerequisite there | Step 5: "It is an offer, not a requirement". The cycle: a repo whose review skill still sits in `~/.claude/skills/` "means moving that skill into its repo first". The two cannot both be right |
| 4 | Standards resolution cannot see `idanalyzer-peer-review` | Step 1 looks only under the repo's `.claude/skills/`. That skill lives in `~/.claude/skills/` and its description names `LendLogic/LendLogic.IDAnalyzer`, so an IDAnalyzer PR falls through to derived checks while a bespoke skill for it exists |

Defects 3 and 4 are one knot seen from both sides, and this pass unties it.

## Decisions

| Decision | Choice |
|---|---|
| The split | A sibling `rationale.md`, referenced from `SKILL.md` in one line, same shape as the delivery cycle's |
| Defect 1 | `graph-review` becomes `superpowers:requesting-code-review`, which exists and is the author-side skill; the `/code-review` sweep this file already allows at step 3 stays as it is |
| Defect 2 | Step 2 gains a condition: where the tree is already at the PR's head, no checkout is performed |
| Defects 3 and 4 | **The relocation is required.** Resolution keeps grading only with a skill under the repo's own `.claude/skills/`; a repo-specific review skill found anywhere else stops the review until it is moved into the repo it describes |
| No escape clause | The requirement is stated without an exemption. A rule with "unless it matters" attached reopens the negotiation it exists to close |
| `description` | Triggering conditions and symptoms only, no summary of the procedure |
| New sections | A rationalizations table and a red-flags list, built from failures this skill actually produced |
| Size target | 1300-1500 words, derived by measurement rather than aspiration — see below |
| Safety criterion | The invariant list **and** an imperative sweep — see *Verification* |

### Why the size target is 1300-1500 and not 500

The previous skill's design set a target by estimate and missed it by 70%, because it never measured
what the rules alone cost. Measured here: the output template is roughly 180 words and is entirely
operative; the eight universal checks, the four-level scale and the five discipline lines are rules
with no argument to remove; the new table and red flags add roughly 290. `writing-skills`' guidance
of under 500 words is unreachable for this file without deleting the output template, and the
template is what makes two reviews of two different repos come back in the same shape. The target is
therefore what the rules actually weigh, and the count is reported, not enforced.

## The split

`SKILL.md` keeps: the frontmatter, a two-sentence overview, the no-concrete-paths rule, standards
resolution, the severity scale, the eight universal checks with their precedence line, the five
discipline lines, the five procedure steps with their commands and guards, the output template with
its verdict logic, the closing list of what the skill does not do, and the two new sections.

`rationale.md` receives, each under a heading naming the rule it explains: why a review skill kept
outside the repo it describes ages in silence; why the scale is stated as a principle rather than as
one repo's breaches; why the universal checks need no repo doc; why every discipline line exists
(they came from real findings); why a closed PR gets a file instead of a comment; why a second round
is a second comment rather than an edit; why the three dispositions are not interchangeable; and why
the relocation is a requirement rather than an offer.

## The four corrections, as they land

**Defect 1.** The frontmatter's negative trigger and the closing boundary both name
`superpowers:requesting-code-review`. The parenthetical at `:33` keeps its point — a skill about
reviewing your own work does not satisfy step 1 — with `code-review` as the surviving example.

**Defect 2.** Step 2 becomes: read the PR, then work from its head — checking it out only if the
tree is not already there. The `fatal:` this avoids is quoted where the condition is stated, because
a reader who ignores the condition meets that message.

**Defect 3 and 4.** Standards resolution gains a step before grading: if a review skill specific to
this repo exists outside it — the reviewer's own `~/.claude/skills/` is where this happens — the
review stops, and the move into that repo is its own task on its own branch.

**The test for "specific to this repo" is that the skill's name or description names that repo**, and
it is what keeps the gate from firing on itself: `pr-peer-review` lives outside every repo it reviews
and names none of them, which is invariant 2 doing double duty. `idanalyzer-peer-review` names
`LendLogic/LendLogic.IDAnalyzer` in its description, so it trips the gate — correctly, since that is
the case the rule exists for. Step 5's "offer, not a
requirement" goes. The consequence is accepted deliberately: an IDAnalyzer PR cannot be reviewed
with its bespoke checks until `idanalyzer-peer-review` moves into that repo, which `README.md`
already records as the outstanding case.

## Verification

The delivery cycle's split taught that an invariant list is necessary and **not sufficient**: all 28
of its invariants survived while two action-bearing sentences leaked into the companion file. So this
pass has two criteria, and the second is the new one.

### 1. The invariant list

Each item exists in the file at `5d6670c` and must exist in the produced `SKILL.md`:

1. Exactly one PR comment per review round, and never an edit of or a re-post over an earlier one.
2. This skill names no concrete repository path, ever.
3. Standards resolution grades with the repo's own review skill when one exists under its
   `.claude/skills/`, and the test is that skill's description, not its directory name.
4. A skill about reviewing your own work before opening a PR does not satisfy that step.
5. Where the repo has its own review skill, its checks and its severities govern; the universal
   checks cover only what it is silent about.
6. With no such skill, the checks are derived from the repo's norms in the stated order.
7. The derived list is written before grading, N checks each with a severity and the doc line that
   backs it; an invented invariant is not a finding; the comment states which norms it graded
   against, and says so plainly when the repo had none.
8. The four severity levels, each stated as a principle.
9. All eight universal checks, each with its severity.
10. Claims in the PR body are verified against the base branch, never against the PR's description
    of itself.
11. Guards are tested by mutating them.
12. Experiments run in a clean tree, or the result is declared provisional.
13. Unsubstantiated findings are discarded explicitly with the reason, under *Checked and cleared*.
14. Tree-scanning checks mean nothing without the head checkout.
15. Scope is GitHub via `gh`; with no remote or no PR, say so and refer to
    `superpowers:requesting-code-review`.
16. The comment is written in English.
17. The PR is read before it is graded, and the grading happens against its head.
18. The working tree must be clean before any checkout, and the user's work is never clobbered.
19. A PR whose state is not `OPEN` gets the comment written to a file, and the review stops.
20. A `/code-review` first pass is optional and its output is never posted raw.
21. The active `gh` account must match the repo per `~/.claude/personal-projects.md`; on a mismatch,
    stop and ask for the switch; no issues, no email, no Slack.
22. The output template, with all four severity headers always present and `_None._` under the empty
    ones.
23. Every finding cites `file:line`.
24. The three dispositions are not interchangeable, and no check is left out of all three.
25. The heading carries the ticket, or the branch, or the literal `no ticket`.
26. The verdict logic: any 🔴 → Request changes; 🟠 without 🔴 → Comment; only 🟡/🔵 or none →
    Approve.
27. A pure-formatting diff gets the one-line response and stops.
28. The skill does not merge, does not approve through `gh pr review --approve`, does not open
    issues, does not commit, does not push, and does not modify the repo under review.
29. The boundary with the author-side skill is stated.
30. **New:** a review skill whose name or description names this specific repo, living anywhere
    outside it, stops the review until it is moved into that repo. A skill that names no repo — this
    one — never trips the gate.

### 2. The imperative sweep

This is what the invariant list alone would miss. From the file at `5d6670c`, build the list of
every sentence that tells the reader to do or not do something — every bold directive, every
`never`, `must`, `always`, `stop`, and every imperative verb opening a clause. Each one must appear
in the produced `SKILL.md`. A directive that appears only in `rationale.md` is a defect, whatever the
invariant list says, because the companion file is explicitly for reading before relaxing a rule,
not before following one.

### 3. Machine-checkable

No `graph-review` anywhere in the skill directory; `superpowers:requesting-code-review` present;
`rationale.md` present and referenced from `SKILL.md`; the `description` naming no procedure step;
`README.md`'s layout block listing both files; the `fatal:` literal for the checkout condition
present and on one line, checked with `grep -F` on the complete string; both files resolving through
`~/.claude/skills/pr-peer-review/`; and the word count reported.

## Out of scope

- **The delivery cycle.** It is not touched: the relocation decision was chosen precisely so the two
  skills agree without editing it.
- **The staleness this pass creates in the delivery cycle.** Correction 2 leaves
  `orchestrated-worktree-delivery/SKILL.md:95-96` stale: it still says this skill runs
  `gh pr checkout` unconditionally. Only that justification clause is affected — the directive it
  supports still stands — and it is routed to its own branch rather than repaired here, for the
  reason in the bullet above.
- **The missing triage point for deferred minors** — the cycle suppresses the dispatch skill's
  whole-branch review, which is where that skill assigns that triage, and this file never reads a
  ledger. Real gap, inherited, needs its own design.
- **Moving `idanalyzer-peer-review` into IDAnalyzer.** Separate repo, separate branch, and now a
  prerequisite for reviewing that repo's PRs.

## Reversal

**2026-08-19, after implementation.** The split did not happen. Measurement during implementation
put the companion at 769 words of newly written prose against only about 118 words carried out of
`SKILL.md`, and found roughly 150 words of removable pure argument in a file the *Why* section
above had assumed was roughly half argument. That assumption is what the split rested on, and it
did not survive being measured.

The diagnosis is the design's, not the implementation's. This document estimated the shape of the
argument instead of measuring it — the same mistake it names the delivery cycle's design as having
made, and the reason given above for why 1300-1500 replaced 500. The correction that section applied
to the *size* estimate did not get applied to the *proportion* estimate the split itself was built
on.

Faced with that, the user decided to drop the split rather than force it to fit: no
`rationale.md`, no relocation of prose out of `SKILL.md`, and no further cutting to chase
1300-1500. The four corrections and the two new sections — the rationalizations table and the
red-flags list — ship inside the one file instead.

What shipped: a single `pr-peer-review/SKILL.md` at 2263 words, no companion file, `README.md`
left untouched. All 30 invariants listed under *Verification* and all 83 directives catalogued by
the imperative sweep are present in that one file.

Of this document, what still binds: the invariant list, correction 1 as specified, the
no-escape-clause decision on defects 3 and 4, and the imperative-sweep detection test — each
describes a property the produced file must have, and it does. Corrections 2 and 3/4 bind in
substance but not in the wording above: implementation reworded correction 2 so the
already-checked-out guard is decidable (`headRefOid` compared against `git rev-parse HEAD`), and
reworded the defects 3/4 gate to add the `review` qualifier its first wording lacked. Where this
document's phrasing of those two differs from `SKILL.md`, the shipped file is authoritative.

Invariant 30's phrasing in the *Verification* list above is now narrower than the shipped gate:
it exempts a skill by "names no repo," where the gate exempts by purpose — a skill not about
reviewing that repo's pull requests, which includes one about running a repo locally — and it
carries no remedy for a repo that has both its own review skill and an outside duplicate. The
invariant's wording is superseded, not the gate wrong: a later audit should compare against
`SKILL.md` itself rather than against invariant 30's older phrasing. Revising the invariant list
to match is left to its own task.

What is now historical only: the *Why* section's premise that roughly half the file is argument,
*The split* section's division of material between two files, and the 1300-1500 size target
together with its justification — they record what was approved before implementation measured
the file and found that premise false.

The cost: the lean-skill goal is abandoned for this file rather than pursued by cutting rules, and
`pr-peer-review/SKILL.md` is now roughly 500 words larger than the 1707 it started at, entirely in
content the user approved.
