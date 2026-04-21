# EventXGames Infrastructure

This repository contains Infrastructure as Code (IaC) for the EventXGames AWS environment.

## Contents

```
nclouds-infrastructure/
├── terraform/
│   ├── modules/
│   │   ├── eks/          # EKS cluster configuration
│   │   ├── aurora/       # Aurora PostgreSQL setup
│   │   ├── elasticache/  # ElastiCache Redis cluster
│   │   ├── s3-cdn/       # S3 + CloudFront
│   │   └── networking/   # VPC, subnets, security groups
│   ├── environments/
│   │   ├── dev/
│   │   ├── staging/
│   │   └── prod/
│   └── main.tf
├── kubernetes/
│   ├── base/             # Base configurations
│   ├── overlays/         # Environment-specific overlays
│   └── apps/             # Application manifests
└── docs/
    └── DEPLOYMENT.md
```

## Terraform Modules

### EKS Module
- Managed node groups (system, application, spot)
- AWS VPC CNI
- Cluster autoscaler
- AWS Load Balancer Controller

### Aurora Module  
- Multi-AZ PostgreSQL 15.4 cluster
- Writer + 2 reader instances
- Automated backups (35-day retention)
- Performance Insights enabled

### ElastiCache Module
- Redis 7.1 cluster mode
- 3 shards with 2 replicas
- Transit encryption enabled
- Auth token from Secrets Manager

### S3 + CloudFront Module
- Versioned S3 bucket
- CloudFront with OAC
- WAF integration
- Cross-region replication for DR

## Environments

### Production VPS (Current)
- IP: 40.90.168.38
- app.eventxgames.com
- app-api.eventxgames.com

### Staging VPS - dev-v26 (NEW)
- IP: 172.188.98.210
- dev-app-v26.eventxgames.com
- dev-api-v26.eventxgames.com
- dev-ws-v26.eventxgames.com

---

## Quick Start

```bash
# Initialize Terraform
cd terraform/environments/dev
terraform init

# Plan changes
terraform plan -out=tfplan

# Apply changes
terraform apply tfplan
```

## Kubernetes Deployments

Application manifests use Kustomize for environment overlays:

```bash
# Deploy to staging
kubectl apply -k kubernetes/overlays/staging

# Deploy to production
kubectl apply -k kubernetes/overlays/prod
```

## Quick Links

- [AWS Architecture](https://github.com/eventxgames-nclouds/nclouds-aws-architecture)
- [Security Documentation](https://github.com/eventxgames-nclouds/nclouds-security)
- [Main Project Index](https://github.com/eventxgames-nclouds/nclouds-overview)
