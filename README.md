# SalesX Platform Deployment

This repository contains the Kubernetes deployment configurations for the SalesX platform, including both the Backend API (Django + Celery + Redis) and Frontend (React/Vite) services.

## 🏗️ Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    AWS EKS Cluster                          │
│                                                             │
│  ┌──────────────────────┐  ┌─────────────────────────────┐ │
│  │   Backend Services   │  │   Frontend Services         │ │
│  │                      │  │                             │ │
│  │  - Django API        │  │  - React App (Nginx)        │ │
│  │  - Celery Worker     │  │  - Autoscaling (2-10)       │ │
│  │  - Celery Beat       │  │                             │ │
│  │  - Redis Cache       │  └─────────────────────────────┘ │
│  │  - Autoscaling       │                                   │
│  └──────────────────────┘                                   │
│           │                          │                      │
│           ▼                          ▼                      │
│  ┌──────────────────────┐                                   │
│  │  ALB Ingress         │                                   │
│  │  (klockwork.ai)      │                                   │
│  │  /api → Backend      │                                   │
│  │  / → Frontend        │                                   │
│  └──────────────────────┘                                   │
└─────────────────────────────────────────────────────────────┘
                    │                    │
         ┌──────────┴────────┬───────────┴──────────┐
         ▼                   ▼                       ▼
   ┌──────────┐        ┌──────────┐          ┌──────────┐
   │   RDS    │        │   ECR    │          │ Secrets  │
   │PostgreSQL│        │ Registry │          │ Manager  │
   └──────────┘        └──────────┘          └──────────┘
```

## 📦 Services

### Backend Services
- **Django API**: RESTful API server with health checks and autoscaling (2-10 replicas)
- **Celery Worker**: Async task processing with queue-based autoscaling (2-8 replicas)
- **Celery Beat**: Scheduled task scheduler (1 replica, singleton)
- **Redis**: Message broker and cache with persistence (1-3 replicas)

### Frontend Service
- **React/Vite App**: Single-page application served via Nginx with autoscaling (2-10 replicas)

### Mobile Apps
- **React Native (Expo)**: iOS and Android applications
- **Build via GitHub Actions**: Automated APK/IPA builds with public download links

## 🗂️ Repository Structure

```
SalesX-Deployment/
├── .github/
│   └── workflows/
│       ├── backend-deploy.yaml       # Backend deployment pipeline
│       └── frontend-deploy.yaml      # Frontend deployment pipeline
├── k8s/
│   ├── backend/
│   │   ├── base/                     # Base Kubernetes manifests
│   │   │   ├── deployment/           # Deployment configs
│   │   │   │   ├── django-backend-deployment.yaml
│   │   │   │   ├── celery-worker-deployment.yaml
│   │   │   │   ├── celery-beat-deployment.yaml
│   │   │   │   └── redis-deployment.yaml
│   │   │   ├── services/             # Service configs
│   │   │   │   ├── django-backend-service.yaml
│   │   │   │   └── redis-service.yaml
│   │   │   ├── ingress/              # Ingress configs
│   │   │   │   └── backend-ingress.yaml
│   │   │   ├── secrets/              # Secret templates
│   │   │   │   └── salesx-backend-secrets.yaml
│   │   │   ├── autoscaling/          # HPA configs
│   │   │   │   ├── django-backend-hpa.yaml
│   │   │   │   ├── celery-worker-hpa.yaml
│   │   │   │   └── redis-hpa.yaml
│   │   │   └── kustomization.yaml
│   │   └── overlays/
│   │       └── prod/                 # Production environment
│   │           ├── patches/          # Environment-specific patches
│   │           └── kustomization.yaml
│   └── frontend/
│       ├── base/                     # Base manifests
│       │   ├── deployment/
│       │   │   └── frontend-deployment.yaml
│       │   ├── services/
│       │   │   └── frontend-service.yaml
│       │   ├── ingress/
│       │   │   └── frontend-ingress.yaml
│       │   ├── secrets/
│       │   │   └── salesx-frontend-secrets.yaml
│       │   ├── autoscaling/
│       │   │   └── frontend-hpa.yaml
│       │   └── kustomization.yaml
│       └── overlays/
│           └── prod/                 # Production environment
│               ├── patches/
│               └── kustomization.yaml
└── README.md                         # This file
```

## 📱 Mobile App Builds

### Build Android & iOS Apps (FREE)

GitHub Actions workflow for building mobile apps with one click!

**Quick Start:**
1. Go to **Actions** → "Build Mobile Apps"
2. Click **"Run workflow"**
3. Select:
   - **Platform:** android | ios | both
   - **Build Type:** release | debug  
   - **Create Release:** ✅ (for public download links)
4. Wait ~20-25 minutes
5. Download from **Releases** tab!

**Features:**
- ✅ 100% FREE (public repo, unlimited builds)
- ✅ Android APK builds
- ✅ iOS Simulator builds
- ✅ Creates GitHub Releases with public download links
- ✅ No EAS subscription needed
- ✅ Share with anyone (no GitHub account required)

**📖 Full Guide:** [MOBILE_BUILD_GUIDE.md](./MOBILE_BUILD_GUIDE.md)

---

## 🚀 Deployment Process

### Prerequisites

1. **AWS Infrastructure**:
   - EKS cluster running and configured
   - ECR repository: `399600302704.dkr.ecr.us-east-1.amazonaws.com/salesx`
   - RDS PostgreSQL database
   - Application Load Balancer Controller installed
   - SSL certificates in ACM
   - Security groups configured

2. **GitHub Secrets**:
   - `AWS_ACCESS_KEY_ID` - AWS credentials for deployment
   - `AWS_SECRET_ACCESS_KEY` - AWS secret key
   - `AWS_REGION` - AWS region (e.g., us-east-1)
   - `EKS_CLUSTER_NAME` - Name of your EKS cluster
   - `PAT_TOKEN` - GitHub Personal Access Token for cross-repo workflows

3. **Kubernetes Secrets**:
   - Update `k8s/backend/base/secrets/salesx-backend-secrets.yaml`
   - Update `k8s/frontend/base/secrets/salesx-frontend-secrets.yaml`

### Automatic Deployment

Deployments are automatically triggered when code is pushed to the `main` branch in the SalesX repository:

1. **Trigger**: Push to `main` branch in `Greatify/SalesX`
2. **Change Detection**: Automatically detects backend/frontend changes
3. **Build**: Docker images are built and pushed to ECR
4. **Deploy**: Kubernetes manifests are updated and applied
5. **Health Check**: Deployment health is verified
6. **Rollback**: Automatic rollback on failure

### Manual Deployment

You can trigger deployments manually from GitHub Actions:

1. Go to `Greatify/SalesX-Deployment` → Actions
2. Select "Backend - Deploy to Production" or "Frontend - Deploy to Production"
3. Click "Run workflow"
4. Fill in:
   - `deploy_branch`: Branch to deploy (default: main)
   - `commit_sha`: Git commit SHA
   - `commit_message`: Commit message

### Local Deployment (for testing)

```bash
# Deploy backend to production
kubectl apply -k k8s/backend/overlays/prod

