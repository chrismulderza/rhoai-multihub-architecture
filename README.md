# RHOAI Multi-Cluster MaaS Architecture

Documentation site for the multi-datacenter Red Hat OpenShift AI Models-as-a-Service platform.

## Prerequisites

Install [mise](https://mise.jdx.dev/getting-started.html) 2026.9.15 or newer.

## Setup

```bash
mise install --locked
mise run install
```

## Build and serve locally

Complete [Setup](#setup) first. `mise run install` syncs the MkDocs packages into `.venv`. Both commands below run that step when the virtualenv is out of date.

Build the static site:

```bash
mise run build
```

The build writes `site/` and exits non-zero when a local link or markdown file is invalid. `site/` stays untracked.

Preview the guide with live reload:

```bash
mise run serve:docs
```

Open `http://127.0.0.1:8000`. Saving a page under `docs/` or `mkdocs.yml` reloads the browser. Press Ctrl-C to stop the server. The preview uses the same strict checks as the build, so a broken local link fails at startup.

## Tasks

```bash
mise run format
mise run check
mise run all
```

`mise run check` checks formatting and builds the site. `mise run all` formats sources, then runs those checks.

## Dependency updates

Edit `requirements.txt`, then regenerate and commit the lockfile:

```bash
mise run lock:requirements
```

## GitHub Pages

Pushes to `main` run [`.github/workflows/deploy-docs.yml`](.github/workflows/deploy-docs.yml). The workflow can also be started manually. It builds the site inside `registry.access.redhat.com/ubi9/python-314` and publishes the `gh-pages` branch.

After the first successful run:

1. Open **Settings > Pages**.
2. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
3. Set **Branch** to `gh-pages` and the folder to `/ (root)`.

The published site is `https://<owner>.github.io/<repository>/`.
