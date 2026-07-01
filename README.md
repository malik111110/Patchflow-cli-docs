# PatchFlow CLI Docs

Documentation website project for PatchFlow CLI.

## Local Development

```bash
npm install
npm run dev
```

The dev server serves the VitePress site from `docs/`.

## Build

```bash
npm run check
npm run build
```

`npm run build` regenerates the CLI command reference before producing the
static site in `docs/.vitepress/dist`.

## Source Sync

This docs repo contains curated product docs. It can also import source
reference documents from the CLI repo:

```bash
PATCHFLOW_CLI_REPO=/Users/digitalcenter/patchflow-cli npm run sync:cli
```

Command reference generation uses the local CLI binary:

```bash
PATCHFLOW_BIN=/Users/digitalcenter/patchflow-cli/patchflow npm run generate:commands
```

## Deploy

The included GitHub Actions workflow builds the site and uploads the static
artifact. Wire it to GitHub Pages, Netlify, Cloudflare Pages, or any static
host.
