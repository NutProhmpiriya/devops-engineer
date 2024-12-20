# Exercise 9: Advanced Security Hardening

## Task
Implement comprehensive security measures ใน Kubernetes cluster รวมถึง advanced RBAC, security policies, และ compliance monitoring

### Requirements
1. Configure Pod Security Standards
2. Implement OPA Gatekeeper
3. Set up Advanced RBAC
4. Configure Audit Logging
5. Implement Network Policies
6. Set up Vulnerability Scanning
7. Configure Secrets Management with Vault

## Solution

1. Pod Security Standards (`pod-security.yaml`):
```yaml
apiVersion: pod-security.kubernetes.io/v1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'MustRunAs'
    ranges:
      - min: 1
        max: 65535
  fsGroup:
    rule: 'MustRunAs'
    ranges:
      - min: 1
        max: 65535
  readOnlyRootFilesystem: true
```

2. OPA Gatekeeper Policies (`gatekeeper-policies.yaml`):
```yaml
apiVersion: constraints.gatekeeper.sh/v1beta1
kind: K8sRequiredLabels
metadata:
  name: ns-must-have-env
spec:
  match:
    kinds:
      - apiGroups: [""]
        kinds: ["Namespace"]
  parameters:
    labels: ["environment"]
---
apiVersion: templates.gatekeeper.sh/v1beta1
kind: ConstraintTemplate
metadata:
  name: k8srequiredregistry
spec:
  crd:
    spec:
      names:
        kind: K8sRequiredRegistry
      validation:
        openAPIV3Schema:
          properties:
            registries:
              type: array
              items:
                type: string
  targets:
    - target: admission.k8s.gatekeeper.sh
      rego: |
        package k8srequiredregistry
        violation[{"msg": msg}] {
          container := input.review.object.spec.containers[_]
          not startswith(container.image, concat("", [registry, "/"]))
          registry := input.parameters.registries[_]
          msg := sprintf("container <%v> registry <%v> is not allowed", [container.name, container.image])
        }
```

3. Advanced RBAC Configuration (`advanced-rbac.yaml`):
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
  resourceNames: ["frontend-*"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pod-reader-binding
subjects:
- kind: Group
  name: pod-readers
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

4. Audit Policy Configuration (`audit-policy.yaml`):
```yaml
apiVersion: audit.k8s.io/v1
kind: Policy
rules:
- level: Metadata
  resources:
  - group: ""
    resources: ["secrets", "configmaps"]
- level: RequestResponse
  resources:
  - group: ""
    resources: ["pods"]
  namespaces: ["kube-system", "default"]
- level: None
  users: ["system:kube-proxy"]
  resources:
  - group: "" # core
    resources: ["endpoints", "services", "services/status"]
```

5. Network Policy (`network-policy.yaml`):
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-monitoring
spec:
  podSelector:
    matchLabels:
      app: monitored
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 9090
```

6. Vault Configuration (`vault-config.yaml`):
```yaml
apiVersion: vault.banzaicloud.com/v1alpha1
kind: Vault
metadata:
  name: vault
spec:
  size: 3
  image: vault:1.9.0
  bankVaultsImage: banzaicloud/bank-vaults:latest
  
  # Vault configuration
  config:
    storage:
      file:
        path: "/vault/file"
    listener:
      tcp:
        address: "0.0.0.0:8200"
        tls_disable: true
    api_addr: http://vault.default:8200
    
  # Unsealing configuration
  unsealConfig:
    kubernetes:
      secretNamespace: default
      secretName: vault-keys
      
  # Authentication configuration
  authService:
    enabled: true
    
  # Service configuration    
  serviceMonitor:
    enabled: true
```

### การติดตั้งและ Configuration

1. ติดตั้ง OPA Gatekeeper:
```bash
kubectl apply -f https://raw.githubusercontent.com/open-policy-agent/gatekeeper/release-3.7/deploy/gatekeeper.yaml

# Apply constraints
kubectl apply -f gatekeeper-policies.yaml
```

2. Configure Audit Logging:
```bash
# Update kube-apiserver configuration
sudo vi /etc/kubernetes/manifests/kube-apiserver.yaml

# Add audit policy configuration
--audit-policy-file=/etc/kubernetes/audit-policy.yaml
--audit-log-path=/var/log/kubernetes/audit.log
```

3. ติดตั้ง Vault:
```bash
# Add Helm repo
helm repo add hashicorp https://helm.releases.hashicorp.com

# Install Vault
helm install vault hashicorp/vault \
  --namespace vault \
  --create-namespace \
  -f vault-values.yaml
```

### การทดสอบ

1. Test Pod Security:
```bash
# Try to create privileged pod
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: privileged-pod
spec:
  containers:
  - name: nginx
    image: nginx
    securityContext:
      privileged: true
EOF
```

2. Test OPA Policies:
```bash
# Try to create namespace without required labels
kubectl create namespace test-namespace

# Try to use unauthorized registry
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: unauthorized-registry
spec:
  containers:
  - name: nginx
    image: unauthorized.registry/nginx
EOF
```

3. Test Network Policies:
```bash
# Test connectivity
kubectl run test-pod --image=busybox -- wget -O- http://service-name

# Test isolation
kubectl run isolated-pod --image=busybox -- wget -O- http://blocked-service
```

### การ Monitor

1. Security Audit:
```bash
# Check audit logs
sudo tail -f /var/log/kubernetes/audit.log

# Analyze with jq
cat /var/log/kubernetes/audit.log | jq 'select(.user.username != "system:kube-proxy")'
```

2. Compliance Monitoring:
```bash
# Check OPA violations
kubectl get constraints

# Check PSP violations
kubectl get events --field-selector reason=FailedCreate
```

### Tips
- Regular security scanning with tools like Trivy
- Implement image signing with Notary
- Use admission controllers for additional security
- Monitor security-related events
- Regular security audits and updates
