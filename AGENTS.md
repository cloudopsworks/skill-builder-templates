# Repository Instructions

This repository is a CloudOps Works template for packaging agent skills across
multiple clients.

## Primary automation rule

When changing, adding, or removing a skill, keep installation automation current for all supported targets:
- Claude Code
- OpenCode
- Codex

Template repositories should not contain concrete skill payloads by default.
In repositories generated from this template, every top-level directory
containing `SKILL.md` is a distributable CloudOps Works skill.

Install and upgrade automation must be maintained in:
- `scripts/install-cloudopsworks-skills.sh`
- `scripts/upgrade-cloudopsworks-skills.sh`

Backward-compatible single-skill wrappers may exist, but the generic multi-skill installers are the source of truth.

## Installation contract

- Claude Code installs full skill directories under `~/.claude/skills/`.
- Codex installs full skill directories under `~/.codex/skills/` or `$CODEX_HOME/skills/`.
- OpenCode installs generated command files under `~/.config/opencode/commands/` or `$OPENCODE_HOME/commands/`.
- Default behavior should favor low-maintenance automation:
  - auto-discover current repo skills from top-level `*/SKILL.md`
  - symlink for directory-based targets when safe
  - forceable overwrite for upgrades
  - generated OpenCode command output derived from each skill's `SKILL.md`

## Documentation treatment

- `README.yaml` is the maintained source of truth for repository documentation.
- `README.md` is generated output and must be regenerated with `tronador readme build` as the last documentation step.
- When documentation changes, edit `README.yaml` first, then regenerate `README.md`.
- Run `tronador docs targets` first when Makefile targets or dependency documentation changed.
- Prefer adding reusable documentation workflow help as a skill when it reduces repeated manual guidance.

## Agent expectations

- Prefer updating the install/upgrade scripts over writing manual install steps.
- Keep README installation instructions aligned with the scripts.
- If any skill `SKILL.md` changes materially, ensure the generated OpenCode output still reflects the latest skill content.
- If a new skill directory is added at the repository root with `SKILL.md`, installation scripts should pick it up without requiring hardcoded lists.
- Verify installer changes by running them against an isolated temporary home before claiming completion.
- Do not introduce new dependencies for installation automation unless explicitly requested.

## Makefile / Tronador contract

- Tronador is required in this repository and must remain included from the Makefile exactly as provided.
- New Make targets may be added when needed, but the Tronador include must not be removed, duplicated, superseded, or replaced by a local reimplementation.
- Treat Tronador-provided behavior as the source of truth for shared automation; extend around it instead of overriding it.
- Direct Make commands are deprecated for README and Git-flow/version operations. Use `tronador readme build`, `tronador docs targets`, and `tronador versions ...` instead; retain the Makefile and unrelated repository-specific Make commands.
- After verifying the corresponding merge completed, destructive cleanup must use `tronador versions feature purge ... --allow-network`, `tronador versions hotfix purge ... --allow-network`, or `tronador versions release purge ... --allow-network`. Fail closed if the WayOfWork policy, merge evidence, authentication, or remote state is missing or contradictory; do not fall back to Make. For a GitFlow release merged into `support/*`, the release must also be back-integrated into `develop` before purge; otherwise Tronador intentionally refuses deletion. CI jobs must install the CLI first with `uses: cloudopsworks/install-tronador-cli@v1`.

## Verification requirements

After installer changes, verify at minimum:
- empty template repositories exit cleanly with no discovered skills
- auto-discovery finds the expected skills after a synthetic top-level skill is added
- Claude Code target creates `skills/<skill>/SKILL.md`
- Codex target creates `skills/<skill>/SKILL.md`
- OpenCode target creates `commands/<skill>/<skill>.md`
- upgrade scripts replace an existing installation cleanly
- generic scripts work for synthetic future skills added in a temporary verification copy

## Scope

These instructions apply to the repository root and all child paths.

## Release management

- Can use a release workflow skill for streamlined release processes, but follow this repository's concrete policy over generic template heuristics.
- Repositories generated from this template use the GitHubFlow-style GitVersion config in `.cloudopsworks/gitversion.yaml` (`main` / `release` / `feature` / `pull-request`, no `develop`).
- For generated repositories, branch release work from `main` using `feature/*`; do not default to `hotfix/*` or `fix/*` unless a repo-local rule explicitly requires it.
- In this template's GitVersion config, `+semver: breaking` maps to a **MINOR** bump. Use `+semver: major` for a true MAJOR release.
- Verify the canonical `# Agents: WayOfWork=githubflow` selector before branch operations. Missing, unsupported, or contradictory metadata is a blocker.
- Start work with `tronador versions feature start "<slug>"`. This command is WayOfWork-aware: GitFlow bases from `develop`; GitHubFlow and trunk-based flows use the configured primary branch.
- Generate `.cloudopsworks/_VERSION` only after `cw-release` verifies GitHub-template authority or the bounded canonical skills source-owner exception (trusted repository identity, known template status, regular non-symlink `.cloudopsworks/.skills`, supported selector, and corroborating source policy). When authorized, run `tronador project version --generate --yes`, then review, stage, commit, and submit the file through the normal PR flow. Generation changes only the file; it does not commit, tag, push, or publish. Marker presence or inherited prose alone never grants authority.
- If GitHub confirms `GH_IS_TEMPLATE=true`, this template's Actions are disabled; do not wait for checks that cannot register. In a generated implementation repository, wait for its configured required checks before merging.
- Use conventional commits for authored commits and merge the PR with a merge commit so GitVersion can read the merge body semver annotation.
- Never push directly to `main`, and do not squash-merge or rebase-merge release PRs.
- If GitHub confirms `GH_IS_TEMPLATE=true`, follow `cw-release` local publication after merge: use `tronador versions tag --publish` to push the tag, then create the GitHub Release with `gh` only if one does not already exist. A generated implementation repository may instead be CI-owned when its active workflows or repo-local policy say so; in that case observe CI and never fall back to local publication.
