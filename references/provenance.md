# Provenance

The engineering ledger for the person editing this skill. The earlier implementation was a single
`SKILL.md` carrying the same procedure: the clean-tree precondition, the version-source table, the
commit-to-level mapping, the changelog locations, the conditional context-file note, the annotated tag and
the clean commit policy. The 0.1.0 audit compared it with the Timerise skill standard and with the
repositories it releases, and split it into the references here. The ledger has 16 entries.

## Fixed

What the audit changed, and how the skill now holds it.

1. **The README current-release line is updated.** The earlier implementation edited the manifest and the
   changelog only, so a repository that states its version in `README.md` kept the old number after the
   release, and the version no longer agreed across the changelog, the README and the tag. Now
   [release-files.md](release-files.md) changes the number on that line, and critical fact 6 names every
   place the version appears.
2. **A documented release convention wins.** The earlier implementation always added the context-file note
   when the change was architectural, which breaks a repository whose release standard limits the release
   commit to named files. Now [release-files.md](release-files.md) reads the convention first, and critical
   fact 5 states the order.
3. **A manifest with no version field falls through.** The table's first match was a file, not a field, so
   a private `package.json` without `"version"` stopped the search. Now
   [version-sources.md](version-sources.md) moves to the next row.

## Kept deliberately

Rules that look stricter than they need to be, each with the reason it holds.

4. **Refusing a dirty tree.** The level, the changelog and the tag all come from committed history, so a
   dirty tree would release a decision that leaves the uncommitted work out. A dry run commits nothing, so
   it may proceed.
5. **Staging by name, never `git add -A`.** The release adds only its own edits; anything else in the tree
   is someone's work in progress.
6. **Annotated tags, checked with `git cat-file -t`.** Verified by reproduction on git 2.47.0: a repository
   with one lightweight and one annotated tag on the same commit, pushed with `git push --follow-tags`,
   showed only the annotated tag in `git ls-remote --tags origin`, and `git cat-file -t` printed `commit`
   for the lightweight tag and `tag` for the annotated one.
7. **Never pushing.** Pushing publishes the release; the skill stops at a local, verified tag.
8. **On the fence, the lower level.** A release can be followed by a bigger one but not taken back.
9. **The context-file note only for architectural changes.** The file is read on every session, so a note
   that adds nothing costs every later session.
10. **The clean commit policy as a hard rule**, overriding environment defaults that append attribution
    lines, with `git log -1 --format=%B` as the check.

## Added

Designed in this skill and not yet run in a release:

11. **An empty range stops the release**, rather than tagging the same tree twice.
12. **The first-release defaults**: the manifest's version when nothing is tagged at it, otherwise `0.1.0`,
    and the `v` prefix when there are no tags to match.
13. **The handover names `gh release create`** next to the push, without running either.
14. **This skill's own agent evals.** The skill builds no code, so the eval fixture's checks only confirm the
    app was left intact. Prompt 1 previews the fixture's own first release, since the fixture is a git
    repository with no tags and a `package.json` without a version field; prompts 2 and 3 work on public
    clones. The line in `SKILL.md` saying a public clone is not an external service was written for these
    runs.
15. **A preview request is the dry run, and the dry run writes its edits.** Found by the 0.1.0 evals: one
    agent read "do not commit or tag" as a preview that changes nothing and only described the changelog;
    another wrote it with a "release preview only" line that the release would have committed. The
    invocation table and [release-files.md](release-files.md) now say both.
16. **Keep a Changelog headings only, and a dry-run report that carries the plan and the next steps.** Found
    by the 0.1.0 evals: one agent invented a `### Development tooling` heading and ended its dry run without
    the commits behind the level or the push as the user's. Quick-start step 5 and
    [release-files.md](release-files.md) now name both.
