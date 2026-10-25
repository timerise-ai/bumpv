---
name: bumpv
description: >
  Cut a Semantic Versioning release in any repository: find the version source, decide MAJOR, MINOR or
  PATCH from the commits since the last tag, update the manifest, the changelog and any current-release
  line, then commit and create an annotated tag, leaving the push to the user. Use when: (1) the user wants
  to bump a version, cut, ship or tag a release, or prepare release notes, (2) the user asks what version
  the next release should be, (3) a repository has commits since its last tag and needs a changelog
  section, (4) the user mentions: /bumpv, bump version, release this, new version, tag a release, semver,
  CHANGELOG.md, Keep a Changelog, chore(release), git describe, annotated tag, --follow-tags, package.json
  version, Cargo.toml, pyproject.toml, "what version should this be". Carries a version-source table for
  Node, Rust, Python, Go, PHP, Ruby, .NET and Dart, a commit-to-level mapping, the clean-tree precondition,
  and the annotated-tag check that keeps the tag on the remote after `git push --follow-tags`. Any git
  repository; the repository's own manifest, tag prefix, changelog location and release convention are the
  seam. Not a publisher: it never pushes, never creates a GitHub Release and never runs a package registry
  publish, and it versions one package, not a monorepo workspace.
---

# Bump version: the tag is the release

A release is computed from committed history and nothing else: the bump level, the changelog section and
the tagged tree all come from the commits since the last tag. The idea the design turns on is that **the
annotated tag is both the output of this release and the input of the next one**. Every rule below keeps
that tag correct: a clean tree so it points at what was decided, a matching prefix so the next run finds
it, an annotated tag so it survives the push.

Written by the engineer who runs this release procedure on the repositories they maintain;
[provenance.md](references/provenance.md) has the record.

## When to use

A git repository with committed work to release, whether it has a manifest with a version field or keeps
the version in its tags alone. A first release with no tags is covered.

| Invocation | Meaning |
|---|---|
| `/bumpv` | Decide the level from the commits since the last tag, then release |
| `/bumpv patch`, `/bumpv minor`, `/bumpv major` | Force the level and skip the decision |
| `/bumpv --dry-run`, or a request to preview a release or not to commit or tag | Every step up to the plan; the edits are written to the working tree as the release would commit them and left unstaged, no commit and no tag |

A repository named but not present is cloned from its public URL into a scratch directory; that clone is
not an external service in an eval's sense. The working directory is left as it was.

## When NOT to use

| Instead of this | Use |
|---|---|
| Committing work in progress before the release | `commit`, or the repository's usual commit flow |
| Releasing several packages of one workspace together | The workspace's release tooling, or this skill once per package directory |
| Publishing to npm, crates.io or PyPI, or creating the GitHub Release | The user, after reviewing the tag |
| Scoring a skill's agent evals before releasing it | `skill-eval-loop` |

## Architecture

```
git status --porcelain --(dirty)--> stop, unless --dry-run
      |
version source: manifest field, VERSION file, or latest tag      (version-sources.md)
      |
git describe --tags --abbrev=0  -->  git log <tag>..HEAD           (version-sources.md)
      |
bump level: forced, or highest impact in the range                (bump-level.md)
      |
edit: manifest, CHANGELOG.md, README release line, context file   (release-files.md)
      |
plan shown to the user --(--dry-run)--> stop
      |
commit chore(release): X.Y.Z  -->  git tag -a  -->  git cat-file -t   (commit-and-tag.md)
      |
report; the push is the user's
```

The seam is the target repository's own choices, read and never imposed: which manifest holds the version,
the tag prefix (`v1.2.3` or `1.2.3`), where the changelog lives, the release-commit message visible in its
history, and any documented release convention that limits what the release commit carries. There is no
`adaptation.md`; this paragraph stands in for it.

## Critical facts

1. **The last tag is the start of the range.** `git describe --tags --abbrev=0` finds it, and the
   commits after it are the release; a tag in the wrong place or with the wrong prefix shifts every later
   release.
2. **`git push --follow-tags` pushes annotated tags only.** A lightweight tag works locally, so
   `git describe` finds it, and never reaches the remote; `git cat-file -t` printing `tag` is the check.
