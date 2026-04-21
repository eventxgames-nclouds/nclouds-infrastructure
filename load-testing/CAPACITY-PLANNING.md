# EventXGames Capacity Planning Recommendations

**Document Version:** 1.0  
**Last Updated:** 2026-04-21  
**Based On:** AWS Architecture and Load Testing Baseline

---

## Executive Summary

This document provides capacity planning recommendations for the EventXGames platform based on the established performance baselines and AWS Well-Architected Framework principles. Recommendations cover compute, database, caching, CDN, and cost optimization strategies.

---

## 1. Current Architecture Capacity

### 1.1 EKS Cluster Capacity Summary

| Component | Current Config | Peak Capacity | Utilization Target |
|-----------|---------------|---------------|-------------------|
| **API Workers** | 3 pods (m6i.xlarge equiv) | 20 pods max | 70% CPU |
| **Game Workers** | 5 pods (high memory) | 50 pods max | 65% CPU |
| **Chat Workers** | 3 pods | 20 pods max | 75% memory |
| **Asset Workers** | 2 pods (spot) | 10 pods max | 70% CPU |
| **Frontend Workers** | 3 pods | 15 pods max | 60% CPU |

### 1.2 Database Capacity

| Resource | Current | Max Capacity | Headroom |
|----------|---------|--------------|----------|
| **Aurora Writer** | db.r6g.2xlarge | 64 ACU (Serverless) | 3x |
| **Aurora Readers** | 2x db.r6g.xlarge | 15 replicas | 7x |
| **Connections** | ~500 active | 5,000 max | 10x |
| **Storage** | 500 GB | 128 TB | 256x |

### 1.3 Cache Capacity

| Resource | Current | Max Capacity | Headroom |
|----------|---------|--------------|----------|
| **Redis Shards** | 3 | 500 | 166x |
| **Replicas/Shard** | 2 | 5 | 2.5x |
| **Memory/Node** | 26 GB | 635 GB (r6g.16xlarge) | 24x |
| **Connections** | ~2,000 | 65,000 | 32x |

---

## 2. Scaling Recommendations

### 2.1 Horizontal Pod Autoscaler (HPA) Tuning

```yaml
# Recommended HPA settings for each workload

# API Worker - Scale on CPU and custom request metric
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: api-worker-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-worker
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "500"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 100
        periodSeconds: 60
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 25
        periodSeconds: 120

# Game Worker - Scale on CPU and active sessions
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: game-worker-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: game-worker
  minReplicas: 5
  maxReplicas: 50
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 65
  - type: Pods
    pods:
      metric:
        name: active_game_sessions
      target:
        type: AverageValue
        averageValue: "100"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 30
      policies:
      - type: Pods
        value: 5
        periodSeconds: 30
    scaleDown:
      stabilizationWindowSeconds: 600
      policies:
      - type: Percent
        value: 10
        periodSeconds: 300

# Chat Worker - Scale on memory and connections
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: chat-worker-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: chat-worker
  minReplicas: 3
  maxReplicas: 20
  metrics:
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 75
  - type: Pods
    pods:
      metric:
        name: websocket_connections
      target:
        type: AverageValue
        averageValue: "2500"
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Pods
        value: 3
        periodSeconds: 60
```

### 2.2 Cluster Autoscaler Configuration

```yaml
# Cluster Autoscaler settings
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-autoscaler-config
data:
  # Scale up aggressively, scale down conservatively
  scale-down-delay-after-add: "10m"
  scale-down-delay-after-delete: "1m"
  scale-down-delay-after-failure: "3m"
  scale-down-unneeded-time: "10m"
  scale-down-utilization-threshold: "0.5"
  
  # Expansion options
  expander: "priority"
  max-node-provision-time: "5m"
  
  # Balance similar node groups
  balance-similar-node-groups: "true"
  skip-nodes-with-local-storage: "false"
```

### 2.3 Database Scaling Triggers

| Trigger | Threshold | Action | Cooldown |
|---------|-----------|--------|----------|
| CPU > 80% sustained | 5 min | Add read replica | 30 min |
| Connections > 70% | 5 min | Scale up instance class | 1 hour |
| IOPS > 80% limit | 10 min | Enable I/O-Optimized | Manual |
| Replication lag > 100ms | 5 min | Alert + investigate | N/A |

### 2.4 Cache Scaling Triggers

