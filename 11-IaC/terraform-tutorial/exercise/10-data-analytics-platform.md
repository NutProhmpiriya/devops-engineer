# Exercise 10: Enterprise Data Analytics Platform
# แบบฝึกหัดที่ 10: แพลตฟอร์มวิเคราะห์ข้อมูลระดับองค์กร

## Objective (วัตถุประสงค์)
Create a comprehensive data analytics platform using GCP's data services.
สร้างแพลตฟอร์มวิเคราะห์ข้อมูลที่ครอบคลุมโดยใช้บริการด้านข้อมูลของ GCP

## Architecture Overview (ภาพรวมสถาปัตยกรรม)
1. Data Ingestion (Pub/Sub, Dataflow)
2. Data Storage (BigQuery, Cloud Storage)
3. Data Processing (Dataproc, Dataflow)
4. Data Warehousing (BigQuery)
5. Data Visualization (Looker)
6. Machine Learning Integration
7. Real-time Analytics

## Requirements (ข้อกำหนด)
1. Support batch and streaming data
2. Implement data governance
3. Enable real-time analytics
4. Set up data quality monitoring
5. Configure disaster recovery

## Expected Solution (แนวทางการแก้ปัญหา)

```hcl
# Data Lake Storage
resource "google_storage_bucket" "data_lake" {
  name          = "${var.project_id}-data-lake"
  location      = var.region
  force_destroy = false

  uniform_bucket_level_access = true
  
  lifecycle_rule {
    condition {
      age = 90
    }
    action {
      type = "SetStorageClass"
      storage_class = "COLDLINE"
    }
  }
}

# BigQuery Data Warehouse
resource "google_bigquery_dataset" "raw_data" {
  dataset_id                  = "raw_data"
  friendly_name              = "Raw Data"
  description                = "Raw data landing zone"
  location                   = var.region
  default_table_expiration_ms = 7776000000 # 90 days

  access {
    role          = "OWNER"
    user_by_email = google_service_account.data_pipeline.email
  }
}

resource "google_bigquery_dataset" "processed_data" {
  dataset_id                  = "processed_data"
  friendly_name              = "Processed Data"
  description                = "Processed and transformed data"
  location                   = var.region
  
  access {
    role          = "READER"
    user_by_email = google_service_account.analytics_users.email
  }
}

# Pub/Sub Topics for Streaming Data
resource "google_pubsub_topic" "raw_events" {
  name = "raw-events"
  
  message_retention_duration = "86600s"
  
  schema_settings {
    schema = google_pubsub_schema.event_schema.id
    encoding = "JSON"
  }
}

resource "google_pubsub_schema" "event_schema" {
  name = "event-schema"
  type = "AVRO"
  definition = jsonencode({
    type = "record"
    name = "Event"
    fields = [
      {
        name = "id"
        type = "string"
      },
      {
        name = "timestamp"
        type = "long"
      },
      {
        name = "data"
        type = "string"
      }
    ]
  })
}

# Dataflow Job
resource "google_dataflow_job" "streaming_job" {
  name              = "streaming-analytics"
  template_gcs_path = "gs://dataflow-templates/latest/Stream_GCS_Text_to_BigQuery"
  temp_gcs_location = "${google_storage_bucket.data_lake.url}/temp"
  
  parameters = {
    inputTopic        = google_pubsub_topic.raw_events.id
    outputTableSpec   = "${google_bigquery_dataset.processed_data.project}:${google_bigquery_dataset.processed_data.dataset_id}.processed_events"
  }

  on_delete = "cancel"
}

# Dataproc Cluster for Batch Processing
resource "google_dataproc_cluster" "analytics_cluster" {
  name     = "analytics-cluster"
  region   = var.region

  cluster_config {
    staging_bucket = google_storage_bucket.data_lake.name

    master_config {
      num_instances = 1
      machine_type = "n1-standard-4"
      disk_config {
        boot_disk_type    = "pd-ssd"
        boot_disk_size_gb = 100
      }
    }

    worker_config {
      num_instances = 2
      machine_type = "n1-standard-4"
      disk_config {
        boot_disk_type    = "pd-standard"
        boot_disk_size_gb = 100
        num_local_ssds    = 1
      }
    }

    preemptible_worker_config {
      num_instances = 2
    }

    software_config {
      image_version = "2.0-debian10"
      override_properties = {
        "dataproc:dataproc.allow.zero.workers" = "true"
      }
      optional_components = ["JUPYTER"]
    }

    autoscaling_config {
      policy_uri = google_dataproc_autoscaling_policy.analytics_policy.name
    }
  }
}

# Data Quality Monitoring
resource "google_bigquery_data_transfer_config" "dq_monitoring" {
  display_name           = "Data Quality Monitoring"
  location              = var.region
  data_source_id        = "scheduled_query"
  schedule              = "every 6 hours"
  destination_dataset_id = google_bigquery_dataset.processed_data.dataset_id
  
  params = {
    query = <<EOF
WITH quality_metrics AS (
  SELECT
    COUNT(*) as total_rows,
    COUNT(DISTINCT id) as unique_ids,
    COUNT(CASE WHEN timestamp IS NULL THEN 1 END) as null_timestamps
  FROM `${google_bigquery_dataset.raw_data.project}.${google_bigquery_dataset.raw_data.dataset_id}.events`
)
SELECT
  CURRENT_TIMESTAMP() as check_time,
  *
FROM quality_metrics
EOF
  }
}

# Data Catalog
resource "google_data_catalog_entry_group" "analytics" {
  entry_group_id = "analytics_catalog"
  
  display_name = "Analytics Catalog"
  description  = "Catalog for analytics data assets"
}

resource "google_data_catalog_entry" "raw_data" {
  entry_group = google_data_catalog_entry_group.analytics.id
  entry_id    = "raw_data"

  user_specified_type = "raw_data"
  user_specified_system = "analytics_platform"

  schema = jsonencode({
    columns = [
      {
        column = "id"
        description = "Unique identifier"
        mode = "REQUIRED"
        type = "STRING"
      },
      {
        column = "timestamp"
        description = "Event timestamp"
        mode = "REQUIRED"
        type = "TIMESTAMP"
      }
    ]
  })
}

# Monitoring and Alerting
resource "google_monitoring_alert_policy" "data_freshness" {
  display_name = "Data Freshness Alert"
  combiner     = "OR"

  conditions {
    display_name = "Data delay"
    condition_threshold {
      filter     = "metric.type=\"custom.googleapis.com/data_freshness\" resource.type=\"global\""
      duration   = "900s"
      comparison = "COMPARISON_GT"
      threshold_value = 3600 # 1 hour
    }
  }

  notification_channels = [google_monitoring_notification_channel.data_team.name]
}

# Disaster Recovery
resource "google_storage_bucket" "dr_backup" {
  name          = "${var.project_id}-dr-backup"
  location      = "US"
  force_destroy = false

  versioning {
    enabled = true
  }

  lifecycle_rule {
    condition {
      age = 30
    }
    action {
      type = "SetStorageClass"
      storage_class = "COLDLINE"
    }
  }
}

resource "google_bigquery_dataset" "dr_dataset" {
  dataset_id                  = "dr_backup"
  friendly_name              = "DR Backup"
  description                = "Disaster recovery backup dataset"
  location                   = "US"
  default_table_expiration_ms = null

  access {
    role          = "OWNER"
    user_by_email = google_service_account.dr_service.email
  }
}
```

## Validation (การตรวจสอบ)
1. Test data ingestion pipeline
2. Verify data transformations
3. Check data quality metrics
4. Test disaster recovery
5. Verify monitoring alerts

## Additional Challenges (ความท้าทายเพิ่มเติม)
1. Implement data lineage tracking
2. Add automated data quality checks
3. Set up cross-region replication
4. Implement data access auditing
5. Add automated data lifecycle management
