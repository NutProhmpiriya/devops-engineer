### Kubernetes: สิ่งที่ควรรู้และวิธีใช้งาน

**Kubernetes (K8s)** เป็นแพลตฟอร์มแบบโอเพ่นซอร์สสำหรับการจัดการ **containerized applications** ในการปรับใช้งาน (deployment), การปรับขนาด (scaling), และการจัดการ (management) โดยรองรับการทำงานแบบอัตโนมัติในคลัสเตอร์ (cluster) ของคอนเทนเนอร์

---

## 1. **พื้นฐานของ Kubernetes**
### 1.1 แนวคิดสำคัญ (Key Concepts):
- **Cluster**: กลุ่มของโหนด (Nodes) ที่ Kubernetes จัดการ
- **Node**:
  - เครื่องใน Cluster ที่ใช้รันแอปพลิเคชัน
  - มี 2 ประเภท:
    - **Master Node**: จัดการคลัสเตอร์
    - **Worker Node**: รัน workload
- **Pod**:
  - หน่วยงานที่เล็กที่สุดใน Kubernetes
  - ภายใน Pod มี 1 หรือหลาย container
- **Service**:
  - ช่วยเปิดเผยแอปพลิเคชันใน Pods ให้สามารถเข้าถึงได้
- **Deployment**:
  - ใช้จัดการ Pods เช่น การปรับขนาด (scaling) หรือการอัปเดต (rolling update)
- **ConfigMap และ Secret**:
  - เก็บค่า configuration และข้อมูลสำคัญ เช่น passwords, API keys

---

## 2. **สถาปัตยกรรม Kubernetes**
### 2.1 Components:
1. **Master Node**:
   - **API Server**: จุดเชื่อมต่อหลักของ Kubernetes
   - **Controller Manager**: จัดการ controllers เช่น replication, node health
   - **Scheduler**: กำหนดว่า Pod จะรันบน Node ใด
   - **etcd**: ระบบจัดเก็บสถานะของคลัสเตอร์

2. **Worker Node**:
   - **Kubelet**: ตัวแทนของ Kubernetes บน Node
   - **Kube Proxy**: จัดการ networking ระหว่าง Pods
   - **Container Runtime**: เช่น Docker หรือ containerd

---

## 3. **การติดตั้ง Kubernetes**
### 3.1 ติดตั้ง Kubernetes ด้วย Minikube:
Minikube ใช้สำหรับการทดลอง Kubernetes บนเครื่องส่วนตัว:
```bash
# ติดตั้ง Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# เริ่มต้นคลัสเตอร์
minikube start

# ตรวจสอบสถานะ
kubectl get nodes
```

### 3.2 ติดตั้ง Kubernetes Cluster ด้วย kubeadm:
ใช้สำหรับติดตั้ง Kubernetes ใน Production:
```bash
# ติดตั้ง kubeadm, kubelet, kubectl
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl

# ตั้งค่า Cluster
sudo kubeadm init
```

---

## 4. **การใช้งานคำสั่ง `kubectl`**
### 4.1 คำสั่งพื้นฐาน:
- ตรวจสอบ Nodes ในคลัสเตอร์:
  ```bash
  kubectl get nodes
  ```
- ตรวจสอบ Pods:
  ```bash
  kubectl get pods
  ```
- สร้าง Resource:
  ```bash
  kubectl apply -f resource.yaml
  ```
- ลบ Resource:
  ```bash
  kubectl delete -f resource.yaml
  ```

---

## 5. **การสร้าง Resource บน Kubernetes**
### 5.1 ตัวอย่างการสร้าง Pod:
ไฟล์ `pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: nginx
    ports:
    - containerPort: 80
```
สร้าง Pod:
```bash
kubectl apply -f pod.yaml
```

### 5.2 การสร้าง Deployment:
ไฟล์ `deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```
สร้าง Deployment:
```bash
kubectl apply -f deployment.yaml
```

### 5.3 การสร้าง Service:
ไฟล์ `service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```
สร้าง Service:
```bash
kubectl apply -f service.yaml
```

---

## 6. **การปรับขนาด (Scaling)**
ปรับจำนวน Replica ของ Deployment:
```bash
kubectl scale deployment my-deployment --replicas=5
```

---

## 7. **การ Debug**
### 7.1 ตรวจสอบ Logs:
```bash
kubectl logs <pod-name>
```

### 7.2 เข้าไปใน Pod:
```bash
kubectl exec -it <pod-name> -- /bin/bash
```

---

## 8. **การจัดการ Configuration**
### 8.1 ConfigMap:
ไฟล์ `configmap.yaml`:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  my-key: my-value
```

ใช้ ConfigMap ใน Pod:
```yaml
env:
- name: MY_ENV_VAR
  valueFrom:
    configMapKeyRef:
      name: my-config
      key: my-key
```

### 8.2 Secret:
ไฟล์ `secret.yaml`:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  password: bXlwYXNzd29yZA==  # base64 encoded
```

ใช้ Secret ใน Pod:
```yaml
env:
- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: password
```

---

## 9. **การอัปเดต (Rolling Updates)**
อัปเดต Deployment:
```bash
kubectl set image deployment/my-deployment nginx=nginx:1.19
```

---

## 10. **Best Practices**
1. **Namespace**:
   - แยกแอปพลิเคชันแต่ละตัวใน Namespace:
     ```bash
     kubectl create namespace my-namespace
     kubectl apply -f deployment.yaml -n my-namespace
     ```

2. **Resource Requests/Limits**:
   - กำหนด CPU/Memory เพื่อป้องกันทรัพยากรหมด:
     ```yaml
     resources:
       requests:
         memory: "64Mi"
         cpu: "250m"
       limits:
         memory: "128Mi"
         cpu: "500m"
     ```

3. **Helm**:
   - ใช้ Helm charts เพื่อลดความซับซ้อนในการจัดการ manifests

4. **Monitoring**:
   - ใช้เครื่องมือ เช่น Prometheus และ Grafana เพื่อตรวจสอบคลัสเตอร์

---