# Deploy frontend to production
kubectl apply -k k8s/frontend/overlays/prod

# Check deployment status
kubectl rollout status deployment/salesx-backend -n salesx-prod
kubectl rollout status deployment/salesx-frontend -n salesx-prod

# View pods
kubectl get pods -n salesx-prod

# View services
kubectl get svc -n salesx-prod

# View ingress
kubectl get ingress -n salesx-prod
```

## 🔧 Configuration

### Backend Configuration

**Environment Variables** (in secrets):
- `DATABASE_URL` - PostgreSQL connection string
- `REDIS_URL` - Redis connection string
- `SECRET_KEY` - Django secret key
- `JWT_SECRET_KEY` - JWT signing key
- `AWS_ACCESS_KEY_ID` - AWS credentials
- `AWS_SECRET_ACCESS_KEY` - AWS secret
- `AWS_STORAGE_BUCKET_NAME` - S3 bucket name
- `AWS_S3_REGION_NAME` - AWS region

**Resource Limits**:
- Django: 512Mi-1Gi memory, 250m-500m CPU
- Celery Worker: 512Mi-2Gi memory, 250m-1000m CPU
- Celery Beat: 256Mi-512Mi memory, 100m-250m CPU
- Redis: 256Mi-1Gi memory, 100m-500m CPU

### Frontend Configuration

**Environment Variables** (in secrets):
- `VITE_API_URL` - Backend API URL (https://klockwork.ai/api)
- `VITE_WS_URL` - WebSocket URL (wss://klockwork.ai/api/ws)

**Resource Limits**:
- Frontend: 128Mi-256Mi memory, 100m-200m CPU

### Autoscaling Configuration

**Backend Autoscaling**:
- Min replicas: 2, Max replicas: 10
- Triggers: CPU > 70%, Memory > 80%

**Celery Worker Autoscaling**:
- Min replicas: 2, Max replicas: 8
- Triggers: CPU > 75%, Memory > 85%

**Frontend Autoscaling**:
- Min replicas: 2, Max replicas: 10
- Triggers: CPU > 70%, Memory > 80%

## 🔍 Monitoring and Health Checks

### Health Endpoints

- **Backend**: `https://klockwork.ai/api/health/`
- **Frontend**: `https://klockwork.ai/`

