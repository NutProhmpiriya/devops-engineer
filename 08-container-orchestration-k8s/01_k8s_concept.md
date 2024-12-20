# Kubernetes (K8s) Core Concepts

## What is Kubernetes?
Kubernetes (K8s) is an open-source container orchestration platform that automates the deployment, scaling, and management of containerized applications.

> 🇹🇭 Kubernetes คือแพลตฟอร์มระบบจัดการคอนเทนเนอร์แบบโอเพนซอร์ส ที่ช่วยในการจัดการ การปรับขนาด และการบริหารแอปพลิเคชันที่อยู่ในรูปแบบคอนเทนเนอร์

### Key Components of Kubernetes
1. **Pod** 
   - The smallest deployable unit in Kubernetes
   - Contains one or more containers
   - Shares network and storage resources
   
   > 🇹🇭 Pod คือหน่วยการทำงานที่เล็กที่สุดใน Kubernetes ซึ่งประกอบด้วยคอนเทนเนอร์หนึ่งตัวหรือมากกว่า และใช้ทรัพยากรเครือข่ายและพื้นที่จัดเก็บข้อมูลร่วมกัน

2. **Node**
   - Physical or virtual machine that runs pods
   - Contains container runtime and kubelet
   
   > 🇹🇭 Node คือเครื่องคอมพิวเตอร์จริงหรือเสมือนที่ใช้รัน pod โดยมีส่วนประกอบหลักคือ container runtime และ kubelet

3. **Cluster**
   - Set of nodes that run containerized applications
   - Consists of control plane and worker nodes
   
   > 🇹🇭 Cluster คือกลุ่มของ node ที่ทำงานร่วมกันเพื่อรันแอปพลิเคชัน ประกอบด้วยส่วนควบคุม (control plane) และ node สำหรับทำงาน

