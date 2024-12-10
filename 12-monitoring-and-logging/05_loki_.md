**Loki** เป็นระบบ **Log Aggregation** แบบโอเพนซอร์สที่พัฒนาโดยทีมงานเดียวกับ Grafana เพื่อช่วยรวบรวมและจัดการ **Logs** จากแหล่งข้อมูลต่าง ๆ Loki ถูกออกแบบมาให้ทำงานได้ง่ายและประหยัดทรัพยากรเมื่อเทียบกับระบบ Log อื่น ๆ เช่น ELK Stack (Elasticsearch, Logstash, Kibana)

---

### **คุณสมบัติเด่นของ Loki**
1. **เรียบง่ายและเบา**:
   - Loki ไม่ทำการจัดทำดัชนี (Indexing) Logs โดยใช้ **Labels** แทน ซึ่งช่วยลดการใช้งานทรัพยากร

2. **ผสานการทำงานกับ Grafana**:
   - Loki สามารถแสดง Logs ใน **Grafana Dashboard** ได้ ทำให้สามารถดู Logs และ Metrics ในที่เดียวกัน

3. **Label-Based**:
   - Loki จัดระเบียบ Logs ด้วย **Labels** เช่น `job`, `instance`, หรือ `region`

4. **รองรับ Multi-Tenant**:
   - Loki รองรับการจัดการ Logs สำหรับหลายองค์กรหรือทีมในระบบเดียว

5. **Scalable**:
   - Loki สามารถขยายตัวได้ง่ายทั้งในรูปแบบ Standalone และ Cluster

6. **Push-Based Logging**:
   - สามารถรับ Logs ผ่าน **Push Gateway** โดยใช้ **Promtail**, **Fluentd**, **Logstash** หรือ **Custom Agents**

---

### **ส่วนประกอบหลักของ Loki**
1. **Loki Server**:
   - เก็บและจัดการ Logs ในรูปแบบ Time-Series โดยใช้ Labels

2. **Promtail**:
   - ตัวเก็บรวบรวม Logs (Log Collector) ที่ดึง Logs จากไฟล์หรือระบบต่าง ๆ แล้วส่งไปยัง Loki

3. **Grafana**:
   - ใช้แสดง Logs จาก Loki พร้อมกับ Metrics หรือ Dashboard อื่น ๆ

4. **Client Libraries**:
   - ใช้ส่ง Logs จากแอปพลิเคชันไปยัง Loki

---

### **การติดตั้ง Loki**
#### ใช้ Docker-Compose:
สร้างไฟล์ `docker-compose.yml`:
```yaml
version: "3.7"
services:
  loki:
    image: grafana/loki:2.8.0
    ports:
      - "3100:3100"
    command: -config.file=/etc/loki/local-config.yaml
    volumes:
      - ./loki-config.yaml:/etc/loki/local-config.yaml

  promtail:
    image: grafana/promtail:2.8.0
    volumes:
      - /var/log:/var/log
      - ./promtail-config.yaml:/etc/promtail/config.yaml
    command: -config.file=/etc/promtail/config.yaml
```

---

#### ติดตั้งผ่าน Helm (Kubernetes):
1. เพิ่ม Helm Repository:
   ```bash
   helm repo add grafana https://grafana.github.io/helm-charts
   ```
2. ติดตั้ง Loki Stack:
   ```bash
   helm install loki grafana/loki-stack
   ```

---

### **การตั้งค่า Loki**
#### ตัวอย่างไฟล์ `loki-config.yaml`:
```yaml
auth_enabled: false
server:
  http_listen_port: 3100

ingester:
  lifecycler:
    ring:
      kvstore:
        store: inmemory
  chunk_idle_period: 5m

schema_config:
  configs:
    - from: 2020-10-24
      store: boltdb
      object_store: filesystem
      schema: v11
      index:
        prefix: index_
        period: 24h

storage_config:
  boltdb:
    directory: /loki/index
  filesystem:
    directory: /loki/chunks

limits_config:
  retention_period: 24h
```

---

### **การเก็บ Logs ด้วย Promtail**
#### ตัวอย่างไฟล์ `promtail-config.yaml`:
```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push

scrape_configs:
  - job_name: "system-logs"
    static_configs:
      - targets:
          - localhost
        labels:
          job: "varlogs"
          __path__: /var/log/*.log
```

---

### **การดู Logs ใน Grafana**
1. เพิ่ม Data Source:
   - เลือก **Loki**
   - ใส่ URL เช่น `http://localhost:3100`
2. สร้าง Dashboard:
   - ไปที่ **Explore**
   - เลือก Data Source เป็น Loki
   - Query Logs เช่น:
     ```logql
     {job="varlogs"}
     ```
   - ใช้ Filter เช่น `|="error"`

---

### **การใช้งาน LogQL (Loki Query Language)**
#### ตัวอย่างคำสั่ง:
1. **ดึง Logs ทั้งหมด**:
   ```logql
   {job="varlogs"}
   ```
2. **ค้นหา Logs ที่มีคำว่า "error"**:
   ```logql
   {job="varlogs"} |= "error"
   ```
3. **นับจำนวน Logs ต่อวินาที**:
   ```logql
   rate({job="varlogs"}[1m])
   ```
4. **กรอง Logs ด้วย Regex**:
   ```logql
   {job="varlogs"} |~ "failed.*connection"
   ```

---

### **การใช้งานในระบบ DevOps**
1. **Monitoring Applications**:
   - รวบรวม Logs จาก Container หรือ Pod ใน Kubernetes
   - ใช้ Logs ร่วมกับ Metrics ใน Grafana เพื่อวิเคราะห์ปัญหา
2. **Debugging & Troubleshooting**:
   - ค้นหา Errors หรือ Events ที่เกิดขึ้นในระบบ
3. **Compliance & Auditing**:
   - เก็บ Logs สำหรับการตรวจสอบความปลอดภัยและการทำงานของระบบ

---

### **ข้อดีของ Loki**
- ประหยัดทรัพยากร: ไม่ทำการจัดทำดัชนี (Index)
- ผสานการทำงานกับ Grafana ได้อย่างราบรื่น
- ติดตั้งและใช้งานง่าย
- รองรับระบบขนาดใหญ่และการใช้งาน Multi-Tenant

### **ข้อจำกัด**
- การค้นหา Logs อาจช้ากว่าเมื่อเทียบกับระบบที่ใช้ Index เช่น Elasticsearch
- ไม่เหมาะสำหรับการ Query Logs แบบซับซ้อนมาก

---