### Monitoring Commands

```bash
# Check pod status
kubectl get pods -n salesx-prod -w

# View pod logs
kubectl logs -f deployment/salesx-backend -n salesx-prod
kubectl logs -f deployment/salesx-celery-worker -n salesx-prod
kubectl logs -f deployment/salesx-frontend -n salesx-prod

# Check HPA status
kubectl get hpa -n salesx-prod

# View events
kubectl get events -n salesx-prod --sort-by='.lastTimestamp'

# Check ingress
kubectl describe ingress -n salesx-prod

# Port forward for local testing
kubectl port-forward svc/salesx-backend 8000:80 -n salesx-prod
kubectl port-forward svc/salesx-frontend 3000:80 -n salesx-prod
```

### Deployment Status

Check deployment history:
```bash
kubectl rollout history deployment/salesx-backend -n salesx-prod
kubectl rollout history deployment/salesx-frontend -n salesx-prod
```

## 🔄 Rollback Procedures

### Automatic Rollback

The deployment workflows automatically roll back on failure.

### Manual Rollback

```bash
# Rollback backend
kubectl rollout undo deployment/salesx-backend -n salesx-prod
kubectl rollout undo deployment/salesx-celery-worker -n salesx-prod
kubectl rollout undo deployment/salesx-celery-beat -n salesx-prod

# Rollback frontend
kubectl rollout undo deployment/salesx-frontend -n salesx-prod

# Rollback to specific revision
kubectl rollout undo deployment/salesx-backend -n salesx-prod --to-revision=2

# Check rollout status
kubectl rollout status deployment/salesx-backend -n salesx-prod
```

## 💾 Backup and Maintenance

### PostgreSQL Backups

**Automatic Backups**:
- **Schedule**: Daily at 2 AM UTC
- **Location**: S3 bucket `salesx-greatify/postgresql/`
- **Retention**: 7 days (automatic deletion)
- **Storage**: EFS for database, S3 for backups

```bash
# Check backup CronJob status
kubectl get cronjob salesx-postgresql-backup -n salesx-prod

# View backup job history
kubectl get jobs -n salesx-prod | grep backup

# Manually trigger a backup
kubectl create job --from=cronjob/salesx-postgresql-backup manual-backup-$(date +%Y%m%d-%H%M%S) -n salesx-prod

# Check backup logs
kubectl logs -f job/manual-backup-YYYYMMDD-HHMMSS -n salesx-prod

# View backups in S3
aws s3 ls s3://salesx-greatify/postgresql/
```

**Restore from Backup**:

```bash
# Download backup from S3
aws s3 cp s3://salesx-greatify/postgresql/salesx-backup-YYYY-MM-DD_HH-MM-SS.sql.gz ./

# Extract backup
gunzip salesx-backup-YYYY-MM-DD_HH-MM-SS.sql.gz

# Restore to PostgreSQL
kubectl exec -i salesx-postgresql-0 -n salesx-production -- psql -U salesx -d salesx < salesx-backup-YYYY-MM-DD_HH-MM-SS.sql

# Verify restoration
kubectl exec -it salesx-postgresql-0 -n salesx-production -- psql -U salesx -d salesx -c "\dt"
```

### Redis Memory Cleanup

**Automatic Cleanup**:
- **Schedule**: Every 5 minutes
- **Action**: Remove expired keys only (safe operation)
- **Purpose**: Prevent memory issues and optimize performance

```bash
# Check Redis cleanup CronJob status
kubectl get cronjob salesx-redis-cleanup -n salesx-prod

# View cleanup job history
kubectl get jobs -n salesx-prod | grep redis-cleanup

# Manually trigger cleanup
kubectl create job --from=cronjob/salesx-redis-cleanup manual-redis-cleanup-$(date +%Y%m%d-%H%M%S) -n salesx-prod

# Check Redis memory usage
kubectl exec -it salesx-redis-0 -n salesx-prod -- redis-cli INFO memory

# Check Redis stats
kubectl exec -it salesx-redis-0 -n salesx-prod -- redis-cli INFO stats | grep -E "expired_keys|evicted_keys"
```

