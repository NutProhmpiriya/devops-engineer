# Exercise 8: Machine Learning Pipeline on GCP
# แบบฝึกหัดที่ 8: Pipeline สำหรับ Machine Learning บน GCP

## Objective (วัตถุประสงค์)
Create an end-to-end ML pipeline using Vertex AI, Cloud Functions, Cloud Run, and BigQuery.
สร้าง ML pipeline แบบครบวงจรโดยใช้ Vertex AI, Cloud Functions, Cloud Run และ BigQuery

## Architecture Overview (ภาพรวมสถาปัตยกรรม)
1. Data ingestion using Cloud Functions
2. Data processing with Dataflow
3. Model training on Vertex AI
4. Model serving with Cloud Run
5. Monitoring with Cloud Operations
6. Automated retraining pipeline

## Requirements (ข้อกำหนด)
1. Implement automated data pipeline
2. Set up model versioning
3. Configure model monitoring
4. Implement A/B testing
5. Set up automated retraining

## Expected Solution (แนวทางการแก้ปัญหา)

```hcl
# Data Storage
resource "google_storage_bucket" "ml_data" {
  name     = "${var.project_id}-ml-data"
  location = var.region
  
  versioning {
    enabled = true
  }
}

# BigQuery Dataset
resource "google_bigquery_dataset" "ml_dataset" {
  dataset_id = "ml_dataset"
  location   = var.region
}

# Training Data Table
resource "google_bigquery_table" "training_data" {
  dataset_id = google_bigquery_dataset.ml_dataset.dataset_id
  table_id   = "training_data"

  time_partitioning {
    type = "DAY"
  }

  schema = jsonencode([
    {
      name = "feature1",
      type = "FLOAT"
    },
    {
      name = "feature2",
      type = "FLOAT"
    },
    {
      name = "label",
      type = "STRING"
    }
  ])
}

# Vertex AI Notebook Instance
resource "google_notebooks_instance" "ml_notebook" {
  name         = "ml-notebook"
  location     = var.zone
  machine_type = "n1-standard-4"

  vm_image {
    project      = "deeplearning-platform-release"
    image_family = "tf-latest-gpu"
  }

  install_gpu_driver = true
  boot_disk_type     = "PD_SSD"
  boot_disk_size_gb  = 100

  no_public_ip    = true
  no_proxy_access = false

  network = google_compute_network.vpc_network.id
  subnet  = google_compute_subnetwork.subnet.id
}

# Cloud Function for Data Ingestion
resource "google_storage_bucket" "function_source" {
  name     = "${var.project_id}-function-source"
  location = var.region
}

resource "google_cloudfunctions_function" "data_ingestion" {
  name        = "data-ingestion"
  description = "Function for data ingestion"
  runtime     = "python39"

  available_memory_mb   = 256
  source_archive_bucket = google_storage_bucket.function_source.name
  source_archive_object = google_storage_bucket_object.function_zip.name
  trigger_http          = true
  entry_point          = "ingest_data"

  environment_variables = {
    DATASET_ID = google_bigquery_dataset.ml_dataset.dataset_id
    TABLE_ID   = google_bigquery_table.training_data.table_id
  }
}

# Cloud Run for Model Serving
resource "google_cloud_run_service" "model_serving" {
  name     = "model-serving"
  location = var.region

  template {
    spec {
      containers {
        image = "gcr.io/${var.project_id}/model-serving"
        resources {
          limits = {
            cpu    = "2.0"
            memory = "2Gi"
          }
        }
        env {
          name  = "MODEL_NAME"
          value = "latest_model"
        }
      }
    }
  }

  traffic {
    percent         = 100
    latest_revision = true
  }
}

# Vertex AI Pipeline
resource "google_vertex_ai_pipeline" "training_pipeline" {
  display_name = "ml-training-pipeline"
  location     = var.region

  pipeline_spec {
    pipeline_manifest = jsonencode({
      components = {
        comp-data-preprocessing = {
          executorLabel = "exec-data-preprocessing"
          inputDefinitions = {
            parameters = {
              input_data = {
                parameterType = "STRING"
              }
            }
          }
        }
        comp-model-training = {
          executorLabel = "exec-model-training"
          inputDefinitions = {
            parameters = {
              training_data = {
                parameterType = "STRING"
              }
            }
          }
        }
      }
      deploymentSpec = {
        executors = {
          "exec-data-preprocessing" = {
            container = {
              image = "gcr.io/${var.project_id}/data-preprocessing"
            }
          }
          "exec-model-training" = {
            container = {
              image = "gcr.io/${var.project_id}/model-training"
            }
          }
        }
      }
    })
  }
}

# Monitoring
resource "google_monitoring_alert_policy" "model_accuracy" {
  display_name = "Model Accuracy Alert"
  combiner     = "OR"

  conditions {
    display_name = "Model accuracy below threshold"
    condition_threshold {
      filter          = "metric.type=\"custom.googleapis.com/ml/model_accuracy\""
      duration        = "300s"
      comparison      = "COMPARISON_LT"
      threshold_value = 0.95
    }
  }

  notification_channels = [google_monitoring_notification_channel.email.name]
}

# A/B Testing Configuration
resource "google_cloud_run_service" "model_a" {
  name     = "model-a"
  location = var.region

  template {
    spec {
      containers {
        image = "gcr.io/${var.project_id}/model-a"
      }
    }
  }
}

resource "google_cloud_run_service" "model_b" {
  name     = "model-b"
  location = var.region

  template {
    spec {
      containers {
        image = "gcr.io/${var.project_id}/model-b"
      }
    }
  }
}

# Traffic splitting
resource "google_cloud_run_service_iam_policy" "traffic_split" {
  location = google_cloud_run_service.model_serving.location
  project  = google_cloud_run_service.model_serving.project
  service  = google_cloud_run_service.model_serving.name

  policy_data = jsonencode({
    bindings = [{
      role = "roles/run.invoker"
      members = [
        "allUsers",
      ]
    }]
  })
}
```

## Validation (การตรวจสอบ)
1. Test data ingestion pipeline
2. Verify model training process
3. Test model serving endpoints
4. Validate A/B testing setup
5. Check monitoring alerts

## Additional Challenges (ความท้าทายเพิ่มเติม)
1. Implement feature store
2. Add model explainability
3. Set up continuous training
4. Implement model versioning
5. Add data validation pipeline
