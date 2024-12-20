# แบบฝึกหัดที่ 2: การใช้งาน Google Kubernetes Engine (GKE)
# Exercise 2: Working with Google Kubernetes Engine

## โจทย์ | Problem
สร้างและจัดการ Kubernetes cluster บน GKE สำหรับ deployment ของแอปพลิเคชัน microservices โดยมีความต้องการดังนี้:

1. สร้าง GKE cluster with:
   - 2 node pools
   - Node pool 1: 2 nodes (e2-medium) สำหรับ general workloads
   - Node pool 2: 1 node (e2-standard-2) สำหรับ database workloads
   - Auto-scaling enabled
   - Regional cluster (asia-southeast1)

2. Deploy ตัวอย่างแอปพลิเคชันที่ประกอบด้วย:
   - Frontend service (Nginx)
   - Backend service (Node.js)
   - Database (MongoDB)
   - Ingress controller

## เฉลย | Solution

### 1. สร้าง GKE Cluster:
```bash
# Create the cluster
gcloud container clusters create multi-service-cluster \
    --region=asia-southeast1 \
    --num-nodes=2 \
    --machine-type=e2-medium \
    --node-labels=workload=general \
    --enable-autoscaling \
    --min-nodes=1 \
    --max-nodes=3

# Create second node pool for database
gcloud container node-pools create db-pool \
    --cluster=multi-service-cluster \
    --region=asia-southeast1 \
    --num-nodes=1 \
    --machine-type=e2-standard-2 \
    --node-labels=workload=database \
    --enable-autoscaling \
    --min-nodes=1 \
    --max-nodes=2
```

### 2. Deploy แอปพลิเคชัน:

#### MongoDB Deployment (mongodb.yaml):
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mongodb
spec:
  serviceName: mongodb
  replicas: 1
  selector:
    matchLabels:
      app: mongodb
  template:
    metadata:
      labels:
        app: mongodb
    spec:
      nodeSelector:
        workload: database
      containers:
      - name: mongodb
        image: mongo:4.4
        ports:
        - containerPort: 27017
---
apiVersion: v1
kind: Service
metadata:
  name: mongodb
spec:
  clusterIP: None
  selector:
    app: mongodb
  ports:
  - port: 27017
```

#### Backend Service (backend.yaml):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      nodeSelector:
        workload: general
      containers:
      - name: backend
        image: node:16-alpine
        ports:
        - containerPort: 3000
---
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  type: ClusterIP
  selector:
    app: backend
  ports:
  - port: 80
    targetPort: 3000
```

#### Frontend Service (frontend.yaml):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      nodeSelector:
        workload: general
      containers:
      - name: frontend
        image: nginx:alpine
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: frontend
spec:
  type: ClusterIP
  selector:
    app: frontend
  ports:
  - port: 80
```

#### Ingress (ingress.yaml):
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  annotations:
    kubernetes.io/ingress.class: "gce"
spec:
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: backend
            port:
              number: 80
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 80
```

### 3. Deploy แอปพลิเคชัน:
```bash
# Get cluster credentials
gcloud container clusters get-credentials multi-service-cluster --region=asia-southeast1

# Apply configurations
kubectl apply -f mongodb.yaml
kubectl apply -f backend.yaml
kubectl apply -f frontend.yaml
kubectl apply -f ingress.yaml
```

## การทดสอบ | Testing
```bash
# Check node pools
kubectl get nodes --label-columns=workload

# Check pods distribution
kubectl get pods -o wide

# Check services
kubectl get services

# Check ingress
kubectl get ingress app-ingress

# Get external IP
export INGRESS_IP=$(kubectl get ingress app-ingress -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl http://$INGRESS_IP
```

## เพิ่มเติม | Additional Notes
- ใช้ Workload Identity สำหรับการเข้าถึง Google Cloud Services
- ตั้งค่า horizontal pod autoscaling (HPA) สำหรับ workloads
- ใช้ Cloud Storage หรือ Persistent Volumes สำหรับ stateful applications
- ตั้งค่า monitoring ด้วย Cloud Operations Suite
- พิจารณาใช้ Cloud Build สำหรับ CI/CD pipeline
