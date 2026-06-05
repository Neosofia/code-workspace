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

## Design, Implementation & Documentation

Always observe these principles:

- **DRY** — one authoritative source for every fact; no copy-paste logic or duplicated config.
- **Separation of Concerns (SoC)** — no tight coupling in documentation, services, and code.
- **Commit lockfiles** — `uv.lock` and equivalent lockfiles MUST be committed.
- **6Cs** — documentation and code should be Complete, Correct, Concise, Clear, Courtesy, Considerate.
- **YAGNI/KISS**
- **Explicit over implicit** — name things precisely; avoid magic numbers, strings, or shorthand.

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

### Changelog (`CHANGELOG.md`)

Write for **people who use or administer the product** (clinicians, operators, integrators reading outcomes). Follow [Keep a Changelog](https://keepachangelog.com/en/1.1.0/): file name `CHANGELOG.md`, title `# Changelog`, sections **Added**, **Changed**, **Fixed**, **Removed**.

- Describe **what they can do or notice** (screens, workflows, behavior)
- Name UI areas the way users see them (**Admin → Users**).
- Put version pins, migrations, secrets, and smoke tests in `INSTALLATION_PLAN.md`, not here.

### Version identifiers

- **CDP UI** uses **CalVer** `YYYY.MM.DD` (release day) in `ui/src/lib/uiVersion.ts` (`UI_RELEASE_VERSION`), shown in the app footer.
- **Backend services** use **semver** from `pyproject.toml`, exposed on `GET /health` as `"version": "<semver>"` alongside `"status"` (orchestrator probes that only check HTTP status remain valid).

### Spec documents (`specs/`)

A baseline spec should always be used to avoid duplication in other specs. For example:
[000-platform-baseline.md](https://github.com/Neosofia/cdp/blob/main/specs/000-platform-baseline.md).

- **Story first** -- Open with why the service exists, how it fits the platform (one or two paragraphs), then client objectives, then functional and operational requirements. No metadata, implementation details, or duplicated content that belongs in other places.
- **Objectives in client language** -- State what operators, end users, and deploying products need to accomplish (intent and outcomes), not a laundry list of API endpoints or UI screens.
- **Requirements with context** -- Each FR/OR states the obligation and, where helpful, a short sentence on why (readable roles, auditability, measurability). Omit RFC-style MUST; requirements are binding by default. Operational requirements focus on what operators must be able to measure or deploy, not SLO numbers or logging implementation detail.
- **DRY and external pointers** -- Universal transport, client platform support, telemetry, contracts, and accessibility live in spec 000; API shape lives in `openapi.json`; security and authorization depth in `SECURITY.md`; aggregation and SLIs in the operational-metrics spec (011). Further reading links only to canonical GitHub URLs in the owning repos, not relative paths across the workspace. Use `--` (double hyphen), not em dashes, in spec prose.
