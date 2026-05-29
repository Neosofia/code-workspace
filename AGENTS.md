# Rules of Engagement

## AI Rules
- Never update this document; unless a human asks you to
- Never commit to the repo; unless a human asks you to
- Always run all tests (with coverage) and verify a clean zero exit code before making any commits or pushing code.
- All commits MUST be signed using SSH. Never bypass signing (e.g., never use `-c commit.gpgsign=false`). Ensure a key is set as the default for all commits (for example: `git config --global gpg.format ssh && git config --global user.signingkey ~/.ssh/id_ed25519.pub && git config --global commit.gpgsign true`).
- When bumping a service version, always treat it as a release workflow: update the metadata file (`pyproject.toml` or equivalent), run the package manager lock-sync step (`uv sync`, etc.), stage both the metadata and lock file, commit the changes, then tag and push.
- Always use `pnpm` for JavaScript/TypeScript package management and `uv` for Python dependency lock-sync and package management.
- Never use local artifacts for publishing or dependency resolution. Only use published CI-built resources for wheels, container images, crates, packages, and similar artifacts.
- Never reference /memories/, tool names, or internal agent state in any file committed to the repo
- **Never change `.env` files** — do not write, copy, create, or modify any `.env`, unless a human explicitly asks you to
- follow tagging convention: `<service>/vX.Y.Z` (e.g. `authentication/v1.2.3`)

## Code Style (Humans and AI)

- Python: follow PEP 8; all imports at the top grouped by standard library, third-party, and local.
- Use `uuidv7()` for all UUID generation in PostgreSQL (do not use `gen_random_uuid()` or `uuid_generate_v*()`).
- Environment variables only for secrets and service configuration with "safe" defaults
- No PHI/PII/SPII in logs, metrics, or error messages 

## Testing

- Integration tests verify the contract surface (api/ui) and exercise the stack down to external dependencies (db/api).
- Integration tests should be happy-path only. Save negative testing for unit tests
- Unit tests cover sad-path behavior with isolated, in-function patching for external library and system calls.
- Do not defeat integration tests by patching the service layer under test; stub only real external dependencies such as DB session creation, HTTP calls, or external SDKs.

## Design, Implementation & Documentation

- **DRY** — one authoritative source for every fact; no copy-paste logic or duplicated config
- **No tight coupling** — this applies to procedures in documentation, services, and code etc.
- **Commit lockfiles** — `uv.lock` and equivalent lockfiles MUST be committed; they pin dependency hashes and are the first line of defence against supply chain attacks
- **6Cs** Complete, Correct, Concise,  Clear, Courtesy, Considerate
- **YAGNI**
- **Explicit over implicit** — name things precisely; avoid magic numbers, strings, or behavior
- **APFE&EIIP** A place for everything and everything in its place
  - Per-service API contract -> `services/<service>/openapi.json` (ADR-0008)
  - Shared/cross-service schemas -> `schemas/` (e.g. `schemas/log.json`)
  - Data model -> Alembic migration in services
  - Research -> ADRs in architecture/adrs
  - Implementation -> code in services
  - Principles - Constitution in architecture/constitution.md
  - Architecture - C4 diagrams in architecture/structurizr
  - Governance -> ADRs, Architecture, Constitution

## Service specifications (gold standard)

Cross-cutting transport, client platform support, logging, API contracts, and accessibility live in [000-platform-baseline.md](https://github.com/Neosofia/cdp/blob/main/specs/000-platform-baseline.md); feature specs **001–018** inherit it and state only component-specific additions.

Specs **000–018** in `cdp/specs/` follow this format (exemplar: [018-user-service.md](https://github.com/Neosofia/cdp/blob/main/specs/018-user-service.md)). When writing or rewriting a service spec:

- **Story first** -- Open with why the service exists, how it fits the platform (one or two paragraphs), then client objectives, then functional and operational requirements. No metadata, implementation details, or duplicated content that belongs in another service's spec or repo.
- **Objectives in client language** -- State what operators, end users, and deploying products need to accomplish (intent and outcomes), not a laundry list of API endpoints or UI screens.
- **Requirements with context** -- Each FR/OR states the obligation and, where helpful, a short sentence on why (readable roles, auditability, measurability). Omit RFC-style MUST; requirements are binding by default. Operational requirements focus on what operators must be able to measure or deploy, not SLO numbers or logging implementation detail.
- **DRY and external pointers** -- Universal transport, client platform support, telemetry, contracts, and accessibility live in spec 000; API shape lives in `openapi.json`; security and authorization depth in `SECURITY.md`; aggregation and SLIs in the operational-metrics spec (011). Further reading links only to canonical GitHub URLs in the owning repos, not relative paths across the workspace. Use `--` (double hyphen), not em dashes, in spec prose.
