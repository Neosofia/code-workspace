# Operations Guide — Local Development

Cross-repo notes for engineers working in the Neosofia multi-repo workspace. Service-specific runbooks (WorkOS, Docker, cloud deploy) stay in each repository’s own `OPERATIONS.md` / `OPS-*.md`.

## Python: `uv sync` is the source of truth

Use [uv](https://docs.astral.sh/uv/) for all Python dependency install and lockfile maintenance. **Do not** add editor-only workarounds (`python.analysis.extraPaths`, `cursorpyright.analysis.extraPaths`, `pyrightconfig.json` `extraPaths`, or similar) to make imports resolve — if the IDE cannot find a module, run `uv sync` in that repo and point the interpreter at the repo `.venv`.

Workspace `.vscode/settings.json` files should only set:

```json
"python.defaultInterpreterPath": "${workspaceFolder}/.venv/bin/python"
```

(or the service path below). That tells Cursor/VS Code which interpreter to use after sync; it is not a substitute for installing packages.

### After clone or pulling dependency changes

```bash
uv sync
```

Commit `uv.lock` whenever `pyproject.toml` dependencies change. For reproducible installs that match CI (hash-pinned), use:

```bash
uv sync --frozen
```

### By repository layout

| Layout | Where to run `uv sync` | Interpreter (typical) |
|--------|------------------------|------------------------|
| **SDK workspace** (`sdk/`) | Repository root | `sdk/.venv/bin/python` |
| **Python service** (`authentication/`, `capabilities/`, `notification/`, …) | Service repository root | `<service>/.venv/bin/python` |
| **Templates** | `templates/python/service/` | `templates/python/service/.venv/bin/python` |
| **Schemas** (tooling only) | `schemas/` repository root | `schemas/.venv/bin/python` |

**SDK workspace:** The root `pyproject.toml` declares a `dev` dependency group on all workspace members (`logenvelope`, `authentication-in-the-middle`, `authorization-in-the-middle`). A single `uv sync` at the repo root installs every package into `.venv` for local development and editor resolution.

```bash
cd sdk
uv sync
uv run --package logenvelope pytest -q   # example: test one package
```

**Python services:** Each service is its own installable project. `uv sync` installs the service and its dependencies (including published SDK wheels per `[tool.uv.sources]`).

**Templates:** Treat `python/service` as the project root (same pattern as a production service).

### When imports still fail in the editor

1. Confirm sync succeeded: `.venv` exists and `uv run python -c "import <package>"` works.
2. Reload the window or re-select the interpreter — it must be that repo’s `.venv`, not a system Python.
3. If `pyproject.toml` changed, run `uv lock` (when needed), `uv sync`, and commit the updated `uv.lock`.

Do not add `extraPaths` (in VS Code settings or `pyrightconfig.json`) to paper over a missing venv. Do not use `PYTHONPATH=.` for local `uv run` — the editable install from `uv sync` already puts the service root on the import path.

### Releases and CI

- Local day-to-day: `uv sync`
- CI / before merge when verifying lock integrity: `uv sync --frozen`
- Bumping a published SDK package: follow the release workflow in [AGENTS.md](AGENTS.md) (version, lock, commit, tag, push)
