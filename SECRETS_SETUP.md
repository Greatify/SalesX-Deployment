# Secrets Setup Guide

This guide explains how to properly set up secrets for the SalesX deployment.

## ⚠️ Important Security Note

**NEVER commit real credentials to Git!** The secret files in this repository contain placeholder values only.

## 🔐 Setting Up Secrets

### Option 1: Create Local Secret Files (Recommended)

1. **Copy the template files with `.local` suffix:**

```bash
cd k8s/backend/base/secrets/

# Copy and edit with real credentials
cp salesx-backup-secrets.yaml salesx-backup-secrets.yaml.local
```

2. **Edit the `.local` files with your real credentials:**

```bash
# Edit with your real AWS credentials
nano salesx-backup-secrets.yaml.local
```

3. **Apply the local files (ignored by git):**

```bash
kubectl apply -f k8s/backend/base/secrets/salesx-backup-secrets.yaml.local -n salesx-production
```

### Option 2: Use kubectl create secret (Direct)

```bash
# Create backup secrets directly
kubectl create secret generic salesx-backup-secrets \
  --from-literal=aws-access-key-id="YOUR_AWS_ACCESS_KEY_ID" \
  --from-literal=aws-secret-access-key="YOUR_AWS_SECRET_ACCESS_KEY" \
  --from-literal=s3-backup-bucket="salesx-greatify" \
  --from-literal=aws-region="ap-south-1" \
  -n salesx-production
```

### Option 3: Use AWS Secrets Manager + CSI Driver (Production)

For production, use AWS Secrets Manager with the CSI Driver instead of Kubernetes secrets:

1. **Store secrets in AWS Secrets Manager:**

```bash
aws secretsmanager create-secret \
  --name salesx/backup/prod \
  --secret-string '{
    "aws-access-key-id": "YOUR_AWS_ACCESS_KEY_ID",
    "aws-secret-access-key": "YOUR_AWS_SECRET_ACCESS_KEY",
    "s3-backup-bucket": "salesx-greatify",
    "aws-region": "ap-south-1"
  }' \
  --region ap-south-1
```

2. **Update the CronJob to use SecretProviderClass** (similar to backend deployment)

## 📋 Required Secrets

### 1. salesx-backup-secrets

Used by PostgreSQL backup CronJob:

- `aws-access-key-id`: AWS Access Key ID with S3 permissions
- `aws-secret-access-key`: AWS Secret Access Key
- `s3-backup-bucket`: S3 bucket name (salesx-greatify)
- `aws-region`: AWS region (ap-south-1)

**Required IAM Permissions:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:GetObject",
        "s3:ListBucket",
        "s3:DeleteObject"
      ],
      "Resource": [
        "arn:aws:s3:::salesx-greatify/*",
        "arn:aws:s3:::salesx-greatify"
      ]
    }
  ]
}
```

### 2. salesx-backend-secrets

Backend application secrets (to be configured):

- `DJANGO_SECRET_KEY`
- `DATABASE_URL`
- `REDIS_URL`
- AWS credentials for S3 media storage
- JWT secrets
- Email credentials

### 3. pgadmin-secret

pgAdmin credentials:

- `pgadmin-email`: Admin email (e.g., admin@salesx.com)
- `pgadmin-password`: Admin password

## 🔄 Rotating Secrets

To rotate credentials:

```bash
# Update the secret
kubectl delete secret salesx-backup-secrets -n salesx-production
kubectl create secret generic salesx-backup-secrets \
  --from-literal=aws-access-key-id="NEW_KEY" \
  --from-literal=aws-secret-access-key="NEW_SECRET" \
  --from-literal=s3-backup-bucket="salesx-greatify" \
  --from-literal=aws-region="ap-south-1" \
  -n salesx-production

# Restart pods that use the secret
kubectl rollout restart cronjob/salesx-postgresql-backup -n salesx-production
```

## ✅ Verification

Check if secrets are created:

```bash
kubectl get secrets -n salesx-production
kubectl describe secret salesx-backup-secrets -n salesx-production
```

## 🚨 If You Accidentally Committed Secrets

If you accidentally committed real secrets to Git:

1. **Immediately rotate the credentials** in AWS
2. **Remove from Git history:**

```bash
# Remove the file from Git history
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch k8s/backend/base/secrets/salesx-backup-secrets.yaml" \
  --prune-empty --tag-name-filter cat -- --all

# Or use git-filter-repo (recommended)
git filter-repo --path k8s/backend/base/secrets/salesx-backup-secrets.yaml --invert-paths

# Force push (be careful!)
git push origin --force --all
```

3. **Better:** Create a new commit with placeholder values and rotate all credentials

## 📚 Best Practices

1. ✅ Use AWS Secrets Manager + CSI Driver for production
2. ✅ Use `.local` suffix for files with real credentials
3. ✅ Add secret files to `.gitignore`
4. ✅ Use different credentials for dev/staging/prod
5. ✅ Rotate credentials regularly
6. ✅ Use least privilege IAM policies
7. ✅ Enable AWS CloudTrail for audit logs

## 🔗 References

- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
- [AWS Secrets Manager](https://aws.amazon.com/secrets-manager/)
- [Secrets Store CSI Driver](https://secrets-store-csi-driver.sigs.k8s.io/)
