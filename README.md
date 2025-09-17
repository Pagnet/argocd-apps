# ArgoCD Applications Repository

This repository contains ALL applications, including ArgoCD and Crossplane management applications, following the exact specifications from the original prompt.

## Structure

```
argocd-apps/
  README.md
  root/
    app-of-apps.yaml
  apps/
    argocd-config.yaml
    crossplane-core.yaml
    crossplane-providers.yaml
    crossplane.yaml
    providers/
      provider-aws.yaml
      providerconfigs.yaml
      irsa/
        serviceaccount.yaml
        iam-instructions.md
  kustomization.yaml
```

This repository contains ALL applications (including ArgoCD and Crossplane management applications) using the App-of-Apps pattern. The argocd-config repository contains only base configurations (projects, repositories, clusters, RBAC).
