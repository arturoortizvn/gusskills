# Why peer review is shaped this way

Every rule in `SKILL.md` that is not self-evident has its argument here. Read this before relaxing
a rule, not before following one.

## Why this skill names no concrete repository path

That is what makes it safe to live outside the repos it reviews: a review skill kept in the
reviewer's home directory ages in silence, because nothing in a PR that renames a service ever
touches a file there. A check that cites a real path is a check that can rot without anyone
noticing, and it rots into a false negative — the reviewer runs it, it finds nothing, and nothing
is what it will always find.

Naming no concrete path is therefore this skill's anti-staleness by construction, not modesty about
its scope. The checks that would rot are exactly the ones it refuses to carry, and they belong
where a PR that breaks them also edits them: in that repo's own review skill.

## Why a review skill for a repo that lives outside it stops the review

Two things used to disagree. `orchestrated-worktree-delivery` treats a repo whose review skill
still sits in `~/.claude/skills/` as meaning that skill has to move into its repo first; this file
used to call the same move "an offer, not a requirement". Both cannot be right, and the offer was
the weaker half: standards resolution looked only under the repo's own `.claude/skills/`, so a
bespoke skill living anywhere else was invisible to the step that exists to find it. A PR then fell
through to derived checks while a skill written for exactly that repo sat unread, ageing.

Requiring the move closes both halves at once. The skill becomes reachable where resolution looks
for it, and it stops accumulating drift against a repo whose PRs never touch it. The requirement
carries no exemption on purpose: a rule with "unless it matters" attached reopens the negotiation
it exists to close, and "just this once" is how the skill stayed outside its repo in the first
place.

The test — the skill's name or description names *that specific repo* — is also what keeps the gate
from firing on this skill. `pr-peer-review` lives outside every repo it reviews and names none of
them, which is the no-concrete-paths rule doing double duty. The cost is accepted knowingly: a repo
whose bespoke skill has not moved yet cannot be reviewed with its bespoke checks until it does.
That is a smaller loss than reviewing it with checks nobody maintains.

## Why the severity scale is stated as a principle

Stated as a principle plus examples, because a list of one repo's breaches does not travel. A
catalogue of past incidents grades the repo it was written for and nothing else; the first review
of an unfamiliar repo has no such catalogue and still has to place a finding.

So the principle decides and the examples only illustrate. When a finding matches no example, the
question is the principle's — does merging it damage the service or make the repo's own
documentation lie — and not whether something like it has been seen before.

## Why the universal checks need no repo doc

These need no repo doc because they hold anywhere: a leaked key, extracted user data in a log,
untested new logic, a doc the diff makes false. Nothing about a particular repo is required to know
they are wrong.

That is also what makes them the floor when a repo documents nothing at all. Everything else has to
be derived from something written, which is why an invented invariant is not a finding — but a
repo with no docs is still not a repo where secrets in fixtures are acceptable.

## Why every discipline line is there

Each line came from a real finding, and this is the genuinely transferable part — the specific
checks belong to a repo, but the ways a review talks itself into a wrong answer are the same
everywhere.

- A guard with no coverage leaves the suite green while it guards nothing, so a guard the PR adds
  is only demonstrated by the test that fails when it is removed.
- On a dirty tree a result can be a false negative: the tree, not the PR, explains what was
  observed.
- Silence reads as "checked and fine", which is why a finding that could not be substantiated has
  to be discarded in writing rather than dropped.
- A tree-scanning check run from the base comes back clean whatever the PR does, and that is a
  false negative on a Blocker.

## Why a closed PR gets a file instead of a comment

Reviewing a closed PR is a calibration exercise, and the people who already shipped it do not need
the notification. The review still has value to the reviewer, and none of that value depends on
landing it in a thread that is finished.

## Why a second round is a second comment and never an edit

So the argument stays readable. An edit destroys the record: round one's findings, what the author
answered, and what the push actually addressed all collapse into a single revised text, and a
reader of the thread can no longer tell which objection was met and which was quietly withdrawn.
Two comments keep the sequence legible, and the second one can be short because the first is still
there.

## Why the three dispositions are not interchangeable

Each names a different state of knowledge, and every one of them is a different claim to the
author. *Checked and cleared* asserts a verification happened. *Could not verify* admits one did
not. *Not applicable* says the check has no subject in this repo.

Stretching the third into the first is the damaging case: a docs-only repo has no test suite, no
env vars and no request path, and reporting those checks as cleared claims evidence that was never
gathered. Inflated coverage is worse than a short review, because the next reader trusts it.
