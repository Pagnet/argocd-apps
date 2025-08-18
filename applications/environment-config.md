# Environment-Specific Configurations

This document describes the differences between development, staging, and production environments for the sample Helm application.

## Environment Overview

| Environment | Purpose | Replicas | Resource Limits | Auto-scaling | Debug | Ingress |
|-------------|---------|----------|-----------------|--------------|-------|---------|
| Development | Development & Testing | 1 | 250m CPU, 256Mi RAM | Disabled | Enabled | Staging SSL |
| Staging | QA & Pre-production | 2 | 500m CPU, 512Mi RAM | 2-5 replicas | Disabled | Staging SSL |
| Production | Live Production | 3 | 1000m CPU, 1Gi RAM | 3-10 replicas | Disabled | Production SSL |

## Configuration Differences

### Development Environment (`applications/dev/`)

- **Resources**: Minimal resources for cost optimization
- **Replicas**: Single replica for development
- **Auto-scaling**: Disabled
- **Debug**: Enabled with debug logging
- **SSL**: Uses staging Let's Encrypt certificates
- **Sync Policy**: Faster retry with shorter backoff
- **Revision History**: Limited to 5 revisions

### Staging Environment (`applications/staging/`)

- **Resources**: Medium resources for testing
- **Replicas**: 2 replicas with auto-scaling (2-5)
- **Auto-scaling**: Enabled with moderate thresholds
- **Debug**: Disabled, info-level logging
- **SSL**: Uses staging Let's Encrypt certificates
- **Sync Policy**: Standard retry configuration
- **Revision History**: 10 revisions

### Production Environment (`applications/prod/`)

- **Resources**: Full resources for production load
- **Replicas**: 3 replicas with auto-scaling (3-10)
- **Auto-scaling**: Enabled with production thresholds
- **Debug**: Disabled, warn-level logging
- **SSL**: Uses production Let's Encrypt certificates
- **Security**: Enhanced security context
- **Sync Policy**: Conservative retry with longer backoff
- **Revision History**: 15 revisions

## Namespace Strategy

Each environment uses separate namespaces:
- Development: `sample-apps-dev`
- Staging: `sample-apps-staging`
- Production: `sample-apps-prod`

## Project Access Control

- **Development**: Developers have full access
- **Staging**: Developers and QA team have access
- **Production**: Developers (read-only), SRE team, and production admins

## Deployment Strategy

1. **Development**: Automated deployment with immediate sync
2. **Staging**: Automated deployment with standard sync policies
3. **Production**: Automated deployment with conservative sync policies

## Monitoring and Health Checks

All environments include health checks for:
- Deployment status
- Service availability

## Resource Management

### CPU and Memory Limits

| Environment | CPU Request | CPU Limit | Memory Request | Memory Limit |
|-------------|-------------|-----------|----------------|--------------|
| Development | 100m | 250m | 128Mi | 256Mi |
| Staging | 250m | 500m | 256Mi | 512Mi |
| Production | 500m | 1000m | 512Mi | 1Gi |

### Auto-scaling Configuration

| Environment | Min Replicas | Max Replicas | Target CPU |
|-------------|--------------|--------------|------------|
| Development | 1 | 1 | N/A (disabled) |
| Staging | 2 | 5 | 70% |
| Production | 3 | 10 | 80% |

## Best Practices

1. **Environment Parity**: Keep configurations as similar as possible
2. **Resource Scaling**: Scale resources proportionally across environments
3. **Security**: Increase security measures in production
4. **Monitoring**: Implement comprehensive monitoring in production
5. **Backup**: Ensure proper backup strategies for production data
