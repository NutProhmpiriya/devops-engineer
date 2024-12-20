# Exercise 5: Create GKE Cluster
# แบบฝึกหัดที่ 5: สร้าง GKE Cluster

## Objective (วัตถุประสงค์)
Create a Google Kubernetes Engine (GKE) cluster with proper node pools and networking.
สร้างคลัสเตอร์ Google Kubernetes Engine (GKE) พร้อม node pools และเครือข่ายที่เหมาะสม

## Requirements (ข้อกำหนด)
1. Create a GKE cluster with:
   - Private nodes
   - Multiple node pools
   - Workload identity
   - Network policies
2. Configure autoscaling
3. Set up monitoring and logging

## Expected Solution (แนวทางการแก้ปัญหา)

```hcl
# main.tf
provider "google" {
  project = "YOUR_PROJECT_ID"
  region  = "asia-southeast1"
}

# VPC Network
resource "google_compute_network" "gke_network" {
  name                    = "gke-network"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "gke_subnet" {
  name          = "gke-subnet"
  ip_cidr_range = "10.0.0.0/24"
  network       = google_compute_network.gke_network.id
  region        = "asia-southeast1"

  secondary_ip_range {
    range_name    = "pod-ranges"
    ip_cidr_range = "10.1.0.0/16"
  }

  secondary_ip_range {
    range_name    = "service-ranges"
    ip_cidr_range = "10.2.0.0/16"
  }
}

# GKE Cluster
resource "google_container_cluster" "primary" {
  name     = "primary-cluster"
  location = "asia-southeast1"

  # We can't create a cluster with no node pool defined, but we want to only use
  # separately managed node pools. So we create the smallest possible default
  # node pool and immediately delete it.
  remove_default_node_pool = true
  initial_node_count       = 1

  network    = google_compute_network.gke_network.name
  subnetwork = google_compute_subnetwork.gke_subnet.name

  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false
    master_ipv4_cidr_block = "172.16.0.0/28"
  }

  ip_allocation_policy {
    cluster_secondary_range_name  = "pod-ranges"
    services_secondary_range_name = "service-ranges"
  }

  workload_identity_config {
    workload_pool = "${var.project_id}.svc.id.goog"
  }

  network_policy {
    enabled = true
  }

  addons_config {
    network_policy_config {
      disabled = false
    }
  }

  monitoring_config {
    enable_components = ["SYSTEM_COMPONENTS", "WORKLOADS"]
  }

  logging_config {
    enable_components = ["SYSTEM_COMPONENTS", "WORKLOADS"]
  }
}

# Node Pools
resource "google_container_node_pool" "general" {
  name       = "general"
  cluster    = google_container_cluster.primary.id
  node_count = 1

  autoscaling {
    min_node_count = 1
    max_node_count = 5
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }

  node_config {
    preemptible  = false
    machine_type = "e2-standard-2"

    service_account = google_service_account.gke_sa.email

    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    workload_metadata_config {
      mode = "GKE_METADATA"
    }
  }
}

resource "google_container_node_pool" "spot" {
  name    = "spot"
  cluster = google_container_cluster.primary.id

  autoscaling {
    min_node_count = 0
    max_node_count = 10
  }

  management {
    auto_repair  = true
    auto_upgrade = true
  }

  node_config {
    preemptible  = true
    machine_type = "e2-standard-2"

    service_account = google_service_account.gke_sa.email

    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]

    workload_metadata_config {
      mode = "GKE_METADATA"
    }
  }
}

# Service Account
resource "google_service_account" "gke_sa" {
  account_id   = "gke-sa"
  display_name = "GKE Service Account"
}
```

## Variables (ตัวแปร)
```hcl
# variables.tf
variable "project_id" {
  description = "Google Cloud Project ID"
  type        = string
}
```

## Outputs (ผลลัพธ์)
```hcl
# outputs.tf
output "cluster_name" {
  value = google_container_cluster.primary.name
}

output "cluster_endpoint" {
  value = google_container_cluster.primary.endpoint
}

output "cluster_ca_certificate" {
  value     = google_container_cluster.primary.master_auth[0].cluster_ca_certificate
  sensitive = true
}
```

## Validation (การตรวจสอบ)
1. Verify cluster creation and node pools
2. Test private node access
3. Verify workload identity configuration
4. Test network policies
5. Check monitoring and logging setup

## Additional Challenge (ความท้าทายเพิ่มเติม)
- Implement node pool taints and tolerations
- Set up cluster autoscaler
- Configure binary authorization
- Implement pod security policies
- Set up Cloud Armor protection
