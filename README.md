# gusskills

Personal Claude Code skills, versioned. Each top-level directory is one skill and holds a
`SKILL.md`; `~/.claude/skills/<name>` is a symlink into this clone, so editing a file here is
editing the live skill.

## Layout

```
gusskills/
  pr-peer-review/{SKILL.md,rationale.md}
  orchestrated-worktree-delivery/{SKILL.md,rationale.md}
  docs/superpowers/{specs,plans}/
```

`~/.claude/skills/pr-peer-review -> ../../Proyectos/Personales/gusskills/pr-peer-review`

The relative form is deliberate: `~/.claude/skills/../../` resolves to `~/`, and it is the
same shape the insforge skills already use, which is how we know skill resolution follows a
symlink.

**Review this repo's PRs from a second clone.** Because the symlinks resolve into this working
tree, `gh pr checkout` here rewrites the live skills: the reviewer's own governing instructions
change mid-review, and a branch left checked out leaves the machine running unmerged code. Clone
it again somewhere else and check the PR out there.

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
