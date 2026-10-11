# Repository Guidance

## Purpose and boundaries
- Keep this dbt Core baseline reusable for a new warehouse project after rename.
- Leave warehouse-specific adapters, credentials, schemas, and business models to the instantiated repo.

## Development
- Preserve the staging, intermediate, and marts boundaries under `models/`.
- Keep project and profile rename instructions in `README.md` aligned with `dbt_project.yml`.

## Validation
- Run `uv run ruff check .`, `uv run ruff format --check .`, `uv run sqlfluff lint . --templater jinja`, and `uv run pre-commit run --all-files`.
- With a configured adapter and local profile, run `uv run dbt deps` and `uv run dbt parse`; live warehouse checks use `uv run dbt build`.

## Architecture constraints
- Sources and tests belong near the models they describe.
- Keep template defaults independent of a specific warehouse or business domain.

## Safety
- Commit examples only; keep real `profiles.yml` and credentials local.

## Branching strategy

- This repository uses **trunk-based development**. Create short-lived change branches from current `main` and target PRs to `main`.
- Follow the [canonical branching and delivery policy](https://github.com/SpencerRWood/codex-config/blob/main/docs/branching-policy.md), available through the installed codex-config source path. Verify the strategy, release association, fetched base and PR target before implementation and record them in Story evidence. Story completion does not authorize publication or deployment; retain existing review, CI, security and rollback requirements.
- This classification applies to maintenance of the template repository. Replace it with the new repository's classification during project creation; applications must use release-branch integration before Story development begins.
