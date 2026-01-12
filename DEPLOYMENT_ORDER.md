# Deployment Order Guide

This guide explains the correct order to deploy SalesX components to avoid dependency issues.

## 📋 Deployment Overview

Some resources must be created manually before applying kustomization to avoid exposing secrets in Git.

## 🚀 Step-by-Step Deployment

### Step 1: Create Namespace

```bash
kubectl create namespace salesx-production
```

### Step 2: Apply Storage (if using EFS)

```bash
# Only if you need EFS for media files
kubectl apply -f k8s/backend/base/storage/efs-storageclass.yaml

# Update the fileSystemId in the storage class first!
```

### Step 3: Create Secrets Manually

**IMPORTANT:** Never commit real secrets to Git!

#### A. Backup Secrets

```bash
kubectl create secret generic salesx-backup-secrets \
  --from-literal=aws-access-key-id="YOUR_AWS_ACCESS_KEY_ID" \
  --from-literal=aws-secret-access-key="YOUR_AWS_SECRET_ACCESS_KEY" \
  --from-literal=s3-backup-bucket="salesx-greatify" \
  --from-literal=aws-region="ap-south-1" \
  -n salesx-production
```

#### B. pgAdmin Secrets

```bash
kubectl create secret generic pgadmin-secret \
  --from-literal=pgadmin-email="admin@salesx.com" \
  --from-literal=pgadmin-password="YOUR_SECURE_PASSWORD" \
  -n salesx-production
```

#### C. Backend Secrets (when ready)

```bash
kubectl create secret generic salesx-backend-secrets \
  --from-literal=DJANGO_SECRET_KEY="YOUR_DJANGO_SECRET_KEY" \
  --from-literal=DATABASE_URL="postgresql://salesx:PASSWORD@salesx-postgresql:5432/salesx" \
  --from-literal=REDIS_URL="redis://salesx-redis:6379/0" \
  --from-literal=CACHE_URL="redis://salesx-redis:6379/1" \
  --from-literal=AWS_ACCESS_KEY_ID="YOUR_AWS_KEY" \
  --from-literal=AWS_SECRET_ACCESS_KEY="YOUR_AWS_SECRET" \
  --from-literal=AWS_STORAGE_BUCKET_NAME="your-bucket" \
  --from-literal=AWS_S3_REGION_NAME="ap-south-1" \
  -n salesx-production
```

### Step 4: Apply Base Infrastructure with Kustomize

```bash
cd /Users/hariharenrs/Documents/01-Active-Projects/Web-Applications/sales/SalesX-Deployment

# Apply base configuration (PostgreSQL, Redis, pgAdmin)
kubectl apply -k k8s/backend/base -n salesx-production
```

This will create:
- ✅ PostgreSQL StatefulSet and Services
- ✅ Redis StatefulSet and Services
- ✅ pgAdmin Deployment and Service
- ✅ Nginx ConfigMaps
- ✅ Media PVC (if using EFS)
- ✅ Backend, Celery deployments (will be pending until secrets exist)
- ✅ HPAs for autoscaling

### Step 5: Apply CronJobs (After Secrets)

Only after backup secrets are created:

```bash
# PostgreSQL backup CronJob
kubectl apply -f k8s/backend/base/cronjobs/postgresql-backup-cronjob.yaml -n salesx-production

# Redis cleanup CronJob
kubectl apply -f k8s/backend/base/cronjobs/redis-cleanup-cronjob.yaml -n salesx-production
```

### Step 6: Verify Deployment

```bash
# Check all resources
kubectl get all -n salesx-production

# Check secrets
kubectl get secrets -n salesx-production

# Check pods
kubectl get pods -n salesx-production

# Check services
kubectl get svc -n salesx-production

# Check CronJobs
kubectl get cronjobs -n salesx-production
```

## 🔍 Troubleshooting

### Pods are Pending

```bash
# Check events
kubectl describe pod <pod-name> -n salesx-production

# Check logs
kubectl logs <pod-name> -n salesx-production
```

### Secrets Missing

```bash
# List secrets
kubectl get secrets -n salesx-production

# Describe secret to verify keys (won't show values)
kubectl describe secret salesx-backup-secrets -n salesx-production
```

### Database Connection Issues

```bash
# Connect to PostgreSQL
kubectl exec -it salesx-postgresql-0 -n salesx-production -- psql -U salesx -d salesx

# Check database
\l
\dt
```

## 📦 Individual Component Deployment

If you prefer to deploy components individually:

### PostgreSQL Only

```bash
kubectl apply -f k8s/backend/base/services/postgresql-service.yaml -n salesx-production
kubectl apply -f k8s/backend/base/deployment/postgresql-statefulset.yaml -n salesx-production
```

### Redis Only

```bash
kubectl apply -f k8s/backend/base/services/redis-service.yaml -n salesx-production
kubectl apply -f k8s/backend/base/deployment/redis-statefulset.yaml -n salesx-production
```

### pgAdmin Only

```bash
kubectl create secret generic pgadmin-secret \
  --from-literal=pgadmin-email="admin@salesx.com" \
  --from-literal=pgadmin-password="admin123" \
  -n salesx-production

kubectl apply -f k8s/backend/base/configmap/pgadmin-config.yaml -n salesx-production
kubectl apply -f k8s/backend/base/deployment/pgadmin-deployment.yaml -n salesx-production
kubectl apply -f k8s/backend/base/services/pgadmin-service.yaml -n salesx-production
```

## 🔄 Update Deployment

To update an existing deployment:

```bash
# Update via kustomize
kubectl apply -k k8s/backend/base -n salesx-production

# Or update specific resources
kubectl apply -f k8s/backend/base/deployment/django-backend-deployment.yaml -n salesx-production

# Restart deployments to pick up changes
kubectl rollout restart deployment/salesx-django-backend -n salesx-production
```

## 🗑️ Cleanup

To remove everything:

```bash
# Delete all resources
kubectl delete -k k8s/backend/base -n salesx-production

# Delete CronJobs
kubectl delete -f k8s/backend/base/cronjobs/ -n salesx-production

# Delete namespace (removes everything)
kubectl delete namespace salesx-production
```

## ⚠️ Important Notes

1. **Never commit real secrets** - Use placeholder values in Git
2. **Create secrets manually** - Use `kubectl create secret` or AWS Secrets Manager
3. **Apply CronJobs after secrets** - They depend on backup secrets
4. **Check pod status** - Ensure all pods are running before proceeding
5. **Use production overlay** - For production, use `kubectl apply -k k8s/backend/overlays/prod`

## 📚 Related Documentation

- [SECRETS_SETUP.md](SECRETS_SETUP.md) - Detailed secrets management guide
- [README.md](README.md) - Complete project documentation
- [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
