# update-all

An agent skill that updates every installed coding agent harness in one run and reports versions before and after.

[Landing page](https://cskwork.github.io/update-all-skill/) | [Skill instructions](SKILL.md)

## Supported harnesses

| Harness | Binary | Update command |
|---|---|---|
| Codex | `codex` | `codex update` |
| Claude Code | `claude` | `claude update` |
| pi | `pi` | `pi update` (`pi update --all` only on explicit request) |
| Antigravity CLI | `agy` | `agy update` |

## Requirements

- An agent host that loads skills from `~/.agents/skills` (or a directory linked to it).
- At least one supported harness CLI on `PATH`. Missing harnesses are reported as `not installed` and skipped.

## Install

```bash
git clone https://github.com/cskwork/update-all-skill ~/.agents/skills/update-all
```

Optional, for Hermes. The destination `~/.hermes/skills/update-all` must not exist yet; create the parent directory first:

```bash
mkdir -p ~/.hermes/skills
ln -s ~/.agents/skills/update-all ~/.hermes/skills/update-all
```

## Usage

```text
/update-all
```

Natural requests also trigger it: "update all", "update harnesses", "update codex claude pi agy", "하네스 업데이트".

To also update pi extensions and packages, say so explicitly, for example `/update-all including pi extensions`. The skill then runs `pi update --all` instead of `pi update`.

## Workflow and verification

1. **Snapshot (before).** `command -v` finds each binary. Installed harnesses run `<cli> --version`. Missing ones are marked `not installed` and skipped.
2. **Update.** Runs `codex update`, `claude update`, `pi update`, `agy update` for installed harnesses. One failure never stops the others; exit code and stderr are kept exactly.
3. **Verify and report (after).** Reruns `<cli> --version` for every originally installed harness, even after a failed update, then prints one table plus exact stderr for each failure.

Example report. **Illustrative only: these are not current or latest versions.**

| Harness | Command | Before | After | Status |
|---|---|---|---|---|
| Codex | `codex update` | 0.46.0 | 0.48.0 | updated |
| Claude Code | `claude update` | 2.0.14 | 2.0.14 | unchanged |
| pi | `pi update` | 0.9.2 | 0.9.2 | failed (exit 1) |
| Antigravity CLI | — | — | — | not installed |

**pi, update, exit 1**

```text
Error: request to update server timed out after 30000 ms
```

Statuses:

- `updated` / `unchanged`: updater exited 0 and both version reads succeeded.
- `failed (exit N)`: the updater exited nonzero, or a version read failed. An updater failure stays `failed` even if the after version differs.
- `not installed`: the binary was not found; nothing was run.

## Safety

- Never installs a missing harness.
- Never elevates privileges (`sudo`) or retries with an alternative installer.
- Never invents a version; failed version reads are reported with phase, exit code, and stderr.
- No failed command is reported as a success.
- Commands run under your agent host's normal approval and sandbox policy.

## GitHub Pages

The landing page lives in [`docs/`](docs/) and is published at <https://cskwork.github.io/update-all-skill/>.

## License

[MIT](LICENSE)