### Kubernetes Architecture
![Kubernetes Architecture](https://d33wubrfki0l68.cloudfront.net/2475489eaf20163ec0f54ddc1d92aa8d4c87c96b/e7c81/images/docs/components-of-kubernetes.svg)

> 🇹🇭 สถาปัตยกรรมของ Kubernetes แบ่งออกเป็น 2 ส่วนหลัก:
> 1. Control Plane (Master Node) - ส่วนควบคุมการทำงานทั้งหมด
> 2. Worker Nodes - ส่วนที่ใช้รันแอปพลิเคชันจริงๆ

#### Control Plane Components
1. **API Server**
   - Frontend for Kubernetes control plane
   - All communications go through here
   
   > 🇹🇭 เป็นส่วนที่ใช้ติดต่อกับ Kubernetes ทั้งหมด ทุกการสื่อสารต้องผ่านที่นี่

2. **etcd**
   - Distributed key-value store
   - Stores all cluster data
   
   > 🇹🇭 ฐานข้อมูลที่เก็บข้อมูลการทำงานทั้งหมดของ cluster

3. **Scheduler**
   - Assigns pods to nodes
   - Considers resources and constraints
   
   > 🇹🇭 ทำหน้าที่จัดสรร pod ไปยัง node ต่างๆ โดยพิจารณาจากทรัพยากรที่มี

4. **Controller Manager**
   - Runs controller processes
   - Handles node failures, replication
   
   > 🇹🇭 ควบคุมการทำงานต่างๆ เช่น การจัดการเมื่อ node ล้มเหลว

### Workload Resources

1. **Deployments**
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
        image: nginx:1.14.2
```

> 🇹🇭 Deployment ใช้สำหรับการ deploy แอปพลิเคชันแบบ stateless โดยสามารถกำหนดจำนวน replicas และจัดการ update ได้

2. **StatefulSets**
   - For stateful applications
   - Provides stable network identities
   
   > 🇹🇭 ใช้สำหรับแอปพลิเคชันที่ต้องเก็บข้อมูล state เช่น ฐานข้อมูล

3. **DaemonSets**
   - Runs pods on all nodes
   - Used for monitoring, logging
   
   > 🇹🇭 ใช้รัน pod บนทุก node เหมาะสำหรับระบบ monitoring

### Networking in Kubernetes

1. **Services**
   - ClusterIP (internal)
   - NodePort (external via node)
   - LoadBalancer (cloud provider)
   
   > 🇹🇭 Service คือวิธีการเข้าถึง pod จากภายนอก มีหลายรูปแบบขึ้นอยู่กับการใช้งาน

2. **Ingress**
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
            name: web
            port:
              number: 80
```

> 🇹🇭 Ingress ใช้จัดการการเข้าถึงจากภายนอกแบบ HTTP/HTTPS สามารถกำหนด routing rules ได้

### Storage

1. **Persistent Volumes (PV)**
   - Storage resource in cluster
   - Independent of pod lifecycle
   
   > 🇹🇭 พื้นที่เก็บข้อมูลถาวรที่ไม่หายไปเมื่อ pod ถูกลบ

2. **Persistent Volume Claims (PVC)**
   - Request for storage by user
   - Can be mapped to PV
   
   > 🇹🇭 การร้องขอพื้นที่เก็บข้อมูลจาก pod เพื่อใช้งาน PV

### Security

1. **RBAC (Role-Based Access Control)**
   - Manage authorization
   - Define roles and permissions
   
   > 🇹🇭 ระบบจัดการสิทธิ์การเข้าถึงใน Kubernetes

2. **Secrets**
   - Store sensitive data
   - Base64 encoded
   
   > 🇹🇭 ใช้เก็บข้อมูลที่เป็นความลับ เช่น password, token

### Monitoring and Logging

1. **Prometheus + Grafana**
   - Popular monitoring solution
   - Metrics collection and visualization
   
   > 🇹🇭 ระบบ monitoring ยอดนิยมสำหรับ Kubernetes

2. **ELK Stack**
   - Logging solution
   - Log aggregation and analysis
   
   > 🇹🇭 ระบบจัดการ log ที่นิยมใช้กับ Kubernetes

### Best Practices

1. **Resource Limits**
```yaml
resources:
  limits:
    cpu: "1"
    memory: "512Mi"
  requests:
    cpu: "0.5"
    memory: "256Mi"
```

> 🇹🇭 การกำหนดขีดจำกัดทรัพยากรเป็นสิ่งสำคัญเพื่อป้องกันการใช้ทรัพยากรมากเกินไป

2. **Health Checks**
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 3
  periodSeconds: 3
```

> 🇹🇭 การตรวจสอบสถานะของ pod เพื่อให้ระบบทำงานได้อย่างต่อเนื่อง

### Basic Commands
```bash
# View all pods
kubectl get pods

# View all nodes
kubectl get nodes

# Create deployment
kubectl create deployment nginx --image=nginx

# Scale deployment
kubectl scale deployment nginx --replicas=3
```

> 🇹🇭 คำสั่งพื้นฐานที่ใช้บ่อยใน Kubernetes เพื่อดูสถานะของ pod, node, สร้าง deployment และปรับขนาดของ deployment

### Troubleshooting Commands
```bash
# ดูรายละเอียด pod
kubectl describe pod <pod-name>

# ดู logs
kubectl logs <pod-name>

# เข้าไปใน container
kubectl exec -it <pod-name> -- /bin/bash

# ดูการใช้ทรัพยากร
kubectl top pods
kubectl top nodes
```

> 🇹🇭 คำสั่งที่ใช้บ่อยในการแก้ไขปัญหาใน Kubernetes

### Benefits of Using Kubernetes
- Automated container orchestration
- Self-healing capabilities
- Automatic scaling
- Load balancing
- Rolling updates and rollbacks

> 🇹🇭 ประโยชน์ของการใช้ Kubernetes
> - ระบบจัดการคอนเทนเนอร์อัตโนมัติ
> - ความสามารถในการซ่อมแซมตัวเอง
> - การปรับขนาดอัตโนมัติ
> - การกระจายโหลด
> - การอัพเดทและถอยกลับเวอร์ชันแบบต่อเนื่อง

## Advanced Topics for DevOps Engineers

### CI/CD Pipeline Integration
1. **GitOps Workflow**
   - ArgoCD
   - Flux CD
   - Jenkins X
   
   > 🇹🇭 การทำ CI/CD แบบ GitOps เป็นวิธีที่นิยมใน Kubernetes โดยใช้ Git เป็นแหล่งความจริงเดียว (Single Source of Truth)

2. **Helm Package Management**
```yaml
# Example Helm Chart structure
mychart/
  Chart.yaml
  values.yaml
  templates/
    deployment.yaml
    service.yaml
    ingress.yaml
```

> 🇹🇭 Helm เป็นเครื่องมือจัดการ Package สำหรับ Kubernetes ช่วยในการ deploy แอปพลิเคชันที่ซับซ้อน

### Infrastructure as Code (IaC)
1. **Terraform with Kubernetes**
```hcl
resource "kubernetes_deployment" "example" {
  metadata {
    name = "example"
    labels = {
      app = "example"
    }
  }
  spec {
    replicas = 3
    selector {
      match_labels = {
        app = "example"
      }
    }
    template {
      metadata {
        labels = {
          app = "example"
        }
      }
      spec {
        container {
          image = "nginx:1.21.6"
          name  = "example"
        }
      }
    }
  }
}
```

> 🇹🇭 การใช้ Terraform ในการจัดการ Infrastructure รวมถึง Kubernetes Cluster

### Advanced Networking
1. **Service Mesh**
   - Istio
   - Linkerd
   - Consul
   
   > 🇹🇭 Service Mesh ช่วยจัดการการสื่อสารระหว่าง Service ความปลอดภัย และ observability

2. **Network Policies**
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
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
```

> 🇹🇭 การกำหนดนโยบายเครือข่ายเพื่อควบคุมการเข้าถึงระหว่าง Pod

### High Availability & Disaster Recovery
1. **Multi-cluster Management**
   - Cluster Federation
   - Multi-cluster Service Mesh
   
   > 🇹🇭 การจัดการหลาย Cluster พร้อมกันเพื่อความเสถียรและการกู้คืน

2. **Backup Solutions**
   - Velero
   - Kasten K10
   
   > 🇹🇭 การสำรองข้อมูลและกู้คืน Kubernetes Cluster

### Advanced Monitoring & Observability
1. **Custom Metrics Pipeline**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: example-app
spec:
  selector:
    matchLabels:
      app: example
  endpoints:
  - port: web
```

2. **Distributed Tracing**
   - Jaeger
   - Zipkin
   - OpenTelemetry
   
   > 🇹🇭 การติดตามการทำงานของระบบแบบ Distributed

### Security Best Practices
1. **Pod Security Policies**
```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  seLinux:
    rule: RunAsAny
  runAsUser:
    rule: MustRunAsNonRoot
  fsGroup:
    rule: RunAsAny
```

2. **Image Security**
   - Container Image Scanning
   - Image Signing
   - Admission Controllers
   
   > 🇹🇭 การรักษาความปลอดภัยของ Container Image และการ Deploy

### Performance Optimization
1. **Resource Optimization**
   - Vertical Pod Autoscaling
   - Horizontal Pod Autoscaling
   - Cluster Autoscaling
   
   > 🇹🇭 การปรับแต่งประสิทธิภาพและทรัพยากรอัตโนมัติ

2. **Cost Optimization**
   - Spot Instances
   - Resource Quotas
   - Cost Analysis Tools
   
   > 🇹🇭 การจัดการต้นทุนและการใช้ทรัพยากรอย่างคุ้มค่า

### Chaos Engineering
1. **Chaos Mesh / Chaos Toolkit**
```yaml
apiVersion: chaos-mesh.org/v1alpha1
kind: PodChaos
metadata:
  name: pod-failure-example
spec:
  action: pod-failure
  mode: one
  selector:
    namespaces:
      - default
    labelSelectors:
      app: web
  duration: "30s"
```

> 🇹🇭 การทดสอบความทนทานของระบบด้วยการจำลองความล้มเหลว

### Cloud Provider Integration
1. **Cloud-specific Services**
   - AWS EKS
   - Google GKE
   - Azure AKS
   
   > 🇹🇭 การใช้งาน Kubernetes บน Cloud Provider ต่างๆ

2. **Cloud-native Features**
   - Auto-scaling Groups
   - Load Balancers
   - Storage Classes
   
   > 🇹🇭 การใช้คุณสมบัติพิเศษของแต่ละ Cloud Provider

### Development Workflow
1. **Local Development**
   - Minikube
   - Kind
   - k3d
   
   > 🇹🇭 การพัฒนาและทดสอบ Kubernetes ในเครื่อง Local

2. **Debug Techniques**
```bash
# Debug with ephemeral container
kubectl debug -it pod-name --image=busybox:1.28 --target=container-name

# Profile application performance
kubectl top pod pod-name --containers

# Network debugging
kubectl exec -it network-tools -- tcpdump -i any
```

> 🇹🇭 เทคนิคการ Debug และแก้ไขปัญหาขั้นสูง
