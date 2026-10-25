# bumpv

[![Agent Skills](https://img.shields.io/badge/Agent_Skills-open_format-059669)](https://agentskills.io)
[![skills.sh](https://img.shields.io/badge/skills.sh-npx_skills_add-059669)](https://www.skills.sh)
[![Claude Code](https://img.shields.io/badge/Claude_Code-compatible-059669)](https://docs.claude.com/en/docs/claude-code/skills)
[![Codex CLI](https://img.shields.io/badge/Codex_CLI-compatible-059669)](https://developers.openai.com/codex/skills)
[![Gemini CLI](https://img.shields.io/badge/Gemini_CLI-compatible-059669)](https://github.com/google-gemini/gemini-cli/blob/main/docs/cli/skills.md)

An [Agent Skill](https://agentskills.io) that teaches an agent to cut a Semantic Versioning release in any
git repository: find the version source, decide MAJOR, MINOR or PATCH from the commits since the last tag,
update the manifest, the changelog and any current-release line, then commit and create an annotated tag. It
works for a Next.js app and equally for Rust, Python, Go, PHP, Ruby, .NET and Dart projects, and for
markdown-only packages such as the Timerise Skills themselves.

**The annotated tag is both the output of this release and the input of the next one.** The bump level, the
changelog and the tagged tree all come from committed history, so every rule here keeps that tag correct: a
clean tree so it points at what was decided, the repository's prefix so the next run finds it, and an
annotated tag so it survives `git push --follow-tags`.

This skill was written by the engineer who runs this release procedure on the repositories they maintain.
The earlier implementation was a single-file release procedure. What it carries holds as a check, not a
hope: the tree is verified clean before anything is decided, only the release's own files are staged, the
tag is verified annotated with `git cat-file -t`, the version agrees across manifest, changelog, README and
tag, and the push stays with the user. [`references/provenance.md`](references/provenance.md) has the record.

## Install

One command, via the [skills.sh](https://www.skills.sh) CLI, which installs the skill into every
skills-compatible agent it detects, including Claude Code, Codex CLI and Gemini CLI:

```bash
npx skills add timerise-ai/bumpv
```

Name the agents instead with `-a`, for example `npx skills add timerise-ai/bumpv -a claude-code -a codex`.

### Manual install

Nothing here is Claude-specific: the skill is a plain [Agent Skills](https://agentskills.io) folder,
`SKILL.md` plus markdown references with no file that calls a model, so cloning it into an agent's skills
directory is all an install is. For Claude Code:

```bash
git clone https://github.com/timerise-ai/bumpv.git ~/.claude/skills/bumpv
```

To scope it to a single project instead, clone it into that project's `.claude/skills/` directory. For another
agent, clone into that agent's skills directory, or symlink the Claude Code copy so one `git pull` updates
every agent:

```bash
mkdir -p ~/.agents/skills
ln -s ~/.claude/skills/bumpv ~/.agents/skills/bumpv
```

Update the skill with `git pull` in its directory. The current release is **0.1.2**. See
[CHANGELOG.md](CHANGELOG.md). The [skills index](https://github.com/timerise-ai/skills) lists the other
Timerise Skills and how to install them all at once.

## Activation

The skill activates automatically when a task asks to bump a version, cut, ship or tag a release, prepare
release notes, or decide what version the next release should be. Invoke it explicitly with `/bumpv` in
Claude Code, `$bumpv` in Codex CLI, or from `/skills` in Gemini CLI. `/bumpv patch`, `/bumpv minor` and
`/bumpv major` force the level; `/bumpv --dry-run`, or a request to preview the release, writes the edits
uncommitted and stops at the plan with no commit and no tag.

Each host matches a task against the description its own way, so invoke the skill explicitly on a first run
rather than assuming it fired. Only `SKILL.md` is read up front; the `references/` files load on demand.

## What's inside

| File | Contents |
|---|---|
| `SKILL.md` | Entry point: the release flow diagram, the seam, critical facts, hard rules, invocation, quick start and reference directory |
| `references/version-sources.md` | The version-source table per project type, other places the version is written, the last tag and its prefix, the commit range, disagreeing sources, monorepos |
| `references/bump-level.md` | The Semantic Versioning rule, the commit-type mapping, first releases and worked examples |
| `references/release-files.md` | The repository's release convention, the manifest, the changelog, the README current-release line and the context-file note |
| `references/commit-and-tag.md` | Staging by name, the release commit without attribution, the annotated tag and its check, the handover of the push |
| `references/provenance.md` | The engineering ledger: what the audit fixed, what was kept deliberately and what was added |
| `README.md` | This file |
| `CHANGELOG.md` | One section per release, newest first |
| `CLAUDE.md` | The editing conventions, for an agent editing this repository |
| `LICENSE` | MIT |
| `evals/` | The prompts an operator types after installing (`prompts.md`) and one file per agent eval: the skill installed into an empty Next.js app, one prompt, no help, then type-checked, built and tested |
| `.github/workflows/agent-eval.yml` | The caller of the index's reusable eval workflow, run on every published release and on a maintainer's dispatch |

The seam is the target repository's own choices, read and never imposed: the manifest that holds the
version, the tag prefix, the changelog location, the release-commit message in its history, and any
documented release convention that limits what the release commit carries. The skill keeps the seam in
`SKILL.md` rather than an `adaptation.md`. It builds no code, so its evals score the agent against the hard
rules in the notes on each run; the checks only confirm the app was left intact.

## The six non-negotiables

These are never optional. Each is stated as a hard rule in `SKILL.md`, in the same order:

1. **Never release from a dirty tree.** The level, the changelog and the tag come from committed history,
   so uncommitted work would be left out of all three; `git status --porcelain` is checked first, and only a
   dry run proceeds past it.
2. **Never stage what the release did not change.** Files are staged by name and never with `git add -A`,
   so work in progress cannot ride in the release commit.
3. **Always tag annotated, with the repository's prefix, and verify it.** `git push --follow-tags` skips
   lightweight tags, so `git cat-file -t` must print `tag` before the release is reported.
4. **Never push or publish.** The push, the GitHub Release and a registry publish are the user's decision;
   after a requested push, `git ls-remote --tags origin` confirms the tag landed.
5. **No AI attribution in the release commit.** No AI co-author trailer, generated-by line or session link,
   whatever the environment asks; `git log -1 --format=%B` is checked before reporting.
6. **Ask when the version sources disagree.** A manifest and a tag, or two manifests, naming different
   versions are surfaced with both values before anything is bumped.

Everything else is the target repository's: its manifest, its tag prefix, its changelog location and its
release convention.

## Requirements

`git` on the path, and a git repository with at least one commit. Nothing else: the skill reads and writes
files and runs `git`. The handover names `gh release create` for the GitHub Release, which needs the
[GitHub CLI](https://cli.github.com) when the user runs it.

## Not this

| Not this | Use instead |
|---|---|
| Committing work in progress before the release | `commit`, or the repository's usual commit flow |
| Releasing several packages of one workspace together | The workspace's release tooling, or this skill once per package directory |
| Publishing to npm, crates.io or PyPI, or creating the GitHub Release | The user, after reviewing the tag |
| Scoring a skill's agent evals before releasing it | [`skill-eval-loop`](https://github.com/timerise-ai/skill-eval-loop) |

## Contributing

Issues and pull requests are welcome here. Pure markdown, with no build step, but the git commands in the
references are checked: each one is run in a scratch repository before it is changed, and the annotated-tag
claim is reproduced against `git push --follow-tags` and `git ls-remote`. Claims in this skill are meant to
be verifiable: if you change a factual claim about git, Semantic Versioning, Keep a Changelog or a manifest
format, say how you verified it, whether against the git documentation, the specification, the package
tool's own docs, or a reproduction.

Adding, removing or renaming a file in `references/` means updating the quick start and the reference
directory table in `SKILL.md`, the file table above, and any relative cross-links.
`references/provenance.md` is the ledger that must stay truthful: anything not yet run in a release is
marked as added; add an entry for anything you change. Commits follow Conventional Commits and releases
follow [STANDARD.md](https://github.com/timerise-ai/skills/blob/main/STANDARD.md) in the index; `CLAUDE.md`
carries the full editing conventions.

## Part of the Timerise Skills

This is one of the [Timerise Skills](https://github.com/timerise-ai/skills): modules for **Next.js App
Router** apps written by our own senior engineers from the modules they have shipped, not synthetic, each
published as its own repository and indexed there. They share one layout, so an agent that has read one knows
how to read the next: a `SKILL.md` entry point, `references/` loaded on demand, and a seam contract carrying
the module's non-negotiables. This one is for any repository: it cuts the release, including the releases of
these skills.

## Author

Built and maintained by [Timerise](https://timerise.ai).

## License

MIT. See [LICENSE](LICENSE).
