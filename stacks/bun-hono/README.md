# Bun + Hono

Start from the generator, then apply only what's listed here:

```bash
bun create hono@latest <name>   # choose the bun template
```

## On top of the generator
- `src/index.ts` only wires the app. Features go in `src/modules/<name>/`, each with its `routes.ts` and tests beside the code.
- Read `PORT` from the environment and validate all env vars at startup.
- Add `GET /healthz` (process is up) and `GET /readyz` (dependencies respond).
- Set a timeout on every outbound call.

## AGENTS.md snippet
```markdown
- Verify: `bunx tsc --noEmit && bun test`
- Dev: `bun run dev`
```

CI: `ci-bun.yml` with `verify: bunx tsc --noEmit && bun test`.
