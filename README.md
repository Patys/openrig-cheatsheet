# OpenRig Cheatsheet

A minimal, interactive command reference for [OpenRig](https://github.com/mvschwarz/openrig).

**Website:** https://patys.github.io/openrig-cheatsheet/

## What's included

- 150 command examples in 14 categories.
- Search and filters for read, change, stop, delete and agent-terminal commands.
- Copy buttons and editable examples for rig names, seats, spec paths and models.
- Eight workflows: first launch, assign a task, continue after a limit, resume work, recover a missing session, reload YAML, change a model and upgrade.
- Workflows above a scrollable command browser; light/dark mode with a remembered preference.
- Links to the upstream documentation and command source.

The page is written in English and uses generic project examples. It is a single HTML file with inline CSS and JavaScript, with no build step or external dependencies.

## Use locally

Download `index.html` and open it in a browser, or serve the repository:

```sh
python3 -m http.server 8000
```

Then open http://localhost:8000.

The command reference works offline. Source links require an internet connection. Commands are copied as text; the page does not execute them.

## GitHub Pages

Publish the repository root from the `main` branch:

1. Open **Settings → Pages**.
2. Select **Deploy from a branch**.
3. Choose **main** and **/ (root)**, then save.

The `.nojekyll` file allows GitHub Pages to serve the page without Jekyll processing. After Pages is enabled, updates to `main` publish automatically.

GitHub Pages from a private repository requires a supported paid GitHub plan. The site can be public while the source repository stays private.

## Reference version

Verified against **OpenRig 0.6.10**, commit `3001cc436656c435e54425443e708e3e2d8c7d68`, on October 14, 2026.

This is an unofficial reference. CLI commands and flags may change. Check `rig --help` and each command's help against your installed version.

Examples that use IDs or file paths require real values from your environment. Model availability depends on your runtime and account. Read each command's notes before running it.

## Changes since the previous reference

Updated from 0.6.5 to the latest published release, **0.6.10** ([release notes](https://github.com/mvschwarz/openrig/releases/tag/v0.6.10)), including relevant changes from 0.6.6–0.6.9:

- Current starter/factory names, providers and seat roles.
- Provider-selective setup, default Herdr installation and optional Ghostty.
- Automatic pre-down snapshots and guidance restoration on resume.
- Asynchronous queue creation, bounded `--verify` receipts and compact JSON fields.
- Exact `capture --lines` tails and the `--history-plus-pane` compatibility option.
- Partial restore diagnostics, startup continuation and explicit fresh priming.
- Provider limit/account inspection, guarded switching and Slack delivery diagnostics.
- Team/kernel permission defaults and the CLI/daemon upgrade sequence.

The examples were checked against the release's CLI source and relevant release notes. This is not an authenticated runtime or provider integration test. Some release limitations remain: `doctor` checks both harnesses, `tui --shared` uses the default tmux socket, and an indeterminate queue verification receipt can coexist with a recorded claim.

## Update the reference

Edit `index.html`:

- Command entries and categories are in the `command-data` JSON block.
- Workflow examples are in the `workflows` array.
- Styles and application logic are embedded in the same file.

Check changed commands against the upstream CLI source and update the version, source links and verification date together.

## Upstream resources

- [OpenRig 0.6.10 release](https://github.com/mvschwarz/openrig/releases/tag/v0.6.10)
- [Registered CLI reference](https://github.com/mvschwarz/openrig/blob/3001cc436656c435e54425443e708e3e2d8c7d68/docs/as-built/cli-reference.md)

- [Getting started](https://github.com/mvschwarz/openrig/blob/3001cc436656c435e54425443e708e3e2d8c7d68/docs/reference/getting-started.md)
- [Rig specification](https://github.com/mvschwarz/openrig/blob/3001cc436656c435e54425443e708e3e2d8c7d68/docs/reference/rig-spec.md)
- [CLI command source](https://github.com/mvschwarz/openrig/tree/3001cc436656c435e54425443e708e3e2d8c7d68/packages/cli/src/commands)
- [GitHub Pages setup](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
