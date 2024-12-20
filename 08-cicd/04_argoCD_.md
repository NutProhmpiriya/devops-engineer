### ArgoCD: สิ่งที่ควรรู้และวิธีใช้งาน

**ArgoCD** เป็นเครื่องมือสำหรับ Continuous Deployment (CD) ที่ออกแบบมาสำหรับ Kubernetes โดยเฉพาะ โดยใช้แนวคิด **GitOps** เพื่อจัดการการปรับใช้แอปพลิเคชันใน Kubernetes Cluster โดย ArgoCD ดึงสถานะของแอปพลิเคชันจาก Git Repository และทำให้คลัสเตอร์ Kubernetes มีสถานะตรงกับที่กำหนดใน Git

---

## 1. **พื้นฐานของ ArgoCD**
### 1.1 Key Concepts:
- **GitOps**:
  - แนวทางที่เก็บสถานะของแอปพลิเคชันใน Git เช่น manifests, Helm charts, หรือ Kustomize
- **Application**:
  - หน่วยงานใน ArgoCD ที่แทนแอปพลิเคชันหนึ่งตัว
- **Sync**:
  - กระบวนการที่ ArgoCD นำ manifests จาก Git ไปปรับใช้ใน Kubernetes
- **Source of Truth**:
  - Git Repository ที่เก็บสถานะที่ต้องการของระบบ
- **Cluster**:
  - Kubernetes Cluster ที่ ArgoCD จะปรับใช้แอปพลิเคชัน

---

## 2. **คุณสมบัติของ ArgoCD**
- **Declarative Configuration**: กำหนดแอปพลิเคชันผ่านไฟล์ YAML
- **Self-Healing**: หากแอปพลิเคชันในคลัสเตอร์เปลี่ยนแปลงไปจาก Git ArgoCD จะ Sync กลับไปสู่สถานะที่ถูกต้อง
- **Multi-Cluster Management**: รองรับการจัดการหลาย Kubernetes Clusters
- **UI และ CLI**: มี Web UI สำหรับการดูสถานะ และ CLI สำหรับจัดการแอปพลิเคชัน
- **RBAC**: รองรับ Role-Based Access Control สำหรับการจัดการผู้ใช้งาน

---

## 3. **การติดตั้ง ArgoCD**
### 3.1 ติดตั้งใน Kubernetes Cluster:
ติดตั้งด้วยคำสั่ง `kubectl`:
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 3.2 ติดตั้ง CLI:
ดาวน์โหลด CLI จาก [ArgoCD CLI releases](https://github.com/argoproj/argo-cd/releases):
```bash
curl -sSL -o /usr/local/bin/argocd https://github.com/argoproj/argo-cd/releases/download/v2.8.0/argocd-linux-amd64
chmod +x /usr/local/bin/argocd
```

### 3.3 เข้าถึง UI:
- Forward port เพื่อเข้าถึง UI:
  ```bash
  kubectl port-forward svc/argocd-server -n argocd 8080:443
  ```
- เข้าสู่ระบบผ่าน `https://localhost:8080`
- ใช้ชื่อผู้ใช้ `admin` และรหัสผ่านที่ได้จากคำสั่ง:
  ```bash
  kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
  ```

---

## 4. **การสร้างแอปพลิเคชัน**
### 4.1 ตัวอย่างการสร้างแอปพลิเคชัน:
สร้างไฟล์ `application.yaml`:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/example/repo.git
    targetRevision: HEAD
    path: k8s
  destination:
    server: https://kubernetes.default.svc
    namespace: default
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

ปรับใช้แอปพลิเคชัน:
```bash
kubectl apply -f application.yaml
```

---

## 5. **การ Sync แอปพลิเคชัน**
- **Manual Sync**:
  รันคำสั่ง:
  ```bash
  argocd app sync my-app
  ```
- **Automatic Sync**:
  ใช้ `syncPolicy.automated` เพื่อให้ ArgoCD Sync อัตโนมัติ:
  ```yaml
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
  ```

---

## 6. **การจัดการหลาย Cluster**
เพิ่ม Kubernetes Cluster อื่นใน ArgoCD:
```bash
argocd cluster add <context-name>
```

ตรวจสอบ Cluster ที่เพิ่มแล้ว:
```bash
argocd cluster list
```

---

## 7. **การใช้งาน CLI**
### 7.1 Login ผ่าน CLI:
```bash
argocd login <argocd-server>
```

### 7.2 คำสั่งพื้นฐาน:
- ดูสถานะแอปพลิเคชัน:
  ```bash
  argocd app get my-app
  ```
- ลบแอปพลิเคชัน:
  ```bash
  argocd app delete my-app
  ```
- อัปเดตแอปพลิเคชัน:
  ```bash
  argocd app refresh my-app
  ```

---

## 8. **การผสาน ArgoCD กับ GitOps Workflow**
### 8.1 ใช้ร่วมกับ Helm:
ArgoCD รองรับ Helm charts:
```yaml
source:
  repoURL: https://charts.example.com
  targetRevision: 1.0.0
  chart: my-chart
```

### 8.2 ใช้ร่วมกับ Kustomize:
ArgoCD รองรับ Kustomize builds:
```yaml
source:
  repoURL: https://github.com/example/repo.git
  targetRevision: HEAD
  path: overlays/dev
```

---

## 9. **การรักษาความปลอดภัย**
- **RBAC**: กำหนดสิทธิ์ผู้ใช้ผ่าน Role-Based Access Control
- **SSO Integration**: รองรับ OAuth2, SAML, LDAP
- **Encryption**: ใช้ TLS สำหรับการเชื่อมต่อระหว่าง ArgoCD และ Cluster

---

## 10. **Best Practices**
1. **เก็บไฟล์ Config ใน Git**:
   - ใช้ Git เป็น Source of Truth สำหรับแอปพลิเคชัน
2. **ใช้ Namespaces แยกแอปพลิเคชัน**:
   - จัดการแอปพลิเคชันแต่ละตัวใน Namespace ของตนเอง
3. **เปิดใช้งาน Self-Healing**:
   - เปิดใช้ `selfHeal: true` เพื่อให้ ArgoCD ปรับสถานะให้ตรงกับ Git
4. **ใช้ Notifications**:
   - ติดตั้ง ArgoCD Notifications สำหรับแจ้งเตือนสถานะแอปพลิเคชัน
5. **ตรวจสอบ ArgoCD Logs**:
   - ใช้คำสั่ง `kubectl logs` เพื่อตรวจสอบการทำงานของ ArgoCD:
     ```bash
     kubectl logs -n argocd deployment/argocd-server
     ```

😊