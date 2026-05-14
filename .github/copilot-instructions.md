# Rules of Engagement

## AI Rules
- Never update this document; unless a human asks you to
- Never commit to the repo; unless a human asks you to
- Always run all tests (with coverage) and verify a clean zero exit code before making any commits or pushing code.
- All commits MUST be signed using SSH. Never bypass signing (e.g., never use `-c commit.gpgsign=false`). Ensure a key is set as the default for all commits (for example: `git config --global gpg.format ssh && git config --global user.signingkey ~/.ssh/id_ed25519.pub && git config --global commit.gpgsign true`).
- When bumping a service version, always treat it as a release workflow: update the metadata file (`pyproject.toml` or equivalent), run the package manager lock-sync step (`uv sync`, etc.), stage both the metadata and lock file, commit the changes, then tag and push.
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
- **6Cs** Complete, Correct, Concise, Courtesy, Clear, Considerate
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
  
