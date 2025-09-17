# ArgoCD Applications Repository

This repository contains ONLY real user applications. All ArgoCD configuration and management has been moved to the argocd-config repository.

## Structure

```
argocd-apps/
  README.md
  kustomization.yaml
```

This repository is reserved for real user applications only. All ArgoCD infrastructure management (App-of-Apps pattern, ArgoCD configuration, Crossplane management) is handled by the argocd-config repository.
