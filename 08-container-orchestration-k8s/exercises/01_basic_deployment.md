# Exercise 1: Basic Deployment and Service

## Task
สร้าง Deployment ของ web application โดยใช้ nginx และทำให้สามารถเข้าถึงได้จากภายนอก cluster

### Requirements
1. สร้าง Deployment ที่มี 3 replicas
2. ใช้ image nginx:1.20
3. กำหนด resource limits และ requests
4. สร้าง Service type LoadBalancer
5. กำหนด labels และ port ให้ถูกต้อง

## Solution

1. สร้างไฟล์ `web-deployment.yaml`:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-web
  labels:
    app: nginx
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
        image: nginx:1.20
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: "0.5"
            memory: "512Mi"
          requests:
            cpu: "0.2"
            memory: "256Mi"
```

2. สร้างไฟล์ `web-service.yaml`:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-web-service
spec:
  type: LoadBalancer
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
```

### คำสั่งที่ใช้
```bash
# สร้าง Deployment
kubectl apply -f web-deployment.yaml

# สร้าง Service
kubectl apply -f web-service.yaml

# ตรวจสอบสถานะ
kubectl get deployments
kubectl get pods
kubectl get services

# ทดสอบการเข้าถึง
curl http://<EXTERNAL-IP>
```

### การตรวจสอบ
1. ตรวจสอบว่ามี pods 3 ตัวทำงานอยู่:
```bash
kubectl get pods | grep nginx-web
```

2. ตรวจสอบ Service:
```bash
kubectl get svc nginx-web-service
```

3. ตรวจสอบ logs:
```bash
kubectl logs -l app=nginx
```

### Tips
- ใช้ `kubectl describe` เพื่อดูรายละเอียดเพิ่มเติมเมื่อมีปัญหา
- ตรวจสอบ events: `kubectl get events`
- ถ้าใช้ Minikube อาจต้องใช้ `minikube tunnel` เพื่อเข้าถึง LoadBalancer
