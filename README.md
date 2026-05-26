# di-devops-gitops

GitOps repository — manages all DI DevOps platform apps via Argo CD.

## Repository structure

```
di-devops-gitops/
├─ charts/
│  └─ generic-app/          # Reusable Helm chart for any containerised app
├─ environments/
│  ├─ dev/apps/             # Per-app Helm values for dev
│  ├─ staging/apps/         # Per-app Helm values for staging
│  └─ prod/apps/            # Per-app Helm values for prod
└─ argocd/
   ├─ projects/             # Argo CD AppProject definitions
   ├─ applicationsets/      # ApplicationSet — auto-generates Applications
   └─ bootstrap/            # One-time bootstrap (apply manually)
```

## How it works

1. Each `environments/<env>/apps/<app>.yaml` file is a Helm values override
   that also contains metadata keys (`appName`, `environment`, `namespace`).
2. The **ApplicationSet** (`argocd/applicationsets/apps-applicationset.yaml`)
   discovers those files via a Git file generator and creates one Argo CD
   Application per file, pointing to `charts/generic-app`.
3. The **root app** (`argocd/bootstrap/root-app.yaml`) watches `argocd/` and
   keeps the project + ApplicationSet in sync automatically.

## Bootstrap (one-time)

```bash
# 1. Install Argo CD (if not already present)
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# 2. Apply the root app — Argo CD takes over from here
kubectl apply -f argocd/bootstrap/root-app.yaml
```

## Adding a new application

```bash
# Copy an existing values file and adjust as needed
cp environments/dev/apps/di-devops-test.yaml environments/dev/apps/<new-app>.yaml
# Edit appName, namespace, image, etc., then push — ApplicationSet picks it up automatically
```

## Adding a new environment

```bash
mkdir -p environments/<new-env>/apps
cp environments/dev/apps/di-devops-test.yaml environments/<new-env>/apps/di-devops-test.yaml
# Edit environment + namespace fields, then push
```
