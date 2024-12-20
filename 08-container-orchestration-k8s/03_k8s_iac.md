# Infrastructure as Code (IaC) with Kubernetes

## Terraform with Kubernetes
Terraform เป็นเครื่องมือในการจัดการ Infrastructure as Code ที่สามารถใช้กับ Kubernetes ได้อย่างมีประสิทธิภาพ

### Example Terraform Configuration
```hcl
# Provider configuration
provider "kubernetes" {
  config_path = "~/.kube/config"
}

# Create namespace
resource "kubernetes_namespace" "example" {
  metadata {
    name = "example"
  }
}

# Create deployment
resource "kubernetes_deployment" "example" {
  metadata {
    name      = "example"
    namespace = kubernetes_namespace.example.metadata[0].name
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
          
          resources {
            limits = {
              cpu    = "0.5"
              memory = "512Mi"
            }
            requests = {
              cpu    = "0.25"
              memory = "256Mi"
            }
          }
        }
      }
    }
  }
}

# Create service
resource "kubernetes_service" "example" {
  metadata {
    name      = "example"
    namespace = kubernetes_namespace.example.metadata[0].name
  }
  spec {
    selector = {
      app = kubernetes_deployment.example.metadata[0].labels.app
    }
    port {
      port        = 80
      target_port = 80
    }
    type = "ClusterIP"
  }
}
```

## Pulumi
Pulumi เป็นอีกทางเลือกหนึ่งในการทำ IaC โดยใช้ภาษาโปรแกรมมิ่งที่คุ้นเคย

### Example Pulumi Configuration (TypeScript)
```typescript
import * as pulumi from "@pulumi/pulumi";
import * as k8s from "@pulumi/kubernetes";

const appLabels = { app: "nginx" };

const deployment = new k8s.apps.v1.Deployment("nginx", {
    spec: {
        selector: { matchLabels: appLabels },
        replicas: 3,
        template: {
            metadata: { labels: appLabels },
            spec: {
                containers: [{
                    name: "nginx",
                    image: "nginx:1.21.6"
                }]
            }
        }
    }
});

export const name = deployment.metadata.name;
```

## Best Practices for IaC

1. **State Management**
   - Use remote state storage
   - Lock state during operations
   - Backup state regularly
   
   > 🇹🇭 การจัดการ State ที่ดีช่วยป้องกันปัญหาการทำงานพร้อมกันและการสูญหายของข้อมูล

2. **Code Organization**
   - Use modules for reusability
   - Separate environments
   - Follow DRY principle
   
   > 🇹🇭 การจัดระเบียบโค้ดที่ดีช่วยให้จัดการและบำรุงรักษาง่าย

3. **Security**
   - Use variables for sensitive data
   - Implement least privilege
   - Regular security audits
   
   > 🇹🇭 การรักษาความปลอดภัยต้องคำนึงถึงตั้งแต่ขั้นตอนการเขียนโค้ด

## Testing IaC

1. **Static Analysis**
   ```bash
   # Terraform
   terraform fmt
   terraform validate
   
   # Conftest
   conftest test deployment.yaml
   ```

2. **Policy as Code**
   ```hcl
   # Example OPA policy
   package kubernetes
   
   deny[msg] {
     input.kind == "Deployment"
     not input.spec.template.spec.securityContext.runAsNonRoot
     msg = "Containers must not run as root"
   }
   ```

## Monitoring and Compliance

1. **Drift Detection**
   - Regular state verification
   - Automated remediation
   - Compliance checking
   
   > 🇹🇭 การตรวจสอบความแตกต่างระหว่าง Infrastructure จริงกับโค้ดเป็นสิ่งสำคัญ

2. **Cost Management**
   - Resource tagging
   - Cost allocation
   - Budget alerts
   
   > 🇹🇭 การจัดการต้นทุนต้องทำตั้งแต่ขั้นตอนการเขียน IaC
