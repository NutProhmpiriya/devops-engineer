# Exercise 11: Advanced Service Mesh Patterns

## Task
Implement advanced service mesh patterns รวมถึง Multi-cluster Service Mesh, Advanced Traffic Management และ Observability

### Requirements
1. Set up Multi-cluster Istio Mesh
2. Implement Advanced Traffic Routing
3. Configure Cross-cluster Load Balancing
4. Set up End-to-end mTLS
5. Implement Advanced Observability
6. Configure Service Level Objectives (SLOs)

## Solution

1. Multi-cluster Istio Configuration (`istio-multicluster.yaml`):
```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: istio-control-plane
spec:
  profile: default
  values:
    global:
      meshID: mesh1
      multiCluster:
        clusterName: cluster1
      network: network1
    gateways:
      istio-ingressgateway:
        serviceAnnotations:
          service.beta.kubernetes.io/aws-load-balancer-type: nlb
---
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: istio-remote
spec:
  profile: remote
  values:
    global:
      meshID: mesh1
      multiCluster:
        clusterName: cluster2
      network: network2
```

2. Advanced Traffic Management (`traffic-management.yaml`):
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: complex-routing
spec:
  hosts:
  - "api.example.com"
  gateways:
  - api-gateway
  http:
  - match:
    - headers:
        user-type:
          exact: premium
      uri:
        prefix: /api/v1
    route:
    - destination:
        host: premium-api
        subset: v2
      weight: 90
    - destination:
        host: premium-api
        subset: v3
      weight: 10
  - match:
    - headers:
        user-region:
          exact: asia
    route:
    - destination:
        host: api-service
        subset: asia-v1
  - route:
    - destination:
        host: api-service
        subset: default-v1
---
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: api-versions
spec:
  host: api-service
  trafficPolicy:
    loadBalancer:
      consistentHash:
        httpCookie:
          name: user_session
          ttl: 3600s
  subsets:
  - name: v1
    labels:
      version: v1
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 100
        http:
          http1MaxPendingRequests: 1
          maxRequestsPerConnection: 10
      outlierDetection:
        consecutive5xxErrors: 3
        interval: 30s
        baseEjectionTime: 30s
```

3. Cross-cluster Load Balancing (`cross-cluster-lb.yaml`):
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: ServiceEntry
metadata:
  name: cross-cluster-api
spec:
  hosts:
  - api.global
  location: MESH_INTERNAL
  ports:
  - number: 80
    name: http
    protocol: HTTP
  resolution: DNS
  endpoints:
  - address: cluster1-api.example.com
    network: network1
    ports:
      http: 80
    weight: 50
  - address: cluster2-api.example.com
    network: network2
    ports:
      http: 80
    weight: 50
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: global-routing
spec:
  hosts:
  - api.global
  http:
  - route:
    - destination:
        host: api.global
    retries:
      attempts: 3
      perTryTimeout: 2s
    timeout: 10s
```

4. Advanced Observability Configuration (`observability.yaml`):
```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: mesh-telemetry
spec:
  tracing:
    - providers:
      - name: jaeger
      randomSamplingPercentage: 100.0
    customTags:
      user_id:
        header:
          name: x-user-id
      environment:
        literal:
          value: production
  metrics:
    - providers:
      - name: prometheus
      overrides:
      - match:
          metric: REQUEST_DURATION
          mode: CLIENT_AND_SERVER
        tagOverrides:
          response_code:
            operation: UPSERT
          request_operation:
            operation: UPSERT
```

5. Service Level Objectives (`slo.yaml`):
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: slo-rules
spec:
  groups:
  - name: availability.rules
    rules:
    - record: availability:success_rate_5m
      expr: |
        sum(rate(istio_requests_total{response_code!~"5.*"}[5m])) 
        / 
        sum(rate(istio_requests_total[5m]))
    - alert: AvailabilitySLOViolation
      expr: |
        avg_over_time(availability:success_rate_5m[1h]) < 0.995
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: Service availability below 99.5%
  - name: latency.rules
    rules:
    - record: latency:p99_5m
      expr: |
        histogram_quantile(0.99, 
          sum(rate(istio_request_duration_milliseconds_bucket[5m])) by (le))
    - alert: LatencySLOViolation
      expr: latency:p99_5m > 500
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: P99 latency above 500ms
```

### การติดตั้งและ Configuration

1. ติดตั้ง Multi-cluster Istio:
```bash
# Install Istio in primary cluster
istioctl install -f istio-multicluster.yaml --context=cluster1-context

# Install Istio in remote cluster
istioctl install -f istio-remote.yaml --context=cluster2-context

# Generate remote secrets
istioctl x create-remote-secret \
  --context=cluster1-context \
  --name=cluster1 | \
  kubectl apply -f - --context=cluster2-context
```

2. Configure Cross-cluster Communication:
```bash
# Set up east-west gateway
samples/multicluster/gen-eastwest-gateway.sh \
  --mesh mesh1 --cluster cluster1 --network network1 | \
  istioctl install -f - --context=cluster1-context

# Expose services
kubectl --context=cluster1-context apply -f \
  samples/multicluster/expose-services.yaml
```

3. Set up Observability Stack:
```bash
# Install monitoring stack
kubectl apply -f monitoring/

# Configure Grafana dashboards
kubectl apply -f grafana-dashboards/

# Set up SLO monitoring
kubectl apply -f slo.yaml
```

### การทดสอบ

1. Test Traffic Routing:
```bash
# Test premium user routing
curl -H "user-type: premium" http://api.example.com/api/v1/test

# Test regional routing
curl -H "user-region: asia" http://api.example.com/api/v2/test

# Test session affinity
curl -b "user_session=123" http://api.example.com/api/v3/test
```

2. Test Cross-cluster Functionality:
```bash
# Deploy test service
kubectl apply -f test-service.yaml --context=cluster1-context
kubectl apply -f test-service.yaml --context=cluster2-context

# Test cross-cluster calls
for i in {1..100}; do
  curl http://api.global/test
done
```

3. Verify Observability:
```bash
# Check distributed tracing
istioctl dashboard jaeger

# View metrics
istioctl dashboard grafana

# Check SLO compliance
kubectl get prometheusrules
```

### การ Monitor

1. Set up Custom Dashboards:
```bash
# Import custom dashboard
curl -X POST http://grafana:3000/api/dashboards/import \
  -H "Content-Type: application/json" \
  -d @custom-dashboard.json
```

2. Configure Alerts:
```bash
# Set up alert manager
kubectl apply -f alert-manager-config.yaml

# Configure notification channels
kubectl apply -f notification-config.yaml
```

### Tips
- ใช้ Network Policies ร่วมกับ Service Mesh
- Monitor control plane performance
- Implement proper retry policies
- Use circuit breakers effectively
- Regular testing of failover scenarios
- Keep service mesh configuration version controlled
- Monitor mesh performance metrics
