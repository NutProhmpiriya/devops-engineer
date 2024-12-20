# Exercise 4: Horizontal Pod Autoscaling

## Task
สร้าง Deployment ที่มี HorizontalPodAutoscaler เพื่อปรับขนาดตามการใช้งาน CPU

### Requirements
1. สร้าง Deployment ที่มี resource requests/limits
2. Configure HorizontalPodAutoscaler
3. สร้าง load testing pod
4. Monitor การ scale
5. Set up metrics-server (ถ้ายังไม่มี)

## Solution

1. สร้าง Deployment (`app-deployment.yaml`):
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: php-apache
spec:
  selector:
    matchLabels:
      run: php-apache
  template:
    metadata:
      labels:
        run: php-apache
    spec:
      containers:
      - name: php-apache
        image: k8s.gcr.io/hpa-example
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

2. สร้าง Service (`app-service.yaml`):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: php-apache
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    run: php-apache
```

3. สร้าง HorizontalPodAutoscaler (`app-hpa.yaml`):
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: php-apache
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: php-apache
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
```

4. สร้าง load generator (`load-generator.yaml`):
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: load-generator
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh", "-c", "while true; do wget -q -O- http://php-apache; done"]
```

### คำสั่งที่ใช้
```bash
# ติดตั้ง metrics-server (ถ้าใช้ Minikube)
minikube addons enable metrics-server

# หรือติดตั้งด้วย kubectl
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# สร้าง Deployment และ Service
kubectl apply -f app-deployment.yaml
kubectl apply -f app-service.yaml

# สร้าง HPA
kubectl apply -f app-hpa.yaml

# ตรวจสอบ HPA
kubectl get hpa

# สร้าง load
kubectl apply -f load-generator.yaml
```

### การตรวจสอบ
1. Monitor HPA:
```bash
kubectl get hpa php-apache --watch
```

2. ดู Pod Scaling:
```bash
kubectl get pods -w
```

3. ตรวจสอบ CPU usage:
```bash
kubectl top pods
```

### การทดสอบ
1. เริ่ม load test:
```bash
# สร้าง load
kubectl apply -f load-generator.yaml

# รอสักครู่แล้วตรวจสอบการ scale
kubectl get hpa
kubectl get pods

# หยุด load
kubectl delete -f load-generator.yaml
```

### Tips
- ตรวจสอบว่า metrics-server ทำงานถูกต้อง
- ใช้ `kubectl describe hpa` เพื่อดูรายละเอียดการ scale
- ถ้า HPA ไม่ทำงาน ตรวจสอบ metrics: `kubectl top pods`
- ปรับค่า CPU threshold ตามความเหมาะสมของแอปพลิเคชัน
