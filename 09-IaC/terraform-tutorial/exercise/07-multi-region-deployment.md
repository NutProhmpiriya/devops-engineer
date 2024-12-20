# Exercise 7: Multi-Region High Availability Deployment
# แบบฝึกหัดที่ 7: การ Deploy แบบ Multi-Region เพื่อความพร้อมใช้งานสูง

## Objective (วัตถุประสงค์)
Create a highly available application deployment across multiple regions with automated failover and global load balancing.
สร้างการ deploy แอปพลิเคชันแบบ highly available ข้ามหลายภูมิภาคพร้อมระบบ failover อัตโนมัติและ global load balancing

## Architecture Overview (ภาพรวมสถาปัตยกรรม)
1. Multi-region GKE clusters
2. Cloud Spanner for global database
3. Global Load Balancer with Cloud CDN
4. Multi-region Cloud Storage
5. Cloud DNS with geo-location routing
6. Disaster Recovery procedures

## Requirements (ข้อกำหนด)
1. Deploy to asia-southeast1 and us-central1
2. Implement automated failover
3. Ensure data consistency across regions
4. Set up global monitoring
5. Configure disaster recovery procedures

## Expected Solution (แนวทางการแก้ปัญหา)

```hcl
# Define regions
locals {
  regions = {
    primary   = "asia-southeast1"
    secondary = "us-central1"
  }
}

# VPC for each region
resource "google_compute_network" "vpc" {
  for_each                = local.regions
  name                    = "vpc-${each.key}"
  auto_create_subnetworks = false
}

# Subnets
resource "google_compute_subnetwork" "subnet" {
  for_each      = local.regions
  name          = "subnet-${each.key}"
  ip_cidr_range = each.key == "primary" ? "10.0.0.0/16" : "10.1.0.0/16"
  network       = google_compute_network.vpc[each.key].id
  region        = each.value

  secondary_ip_range {
    range_name    = "pods"
    ip_cidr_range = each.key == "primary" ? "192.168.0.0/20" : "192.168.16.0/20"
  }

  secondary_ip_range {
    range_name    = "services"
    ip_cidr_range = each.key == "primary" ? "192.168.32.0/20" : "192.168.48.0/20"
  }
}

# GKE Clusters
resource "google_container_cluster" "gke" {
  for_each = local.regions
  name     = "gke-${each.key}"
  location = each.value

  network    = google_compute_network.vpc[each.key].name
  subnetwork = google_compute_subnetwork.subnet[each.key].name

  private_cluster_config {
    enable_private_nodes    = true
    enable_private_endpoint = false
    master_ipv4_cidr_block = each.key == "primary" ? "172.16.0.0/28" : "172.16.0.16/28"
  }

  master_authorized_networks_config {
    cidr_blocks {
      cidr_block   = "0.0.0.0/0"
      display_name = "all"
    }
  }

  ip_allocation_policy {
    cluster_secondary_range_name  = "pods"
    services_secondary_range_name = "services"
  }
}

# Cloud Spanner
resource "google_spanner_instance" "main" {
  name         = "multi-region-spanner"
  config       = "asia-southeast1"
  display_name = "Multi-Region Spanner Instance"
  num_nodes    = 3

  instance_config {
    name = "asia-southeast1"
  }
}

# Global Load Balancer
resource "google_compute_global_forwarding_rule" "default" {
  name       = "global-rule"
  target     = google_compute_target_http_proxy.default.id
  port_range = "80"
}

resource "google_compute_target_http_proxy" "default" {
  name    = "target-proxy"
  url_map = google_compute_url_map.default.id
}

resource "google_compute_url_map" "default" {
  name            = "url-map"
  default_service = google_compute_backend_service.default.id
}

resource "google_compute_backend_service" "default" {
  name        = "backend-service"
  port_name   = "http"
  protocol    = "HTTP"
  timeout_sec = 10

  dynamic "backend" {
    for_each = local.regions
    content {
      group = google_compute_instance_group_manager.default[backend.key].instance_group
    }
  }

  health_checks = [google_compute_health_check.default.id]
}

# Cloud DNS
resource "google_dns_managed_zone" "default" {
  name        = "app-zone"
  dns_name    = "app.example.com."
  description = "DNS zone for the application"

  dnssec_config {
    state = "on"
  }
}

resource "google_dns_record_set" "geo" {
  for_each = local.regions
  name = "geo.${google_dns_managed_zone.default.dns_name}"
  type = "A"
  ttl  = 300

  managed_zone = google_dns_managed_zone.default.name

  rrdatas = [google_compute_global_forwarding_rule.default.ip_address]

  geo_routing_policy {
    dynamic "location" {
      for_each = each.key == "primary" ? ["asia-1"] : ["us-1"]
      content {
        region = location.value
      }
    }
  }
}

# Monitoring
resource "google_monitoring_dashboard" "dashboard" {
  dashboard_json = jsonencode({
    displayName = "Multi-Region Dashboard"
    gridLayout = {
      widgets = [
        {
          title = "GKE Cluster Status"
          xyChart = {
            dataSets = [
              {
                timeSeriesQuery = {
                  timeSeriesFilter = {
                    filter = "metric.type=\"kubernetes.io/container/cpu/core_usage_time\""
                  }
                }
              }
            ]
          }
        }
      ]
    }
  })
}

# Disaster Recovery Procedures
resource "google_storage_bucket" "dr_backup" {
  for_each = local.regions
  name     = "dr-backup-${each.key}"
  location = each.value

  versioning {
    enabled = true
  }

  lifecycle_rule {
    condition {
      age = 30
    }
    action {
      type = "Delete"
    }
  }
}

# Failover Configuration
resource "google_compute_region_backend_service" "failover" {
  for_each = local.regions
  name     = "failover-backend-${each.key}"
  region   = each.value

  health_checks = [google_compute_health_check.default.id]
  failover_policy {
    disable_connection_drain_on_failover = true
    drop_traffic_if_unhealthy           = true
    failover_ratio                      = 0.1
  }
}
```

## Validation (การตรวจสอบ)
1. Test failover scenarios
2. Verify data replication
3. Test global load balancing
4. Monitor latency across regions
5. Verify DNS routing

## Additional Challenges (ความท้าทายเพิ่มเติม)
1. Implement cross-region VPC peering
2. Add Cloud Armor security policies
3. Set up backup and restore procedures
4. Implement cost optimization
5. Add compliance monitoring
