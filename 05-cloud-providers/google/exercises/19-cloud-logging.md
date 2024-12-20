# แบบฝึกหัดที่ 19: การใช้งาน Cloud Logging
# Exercise 19: Using Cloud Logging

## โจทย์ | Problem
ตั้งค่าและจัดการระบบ logging ด้วย Cloud Logging โดยมีความต้องการดังนี้:

1. ตั้งค่า:
   - Log routing
   - Log sinks
   - Log metrics
   - Log-based alerts

2. จัดการ:
   - Log aggregation
   - Log analysis
   - Log export
   - Log retention

## เฉลย | Solution

### 1. ตั้งค่า Log Router:
```bash
# Create log bucket
gcloud logging buckets create app-logs \
    --location=asia-southeast1 \
    --description="Application logs" \
    --retention-days=30

# Create log sink to Cloud Storage
gcloud logging sinks create storage-sink \
    storage.googleapis.com/$PROJECT_ID-logs \
    --log-filter="resource.type=gce_instance" \
    --description="Export logs to Cloud Storage"

# Create log sink to BigQuery
gcloud logging sinks create bigquery-sink \
    bigquery.googleapis.com/projects/$PROJECT_ID/datasets/logs \
    --log-filter="severity >= WARNING" \
    --description="Export severe logs to BigQuery"

# Create log sink to Pub/Sub
gcloud logging sinks create pubsub-sink \
    pubsub.googleapis.com/projects/$PROJECT_ID/topics/logs \
    --log-filter="resource.type=cloud_run_revision" \
    --description="Stream logs to Pub/Sub"
```

### 2. Python Logging Integration (app_logging.py):
```python
import google.cloud.logging
import logging
import json
from datetime import datetime

class CloudLogger:
    def __init__(self, project_id, log_name):
        self.client = google.cloud.logging.Client(project=project_id)
        self.logger = self.client.logger(log_name)
        
        # Setup Python standard logging integration
        self.client.setup_logging()

    def log_event(self, severity, message, **kwargs):
        """Log an event with structured data."""
        payload = {
            'message': message,
            'timestamp': datetime.utcnow().isoformat(),
            'metadata': kwargs
        }
        
        self.logger.log_struct(
            payload,
            severity=severity
        )

    def log_error(self, error, **kwargs):
        """Log an error with stack trace."""
        import traceback
        
        payload = {
            'error_message': str(error),
            'stack_trace': traceback.format_exc(),
            'timestamp': datetime.utcnow().isoformat(),
            'metadata': kwargs
        }
        
        self.logger.log_struct(
            payload,
            severity='ERROR'
        )

class RequestLogger:
    def __init__(self, cloud_logger):
        self.logger = cloud_logger

    def log_request(self, request, response, duration_ms):
        """Log HTTP request details."""
        payload = {
            'method': request.method,
            'path': request.path,
            'status_code': response.status_code,
            'duration_ms': duration_ms,
            'user_agent': request.headers.get('User-Agent'),
            'ip': request.remote_addr,
            'timestamp': datetime.utcnow().isoformat()
        }
        
        severity = 'ERROR' if response.status_code >= 500 else (
            'WARNING' if response.status_code >= 400 else 'INFO'
        )
        
        self.logger.log_struct(
            payload,
            severity=severity
        )
```

### 3. สร้าง Log Metrics:
```bash
# Create counter metric for errors
gcloud logging metrics create error_count \
    --description="Count of error logs" \
    --filter="severity>=ERROR" \
    --metric-descriptor-type=int64

# Create distribution metric for latency
gcloud logging metrics create request_latency \
    --description="Distribution of request latencies" \
    --filter="resource.type=cloud_run_revision" \
    --metric-descriptor-type=distribution \
    --value-extractor="EXTRACT(jsonPayload.duration_ms)"

# Create counter metric for specific errors
gcloud logging metrics create auth_failures \
    --description="Count of authentication failures" \
    --filter='resource.type="cloud_run_revision" AND jsonPayload.error_type="AuthenticationError"'
```

### 4. ตั้งค่า Log-based Alerts:
```bash
# Create alert for high error rate
gcloud alpha monitoring policies create \
    --display-name="High Error Rate" \
    --condition-display-name="Error rate > 1%" \
    --condition-filter='metric.type="logging.googleapis.com/user/error_count"' \
    --condition-threshold-value=0.01 \
    --condition-threshold-duration=300s \
    --notification-channels="projects/$PROJECT_ID/notificationChannels/$CHANNEL_ID"

# Create alert for high latency
gcloud alpha monitoring policies create \
    --display-name="High Latency" \
    --condition-display-name="P95 latency > 1s" \
    --condition-filter='metric.type="logging.googleapis.com/user/request_latency"' \
    --condition-threshold-value=1000 \
    --condition-threshold-duration=300s \
    --notification-channels="projects/$PROJECT_ID/notificationChannels/$CHANNEL_ID"
```

