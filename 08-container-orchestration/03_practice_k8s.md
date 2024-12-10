### สิ่งที่ควรรู้เกี่ยวกับ Kubernetes (K8s)

Kubernetes (K8s) เป็นแพลตฟอร์มที่ทรงพลังและมีความซับซ้อน การทำความเข้าใจพื้นฐาน รวมถึงฟีเจอร์และเครื่องมือที่เกี่ยวข้อง จะช่วยให้คุณใช้งาน Kubernetes ได้อย่างมีประสิทธิภาพมากขึ้น

---

## 1. **พื้นฐานของ Kubernetes**
- **Cluster**: กลุ่มของโหนดที่ Kubernetes ใช้จัดการแอปพลิเคชัน
  - **Master Node**: จัดการ Control Plane (API Server, Scheduler, Controller Manager)
  - **Worker Node**: รันแอปพลิเคชันใน Pods
- **Pod**: หน่วยงานเล็กที่สุดที่ Kubernetes จัดการ โดยใน Pod จะมี Container 1 หรือมากกว่า
- **Namespace**: ช่วยจัดการแอปพลิเคชันหลายตัวใน Cluster เดียว
- **Service**: เปิดเผย Pods ให้สามารถเข้าถึงได้ เช่น ClusterIP, NodePort, LoadBalancer
- **Volume**: ใช้เก็บข้อมูลที่ต้องการความถาวรระหว่างการรัน Pods

---

## 2. **คำสั่ง `kubectl`**
- **ดูข้อมูลทรัพยากร**:
  ```bash
  kubectl get nodes
  kubectl get pods -n <namespace>
  kubectl get services
  ```
- **สร้างและลบทรัพยากร**:
  ```bash
  kubectl apply -f <file>.yaml
  kubectl delete -f <file>.yaml
  ```
- **Debug และตรวจสอบ**:
  ```bash
  kubectl logs <pod-name>
  kubectl exec -it <pod-name> -- /bin/bash
  kubectl describe pod <pod-name>
  ```

---

## 3. **ชนิดของ Kubernetes Resources**
- **Workload Resources**:
  - **Pod**: หน่วยงานพื้นฐาน
  - **Deployment**: ใช้ปรับขนาด (Scaling) และอัปเดตแอปพลิเคชัน
  - **StatefulSet**: สำหรับแอปพลิเคชันที่ต้องการการจัดเก็บสถานะ เช่น Database
  - **DaemonSet**: ใช้กระจาย Pod ให้รันในทุก Node
  - **Job และ CronJob**: ใช้สำหรับรันงานแบบชั่วคราว

- **Networking Resources**:
  - **Service**: เปิดเผย Pods ผ่าน ClusterIP, NodePort, หรือ LoadBalancer
  - **Ingress**: จัดการ HTTP/HTTPS Routing

- **Storage Resources**:
  - **PersistentVolume (PV)**: จัดการพื้นที่เก็บข้อมูล
  - **PersistentVolumeClaim (PVC)**: ใช้เพื่อร้องขอ PV

---

## 4. **แนวคิดสำคัญใน Kubernetes**
### 4.1 **Declarative vs Imperative**
- **Declarative**: กำหนดสถานะที่ต้องการ เช่น ใช้ไฟล์ YAML
- **Imperative**: ใช้คำสั่ง `kubectl` เพื่อดำเนินการทันที
  - Declarative จะเหมาะกับการจัดการระยะยาวและการทำ CI/CD

### 4.2 **Control Plane**
- Kubernetes ใช้ Control Plane ในการตรวจสอบและปรับ Cluster ให้ตรงกับสถานะที่ต้องการ:
  - API Server: รับคำสั่งจากผู้ใช้หรือ CI/CD
  - Scheduler: จัดการตำแหน่งการรัน Pod ใน Node
  - etcd: เก็บสถานะของ Cluster

---

## 5. **การปรับขนาด (Scaling)**
- **Horizontal Scaling**:
  เพิ่มหรือลดจำนวน Pods:
  ```bash
  kubectl scale deployment <deployment-name> --replicas=3
  ```

