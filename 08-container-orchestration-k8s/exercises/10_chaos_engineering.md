# Exercise 10: Chaos Engineering

## Task
ทำการทดสอบความทนทานของระบบด้วย Chaos Engineering โดยใช้ Chaos Mesh

### Requirements
1. ติดตั้ง Chaos Mesh
2. ทำ Network Chaos Experiments
3. ทำ Pod Chaos Experiments
4. ทำ IO Chaos Experiments
5. Monitor และ Analyze Results
6. Implement Auto-remediation

## Solution

1. Chaos Mesh Installation (`chaos-mesh.yaml`):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: chaos-testing
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: chaos-mesh
  namespace: chaos-testing
spec:
  interval: 5m
  chart:
    spec:
      chart: chaos-mesh
      version: 2.5.0
      sourceRef:
        kind: HelmRepository
        name: chaos-mesh
        namespace: flux-system
  values:
    dashboard:
      create: true
```

2. Network Chaos Experiment (`network-delay.yaml`):
```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: NetworkChaos
metadata:
  name: web-delay
  namespace: chaos-testing
spec:
  action: delay
  mode: all
  selector:
    namespaces:
      - default
    labelSelectors:
      'app': 'web'
  delay:
    latency: '100ms'
    correlation: '100'
    jitter: '0ms'
  duration: '5m'
  scheduler:
    cron: '@every 10m'
```

3. Pod Failure Experiment (`pod-failure.yaml`):
```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-failure
  namespace: chaos-testing
spec:
  action: pod-failure
  mode: one
  selector:
    namespaces:
      - default
    labelSelectors:
      'app': 'web'
  duration: '30s'
  scheduler:
    cron: '@every 5m'
```

4. IO Chaos Experiment (`io-delay.yaml`):
```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: IOChaos
metadata:
  name: io-delay
  namespace: chaos-testing
spec:
  action: latency
  mode: one
  selector:
    namespaces:
      - default
    labelSelectors:
      'app': 'database'
  volumePath: /data
  path: '/data/**'
  delay: '100ms'
  percent: 100
  duration: '30s'
  scheduler:
    cron: '@every 5m'
```

5. Stress Test Experiment (`stress-test.yaml`):
```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: StressChaos
metadata:
  name: burn-cpu
  namespace: chaos-testing
spec:
  mode: one
  selector:
    namespaces:
      - default
    labelSelectors:
      'app': 'web'
  stressors:
    cpu:
      workers: 1
      load: 100
    memory:
      workers: 1
      size: '256MB'
  duration: '1m'
  scheduler:
    cron: '@every 15m'
```

6. Auto-remediation Configuration (`auto-remediation.yaml`):
```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: AutoRemediation
metadata:
  name: auto-heal
  namespace: chaos-testing
spec:
  selector:
    namespaces:
      - default
    labelSelectors:
      'app': 'web'
  remediation:
    - type: restart
      maxRetries: 3
      interval: 10s
    - type: scale
      minReplicas: 2
      maxReplicas: 5
      interval: 30s
  triggers:
    - type: probe
      probe:
        httpGet:
          path: /health
          port: 8080
        periodSeconds: 10
        failureThreshold: 3
```

### การติดตั้งและ Configuration

1. ติดตั้ง Chaos Mesh:
```bash
# Add Helm repo
helm repo add chaos-mesh https://charts.chaos-mesh.org

# Create namespace
kubectl create ns chaos-testing

# Install Chaos Mesh
helm install chaos-mesh chaos-mesh/chaos-mesh \
  --namespace=chaos-testing \
  --set dashboard.create=true
```

2. Set up Monitoring:
```bash
# Install Prometheus Operator
kubectl apply -f prometheus-operator.yaml

# Configure ServiceMonitor
kubectl apply -f chaos-mesh-monitor.yaml
```

### การทำ Experiments

1. Network Chaos:
```bash
# Apply network delay
kubectl apply -f network-delay.yaml

# Monitor effects
kubectl logs -n chaos-testing -l app=chaos-dashboard

# Check application metrics
curl http://prometheus:9090/api/v1/query?query=http_request_duration_seconds
```

2. Pod Failure:
```bash
# Start pod failure experiment
kubectl apply -f pod-failure.yaml

# Monitor pod status
watch kubectl get pods -l app=web

# Check recovery time
kubectl describe pods -l app=web
```

3. IO Chaos:
```bash
# Apply IO latency
kubectl apply -f io-delay.yaml

# Monitor database performance
kubectl exec -it $(kubectl get pod -l app=database -o jsonpath='{.items[0].metadata.name}') \
  -- mysql -e "SHOW GLOBAL STATUS LIKE 'Slow_queries'"
```

### การ Monitor และ Analysis

1. Prometheus Queries:
```bash
# Query error rate
rate(http_requests_total{status=~"5.."}[5m])

# Query latency
histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))

# Query availability
sum(up{job="web"}) / count(up{job="web"})
```

2. Grafana Dashboard:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: chaos-dashboard
  namespace: monitoring
data:
  chaos-overview.json: |
    {
      "dashboard": {
        "panels": [
          {
            "title": "Error Rate",
            "type": "graph",
            "targets": [
              {
                "expr": "rate(http_requests_total{status=~\"5..\"}[5m])"
              }
            ]
          },
          {
            "title": "Latency",
            "type": "graph",
            "targets": [
              {
                "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))"
              }
            ]
          }
        ]
      }
    }
```

### การ Analyze Results

1. Collect Metrics:
```bash
# Export metrics
kubectl exec -n monitoring prometheus-k8s-0 -- curl -G http://localhost:9090/api/v1/query \
  --data-urlencode 'query=rate(http_requests_total{status=~"5.."}[1h])' > error_rates.json

# Analyze with jq
cat error_rates.json | jq '.data.result[] | {metric: .metric, value: .value[1]}'
```

2. Generate Report:
```bash
#!/bin/bash
echo "Chaos Engineering Report" > report.md
echo "======================" >> report.md
echo "" >> report.md

# Add error rates
echo "## Error Rates" >> report.md
kubectl exec -n monitoring prometheus-k8s-0 -- curl -G http://localhost:9090/api/v1/query \
  --data-urlencode 'query=rate(http_requests_total{status=~"5.."}[1h])' | \
  jq -r '.data.result[] | "* \(.metric.service): \(.value[1])"' >> report.md

# Add latency data
echo "" >> report.md
echo "## Latency (95th percentile)" >> report.md
kubectl exec -n monitoring prometheus-k8s-0 -- curl -G http://localhost:9090/api/v1/query \
  --data-urlencode 'query=histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[1h]))' | \
  jq -r '.data.result[] | "* \(.metric.service): \(.value[1])ms"' >> report.md
```

### Tips
- Start with small experiments
- Monitor application SLOs
- Document all findings
- Implement circuit breakers
- Use canary deployments
- Regular chaos experiments
- Automate remediation