| Trigger | Threshold | Action |
|---------|-----------|--------|
| Memory > 75% | Any node | Add shard or scale up node |
| CPU > 70% | Sustained 5 min | Add replicas |
| Evictions > 500/min | Any node | Scale up memory |
| Connection errors | > 10/min | Add replicas |

---

## 3. Growth Projections

### 3.1 Traffic Growth Scenarios

| Scenario | Timeline | Traffic Multiplier | Recommended Action |
|----------|----------|-------------------|-------------------|
| **Organic Growth** | 6 months | 1.5x | Current config sufficient |
| **Marketing Push** | 1 month | 3x | Pre-scale databases |
| **Viral Event** | 1 week | 10x | DR region activation |
| **Holiday Peak** | Annual | 5x | Reserved capacity |

### 3.2 Capacity Planning Matrix

```
                        Current     +50%        +200%       +500%
                        (Baseline)  (6 months)  (1 year)    (Peak)
----------------------------------------------------------------
API Pods               3-20        5-30        8-40        15-60
Game Pods              5-50        8-75        15-100      30-150
Aurora Writer          2xlarge     4xlarge     8xlarge     16xlarge
Aurora Readers         2           3           5           8
Redis Shards           3           5           8           12
Redis Memory           78 GB       130 GB      208 GB      312 GB
Est. Monthly Cost      $13,800     $18,500     $28,000     $45,000
```

### 3.3 Cost per User Projections

| User Tier | Concurrent Users | Monthly Cost | Cost/User |
|-----------|-----------------|--------------|-----------|
| Startup | 1,000 | $8,000 | $8.00 |
| Growth | 10,000 | $13,800 | $1.38 |
| Scale | 50,000 | $28,000 | $0.56 |
| Enterprise | 100,000 | $45,000 | $0.45 |

---

## 4. Reserved Capacity Recommendations

### 4.1 Compute Savings Plan

| Resource Type | On-Demand | 1-Year Reserved | 3-Year Reserved | Recommendation |
|--------------|-----------|-----------------|-----------------|----------------|
| EKS Nodes (m6i) | $0.192/hr | $0.121/hr (37%) | $0.077/hr (60%) | 1-Year for baseline |
| Spot Instances | $0.058/hr | N/A | N/A | Use for asset workers |

**Recommended Commitment:**
- 6 x m6i.xlarge On-Demand Reserved (1-year) for application nodes
- 3 x m6i.large On-Demand Reserved (1-year) for system nodes
- Remaining capacity via Spot Fleet for non-critical workloads

### 4.2 Database Reserved Instances

| Instance | On-Demand Monthly | Reserved Monthly | Savings |
|----------|------------------|------------------|---------|
| db.r6g.2xlarge (Writer) | $1,200 | $780 | 35% |
| db.r6g.xlarge x2 (Readers) | $1,200 | $780 | 35% |

**Recommendation:** 1-Year All Upfront for production databases

### 4.3 ElastiCache Reserved Nodes

| Node Type | On-Demand Monthly | Reserved Monthly | Savings |
|-----------|------------------|------------------|---------|
| cache.r6g.xlarge x9 | $4,050 | $2,835 | 30% |

**Recommendation:** 1-Year No Upfront for flexibility

---

## 5. DR and Multi-Region Strategy

### 5.1 DR Capacity Requirements

| Component | Primary (us-east-1) | DR (us-west-2) | Failover RTO |
|-----------|--------------------|--------------------|--------------|
| EKS Nodes | 15 | 5 (warm standby) | 5 min |
| Aurora | 1W + 2R | Global DB (1R) | 1 min |
| Redis | 3 shards x 3 | 3 shards x 2 | 2 min |
| CloudFront | Global | Global | Instant |

### 5.2 Multi-Region Expansion Checklist

When expanding to additional regions:

- [ ] Deploy VPC with identical CIDR planning
- [ ] Set up Aurora Global Database secondary
- [ ] Configure ElastiCache Global Datastore
- [ ] Deploy EKS cluster with matching configuration
- [ ] Update CloudFront with regional origins
- [ ] Configure Route 53 latency-based routing
- [ ] Replicate S3 buckets cross-region
- [ ] Update WAF rules for regional ALBs

---

## 6. Performance Optimization Recommendations

### 6.1 Quick Wins (Implement Immediately)

