# AWS Provider Security Setup Guide

## Overview
This guide provides step-by-step instructions for setting up secure AWS provider configuration using IRSA (IAM Roles for Service Accounts).

## Prerequisites
- EKS cluster with OIDC provider enabled
- AWS CLI configured with appropriate permissions
- kubectl configured to access your EKS cluster

## Step 1: Enable OIDC Provider (if not already enabled)

```bash
# Get cluster OIDC issuer URL
aws eks describe-cluster --name <cluster-name> --query "cluster.identity.oidc.issuer" --output text

# Create OIDC provider
eksctl utils associate-iam-oidc-provider --cluster <cluster-name> --approve
```

## Step 2: Create IAM Policy

Create a policy with the necessary AWS permissions:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:*",
                "s3:*",
                "rds:*",
                "eks:*",
                "iam:*",
                "lambda:*",
                "secretsmanager:*",
                "elasticache:*",
                "route53:*",
                "ecr:*"
            ],
            "Resource": "*"
        }
    ]
}
```

## Step 3: Create IAM Role

```bash
# Create trust policy
cat > trust-policy.json << EOF
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "arn:aws:iam::<ACCOUNT-ID>:oidc-provider/oidc.eks.<REGION>.amazonaws.com/id/<OIDC-ID>"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "oidc.eks.<REGION>.amazonaws.com/id/<OIDC-ID>:sub": "system:serviceaccount:crossplane-system:crossplane",
                    "oidc.eks.<REGION>.amazonaws.com/id/<OIDC-ID>:aud": "sts.amazonaws.com"
                }
            }
        }
    ]
}
EOF

# Create IAM role
aws iam create-role \
    --role-name crossplane-provider-aws \
    --assume-role-policy-document file://trust-policy.json

# Attach policy to role
aws iam attach-role-policy \
    --role-name crossplane-provider-aws \
    --policy-arn arn:aws:iam::<ACCOUNT-ID>:policy/CrossplaneProviderAWS
```

## Step 4: Create Service Account

```bash
# Create service account
kubectl create serviceaccount crossplane -n crossplane-system

# Annotate service account with IAM role
kubectl annotate serviceaccount crossplane \
    -n crossplane-system \
    eks.amazonaws.com/role-arn=arn:aws:iam::<ACCOUNT-ID>:role/crossplane-provider-aws
```

## Step 5: Update ProviderConfig

Update the ProviderConfig to use the specific role:

```yaml
apiVersion: aws.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: default
spec:
  credentials:
    source: InjectedIdentity
  region: us-east-1
  assumeRoleWithWebIdentity:
    roleARN: arn:aws:iam::<ACCOUNT-ID>:role/crossplane-provider-aws
    webIdentityTokenFile: "/var/run/secrets/eks.amazonaws.com/serviceaccount/token"
```

## Security Best Practices

1. **Least Privilege**: Only grant the minimum permissions required
2. **Resource Scoping**: Use resource-specific policies when possible
3. **Regular Auditing**: Monitor and audit AWS API calls
4. **Secret Rotation**: Regularly rotate IAM credentials
5. **Network Security**: Use VPC endpoints for AWS services when possible

## Troubleshooting

### Common Issues

1. **Permission Denied**: Check IAM role permissions and trust policy
2. **OIDC Issues**: Verify OIDC provider is correctly configured
3. **Service Account**: Ensure service account is properly annotated

### Debug Commands

```bash
# Check service account
kubectl describe serviceaccount crossplane -n crossplane-system

# Check provider status
kubectl get providers -n crossplane-system

# Check provider logs
kubectl logs -n crossplane-system -l app=crossplane
```

## References

- [Crossplane AWS Provider Documentation](https://docs.crossplane.io/latest/getting-started/provider-aws/)
- [EKS IAM Roles for Service Accounts](https://docs.aws.amazon.com/eks/latest/userguide/iam-roles-for-service-accounts.html)
- [AWS IAM Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
