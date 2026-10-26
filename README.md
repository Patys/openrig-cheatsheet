# OpenRig Cheatsheet

[Open the cheatsheet](https://patys.github.io/openrig-cheatsheet/) for copyable workflows and a searchable command reference.

Targets **OpenRig 0.7.2**, release commit [`8b180689`](https://github.com/mvschwarz/openrig/tree/8b180689e7004b5506fb9f76802e65572c6f6622). Commands were checked against the release source on October 26, 2026; provider integrations were not run. Use `rig <command> --help` to check your installed version.

## Changes since 0.7.1

- Added separate account homes, working per-seat account switching and live Codex ↔ Claude runtime handover.
- Added per-member Claude allow/deny rules and five workflows for accounts, runtime, rules and MCP handoff.
- Updated Codex restore/restart, Claude restore stages, fresh-session decisions and input-delivery guidance.
- Updated the pinned release, install commands, source links and managed core version to 0.1.19.
- Kept workflows before the command browser, all 14 categories and the existing persistent dark mode.

### Upgrade

Save active work and close the TUI before stopping the team. Preserve your instance environment and kernel choice.

```sh
rig down my-project
npm install -g @openrig/cli@0.7.2
rig --version
rig daemon stop
rig daemon start
rig up my-project --existing
rig restore-check --rig my-project --full --json
rig ps --nodes --rig my-project --full
```

The daemon restart is required: **six database updates from 0.7.1** apply on first start. If you normally use `--no-kernel`, keep it on `daemon start`. Older upgrades also cross the six migrations in 0.7.0 and eight in 0.7.1. Core **0.1.19** refreshes managed plugin/skill files, including `delegating-work` and `openrig-user`; deliberate edits must be reapplied from `$OPENRIG_HOME/backups/managed-refresh/`. Restarting the daemon alone does not reload surviving agent hooks. Reopen the TUI and pre-0.7.2 terminal views.

### Account homes and per-seat login

Sign in natively to separate **absolute directories on the daemon host**, then merge their paths into the matching members of your existing spec:

```sh
# Set these variables to your own absolute paths first.
CLAUDE_CONFIG_DIR="$CLAUDE_ACCOUNT_HOME" claude
CODEX_HOME="$CODEX_ACCOUNT_HOME" codex login
```

```yaml
# Fields inside existing pods[].members[]:
- id: owner
  runtime: claude-code
  env:
    CLAUDE_CONFIG_DIR: /absolute/path/to/claude-account
- id: backend
  runtime: codex
  env:
    CODEX_HOME: /absolute/path/to/codex-account
```

Member `env:` wins over rig `env:`. Relative homes are ignored and reported. **`rig up --env` no longer applies these two variables**; other settings still work. Apply YAML through the Reload YAML workflow, then read `rig seat status <seat> --json` and `rig provider status --json` for the effective home and source. Homes hold login and native history; changing a home does not copy the old conversation. Never put tokens in YAML.

For **Codex on Linux**, switch to an account profile already saved in that seat's home:

```sh
rig auth list --runtime codex --codex-home "$CODEX_ACCOUNT_HOME"
rig provider precheck --seat dev-backend@my-project --to-account "$ACCOUNT_NAME" --json
# After reviewing a safe precheck:
rig provider switch --seat dev-backend@my-project --to-account "$ACCOUNT_NAME" --json
# After a completed tool turn; ACCOUNT_LABEL is the native /status Account line:
rig seat restart dev-backend@my-project --keep-conversation --verify --expect-account "$ACCOUNT_LABEL" --json
```

`ACCOUNT_NAME` is a saved profile name, not a provider account reference. The switch saves the old login as `before-switch-<time>`, activates the target and restarts the same conversation. Other seats sharing that Codex home keep the old login until refresh, then need the printed `rig seat restart <peer> --keep-conversation --reason "..."`. This switch is unavailable on macOS.

For **Claude**, use `claude setup-token` to create a subscription token, save it in a file with mode `0600`, and set `CLAUDE_TOKEN_FILE` to its absolute path:

```sh
rig provider precheck --seat dev-owner@my-project --to-account "$ACCOUNT_LABEL" --token-file "$CLAUDE_TOKEN_FILE" --json
rig provider switch --seat dev-owner@my-project --to-account "$ACCOUNT_LABEL" --token-file "$CLAUDE_TOKEN_FILE" --json
# To return, precheck --stored-login first:
rig provider switch --seat dev-owner@my-project --stored-login --json
```

Claude snapshots and relaunches on the same conversation; save in-flight work first. OpenRig stores the token path and your label, not the token. Tools, hooks and MCP children inherit the process token, so do not print their environment. Token auth cannot use claude.ai connectors or Remote Control. A label is not a verified account email. Inherited `CLAUDE_CODE_OAUTH_TOKEN` can still outrank stored login; inspect the report.

A timeout or unknown result is not a reason to repeat a switch. Inspect seat status, the Codex restart plan's `lastRestart`, or the reported Claude snapshot recovery command. Ordinary usage refresh reads the daemon's logins, not every seat home. On-demand token usage costs one model turn:

```sh
rig provider status --refresh --provider claude --claude-token-file "$CLAUDE_TOKEN_FILE" --credential-label "$ACCOUNT_LABEL"
```

### Runtime and permissions

```sh
rig seat set-runtime dev-backend@my-project --runtime claude-code --reason 'Continue on Claude' --dry-run --json
rig seat set-runtime dev-backend@my-project --runtime claude-code --reason 'Continue on Claude' --json
rig seat status dev-backend@my-project --json
# Only for an interrupted switch:
rig seat set-runtime dev-backend@my-project --recover --json
```

Use `--runtime codex` for the reverse direction. Single-pane live Codex/Claude seats only. The successor starts and confirms its handover before the old runtime retires; address, edges, queue rows and claims remain. **A new native conversation starts**: history is not converted. Pane/scrollback change, source model/effort/Codex profile/native permission selection are cleared, and two processes run until cutover. Choose target settings with `--model`, `--effort`, `--codex-config-profile` or `--permission-mode`. The YAML is not rewritten. Claude agent-file launch additions and startup slash commands are not applied. Sends held as `runtime_switch_in_progress` must be resent after cutover. The staged successor uses `--confirm <switch-id> --note "<next step>"`; this is not an operator shortcut to skip reading the handover.

For Claude rules, use a JSON file or member YAML:

```json
{"allow":["Read(*)"],"deny":["Bash(git push:*)"]}
```

```sh
rig seat set-permissions dev-owner@my-project --rules "$CLAUDE_RULES_FILE" --reason 'Seat tool rules' --json
rig seat status dev-owner@my-project --json
```

```yaml
# Inside the existing Claude member:
claude_permissions:
  allow: ["Read(*)"]
  deny: ["Bash(git push:*)"]
```

Choose exactly one of `--mode` or `--rules`. Rules are audited and apply on the next managed launch/resume; the running process and shared project settings do not change. Empty lists clear seat rules. Allows do not withdraw broader inherited grants; denies are not guaranteed in `full_bypass`. These are native Claude tool rules, not OS or credential isolation. Native checks used Claude Code 2.1.309. Before downgrading to code before #1914, set equivalent native rules: older launch code omits stored seat denies. Codex's per-member counterpart is not in this release.

### Restore, resume and fresh conversations

- Codex default `workspace-write` retains commit access and topology writes (`LEARNED.md`) after resume/fork/restore; the daemon saves a recap refused by the sandbox.
- In-place `seat restart --keep-conversation` now reads the running Codex home and preserves writable roots with spaces. An unproven resume may still mean the pane restarted: read its result and native `/status` before retrying.
- **Ordinary Codex resume/restore can still misread workspace or queue-state paths with spaces.** The restart fix does not prove ordinary restore works.
- Codex sign-in reports `login_required`; model choosers can report `model_selection_gate`. Restored footer/model recognition improved. Resolve native prompts in the terminal.
- Claude managed restore grants reads of its own files at next launch/resume, including symlinked paths. Relative custom message paths and other restore actions may still ask. `rig compact <seat> --state` shows the owed stage and last delivery result; check its restore map before a manual request. Do not repeat compaction just because restoration is delayed.
- On macOS, verify Claude attention state in its terminal. Startup text may need Enter; a refusal can leave pasted input unsent. Inspect before resending.

To choose fresh native conversations while retaining durable work:

```sh
rig ps --nodes --rig my-project --full
# After saving active work; skip down if already stopped:
rig down my-project
rig up my-project --existing --fresh dev.owner dev.backend dev.check dev.frontend --yes
rig ps --nodes --rig my-project --full
```

Replace every logical `pod.member` ID with your own. Unlisted seats follow normal restoration. `up --existing --fresh` primes new conversations from durable context; `seat launch --fresh` creates a blank occupant. Neither is an account switch. `rig seat continue` only delivers pending startup context.

### MCP handoff

The MCP server exposes **36 tools**, adding `rig_queue_handoff` and `rig_queue_unclaim`. These are assistant tool calls, not shell commands:

```text
rig_whoami({})
rig_queue_handoff({"qitemId":"<held-row>","to":"<next-seat>","note":"Ready for review"})
rig_queue_show({"qitemId":"<held-row>"})
# Alternative for claimed work returned to its destination:
rig_queue_unclaim({"qitemId":"<claimed-row>","reason":"Return to destination queue"})
```

Handoff closes the held item and creates a successor linked by `chainOfRecord`; follow the original item with `rig_queue_show`. Do not manually close and recreate the work. On timeout, read the original before retrying. Unclaim returns an in-progress item to pending for its destination. Writes use `OPENRIG_SESSION_NAME` over stdio or the authenticated HTTP grant's seat.

### Breaking and operational caveats

- Linux launches inherit the daemon's `XDG_RUNTIME_DIR` with `DBUS_SESSION_BUS_ADDRESS=disabled:`. Declare a needed bus address in rig/member `env:`; this is hygiene, not isolation. Codex stdio MCP needs `env_vars = ["XDG_RUNTIME_DIR"]` in its `config.toml`.
- Mid-turn Codex sends join and may redirect the running turn; use `--wait-for-idle` for separate work. Real native dialogs still block sends. Claude long-paste issue #14 remains open; Pi recognition covers specific layouts.
- Custom core plugin files affect automatic Codex hook approval. Inspect managed-refresh backups and native hook review. A `hooks.json` home receives managed activity hooks there; OpenRig removes only its old inline block from `config.toml`.
- Recap writes through symlinks at `RECAP.md` or `recap-superseded/` now refuse with `recap_write_refused`.
- Cross-host queue timeout is `INDETERMINATE`; reconcile the original row ID. Retry only with the same `--id` and payload after checking; a failed read is not proof of absence.
- Slack posts now begin with `From <seat> on <machine>`; update parsers. Channel mapping still needs a daemon restart. Replies can escape their thread without `--reply-to`; the follow-up #2055 is outside 0.7.2.
- `haiku` with Claude plan mode can answer with Sonnet; use the full Haiku ID if required. Seat status includes `permissions.account.advisories`, even when empty.
- Runtime/account native coverage is partial; one tested runtime direction needed an extra Enter. No full home matrix, abrupt 0.7.2 power-loss recovery or universal performance improvement is claimed.
- The database conflict check needs `lsof`; a missing check is not a confirmed clean database. Large missions trees can still stall the daemon. Slow daemon start may take 20–40 seconds: start it before a TUI restore.

### Command delta: 0.7.1 → 0.7.2

This table covers release-noted new commands/options and changed behavior, including fixes without syntax changes. Unchanged commands used in the workflows keep their syntax.

| Command / interface | Change |
| --- | --- |
| `npm install -g @openrig/cli@0.7.2` | New pin; daemon stop/start required after install. |
| `npx skills add …/tree/v0.7.2/skills/rigs …` | Guided-setup source now pinned to 0.7.2. |
| `rig up --env` | Ignores `CLAUDE_CONFIG_DIR` / `CODEX_HOME`; use spec `env:`. |
| `rig provider precheck --seat … --to-account …` | Previews the seat restart and login saved in its effective home. Claude adds `--token-file` or `--stored-login`. |
| `rig provider switch --seat …` | Now executes. Codex uses a saved home profile (Linux); Claude supports `--to-account … --token-file …` or `--stored-login`. `--force-unsafe` cannot override seat checks. |
| `rig provider status` / `rig seat status` | Show effective home and source; status includes account advisories and runtime/restore state. |
| `rig provider status --refresh --provider claude` | Adds `--claude-token-file` and `--credential-label`; one model turn per token refresh. |
| `rig seat set-runtime` | New: `--runtime claude-code\|codex --reason`; `--dry-run`, `--recover`, target settings and successor `--confirm … --note …`. |
| `rig seat set-permissions` | Adds `--rules <file.json>`, exclusive with `--mode`. |
| `rig seat restart --keep-conversation` | Fixes effective-home proof and spaced writable roots; reports uncertain actual restart. Existing plan/verify syntax remains. |
| `rig up --existing` / `rig launch` / restore | Better Codex footer/readiness detection, login/model gates and restored sandbox access. `launch <rig> <seat>` can first-start one materialized seat. |
| `rig seat clear-attention <seat> --reason …` | Clears the latest hook prompt after a person declines a Claude permission. |
| `rig send` / queue wakes | Better Claude/Pi dialog/draft detection; Codex steer explanation; runtime-switch sends retained and require resending. One retry when approval attention clears. |
| `rig capture` | Warns below 80×24 and prints resize guidance. |
| `rig compact <seat> --state` | Shows owed restore stage and last delivery result; next-poll restore and managed-read fixes. |
| `rig_queue_handoff` / `rig_queue_unclaim` (MCP) | New tool calls; CLI queue handoff/unclaim syntax is unchanged. |
| `rig stream act <id>` | New: `--work <row>`, `--decline "<reason>"`, or `--duplicate <id>`; `stream show` displays actions. |
| `rig terminal open <view>` | Adds `--provider rex`; Rex supports `--window`. Reopen old views. |
| `rig context get reference/acp-agents.md` | Referenced ACP guide is now packaged. |
| `rig down` / `rig start` / `rig archive` | Dead tmux cleanup permits archive/up; terminal-only teams restore and name stopped terminals. |
| `rig bundle install` | Compose v2 precheck refuses with `compose_missing` before writes; `--plan` warns. |
| `rig export` | Copies culture file and names external agent references. |
| `rig queue resolve … --decision …` | Releases typed blockers (`external:`, `fold:`, `auth:`). |
| `rig queue create` / `--host` | Reads back uncertain creation once by ID; cross-host timeout is `INDETERMINATE` after 15 seconds. |
| `rig queue whoami` / `rig proof add` / `rig scope mission create` | Fix project-tagged lookup, cross-project slice lookup and persisted mission name. |
| `rig queue update --help` / `rig skill audit` / `rig acp run --help` | Clearer cancellation (`--state done --closure-reason canceled`), audit findings and shipped-guide help; no new cancellation syntax. |

## Everyday workflows

Set the rig name, target seat, YAML path and model under **Customize examples**, then choose a workflow. The Start here guide explains seats, leads, context and when to use `send`, `queue`, `ask` or `context`. It adapts the practical ideas from [Mike Schwarz’s OpenRig video](https://www.youtube.com/watch?v=CYRJDTcTKhQ).

Available workflows:

| Workflow | Use it to |
| --- | --- |
| Guided setup | Install the rigs skill and let your current agent guide setup. |
| First launch | Validate and launch a team from YAML. |
| Install a bundle | Inspect and install Workshop or another published team. |
| Project context | Orient the agent in existing project systems before implementation. |
| Assign a task | Create durable work and follow its owner through review. |
| Ask an expert | Search past decisions or explicitly wake retained agent context. |
| Share a team | Package the spec and declared resources for reuse. |
| After a limit | Inspect quota and continue the existing task after reset. |
| Resume work | Restore a saved team and inspect each seat. |
| Fresh sessions | Start new conversations after saving active work. |
| Restore decision | Resolve `awaiting-decision` or an unavailable conversation. |
| Restart Codex | Restart one seat on the same conversation, inspect the restart plan first. |
| Reload YAML | Apply an edited team configuration. |
| Change model | Save a model selection for the next managed launch. |
| Repair pane | Repair a missing managed pane binding after upgrading. |
| Tune effort | Save and inspect per-seat reasoning effort. |
| Move seat | Change a working folder and adopt it with a rebuild handover. |
| Compact Claude | Run managed compaction and verify the result. |
| Control input | Choose automatic input levels and peer intervention policy. |
| Quiet hours | Hold routine asks overnight and tune Slack delivery. |
| Seat skills | Inspect and apply skills outside the project. |
| Live edges | Preview and change a running team’s relationships. |
| Upgrade | Install 0.7.2, restart the daemon and resume the team. |
| Seat accounts | Sign in to separate homes and declare member env. |
| Switch account | Precheck and switch one seat to a different login. |
| Switch runtime | Handover between Codex and Claude Code. |
| Claude rules | Save per-seat allow/deny tool rules. |
| MCP handoff | Pass claimed work on or return it to pending. |

`rig ask` searches retained history. `rig ask --wake` executes a one-shot agent and uses runtime quota; check its account of the past against current source documents. The video is a conceptual guide; command syntax remains checked against 0.7.2.

There are 28 workflows. The command browser has 231 examples in 14 categories, with search, operation filters, copy buttons and source links. Commands are copied as text; the page does not execute them.

## Quick commands

Replace `my-project` and `dev-owner` with names from your own team.

```sh
rig --version
rig daemon status
rig ps --nodes --rig my-project --full
rig queue list -A --destination dev-owner@my-project
rig capture dev-owner@my-project --lines 100
rig terminal open my-project
```

`rig up my-project --existing` uses saved configuration. Launch from the YAML file to apply edits. `--fresh` starts new native conversations and takes logical seat IDs such as `dev.owner`, rather than session addresses such as `dev-owner@my-project`.

`rig down` asks before stopping the rig you are inside; noninteractive use must confirm with `--yes`. Start the daemon before `rig up --plan`. `rig ps --nodes` adds an ATTN column, so prefer JSON for scripts.

For a seat at `awaiting-decision`, read the recovery command before choosing fresh priming. A partial restore can leave other seats running successfully. `rig seat continue` only delivers pending startup context.

Codex in-place restart now uses the running home; an uncertain resume can still follow a real pane restart. Inspect the result before retrying. Ordinary restore with spaces remains limited. For Claude, inspect pending restore state and native prompts before repeating a request.

## Run locally

Open `index.html` directly, or serve the repository:

```sh
python3 -m http.server 8000
```

Open [localhost:8000](http://localhost:8000). The single HTML file has no build step or external dependencies and works offline; source links need a connection.

## Maintain and publish

Edit `index.html`: `command-data` contains categories and commands; `workflows` contains the step-by-step examples. Check changed commands against the tagged release source, then update the version, commit, date and example counts together.

GitHub Pages serves `main` from `/ (root)`. Keep `.nojekyll`; pushes publish automatically when Pages is enabled.

## Upstream references

- [OpenRig 0.7.2 release](https://github.com/mvschwarz/openrig/releases/tag/v0.7.2)
- [CLI source at the release commit](https://github.com/mvschwarz/openrig/tree/8b180689e7004b5506fb9f76802e65572c6f6622/packages/cli/src/commands)
- [Human quickstart](https://github.com/mvschwarz/openrig/blob/8b180689e7004b5506fb9f76802e65572c6f6622/docs/reference/quickstart.md)
- [Getting started](https://github.com/mvschwarz/openrig/blob/8b180689e7004b5506fb9f76802e65572c6f6622/docs/reference/getting-started.md)
- [Rig specification](https://github.com/mvschwarz/openrig/blob/8b180689e7004b5506fb9f76802e65572c6f6622/docs/reference/rig-spec.md)

The [Workshop listing](https://openrig.dev/rigs/workshop) supplies the current pinned team source and runtime choices. It declares broad access and non-interruptive launches; inspect the declared behavior before installing.