### 5. สร้าง Log Analysis Query (log_analysis.sql):
```sql
-- Analysis of error patterns
SELECT
  timestamp,
  jsonPayload.error_message as error,
  COUNT(*) as error_count
FROM `project_id.dataset_id.log_table`
WHERE severity = 'ERROR'
GROUP BY timestamp, error
ORDER BY error_count DESC
LIMIT 10;

-- Latency analysis
SELECT
  TIMESTAMP_TRUNC(timestamp, HOUR) as hour,
  AVG(jsonPayload.duration_ms) as avg_latency,
  APPROX_QUANTILES(jsonPayload.duration_ms, 100)[OFFSET(95)] as p95_latency
FROM `project_id.dataset_id.log_table`
GROUP BY hour
ORDER BY hour DESC;

-- User activity patterns
SELECT
  jsonPayload.ip as client_ip,
  COUNT(*) as request_count,
  AVG(jsonPayload.duration_ms) as avg_latency
FROM `project_id.dataset_id.log_table`
GROUP BY client_ip
HAVING request_count > 1000
ORDER BY request_count DESC;
```

### 6. Log Processing Script (process_logs.py):
```python
from google.cloud import bigquery
from google.cloud import storage
import json
import gzip
from datetime import datetime, timedelta

def process_logs(project_id, dataset_id, table_id):
    """Process and analyze logs from BigQuery."""
    client = bigquery.Client(project=project_id)
    
    query = f"""
    SELECT
        timestamp,
        severity,
        jsonPayload,
        resource.type as resource_type
    FROM `{project_id}.{dataset_id}.{table_id}`
    WHERE timestamp > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 1 DAY)
    """
    
    query_job = client.query(query)
    
    # Process results
    error_count = 0
    warning_count = 0
    resource_counts = {}
    
    for row in query_job:
        if row.severity == 'ERROR':
            error_count += 1
        elif row.severity == 'WARNING':
            warning_count += 1
            
        resource_counts[row.resource_type] = \
            resource_counts.get(row.resource_type, 0) + 1
    
    # Generate report
    report = {
        'timestamp': datetime.utcnow().isoformat(),
        'error_count': error_count,
        'warning_count': warning_count,
        'resource_distribution': resource_counts
    }
    
    # Save report
    storage_client = storage.Client(project=project_id)
    bucket = storage_client.bucket(f'{project_id}-log-reports')
    blob = bucket.blob(f'daily-report-{datetime.utcnow().date()}.json.gz')
    
    # Compress and upload
    compressed_data = gzip.compress(json.dumps(report).encode('utf-8'))
    blob.upload_from_string(compressed_data, content_type='application/gzip')
    
    return report
```

### 7. Log Export Configuration:
```bash
# Create export dataset
bq mk --dataset \
    --description="Log Analysis Dataset" \
    --location=asia-southeast1 \
    $PROJECT_ID:log_analysis

# Create log sink to dataset
gcloud logging sinks create bq-detailed-sink \
    bigquery.googleapis.com/projects/$PROJECT_ID/datasets/log_analysis \
    --log-filter="resource.type=( gce_instance OR cloud_run_revision OR k8s_container )"

# Grant permissions
export SINK_SA=$(gcloud logging sinks describe bq-detailed-sink --format='get(writerIdentity)')
bq add-iam-policy-binding $PROJECT_ID:log_analysis \
    --member=$SINK_SA \
    --role=roles/bigquery.dataEditor
```

## การทดสอบ | Testing
```python
# Test logging integration
logger = CloudLogger('your-project-id', 'test-log')

# Test structured logging
logger.log_event('INFO', 'Test event', user_id='123', action='login')

# Test error logging
try:
    raise ValueError("Test error")
except Exception as e:
    logger.log_error(e, context='testing')

# Test request logging
request_logger = RequestLogger(logger)
request_logger.log_request(
    request=mock_request,
    response=mock_response,
    duration_ms=150
)

# Test log processing
report = process_logs(
    'your-project-id',
    'log_analysis',
    'logs_20240120'
)
print(json.dumps(report, indent=2))
```

## เพิ่มเติม | Additional Notes
- ใช้ structured logging สำหรับ better analysis
- ตั้งค่า log sampling สำหรับ high-volume logs
- ใช้ log-based metrics สำหรับ monitoring
- ทำ regular log analysis
- พิจารณาใช้ Cloud Logging API สำหรับ custom integration
