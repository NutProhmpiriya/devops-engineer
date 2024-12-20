# Exercise 6: Microservices Architecture with GCP
# แบบฝึกหัดที่ 6: สถาปัตยกรรมไมโครเซอร์วิสบน GCP

## Objective (วัตถุประสงค์)
Create a complete microservices architecture using multiple GCP services including Cloud Run, Cloud Pub/Sub, Cloud SQL, and Cloud Storage.
สร้างสถาปัตยกรรมไมโครเซอร์วิสโดยใช้บริการหลากหลายของ GCP

## Architecture Overview (ภาพรวมสถาปัตยกรรม)
1. Frontend service on Cloud Run
2. Backend services on Cloud Run
3. Cloud SQL for database
4. Cloud Pub/Sub for event messaging
5. Cloud Storage for file storage
6. Load Balancer with Cloud CDN
7. Secret Manager for sensitive data
8. VPC with private service access

## Requirements (ข้อกำหนด)
1. All services must be privately accessible
2. Implement zero-trust security model
3. Set up monitoring and alerting
4. Configure auto-scaling
5. Implement disaster recovery

## Expected Solution (แนวทางการแก้ปัญหา)

```hcl
# main.tf

# VPC Configuration
resource "google_compute_network" "vpc" {
  name                    = "microservices-vpc"
  auto_create_subnetworks = false
}

resource "google_compute_subnetwork" "subnet" {
  name          = "private-subnet"
  ip_cidr_range = "10.0.0.0/16"
  network       = google_compute_network.vpc.id
  region        = var.region

  private_ip_google_access = true
}

# Cloud SQL
resource "google_sql_database_instance" "main" {
  name             = "microservices-db"
  database_version = "POSTGRES_13"
  region           = var.region

  settings {
    tier = "db-f1-micro"
    ip_configuration {
      ipv4_enabled    = false
      private_network = google_compute_network.vpc.id
    }
    backup_configuration {
      enabled = true
      point_in_time_recovery_enabled = true
    }
  }
}

# Cloud Storage
resource "google_storage_bucket" "media" {
  name          = "${var.project_id}-media"
  location      = "ASIA"
  force_destroy = false

  uniform_bucket_level_access = true
  versioning {
    enabled = true
  }

  lifecycle_rule {
    condition {
      age = 30
    }
    action {
      type = "SetStorageClass"
      storage_class = "NEARLINE"
    }
  }
}

# Pub/Sub Topics and Subscriptions
resource "google_pubsub_topic" "events" {
  name = "microservices-events"

  message_retention_duration = "86600s"
}

resource "google_pubsub_subscription" "events_sub" {
  name  = "events-subscription"
  topic = google_pubsub_topic.events.name

  ack_deadline_seconds = 20

  retry_policy {
    minimum_backoff = "10s"
  }

  dead_letter_policy {
    dead_letter_topic = google_pubsub_topic.dead_letter.id
    max_delivery_attempts = 5
  }
}

# Cloud Run Services
resource "google_cloud_run_service" "frontend" {
  name     = "frontend"
  location = var.region

  template {
    spec {
      containers {
        image = "gcr.io/${var.project_id}/frontend"
        resources {
          limits = {
            cpu    = "1000m"
            memory = "512Mi"
          }
        }
        env {
          name  = "DB_HOST"
          value = google_sql_database_instance.main.private_ip_address
        }
      }
      service_account_name = google_service_account.run_sa.email
    }

    metadata {
      annotations = {
        "autoscaling.knative.dev/maxScale" = "10"
        "run.googleapis.com/vpc-access-connector" = google_vpc_access_connector.connector.name
      }
    }
  }

  traffic {
    percent         = 100
    latest_revision = true
  }
}

# Load Balancer
resource "google_compute_global_address" "default" {
  name = "lb-ip"
}

resource "google_compute_global_forwarding_rule" "default" {
  name                  = "lb-forwarding-rule"
  ip_protocol          = "TCP"
  load_balancing_scheme = "EXTERNAL"
  port_range           = "443"
  target               = google_compute_target_https_proxy.default.id
  ip_address           = google_compute_global_address.default.id
}

# Cloud Armor
resource "google_compute_security_policy" "policy" {
  name = "microservices-security-policy"

  rule {
    action   = "deny(403)"
    priority = "1000"
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = ["9.9.9.0/24"]  # Blocked IPs
      }
    }
    description = "Deny access to specified IPs"
  }

  rule {
    action   = "allow"
    priority = "2147483647"
    match {
      versioned_expr = "SRC_IPS_V1"
      config {
        src_ip_ranges = ["*"]
      }
    }
    description = "Default rule"
  }
}

# Monitoring
resource "google_monitoring_alert_policy" "cpu_utilization" {
  display_name = "High CPU Utilization"
  combiner     = "OR"
  conditions {
    display_name = "CPU utilization"
    condition_threshold {
      filter     = "metric.type=\"run.googleapis.com/container/cpu/utilization\" resource.type=\"cloud_run_revision\""
      duration   = "300s"
      comparison = "COMPARISON_GT"
      threshold_value = 0.8
    }
  }

  notification_channels = [google_monitoring_notification_channel.email.name]
}
```

## Validation (การตรวจสอบ)
1. Test private connectivity between services
2. Verify auto-scaling behavior
3. Test disaster recovery procedures
4. Monitor service latency and performance
5. Verify security policies

## Additional Challenges (ความท้าทายเพิ่มเติม)
1. Implement blue-green deployments
2. Add GraphQL API Gateway
3. Set up distributed tracing
4. Implement circuit breakers
5. Add chaos engineering tests
