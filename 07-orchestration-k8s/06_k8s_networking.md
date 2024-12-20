# Kubernetes Networking

## Network Fundamentals
1. **Pod Networking**
   - Container Network Interface (CNI)
   - Pod-to-Pod Communication
   - Network Plugins (Calico, Flannel)
   
   > 🇹🇭 พื้นฐานการสื่อสารระหว่าง Pod ใน Kubernetes

2. **Service Types**
```yaml
# ClusterIP Service
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  type: ClusterIP

# LoadBalancer Service
apiVersion: v1
kind: Service
metadata:
  name: my-loadbalancer
spec:
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
  type: LoadBalancer
```

## Service Mesh
1. **Istio**
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: reviews-route
spec:
  hosts:
  - reviews
  http:
  - match:
    - headers:
        end-user:
          exact: jason
    route:
    - destination:
        host: reviews
        subset: v2
  - route:
    - destination:
        host: reviews
        subset: v1
```

2. **Features**
   - Traffic Management
   - Security
   - Observability
   
   > 🇹🇭 Service Mesh ช่วยจัดการการสื่อสารระหว่าง Service ได้อย่างมีประสิทธิภาพ

## Ingress Controllers
1. **Nginx Ingress**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /testpath
        pathType: Prefix
        backend:
          service:
            name: test
            port:
              number: 80
```

2. **SSL/TLS Configuration**
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: testsecret-tls
  namespace: default
data:
  tls.crt: base64_encoded_cert
  tls.key: base64_encoded_key
type: kubernetes.io/tls
```

## Network Policies
1. **Pod Isolation**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
```

2. **Egress Control**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: limit-egress
spec:
  podSelector:
    matchLabels:
      app: myapp
  policyTypes:
  - Egress
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
```

## DNS and Service Discovery
1. **CoreDNS Configuration**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns
  namespace: kube-system
data:
  Corefile: |
    .:53 {
        errors
        health
        kubernetes cluster.local in-addr.arpa ip6.arpa {
           pods insecure
           upstream
           fallthrough in-addr.arpa ip6.arpa
        }
        prometheus :9153
        forward . /etc/resolv.conf
        cache 30
        loop
        reload
        loadbalance
    }
```

2. **Service Discovery**
   - DNS Records
   - Environment Variables
   - Service Resolution
   
   > 🇹🇭 การค้นหา Service ใน Kubernetes ทำได้หลายวิธี

## Load Balancing
1. **Internal Load Balancing**
   - Service Load Balancing
   - Session Affinity
   - Load Balancing Algorithms
   
   > 🇹🇭 การกระจายโหลดภายใน Cluster

2. **External Load Balancing**
   - Cloud Provider Integration
   - MetalLB for Bare Metal
   - Global Load Balancing
   
   > 🇹🇭 การกระจายโหลดสำหรับการเข้าถึงจากภายนอก

## Network Troubleshooting
1. **Common Tools**
```bash
# Network debugging
kubectl run tmp-shell --rm -i --tty --image nicolaka/netshoot -- /bin/bash

# DNS debugging
kubectl run tmp-dns --rm -i --tty --image=busybox -- nslookup kubernetes.default

# Port forwarding
kubectl port-forward service/my-service 8080:80
```

2. **Monitoring Network**
   - Network Metrics
   - Traffic Analysis
   - Latency Monitoring
   
   > 🇹🇭 การตรวจสอบและแก้ไขปัญหาเครือข่าย
