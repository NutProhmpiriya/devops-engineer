**Grafana** เป็นเครื่องมือ **โอเพนซอร์ส** ที่ใช้สำหรับการสร้าง **Dashboard** และ **Visualization** เพื่อแสดงข้อมูลจากระบบ Monitoring, Logs, หรือ Time-Series Database ได้อย่างชัดเจนและเข้าใจง่าย โดย Grafana สามารถทำงานร่วมกับระบบ Monitoring เช่น Prometheus, Elasticsearch, InfluxDB และ MySQL ได้อย่างมีประสิทธิภาพ

---

### **คุณสมบัติเด่นของ Grafana**
1. **Dashboard Visualization**:
   - สร้างกราฟหรือแดชบอร์ดที่สวยงาม และปรับแต่งได้หลากหลายรูปแบบ เช่น กราฟเส้น, ตาราง, เกจวัด, Heatmap ฯลฯ

2. **รองรับ Data Sources หลากหลาย**:
   - Prometheus
   - InfluxDB
   - Elasticsearch
   - MySQL / PostgreSQL
   - Loki (สำหรับ Logs)
   - และอื่น ๆ อีกมากมาย

3. **Alerting**:
   - ตั้งค่าแจ้งเตือนเมื่อข้อมูลหรือ Metrics มีค่าผิดปกติ เช่น แจ้งเตือนผ่าน Email, Slack, PagerDuty

4. **Plugins และ Integration**:
   - เพิ่ม Plugin สำหรับ Data Sources และ Panel ได้ เช่น Worldmap, Pie Chart
   - ทำงานร่วมกับ Kubernetes, Docker, และระบบ DevOps อื่น ๆ ได้ดี

5. **Role-Based Access Control (RBAC)**:
   - กำหนดสิทธิ์การเข้าถึง Dashboard ระหว่างผู้ใช้ในทีม

6. **Templating และ Variables**:
   - ใช้ตัวแปร (Variables) เพื่อสร้าง Dashboard แบบ Dynamic ที่สามารถกรองข้อมูลได้ตามต้องการ

7. **Custom Query และ Transformation**:
   - รองรับการเขียน Query แบบปรับแต่งเอง และ Transform ข้อมูลก่อนแสดงผล

---

### **การใช้งาน Grafana**
#### 1. **ติดตั้ง Grafana**
- **ใช้ Docker (วิธีง่ายที่สุด):**
   ```bash
   docker run -d -p 3000:3000 grafana/grafana
   ```
- **ใช้ Helm (สำหรับ Kubernetes):**
   ```bash
   helm repo add grafana https://grafana.github.io/helm-charts
   helm install grafana grafana/grafana
   ```

#### 2. **เข้าสู่ระบบ Grafana**
- เปิดเบราว์เซอร์แล้วไปที่: [http://localhost:3000](http://localhost:3000)
- ชื่อผู้ใช้เริ่มต้น: `admin`
- รหัสผ่านเริ่มต้น: `admin` (ระบบจะให้เปลี่ยนรหัสทันทีหลังเข้าสู่ระบบครั้งแรก)

---

### **เพิ่ม Data Source**
1. เข้าสู่ **Configuration > Data Sources**
2. เลือก Data Source ที่ต้องการ เช่น:
   - **Prometheus**: ใส่ URL เช่น `http://localhost:9090`
   - **MySQL**: กรอก Host, Database, User, และ Password
3. คลิก **Save & Test**

---

### **สร้าง Dashboard**
1. ไปที่ **Create > Dashboard**
2. เพิ่ม **Panel**:
   - เลือกรูปแบบกราฟ (เช่น Time Series, Table)
   - เขียน Query ดึงข้อมูลจาก Data Source เช่น:
     - **Prometheus**: 
       ```promql
       rate(http_requests_total[5m])
       ```
     - **MySQL**:
       ```sql
       SELECT time, value FROM metrics WHERE metric_name = 'cpu_usage';
       ```
3. ปรับแต่ง Panel (เช่น เปลี่ยนชื่อ, ใส่สี, หรือกำหนด Threshold)
4. บันทึก Dashboard

---

### **ตั้งค่าแจ้งเตือน (Alerting)**
1. เลือก Panel ที่ต้องการตั้งค่า Alert
2. ไปที่ **Alert > Create Alert Rule**
3. กำหนดเงื่อนไข (Condition):
   - เช่น ถ้าค่า **CPU Usage** เกิน 80% ให้แจ้งเตือน:
     ```promql
     avg(node_cpu_seconds_total{mode="idle"}) < 20
     ```
4. เลือกช่องทางแจ้งเตือน (Notification Channel) เช่น Email หรือ Slack
5. บันทึก Alert Rule

---

### **การใช้งานขั้นสูง**
#### 1. **การใช้ Variables**
- ตัวอย่างการสร้างตัวแปร:
  - Variable: `region`
  - Query: `label_values(instance, "region")`
- ใช้ตัวแปรใน Query:
  ```promql
  node_cpu_seconds_total{region="$region"}
  ```

#### 2. **การติดตั้ง Plugins**
- ติดตั้ง Plugin ผ่านคำสั่ง:
   ```bash
   grafana-cli plugins install grafana-piechart-panel
   ```
- หรือใช้ Plugin Store ใน UI ของ Grafana

#### 3. **Multi-Tenant Support**
- สร้าง **Organizations** เพื่อแบ่ง Dashboard ตามกลุ่มผู้ใช้
- กำหนดสิทธิ์ให้กับผู้ใช้แต่ละกลุ่ม

---

### **ตัวอย่างการใช้งานใน DevOps**
1. **Monitoring Kubernetes**:
   - ใช้ Prometheus เก็บ Metrics จาก Kubernetes
   - ใช้ Grafana แสดงข้อมูล เช่น:
     - จำนวน Pod ที่ทำงานอยู่
     - CPU/Memory Usage ของ Node

2. **Logs Visualization**:
   - ใช้ Grafana Loki เก็บ Logs จากแอปพลิเคชัน
   - สร้าง Dashboard แสดง Logs และ Errors

3. **Business Metrics**:
   - แสดงผลการขายหรือการใช้งานระบบ เช่น การแสดงจำนวนผู้ใช้ที่ Active

---

### **สรุป**
Grafana เป็นเครื่องมือที่ทรงพลังและยืดหยุ่นสำหรับการสร้าง Dashboard และ Visualization ช่วยให้ทีม DevOps, SRE, และผู้ดูแลระบบสามารถติดตามสถานะของระบบและแก้ปัญหาได้รวดเร็ว หากต้องการคำแนะนำในการตั้งค่าเฉพาะทาง แจ้งมาได้เลยครับ! 😊