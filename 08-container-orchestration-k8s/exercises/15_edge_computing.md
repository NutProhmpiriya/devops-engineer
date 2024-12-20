# Exercise 15: Edge Computing with K3s and KubeEdge

## Task
สร้าง Edge Computing Infrastructure โดยใช้ K3s และ KubeEdge สำหรับ IoT และ Edge Devices

### Requirements
1. ติดตั้ง K3s Cluster
2. Set up KubeEdge
3. Configure Device Management
4. Implement Edge Intelligence
5. Set up Data Synchronization
6. Configure Edge Security

## Solution

1. K3s Installation (`k3s-config.yaml`):
```yaml
apiVersion: helm.cattle.io/v1
kind: HelmChartConfig
metadata:
  name: k3s-server
  namespace: kube-system
spec:
  valuesContent: |-
    global:
      systemDefaultRegistry: ""
    server:
      tls:
        selfSigned: true
      datastore:
        type: etcd
      serviceLB:
        enabled: true
      metrics:
        enabled: true
        serviceMonitor:
          enabled: true
```

2. KubeEdge Configuration (`kubeedge.yaml`):
```yaml
apiVersion: installer.kubeedge.io/v1alpha1
kind: CloudCore
metadata:
  name: cloudcore
  namespace: kubeedge
spec:
  modules:
    cloudHub:
      advertiseAddress:
        - "xxx.xxx.xxx.xxx"
      nodeLimit: 1000
    cloudStream:
      enable: true
      streamPort: 10003
      tunnelPort: 10004
    dynamicController:
      enable: true
    router:
      enable: true
---
apiVersion: installer.kubeedge.io/v1alpha1
kind: EdgeCore
metadata:
  name: edgecore
  namespace: kubeedge
spec:
  modules:
    edged:
      registerNode: true
      hostname: edge-device-001
      nodeIP: "yyy.yyy.yyy.yyy"
    edgeHub:
      websocket:
        server: "xxx.xxx.xxx.xxx:10000"
    eventBus:
      mqttMode: internal
      mqttQOS: 0
    metaManager:
      podStatusSyncInterval: 60
    serviceAccountToken: true
```

3. Device Model and Instance (`device-model.yaml`):
```yaml
apiVersion: devices.kubeedge.io/v1alpha2
kind: DeviceModel
metadata:
  name: sensor-model
  namespace: default
spec:
  properties:
    - name: temperature
      description: temperature in degree celsius
      type:
        int:
          accessMode: ReadOnly
          maximum: 100
          minimum: -20
    - name: humidity
      description: humidity percentage
      type:
        int:
          accessMode: ReadOnly
          maximum: 100
          minimum: 0
  protocol:
    modbus:
      slaveID: 1
---
apiVersion: devices.kubeedge.io/v1alpha2
kind: Device
metadata:
  name: temperature-sensor-01
  namespace: default
spec:
  deviceModelRef:
    name: sensor-model
  protocol:
    modbus:
      slaveID: 1
  nodeSelector:
    nodeSelectorTerms:
    - matchExpressions:
      - key: node-role.kubernetes.io/edge
        operator: In
        values:
        - "true"
  propertyVisitors:
    - propertyName: temperature
      modbus:
        register: CoilRegister
        offset: 1
        limit: 1
    - propertyName: humidity
      modbus:
        register: CoilRegister
        offset: 2
        limit: 1
```

4. Edge Intelligence Configuration (`edge-ai.yaml`):
```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: edge-ai-processor
  namespace: default
spec:
  selector:
    matchLabels:
      app: edge-ai
  template:
    metadata:
      labels:
        app: edge-ai
    spec:
      nodeSelector:
        node-role.kubernetes.io/edge: "true"
      containers:
      - name: ai-processor
        image: edge-ai-processor:latest
        resources:
          limits:
            memory: 1Gi
            cpu: "1"
          requests:
            memory: 512Mi
            cpu: "0.5"
        volumeMounts:
        - name: model-storage
          mountPath: /models
        env:
        - name: MODEL_PATH
          value: "/models/edge-model.pb"
        - name: INFERENCE_THRESHOLD
          value: "0.8"
      volumes:
      - name: model-storage
        persistentVolumeClaim:
          claimName: model-storage
```

5. Data Synchronization (`sync-config.yaml`):
```yaml
apiVersion: rules.kubeedge.io/v1
kind: RuleEndpoint
metadata:
  name: temperature-sync
  namespace: default
spec:
  ruleEndpointType: "rest"
  properties:
    url: "http://cloud-storage:8080/api/v1/data"
---
apiVersion: rules.kubeedge.io/v1
kind: Rule
metadata:
  name: sync-rule
  namespace: default
spec:
  source: "device-temperature-sensor-01"
  sourceResource: ["/temperature"]
  target: "temperature-sync"
  targetResource: ["/upload"]
  interval: 60
```

6. Edge Security Configuration (`edge-security.yaml`):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: edge-certificates
  namespace: kubeedge
type: Opaque
data:
  ca.crt: <base64-encoded-ca-cert>
  edge.crt: <base64-encoded-edge-cert>
  edge.key: <base64-encoded-edge-key>
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: edge-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      app: edge-device
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: kubeedge
    ports:
    - protocol: TCP
      port: 10000
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kubeedge
    ports:
    - protocol: TCP
      port: 10000
```

### การติดตั้งและ Configuration

1. ติดตั้ง K3s:
```bash
# Install K3s server
curl -sfL https://get.k3s.io | sh -

# Get kubeconfig
sudo cat /etc/rancher/k3s/k3s.yaml > ~/.kube/config

# Apply K3s configuration
kubectl apply -f k3s-config.yaml
```

2. ติดตั้ง KubeEdge:
```bash
# Install CloudCore
keadm init --advertise-address="xxx.xxx.xxx.xxx"

# Install EdgeCore on edge device
keadm join --cloudcore-ipport=xxx.xxx.xxx.xxx:10000 \
  --token=${TOKEN}

# Apply KubeEdge configuration
kubectl apply -f kubeedge.yaml
```

3. Set up Edge Devices:
```bash
# Create device model and instance
kubectl apply -f device-model.yaml

# Verify device registration
kubectl get devices
```

### การทดสอบ

1. Test Device Communication:
```bash
# Check device status
kubectl get device temperature-sensor-01 -o yaml

# Monitor device data
kubectl logs -f -l app=edge-ai
```

2. Test Edge Intelligence:
```bash
# Deploy AI model
kubectl apply -f edge-ai.yaml

# Monitor inference results
kubectl logs -f -l app=edge-ai-processor
```

3. Verify Data Synchronization:
```bash
# Check sync status
kubectl get rules

# Monitor data flow
kubectl logs -f -l app=sync-controller
```

### การ Monitor

1. Edge Metrics:
```bash
# Deploy edge monitoring
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: edge-monitor
spec:
  selector:
    matchLabels:
      app: edge-device
  endpoints:
  - port: metrics
EOF
```

2. Set up Alerts:
```bash
# Configure alert rules
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: edge-alerts
spec:
  groups:
  - name: edge.rules
    rules:
    - alert: EdgeDeviceOffline
      expr: up{job="edge-device"} == 0
      for: 5m
      labels:
        severity: critical
EOF
```

### Tips
- Implement proper edge device authentication
- Use lightweight containers for edge
- Monitor edge device resources
- Implement data buffering
- Regular security updates
- Test offline capabilities
- Monitor network connectivity
- Implement edge device updates
