# Centralize the documentation toolchain in mise

## Status

Accepted

## Context

The MkDocs site depends on Python, uv, and the packages listed in `requirements.txt`. A virtualenv created from a machine-local Python install does not travel with the repository, so two clones can build with different interpreters and different package sets.

## Decision

`mise.toml` owns the tool pins and the task names. The task graph follows the fmind mise skill and the documentation-site task file, with MkDocs kept as the site generator.

- Python is pinned to 3.14.8, uv to 0.13.1, and dprint to 0.63.0. `mise.lock` records the download URL and checksum for each platform. Install those tools with `mise install --locked`.
- `min_version` is 2026.9.15, the mise release that verifies this repository. The python-stack template asks for 2026.10.11, and lockfile revision 3 needs 2026.9.16 or newer. This environment is 2026.9.15 and writes lockfile revision 2, so the template floor is held back.
- dprint 0.63.0 is the newest release this mise install will select. One newer dprint release is hidden by `minimum_release_age`.
- mise creates and activates `.venv` from the pinned Python with `_.python.venv`. `mise run install` syncs `requirements.lock` into that virtualenv. `UV_PYTHON` is unset, because a path there makes `uv pip sync` install into the mise Python prefix.
- `mise run format` formats Markdown, JSON, TOML, and YAML with dprint. `mise run check` runs `check:format` and `build:docs` together. `build:docs` runs `mkdocs build --strict`. `mise run serve:docs` previews the site with MkDocs live reload. `mise run all` formats, then checks. `mise run lock:requirements` regenerates `requirements.lock`.
- There is no test suite, so this repository does not define `test`. There is no Git hook runner, so `install` does not call `lefthook`.

## Consequences

A clone with mise 2026.9.15 or newer installs the same interpreter, the same formatter, and the same documentation packages without a preinstalled system Python. `.venv` and the generated `site/` directory stay untracked. This ADR is excluded from the published documentation site.
