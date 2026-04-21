# EventXGames Load Testing Suite

Performance load testing framework for the AWS migration project using k6.

## Quick Start

### Prerequisites

```bash
# Install k6
# macOS
brew install k6

# Linux
sudo gpg -k
sudo gpg --no-default-keyring --keyring /usr/share/keyrings/k6-archive-keyring.gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
echo "deb [signed-by=/usr/share/keyrings/k6-archive-keyring.gpg] https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
sudo apt-get update
sudo apt-get install k6

# Docker
docker pull grafana/k6
```

### Running Tests

```bash
# Set environment variables
export BASE_URL="https://api.eventxgames.com"
export CDN_URL="https://cdn.eventxgames.com"
export WS_URL="wss://ws.eventxgames.com"
export AUTH_TOKEN="your-test-auth-token"
export ENVIRONMENT="staging"

# Run individual tests
k6 run scripts/api-load-test.js
k6 run scripts/game-session-test.js
k6 run scripts/asset-loading-test.js
k6 run scripts/websocket-test.js

# Run with custom options
k6 run --vus 100 --duration 5m scripts/api-load-test.js

# Output results to JSON
k6 run --out json=results/api-test-results.json scripts/api-load-test.js
```

## Test Scenarios

### 1. API Load Test (`api-load-test.js`)

Simulates REST API traffic with 1000 concurrent users.

- **Duration:** 10 minutes
- **Peak VUs:** 1500 (with spike)
- **Endpoints:** User profiles, game listings, leaderboards, search
- **Targets:** P50 < 100ms, P95 < 500ms, P99 < 1s

### 2. Game Session Test (`game-session-test.js`)

Simulates complete game session lifecycle.

- **Duration:** 21 minutes
- **Peak Sessions:** 750 concurrent
- **Operations:** Create session, update state, record achievements, end session
- **Targets:** Session create < 2s, State update < 200ms

### 3. Asset Loading Test (`asset-loading-test.js`)

Tests CloudFront CDN and S3 origin performance.

- **Duration:** 7.5 minutes
- **Peak Rate:** 333 req/s (burst)
- **Asset Types:** Images, audio, data files, fonts, video
- **Targets:** P50 < 50ms, Cache hit > 85%

### 4. WebSocket Test (`websocket-test.js`)

Tests real-time messaging infrastructure.

- **Duration:** 23 minutes
- **Peak Connections:** 6000 concurrent
- **Operations:** Connect, chat, game state sync, presence updates
- **Targets:** Connect < 5s, Message latency < 100ms

## Directory Structure

```
load-testing/
├── README.md                    # This file
├── BASELINE-METRICS.md          # Performance targets and SLOs
├── CAPACITY-PLANNING.md         # Scaling recommendations
├── scripts/
│   ├── api-load-test.js         # API load test
│   ├── game-session-test.js     # Game session test
│   ├── asset-loading-test.js    # CDN/asset test
│   └── websocket-test.js        # WebSocket test
├── dashboards/
│   └── cloudwatch-performance-dashboard.json
└── results/                     # Test results (gitignored)
```

## Performance Targets

| Metric | P50 | P95 | P99 |
|--------|-----|-----|-----|
| API Latency | < 100ms | < 500ms | < 1000ms |
| Asset Latency | < 50ms | < 200ms | < 500ms |
| WS Message Latency | < 50ms | < 100ms | < 200ms |
| Error Rate | - | - | < 1% |

## CloudWatch Dashboard

Import the dashboard to AWS CloudWatch:

```bash
aws cloudwatch put-dashboard \
  --dashboard-name EventXGames-Performance \
  --dashboard-body file://dashboards/cloudwatch-performance-dashboard.json
```

## CI/CD Integration

Add to your pipeline:

```yaml
# GitHub Actions example
load-test:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v4
    - name: Install k6
      run: |
        sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys C5AD17C747E3415A3642D57D77C6C491D6AC1D69
        echo "deb https://dl.k6.io/deb stable main" | sudo tee /etc/apt/sources.list.d/k6.list
        sudo apt-get update && sudo apt-get install k6
    - name: Run API Load Test
      run: k6 run --out json=results.json scripts/api-load-test.js
      env:
        BASE_URL: ${{ secrets.STAGING_API_URL }}
        AUTH_TOKEN: ${{ secrets.LOAD_TEST_TOKEN }}
```

## Grafana Integration

For real-time visualization, run k6 with Prometheus output:

```bash
k6 run --out experimental-prometheus-rw scripts/api-load-test.js
```

Or use InfluxDB:

```bash
k6 run --out influxdb=http://localhost:8086/k6 scripts/api-load-test.js
```

## Troubleshooting

### High Error Rates
- Check target service health endpoints
- Verify AUTH_TOKEN is valid
- Ensure network connectivity to target

### Slow Performance
- Verify k6 is running from a location near the target
- Check for rate limiting on target API
- Consider running from EC2 in same region

### Connection Refused
- Verify URLs are correct
- Check security groups allow traffic from test runner
- Ensure services are running

## References

- [k6 Documentation](https://k6.io/docs/)
- [AWS Well-Architected Framework - Performance](https://docs.aws.amazon.com/wellarchitected/latest/performance-efficiency-pillar/)
- [EventXGames AWS Architecture](../docs/architecture/AWS_ARCHITECTURE.md)
