# EventXGames Performance Baseline Metrics

**Document Version:** 1.0  
**Last Updated:** 2026-04-21  
**Status:** Baseline Targets (Pre-Migration)

---

## Executive Summary

This document establishes the performance baseline targets for the EventXGames platform migration from Azure to AWS. These targets are derived from AWS Well-Architected Framework performance efficiency pillar recommendations and industry best practices for gaming platforms.

---

## 1. Latency Targets

### 1.1 API Response Time SLOs

| Metric | Target | Critical Threshold | Measurement |
|--------|--------|-------------------|-------------|
| **P50 (Median)** | < 100ms | < 150ms | 50th percentile of all API requests |
| **P95** | < 500ms | < 750ms | 95th percentile of all API requests |
| **P99** | < 1000ms | < 1500ms | 99th percentile of all API requests |
| **P99.9** | < 2000ms | < 3000ms | 99.9th percentile (tail latency) |

### 1.2 Latency by Endpoint Category

| Endpoint Category | P50 Target | P95 Target | Notes |
|------------------|------------|------------|-------|
| Health/Status | < 10ms | < 50ms | Must be ultra-fast for load balancer health checks |
| User Profile | < 100ms | < 300ms | Includes Redis cache lookup |
| Game Listings | < 80ms | < 250ms | Paginated, cached responses |
| Leaderboards | < 50ms | < 150ms | Redis sorted sets, highly optimized |
| Search | < 150ms | < 400ms | Elasticsearch/OpenSearch backed |
| Game State Save | < 100ms | < 300ms | Write to Aurora + Redis |
| Asset Delivery (CDN) | < 50ms | < 100ms | CloudFront edge cached |

### 1.3 WebSocket Metrics

| Metric | Target | Critical Threshold |
|--------|--------|-------------------|
| Connection Time | < 2s | < 5s |
| Message Latency (P95) | < 100ms | < 200ms |
| Reconnection Time | < 3s | < 5s |

---

## 2. Throughput Targets

### 2.1 API Throughput

| Metric | Baseline Capacity | Burst Capacity | Notes |
|--------|------------------|----------------|-------|
| **Requests/Second** | 5,000 | 10,000 | Sustained load capacity |
| **Concurrent Users** | 10,000 | 20,000 | Active authenticated users |
| **Peak Hour Requests** | 15M/hour | 30M/hour | Expected peak traffic |

### 2.2 Game Session Capacity

| Metric | Baseline | Peak | Notes |
|--------|----------|------|-------|
| **Concurrent Sessions** | 5,000 | 10,000 | Active game sessions |
| **Session Creates/min** | 500 | 1,000 | New session creation rate |
| **State Updates/sec** | 10,000 | 25,000 | Game state save operations |

### 2.3 WebSocket Capacity

| Metric | Baseline | Peak | Notes |
|--------|----------|------|-------|
| **Concurrent Connections** | 50,000 | 100,000 | Active WebSocket connections |
| **Messages/Second** | 100,000 | 250,000 | Aggregate message throughput |
| **Connections/Second** | 500 | 1,000 | New connection rate |

### 2.4 Asset Delivery

| Metric | Baseline | Burst | Notes |
|--------|----------|-------|-------|
| **Requests/Second** | 10,000 | 50,000 | CloudFront edge requests |
| **Bandwidth** | 5 Gbps | 20 Gbps | Peak transfer rate |
| **Cache Hit Ratio** | > 85% | > 80% | During normal operation |

---

## 3. Error Rate Targets

### 3.1 HTTP Error Rates

| Error Category | Target | Critical Threshold | Action |
|---------------|--------|-------------------|--------|
| **5xx Errors** | < 0.1% | < 0.5% | Server-side errors requiring investigation |
| **4xx Errors** | < 2% | < 5% | Client errors (expected for auth/validation) |
| **Timeout Errors** | < 0.05% | < 0.1% | Request timeouts |
| **Connection Errors** | < 0.01% | < 0.05% | TCP connection failures |

