# Rules of Engagement

## AI Rules

- Never update this document unless a human asks you to.
- Never ship (commit, tag, or push) unless asked to do so.
- Run all tests (with coverage) and verify a clean zero exit code and the repo's coverage threshold (90% unless CI or `pyproject.toml` states otherwise).
- All commits MUST be signed. Never bypass signing!
- When bumping a service version, treat it as a release workflow: update the metadata file (`pyproject.toml` or equivalent), run the package manager lock-sync step (`uv sync`, etc.), stage both the metadata and lock file, commit, then tag and push when asked. When the release has user-visible or operational impact, also update `CHANGELOG.md` (what changed) and `INSTALLATION_PLAN.md` (operator steps and verification). Do not put changelog prose in `INSTALLATION_PLAN.md` or operator steps in `CHANGELOG.md`.
- Always use `pnpm` for JavaScript/TypeScript package management and `uv` for Python dependency lock-sync and package management.
- Never use local artifacts for publishing or dependency resolution. Only use published CI-built resources for wheels, container images, crates, packages, and similar artifacts.
- Never manually upload build artifacts; let CI pipelines handle the build.
- Never reference /memories/, tool names, or internal agent state in any file committed to the repo.
- Never change `.env` files — do not write, copy, create, or modify any `.env` unless a human explicitly asks you to.
- Follow the established tagging convention: `<service>/vX.Y.Z` (e.g. `authentication/v1.2.3`).

## Code Style

- Python: follow PEP 8; all imports at the top grouped by standard library, third-party, and local.
- Use `uuidv7` for all UUID generation.
- Environment variables only for secrets and service configuration with "safe" defaults.
- No PHI/PII/SPII in logs, metrics, or error messages; exceptions must not expose stack traces.

## Testing Rules

- Integration tests verify the contract surface (api/ui) and exercise the stack down to external dependencies (db/api).
- Integration tests should be happy-path only. Save negative testing for unit tests.
- Unit tests cover sad-path behavior with isolated, in-function patching for external library and system calls.
- Do not defeat integration tests by patching the service layer under test; stub only real external dependencies such as DB session creation, HTTP calls, or external SDKs.
- Browser UI: happy-path e2e only (Playwright); no unit or integration tests.

## Design, Implementation & Documentation

Always observe these principles:

- **DRY** — one authoritative source for every fact; no copy-paste logic or duplicated config.
- **Separation of Concerns (SoC)** — no tight coupling in documentation, services, and code.
- **Commit lockfiles** — `uv.lock` and equivalent lockfiles MUST be committed.
- **6Cs** — documentation and code should be Complete, Correct, Concise, Clear, Courtesy, Considerate.
- **YAGNI/KISS**
- **Explicit over implicit** — name things precisely; avoid magic numbers, strings, or shorthand.
- **No denormalization** — Each fact has one authoritative owner. Do not copy fields from another service or entity onto local rows or API payloads for convenience. Store foreign keys and resolve at read time (call the owner, or use data the caller already has).

# All the Things Place

AKA **APFE&EIIP** — a place for everything and everything in its place.

**Scope:** Platform-wide paths apply to product and reference repos (e.g. CDP). Per-service paths apply to every deployable service repo (authentication, user, capabilities, etc.). Shared contract artifacts live in the [`schemas`](https://github.com/Neosofia/schemas) repo.

## TL;DR

**Constitution** = enduring must-haves. **ADR** = why we chose a structure. **Spec** = what a component must do. **`openapi.json`** = exact API contract. **Per-repo ops / security / release / install** = run, protect, announce, and deploy.

## Platform-wide (product repos, e.g. CDP)

| Location | Purpose |
|----------|---------|
| `specs/` | Product specifications (what each component must do) |
| `architecture/constitution.md` | Constitution — stable, non-negotiable principles every other artifact must obey or explicitly amend |
| `architecture/adrs/` | ADRs — durable record of a significant architectural choice (context, decision, consequences) |
| `architecture/structurizr/` | C4 diagrams and related architecture views |

## Per service repo

| Location | Purpose |
|----------|---------|
| `README.md` | Product overview and pointers to other docs in the repo |
| `OPERATIONS.md` | How the product is operated locally and in the cloud |
| `SECURITY.md` | Security posture, controls, and reviewer guidance |
| `openapi.json` | Machine-readable API contract (repo root; not served over HTTP) |
| `CHANGELOG.md` | Human-readable changelog per [Keep a Changelog](https://keepachangelog.com/) (what changed for users of the product; not operator steps) |
| `INSTALLATION_PLAN.md` | Product Installation Plan (PIP): per-version deploy steps, configuration changes, post-deploy verification, and evidence (not a changelog) |

Data model changes belong in Alembic migrations in the service repo (see `OPERATIONS.md`).

**Documentation gold standards** (changelog, installation plan, version identifiers, service spec format): [Documentation gold standards](https://neosofia.tech/resources/guides/documentation/) in the corporate resources guides.
