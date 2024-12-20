# Exercise 2: Create Cloud Storage Buckets
# แบบฝึกหัดที่ 2: สร้าง Cloud Storage Buckets

## Objective (วัตถุประสงค์)
Create Google Cloud Storage buckets with different configurations and access controls.
สร้าง Google Cloud Storage buckets ที่มีการตั้งค่าและการควบคุมการเข้าถึงที่แตกต่างกัน

## Requirements (ข้อกำหนด)
1. Create two storage buckets:
   - Public bucket for static website hosting
   - Private bucket for sensitive data
2. Configure lifecycle rules
3. Set up IAM permissions

## Expected Solution (แนวทางการแก้ปัญหา)

```hcl
# main.tf
provider "google" {
  project = "YOUR_PROJECT_ID"
  region  = "asia-southeast1"
}

# Public bucket for website
resource "google_storage_bucket" "website" {
  name          = "public-website-bucket-${var.project_id}"
  location      = "ASIA"
  force_destroy = true

  website {
    main_page_suffix = "index.html"
    not_found_page   = "404.html"
  }

  uniform_bucket_level_access = true

  lifecycle_rule {
    condition {
      age = 365
    }
    action {
      type = "Delete"
    }
  }
}

# Make bucket public
resource "google_storage_bucket_iam_member" "public_read" {
  bucket = google_storage_bucket.website.name
  role   = "roles/storage.objectViewer"
  member = "allUsers"
}

# Private bucket
resource "google_storage_bucket" "private" {
  name          = "private-data-bucket-${var.project_id}"
  location      = "ASIA"
  force_destroy = true

  versioning {
    enabled = true
  }

  encryption {
    default_kms_key_name = google_kms_crypto_key.bucket_key.id
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

# KMS key for encryption
resource "google_kms_crypto_key" "bucket_key" {
  name     = "bucket-key"
  key_ring = google_kms_key_ring.bucket_keyring.id
}

resource "google_kms_key_ring" "bucket_keyring" {
  name     = "bucket-keyring"
  location = "global"
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

## Validation (การตรวจสอบ)
1. Verify that the public bucket is accessible via web browser
2. Try to upload files to both buckets
3. Verify that lifecycle rules are applied
4. Check encryption settings on the private bucket

## Additional Challenge (ความท้าทายเพิ่มเติม)
- Add CORS configuration for the public bucket
- Implement object versioning
- Create a Cloud Function that triggers on file upload
- Set up bucket notifications
