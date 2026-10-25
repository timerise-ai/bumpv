# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

An [Agent Skill](https://agentskills.io) package: markdown only. There is no `package.json` here and nothing
in this repository executes. It teaches an agent to cut a Semantic Versioning release in any git repository:
find the version source, decide the level from the commits since the last tag, edit the manifest, changelog
and README release line, commit, and create an annotated tag, leaving the push to the user.

Keep the two straight: the `git` commands and manifest edits in `references/` run against the **target**
repository being released, not this one. This repository is released by its own procedure, under section 9
of the index's STANDARD.md.

The skill was written by the engineer who runs this release procedure. `references/provenance.md` is the
rationale layer: what the 0.1.0 audit fixed against the earlier single-file implementation, what was kept
deliberately and why, and what was added and not yet run in a release. Its entry count is stated in its
intro; keep it exact.

## Structure

- `SKILL.md`: entry point, loaded whole on every activation, so it stays between 130 and 160 lines, the
  closing index line aside. Frontmatter `description` is the trigger surface; the body carries the release
  flow diagram and the seam paragraph that stands in for `adaptation.md`, six critical facts, six hard
  rules, the invocation table, the quick start, the reference directory and the closing index line.
- `README.md`: the human-facing front door, in the section order of the index's STANDARD.md: install,
  activation, file table, six non-negotiables, requirements, *Not this*, contributing, footer.
- `references/version-sources.md`, `bump-level.md`, `release-files.md`, `commit-and-tag.md`,
  `provenance.md`: one topic each, loaded on demand, in the order the release runs.
- `evals/`: `prompts.md` holds what an operator types after installing. Prompt 1 works on the eval fixture
  itself, a git repository with no tags and no version field; prompts 2 and 3 clone public repositories.
  The skill builds no code, so the checks only confirm the fixture app was left intact and the notes carry
  the score. Every other file there is one eval run: measured frontmatter that is never edited, then the
  notes of the person who ran it. Add a prompt rather than rewording one that has results. The procedure is
  section 10 of the index's STANDARD.md.
- `.github/workflows/agent-eval.yml`: the caller of the index's reusable eval workflow, copied verbatim
  from STANDARD.md section 10. It is the same in every skill; never edit it, and never add a trigger on
  `push` or `pull_request`.

## Editing conventions

- **Commands are checked before they change.** Run any changed `git` command in a scratch repository first;
  the annotated-tag claim was verified by pushing a lightweight and an annotated tag with
  `git push --follow-tags` and reading `git ls-remote --tags`.
- **Identifiers are shared across files.** The invocation forms (`/bumpv`, `patch`, `minor`, `major`,
  `--dry-run`), the commit message `chore(release): X.Y.Z`, the tag form `vX.Y.Z` and the names "version
  source", "current-release line" and "release convention" appear in several files. Change them in all or
  none.
- **Keep the three tables in sync** with `references/`: the reference directory and the quick start in
  `SKILL.md`, and the file table in `README.md`. Links are relative.
- **The hard rules and the non-negotiables move together**: the six hard rules in `SKILL.md` and the six
  non-negotiables in `README.md` state the same six things in the same order, and neither is ever presented
  as optional.
- **The odd-looking parts stay.** Refusing a dirty tree, staging by name, the `git cat-file -t` check, the
  lower level on the fence, the narrow condition for the context-file note: each has an entry under *Kept
  deliberately* in `provenance.md`. Read it before simplifying.
- **Mark additions as additions.** Anything not yet run in a release belongs under *Added* in
  `provenance.md`, and the ledger's entry count in its intro moves with it.
- **Nothing is renamed by the target.** The target repository supplies its manifest, tag prefix, changelog
  location and release convention; every name in this skill is the skill's contract.
- **Plain punctuation, wrapped at 110 columns.** No em-dashes, en-dashes, arrows, middle dots or smart quotes
  in prose or tables; ranges are written "1 to 4".
- Commits follow Conventional Commits with no AI attribution trailers; releases follow the index's
  STANDARD.md, where the release commit carries only `CHANGELOG.md` and the README version line.