### 3.2 WebSocket Error Rates

| Error Type | Target | Critical Threshold |
|-----------|--------|-------------------|
| Connection Failures | < 1% | < 3% |
| Abnormal Closures | < 0.5% | < 1% |
| Message Delivery Failures | < 0.1% | < 0.5% |

---

## 4. Database Performance Metrics

### 4.1 Aurora PostgreSQL

| Metric | Target | Critical Threshold | Notes |
|--------|--------|-------------------|-------|
| **Query Latency (P95)** | < 50ms | < 100ms | Simple queries |
| **Complex Query (P95)** | < 200ms | < 500ms | Joins, aggregations |
| **Connection Utilization** | < 70% | < 85% | Connection pool usage |
| **CPU Utilization** | < 60% | < 80% | Writer instance |
| **Read IOPS** | < 30,000 | < 40,000 | Peak read operations |
| **Write IOPS** | < 10,000 | < 15,000 | Peak write operations |
| **Replication Lag** | < 20ms | < 100ms | Primary to replica |

### 4.2 ElastiCache Redis

| Metric | Target | Critical Threshold | Notes |
|--------|--------|-------------------|-------|
| **GET Latency (P99)** | < 2ms | < 5ms | Cache read |
| **SET Latency (P99)** | < 3ms | < 10ms | Cache write |
| **Memory Utilization** | < 70% | < 85% | Per node |
| **CPU Utilization** | < 65% | < 80% | Per node |
| **Cache Hit Ratio** | > 90% | > 85% | Application cache |
| **Evictions/min** | < 100 | < 500 | Memory pressure indicator |
| **Connection Count** | < 5,000 | < 8,000 | Per cluster |

---

## 5. Infrastructure Metrics

### 5.1 EKS Cluster

| Metric | Target | Critical Threshold | Notes |
|--------|--------|-------------------|-------|
| **Pod CPU (avg)** | < 60% | < 80% | Across all pods |
| **Pod Memory (avg)** | < 70% | < 85% | Across all pods |
| **Node CPU (avg)** | < 65% | < 80% | Cluster average |
| **Node Memory (avg)** | < 75% | < 85% | Cluster average |
| **Pod Restart Rate** | < 1/hour | < 5/hour | Per deployment |
| **HPA Scale Events** | < 10/hour | < 20/hour | Normal operation |

### 5.2 Load Balancer (ALB)

| Metric | Target | Critical Threshold |
|--------|--------|-------------------|
| **Request Count** | Monitored | Alert on 2x baseline |
| **Target Response Time** | < 200ms | < 500ms |
| **Healthy Host Count** | 100% | > 80% |
| **HTTP 5xx Count** | < 10/min | < 50/min |

### 5.3 CloudFront

| Metric | Target | Critical Threshold |
|--------|--------|-------------------|
| **Cache Hit Ratio** | > 85% | > 75% |
| **Error Rate** | < 0.1% | < 0.5% |
| **Origin Latency** | < 200ms | < 500ms |
| **Bytes Downloaded** | Monitored | Alert on anomalies |

---

## 6. Test Scenarios Summary

### 6.1 API Load Test

```
Scenario: api_load
VUs: 0 -> 1000 (60s ramp) -> 1000 (5m sustain) -> 1500 (spike) -> 0
Duration: 10 minutes total
Success Criteria:
  - P50 < 100ms
  - P95 < 500ms
  - P99 < 1000ms
  - Error Rate < 1%
```

### 6.2 Game Session Test

```
Scenario: game_sessions
VUs: 0 -> 500 (2m ramp) -> 500 (10m sustain) -> 750 (spike) -> 0
Duration: 21 minutes total
Success Criteria:
  - Session Create < 2s (P95)
  - State Update < 200ms (P95)
  - Session End < 1s (P95)
  - Error Rate < 2%
```

### 6.3 Asset Loading Test

