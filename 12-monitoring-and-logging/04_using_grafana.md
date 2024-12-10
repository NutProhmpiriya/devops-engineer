**การใช้งาน Grafana** สามารถเริ่มต้นได้ง่าย ๆ ด้วยขั้นตอนเหล่านี้:

---

### **1. ติดตั้ง Grafana**
#### ใช้ Docker:
```bash
docker run -d -p 3000:3000 grafana/grafana
```

#### ใช้ Helm (สำหรับ Kubernetes):
```bash
helm repo add grafana https://grafana.github.io/helm-charts
helm install grafana grafana/grafana
```

#### ใช้ Binary:
1. ดาวน์โหลดจาก [Grafana Official Website](https://grafana.com/grafana/download).
2. แตกไฟล์และรัน:
   ```bash
   ./bin/grafana-server web
   ```

---

### **2. เข้าสู่ระบบ Grafana**
1. เปิดเบราว์เซอร์และเข้าไปที่: [http://localhost:3000](http://localhost:3000)
2. เข้าสู่ระบบ:
   - **Username**: `admin`
   - **Password**: `admin` (เปลี่ยนรหัสผ่านทันทีหลังเข้าสู่ระบบครั้งแรก)

---

### **3. เพิ่ม Data Source**
1. ไปที่ **Configuration > Data Sources**
2. คลิก **Add Data Source**
3. เลือกประเภท Data Source เช่น:
   - **Prometheus**: กรอก URL เช่น `http://localhost:9090`
   - **MySQL**: ใส่ข้อมูล Host, Database, User, และ Password
   - **Elasticsearch**: ใส่ URL และ Index Name
4. คลิก **Save & Test**

---

### **4. สร้าง Dashboard**
1. ไปที่ **Create > Dashboard**
2. เลือก **Add new panel**
3. ตั้งค่าการแสดงผล:
   - เลือกรูปแบบ (Graph, Gauge, Table)
   - เขียน Query ดึงข้อมูลจาก Data Source:
     - ตัวอย่างสำหรับ Prometheus:
       ```promql
       rate(http_requests_total[5m])
       ```
     - ตัวอย่างสำหรับ MySQL:
       ```sql
       SELECT time, value FROM metrics WHERE metric_name = 'cpu_usage';
       ```
   - ปรับแต่งการแสดงผล (เช่น เปลี่ยนสี, ตั้งชื่อ Panel)
4. คลิก **Apply** เพื่อเพิ่ม Panel ลงใน Dashboard

---

### **5. การแจ้งเตือน (Alerting)**
1. เลือก Panel ที่ต้องการตั้งค่า Alert
2. คลิก **Alert > Create Alert Rule**
3. ตั้งค่าเงื่อนไข (Condition):
   - เช่น CPU Usage เกิน 80%:
     ```promql
     avg(node_cpu_seconds_total{mode="idle"}) < 20
     ```
4. ตั้งค่าการแจ้งเตือนผ่าน Notification Channel:
   - ไปที่ **Alerting > Notification channels**
   - เลือกช่องทาง เช่น Email, Slack
   - เพิ่มช่องทางใน Alert Rule
5. คลิก **Save** เพื่อบันทึก

---

### **6. การใช้ตัวแปร (Variables)**
1. ไปที่ **Dashboard Settings > Variables**
2. คลิก **Add variable**:
   - ตัวอย่าง: สร้างตัวแปร `region` ดึงค่าจาก Prometheus:
     ```promql
     label_values(node_cpu_seconds_total, region)
     ```
3. ใช้ตัวแปรใน Query:
   ```promql
   node_cpu_seconds_total{region="$region"}
   ```

---

### **7. การใช้งาน Plugin**
1. ติดตั้ง Plugin ผ่านคำสั่ง:
   ```bash
   grafana-cli plugins install grafana-piechart-panel
   ```
2. รีสตาร์ท Grafana:
   ```bash
   sudo systemctl restart grafana-server
   ```
3. ใช้ Plugin ที่ติดตั้งในการสร้าง Dashboard

---

### **ตัวอย่างการใช้งานจริง**
#### **ตัวอย่าง 1: Monitoring Kubernetes**
1. ใช้ Prometheus รวบรวม Metrics จาก Kubernetes
2. สร้าง Dashboard เพื่อแสดงข้อมูล เช่น:
   - จำนวน Pod ที่ Running:
     ```promql
     kube_pod_status_phase{phase="Running"}
     ```
   - CPU Usage ของ Pod:
     ```promql
     sum(rate(container_cpu_usage_seconds_total[5m])) by (pod)
     ```

#### **ตัวอย่าง 2: Business Metrics**
1. ใช้ MySQL หรือ Elasticsearch เก็บข้อมูลธุรกิจ เช่น จำนวนผู้ใช้งาน
2. สร้าง Dashboard เพื่อแสดงผล เช่น:
   - จำนวนผู้ใช้งานที่ Active:
     ```sql
     SELECT time, active_users FROM user_activity;
     ```

---

### **8. การจัดการผู้ใช้และสิทธิ์**
1. ไปที่ **Configuration > Users**
2. เพิ่มผู้ใช้ใหม่และกำหนดสิทธิ์ เช่น Viewer, Editor, Admin
3. ใช้ **Organizations** เพื่อแยก Dashboard ตามทีม

---

### **เคล็ดลับการใช้งาน**
1. ใช้ **Templates และ Variables** เพื่อสร้าง Dashboard ที่ยืดหยุ่น
2. ใช้ **Annotations** เพื่อบันทึกเหตุการณ์สำคัญในกราฟ
3. ใช้ **Auto Refresh** เพื่ออัปเดต Dashboard แบบเรียลไทม์

