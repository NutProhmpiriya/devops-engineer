# Kubernetes Monitoring and Observability

## Monitoring Stack
1. **Prometheus + Grafana**
```yaml
# Prometheus ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-monitor
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
```

2. **Metrics Collection**
   - Node Metrics
   - Pod Metrics
   - Application Metrics
   
   > 🇹🇭 การเก็บ Metrics ที่สำคัญเพื่อดูสถานะของระบบ

## Logging
1. **EFK Stack**
   - Elasticsearch
   - Fluentd/Fluent Bit
   - Kibana
   
   > 🇹🇭 Stack มาตรฐานสำหรับการจัดการ Log

2. **Log Aggregation**
```yaml
# Fluent Bit ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluent-bit-config
data:
  fluent-bit.conf: |
    [SERVICE]
        Flush         1
        Log_Level     info
        Daemon        off
        
    [INPUT]
        Name             tail
        Path             /var/log/containers/*.log
        Parser           docker
        Tag              kube.*
        
    [OUTPUT]
        Name             es
        Match            kube.*
        Host             elasticsearch
        Port             9200
        Index            kubernetes_cluster
```

## Tracing
1. **Jaeger/Zipkin**
   - Distributed Tracing
   - Request Flow
   - Performance Analysis
   
   > 🇹🇭 การติดตามการทำงานของระบบแบบ Distributed

2. **OpenTelemetry**
```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: OpenTelemetryCollector
metadata:
  name: demo
spec:
  config: |
    receivers:
      otlp:
        protocols:
          grpc:
          http:
    processors:
      batch:
    exporters:
      jaeger:
        endpoint: jaeger-collector:14250
        tls:
          insecure: true
```

## Alerting
1. **Alert Rules**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: prometheus-example-rules
spec:
  groups:
  - name: example
    rules:
    - alert: HighCPUUsage
      expr: node_cpu_usage_percentage > 80
      for: 5m
      labels:
        severity: warning
      annotations:
        description: "CPU usage is above 80%"
```

2. **Alert Channels**
   - Email
   - Slack
   - PagerDuty
   
   > 🇹🇭 การแจ้งเตือนไปยังช่องทางต่างๆ เมื่อเกิดปัญหา

## Dashboards
1. **Grafana Dashboards**
   - System Overview
   - Application Metrics
   - Custom Metrics
   
   > 🇹🇭 การแสดงผลข้อมูลในรูปแบบที่เข้าใจง่าย

2. **Custom Metrics**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: custom-metrics-apiserver
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8443"
spec:
  ports:
  - port: 443
    targetPort: 8443
  selector:
    app: custom-metrics-apiserver
```

## Performance Analysis
1. **Resource Metrics**
   - CPU Usage
   - Memory Usage
   - Network I/O
   
   > 🇹🇭 การวิเคราะห์การใช้ทรัพยากรของระบบ

2. **Application Metrics**
   - Response Time
   - Error Rate
   - Request Rate
   
   > 🇹🇭 การวัดประสิทธิภาพของแอปพลิเคชัน

## Cost Monitoring
1. **Resource Usage**
   - Namespace Usage
   - Pod Usage
   - Node Usage
   
   > 🇹🇭 การติดตามการใช้ทรัพยากรเพื่อควบคุมค่าใช้จ่าย

2. **Cost Allocation**
   - Team/Project Based
   - Environment Based
   - Service Based
   
   > 🇹🇭 การแบ่งค่าใช้จ่ายตามการใช้งานจริง
