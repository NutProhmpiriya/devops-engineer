# แบบฝึกหัดที่ 3: การจัดการ Cloud Storage
# Exercise 3: Managing Cloud Storage

## โจทย์ | Problem
สร้างระบบจัดการไฟล์ด้วย Cloud Storage โดยมีความต้องการดังนี้:

1. สร้าง Cloud Storage buckets:
   - Bucket สำหรับ static website hosting
   - Bucket สำหรับ backup files (ต้องการเก็บ version)
   - Bucket สำหรับ temporary files (ลบอัตโนมัติหลัง 7 วัน)

2. กำหนด IAM permissions และ lifecycle policies
3. ตั้งค่า CORS สำหรับ static website
4. สร้าง signed URLs สำหรับการเข้าถึงไฟล์แบบชั่วคราว

## เฉลย | Solution

### 1. สร้าง Cloud Storage Buckets:
```bash
# Create website bucket
gsutil mb -l asia-southeast1 gs://my-static-website-$(gcloud config get-value project)

# Create backup bucket with versioning
gsutil mb -l asia-southeast1 gs://my-backup-files-$(gcloud config get-value project)
gsutil versioning set on gs://my-backup-files-$(gcloud config get-value project)

# Create temporary files bucket
gsutil mb -l asia-southeast1 gs://my-temp-files-$(gcloud config get-value project)
```

### 2. กำหนด IAM Permissions:
```bash
# Set website bucket public
gsutil iam ch allUsers:objectViewer gs://my-static-website-$(gcloud config get-value project)

# Set backup bucket permissions for specific service account
gsutil iam ch serviceAccount:backup@$(gcloud config get-value project).iam.gserviceaccount.com:objectAdmin gs://my-backup-files-$(gcloud config get-value project)

# Create custom IAM policy for temp bucket
cat > /tmp/policy.json << EOF
{
  "bindings": [
    {
      "members": [
        "serviceAccount:temp-access@$(gcloud config get-value project).iam.gserviceaccount.com"
      ],
      "role": "roles/storage.objectViewer"
    }
  ]
}
EOF

gsutil iam set /tmp/policy.json gs://my-temp-files-$(gcloud config get-value project)
```

### 3. ตั้งค่า Lifecycle Policies:
```bash
# Create lifecycle policy for temp files
cat > lifecycle.json << EOF
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "Delete"},
        "condition": {
          "age": 7
        }
      }
    ]
  }
}
EOF

gsutil lifecycle set lifecycle.json gs://my-temp-files-$(gcloud config get-value project)
```

### 4. ตั้งค่า CORS สำหรับ Website Bucket:
```bash
# Create CORS configuration
cat > cors.json << EOF
[
    {
      "origin": ["*"],
      "method": ["GET", "HEAD", "OPTIONS"],
      "responseHeader": ["Content-Type"],
      "maxAgeSeconds": 3600
    }
]
EOF

gsutil cors set cors.json gs://my-static-website-$(gcloud config get-value project)
```

### 5. สร้าง Sample Website และ Upload:
```bash
# Create sample index.html
cat > index.html << EOF
<!DOCTYPE html>
<html>
<head>
    <title>My Static Website</title>
</head>
<body>
    <h1>Welcome to my website hosted on Google Cloud Storage!</h1>
</body>
</html>
EOF

# Upload to website bucket
gsutil cp index.html gs://my-static-website-$(gcloud config get-value project)

# Set website configuration
gsutil web set -m index.html gs://my-static-website-$(gcloud config get-value project)
```

### 6. สร้าง Signed URL:
```python
from google.cloud import storage
import datetime

def create_signed_url(bucket_name, blob_name):
    storage_client = storage.Client()
    bucket = storage_client.bucket(bucket_name)
    blob = bucket.blob(blob_name)

    url = blob.generate_signed_url(
        version="v4",
        expiration=datetime.timedelta(minutes=15),
        method="GET"
    )
    return url
```

## การทดสอบ | Testing
```bash
# Test website access
curl http://storage.googleapis.com/my-static-website-$(gcloud config get-value project)/index.html

# Test file upload to backup bucket
echo "test backup" > test.txt
gsutil cp test.txt gs://my-backup-files-$(gcloud config get-value project)/

# List versions
gsutil ls -a gs://my-backup-files-$(gcloud config get-value project)/test.txt

# Test lifecycle policy (will need to wait 7 days to verify)
gsutil cp test.txt gs://my-temp-files-$(gcloud config get-value project)/
```

## เพิ่มเติม | Additional Notes
- ใช้ Cloud CDN สำหรับ static website ในสภาพแวดล้อมการผลิต
- พิจารณาใช้ Object Lifecycle Management สำหรับ cost optimization
- ตั้งค่า logging และ monitoring สำหรับการเข้าถึง buckets
- ใช้ Cloud KMS สำหรับการเข้ารหัสข้อมูลที่สำคัญ
- สร้าง backup strategy ที่เหมาะสมสำหรับข้อมูลสำคัญ