- **Vertical Scaling**:
  ปรับขนาด CPU/Memory ของ Pod ในไฟล์ YAML:
  ```yaml
  resources:
    requests:
      memory: "128Mi"
      cpu: "500m"
    limits:
      memory: "256Mi"
      cpu: "1000m"
  ```

---

## 6. **การจัดการ Configuration**
- **ConfigMap**: ใช้เก็บค่าคอนฟิก เช่น Environment Variables
- **Secrets**: ใช้เก็บข้อมูลสำคัญ เช่น Passwords
  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: app-config
  data:
    DATABASE_HOST: localhost
  ```

- ใช้ ConfigMap ใน Deployment:
  ```yaml
  env:
    - name: DATABASE_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DATABASE_HOST
  ```

---

## 7. **การจัดการ Networking**
- **Cluster Networking**:
  - แต่ละ Pod สามารถสื่อสารกันได้ใน Cluster
- **Service**:
  - **ClusterIP**: ค่าเริ่มต้นสำหรับสื่อสารภายใน Cluster
  - **NodePort**: เปิดพอร์ตบน Node
  - **LoadBalancer**: เชื่อมโยงกับ Load Balancer บน Cloud
- **Ingress**:
  ใช้จัดการ HTTP/HTTPS Traffic:
  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: example-ingress
  spec:
    rules:
    - host: example.com
      http:
        paths:
        - path: /
          pathType: Prefix
          backend:
            service:
              name: my-service
              port:
                number: 80
  ```

---

## 8. **การจัดการ Storage**
- **PersistentVolume (PV)**:
  กำหนดพื้นที่เก็บข้อมูล:
  ```yaml
  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: my-pv
  spec:
    capacity:
      storage: 10Gi
    accessModes:
      - ReadWriteOnce
    hostPath:
      path: /data
  ```

- **PersistentVolumeClaim (PVC)**:
  ร้องขอ PV:
  ```yaml
  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: my-pvc
  spec:
    accessModes:
      - ReadWriteOnce
    resources:
      requests:
        storage: 10Gi
  ```

---

## 9. **การ Debug**
- **ตรวจสอบ Logs**:
  ```bash
  kubectl logs <pod-name>
  ```
- **ตรวจสอบ Events**:
  ```bash
  kubectl get events
  ```
- **เข้าไปใน Pod**:
  ```bash
  kubectl exec -it <pod-name> -- /bin/bash
  ```

---

## 10. **การทำ CI/CD กับ Kubernetes**
- ใช้เครื่องมือ เช่น:
  - **Helm**: จัดการการปรับใช้แอปพลิเคชัน
  - **ArgoCD**: สำหรับ GitOps
  - **Jenkins หรือ GitHub Actions**: สำหรับ Automate Deployment

---

## 11. **เครื่องมือเสริมสำหรับ Kubernetes**
- **Kubectl Plugins**: เช่น `kubectl krew`
- **K9s**: Interface แบบ CLI สำหรับการดูแล Cluster
- **Lens**: Interface แบบ GUI สำหรับการจัดการ Cluster
- **Prometheus และ Grafana**: สำหรับ Monitoring
- **Fluentd หรือ Elasticsearch**: สำหรับ Log Aggregation

---

## 12. **Best Practices**
1. **ใช้ Namespace**:
   - แยกแอปพลิเคชันหรือทีมใน Namespace ต่างๆ
2. **จำกัดทรัพยากรของ Pods**:
   - ใช้ `resources` เพื่อป้องกัน Node ถูกใช้งานจนหมด
3. **ใช้ Readiness และ Liveness Probes**:
   - ตรวจสอบสุขภาพของ Pods:
     ```yaml
     livenessProbe:
       httpGet:
         path: /healthz
         port: 8080
       initialDelaySeconds: 3
       periodSeconds: 5
     ```
4. **เก็บ Logs และ Monitor Cluster**:
   - ใช้เครื่องมือเช่น Prometheus หรือ Fluentd
5. **ใช้ Infrastructure as Code**:
   - เก็บ YAML ไฟล์ใน Git Repository และจัดการด้วย CI/CD

