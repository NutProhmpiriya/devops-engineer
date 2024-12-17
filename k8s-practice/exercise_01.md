การสร้างแบบฝึกหัด Kubernetes (k8s) จากพื้นฐานไปจนถึงขั้นสูง สามารถจัดเป็นขั้นตอนต่อเนื่องเพื่อให้ผู้เรียนค่อยๆ พัฒนาและเข้าใจได้ลึกซึ้งขึ้น ตามลำดับดังนี้:

---

### **ขั้นตอนที่ 1: พื้นฐาน Kubernetes**
#### **Exercise 1.1: ติดตั้ง Kubernetes**
- **วัตถุประสงค์**: รู้จักและติดตั้ง Kubernetes บนเครื่อง Local
- **คำสั่ง**:
  1. ติดตั้ง `Minikube` บนเครื่อง Local
  2. ติดตั้ง `kubectl` เพื่อใช้สั่งงาน Kubernetes cluster
- **การทดสอบ**:
   - รันคำสั่ง `kubectl get nodes` และแคปเจอร์ผลลัพธ์

#### **Exercise 1.2: สร้าง Pod อย่างง่าย**
- **วัตถุประสงค์**: เรียนรู้การสร้างและจัดการ Pod
- **คำสั่ง**:
  1. สร้าง Pod โดยใช้ไฟล์ YAML
      ```yaml
      apiVersion: v1
      kind: Pod
      metadata:
        name: nginx-pod
      spec:
        containers:
          - name: nginx
            image: nginx:latest
            ports:
              - containerPort: 80
      ```
  2. Apply ไฟล์นี้ด้วย `kubectl apply -f pod.yaml`
- **การทดสอบ**:
   - ตรวจสอบ Pod ด้วย `kubectl get pods`
   - ทดสอบเข้าถึง Pod ภายใน cluster โดยใช้คำสั่ง `kubectl port-forward`

---

### **ขั้นตอนที่ 2: ทำความเข้าใจกับ Services และ Deployments**
#### **Exercise 2.1: สร้าง Service เพื่อ expose Pod**
- **วัตถุประสงค์**: สร้าง Service ให้ Pod สามารถรับการเข้าถึงจากภายนอก
- **คำสั่ง**:
  1. สร้างไฟล์ YAML สำหรับ Service
      ```yaml
      apiVersion: v1
      kind: Service
      metadata:
        name: nginx-service
      spec:
        type: NodePort
        selector:
          app: nginx
        ports:
          - protocol: TCP
            port: 80
            targetPort: 80
      ```
  2. Deploy และตรวจสอบ NodePort
- **การทดสอบ**:
   - ใช้ `kubectl get services` ดูหมายเลขพอร์ต
   - เปิดเว็บเบราว์เซอร์ทดสอบการเข้าถึง NGINX

#### **Exercise 2.2: สร้าง Deployment**
- **วัตถุประสงค์**: เข้าใจการใช้ Deployments สำหรับ scaling และ update
- **คำสั่ง**:
  1. สร้าง Deployment
      ```yaml
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: nginx-deployment
      spec:
        replicas: 3
        selector:
          matchLabels:
            app: nginx
        template:
          metadata:
            labels:
              app: nginx
          spec:
            containers:
            - name: nginx
              image: nginx:latest
              ports:
              - containerPort: 80
      ```
  2. Deploy และตรวจสอบ Pods ที่ถูกสร้าง
- **การทดสอบ**:
   - ตรวจสอบ `kubectl get deployments` และ `kubectl get pods -o wide`
   - ลอง scale ขึ้นหรือลงด้วย `kubectl scale`

---

### **ขั้นตอนที่ 3: ใช้งาน ConfigMaps และ Secrets**
#### **Exercise 3.1: ใช้ ConfigMap**
- **วัตถุประสงค์**: จัดการคอนฟิกแยกจาก container
- **คำสั่ง**:
  1. สร้าง ConfigMap:
      ```yaml
      apiVersion: v1
      kind: ConfigMap
      metadata:
        name: example-config
      data:
        app.properties: |
          key1=value1
          key2=value2
      ```
  2. ใช้ ConfigMap ใน Pod:
      ```yaml
      ...
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
          name: example-config
      ```
- **การทดสอบ**:
   - ตรวจสอบไฟล์คอนฟิกภายใน container

---

### **ขั้นตอนที่ 4: การทำงานกับ Persistent Storage**
#### **Exercise 4.1: ใช้ Persistent Volume และ Persistent Volume Claim**
- **วัตถุประสงค์**: เชื่อมต่อ Storage ให้ Pod เก็บข้อมูลถาวร
- **คำสั่ง**:
  1. สร้าง PersistentVolume (PV)
  2. สร้าง PersistentVolumeClaim (PVC)
  3. เชื่อม PVC กับ Pod

---

### **ขั้นตอนที่ 5: Kubernetes Networking และ Ingress**
#### **Exercise 5.1: ติดตั้ง Ingress Controller**
- **วัตถุประสงค์**: ใช้ Ingress สำหรับจัดการเส้นทาง HTTP/HTTPS
- **คำสั่ง**:
  1. ติดตั้ง NGINX Ingress Controller
  2. สร้าง Ingress Resource สำหรับ routing traffic

---

### **ขั้นตอนที่ 6: Monitoring และ Logging**
#### **Exercise 6.1: ติดตั้ง Prometheus และ Grafana**
- **วัตถุประสงค์**: ตรวจสอบ cluster และ resource
- **คำสั่ง**:
  1. ใช้ Helm ติดตั้ง Prometheus
  2. ติดตั้ง Grafana และเชื่อมกับ Prometheus

---

### **ขั้นตอนที่ 7: CI/CD กับ Kubernetes**
#### **Exercise 7.1: ใช้ GitHub Actions เพื่อ Deploy ไปยัง Kubernetes**
- **วัตถุประสงค์**: สร้าง workflow ที่ทำการ build และ deploy อัตโนมัติ
- **คำสั่ง**:
  1. เขียนไฟล์ GitHub Actions YAML
  2. ใช้ `kubectl` และ `kustomize` สำหรับ deploy

---

### **ขั้นตอนที่ 8: ปรับปรุงและปรับแต่ง**
- เรียนรู้ Horizontal Pod Autoscaling (HPA)
- ทดลองใช้ Resource Limits และ Requests
- ทำ Canary Deployment หรือ Blue/Green Deployment

---

