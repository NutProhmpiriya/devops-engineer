# Exercise 9: Enterprise Security and Compliance Infrastructure
# แบบฝึกหัดที่ 9: โครงสร้างพื้นฐานด้านความปลอดภัยและการปฏิบัติตามกฎระดับองค์กร

## Objective (วัตถุประสงค์)
Create a comprehensive security and compliance infrastructure using GCP security services.
สร้างโครงสร้างพื้นฐานด้านความปลอดภัยและการปฏิบัติตามกฎระเบียบโดยใช้บริการด้านความปลอดภัยของ GCP

## Architecture Overview (ภาพรวมสถาปัตยกรรม)
1. Organization Policy Service
2. Cloud Asset Inventory
3. Security Command Center
4. Cloud KMS and Cloud HSM
5. VPC Service Controls
6. Identity-Aware Proxy
7. Binary Authorization
8. Cloud Audit Logs

## Requirements (ข้อกำหนด)
1. Implement zero-trust security model
2. Set up compliance logging and monitoring
3. Configure data loss prevention
4. Implement access boundary controls
5. Set up security alerting

## Expected Solution (แนวทางการแก้ปัญหา)

```hcl
# Organization Policies
resource "google_org_policy_policy" "vm_external_ip" {
  name   = "projects/${var.project_id}/policies/compute.vmExternalIpAccess"
  parent = "projects/${var.project_id}"

  spec {
    rules {
      deny_all = "TRUE"
    }
  }
}

# VPC Service Controls
resource "google_access_context_manager_service_perimeter" "secure_perimeter" {
  parent = "accessPolicies/${google_access_context_manager_access_policy.default.name}"
  name   = "accessPolicies/${google_access_context_manager_access_policy.default.name}/servicePerimeters/secure_perimeter"
  title  = "Secure Service Perimeter"
  status {
    restricted_services = [
      "storage.googleapis.com",
      "bigquery.googleapis.com",
      "cloudfunctions.googleapis.com"
    ]
    resources = ["projects/${var.project_number}"]
    
    ingress_policies {
      ingress_from {
        identities = ["serviceAccount:${google_service_account.trusted_sa.email}"]
        sources {
          resource = "projects/${var.trusted_project}"
        }
      }
      ingress_to {
        resources = ["*"]
        operations {
          service_name = "storage.googleapis.com"
          method_selectors {
            method = "google.storage.objects.get"
          }
        }
      }
    }
  }
}

# Cloud KMS Configuration
resource "google_kms_key_ring" "secure_ring" {
  name     = "secure-key-ring"
  location = "global"
}

resource "google_kms_crypto_key" "key" {
  name            = "secure-crypto-key"
  key_ring        = google_kms_key_ring.secure_ring.id
  rotation_period = "7776000s" # 90 days

  version_template {
    algorithm        = "GOOGLE_SYMMETRIC_ENCRYPTION"
    protection_level = "HSM"
  }
}

# Cloud DLP Configuration
resource "google_data_loss_prevention_inspect_template" "dlp" {
  parent       = "projects/${var.project_id}/locations/global"
  display_name = "Sensitive Data Template"
  description  = "Template for scanning sensitive data"

  inspect_config {
    info_types {
      name = "CREDIT_CARD_NUMBER"
    }
    info_types {
      name = "EMAIL_ADDRESS"
    }
    info_types {
      name = "PHONE_NUMBER"
    }
    
    min_likelihood = "LIKELY"
    
    rule_set {
      info_types {
        name = "EMAIL_ADDRESS"
      }
      rules {
        exclusion_rule {
          regex {
            pattern = ".+@company.com"
          }
          matching_type = "MATCHING_TYPE_FULL_MATCH"
        }
      }
    }
  }
}

# Binary Authorization
resource "google_binary_authorization_policy" "policy" {
  admission_whitelist_patterns {
    name_pattern = "gcr.io/google_containers/*"
  }

  default_admission_rule {
    evaluation_mode  = "ALWAYS_DENY"
    enforcement_mode = "ENFORCED_BLOCK_AND_AUDIT_LOG"
  }

  cluster_admission_rules {
    cluster         = "*"
    evaluation_mode = "REQUIRE_ATTESTATION"
    enforcement_mode = "ENFORCED_BLOCK_AND_AUDIT_LOG"
    
    require_attestations_by = [
      google_binary_authorization_attestor.security_attestor.name,
    ]
  }
}

# Security Command Center
resource "google_scc_source" "custom_source" {
  display_name = "Custom Security Source"
  description  = "Custom security findings source"
  organization = var.org_id
}

resource "google_scc_notification_config" "notification" {
  config_id    = "scc-notification"
  organization = var.org_id
  description  = "Security Command Center notifications"
  
  pubsub_topic = google_pubsub_topic.scc_notifications.id
  
  streaming_config {
    filter = "category=\"OPEN_FIREWALL\""
  }
}

# Audit Logging
resource "google_project_iam_audit_config" "audit_config" {
  project = var.project_id
  service = "allServices"
  
  audit_log_config {
    log_type = "ADMIN_READ"
  }
  audit_log_config {
    log_type = "DATA_WRITE"
  }
  audit_log_config {
    log_type = "DATA_READ"
  }
}

# Identity-Aware Proxy
resource "google_iap_brand" "project_brand" {
  support_email     = "support@company.com"
  application_title = "Cloud IAP Protected Application"
}

resource "google_iap_client" "project_client" {
  display_name = "IAP Client"
  brand        = google_iap_brand.project_brand.name
}

# Security Monitoring
resource "google_monitoring_alert_policy" "security_alert" {
  display_name = "Security Violation Alert"
  combiner     = "OR"

  conditions {
    display_name = "IAP Access Denied"
    condition_threshold {
      filter     = "resource.type=\"gce_instance\" AND metric.type=\"iap.googleapis.com/access_denied_count\""
      duration   = "300s"
      comparison = "COMPARISON_GT"
      threshold_value = 10
    }
  }

  notification_channels = [google_monitoring_notification_channel.security_team.name]
}
```

## Validation (การตรวจสอบ)
1. Test VPC Service Controls
2. Verify DLP scanning
3. Test Binary Authorization
4. Check audit logs
5. Verify security alerts

## Additional Challenges (ความท้าทายเพิ่มเติม)
1. Implement custom security controls
2. Add automated compliance reporting
3. Set up security score monitoring
4. Implement automated remediation
5. Add threat detection rules
