# Exercise 8: Multi-cluster Federation and Management

## Task
สร้างและจัดการ Multi-cluster Kubernetes environment พร้อมทั้ง implement disaster recovery

### Requirements
1. Set up Kubernetes Federation
2. Configure Cross-cluster Service Discovery
3. Implement Global Load Balancing
4. Set up Disaster Recovery
5. Manage Multi-cluster Security
6. Monitor Multiple Clusters

## Solution

1. Federation Control Plane Setup (`federation-namespace.yaml`):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: kube-federation-system
---
apiVersion: core.kubefed.io/v1beta1
kind: KubeFedConfig
metadata:
  name: kubefed
  namespace: kube-federation-system
spec:
  scope: Namespaced
  controllerDuration:
    availableDelay: 20s
    unavailableDelay: 60s
  leaderElect:
    resourceLock: configmaps
    retryPeriod: 3s
    leaseDuration: 15s
    renewDeadline: 10s
```

2. Cluster Registration (`cluster-registration.yaml`):
```yaml
apiVersion: core.kubefed.io/v1beta1
kind: KubeFedCluster
metadata:
  name: cluster1
  namespace: kube-federation-system
spec:
  apiEndpoint: https://cluster1.example.com
  secretRef:
    name: cluster1-secret
---
apiVersion: core.kubefed.io/v1beta1
kind: KubeFedCluster
metadata:
  name: cluster2
  namespace: kube-federation-system
spec:
  apiEndpoint: https://cluster2.example.com
  secretRef:
    name: cluster2-secret
```

3. Federated Service Type (`federated-service.yaml`):
```yaml
apiVersion: types.kubefed.io/v1beta1
kind: FederatedServiceType
metadata:
  name: federated-service
spec:
  template:
    spec:
      type: LoadBalancer
      ports:
      - port: 80
        targetPort: 8080
  placement:
    clusters:
    - name: cluster1
    - name: cluster2
  overrides:
  - clusterName: cluster1
    clusterOverrides:
    - path: "/spec/ports/0/port"
      value: 8080
```

4. Global DNS Configuration (`global-dns.yaml`):
```yaml
apiVersion: multiclusterdns.kubefed.io/v1alpha1
kind: Domain
metadata:
  name: example.com
  namespace: kube-federation-system
spec:
  domain: example.com
---
apiVersion: multiclusterdns.kubefed.io/v1alpha1
kind: ServiceDNSRecord
metadata:
  name: myapp
  namespace: kube-federation-system
spec:
  domainRef: example.com
  recordTTL: 300
```

5. Disaster Recovery Configuration (`disaster-recovery.yaml`):
```yaml
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-backup
  namespace: velero
spec:
  schedule: "0 0 * * *"
  template:
    includedNamespaces:
    - "*"
    excludedNamespaces:
    - kube-system
    includeClusterResources: true
    hooks:
      resources:
      - name: backup-hook
        includedNamespaces:
        - "*"
        labelSelector:
          matchLabels:
            app: myapp
---
apiVersion: velero.io/v1
kind: BackupStorageLocation
metadata:
  name: default
  namespace: velero
spec:
  provider: aws
  objectStorage:
    bucket: my-backup-bucket
  config:
    region: us-west-1
```

6. Multi-cluster Monitoring (`prometheus-federation.yaml`):
```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 2
  serviceAccountName: prometheus
  serviceMonitorSelector:
    matchLabels:
      team: frontend
  additionalScrapeConfigs:
    name: additional-scrape-configs
    key: prometheus-additional.yaml
  externalLabels:
    cluster: primary
  remoteWrite:
  - url: "http://thanos-receive:19291/api/v1/receive"
```

### การติดตั้งและ Configuration

1. ติดตั้ง Federation:
```bash
# Install kubefedctl
curl -LO https://github.com/kubernetes-sigs/kubefed/releases/download/v0.8.1/kubefedctl-0.8.1-linux-amd64.tgz
tar xzf kubefedctl-0.8.1-linux-amd64.tgz
chmod +x kubefedctl
sudo mv kubefedctl /usr/local/bin/

# Initialize Federation
kubefedctl init federation \
  --host-cluster-context=cluster1-context \
  --federation-namespace=kube-federation-system

# Join clusters
kubefedctl join cluster1 --cluster-context cluster1-context --host-cluster-context cluster1-context
kubefedctl join cluster2 --cluster-context cluster2-context --host-cluster-context cluster1-context
```

2. Configure Global Load Balancing:
```bash
# Install MetalLB
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.12.1/manifests/metallb.yaml

# Configure address pool
kubectl apply -f metallb-config.yaml
```

3. Set up Disaster Recovery:
```bash
# Install Velero
velero install \
  --provider aws \
  --plugins velero/velero-plugin-for-aws:v1.2.0 \
  --bucket my-backup-bucket \
  --backup-location-config region=us-west-1 \
  --snapshot-location-config region=us-west-1 \
  --secret-file ./credentials-velero
```

### การทดสอบ

1. Test Federation:
```bash
# Check cluster status
kubefedctl check
kubectl -n kube-federation-system get kubefedclusters

# Deploy federated application
kubectl apply -f federated-deployment.yaml
```

2. Test Disaster Recovery:
```bash
# Create backup
velero backup create my-backup --include-namespaces=default

# Simulate disaster
kubectl delete namespace default

# Restore
velero restore create --from-backup my-backup
```

3. Verify Global DNS:
```bash
# Test DNS resolution
dig myapp.example.com

# Check service availability
curl -v http://myapp.example.com
```

### การ Monitor

1. Set up Prometheus Federation:
```bash
# Deploy Thanos
kubectl apply -f thanos-deployment.yaml

# Configure remote write
kubectl apply -f prometheus-federation.yaml
```

2. Configure Grafana:
```bash
# Add Thanos data source
kubectl apply -f grafana-datasource.yaml

# Import multi-cluster dashboard
kubectl apply -f grafana-dashboard.yaml
```

### Tips
- ใช้ `kubefedctl enable` เพื่อ enable federation สำหรับ resource types
- ตรวจสอบ network connectivity ระหว่าง clusters
- Monitor federation control plane logs
- Regular testing ของ disaster recovery procedures
- Implement proper RBAC across clusters
