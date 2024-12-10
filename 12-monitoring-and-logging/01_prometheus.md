**Prometheus** เป็นระบบ **Monitoring** และ **Alerting** แบบโอเพนซอร์สที่ได้รับความนิยมสูงในกลุ่ม DevOps และ SRE (Site Reliability Engineering) พัฒนาขึ้นโดย SoundCloud และปัจจุบันอยู่ภายใต้การดูแลของ **Cloud Native Computing Foundation (CNCF)** 

---

### **คุณสมบัติเด่นของ Prometheus**
1. **Time-Series Database**:
   - เก็บข้อมูลในรูปแบบ Time-Series (ค่าที่เปลี่ยนแปลงตามเวลา) เช่น การใช้งาน CPU, Memory, หรือ Network

2. **Pull-Based Model**:
   - Prometheus ดึงข้อมูล (Pull) จากระบบเป้าหมายผ่าน HTTP โดยใช้ **Endpoints** ที่เรียกว่า **Exporters**

3. **Multi-dimensional Data Model**:
   - ใช้ระบบ Label สำหรับการกรองและการจัดกลุ่มข้อมูล เช่น `http_requests_total{method="GET", status="200"}`

4. **PromQL (Prometheus Query Language)**:
   - ภาษา Query ที่ใช้สำหรับดึงข้อมูลจากฐานข้อมูลของ Prometheus

5. **Alerting**:
   - ใช้ร่วมกับ **Alertmanager** เพื่อแจ้งเตือนเมื่อเกิดเหตุการณ์ผิดปกติ

6. **โอเพนซอร์สและมี Ecosystem ใหญ่**:
   - รองรับการทำงานร่วมกับ Grafana, Kubernetes, Docker และระบบอื่น ๆ ได้ดี

---

### **ส่วนประกอบหลักของ Prometheus**
1. **Prometheus Server**:
   - รับผิดชอบการเก็บข้อมูล (scrape) และจัดเก็บลงใน Time-Series Database
2. **Exporters**:
   - ตัวกลางที่ช่วยให้ Prometheus ดึงข้อมูลจากระบบ เช่น **Node Exporter** สำหรับข้อมูลเซิร์ฟเวอร์, **Blackbox Exporter** สำหรับตรวจสอบ Endpoint
3. **Alertmanager**:
   - ใช้สำหรับการกำหนดและส่งการแจ้งเตือน เช่น Email, Slack, PagerDuty
4. **Pushgateway**:
   - สำหรับการส่งข้อมูลไปยัง Prometheus ในกรณีที่ระบบไม่รองรับ Pull Model
5. **Dashboarding Tools**:
   - ใช้ร่วมกับ **Grafana** เพื่อสร้างแดชบอร์ดสำหรับแสดงผลข้อมูล

---

### **ตัวอย่างการใช้งาน Prometheus**
#### 1. **ติดตั้ง Prometheus**
ดาวน์โหลดจาก [Prometheus Official Website](https://prometheus.io/download/) หรือใช้ Docker:
```bash
docker run -p 9090:9090 prom/prometheus
```

#### 2. **ตั้งค่า Config (`prometheus.yml`)**:
```yaml
global:
  scrape_interval: 15s  # ความถี่ในการดึงข้อมูล

scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets: ["localhost:9100"]  # Endpoint ของ Node Exporter
```

#### 3. **รัน Prometheus**:
```bash
./prometheus --config.file=prometheus.yml
```

#### 4. **ติดตั้ง Exporters**:
ติดตั้ง **Node Exporter** เพื่อดึงข้อมูลเกี่ยวกับเซิร์ฟเวอร์:
```bash
docker run -d -p 9100:9100 prom/node-exporter
```

#### 5. **ใช้ PromQL ดึงข้อมูล**:
ตัวอย่าง Query:
- จำนวน HTTP Requests:
  ```promql
  http_requests_total
  ```
- อัตราการร้องขอ HTTP ที่สำเร็จ:
  ```promql
  rate(http_requests_total{status="200"}[5m])
  ```

---

### **การแจ้งเตือนด้วย Alertmanager**
#### ตัวอย่าง Config สำหรับ Alerting:
```yaml
alerting:
  alertmanagers:
    - static_configs:
        - targets: ["localhost:9093"]

rule_files:
  - "alerts.yml"
```

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

---

### **การใช้งานร่วมกับ Grafana**
1. ติดตั้ง Grafana:
   ```bash
   docker run -d -p 3000:3000 grafana/grafana
   ```
2. เพิ่ม **Prometheus** เป็น Data Source ใน Grafana
3. สร้าง Dashboard และใช้ PromQL เพื่อแสดงผลข้อมูล เช่น CPU Usage, Memory Usage

---

### **การใช้งานใน Kubernetes**
Prometheus มี Integration ที่ดีมากกับ Kubernetes:
- ใช้ **kube-prometheus-stack** (Helm Chart) เพื่อติดตั้ง Prometheus, Grafana และ Alertmanager พร้อมกัน
- ตรวจสอบ Cluster Metrics เช่น:
  - Pod CPU/Memory
  - Node Disk/Network

ติดตั้งผ่าน Helm:
```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack
```

---

