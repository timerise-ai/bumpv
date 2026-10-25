---
prompts:
  - prompt: "Preview the first release of this app: which version it should be, why, and the release notes you would write. Do not commit or tag anything."
    stack: No data store
  - prompt: "Clone https://github.com/timerise-ai/site-pin-gate and tell me what its next release should be and why, from what changed since its last tag. Leave the clone unchanged."
    stack: No data store
  - prompt: "Clone https://github.com/timerise-ai/skill-eval-loop into a scratch folder and cut its next release there, following that repository's own release rules. Do not push."
    stack: No data store
---

# Prompts

What an operator types after installing this skill, in their own words. An agent eval installs the skill
into an empty Next.js app, gives the agent one of these prompts and no further help, then type-checks,
builds and tests the result; the first prompt runs before every release. The fixture is itself a git
repository with no tags and a `package.json` without a version field, so prompt 1 works on the fixture and
shows how the agent finds a version source and plans a first release without committing. Prompts 2 and 3
name a public repository to clone: one to decide a level from real tagged history, one to cut a release that
must follow the target's own convention, with an annotated tag and no push. This skill builds no code, so
the checks only confirm the agent left the app intact: what a run shows is in its notes, scored against the
hard rules in `SKILL.md`. The results are the other files in this folder. Section 10 of
[STANDARD.md](https://github.com/timerise-ai/skills/blob/main/STANDARD.md) says how a run is made.
