# Exercise 2: Stateful Database Deployment

## Task
สร้าง StatefulSet สำหรับ MySQL database พร้อม Persistent Volume และ Secret สำหรับ password

### Requirements
1. สร้าง StatefulSet สำหรับ MySQL
2. กำหนด Persistent Volume Claim
3. สร้าง Secret สำหรับ root password
4. สร้าง Service สำหรับเข้าถึง database
5. กำหนด health checks

## Solution

1. สร้าง Secret สำหรับ MySQL password (`mysql-secret.yaml`):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-secret
type: Opaque
data:
  # 'mypassword' in base64
  MYSQL_ROOT_PASSWORD: bXlwYXNzd29yZA==
```

2. สร้าง PersistentVolumeClaim (`mysql-pvc.yaml`):
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
```

3. สร้าง StatefulSet (`mysql-statefulset.yaml`):
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql
  replicas: 1
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
          name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: MYSQL_ROOT_PASSWORD
        livenessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          tcpSocket:
            port: 3306
          initialDelaySeconds: 5
          periodSeconds: 5
        volumeMounts:
        - name: mysql-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-storage
        persistentVolumeClaim:
          claimName: mysql-pvc
```

4. สร้าง Service (`mysql-service.yaml`):
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
  - port: 3306
    targetPort: 3306
  selector:
    app: mysql
  clusterIP: None
```

### คำสั่งที่ใช้
```bash
# สร้าง Secret
kubectl apply -f mysql-secret.yaml

# สร้าง PVC
kubectl apply -f mysql-pvc.yaml

# สร้าง StatefulSet และ Service
kubectl apply -f mysql-statefulset.yaml
kubectl apply -f mysql-service.yaml

# ตรวจสอบสถานะ
kubectl get statefulset mysql
kubectl get pods -l app=mysql
kubectl get pvc
kubectl get services mysql
```

### การทดสอบ
1. เชื่อมต่อกับ MySQL:
```bash
kubectl run -it --rm --image=mysql:8.0 --restart=Never mysql-client -- mysql -h mysql -pมายพาสเวิร์ด
```

2. ทดสอบการเก็บข้อมูล:
```sql
CREATE DATABASE testdb;
USE testdb;
CREATE TABLE users (id INT, name VARCHAR(255));
INSERT INTO users VALUES (1, 'test user');
SELECT * FROM users;
```

### Tips
- ใช้ `kubectl logs` เพื่อตรวจสอบ log ของ MySQL
- ตรวจสอบ persistent volume: `kubectl get pv`
- ถ้า pod ไม่ขึ้น ให้ตรวจสอบ events: `kubectl describe pod mysql-0`