### EFS Storage Management

**Check EFS usage**:

```bash
# Check PVC status
kubectl get pvc -n salesx-prod

# Check PostgreSQL storage usage
kubectl exec -it salesx-postgresql-0 -n salesx-prod -- df -h /var/lib/postgresql/data

# View EFS file system in AWS
aws efs describe-file-systems --region ap-south-1
```

### Maintenance Tasks

**Database Maintenance**:

```bash
# Run vacuum analyze
kubectl exec -it salesx-postgresql-0 -n salesx-production -- psql -U salesx -d salesx -c "VACUUM ANALYZE;"

# Check database size
kubectl exec -it salesx-postgresql-0 -n salesx-production -- psql -U salesx -d salesx -c "SELECT pg_size_pretty(pg_database_size('salesx'));"

# Check table sizes
kubectl exec -it salesx-postgresql-0 -n salesx-production -- psql -U salesx -d salesx -c "SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size FROM pg_tables ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC LIMIT 10;"
```

**Redis Maintenance**:

```bash
# Check Redis info
kubectl exec -it salesx-redis-0 -n salesx-prod -- redis-cli INFO

# Get key count
kubectl exec -it salesx-redis-0 -n salesx-prod -- redis-cli DBSIZE

# Monitor Redis in real-time
kubectl exec -it salesx-redis-0 -n salesx-prod -- redis-cli --stat

# Check slow queries
kubectl exec -it salesx-redis-0 -n salesx-prod -- redis-cli SLOWLOG GET 10
```

## 🐛 Troubleshooting

### Common Issues

**1. Image Pull Errors**
```bash
# Verify ECR credentials
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 399600302704.dkr.ecr.us-east-1.amazonaws.com

# Check if image exists
aws ecr describe-images --repository-name salesx --region us-east-1
```

**2. Database Connection Issues**
```bash
# Check database secret
kubectl get secret salesx-backend-secrets -n salesx-prod -o yaml

# Test database connection from pod
kubectl exec -it deployment/salesx-backend -n salesx-prod -- python manage.py dbshell
```

**3. Pod Not Starting**
```bash
# Describe pod for events
kubectl describe pod <pod-name> -n salesx-prod

# Check logs
kubectl logs <pod-name> -n salesx-prod --previous
```

**4. Ingress Not Working**
```bash
# Check ALB controller
kubectl logs -n kube-system deployment/aws-load-balancer-controller

# Verify ingress annotations
kubectl describe ingress salesx-ingress -n salesx-prod
```

### Debug Mode

To enable debug mode temporarily:
```bash
kubectl set env deployment/salesx-backend DEBUG=True -n salesx-prod
```

## 🔐 Security

- All services run as non-root users
- Secrets are stored in Kubernetes secrets (should be externalized to AWS Secrets Manager)
- SSL/TLS termination at ALB
- Security groups restrict access
- Regular vulnerability scanning via Trivy

## 📝 Updating Secrets

**Important**: Never commit actual secrets to Git!

```bash
# Create/update backend secrets
kubectl create secret generic salesx-backend-secrets \
  --from-literal=database-url='postgresql://...' \
  --from-literal=redis-url='redis://...' \
  --from-literal=secret-key='...' \
  --from-literal=jwt-secret-key='...' \
  --from-literal=aws-access-key-id='...' \
  --from-literal=aws-secret-access-key='...' \
  --dry-run=client -o yaml | kubectl apply -f - -n salesx-prod

# Create/update frontend secrets
kubectl create secret generic salesx-frontend-secrets \
  --from-literal=api-url='https://klockwork.ai/api' \
  --from-literal=ws-url='wss://klockwork.ai/api/ws' \
  --dry-run=client -o yaml | kubectl apply -f - -n salesx-prod
```

## 🤝 Contributing

1. Create a feature branch
2. Make changes to Kubernetes manifests
3. Test locally with `kubectl apply -k k8s/[service]/overlays/prod --dry-run=client`
4. Create a pull request
5. After approval, changes will be deployed

## 📞 Support

For issues or questions:
- Check the GitHub Actions logs
- Review pod logs: `kubectl logs -f deployment/salesx-backend -n salesx-prod`
- Check Kubernetes events: `kubectl get events -n salesx-prod`

## 📄 License

Proprietary - All rights reserved

---

**Last Updated**: January 2026  
**Maintainer**: DevOps Team
