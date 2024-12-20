# แบบฝึกหัดที่ 6: การพัฒนาและ Deploy Cloud Functions
# Exercise 6: Developing and Deploying Cloud Functions

## โจทย์ | Problem
สร้างระบบประมวลผลรูปภาพอัตโนมัติด้วย Cloud Functions โดยมีความต้องการดังนี้:

1. สร้าง Cloud Function ที่ทำงานเมื่อมีการอัพโหลดรูปภาพใหม่ใน Cloud Storage:
   - ปรับขนาดรูปภาพให้เป็น thumbnail
   - เพิ่ม watermark
   - บันทึกข้อมูล metadata ลงใน Firestore

2. สร้าง HTTP Function สำหรับดูสถานะการประมวลผล

3. ตั้งค่า authentication และ monitoring

## เฉลย | Solution

### 1. สร้าง Cloud Function สำหรับประมวลผลรูปภาพ (main.py):
```python
from google.cloud import storage
from google.cloud import firestore
from PIL import Image
import tempfile
import os

def process_image(event, context):
    """Cloud Function triggered by Cloud Storage."""
    bucket = event['bucket']
    name = event['name']
    
    # Skip if this is already a thumbnail
    if name.startswith('thumb_'):
        return

    # Get the image
    storage_client = storage.Client()
    bucket = storage_client.bucket(bucket)
    blob = bucket.blob(name)
    
    # Download to temp file
    _, temp_local_filename = tempfile.mkstemp()
    blob.download_to_filename(temp_local_filename)
    
    # Process with PIL
    with Image.open(temp_local_filename) as image:
        # Create thumbnail
        thumbnail = image.copy()
        thumbnail.thumbnail((200, 200))
        
        # Add watermark
        watermark = Image.open('watermark.png')
        position = ((thumbnail.width - watermark.width),
                   (thumbnail.height - watermark.height))
        thumbnail.paste(watermark, position, watermark)
        
        # Save thumbnail
        thumb_filename = f"/tmp/thumb_{os.path.basename(temp_local_filename)}"
        thumbnail.save(thumb_filename)
        
        # Upload thumbnail
        thumb_blob = bucket.blob(f"thumbnails/thumb_{name}")
        thumb_blob.upload_from_filename(thumb_filename)
        
        # Store metadata in Firestore
        db = firestore.Client()
        doc_ref = db.collection('images').document(name)
        doc_ref.set({
            'originalName': name,
            'thumbnailName': f"thumb_{name}",
            'processedAt': context.timestamp,
            'size': blob.size,
            'contentType': blob.content_type
        })
        
        # Cleanup
        os.remove(temp_local_filename)
        os.remove(thumb_filename)
        
    return f"Successfully processed {name}"
```

### 2. สร้าง HTTP Function สำหรับดูสถานะ (status.py):
```python
from google.cloud import firestore
from flask import jsonify

def get_image_status(request):
    """HTTP Cloud Function to get image processing status."""
    # Set CORS headers
    headers = {
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'GET',
        'Access-Control-Allow-Headers': 'Content-Type'
    }
    
    # Handle OPTIONS request for CORS
    if request.method == 'OPTIONS':
        return ('', 204, headers)
    
    # Get image name from query parameters
    image_name = request.args.get('image')
    if not image_name:
        return (jsonify({'error': 'Image name is required'}), 400, headers)
    
    # Get status from Firestore
    db = firestore.Client()
    doc_ref = db.collection('images').document(image_name)
    doc = doc_ref.get()
    
    if not doc.exists:
        return (jsonify({'error': 'Image not found'}), 404, headers)
    
    return (jsonify(doc.to_dict()), 200, headers)
```

### 3. ตั้งค่า Dependencies (requirements.txt):
```text
google-cloud-storage==2.5.0
google-cloud-firestore==2.7.2
Pillow==9.3.0
```

### 4. Deploy Functions:
```bash
# Deploy image processing function
gcloud functions deploy process_image \
    --runtime python39 \
    --trigger-resource YOUR_BUCKET_NAME \
    --trigger-event google.storage.object.finalize \
    --memory 512MB \
    --timeout 300s

# Deploy status check function
gcloud functions deploy get_image_status \
    --runtime python39 \
    --trigger-http \
    --allow-unauthenticated
```

### 5. ตั้งค่า IAM และ Security:
```bash
# Create service account for functions
gcloud iam service-accounts create image-processor \
    --display-name="Image Processor Service Account"

# Grant necessary permissions
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:image-processor@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/storage.objectViewer"

gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:image-processor@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/datastore.user"
```

## การทดสอบ | Testing
```bash
# Upload test image
gsutil cp test-image.jpg gs://YOUR_BUCKET_NAME/

# Check function logs
gcloud functions logs read process_image

# Test status API
curl "https://REGION-PROJECT_ID.cloudfunctions.net/get_image_status?image=test-image.jpg"

# Monitor function metrics
gcloud monitoring metrics list \
    --filter="metric.type=cloudfunctions.googleapis.com/function/execution_count"
```

## เพิ่มเติม | Additional Notes
- ใช้ Cloud Tasks สำหรับการประมวลผลที่ใช้เวลานาน
- พิจารณาใช้ Cloud Run สำหรับ workloads ที่ต้องการ scale
- ตั้งค่า error handling และ retry logic
- ใช้ Cloud KMS สำหรับการจัดการ secrets
- เพิ่ม monitoring และ alerting สำหรับ errors
