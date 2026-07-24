# GKE HTTPS Setup Guide

## Prerequisites
- GCP project with GKE cluster running
- kubectl configured
- Helm installed
- A registered domain with DNS access

## Step 1 — Reserve a Global Static IP
```bash
gcloud compute addresses create your-static-ip-name \
  --global \
  --project YOUR_GCP_PROJECT_ID
```
Note: Must be GLOBAL not regional for GCE Ingress.

## Step 2 — Install cert-manager
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm install cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --set crds.enabled=true
```
Wait for cert-manager pods to be fully running before next step:
```bash
kubectl get pods -n cert-manager
```

## Step 3 — Apply namespace
```bash
kubectl create namespace your-app-namespace
```

## Step 4 — Apply Ingress manifests IN THIS ORDER
```bash
# 1. Apply cert-manager issuer first
kubectl apply -f infrastructure/k8s/base/ingress/issuer.yml

# 2. Verify issuer is ready before continuing
kubectl describe issuer letsencrypt-staging -n your-app-namespace

# 3. Apply TLS secrets
kubectl apply -f infrastructure/k8s/base/ingress/secret.yml

# 4. Apply Ingress last
kubectl apply -f infrastructure/k8s/base/ingress/ingress.yml
```

## Step 5 — Verify TLS certificate
```bash
kubectl describe certificate -n your-app-namespace
kubectl describe certificaterequest -n your-app-namespace
```

## Common mistakes
1. Applying Ingress before cert-manager is ready → silent TLS failure
2. Using regional static IP instead of global → Ingress never gets address
3. Wrong issuer name in ingress annotation → certificate never issues
4. Not waiting for issuer to be Ready before applying Ingress

## GitHub Secrets required
Add these to your GitHub repository secrets:
| Secret | Description |
|---|---|
| GCP_PROJECT_ID | Your GCP project ID |
| GCP_SA_KEY | Service account JSON key with GKE + GAR permissions |
| GKE_CLUSTER | Your GKE cluster name |
| GKE_REGION | GCP region (e.g. us-central1) |
| GAR_REPOSITORY | Artifact Registry repository name |
| APP_SECRETS | JSON object of all app environment variables |
| TURBO_TOKEN | Turborepo remote cache token (optional) |
| TURBO_TEAM | Turborepo team name (optional) |
