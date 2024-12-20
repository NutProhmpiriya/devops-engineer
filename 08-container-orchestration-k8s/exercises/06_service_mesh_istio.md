# Exercise 6: Service Mesh with Istio

## Task
ติดตั้งและ configure Istio Service Mesh พร้อมทำ Blue-Green Deployment และ Canary Testing

### Requirements
1. ติดตั้ง Istio และ configure
2. Deploy microservices application (3 services)
3. ทำ Blue-Green Deployment
4. ทำ Canary Testing with traffic splitting
5. Configure mutual TLS และ circuit breaking
6. Setup monitoring with Kiali, Jaeger, และ Prometheus

## Solution

1. สร้าง Namespace และ enable Istio injection:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: microservices
  labels:
    istio-injection: enabled
```

2. สร้าง Microservices (`microservices.yaml`):
```yaml
# Frontend Service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend
  namespace: microservices
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
        version: v1
    spec:
      containers:
      - name: frontend
        image: nginx:1.19
        ports:
        - containerPort: 80
---
# API Service (Blue version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-blue
  namespace: microservices
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
      version: blue
  template:
    metadata:
      labels:
        app: api
        version: blue
    spec:
      containers:
      - name: api
        image: your-api-image:blue
        ports:
        - containerPort: 8080
---
# API Service (Green version)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-green
  namespace: microservices
spec:
  replicas: 2
  selector:
    matchLabels:
      app: api
      version: green
  template:
    metadata:
      labels:
        app: api
        version: green
    spec:
      containers:
      - name: api
        image: your-api-image:green
        ports:
        - containerPort: 8080
---
# Database Service
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database
  namespace: microservices
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: database
        image: postgres:13
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: password
```

3. Virtual Service สำหรับ Traffic Management (`virtual-service.yaml`):
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: api-routing
  namespace: microservices
spec:
  hosts:
  - api.example.com
  gateways:
  - api-gateway
  http:
  - route:
    - destination:
        host: api
        subset: blue
      weight: 90
    - destination:
        host: api
        subset: green
      weight: 10
```

4. Destination Rules สำหรับ Circuit Breaking (`destination-rules.yaml`):
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: DestinationRule
metadata:
  name: api-circuit-breaker
  namespace: microservices
spec:
  host: api
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 100
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 3
      interval: 30s
      baseEjectionTime: 30s
  subsets:
  - name: blue
    labels:
      version: blue
  - name: green
    labels:
      version: green
```

5. Gateway Configuration (`gateway.yaml`):
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: api-gateway
  namespace: microservices
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "api.example.com"
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: MUTUAL
      serverCertificate: /etc/certs/server-cert.pem
      privateKey: /etc/certs/private-key.pem
      caCertificates: /etc/certs/ca-cert.pem
    hosts:
    - "api.example.com"
```

### คำสั่งที่ใช้

1. ติดตั้ง Istio:
```bash
# ติดตั้ง istioctl
curl -L https://istio.io/downloadIstio | sh -

# ติดตั้ง Istio core components
istioctl install --set profile=demo

# ติดตั้ง addons
kubectl apply -f samples/addons/
```

2. Deploy applications:
```bash
# Create namespace and enable injection
kubectl apply -f namespace.yaml

# Deploy microservices
kubectl apply -f microservices.yaml
kubectl apply -f virtual-service.yaml
kubectl apply -f destination-rules.yaml
kubectl apply -f gateway.yaml
```

3. Monitor และ Visualize:
```bash
# Start Kiali dashboard
istioctl dashboard kiali

# Start Jaeger dashboard
istioctl dashboard jaeger

# View metrics in Grafana
istioctl dashboard grafana
```

### การทดสอบ

1. ทดสอบ Blue-Green Deployment:
```bash
# ทดสอบ traffic routing
for i in {1..100}; do
  curl -H "Host: api.example.com" http://$GATEWAY_IP/
done

# ตรวจสอบการกระจาย traffic
kubectl -n microservices logs -l app=api -c istio-proxy
```

2. ทดสอบ Circuit Breaking:
```bash
# Generate load for circuit breaking
kubectl run fortio -n microservices \
  --image=fortio/fortio \
  -- load -c 3 -qps 0 -t 10s "http://api:8080/health"
```

3. ตรวจสอบ mTLS:
```bash
# Verify mTLS
istioctl authn tls-check api.microservices.svc.cluster.local
```

### การ Monitor

1. Kiali Dashboard:
- ดู Service Graph
- ตรวจสอบ Traffic Flow
- Monitor Health

2. Jaeger:
- ดู Distributed Traces
- Analyze Latency
- Debug Issues

3. Grafana:
- Monitor Performance
- View Custom Metrics
- Set up Alerts

### Tips
- ใช้ `istioctl analyze` เพื่อตรวจสอบ configuration
- ตรวจสอบ Envoy proxies: `istioctl proxy-status`
- Debug ด้วย `istioctl proxy-config`
- Monitor Istio control plane: `kubectl -n istio-system logs -l app=istiod`
