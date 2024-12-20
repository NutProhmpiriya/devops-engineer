# Kubernetes Security

## Security Fundamentals
1. **Authentication**
   - Service Accounts
   - User Authentication
   - X.509 Certificates
   
   > 🇹🇭 การยืนยันตัวตนใน Kubernetes มีหลายรูปแบบ ต้องเลือกใช้ให้เหมาะสม

2. **Authorization (RBAC)**
```yaml
# Role example
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

# RoleBinding example
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

> 🇹🇭 RBAC เป็นระบบจัดการสิทธิ์มาตรฐานใน Kubernetes ต้องกำหนดให้รัดกุม

## Container Security
1. **Image Security**
   - Vulnerability Scanning
   - Image Signing
   - Private Registry
   
   > 🇹🇭 ความปลอดภัยของ Container Image เป็นจุดเริ่มต้นที่สำคัญ

2. **Runtime Security**
```yaml
# Pod Security Context
apiVersion: v1
kind: Pod
metadata:
  name: security-context-demo
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: sec-ctx-demo
    image: busybox
    command: [ "sh", "-c", "sleep 1h" ]
    securityContext:
      allowPrivilegeEscalation: false
      capabilities:
        drop:
        - ALL
```

## Network Security
1. **Network Policies**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: api-allow
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 8080
```

2. **TLS Configuration**
   - Certificates Management
   - Secure Ingress
   - mTLS with Service Mesh
   
   > 🇹🇭 การเข้ารหัสการสื่อสารระหว่าง Service เป็นสิ่งจำเป็น

## Secrets Management
1. **Kubernetes Secrets**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: dXNlcm5hbWU=
  password: cGFzc3dvcmQ=
```

2. **External Secrets**
   - HashiCorp Vault
   - AWS Secrets Manager
   - Azure Key Vault
   
   > 🇹🇭 ควรใช้ระบบจัดการ Secret ภายนอกที่มีความปลอดภัยสูง

## Security Best Practices
1. **Pod Security Standards**
   - Privileged
   - Baseline
   - Restricted
   
   > 🇹🇭 มาตรฐานความปลอดภัยของ Pod ที่ควรปฏิบัติตาม

2. **Audit Logging**
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: RequestResponse
  resources:
  - group: ""
    resources: ["pods"]
```

3. **Regular Security Scanning**
   - Vulnerability Scanning
   - Configuration Scanning
   - Compliance Checking
   
   > 🇹🇭 ต้องทำการตรวจสอบความปลอดภัยอย่างสม่ำเสมอ

## Incident Response
1. **Detection**
   - Security Monitoring
   - Alert Configuration
   - Threat Detection
   
   > 🇹🇭 การตรวจจับภัยคุกคามต้องทำแบบ Real-time

2. **Response Plan**
   - Incident Documentation
   - Response Procedures
   - Recovery Plans
   
   > 🇹🇭 ต้องมีแผนรับมือเหตุการณ์ที่ชัดเจน
