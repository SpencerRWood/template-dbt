# template-dbt

A reusable dbt Core project template with `uv`, SQLFluff, Ruff, pre-commit,
GitHub Actions, and semantic-release wired together for analytics
transformation repositories.

## Creating A Project

After copying this template into a new repository:

1. Rename the project in `pyproject.toml` and `dbt_project.yml`.
2. Update `dbt_project.yml` so `name`, `profile`, and the top-level
   `models:` key use the new dbt project name.
3. Install the warehouse adapter for the concrete project.
4. Copy `profiles.example.yml` to your local dbt profiles directory or to an
   uncommitted `profiles.yml`.
5. Configure the required `DBT_*` environment variables.
6. Run `uv sync --frozen --group dev`.
7. Run `uv run dbt debug` after the adapter and profile are configured.
8. Begin defining sources and staging models.

Adapter dependencies belong to the instantiated project, not this generic
template. Common choices include:

```sh
uv add dbt-postgres
uv add dbt-snowflake
uv add dbt-bigquery
uv add dbt-duckdb
uv lock
```

## Architecture

```text
source
  |
  v
staging
  |
  v
intermediate
  |
  v
marts
```

`staging` models clean and standardize source-shaped data, usually one model
per source object. They default to views.

`intermediate` models express reusable transformation steps that are too
specific for staging and not final enough for marts. They default to views;
individual projects can use ephemeral models where that is clearer.

`marts` models are final business-facing facts and dimensions. They default to
tables.

## Naming Conventions

Recommended model names:

```text
stg_<source>__<entity>
int_<entity>__<purpose>
dim_<entity>
fct_<entity>
```

Examples for documentation only:

```text
stg_raw__events
int_events__sessionized
dim_visitors
fct_sessions
```

Source YAML files should live near the staging models they support and use
names such as `_sources.yml` or `<source>__sources.yml`.

Model YAML files should sit beside the models they document, commonly as
`_<layer>__models.yml` or `<model_name>.yml` for larger models.

Singular SQL tests live under `tests/` and should be named for the assertion
they enforce, such as `assert_<condition>.sql`.

Macros live under `macros/` and should use verb-style names for operations or
noun-style names for reusable expressions.

Seeds live under `seeds/` and should be small, source-controlled reference
datasets. Do not use seeds for large operational data.

Snapshots live under `snapshots/` and should be named for the source entity
whose history they capture.

## Testing Standards

Use dbt tests wherever models encode assumptions. Prefer built-in tests for
common constraints:

- `not_null`
- `unique`
- `relationships`
- `accepted_values`

Use singular SQL tests under `tests/` for assertions that do not fit built-in
tests cleanly.

## Local Setup

Install dependencies into the local environment:

```sh
uv sync --frozen --group dev
```

Install pre-commit hooks:

```sh
uv run pre-commit install
```

Run static checks:

```sh
uv run ruff check .
uv run ruff format --check .
uv run sqlfluff lint . --templater jinja
uv run pre-commit run --all-files
```

Apply safe formatting:

```sh
uv run ruff check --fix .
uv run ruff format .
uv run sqlfluff format .
```

Run dbt project checks:

```sh
uv run dbt deps
uv run dbt parse
uv run dbt compile
uv run dbt test
uv run dbt build
```

`dbt parse` can validate project structure once a compatible adapter and local
profile are available. `dbt compile`, `dbt test`, and `dbt build` usually need
a real warehouse connection.

## SQLFluff

SQLFluff is configured for dbt templating with a conservative default dialect
of `ansi`. The base template uses `--templater jinja` in always-on CI because
the dbt templater needs an installed adapter and a valid profile. After adding
an adapter, run the stricter dbt-aware lint locally with:

```sh
uv run sqlfluff lint .
uv run sqlfluff format .
```

When creating a real project, update `.sqlfluff` to match the target warehouse:

```ini
[sqlfluff]
dialect = postgres
```

Use dialects such as `snowflake`, `bigquery`, `postgres`, or `duckdb` to match
the selected adapter.

## Profiles And Secrets

Do not commit real dbt credentials. `profiles.example.yml` demonstrates
environment-variable-driven configuration and is intentionally only an example.

dbt normally discovers profiles in `~/.dbt/profiles.yml`. You can also point
dbt at a repository-local uncommitted profile while developing:

```sh
DBT_PROFILES_DIR=. uv run dbt debug
```

Provide values through your shell, a local `.env` file, your secrets manager,
or CI secrets. `.env`, `.env.*`, and local `profiles.yml` files are ignored by
Git.

## GitHub Actions

`CI` runs on pull requests targeting `main` and pushes to non-`main` branches.
It installs dependencies with `uv`, runs Ruff, adapter-free SQLFluff,
`dbt deps`, and pre-commit.

The base template does not include a warehouse adapter, so instantiated
projects should add the adapter dependency before relying on CI `dbt parse`.
Set the repository variable `DBT_PARSE_ENABLED=true` and configure the
`DBT_HOST`, `DBT_USER`, `DBT_PASSWORD`, `DBT_DATABASE`, and `DBT_SCHEMA`
secrets to enable the included parse step. For full `dbt compile`, `dbt test`,
or `dbt build` in CI, add a separate credential-gated integration job with
warehouse-specific secrets.

## Semantic Release

semantic-release is configured to parse conventional commits, update
`project.version` in `pyproject.toml`, create tags like `v0.1.0`, and create
GitHub releases with the workflow-provided `GITHUB_TOKEN`.

Version bumps follow the same branch and commit conventions as the template
family:

- `feat:` creates minor releases
- `fix:` and `perf:` create patch releases
- `chore:`, `ci:`, `docs:`, `refactor:`, `style:`, and `test:` do not create
  releases by themselves

Normal local work should happen on a branch. Pre-commit blocks direct commits
to `main`; the GitHub workflows set `SKIP=no-commit-to-branch` for automation.
