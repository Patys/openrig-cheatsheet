# OpenRig Cheatsheet

[Open the cheatsheet](https://patys.github.io/openrig-cheatsheet/) for copyable workflows and a searchable command reference.

Targets **OpenRig 0.6.11**, release commit [`379f0a0d`](https://github.com/mvschwarz/openrig/tree/379f0a0d106e65164c615818927267c2082ebcb4). Commands were checked against the release source on October 19, 2026; provider integrations were not run. Use `rig <command> --help` to check your installed version.

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
| Restart Codex | Restart one seat on the same conversation, **Linux only**. |
| Reload YAML | Apply an edited team configuration. |
| Change model | Save a model selection for the next managed launch. |
| Upgrade | Install 0.6.11, restart the daemon and resume the team. |

`rig ask` searches retained history. `rig ask --wake` executes a one-shot agent and uses runtime quota; check its account of the past against current source documents. The video is a conceptual guide; command syntax remains checked against 0.6.11.

The command browser has 178 examples in 14 categories, with search, operation filters, copy buttons and source links. Commands are copied as text; the page does not execute them.

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

For a seat at `awaiting-decision`, read the recovery command before choosing fresh priming. A partial restore can leave other seats running successfully. `rig seat continue` only delivers pending startup context.

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

- [OpenRig 0.6.11 release](https://github.com/mvschwarz/openrig/releases/tag/v0.6.11)
- [CLI source at the release commit](https://github.com/mvschwarz/openrig/tree/379f0a0d106e65164c615818927267c2082ebcb4/packages/cli/src/commands)
- [Getting started](https://github.com/mvschwarz/openrig/blob/379f0a0d106e65164c615818927267c2082ebcb4/docs/reference/getting-started.md)
- [Rig specification](https://github.com/mvschwarz/openrig/blob/379f0a0d106e65164c615818927267c2082ebcb4/docs/reference/rig-spec.md)

The [Workshop listing](https://openrig.dev/rigs/workshop) supplies the current pinned team source and runtime choices. It declares broad access and non-interruptive launches; inspect the declared behavior before installing.
