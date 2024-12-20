# แบบฝึกหัดที่ 10: การใช้ Terraform กับ Google Cloud
# Exercise 10: Using Terraform with Google Cloud

## โจทย์ | Problem
สร้างโครงสร้างพื้นฐานบน Google Cloud ด้วย Terraform โดยมีความต้องการดังนี้:

1. สร้าง Infrastructure as Code สำหรับ:
   - VPC Network และ Subnets
   - GKE Cluster
   - Cloud SQL
   - Load Balancer
   - Cloud Storage

2. จัดการ:
   - State management
   - Workspaces
   - Modules
   - Variables และ outputs

## เฉลย | Solution

### 1. โครงสร้างโปรเจค:
```
project/
├── environments/
│   ├── dev/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── terraform.tfvars
│   └── prod/
│       ├── main.tf
│       ├── variables.tf
│       └── terraform.tfvars
├── modules/
│   ├── gke/
│   ├── sql/
│   ├── vpc/
│   └── lb/
└── backend.tf
```

### 2. Backend Configuration (backend.tf):
```hcl
terraform {
  backend "gcs" {
    bucket = "tf-state-prod"
    prefix = "terraform/state"
  }
  required_providers {
    google = {
      source  = "hashicorp/google"
      version = "~> 4.0"
    }
  }
}
```

### 3. VPC Module (modules/vpc/main.tf):
```hcl
variable "project_id" {}
variable "region" {}
variable "environment" {}

resource "google_compute_network" "vpc" {
  name                    = "${var.environment}-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "subnet" {
  name          = "${var.environment}-subnet"
  ip_cidr_range = "10.0.0.0/24"
  region        = var.region
  network       = google_compute_network.vpc.id

  secondary_ip_range {
    range_name    = "pods"
    ip_cidr_range = "10.1.0.0/16"
  }

  secondary_ip_range {
    range_name    = "services"
    ip_cidr_range = "10.2.0.0/16"
  }
}

output "network_name" {
  value = google_compute_network.vpc.name
}

output "subnet_name" {
  value = google_compute_subnetwork.subnet.name
}
```

### 4. GKE Module (modules/gke/main.tf):
```hcl
variable "project_id" {}
variable "region" {}
variable "network_name" {}
variable "subnet_name" {}

resource "google_container_cluster" "primary" {
  name     = "primary-cluster"
  location = var.region

  # Remove default node pool
  remove_default_node_pool = true
  initial_node_count       = 1

  network    = var.network_name
  subnetwork = var.subnet_name

  ip_allocation_policy {
    cluster_secondary_range_name  = "pods"
    services_secondary_range_name = "services"
  }
}

resource "google_container_node_pool" "primary_nodes" {
  name       = "primary-node-pool"
  location   = var.region
  cluster    = google_container_cluster.primary.name
  node_count = 2

  node_config {
    machine_type = "e2-medium"
    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]
  }
}
```

### 5. Cloud SQL Module (modules/sql/main.tf):
```hcl
variable "project_id" {}
variable "region" {}
variable "network_name" {}

resource "google_sql_database_instance" "main" {
  name             = "main-instance"
  database_version = "POSTGRES_13"
  region           = var.region

  settings {
    tier = "db-f1-micro"
    ip_configuration {
      ipv4_enabled    = false
      private_network = var.network_name
    }
    backup_configuration {
      enabled = true
      start_time = "23:00"
    }
  }
}

resource "google_sql_database" "database" {
  name     = "app-database"
  instance = google_sql_database_instance.main.name
}

resource "google_sql_user" "users" {
  name     = "app-user"
  instance = google_sql_database_instance.main.name
  password = random_password.password.result
}

resource "random_password" "password" {
  length  = 16
  special = true
}
```

### 6. Load Balancer Module (modules/lb/main.tf):
```hcl
variable "project_id" {}
variable "region" {}

resource "google_compute_global_address" "default" {
  name = "global-lb-ip"
}

resource "google_compute_health_check" "default" {
  name = "http-basic-check"

  http_health_check {
    port = 80
  }
}

resource "google_compute_backend_service" "default" {
  name        = "backend-service"
  port_name   = "http"
  protocol    = "HTTP"
  timeout_sec = 10

  health_checks = [google_compute_health_check.default.id]
}

resource "google_compute_url_map" "default" {
  name            = "url-map"
  default_service = google_compute_backend_service.default.id
}

resource "google_compute_target_http_proxy" "default" {
  name    = "http-lb-proxy"
  url_map = google_compute_url_map.default.id
}

resource "google_compute_global_forwarding_rule" "default" {
  name       = "global-rule"
  target     = google_compute_target_http_proxy.default.id
  port_range = "80"
  ip_address = google_compute_global_address.default.address
}
```

### 7. Environment Configuration (environments/dev/main.tf):
```hcl
provider "google" {
  project = var.project_id
  region  = var.region
}

module "vpc" {
  source      = "../../modules/vpc"
  project_id  = var.project_id
  region      = var.region
  environment = terraform.workspace
}

module "gke" {
  source       = "../../modules/gke"
  project_id   = var.project_id
  region       = var.region
  network_name = module.vpc.network_name
  subnet_name  = module.vpc.subnet_name
}

module "sql" {
  source       = "../../modules/sql"
  project_id   = var.project_id
  region       = var.region
  network_name = module.vpc.network_name
}

module "lb" {
  source     = "../../modules/lb"
  project_id = var.project_id
  region     = var.region
}
```

## การทดสอบ | Testing
```bash
# Initialize Terraform
terraform init

# Create workspace
terraform workspace new dev

# Plan deployment
terraform plan -var-file="terraform.tfvars"

# Apply changes
terraform apply -var-file="terraform.tfvars"

# Verify resources
gcloud compute networks list
gcloud container clusters list
gcloud sql instances list
gcloud compute forwarding-rules list

# Destroy resources
terraform destroy -var-file="terraform.tfvars"
```

## เพิ่มเติม | Additional Notes
- ใช้ workspaces สำหรับแต่ละ environment
- เก็บ state ไว้ใน Cloud Storage
- ใช้ variables สำหรับค่าที่แตกต่างกันในแต่ละ environment
- ทำ remote state locking
- ใช้ data sources สำหรับ existing resources