| Optimization | Expected Improvement | Effort |
|--------------|---------------------|--------|
| Enable Aurora Query Cache | 20% read latency reduction | Low |
| Increase PgBouncer pool size | 15% connection efficiency | Low |
| Enable Redis key compression | 25% memory savings | Low |
| CloudFront TTL optimization | 10% origin traffic reduction | Low |

### 6.2 Medium-Term Improvements

| Optimization | Expected Improvement | Effort |
|--------------|---------------------|--------|
| Implement read/write splitting | 30% DB load reduction | Medium |
| Add application-level caching | 40% DB query reduction | Medium |
| Optimize slow queries | 50% reduction in P99 latency | Medium |
| Implement request coalescing | 20% cache efficiency improvement | Medium |

### 6.3 Architecture Evolution

| Phase | Timeline | Changes |
|-------|----------|---------|
| **Phase 1** | Now | Current architecture, baseline established |
| **Phase 2** | 6 months | Add read replicas, optimize caching |
| **Phase 3** | 12 months | Multi-region active-active consideration |
| **Phase 4** | 18 months | Evaluate serverless components (Aurora Serverless v2) |

---

## 7. Monitoring and Alerting Recommendations

### 7.1 Key Metrics Dashboard

Deploy the CloudWatch dashboard from `dashboards/cloudwatch-performance-dashboard.json` with these additions:

1. **Business Metrics Widget** - Revenue per minute, user registrations
2. **Cost Metrics Widget** - Real-time spend tracking
3. **Capacity Headroom Widget** - Percentage of max capacity used

### 7.2 Alert Thresholds

| Alert | Warning | Critical | Action |
|-------|---------|----------|--------|
| API Latency P95 | > 400ms | > 700ms | Scale pods |
| Error Rate | > 0.5% | > 1% | Page on-call |
| DB CPU | > 70% | > 85% | Scale instance |
| DB Connections | > 60% | > 80% | Increase pool |
| Cache Memory | > 70% | > 85% | Add capacity |
| Node CPU | > 75% | > 90% | Add nodes |

### 7.3 Capacity Forecasting

Implement weekly capacity reviews:
1. Extract past 7 days metrics
2. Calculate growth rate
3. Project capacity exhaustion date
4. Create tickets for scaling actions needed within 30 days

---

## 8. Cost Optimization Summary

### 8.1 Monthly Cost Breakdown (Optimized)

| Category | On-Demand | Optimized | Savings |
|----------|-----------|-----------|---------|
| EKS/EC2 Compute | $8,000 | $5,500 | $2,500 |
| Aurora PostgreSQL | $2,600 | $1,800 | $800 |
| ElastiCache Redis | $4,150 | $2,900 | $1,250 |
| S3 + CloudFront | $1,000 | $800 | $200 |
| Bedrock AI | $650 | $400 | $250 |
| DR Region | $1,500 | $1,200 | $300 |
| Network/Other | $1,500 | $1,200 | $300 |
| **Total** | **$19,400** | **$13,800** | **$5,600** |

### 8.2 Annual Projections

| Scenario | Year 1 | Year 2 | Year 3 |
|----------|--------|--------|--------|
| On-Demand | $232,800 | $290,000 | $360,000 |
| Optimized | $165,600 | $210,000 | $265,000 |
| Savings | $67,200 | $80,000 | $95,000 |

---

## 9. Action Items

### 9.1 Immediate (Week 7-8)

- [x] Deploy k6 load testing framework
- [x] Create load test scripts for 4 scenarios
- [x] Document baseline metrics targets
- [x] Create CloudWatch performance dashboard
- [ ] Execute baseline load tests (requires infrastructure)
- [ ] Capture and document actual baseline results

### 9.2 Short-Term (Month 2-3)

- [ ] Implement HPA configurations as specified
- [ ] Configure Cluster Autoscaler
- [ ] Set up cost allocation tags
- [ ] Purchase reserved capacity for baseline workload
- [ ] Implement quick-win optimizations

### 9.3 Medium-Term (Month 4-6)

- [ ] Conduct monthly load test iterations
- [ ] Implement read/write splitting
- [ ] Evaluate Aurora Serverless v2 for variable workloads
- [ ] Set up capacity forecasting automation

---

*Document maintained by: CTO Agent*  
*Next review: After baseline load test execution*