```
Scenario: asset_burst
Rate: 500 req/s (5m) -> 333 req/s burst (30s, ~10K total) -> 200 req/s (2m)
Duration: 7.5 minutes total
Success Criteria:
  - P50 < 50ms (cached)
  - P95 < 200ms
  - P99 < 500ms
  - Error Rate < 0.1%
  - Cache Hit > 85%
```

### 6.4 WebSocket Test

```
Scenario: websocket_load
VUs: 0 -> 1000 -> 2500 -> 5000 (sustain) -> 6000 (spike) -> 0
Duration: 23 minutes total
Success Criteria:
  - Connect Time < 5s (P95)
  - Message Latency < 100ms (P95)
  - Error Rate < 2%
  - Connection Error Rate < 5%
```

---

## 7. Baseline Establishment Process

### 7.1 Pre-Test Checklist

- [ ] All EKS workloads deployed and healthy
- [ ] Aurora PostgreSQL cluster operational with replicas
- [ ] ElastiCache Redis cluster operational with replicas
- [ ] CloudFront distribution active and verified
- [ ] Monitoring dashboards configured
- [ ] Alert thresholds set (but not alerting during test)
- [ ] Test data seeded in databases

### 7.2 Test Execution Order

1. **Warm-up Phase** (15 min)
   - Run light load (10% of baseline) to warm caches
   - Verify all systems responding normally

2. **API Load Test** (10 min)
   - Execute api-load-test.js
   - Monitor database and cache metrics

3. **Cool-down** (5 min)
   - Allow systems to stabilize

4. **Game Session Test** (21 min)
   - Execute game-session-test.js
   - Focus on Redis and Aurora write performance

5. **Cool-down** (5 min)

6. **Asset Loading Test** (7.5 min)
   - Execute asset-loading-test.js
   - Monitor CloudFront and S3 metrics

7. **Cool-down** (5 min)

8. **WebSocket Test** (23 min)
   - Execute websocket-test.js
   - Monitor connection counts and message throughput

9. **Combined Load Test** (30 min) - Optional
   - Run all tests simultaneously at 50% intensity
   - Validates system under mixed workload

### 7.3 Results Collection

After each test, collect:
- k6 summary output (JSON format)
- CloudWatch metrics export
- Database Performance Insights data
- CloudFront access logs sample
- Any error logs or alerts triggered

---

## 8. Acceptance Criteria

### 8.1 Must Pass

| Criteria | Requirement |
|----------|-------------|
| API P95 Latency | < 500ms |
| Error Rate | < 1% |
| Zero Downtime | No service interruptions |
| Database Stability | No connection pool exhaustion |
| Cache Effectiveness | Hit ratio > 85% |

### 8.2 Should Pass

| Criteria | Requirement |
|----------|-------------|
| API P50 Latency | < 100ms |
| WebSocket P95 Latency | < 100ms |
| Asset P95 Latency | < 200ms |
| Auto-scaling | Responds within 2 minutes |

### 8.3 Nice to Have

| Criteria | Requirement |
|----------|-------------|
| API P99 Latency | < 800ms |
| Zero 5xx Errors | During sustained load |
| Linear Scaling | Throughput scales with resources |

---

## 9. Appendix: Metric Collection Queries

### CloudWatch Insights Query Examples

```sql
-- API Latency Distribution
fields @timestamp, @message
| filter @logStream like /api-worker/
| stats avg(duration) as avg_latency,
        percentile(duration, 50) as p50,
        percentile(duration, 95) as p95,
        percentile(duration, 99) as p99
  by bin(1m)

-- Error Rate by Endpoint
fields @timestamp, status_code, endpoint
| filter status_code >= 500
| stats count(*) as error_count by endpoint
| sort error_count desc
| limit 20

-- WebSocket Connection Metrics
fields @timestamp, event_type, connection_count
| filter @logStream like /chat-worker/
| stats max(connection_count) as peak_connections,
        avg(connection_count) as avg_connections
  by bin(5m)
```

---

*Document maintained by: CTO Agent*  
*Review frequency: After each load test iteration*
