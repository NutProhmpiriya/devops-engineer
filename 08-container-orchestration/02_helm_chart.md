### Helm Chart: สิ่งที่ควรรู้และองค์ประกอบสำคัญ

**Helm** เป็นเครื่องมือสำหรับจัดการ Kubernetes applications ด้วย **Helm Charts** ซึ่งเป็นแพ็กเกจที่ประกอบไปด้วยไฟล์ต่างๆ ที่จำเป็นในการติดตั้งและจัดการแอปพลิเคชันบน Kubernetes

---

## 1. **โครงสร้างของ Helm Chart**
เมื่อคุณสร้าง Helm Chart ใหม่ จะได้โครงสร้างโฟลเดอร์ดังนี้:
```
my-chart/
├── Chart.yaml        # ข้อมูลเมตาของ Chart
├── values.yaml       # ค่าเริ่มต้นสำหรับ Templates
├── templates/        # ไฟล์ YAML Template สำหรับ Kubernetes
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── _helpers.tpl  # ฟังก์ชันและตัวแปรที่ใช้ใน Templates
│   └── NOTES.txt     # ข้อความแนะนำหลังการติดตั้ง
├── charts/           # Chart dependencies (optional)
└── README.md         # ข้อมูลการใช้งาน Chart
```

---

## 2. **องค์ประกอบสำคัญใน Helm Chart**
### 2.1 **Chart.yaml**
- ไฟล์ที่เก็บข้อมูลเมตาของ Chart เช่น ชื่อ, เวอร์ชัน และคำอธิบาย
- ตัวอย่าง:
  ```yaml
  apiVersion: v2
  name: my-chart
  description: A Helm chart for my application
  type: application
  version: 0.1.0
  appVersion: 1.0.0
  ```

### 2.2 **values.yaml**
- กำหนดค่าพื้นฐานที่ใช้ใน Templates
- ตัวอย่าง:
  ```yaml
  replicaCount: 2
  image:
    repository: nginx
    tag: latest
  service:
    type: ClusterIP
    port: 80
  ```

### 2.3 **templates/** 
- ไฟล์ YAML Template ที่กำหนด Kubernetes Resources (เช่น Deployment, Service, Ingress)
- ใช้ตัวแปรจาก `values.yaml` และมีการใช้งาน Go Templating
- ตัวอย่าง `deployment.yaml`:
  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: {{ .Release.Name }}-deployment
  spec:
    replicas: {{ .Values.replicaCount }}
    selector:
      matchLabels:
        app: {{ .Release.Name }}
    template:
      metadata:
        labels:
          app: {{ .Release.Name }}
      spec:
        containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
          - containerPort: 80
  ```

### 2.4 **_helpers.tpl**
- เก็บฟังก์ชันที่ใช้ซ้ำใน Template เพื่อเพิ่มความยืดหยุ่น
- ตัวอย่าง:
  ```tpl
  {{- define "fullname" -}}
  {{ .Release.Name }}-{{ .Chart.Name }}
  {{- end -}}
  ```

### 2.5 **NOTES.txt**
- ข้อความที่จะแสดงให้ผู้ใช้หลังติดตั้ง Chart สำเร็จ
- ตัวอย่าง:
  ```txt
  Your application has been deployed!
  You can access it using the following command:
  kubectl port-forward svc/{{ .Release.Name }} 8080:80
  ```

### 2.6 **charts/**
- โฟลเดอร์สำหรับเก็บ Chart dependencies
- ใช้ `requirements.yaml` หรือ `Chart.yaml` เพื่อกำหนด dependencies:
  ```yaml
  dependencies:
    - name: dependency-chart
      version: 1.2.3
      repository: https://example.com/charts
  ```

---

## 3. **การใช้งาน Helm Chart**
### 3.1 สร้าง Helm Chart ใหม่:
```bash
helm create my-chart
```

### 3.2 ติดตั้ง Chart:
```bash
helm install my-release my-chart
```

### 3.3 ตรวจสอบการติดตั้ง:
```bash
helm list
```

### 3.4 อัปเดต Chart:
```bash
helm upgrade my-release my-chart
```

### 3.5 ลบ Chart:
```bash
helm uninstall my-release
```

---

## 4. **ค่าที่กำหนดใน Chart**
### Override ค่าใน `values.yaml`:
1. ใช้ `--set` เพื่อระบุค่า:
   ```bash
   helm install my-release my-chart --set replicaCount=3
   ```

2. ใช้ไฟล์ Custom Values:
   ```bash
   helm install my-release my-chart -f custom-values.yaml
   ```

---

## 5. **Best Practices สำหรับ Helm Chart**
1. **ใช้ `_helpers.tpl`**:
   - แยกฟังก์ชันที่ใช้ซ้ำออกจาก Templates
2. **จัดการค่า Default อย่างระมัดระวัง**:
   - ใช้ `values.yaml` เพื่อให้ผู้ใช้กำหนดค่าที่เหมาะสม
3. **รองรับ Dependencies**:
   - ใช้ `charts/` สำหรับแอปพลิเคชันที่มี Dependencies เช่น Database
4. **ทดสอบ Template**:
   - ใช้คำสั่ง `helm template` เพื่อดูผลลัพธ์:
     ```bash
     helm template my-chart
     ```
5. **เพิ่มคำอธิบายใน `NOTES.txt`**:
   - ช่วยให้ผู้ใช้เข้าใจวิธีการเข้าถึงแอปพลิเคชันหลังติดตั้ง

---

## 6. **ตัวอย่างการใช้งาน Helm Chart กับ Kubernetes**
1. สร้าง Chart ใหม่:
   ```bash
   helm create nginx-chart
   ```

2. แก้ไข `values.yaml`:
   ```yaml
   replicaCount: 3
   image:
     repository: nginx
     tag: 1.21.6
   service:
     type: LoadBalancer
     port: 80
   ```

3. ติดตั้ง Chart:
   ```bash
   helm install nginx-release nginx-chart
   ```

4. ตรวจสอบ Pod และ Service:
   ```bash
   kubectl get pods
   kubectl get svc
   ```

---

## 7. **Helm Repositories**
### 7.1 เพิ่ม Repository ใหม่:
```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
```

### 7.2 ค้นหา Chart:
```bash
helm search repo nginx
```

### 7.3 ดาวน์โหลด Chart:
```bash
helm pull bitnami/nginx
```
