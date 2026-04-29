# Cosy Domain Provider — Deployment

Kubernetes manifests for the Cosy Domain Provider, managed via [Kustomize](https://kustomize.io/) and deployed with [ArgoCD](https://argo-cd.readthedocs.io/).

## How it works

Images are built and pushed to `ghcr.io` by CI in the Backend and Frontend repos. CI then updates the image tags in the relevant `overlays/*/kustomization.yaml` and pushes to this repo. ArgoCD detects the change and syncs the cluster automatically.

No manual `kubectl apply` is needed for normal deployments.

## First-time setup (new cluster)

1. Apply ArgoCD to the cluster and configure it to watch this repo.
2. Apply the required Kubernetes secrets — see [`docs/secrets-required.md`](docs/secrets-required.md).
3. Apply the ArgoCD Application manifests:

```bash
kubectl apply -f argocd/staging-app.yaml
kubectl apply -f argocd/prod-app.yaml
```

ArgoCD will create the namespaces and deploy everything automatically.

## Related repositories

| Repository | Description |
|---|---|
| [Cosy-Domain-Provider-Backend](https://github.com/Magenta-Mause/Cosy-Domain-Provider-Backend) | Spring Boot backend — builds and pushes the backend image |
| [Cosy-Domain-Provider-Frontend](https://github.com/Magenta-Mause/Cosy-Domain-Provider-Frontend) | React + Vite frontend — builds and pushes the frontend image |
| [Cosy-Domain-Provider-Systemtest](https://github.com/Magenta-Mause/Cosy-Domain-Provider-Systemtest) | Playwright end-to-end tests |
