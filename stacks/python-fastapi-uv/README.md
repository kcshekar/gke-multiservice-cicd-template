# Python + FastAPI + uv

Start from the generator, then apply only what's listed here:

```bash
uv init --package <name> && cd <name>
uv add fastapi 'uvicorn[standard]' && uv add --dev pytest ruff
```

## On top of the generator
- `src/<name>/main.py` only wires the app. Features go in `src/<name>/modules/<feature>/`, with tests in `tests/`.
- In this `Dockerfile`, replace `your_package` with the package name.
- Add `GET /healthz` and `GET /readyz`. Read settings from the environment at startup.
- Set a timeout on every outbound call, including LLM calls.

## AGENTS.md snippet
```markdown
- Verify: `uv run ruff check . && uv run pytest -q`
- Dev: `uv run uvicorn your_package.main:app --reload`
```

CI: `ci-python-uv.yml` with `verify: uv run ruff check . && uv run pytest -q`.
