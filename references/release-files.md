# The files a release edits

This reference makes the release's own edits: the manifest, the changelog, a README current-release line
and, when warranted, the AI context file. It assumes the new version from [bump-level.md](bump-level.md)
and the sources found in [version-sources.md](version-sources.md).

## The repository's convention first

Read the repository's `CLAUDE.md`, `AGENTS.md`, `CONTRIBUTING.md` or release standard before editing. **A
documented release convention wins over every default in this file**: a rule on where the changelog lives,
how a section opens, or which files the release commit may carry. When a convention limits the release
commit to named files, the release edits only those files, and the context-file note below is skipped.

## The manifest

Update the version field found in [version-sources.md](version-sources.md) to the new number, and nothing
else in the file. A tag-only project has no manifest edit.

## The changelog

Record the release in the most conventional location that exists:

1. `CHANGELOG.md` in [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) style: a new
   `## [X.Y.Z] - YYYY-MM-DD` section at the top, under the file's preamble and above the previous release.
   An `## [Unreleased]` section, when present, becomes the new release section.
2. A "Version History" or "Changelog" section in `README.md`: add the release there.
3. Neither: create `CHANGELOG.md` with the Keep a Changelog preamble and the new section.

Write the section from the commits in the range, for a person reading release notes: open with one or two
sentences on what the release is, then `### Added`, `### Changed`, `### Fixed` and `### Removed` as needed.
Call out breaking changes explicitly, with what broke and how to migrate. No raw commit hashes. A release
that changes only documentation says so in its first line.

The date is today's date in `YYYY-MM-DD`.

## The README current-release line

When `README.md` states the current version, such as `The current release is **0.4.0**.`, change the
number to the new version. Change only the number: the line's wording is the repository's.

## The AI context file

When the repository has a `CLAUDE.md` or `AGENTS.md`, and the release changes how an assistant should work
with the codebase, add a short "Recent update" note: the new version, the date, and the changes that matter
architecturally, such as new API routes, schema changes, or a changed processing flow or architecture.

Skip it for routine UI changes, documentation-only releases and anything without architectural weight, and
whenever the repository's release convention limits what the release commit carries. The context file is
read on every session, so a note that adds nothing costs every later session.

## After the edits

Every place that names the version now names the new one: the manifest, the newest changelog section and
the README line. The tag created in [commit-and-tag.md](commit-and-tag.md) makes the fourth. In a dry run,
the edits stay in the working tree, uncommitted, for the user to inspect with `git diff`.
