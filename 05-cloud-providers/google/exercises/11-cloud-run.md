# แบบฝึกหัดที่ 11: การใช้งาน Cloud Run
# Exercise 11: Using Cloud Run

## โจทย์ | Problem
พัฒนาและ deploy microservices บน Cloud Run โดยมีความต้องการดังนี้:

1. สร้าง:
   - Containerized microservices
   - Custom domains
   - Authentication
   - Auto scaling rules

2. ตั้งค่า:
   - CI/CD pipeline
   - Traffic splitting
   - VPC connections
   - Monitoring

## เฉลย | Solution

### 1. สร้าง Microservice (app.py):
```python
from flask import Flask, request
import os

app = Flask(__name__)

@app.route('/')
def hello():
    name = request.args.get('name', 'World')
    return f'Hello {name}!'

@app.route('/health')
def health():
    return {'status': 'healthy'}

if __name__ == '__main__':
    port = int(os.getenv('PORT', '8080'))
    app.run(host='0.0.0.0', port=port)
```

### 2. สร้าง Dockerfile:
```dockerfile
FROM python:3.9-slim

WORKDIR /app
COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .
CMD ["python", "app.py"]
```

### 3. สร้าง Cloud Build Configuration (cloudbuild.yaml):
```yaml
steps:
  - name: 'gcr.io/cloud-builders/docker'
    args: ['build', '-t', 'gcr.io/$PROJECT_ID/myapp', '.']
  
  - name: 'gcr.io/cloud-builders/docker'
    args: ['push', 'gcr.io/$PROJECT_ID/myapp']
  
  - name: 'gcr.io/google.com/cloudsdktool/cloud-sdk'
    entrypoint: gcloud
    args:
      - 'run'
      - 'deploy'
      - 'myapp'
      - '--image'
      - 'gcr.io/$PROJECT_ID/myapp'
      - '--region'
      - 'asia-southeast1'
      - '--platform'
      - 'managed'
      - '--allow-unauthenticated'

images:
  - 'gcr.io/$PROJECT_ID/myapp'
```

### 4. Deploy Service:
```bash
# Build and deploy manually
gcloud builds submit --config=cloudbuild.yaml

# Deploy with specific configuration
gcloud run deploy myapp \
    --image=gcr.io/$PROJECT_ID/myapp \
    --region=asia-southeast1 \
    --platform=managed \
    --memory=512Mi \
    --cpu=1 \
    --max-instances=10 \
    --min-instances=1 \
    --port=8080 \
    --timeout=300s \
    --concurrency=80
```

### 5. ตั้งค่า Custom Domain:
```bash
# Verify domain ownership
gcloud domains verify example.com

# Map custom domain
gcloud beta run domain-mappings create \
    --service=myapp \
    --domain=api.example.com \
    --region=asia-southeast1
```

### 6. ตั้งค่า Authentication:
```bash
# Create service account
gcloud iam service-accounts create cloud-run-invoker \
    --display-name="Cloud Run Invoker"

# Grant permissions
gcloud run services add-iam-policy-binding myapp \
    --member="serviceAccount:cloud-run-invoker@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/run.invoker" \
    --region=asia-southeast1

# Get ID token for authentication
TOKEN=$(gcloud auth print-identity-token)
curl -H "Authorization: Bearer $TOKEN" https://myapp-hash-se.a.run.app
```

### 7. ตั้งค่า Traffic Splitting:
```bash
# Deploy new version
gcloud run deploy myapp \
    --image=gcr.io/$PROJECT_ID/myapp:v2 \
    --region=asia-southeast1 \
    --tag=v2

# Split traffic
gcloud run services update-traffic myapp \
    --region=asia-southeast1 \
    --to-tags=v2=20 \
    --to-latest=80
```

### 8. ตั้งค่า VPC Connector:
```bash
# Create VPC connector
gcloud compute networks vpc-access connectors create my-connector \
    --region=asia-southeast1 \
    --network=default \
    --range=10.8.0.0/28

# Use connector with Cloud Run
gcloud run services update myapp \
    --vpc-connector=my-connector \
    --region=asia-southeast1
```

### 9. ตั้งค่า Monitoring:
```bash
# Create uptime check
gcloud monitoring uptime-checks create http myapp-check \
    --display-name="MyApp Health Check" \
    --uri="https://myapp-hash-se.a.run.app/health"

# Create alert policy
gcloud alpha monitoring policies create \
    --display-name="High Latency Alert" \
    --condition-display-name="Latency > 1s" \
    --condition-filter='resource.type="cloud_run_revision" AND metric.type="run.googleapis.com/request_latencies"' \
    --condition-threshold-value=1000 \
    --condition-threshold-duration=300s
```

## การทดสอบ | Testing
```bash
# Test deployment
curl https://myapp-hash-se.a.run.app

# Test authentication
curl -H "Authorization: Bearer $TOKEN" https://myapp-hash-se.a.run.app

# Test auto scaling
hey -n 1000 -c 100 https://myapp-hash-se.a.run.app

# Monitor metrics
gcloud monitoring metrics list \
    --filter="metric.type=run.googleapis.com/request_count"

# View logs
gcloud logging read "resource.type=cloud_run_revision" \
    --limit=10
```

## เพิ่มเติม | Additional Notes
- ใช้ Cloud Build triggers สำหรับ automated deployments
- ตั้งค่า error reporting และ tracing
- ใช้ Secret Manager สำหรับ sensitive data
- พิจารณาใช้ Cloud Run jobs สำหรับ batch processing
- ทำ load testing ก่อน production deployment
