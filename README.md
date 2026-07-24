# GKE Multi-Service CI/CD Pipeline Template

A production-grade CI/CD pipeline template for deploying 
multi-language microservices to Google Kubernetes Engine (GKE).

Supports Node.js, Bun, Next.js, and Python services in a 
single monorepo — with automated testing, Docker builds, 
Google Artifact Registry, Kustomize-based deployments, 
and HTTPS via cert-manager + Let's Encrypt.

---

## What this template includes

**build.yml — Pull Request pipeline**
- Lint + TypeScript type-checking
- Node.js tests with real MongoDB + Valkey service containers
- Python tests via Pytest + uv
- Full monorepo build via Turborepo
- Security audit (Node.js + Python dependencies)

**deploy.yml — Push to main / Release pipeline**
- Environment resolution (staging vs production)
- Parallel Docker builds per service via matrix strategy
- Push to Google Artifact Registry
- Kustomize-based Kubernetes manifest generation
- Secret sync from GitHub Secrets to Kubernetes
- Automated rollout with health verification

**Infrastructure**
- GKE Ingress with GCE controller
- TLS via cert-manager + Let's Encrypt
- Kustomize overlays for staging and production

---

## Architecture

```
GitHub Push / PR
   │
   ▼
┌─────────────────────────────────┐
│ build.yml                       │
│ lint → test → build → audit     │
└─────────────────┬───────────────┘
                   │ (on merge to main)
                   ▼
┌─────────────────────────────────┐
│ deploy.yml                      │
│ resolve-env                     │
│                                 │
│   ▼                             │
│ build & push (matrix)           │
│ ├── service-1                   │
│ ├── service-2                   │
│ ├── ai-engine (Python)          │
│ └── web (Next.js)               │
│                                 │
│   ▼                             │
│ deploy to GKE                   │
│ ├── sync secrets                │
│ ├── kustomize build             │
│ ├── kubectl apply               │
│ └── verify rollout              │
└─────────────────────────────────┘
```

---

## Services supported

| Service | Runtime | Description |
|---|---|---|
| orchestrator-service | Node.js | Main backend service |
| bridge-service | Node.js / Bun | Integration bridge |
| ai-engine | Python 3.11 | AI/ML processing service |
| web | Next.js | Frontend application |

---

## Prerequisites

- GCP project with billing enabled
- GKE cluster running
- Google Artifact Registry repository created
- kubectl installed and configured
- Helm installed (for cert-manager)
- Domain with DNS access

---

## Quick start

### Step 1 — Clone this template
```bash
git clone https://github.com/kcshekar/gke-multiservice-cicd-template
cd gke-multiservice-cicd-template
```

### Step 2 — Configure GitHub Secrets
Add these secrets to your GitHub repository
(Settings → Secrets and variables → Actions):

| Secret | Description |
|---|---|
| GCP_PROJECT_ID | Your GCP project ID |
| GCP_SA_KEY | Service account JSON key |
| GKE_CLUSTER | Your GKE cluster name |
| GKE_REGION | GCP region (e.g. us-central1) |
| GAR_REPOSITORY | Artifact Registry repo name |
| APP_SECRETS | JSON of all app environment variables |
| TURBO_TOKEN | Turborepo remote cache token (optional) |
| TURBO_TEAM | Turborepo team name (optional) |

### Step 3 — Update deploy.yml
Replace placeholder values in deploy.yml:
- Service names to match your services
- Dockerfile paths for each service
- Namespace to match your app

### Step 4 — Set up HTTPS on GKE
Follow the detailed guide in docs/setup-guide.md

**Critical order — do not skip steps:**
1. Install cert-manager first
2. Verify cert-manager is running
3. Apply issuer
4. Apply secrets
5. Apply ingress last

Skipping this order causes silent TLS failures
with no useful error messages.

### Step 5 — Deploy
```bash
# Staging — push to main branch
git push origin main

# Production — create a GitHub release
gh release create v1.0.0
```

---

## How staging vs production works

| Trigger | Environment |
|---|---|
| Push to main | Staging |
| GitHub Release published | Production |

Kustomize overlays in infrastructure/k8s/overlays/ 
handle environment-specific configuration.

---

## Key design decisions

**Why Turborepo?**
Monorepo build caching — only rebuilds services that changed.
Significantly faster CI on large repos.

**Why matrix strategy for Docker builds?**
Builds all service images in parallel — cuts deploy time 
proportionally to number of services.

**Why Valkey instead of Redis in tests?**
Valkey is the open-source Redis fork. Drop-in compatible,
no licensing concerns.

**Why uv for Python?**
Significantly faster than pip. Deterministic installs
via uv.lock. Built-in virtualenv management.

**Why GCE Ingress instead of nginx-ingress?**
Native GCP integration — automatically provisions 
Google Cloud Load Balancer. Better for GKE on GCP.

**Why Kustomize instead of Helm?**
No templating language to learn. Plain YAML with overlays.
Easier to read and audit.

---

## Common issues

**TLS certificate stuck in pending**
→ cert-manager not fully running before issuer was applied.
→ Reinstall cert-manager, wait for all pods Ready, reapply.

**Ingress has no external IP after 10+ minutes**
→ Static IP must be GLOBAL not regional.
→ Check: gcloud compute addresses list

**Build fails: bun.lock not found**
→ Run bun install locally and commit bun.lock

**Python tests fail in CI but pass locally**
→ Check uv.lock is committed — uv sync --frozen 
  requires the lockfile.

---

## Folder structure

```
.
├── .github/
│   └── workflows/
│       ├── build.yml        # PR pipeline
│       └── deploy.yml       # Deploy pipeline
├── infrastructure/
│   └── k8s/
│       ├── base/
│       │   ├── kustomization.yaml
│       │   └── ingress/
│       │       ├── ingress.yml
│       │       ├── issuer.yml
│       │       └── secret.yml
│       └── overlays/
│           ├── staging/
│           └── production/
├── docs/
│   └── setup-guide.md
├── .env.example
├── .gitignore
└── README.md
```

---

## Author

Built by [Korpole Chandra Shekar](https://korpolechandrashekar.com)  
Senior Software Engineer · Node.js · GKE · AI/LLM Pipelines  
[LinkedIn](https://www.linkedin.com/in/chandra-shekar-korpole) · 
[Portfolio](https://korpolechandrashekar.com)

---

## License

MIT — use freely, attribution appreciated.
