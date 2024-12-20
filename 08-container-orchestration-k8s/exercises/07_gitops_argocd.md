# Exercise 7: GitOps with ArgoCD

## Task
ติดตั้งและ configure ArgoCD เพื่อทำ GitOps พร้อมทั้งสร้าง CI/CD pipeline แบบ complete

### Requirements
1. ติดตั้ง ArgoCD
2. สร้าง Git repository structure
3. Configure multi-environment deployments (dev, staging, prod)
4. ทำ Automated rollbacks
5. Implement secret management
6. Set up notifications
7. Configure RBAC

## Solution

1. สร้าง Git Repository Structure:
```
.
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   └── patch.yaml
│   └── prod/
│       ├── kustomization.yaml
│       └── patch.yaml
└── argocd/
    ├── applications/
    │   ├── dev.yaml
    │   ├── staging.yaml
    │   └── prod.yaml
    └── projects/
        └── myapp-project.yaml
```

2. Base Kustomization (`base/kustomization.yaml`):
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- deployment.yaml
- service.yaml
commonLabels:
  app: myapp
```

3. Environment Overlay (`overlays/prod/kustomization.yaml`):
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
bases:
- ../../base
patches:
- path: patch.yaml
namePrefix: prod-
namespace: prod
```

4. ArgoCD Application (`argocd/applications/prod.yaml`):
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-prod
  namespace: argocd
spec:
  project: myapp
  source:
    repoURL: https://github.com/your-org/your-repo.git
    targetRevision: HEAD
    path: overlays/prod
  destination:
    server: https://kubernetes.default.svc
    namespace: prod
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
    - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
```

5. ArgoCD Project (`argocd/projects/myapp-project.yaml`):
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: myapp
  namespace: argocd
spec:
  description: MyApp Project
  sourceRepos:
  - '*'
  destinations:
  - namespace: dev
    server: https://kubernetes.default.svc
  - namespace: staging
    server: https://kubernetes.default.svc
  - namespace: prod
    server: https://kubernetes.default.svc
  clusterResourceWhitelist:
  - group: '*'
    kind: '*'
  namespaceResourceBlacklist:
  - group: ''
    kind: ResourceQuota
  - group: ''
    kind: LimitRange
  roles:
  - name: project-admin
    description: Project Admin
    policies:
    - p, proj:myapp:project-admin, applications, *, myapp/*, allow
    groups:
    - myapp-admins
```

6. Notification Configuration (`argocd/notifications.yaml`):
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-notifications-cm
data:
  trigger.on-sync-succeeded: |
    - when: app.status.operationState.phase in ['Succeeded']
      send: [slack]
  trigger.on-sync-failed: |
    - when: app.status.operationState.phase in ['Error', 'Failed']
      send: [slack, email]
  template.slack: |
    message: |
      Application {{.app.metadata.name}} sync {{.app.status.operationState.phase}}
      Revision: {{.app.status.sync.revision}}
  service.slack: |
    token: $slack-token
    channel: deployments
```

### การติดตั้ง

1. ติดตั้ง ArgoCD:
```bash
# Create namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Get admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

# Port forward
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

2. Configure Git Repository:
```bash
# Add private repository
argocd repo add https://github.com/your-org/your-repo.git --username git --password <personal-access-token>

# Add public repository
argocd repo add https://github.com/your-org/your-repo.git
```

3. Deploy Applications:
```bash
# Create project
kubectl apply -f argocd/projects/myapp-project.yaml

# Deploy applications
kubectl apply -f argocd/applications/
```

### การทำ Progressive Delivery

1. Canary Deployment:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: myapp
spec:
  replicas: 5
  strategy:
    canary:
      steps:
      - setWeight: 20
      - pause: {duration: 1h}
      - setWeight: 40
      - pause: {duration: 1h}
      - setWeight: 60
      - pause: {duration: 1h}
      - setWeight: 80
      - pause: {duration: 1h}
```

2. Automated Rollback:
```yaml
spec:
  progressDeadlineSeconds: 600
  analysis:
    templates:
    - templateName: success-rate
    args:
    - name: service-name
      value: myapp-prod
```

### การ Monitor

1. ArgoCD Dashboard:
- Application Health
- Sync Status
- Deployment History

2. Notifications:
- Slack Integration
- Email Alerts
- Custom Webhooks

### Tips
- ใช้ `argocd app diff` เพื่อตรวจสอบการเปลี่ยนแปลง
- Set up automated pruning ของ resources ที่ไม่ใช้
- ใช้ ApplicationSets สำหรับ multi-cluster deployments
- Configure SSO สำหรับ authentication
- Regular backup ของ ArgoCD configurations
