**การใช้งาน Prometheus** เริ่มต้นได้ไม่ยาก โดยมีขั้นตอนดังนี้:

---

### 1. **ติดตั้ง Prometheus**
#### ติดตั้งด้วย Docker (วิธีง่ายที่สุด):
```bash
docker run -p 9090:9090 prom/prometheus
```

#### ติดตั้งแบบ Binary:
1. ดาวน์โหลด Prometheus จาก [Prometheus Official Website](https://prometheus.io/download/).
2. แตกไฟล์และเรียกใช้:
   ```bash
   ./prometheus --config.file=prometheus.yml
   ```

#### ติดตั้งบน Kubernetes (Helm Chart):
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/prometheus
```

---

### 2. **ตั้งค่า Configuration (prometheus.yml)**

Prometheus ใช้ไฟล์ `prometheus.yml` เพื่อกำหนดเป้าหมาย (Targets) และการตั้งค่าต่าง ๆ เช่น:
```yaml
global:
  scrape_interval: 15s  # ความถี่ในการดึงข้อมูล

scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]  # Exporter Endpoint
```

---

### 3. **รัน Prometheus**
รัน Prometheus พร้อมไฟล์ Configuration:
```bash
./prometheus --config.file=prometheus.yml
```

---

### 4. **ติดตั้ง Exporters เพื่อดึง Metrics**
Exporters เป็นตัวช่วยดึง Metrics จากระบบเป้าหมาย เช่น เซิร์ฟเวอร์, ฐานข้อมูล, หรือแอปพลิเคชัน

#### ติดตั้ง Node Exporter:
```bash
docker run -d -p 9100:9100 prom/node-exporter
```

#### เพิ่ม Endpoint ใน `prometheus.yml`:
```yaml
scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]
```

---

### 5. **เปิด UI ของ Prometheus**
1. เปิดเบราว์เซอร์แล้วไปที่: [http://localhost:9090](http://localhost:9090)
2. ใช้ PromQL Query ดึงข้อมูล เช่น:
   - การดูว่า Node Exporter ทำงานอยู่หรือไม่:
     ```promql
     up
     ```
   - การดูการใช้ CPU:
     ```promql
     node_cpu_seconds_total
     ```

---

### 6. **สร้างการแจ้งเตือน (Alerting)**
#### ตัวอย่างไฟล์ Rule (`alerts.yml`):
```yaml
groups:
  - name: InstanceDown
    rules:
      - alert: InstanceDown
        expr: up == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Instance {{ $labels.instance }} is down"
          description: "The instance {{ $labels.instance }} has been down for more than 1 minute."
```

#### เพิ่มใน `prometheus.yml`:
```yaml
rule_files:
  - "alerts.yml"
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["localhost:9093"]
```

#### ติดตั้ง Alertmanager:
```bash
docker run -p 9093:9093 prom/alertmanager
```

---

### 7. **ใช้ Grafana เพื่อแสดงผล Metrics**
1. ติดตั้ง Grafana:
   ```bash
   docker run -d -p 3000:3000 grafana/grafana
   ```
2. เพิ่ม Prometheus เป็น Data Source ใน Grafana:
   - URL: `http://localhost:9090`
3. สร้าง Dashboard และใช้ Query เช่น:
   - ดู CPU Usage:
     ```promql
     node_cpu_seconds_total
     ```
   - ดู Memory Usage:
     ```promql
     node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes * 100
     ```

---

### 8. **การตรวจสอบและบำรุงรักษา**
1. **ตรวจสอบ Logs ของ Prometheus**:
   ```bash
   tail -f prometheus.log
   ```
2. **ตรวจสอบ Targets**:
   - เปิด Prometheus UI แล้วไปที่เมนู **Status > Targets**
3. **ปรับแต่ง Scrape Interval หรือ Retention**:
   ```yaml
   global:
     scrape_interval: 10s  # ลดช่วงเวลาการดึงข้อมูล
   storage.tsdb.retention.time: 15d  # เก็บข้อมูลนาน 15 วัน
   ```

---

### ตัวอย่างการใช้งานในระบบจริง
#### ใช้ใน Kubernetes:
1. ติดตั้ง Prometheus ด้วย Helm:
   ```bash
   helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
   helm install prometheus prometheus-community/kube-prometheus-stack
   ```
2. ใช้ Prometheus Monitoring สำหรับ Kubernetes:
   - ตรวจสอบ Pod Status:
     ```promql
     kube_pod_status_phase{phase="Running"}
     ```
   - ตรวจสอบ CPU ของ Pod:
     ```promql
     sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)
     ```

