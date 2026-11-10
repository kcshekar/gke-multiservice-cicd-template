# Expo (React Native)

Mobile has no Kubernetes deploy and no Dockerfile. It ships through EAS.

Start from the generator, then apply only what's listed here:

```bash
pnpm create expo-app@latest <name>
```

## On top of the generator
- `app/` holds routes only (Expo Router). Features go in `src/features/<name>/`.
- `eas build` and `eas submit` for store builds; profiles `preview` and `production` in `eas.json`.
- Only `EXPO_PUBLIC_*` values reach the app, and they're readable by anyone with the app. Keep secrets on the backend.
- Generate the API client from the backend's OpenAPI spec so the contract can't drift.

## AGENTS.md snippet
```markdown
- Verify: `pnpm tsc --noEmit && pnpm lint && pnpm test`
- Dev: `pnpm expo start`
```

CI: `ci-node-pnpm.yml` with the same verify command.
