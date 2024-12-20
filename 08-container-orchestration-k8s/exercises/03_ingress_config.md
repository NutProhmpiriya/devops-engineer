# Exercise 3: Ingress Configuration

## Task
สร้าง Ingress Controller และ configure routing rules สำหรับ 2 web applications

### Requirements
1. Deploy 2 web applications (app1 และ app2)
2. สร้าง Ingress rules สำหรับ path-based routing
3. Configure TLS
4. Set up health checks
5. Add annotations สำหรับ rate limiting

## Solution

1. สร้าง Deployments สำหรับ apps (`apps-deployment.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app1
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app1
  template:
    metadata:
      labels:
        app: app1
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app2
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app2
  template:
    metadata:
      labels:
        app: app2
    spec:
      containers:
      - name: nginx
        image: nginx:1.20
        ports:
        - containerPort: 80
```

2. สร้าง Services (`apps-service.yaml`):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app1-service
spec:
  selector:
    app: app1
  ports:
  - port: 80
    targetPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: app2-service
spec:
  selector:
    app: app2
  ports:
  - port: 80
    targetPort: 80
```

3. สร้าง TLS Secret (`tls-secret.yaml`):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: <base64-encoded-cert>
  tls.key: <base64-encoded-key>
```

4. สร้าง Ingress (`apps-ingress.yaml`):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: apps-ingress
  annotations:
    nginx.ingress.kubernetes.io/rate-limit-rpm: "100"
    nginx.ingress.kubernetes.io/enable-cors: "true"
spec:
  tls:
  - hosts:
    - apps.example.com
    secretName: tls-secret
  rules:
  - host: apps.example.com
    http:
      paths:
      - path: /app1
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
      - path: /app2
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

### คำสั่งที่ใช้
```bash
# สร้าง Deployments และ Services
kubectl apply -f apps-deployment.yaml
kubectl apply -f apps-service.yaml

# สร้าง TLS Secret
kubectl apply -f tls-secret.yaml

# สร้าง Ingress
kubectl apply -f apps-ingress.yaml

# ตรวจสอบสถานะ
kubectl get ingress
kubectl get pods
kubectl get services
```

### การทดสอบ
1. เพิ่ม DNS entry ใน /etc/hosts:
```
127.0.0.1 apps.example.com
```

2. ทดสอบการเข้าถึง:
```bash
# ทดสอบ app1
curl -k https://apps.example.com/app1

# ทดสอบ app2
curl -k https://apps.example.com/app2
```

### การตรวจสอบ
1. ตรวจสอบ Ingress status:
```bash
kubectl describe ingress apps-ingress
```

2. ตรวจสอบ logs:
```bash
kubectl logs -n ingress-nginx -l app.kubernetes.io/name=ingress-nginx
```

### Tips
- ตรวจสอบ Ingress Controller ว่าทำงานถูกต้อง
- ใช้ `kubectl get events` เพื่อดู events ที่เกิดขึ้น
- ถ้าใช้ Minikube ต้องเปิด Ingress addon: `minikube addons enable ingress`
