# ArgoCD Applications Repository

This repository contains the application definitions for ArgoCD using the App-of-Apps pattern with external Helm charts.

## Repository Structure

```
argocd-apps/
├── applications/          # Application definitions
├── apps/                 # App-of-Apps applications
├── projects/             # ArgoCD project definitions
└── README.md            # This file
```

## Overview

This repository is managed by ArgoCD and contains:

- **Applications**: Individual application definitions
- **Apps**: App-of-Apps pattern applications
- **Projects**: ArgoCD project definitions for access control

**Note**: Helm charts are stored in the external Helm repository at `https://blu-helm-charts.blu.com.br/api/charts`

## How It Works

1. **Bootstrap**: The `argocd-configuration` repository deploys the bootstrap application
2. **Configuration Manager**: Manages ArgoCD configuration and projects
3. **Applications Manager**: Deploys applications from this repository
4. **App-of-Apps**: Each application can manage other applications
5. **Helm Charts**: Applications reference charts from the external Helm repository

## Adding New Applications

### 1. Create Application Definition

Create an application file in `apps/`:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://blu-helm-charts.blu.com.br/api/charts
    targetRevision: "1.0.0"  # Chart version
    chart: my-app  # Chart name from Helm repository
    helm:
      values: |
        # Your Helm values here
        image:
          tag: "latest"
        replicaCount: 3
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 2. Commit and Push

```bash
git add .
git commit -m "Add new application: my-app"
git push origin main
```

## Helm Chart Integration

### Using External Helm Repository

Applications reference charts from the external Helm repository:

```yaml
source:
  repoURL: https://blu-helm-charts.blu.com.br/api/charts
  targetRevision: "1.0.0"  # Chart version
  chart: chart-name  # Chart name
```

### Values Override

You can override Helm values in the application definition:

```yaml
helm:
  values: |
    image:
      tag: "latest"
    replicaCount: 3
    resources:
      limits:
        cpu: 1000m
        memory: 1Gi
```

### External Values Files

Reference external values files from your Helm repository:

```yaml
helm:
  valueFiles:
    - values-prod.yaml
    - values-override.yaml
```

## Projects

Projects define access control and resource permissions:

- **default**: Basic application deployment permissions
- **argocd-configuration**: ArgoCD configuration management

## Best Practices

1. **Use External Helm Charts**: Reference charts from your Helm repository
2. **Version Management**: Always specify chart versions for consistency
3. **Environment Separation**: Use different values for different environments
4. **Resource Limits**: Always define resource requests and limits
5. **Health Checks**: Include health checks for critical resources
6. **Labels**: Use consistent labeling for organization

## Troubleshooting

### Application Sync Issues

1. Check application status: `argocd app get <app-name>`
2. View logs: `argocd app logs <app-name>`
3. Check events: `kubectl get events -n <namespace>`

### Helm Chart Issues

1. Verify chart exists: Check your Helm repository
2. Validate chart version: Ensure version exists
3. Check values: Verify Helm values are correct
4. Repository access: Ensure ArgoCD can access the Helm repository

## Security

- Repository credentials are stored in Kubernetes secrets
- Projects enforce access control and resource permissions
- RBAC policies restrict user access based on groups
- Helm repository access is controlled by ArgoCD projects

## Support

For issues related to:
- **Applications**: Check ArgoCD UI or CLI
- **Helm Charts**: Verify chart availability in your Helm repository
- **Configuration**: Review project and repository settings
