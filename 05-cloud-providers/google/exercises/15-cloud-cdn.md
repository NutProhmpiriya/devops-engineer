# แบบฝึกหัดที่ 15: การใช้งาน Cloud CDN
# Exercise 15: Using Cloud CDN

## โจทย์ | Problem
ตั้งค่าและจัดการ Cloud CDN สำหรับการ deliver content แบบ global โดยมีความต้องการดังนี้:

1. ตั้งค่า:
   - CDN configuration
   - Cache policies
   - SSL certificates
   - Custom domains

2. จัดการ:
   - Cache invalidation
   - Logging และ monitoring
   - Security
   - Performance optimization

## เฉลย | Solution

### 1. ตั้งค่า Backend Bucket:
```bash
# Create Cloud Storage bucket
gsutil mb -l asia-southeast1 gs://$PROJECT_ID-cdn-content

# Upload test content
gsutil cp -r ./static/* gs://$PROJECT_ID-cdn-content/

# Create backend bucket
gcloud compute backend-buckets create cdn-backend \
    --gcs-bucket-name=$PROJECT_ID-cdn-content \
    --enable-cdn
```

### 2. ตั้งค่า Load Balancer with CDN:
```bash
# Create URL map
gcloud compute url-maps create cdn-map \
    --default-backend-bucket=cdn-backend

# Create HTTP(S) proxy
gcloud compute target-http-proxies create http-lb-proxy \
    --url-map=cdn-map

# Create forwarding rule
gcloud compute forwarding-rules create http-content-rule \
    --global \
    --target-http-proxy=http-lb-proxy \
    --ports=80
```

### 3. ตั้งค่า SSL และ Custom Domain:
```bash
# Create SSL certificate
gcloud compute ssl-certificates create cdn-cert \
    --domains=cdn.example.com \
    --global

# Create HTTPS proxy
gcloud compute target-https-proxies create https-lb-proxy \
    --url-map=cdn-map \
    --ssl-certificates=cdn-cert

# Create HTTPS forwarding rule
gcloud compute forwarding-rules create https-content-rule \
    --global \
    --target-https-proxy=https-lb-proxy \
    --ports=443
```

### 4. ตั้งค่า Cache Policies:
```bash
# Update backend bucket with cache policy
gcloud compute backend-buckets update cdn-backend \
    --enable-cdn \
    --cache-mode=CACHE_ALL_STATIC \
    --default-ttl=3600

# Create custom cache key policy
gcloud compute backend-buckets update cdn-backend \
    --cache-key-policy-include-host \
    --cache-key-policy-include-protocol \
    --cache-key-policy-include-query-string=false

# Set cache control headers in Cloud Storage
gsutil -m setmeta -h "Cache-Control:public, max-age=3600" \
    gs://$PROJECT_ID-cdn-content/**/*.{jpg,png,css,js}
```

### 5. Python CDN Management Script (cdn_manager.py):
```python
from google.cloud import storage
from google.cloud import monitoring_v3
import time

class CDNManager:
    def __init__(self, project_id, bucket_name):
        self.storage_client = storage.Client()
        self.bucket = self.storage_client.bucket(bucket_name)
        self.project_id = project_id

    def upload_content(self, source_path, destination_blob_name, content_type=None):
        """Upload content to Cloud Storage with appropriate headers."""
        blob = self.bucket.blob(destination_blob_name)
        
        # Set cache control headers
        cache_control = "public, max-age=3600"
        if content_type and content_type.startswith('image/'):
            cache_control = "public, max-age=86400"  # 24 hours for images
        
        metadata = {
            'Cache-Control': cache_control,
            'Content-Type': content_type
        }
        
        blob.upload_from_filename(
            source_path,
            content_type=content_type,
            metadata=metadata
        )
        
        print(f"Uploaded {source_path} to {destination_blob_name}")

    def invalidate_cache(self, paths):
        """Invalidate CDN cache for specific paths."""
        from google.cloud import compute_v1
        
        client = compute_v1.UrlMapsClient()
        
        # Create cache invalidation request
        request = compute_v1.InvalidateCacheRequest()
        request.path = paths
        
        operation = client.invalidate_cache(
            project=self.project_id,
            url_map="cdn-map",
            cache_invalidation_rule=request
        )
        
        operation.result()  # Wait for completion
        print(f"Cache invalidated for paths: {paths}")

    def monitor_cdn_metrics(self, minutes=30):
        """Monitor CDN metrics."""
        client = monitoring_v3.MetricServiceClient()
        project_name = f"projects/{self.project_id}"
        
        interval = monitoring_v3.TimeInterval({
            'end_time': {'seconds': int(time.time())},
            'start_time': {'seconds': int(time.time() - minutes * 60)},
        })
        
        results = client.list_time_series(
            request={
                "name": project_name,
                "filter": 'metric.type = "loadbalancing.googleapis.com/https/request_count"',
                "interval": interval,
                "view": monitoring_v3.ListTimeSeriesRequest.TimeSeriesView.FULL
            }
        )
        
        for result in results:
            print(f"Metric: {result.metric.type}")
            for point in result.points:
                print(f"Value: {point.value.int64_value}, Time: {point.interval.end_time}")
```

### 6. ตั้งค่า Security:
```bash
# Create security policy
gcloud compute security-policies create cdn-policy \
    --description="CDN Security Policy"

# Add WAF rules
gcloud compute security-policies rules create 1000 \
    --security-policy=cdn-policy \
    --description="Block SQL injection" \
    --expression="evaluatePreconfiguredWaf('sqli-v33-stable')" \
    --action=deny-403

# Apply security policy
gcloud compute backend-buckets update cdn-backend \
    --security-policy=cdn-policy
```

### 7. ตั้งค่า Monitoring:
```bash
# Create monitoring dashboard
cat > dashboard.json << EOF
{
  "displayName": "CDN Monitoring",
  "gridLayout": {
    "widgets": [
      {
        "title": "Cache Hit Ratio",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"loadbalancing.googleapis.com/https/request_count\" resource.type=\"https_lb_rule\""
              }
            }
          }]
        }
      }
    ]
  }
}
EOF

gcloud monitoring dashboards create --config-from-file=dashboard.json

# Create alerts
gcloud alpha monitoring policies create \
    --display-name="Low Cache Hit Rate" \
    --condition-display-name="Cache hit rate < 80%" \
    --condition-filter='metric.type="loadbalancing.googleapis.com/https/cache/hit_rate"' \
    --condition-threshold-value=0.8 \
    --condition-threshold-duration=300s
```

## การทดสอบ | Testing
```python
# Test CDN management
cdn = CDNManager('your-project-id', 'cdn-bucket-name')

# Upload content
cdn.upload_content(
    'static/image.jpg',
    'images/image.jpg',
    'image/jpeg'
)

# Invalidate cache
cdn.invalidate_cache(['/images/*'])

# Monitor metrics
cdn.monitor_cdn_metrics(minutes=30)

# Test CDN endpoints
curl -I https://cdn.example.com/images/image.jpg
curl -I -H "Cache-Control: no-cache" https://cdn.example.com/images/image.jpg
```

## เพิ่มเติม | Additional Notes
- ใช้ compression สำหรับ text-based content
- ตั้งค่า different TTLs ตาม content type
- ใช้ Cloud Armor สำหรับ security
- ทำ performance testing
- พิจารณาใช้ Cloud CDN Premium Tier
