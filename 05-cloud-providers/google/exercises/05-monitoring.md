# แบบฝึกหัดที่ 5: การตั้งค่า Monitoring และ Logging
# Exercise 5: Setting up Monitoring and Logging

## โจทย์ | Problem
ตั้งค่าระบบ monitoring และ logging สำหรับแอปพลิเคชันที่รันบน GKE โดยมีความต้องการดังนี้:

1. ตั้งค่า monitoring:
   - CPU และ Memory usage alerts
   - Custom metrics สำหรับ application performance
   - Uptime checks
   - Dashboard แสดงผลสถานะระบบ

2. ตั้งค่า logging:
   - Log exports ไปยัง Cloud Storage
   - Log-based metrics
   - Error reporting
   - Custom log parsing

## เฉลย | Solution

### 1. ตั้งค่า Monitoring Alerts:
```bash
# Create CPU usage alert
gcloud beta monitoring policies create \
  --display-name="High CPU Usage Alert" \
  --condition-display-name="CPU usage above 80%" \
  --condition-filter='metric.type="kubernetes.io/container/cpu/utilization" resource.type="k8s_container"' \
  --condition-threshold-value=0.8 \
  --condition-threshold-duration=300s \
  --notification-channels="projects/$PROJECT_ID/notificationChannels/$CHANNEL_ID"

# Create Memory usage alert
gcloud beta monitoring policies create \
  --display-name="High Memory Usage Alert" \
  --condition-display-name="Memory usage above 85%" \
  --condition-filter='metric.type="kubernetes.io/container/memory/utilization" resource.type="k8s_container"' \
  --condition-threshold-value=0.85 \
  --condition-threshold-duration=300s \
  --notification-channels="projects/$PROJECT_ID/notificationChannels/$CHANNEL_ID"
```

### 2. สร้าง Custom Metrics:
```python
from opencensus.ext.stackdriver import stats_exporter
from opencensus.stats import aggregation
from opencensus.stats import measure
from opencensus.stats import stats
from opencensus.stats import view
from opencensus.tags import tag_key
from opencensus.tags import tag_map
import time

# Create measure
m_latency_ms = measure.MeasureFloat(
    "task_latency",
    "The task latency in milliseconds",
    "ms")

# Create tag key
key_method = tag_key.TagKey("method")

# Create view
latency_view = view.View(
    "task_latency_distribution",
    "The distribution of the task latencies",
    [],
    m_latency_ms,
    aggregation.DistributionAggregation([0, 25, 50, 75, 100, 200, 400, 800, 1000]))

# Create exporter
exporter = stats_exporter.new_stats_exporter()
stats.stats.view_manager.register_exporter(exporter)

# Register view
stats.stats.view_manager.register_view(latency_view)
```

### 3. ตั้งค่า Uptime Checks:
```bash
# Create uptime check
gcloud monitoring uptime-check-configs create http-check \
    --display-name="Web App Check" \
    --http-check-path="/" \
    --ports=443 \
    --hostname="myapp.example.com" \
    --check-interval=300s \
    --timeout=10s
```

### 4. ตั้งค่า Log Exports:
```bash
# Create log sink to Cloud Storage
gcloud logging sinks create app-logs-storage \
    storage.googleapis.com/my-app-logs \
    --log-filter='resource.type="k8s_container"'

# Create log-based metric
gcloud logging metrics create error_count \
    --description="Count of error log entries" \
    --log-filter='severity>=ERROR' \
    --metric-descriptor-type=int64 \
    --metric-descriptor-metric-kind=delta \
    --metric-descriptor-value-type=int64
```

### 5. สร้าง Dashboard (dashboard.json):
```json
{
  "displayName": "Application Dashboard",
  "gridLayout": {
    "columns": "2",
    "widgets": [
      {
        "title": "CPU Usage",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"kubernetes.io/container/cpu/utilization\" resource.type=\"k8s_container\"",
                "aggregation": {
                  "alignmentPeriod": "60s",
                  "crossSeriesReducer": "REDUCE_MEAN",
                  "groupByFields": [
                    "resource.label.\"container_name\""
                  ],
                  "perSeriesAligner": "ALIGN_MEAN"
                }
              }
            }
          }]
        }
      },
      {
        "title": "Memory Usage",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"kubernetes.io/container/memory/utilization\" resource.type=\"k8s_container\"",
                "aggregation": {
                  "alignmentPeriod": "60s",
                  "crossSeriesReducer": "REDUCE_MEAN",
                  "groupByFields": [
                    "resource.label.\"container_name\""
                  ],
                  "perSeriesAligner": "ALIGN_MEAN"
                }
              }
            }
          }]
        }
      }
    ]
  }
}
```

### 6. ตั้งค่า Error Reporting:
```python
from google.cloud import error_reporting

def report_exception(exc):
    client = error_reporting.Client()
    client.report_exception()

try:
    # Your application code here
    raise Exception('Custom error message')
except Exception as exc:
    report_exception(exc)
```

## การทดสอบ | Testing
```bash
# Test alert policies
gcloud beta monitoring policies list

# View custom metrics
gcloud monitoring metrics list --filter="metric.type=contains:task_latency"

# Check uptime check status
gcloud monitoring uptime-check-configs list

# View logs
gcloud logging read 'resource.type="k8s_container"' --limit=5

# Test log-based metrics
gcloud logging metrics list

# View error reports
gcloud beta error-reporting events list
```

## เพิ่มเติม | Additional Notes
- ใช้ Workload Identity สำหรับการเข้าถึง Google Cloud APIs
- พิจารณาใช้ Cloud Trace สำหรับ distributed tracing
- ตั้งค่า log retention ตามความเหมาะสม
- ใช้ Cloud Monitoring API สำหรับ custom integrations
- สร้าง SLOs (Service Level Objectives) สำหรับ critical services
