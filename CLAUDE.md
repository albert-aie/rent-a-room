# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

"Rent a Room" is a FastAPI booking API built incrementally as a course project ("sections 1-10" per `pyproject.toml`). Python 3.14, managed with `uv`. There is no git repository and no test suite yet.

## Commands

```bash
uv sync                          # install deps (including the dev group with ruff)
uv run fastapi dev main.py       # dev server with reload; docs at http://localhost:8000/docs
uv run ruff check --fix .        # lint (only isort rules "I" are enabled)
uv run ruff format .             # format
```

VS Code is set up to format and fix with ruff on save. `.vscode/launch.json` debugs via `uvicorn main:app --reload`.

## Architecture

- `main.py` holds the whole app: the FastAPI instance (with OpenAPI metadata and tags), the request-parameter Pydantic models (`AppCookies`, `AppHeaders`, `RoomQueryParams`) and every route. Cookies, headers and query params are read as Pydantic models through `Annotated[Model, Cookie()/Header()/Query()]`.
- `database.py` builds a synchronous SQLModel engine for SQLite (`database.db` in the working directory, `echo=True`) and exposes `SessionDep` for dependency injection. It runs `import models` so that `SQLModel.metadata.create_all()` picks up the table classes. Any new table module has to be imported here too.
- `create_db_and_tables()` runs in the app's `lifespan` at startup. There are no migrations yet, although `alembic` is a dependency.
- `models.py` defines the `Room` table (`rooms`). **The `/rooms` routes do not use the database yet.** They read the hardcoded dicts `apartment`, `house` and `studio` in `main.py`. Only `GET /` touches the DB, and only to run a `SELECT 5` connectivity check.
- `assets/` is served as static files at `/assets`. The directory must exist or the app fails at startup.
- Some dependencies are installed but not used yet: `aiosqlite`, `greenlet` (async SQLAlchemy), `alembic` and `pydantic-extra-types[phonenumbers]`.
