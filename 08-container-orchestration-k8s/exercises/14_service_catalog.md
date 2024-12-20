# Exercise 14: Service Catalog and Cloud Services Integration

## Task
สร้าง Service Catalog สำหรับ Cloud Services บน Kubernetes โดยใช้ Open Service Broker API และ Crossplane

### Requirements
1. ติดตั้ง Service Catalog
2. Configure Cloud Providers (AWS, GCP, Azure)
3. Implement Service Provisioning
4. Set up Service Binding
5. Create Custom Resources
6. Implement Multi-cloud Orchestration

## Solution

1. Service Catalog Installation (`service-catalog.yaml`):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: service-catalog
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: service-catalog
  namespace: service-catalog
spec:
  interval: 5m
  chart:
    spec:
      chart: svc-cat/catalog
      version: 0.3.1
      sourceRef:
        kind: HelmRepository
        name: svc-cat
        namespace: flux-system
  values:
    controllerManager:
      healthcheck:
        enabled: true
    webhook:
      service:
        type: ClusterIP
```

2. Crossplane Installation (`crossplane.yaml`):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: crossplane-system
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: crossplane
  namespace: crossplane-system
spec:
  interval: 5m
  chart:
    spec:
      chart: crossplane-stable/crossplane
      version: 1.6.2
      sourceRef:
        kind: HelmRepository
        name: crossplane-stable
        namespace: flux-system
  values:
    metrics:
      enabled: true
    provider:
      packages:
        - crossplane/provider-aws
        - crossplane/provider-gcp
        - crossplane/provider-azure
```

3. Cloud Provider Configuration (`cloud-providers.yaml`):
```yaml
apiVersion: aws.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: aws-provider
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-creds
      key: credentials
---
apiVersion: gcp.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: gcp-provider
spec:
  projectID: my-project
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: gcp-creds
      key: credentials
---
apiVersion: azure.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: azure-provider
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: azure-creds
      key: credentials
```

4. Custom Resource Definition (`database-class.yaml`):
```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: CompositeResourceDefinition
metadata:
  name: xdatabases.database.example.org
spec:
  group: database.example.org
  names:
    kind: XDatabase
    plural: xdatabases
  claimNames:
    kind: Database
    plural: databases
  versions:
  - name: v1alpha1
    served: true
    referenceable: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              parameters:
                type: object
                properties:
                  storageGB:
                    type: integer
                  version:
                    type: string
                required:
                - storageGB
                - version
```

5. Composition for Multi-cloud Database (`database-composition.yaml`):
```yaml
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: database.aws
  labels:
    provider: aws
    service: database
spec:
  compositeTypeRef:
    apiVersion: database.example.org/v1alpha1
    kind: XDatabase
  writeConnectionSecretsToNamespace: crossplane-system
  resources:
  - name: rds
    base:
      apiVersion: database.aws.crossplane.io/v1beta1
      kind: RDSInstance
      spec:
        forProvider:
          region: us-west-2
          dbInstanceClass: db.t2.small
          masterUsername: admin
          engine: postgres
          engineVersion: "13.4"
          skipFinalSnapshotBeforeDeletion: true
          publiclyAccessible: false
    patches:
    - fromFieldPath: spec.parameters.storageGB
      toFieldPath: spec.forProvider.allocatedStorage
    - fromFieldPath: spec.parameters.version
      toFieldPath: spec.forProvider.engineVersion
---
apiVersion: apiextensions.crossplane.io/v1
kind: Composition
metadata:
  name: database.gcp
  labels:
    provider: gcp
    service: database
spec:
  compositeTypeRef:
    apiVersion: database.example.org/v1alpha1
    kind: XDatabase
  writeConnectionSecretsToNamespace: crossplane-system
  resources:
  - name: cloudsql
    base:
      apiVersion: database.gcp.crossplane.io/v1beta1
      kind: CloudSQLInstance
      spec:
        forProvider:
          databaseVersion: POSTGRES_13
          region: us-central1
          settings:
            tier: db-custom-1-3840
            ipConfiguration:
              ipv4Enabled: true
    patches:
    - fromFieldPath: spec.parameters.storageGB
      toFieldPath: spec.forProvider.settings.dataDiskSizeGb
```

6. Service Instance Request (`service-instance.yaml`):
```yaml
apiVersion: database.example.org/v1alpha1
kind: Database
metadata:
  name: production-db
  namespace: default
spec:
  parameters:
    storageGB: 20
    version: "13.4"
  compositionSelector:
    matchLabels:
      provider: aws
      service: database
---
apiVersion: database.example.org/v1alpha1
kind: Database
metadata:
  name: staging-db
  namespace: default
spec:
  parameters:
    storageGB: 10
    version: "13.4"
  compositionSelector:
    matchLabels:
      provider: gcp
      service: database
```

### การติดตั้งและ Configuration

1. ติดตั้ง Service Catalog และ Crossplane:
```bash
# Add Helm repositories
helm repo add svc-cat https://kubernetes-sigs.github.io/service-catalog
helm repo add crossplane-stable https://charts.crossplane.io/stable

# Install components
kubectl apply -f service-catalog.yaml
kubectl apply -f crossplane.yaml
```

2. Configure Cloud Providers:
```bash
# Create secrets for cloud credentials
kubectl create secret generic aws-creds \
  --from-file=credentials=./aws-credentials.json \
  -n crossplane-system

kubectl create secret generic gcp-creds \
  --from-file=credentials=./gcp-credentials.json \
  -n crossplane-system

kubectl create secret generic azure-creds \
  --from-file=credentials=./azure-credentials.json \
  -n crossplane-system

# Apply provider configurations
kubectl apply -f cloud-providers.yaml
```

3. Set up Custom Resources:
```bash
# Create CRDs and compositions
kubectl apply -f database-class.yaml
kubectl apply -f database-composition.yaml

# Request service instances
kubectl apply -f service-instance.yaml
```

### การทดสอบ

1. Verify Service Catalog:
```bash
# Check service classes
kubectl get clusterserviceclasses

# Check service plans
kubectl get clusterserviceplans
```

2. Test Database Provisioning:
```bash
# Check database status
kubectl get databases

# Get connection details
kubectl get secret production-db-connection \
  -o jsonpath='{.data.connection}' | base64 -d
```

3. Monitor Cloud Resources:
```bash
# Check AWS RDS instance
aws rds describe-db-instances

# Check GCP Cloud SQL instance
gcloud sql instances list
```

### การ Monitor

1. Set up Monitoring:
```bash
# Deploy Prometheus ServiceMonitor
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: crossplane
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: crossplane
  endpoints:
  - port: metrics
EOF
```

2. Create Alerts:
```bash
# Configure alert rules
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: crossplane-alerts
  namespace: monitoring
spec:
  groups:
  - name: crossplane
    rules:
    - alert: CrossplanePodCrashLooping
      expr: rate(kube_pod_container_status_restarts_total{namespace="crossplane-system"}[15m]) > 0
      for: 15m
      labels:
        severity: critical
      annotations:
        description: Crossplane pod is crash looping
EOF
```

### Tips
- Implement proper RBAC for service access
- Use resource quotas
- Monitor provisioning costs
- Implement backup strategies
- Regular testing of failover
- Document service catalog
- Monitor usage patterns
