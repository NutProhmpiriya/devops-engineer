# Exercise 5: Monitoring Setup with Prometheus and Grafana

## Task
ติดตั้งและ configure ระบบ monitoring ด้วย Prometheus และ Grafana

### Requirements
1. ติดตั้ง Prometheus Operator
2. Configure ServiceMonitor
3. ติดตั้ง Grafana
4. สร้าง Dashboard
5. ตั้งค่า Alerts

## Solution

1. สร้าง Namespace (`monitoring-namespace.yaml`):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: monitoring
```

2. สร้าง Prometheus (`prometheus-deployment.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - name: prometheus
        image: prom/prometheus:v2.30.3
        ports:
        - containerPort: 9090
        volumeMounts:
        - name: config
          mountPath: /etc/prometheus/
      volumes:
      - name: config
        configMap:
          name: prometheus-config
```

3. สร้าง Prometheus ConfigMap (`prometheus-config.yaml`):
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: monitoring
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
    scrape_configs:
      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
        - role: pod
        relabel_configs:
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
          action: keep
          regex: true
```

4. สร้าง Grafana (`grafana-deployment.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana
  namespace: monitoring
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
  template:
    metadata:
      labels:
        app: grafana
    spec:
      containers:
      - name: grafana
        image: grafana/grafana:8.2.0
        ports:
        - containerPort: 3000
        env:
        - name: GF_SECURITY_ADMIN_PASSWORD
          value: "admin"
        - name: GF_USERS_ALLOW_SIGN_UP
          value: "false"
```

5. สร้าง Services (`monitoring-services.yaml`):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: prometheus
  namespace: monitoring
spec:
  selector:
    app: prometheus
  ports:
  - port: 9090
    targetPort: 9090
---
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: monitoring
spec:
  selector:
    app: grafana
  ports:
  - port: 3000
    targetPort: 3000
  type: LoadBalancer
```

### คำสั่งที่ใช้
```bash
# สร้าง namespace
kubectl apply -f monitoring-namespace.yaml

# ติดตั้ง Prometheus
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus-deployment.yaml

# ติดตั้ง Grafana
kubectl apply -f grafana-deployment.yaml

# สร้าง Services
kubectl apply -f monitoring-services.yaml

# ตรวจสอบสถานะ
kubectl get all -n monitoring
```

### การตั้งค่า Grafana
1. เข้าถึง Grafana UI:
```bash
# Get Grafana URL
kubectl get svc grafana -n monitoring
```

2. Login ด้วย:
- Username: admin
- Password: admin

3. เพิ่ม Prometheus Data Source:
- URL: http://prometheus:9090
- Access: Server (default)

4. Import Dashboard:
- Import dashboard ID 315 (Kubernetes Cluster Monitoring)
- Select Prometheus data source

### การตั้งค่า Alerts
1. สร้าง Alert Rule ใน Prometheus (`alert-rules.yaml`):
```yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: example-alert
  namespace: monitoring
spec:
  groups:
  - name: example
    rules:
    - alert: HighCPUUsage
      expr: container_cpu_usage_seconds_total > 0.8
      for: 5m
      labels:
        severity: warning
      annotations:
        description: "Container CPU usage is above 80%"
```

### Tips
- ตรวจสอบ logs ของ Prometheus และ Grafana
- ใช้ port-forward ถ้าไม่มี LoadBalancer
- สำรอง Grafana dashboards
- ตั้งค่า retention period ของ Prometheus ตามความเหมาะสม
