# Exercise 13: Serverless with Knative

## Task
สร้าง Serverless Platform ด้วย Knative พร้อมทั้ง implement Event-driven Architecture

### Requirements
1. ติดตั้ง Knative Serving และ Eventing
2. Configure Auto-scaling
3. Set up Event Sources
4. Implement Event Brokers
5. Create Event-driven Workflows
6. Configure Cold Start Optimization

## Solution

1. Knative Installation (`knative-install.yaml`):
```yaml
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeServing
metadata:
  name: knative-serving
  namespace: knative-serving
spec:
  version: "1.0.0"
  ingress:
    kourier:
      enabled: true
---
apiVersion: operator.knative.dev/v1alpha1
kind: KnativeEventing
metadata:
  name: knative-eventing
  namespace: knative-eventing
spec:
  version: "1.0.0"
  source:
    kafka:
      enabled: true
```

2. Serverless Service (`service.yaml`):
```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: event-processor
  namespace: default
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/class: "kpa.autoscaling.knative.dev"
        autoscaling.knative.dev/metric: "concurrency"
        autoscaling.knative.dev/target: "100"
        autoscaling.knative.dev/minScale: "1"
        autoscaling.knative.dev/maxScale: "10"
    spec:
      containers:
      - image: event-processor:latest
        resources:
          limits:
            memory: 512Mi
            cpu: "1"
          requests:
            memory: 256Mi
            cpu: "0.5"
        env:
        - name: SCALE_TARGET
          value: "100"
```

3. Event Source Configuration (`event-source.yaml`):
```yaml
apiVersion: sources.knative.dev/v1
kind: KafkaSource
metadata:
  name: kafka-source
spec:
  consumerGroup: knative-group
  bootstrapServers:
  - my-cluster-kafka-bootstrap.kafka:9092
  topics:
  - orders
  sink:
    ref:
      apiVersion: eventing.knative.dev/v1
      kind: Broker
      name: default
---
apiVersion: sources.knative.dev/v1
kind: PingSource
metadata:
  name: test-ping-source
spec:
  schedule: "*/1 * * * *"
  data: '{"message": "Hello world!"}'
  sink:
    ref:
      apiVersion: eventing.knative.dev/v1
      kind: Broker
      name: default
```

4. Event Broker and Trigger (`event-routing.yaml`):
```yaml
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  name: default
  namespace: default
spec:
  config:
    apiVersion: v1
    kind: ConfigMap
    name: config-br-default-channel
    namespace: knative-eventing
---
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: order-processor
spec:
  broker: default
  filter:
    attributes:
      type: order.created
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: order-processor
---
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: notification-sender
spec:
  broker: default
  filter:
    attributes:
      type: order.processed
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: notification-service
```

5. Cold Start Optimization (`optimization.yaml`):
```yaml
apiVersion: serving.knative.dev/v1
kind: Configuration
metadata:
  name: event-processor
spec:
  template:
    metadata:
      annotations:
        # Keep 1 replica always running
        autoscaling.knative.dev/minScale: "1"
        # Pre-pull images
        scheduler.alpha.kubernetes.io/priorityClassName: "high-priority"
    spec:
      containers:
      - image: event-processor:latest
        # Add readiness probe
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 1
          periodSeconds: 2
        # Add startup probe
        startupProbe:
          httpGet:
            path: /startup
            port: 8080
          failureThreshold: 30
          periodSeconds: 10
        resources:
          limits:
            memory: 512Mi
            cpu: "1"
          requests:
            memory: 256Mi
            cpu: "0.5"
```

6. Monitoring Configuration (`monitoring.yaml`):
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: knative-monitoring
  namespace: monitoring
spec:
  selector:
    matchLabels:
      serving.knative.dev/service: event-processor
  endpoints:
  - port: metrics
    interval: 30s
---
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: knative-alerts
spec:
  groups:
  - name: knative.rules
    rules:
    - alert: HighLatency
      expr: |
        histogram_quantile(0.95, 
          sum(rate(knative_serving_revision_request_latencies_bucket[5m])) 
          by (le, configuration_name)
        ) > 1000
      for: 5m
      labels:
        severity: warning
      annotations:
        summary: High latency for service {{ $labels.configuration_name }}
```

### การติดตั้งและ Configuration

1. ติดตั้ง Knative:
```bash
# Install Knative Operator
kubectl apply -f https://github.com/knative/operator/releases/download/knative-v1.0.0/operator.yaml

# Install Serving and Eventing
kubectl apply -f knative-install.yaml

# Verify installation
kubectl get pods -n knative-serving
kubectl get pods -n knative-eventing
```

2. Configure DNS:
```bash
# Configure Magic DNS
kubectl apply -f https://github.com/knative/serving/releases/download/v1.0.0/serving-default-domain.yaml

# Verify domain configuration
kubectl get ksvc
```

3. Set up Event Sources:
```bash
# Install Kafka source
kubectl apply -f event-source.yaml

# Verify source installation
kubectl get kafkasource
kubectl get pingsource
```

### การทดสอบ

1. Deploy Service:
```bash
# Deploy event processor
kubectl apply -f service.yaml

# Test scaling
hey -z 30s -c 50 http://event-processor.default.example.com
```

2. Test Event Flow:
```bash
# Send test event
curl -v "http://broker-ingress.knative-eventing.svc.cluster.local/default/default" \
  -X POST \
  -H "Ce-Id: 123456789" \
  -H "Ce-Specversion: 1.0" \
  -H "Ce-Type: order.created" \
  -H "Ce-Source: test" \
  -H "Content-Type: application/json" \
  -d '{"orderId": "12345"}'

# Check logs
kubectl logs -l serving.knative.dev/service=event-processor
```

3. Monitor Performance:
```bash
# Check scaling metrics
kubectl get pod -l serving.knative.dev/service=event-processor -w

# View metrics
kubectl port-forward -n monitoring svc/prometheus-k8s 9090
```

### การ Monitor

1. Set up Metrics:
```bash
# Configure Prometheus
kubectl apply -f monitoring.yaml

# Set up Grafana dashboard
kubectl apply -f knative-dashboard.yaml
```

2. View Logs:
```bash
# Stream logs
stern event-processor -n default

# Check event delivery
kubectl logs -l eventing.knative.dev/broker=default -c broker-filter
```

### Tips
- Use proper resource limits
- Implement graceful shutdown
- Monitor cold start latency
- Use event batching when possible
- Implement proper error handling
- Regular testing of auto-scaling
- Monitor event delivery success rate
