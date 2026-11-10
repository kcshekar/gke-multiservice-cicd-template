FROM python:3.13-slim AS build
COPY --from=ghcr.io/astral-sh/uv:0.13.7 /uv /usr/local/bin/uv
WORKDIR /app
ENV UV_LINK_MODE=copy
COPY pyproject.toml uv.lock README.md ./
RUN uv sync --frozen --no-dev --no-install-project
COPY src ./src
RUN uv sync --frozen --no-dev --no-editable

FROM python:3.13-slim
WORKDIR /app
COPY --from=build /app/.venv ./.venv
ENV PATH=/app/.venv/bin:$PATH
USER 1000
EXPOSE 8080
CMD ["uvicorn", "your_package.main:app", "--host", "0.0.0.0", "--port", "8080"]
