# GKE setup guide

Run these once per cluster or project. Replace the capitalised placeholders.

## 1. Reserve global static IPs
GCE Ingress needs a global address, not a regional one. Do this once per environment.
```bash
gcloud compute addresses create your-static-ip-prod --global --project PROJECT_ID
```
Point the domain's DNS A record at the address before step 6.

## 2. Install the cluster add-ons
Check for newer versions first. These were current on 2026-11-10.
```bash
helm repo add jetstack https://charts.jetstack.io
helm repo add external-secrets https://charts.external-secrets.io
helm repo update
helm install cert-manager jetstack/cert-manager -n cert-manager --create-namespace \
  --version v1.21.3 --set crds.enabled=true
helm install external-secrets external-secrets/external-secrets -n external-secrets --create-namespace \
  --version 2.15.0
```

## 3. Let the cluster read Secret Manager
Put the app's env vars in one Secret Manager secret, as a JSON object. Never commit them.
```bash
gcloud iam service-accounts create your-reader --project PROJECT_ID
gcloud secrets add-iam-policy-binding your-app-prod-config --project PROJECT_ID \
  --member "serviceAccount:your-reader@PROJECT_ID.iam.gserviceaccount.com" \
  --role roles/secretmanager.secretAccessor
gcloud iam service-accounts add-iam-policy-binding your-reader@PROJECT_ID.iam.gserviceaccount.com \
  --member "serviceAccount:PROJECT_ID.svc.id.goog[your-app/external-secrets-reader]" \
  --role roles/iam.workloadIdentityUser
```

## 4. Let GitHub deploy without a key
This uses GitHub OIDC, and only the named repo can sign in.
```bash
gcloud iam workload-identity-pools create github --location global --project PROJECT_ID
gcloud iam workload-identity-pools providers create-oidc github --location global \
  --workload-identity-pool github --project PROJECT_ID \
  --issuer-uri https://token.actions.githubusercontent.com \
  --attribute-mapping "google.subject=assertion.sub,attribute.repository=assertion.repository" \
  --attribute-condition "assertion.repository=='OWNER/REPO'"
gcloud iam service-accounts add-iam-policy-binding deployer@PROJECT_ID.iam.gserviceaccount.com \
  --role roles/iam.workloadIdentityUser \
  --member "principalSet://iam.googleapis.com/projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/github/attribute.repository/OWNER/REPO"
```
The `deployer` account needs `roles/artifactregistry.writer` and `roles/container.developer`.

## 5. Protect production
In GitHub, add required reviewers to the `production` environment. Environments with reviewers need a public repo or a paid plan. On a private Free repo, trigger production only by publishing a release yourself.

## 6. Deploy staging first
```bash
kubectl apply -k k8s/overlays/staging
kubectl describe certificate app-tls -n your-app-staging
```
A staging certificate is untrusted by browsers; that's expected. Once it's issued, deploy production, which uses `letsencrypt-prod`.

## Common problems
| Symptom | Cause |
|---|---|
| Ingress never gets an address | The static IP is regional, or its name doesn't match the overlay |
| Certificate stuck pending | DNS doesn't point at the IP yet, or port 80 is closed. If the HTTPS redirect is in the way, remove the `FrontendConfig` annotation until the first certificate is issued |
| `no matches for kind` on apply | cert-manager or External Secrets isn't installed (step 2) |
| Backends unhealthy | `/readyz` doesn't return 200 on port 8080 |
| Pods rejected by the namespace | The image runs as root; the namespace enforces the `restricted` policy |
