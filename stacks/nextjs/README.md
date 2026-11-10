# Next.js

Start from the generator, then apply only what's listed here:

```bash
pnpm create next-app@latest <name> --ts --app --src-dir --eslint --tailwind --use-pnpm
```

## On top of the generator
- `src/app/` holds routes only. Features go in `src/features/<name>/`, shared UI in `src/components/`.
- Set `output: "standalone"` in `next.config.ts` and copy this `Dockerfile`.
- Add `src/app/healthz/route.ts` and `src/app/readyz/route.ts`. Each returns 200; `readyz` also checks what the app can't serve without.
- Tests: Vitest for logic, Playwright with axe for the main flows (T1).

## AGENTS.md snippet
```markdown
- Verify: `pnpm tsc --noEmit && pnpm lint && pnpm test`
- Dev: `pnpm dev`
```

CI: `ci-node-pnpm.yml` with `verify: pnpm tsc --noEmit && pnpm lint && pnpm test`.
