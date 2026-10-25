# Version sources and the commit range

This reference finds the current version, the last release tag and the commits since it. It assumes a git
repository with a clean tree, or a dry run.

## The version source

Detect the canonical version from the first match in this table. This is the step that makes the skill
work outside Node projects.

| Project | Manifest | Version field |
|---|---|---|
| Node | `package.json` | `"version"` |
| Rust | `Cargo.toml` | `[package] version` |
| Python | `pyproject.toml` | `[project] version` or `[tool.poetry] version` |
| Python, legacy | `setup.py`, `setup.cfg` or `<pkg>/__init__.py` | `version=` or `__version__` |
| Go | none, the git tag is the version | none |
| PHP | `composer.json` | `"version"`, often omitted, then the tags |
| Ruby | `*.gemspec` or `lib/<gem>/version.rb` | `VERSION =` |
| .NET | `*.csproj` | `<Version>` |
| Dart and Flutter | `pubspec.yaml` | `version:` |
| Fallback | `VERSION` file, else the git tags | the file's contents, or the latest tag |

A manifest that exists without a version field, such as a private `package.json`, is not a version source:
fall through to the next row. When no row yields a version, the latest tag is the authoritative version and the
release edits no manifest. The tag created in [commit-and-tag.md](commit-and-tag.md) is then the whole bump.

### Other places the version is written

Some repositories repeat the version outside the manifest. Look for them now, so the release keeps them in
agreement:

- A current-release line in `README.md`, such as `The current release is **0.4.0**.`
- The newest section of `CHANGELOG.md`.

A repository whose only version record is its changelog and a README line, such as a markdown-only package,
is versioned by those two and the tag; there is no manifest to edit.

## The last release

```bash
git describe --tags --abbrev=0
```

It prints the nearest tag reachable from `HEAD`, or fails when there is none. Note the tag's prefix style,
`v1.2.3` or bare `1.2.3`, so the new tag matches it. A repository with no tags is making its first release.

## The commit range

- With a prior tag: `git log --oneline <last-tag>..HEAD`.
- First release, no tags: `git log --oneline`, the whole history, and the release is the project's first
  published version.

An empty range means there is nothing to release. Say so and stop, rather than tagging the same tree twice.

## When the sources disagree

If the latest tag and the manifest name different versions, or two manifests do, show both values and ask
which is authoritative before bumping. Do not pick one: a wrong choice moves every later release.

## Monorepos

This skill versions a single package. In a workspace, run it once per package directory, where that
package's manifest and tag scheme apply, or use the workspace's own release tooling.

## Pre-release and build metadata

Add a pre-release or build suffix, such as `-rc.1` or `+build.5`, only when the user asks. The default is a
plain `X.Y.Z`.
