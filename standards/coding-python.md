# Python Coding Standards

Applies to every Python service and tool in the portfolio (FastAPI backends, CLI tools).

## 1. Tooling Baseline

- Python 3.12+ for new projects; an existing project's minimum version is declared explicitly (`requires-python` in `pyproject.toml`) and enforced in CI.
- `ruff` for linting and import sorting, `black` for formatting; both run as CI gates (`ruff check .`, `black --check .`), not just as local editor integrations.
- Type hints on every public function signature; `mypy` (or `pyright`) runs in CI in at least basic strictness mode. A function without type hints is acceptable only for trivial, private (`_prefixed`) helpers.

## 2. Dependency Management

- `pyproject.toml` with a lock file (`poetry.lock`, `uv.lock`, or `requirements.txt` pinned via `pip-compile`) is committed; CI installs from the lock file, not a loose `requirements.txt` with unpinned versions.
- Virtual environments are never committed; `.venv`/`venv` is `.gitignore`d.
- `pip-audit` (or the ecosystem-equivalent) runs in CI; a new high/critical advisory blocks merge unless explicitly, temporarily accepted with a stated reason.

## 3. Project Structure

- `src/` layout (`src/<package_name>/...`) for anything intended to be installed as a package, so tests cannot accidentally import from the working directory instead of the installed package.
- One module per cohesive concern; a module that accumulates unrelated helpers is split before it becomes a dumping ground.
- Configuration is loaded via environment variables (using `pydantic-settings` or an equivalent typed settings loader), never read ad hoc with scattered `os.environ.get(...)` calls through the codebase.

## 4. Error Handling

- Custom exception classes for domain errors, inheriting from a single project-level base exception, so calling code can catch at the right granularity.
- Never use a bare `except:`; catch the specific exception type expected, and let genuinely unexpected exceptions propagate rather than being silently swallowed.
- FastAPI services translate domain exceptions to HTTP responses via a centralized exception handler, not scattered `try/except` blocks duplicated in every endpoint.

## 5. Async and I/O

- FastAPI endpoints doing I/O (database, HTTP calls to other services) are `async def` using an async driver (`asyncpg`, `httpx` in async mode); blocking calls inside an async endpoint (a synchronous DB driver, `requests` instead of `httpx`) silently stall the event loop and degrade throughput under load.
- CPU-bound work inside an async application is offloaded to a thread/process pool (`run_in_executor` or a task queue), not run inline on the event loop.

## 6. Data Validation

- Pydantic models (v2) define every API request/response shape; validation happens at the boundary, not deep inside business logic.
- Database models and API schema models are kept as distinct types (even if generated from a shared source), so a database migration does not silently change the public API contract.

## 7. Testing

- `pytest` for unit and integration tests; fixtures for database/external-service setup live in `conftest.py`, not duplicated per test file.
- Integration tests against a database run against a real, ephemeral instance (SQLite for speed where the production database is Postgres-compatible enough, or a containerized Postgres via `testcontainers` when exact compatibility matters), never mocked at the ORM layer for anything beyond pure unit tests. This portfolio has previously been burned by a mocked-database test suite passing while the real migration failed in production; integration tests exist specifically to catch that class of gap.
- Coverage target is roughly 80%; the goal is meaningful coverage of business logic, not a number chased by testing trivial getters or framework-provided code.

## 8. Security Specifics

- SQL access goes through the ORM's parameterized query builder or explicit parameter binding; string-formatted SQL is never acceptable, in any code path, including internal admin scripts.
- Secrets are read from environment variables or Key Vault at runtime, never imported from a committed settings module with a default value that happens to be a real credential.
- `subprocess` calls that incorporate any external input use argument lists (`subprocess.run([...])`), never `shell=True` with string-interpolated input.
