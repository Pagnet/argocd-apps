# AWS IRSA Setup Instructions

## Prerequisites
- EKS cluster with OIDC provider configured
- AWS CLI configured with appropriate permissions
- kubectl configured for the target cluster

## Steps to Configure IRSA

### 1. Detect OIDC Provider
```bash
# Get cluster OIDC issuer URL
CLUSTER_NAME=mgmt
OIDC_ISSUER=$(aws eks describe-cluster --name $CLUSTER_NAME --query "cluster.identity.oidc.issuer" --output text)
echo "OIDC Issuer: $OIDC_ISSUER"

# Check if OIDC provider exists
OIDC_ID=$(echo $OIDC_ISSUER | cut -d '/' -f 5)
aws iam list-open-id-connect-providers | grep $OIDC_ID || {
  echo "Creating OIDC provider..."
  aws iam create-open-id-connect-provider \
    --url $OIDC_ISSUER \
    --client-id-list sts.amazonaws.com \
    --thumbprint-list 9e99a48a9960b14926bb7f3b02e22da2b0ab7280
}
```

### 2. Create IAM Policy for S3
```bash
cat > CrossplaneS3Minimal.json << 'EOF'
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:CreateBucket",
        "s3:DeleteBucket",
        "s3:GetBucket*",
        "s3:ListBucket*",
        "s3:PutBucketTagging",
        "s3:PutBucketVersioning",
        "s3:PutEncryptionConfiguration",
        "s3:GetEncryptionConfiguration"
      ],
      "Resource": "*"
    }
  ]
}
EOF

# Create the policy
aws iam create-policy \
  --policy-name CrossplaneS3Minimal \
  --policy-document file://CrossplaneS3Minimal.json
```

### 3. Create IAM Role
```bash
AWS_ACCOUNT_ID=$(aws sts get-caller-identity --query Account --output text)
OIDC_PROVIDER=$(echo $OIDC_ISSUER | sed 's|https://||')

cat > trust-policy.json << EOF
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::${AWS_ACCOUNT_ID}:oidc-provider/${OIDC_PROVIDER}"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "${OIDC_PROVIDER}:sub": "system:serviceaccount:crossplane-system:provider-aws"
        }
      }
    }
  ]
}
EOF

# Create the role
aws iam create-role \
  --role-name crossplane-provider-aws \
  --assume-role-policy-document file://trust-policy.json
```

### 4. Attach Policy to Role
```bash
aws iam attach-role-policy \
  --role-name crossplane-provider-aws \
  --policy-arn arn:aws:iam::${AWS_ACCOUNT_ID}:policy/CrossplaneS3Minimal
```

### 5. Verify Provider Deployment
```bash
# Check if provider uses correct service account
kubectl -n crossplane-system get deployment -l pkg.crossplane.io/provider=provider-aws -o yaml | grep serviceAccountName

# If needed, patch the deployment
kubectl -n crossplane-system patch deployment \
  $(kubectl -n crossplane-system get deployment -l pkg.crossplane.io/provider=provider-aws -o name) \
  --type='json' \
  -p='[{"op":"replace","path":"/spec/template/spec/serviceAccountName","value":"provider-aws"}]'
```

### 6. Verification
```bash
# Check role ARN
echo "Role ARN: arn:aws:iam::${AWS_ACCOUNT_ID}:role/crossplane-provider-aws"

# Verify provider pods are running
kubectl -n crossplane-system get pods -l pkg.crossplane.io/provider=provider-aws

# Check provider config
kubectl -n crossplane-system get providerconfig default -o yaml
```
