# Deciding the bump level

This reference turns the commit range from [version-sources.md](version-sources.md) into MAJOR, MINOR or
PATCH. A level the user forced, as in `/bumpv minor`, is used as given and this file is skipped.

## The rule

Follow [Semantic Versioning](https://semver.org/):

- **MAJOR, X.0.0**: changes incompatible with existing usage.
- **MINOR, 0.X.0**: new functionality that keeps existing usage working.
- **PATCH, 0.0.X**: fixes and improvements that keep existing usage working.

**The highest-impact change in the range sets the level**, MAJOR over MINOR over PATCH. The range is judged
as a whole: several features together are still one MINOR. **When a change is genuinely on the fence, take
the lower level**, since a release can be followed by a bigger one but not taken back.

A repository that documents its own mapping, for example "a new reference is a MINOR", applies that mapping
first; the tables below fill in what it leaves open.

## Commit types to levels

**MAJOR, breaking changes**

- `BREAKING CHANGE:` in a commit footer, or `!` after the type, as in `feat!:`
- Removed or renamed public APIs, components or functions
- Request or response shape changes on existing API routes
- Schema changes that require a migration
- A changed auth or security model

**MINOR, new features**

- `feat:` or `new:` commits that add functionality
- New endpoints, components, pages or config options
- Enhancements to existing features that do not break callers

**PATCH, fixes and maintenance**

- `fix:`, `perf:`, and `refactor:` with no behaviour change
- `style:`, `docs:`, `test:`, `chore:`, dependency bumps

When the commits do not follow Conventional Commits, infer the intent from the messages and the diffs rather
than from a prefix.

## First release

With no tags, the release is the first published version. Use the manifest's current version when it has
one and nothing has been tagged at it; otherwise `0.1.0` for a project still settling its interface, or what
the user names.

## Examples

**PATCH, 0.1.5 to 0.1.6**: fixes and maintenance only.

```
git log --oneline v0.1.5..HEAD
fix(auth): resolve login redirect issue
fix(dashboard): correct chart rendering bug
docs: update installation instructions
chore: update dependencies
```

**MINOR, 0.1.6 to 0.2.0**: new features, nothing broken.

```
git log --oneline v0.1.6..HEAD
feat(reports): add CSV export
feat(dashboard): energy balance charts
fix(auth): session timeout
refactor(api): optimize queries
```

**MAJOR, 0.2.0 to 1.0.0**: a breaking change.

```
git log --oneline v0.2.0..HEAD
feat(api): redesign authentication

BREAKING CHANGE: endpoints now require JWT instead of session cookies
feat(dashboard): real-time monitoring
fix(reports): calculation errors
```

**A project without `package.json`, Rust, 0.4.2 to 0.5.0**: the same rule, a different manifest. The
version lives in `Cargo.toml` under `[package] version`, and the tags are bare, `0.4.2` rather than
`v0.4.2`.

```
git log --oneline 0.4.2..HEAD
feat(client): add async retry policy
fix(parser): handle empty payloads
```

The release updates `Cargo.toml`, writes the `CHANGELOG.md` section and tags `0.5.0`.
