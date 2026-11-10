# GKE multi-service CI/CD template

Kubernetes manifests, shared GitHub Actions workflows and per-stack starting points for services on Google Kubernetes Engine.

- No keys in GitHub: deploys sign in with OIDC.
- No secrets in git: cert-manager creates TLS Secrets, and External Secrets copies app config from Secret Manager.
- Every action is pinned to a commit SHA, and Renovate keeps the pins current after a 7-day wait.

## What's here

```text
k8s/base/            Namespace, ServiceAccount, NetworkPolicy
k8s/base/app/        one service: Deployment, Service, BackendConfig, HPA, PDB
k8s/components/      ingress (GCE Ingress, HTTPS redirect, Let's Encrypt issuers)
                     external-secrets (Secret Manager to a Kubernetes Secret)
k8s/overlays/        staging, production: domains, replicas, resources, image tags
.github/workflows/   reusable: ci-bun, ci-node-pnpm, ci-python-uv, security-scan,
                     build-push, deploy-gke
stacks/              nextjs, bun-hono, node-express-turbo, python-fastapi-uv,
                     expo-react-native
default.json         shared Renovate preset
```

## Use the Kubernetes files
1. Copy `k8s/` into your repo.
2. Replace the placeholders: `your-app`, `your-project`, `your-repo`, `your-reader`, `your-static-ip`, `example.com`, `you@example.com`.
3. Add a service by copying `k8s/base/app/`, renaming it, and listing it in `k8s/base/kustomization.yaml`. Add its route to the Ingress in each overlay.
4. Follow [the setup guide](docs/setup-guide.md) once per cluster.

Your app must serve `GET /healthz` and `GET /readyz` on port 8080, as a non-root user.

## Use the workflows
Pin to a commit SHA of this repo, and bump it when you want a change.

```yaml
permissions:
  contents: read

jobs:
  verify:
    uses: OWNER/gke-multiservice-cicd-template/.github/workflows/ci-bun.yml@COMMIT_SHA
    with:
      bun-version: "1.4.4"
      postgres-image: postgres:16
      verify: bun run typecheck && bun run test
  scan:
    uses: OWNER/gke-multiservice-cicd-template/.github/workflows/security-scan.yml@COMMIT_SHA
```

A deploy runs `build-push.yml` once per image, then `deploy-gke.yml`. Give the calling jobs `permissions: { id-token: write, contents: read }`.

## Use a stack
Run the official generator named in `stacks/<stack>/README.md`, then apply the short list there. These are not project skeletons, because copied skeletons go stale.

## Renovate
Install the Renovate GitHub app on the repo and add this `renovate.json`:

```json
{ "extends": ["github>OWNER/gke-multiservice-cicd-template"] }
```

## Author

Built by [Korpole Chandra Shekar](https://korpolechandrashekar.com)
Senior Software Engineer · Node.js · GKE · AI/LLM Pipelines
[LinkedIn](https://www.linkedin.com/in/chandra-shekar-korpole) ·
[Portfolio](https://korpolechandrashekar.com)

## License

MIT — use freely, attribution appreciated.