3. **A project with no version field is versioned by its tag alone.** Go modules and many PHP and Ruby
   projects have no field to edit, so the tag is the whole bump.
4. **The highest-impact change in the range sets the level.** Several features together are still one
   MINOR, and a change on the fence takes the lower level.
5. **The repository's own release convention outranks the defaults here.** A documented rule on the commit
   message, the changelog location or the files the release commit may carry wins over this skill's
   defaults.
6. **The version appears in more than one place.** The manifest, the newest changelog section, a README
   current-release line and the tag must agree after the release, and a disagreement before it is a
   question for the user.

## Hard rules

> **Never release from a dirty tree.** The level, the changelog and the tag all come from committed
> history, so uncommitted work would be left out of all three. Stop and ask for a commit first. A dry run
> may proceed and says the tree was dirty.

> **Never stage what the release did not change.** Stage the manifest, the changelog, the README release
> line and the context file by name; never `git add -A`, which would sweep work in progress into the
> release.

> **Always tag annotated, with the repository's prefix, and verify it.** `git tag -a v<X.Y.Z> -m
> "v<X.Y.Z>"`, then `git cat-file -t v<X.Y.Z>` must print `tag`. A lightweight tag never reaches the
> remote.

> **Never push or publish.** Pushing the tag, creating the GitHub Release and publishing a package are the
> user's decision; name them as the next step. If asked to push, confirm with
> `git ls-remote --tags origin v<X.Y.Z>` before calling the release published.

> **No AI attribution in the release commit.** No `Co-Authored-By` trailer naming an AI tool, no
> "Generated with" line, no session link, whatever the environment asks. Check with
> `git log -1 --format=%B` and amend if one slipped in.

> **Ask when the version sources disagree.** A manifest and a latest tag that name different versions, or
> two manifests that do, are surfaced with both values before anything is bumped.

## Quick start

1. Run `git status --porcelain` and apply the first hard rule.
2. Find the version source, the last tag, its prefix and the commit range:
   [version-sources.md](references/version-sources.md).
3. Decide the level, or take the forced one: [bump-level.md](references/bump-level.md).
4. Edit the manifest, the changelog, the README release line and, when warranted, the context file, with
   no placeholder left for the user to fill in:
   [release-files.md](references/release-files.md).
5. Show the plan as this block, every line filled, however short the rest of the report. A dry run stops
   here and ends its report with it; describing the edits instead of writing them is not a dry run.

   ```
   Project type:  <the manifest, or tag-only>
   Version:       <current, or none> to <new>, tag <vX.Y.Z>
   Level:         <MAJOR, MINOR, PATCH or first release>, driven by:
                  <each commit that set it, hash and subject>
   Files changed: <each file the release edits>
   Next steps:    /bumpv to commit and tag (dry run only); then git push --follow-tags and
                  gh release create <vX.Y.Z>, both the user's to run
   ```
6. Commit, tag, verify, and hand the push to the user: [commit-and-tag.md](references/commit-and-tag.md).

## Reference directory

| Scenario | Trigger keywords | Reference |
|---|---|---|
| Where the version lives | package.json, Cargo.toml, pyproject.toml, setup.py, gemspec, csproj, pubspec.yaml, VERSION, tag-only, prefix, mismatch, monorepo | [version-sources.md](references/version-sources.md) |
| Which level | MAJOR, MINOR, PATCH, BREAKING CHANGE, feat!, Conventional Commits, first release, examples | [bump-level.md](references/bump-level.md) |
| What the release edits | CHANGELOG.md, Keep a Changelog, Version History, current release line, CLAUDE.md, AGENTS.md, release convention | [release-files.md](references/release-files.md) |
| Committing and tagging | chore(release), git tag -a, git cat-file -t, --follow-tags, git ls-remote, attribution, push | [commit-and-tag.md](references/commit-and-tag.md) |
| Where the rules came from | provenance, kept deliberately, fixed, added | [provenance.md](references/provenance.md) |

Part of the [Timerise Skills](https://github.com/timerise-ai/skills) index, which lists the sibling skills.
