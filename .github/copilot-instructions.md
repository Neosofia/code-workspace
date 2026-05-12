# Rules of Engagement

## AI Rules
- Never update this document; unless a human asks you to
- Never commit to the repo; unless a human asks you to
- All commits MUST be signed using SSH. Never bypass signing (e.g., never use `-c commit.gpgsign=false`). Ensure a key is set as the default for all commits (for example: `git config --global gpg.format ssh && git config --global user.signingkey ~/.ssh/id_ed25519.pub && git config --global commit.gpgsign true`).
- When bumping a service version, always treat it as a release workflow: update the metadata file (`pyproject.toml` or equivalent), run the package manager lock-sync step (`uv sync`, etc.), stage both the metadata and lock file, commit the changes, then tag and push.
- Never reference /memories/, tool names, or internal agent state in any file committed to the repo
- **Never change `.env` files** — do not write, copy, create, or modify any `.env`, unless a human explicitly asks you to
- When using spec kit
  - don't put contracts in the spec folder; each service owns its API contract at `services/<service>/openapi.json` cross-service/shared schemas (e.g. `log.json`) are here: https://github.com/Neosofia/schemas
  - Exclude all technical details from specs unless they are essential to understanding user needs and behavior, or when using the tech as an example/possible solution.
  - after implementation is complete, run a **distillation pass**:
    1. identify every transient spec-kit artifact:
       - `research.md`, `plan.md`, `tasks.md`
       - `checklists/`
       - `contracts/` drafts
       - `data-model.md`
       - `quickstart.md`
    2. decide for each artifact whether to:
       - **memorialize** — promote durable decisions to their canonical home
         - ADR for technical decisions
         - Constitution for principles
         - `SECURITY.md` / `README.md` for operational facts
         - code comments or migrations for data model
         - `openapi.json` for contracts
       - **let go** — delete the file; spec-kit scaffolding is not a deliverable
    3. keep the `spec.md` as the human-readable feature record, but update it to link forward to memorialized artifacts, not backward to deleted scaffolding
    4. a feature is not "done" until the distillation pass leaves the repo DRY
    5. trigger the pass with `distill NNN` (e.g. `distill 014`)

## Code Style (Humans and AI)

- Python: follow PEP 8; all imports at the top grouped by standard library, third-party, and local.
- Use `uuidv7()` for all UUID generation in PostgreSQL (do not use `gen_random_uuid()` or `uuid_generate_v*()`).
- Environment variables only for secrets and service configuration with "safe" defaults
- No PHI/PII/SPII in logs, metrics, or error messages 

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
  - Research -> ADRs in architecture/structurizr/decisions
  - Implementation -> code in services
  - Principles - Constitution in architecture/constitution.md
  - Architecture - C4 diagrams in architecture/structurizr
  - Governance -> ADRs, Architecture, Constitution
  
