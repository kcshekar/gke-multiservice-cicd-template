# Node + Express + Turborepo

Start from the generator, then apply only what's listed here:

```bash
pnpm dlx create-turbo@latest <name> --package-manager pnpm
```

## On top of the generator
- `apps/<service>/` for each deployable, `packages/<name>/` for shared code.
- Inside a service, one folder per feature under `src/modules/`, with tests beside the code.
- Read `PORT` from the environment and validate all env vars at startup.
- Add `GET /healthz` and `GET /readyz`. Set a timeout on every outbound call.
- Each deployable needs a `Dockerfile` based on this one; see the build command at its top.

## AGENTS.md snippet
```markdown
- Verify: `pnpm turbo run typecheck test`
- Dev: `pnpm turbo run dev`
```

CI: `ci-node-pnpm.yml` with `verify: pnpm turbo run typecheck test`.
