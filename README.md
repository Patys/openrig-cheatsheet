# OpenRig Cheatsheet

A minimal, interactive command reference for [OpenRig](https://github.com/mvschwarz/openrig).

**Website:** https://patys.github.io/openrig-cheatsheet/

## What's included

- 119 command examples in 14 categories.
- Search and filters for read, change, stop, delete and agent-terminal commands.
- Copy buttons and editable examples for rig names, seats, spec paths and models.
- Five workflows: first launch, assign a task, resume work, reload YAML and change a model.
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

Verified against **OpenRig 0.6.5**, commit `5ea35e93`, on October 6, 2026.

This is an unofficial reference. CLI commands and flags may change. Check `rig --help` and each command's help against your installed version.

Examples that use IDs or file paths require real values from your environment. Model availability depends on your runtime and account. Read each command's notes before running it.

## Update the reference

Edit `index.html`:

- Command entries and categories are in the `command-data` JSON block.
- Workflow examples are in the `workflows` array.
- Styles and application logic are embedded in the same file.

Check changed commands against the upstream CLI source and update the version, source links and verification date together.

## Upstream resources

- [Getting started](https://github.com/mvschwarz/openrig/blob/5ea35e93/docs/reference/getting-started.md)
- [Rig specification](https://github.com/mvschwarz/openrig/blob/5ea35e93/docs/reference/rig-spec.md)
- [CLI command source](https://github.com/mvschwarz/openrig/tree/5ea35e93/packages/cli/src/commands)
- [GitHub Pages setup](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
