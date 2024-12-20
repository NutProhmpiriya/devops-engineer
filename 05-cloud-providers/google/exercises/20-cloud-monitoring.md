# แบบฝึกหัดที่ 20: การใช้งาน Cloud Monitoring
# Exercise 20: Using Cloud Monitoring

## โจทย์ | Problem
ตั้งค่าและจัดการระบบ monitoring ด้วย Cloud Monitoring โดยมีความต้องการดังนี้:

1. ตั้งค่า:
   - Custom metrics
   - Dashboards
   - Alert policies
   - Uptime checks

2. จัดการ:
   - Metric collection
   - Monitoring agents
   - SLO/SLI tracking
   - Incident response

## เฉลย | Solution

### 1. Python Monitoring Integration (monitoring_client.py):
```python
from google.cloud import monitoring_v3
from google.cloud.monitoring_v3 import MetricServiceClient
import time
from datetime import datetime, timedelta

class MetricsClient:
    def __init__(self, project_id):
        self.project_id = project_id
        self.client = monitoring_v3.MetricServiceClient()
        self.project_name = f"projects/{project_id}"

    def create_metric_descriptor(self, metric_type, display_name, description, unit):
        """Create a custom metric descriptor."""
        descriptor = monitoring_v3.MetricDescriptor()
        descriptor.type = f"custom.googleapis.com/{metric_type}"
        descriptor.metric_kind = monitoring_v3.MetricDescriptor.MetricKind.GAUGE
        descriptor.value_type = monitoring_v3.MetricDescriptor.ValueType.DOUBLE
        descriptor.description = description
        descriptor.display_name = display_name
        descriptor.unit = unit

        descriptor = self.client.create_metric_descriptor(
            name=self.project_name,
            metric_descriptor=descriptor
        )
        return descriptor

    def write_time_series(self, metric_type, value, labels=None):
        """Write a single point to a time series."""
        series = monitoring_v3.TimeSeries()
        series.metric.type = f"custom.googleapis.com/{metric_type}"
        
        if labels:
            series.metric.labels.update(labels)

        series.resource.type = "global"
        series.resource.labels["project_id"] = self.project_id

        point = monitoring_v3.Point()
        point.value.double_value = value
        point.interval.end_time.seconds = int(time.time())
        series.points = [point]

        self.client.create_time_series(
            name=self.project_name,
            time_series=[series]
        )

    def read_time_series(self, metric_type, minutes=30):
        """Read time series data."""
        now = time.time()
        seconds = int(now)
        nanos = int((now - seconds) * 10**9)
        interval = monitoring_v3.TimeInterval({
            "end_time": {"seconds": seconds, "nanos": nanos},
            "start_time": {"seconds": (seconds - 60 * minutes), "nanos": nanos},
        })

        results = self.client.list_time_series(
            request={
                "name": self.project_name,
                "filter": f'metric.type = "custom.googleapis.com/{metric_type}"',
                "interval": interval,
                "view": monitoring_v3.ListTimeSeriesRequest.TimeSeriesView.FULL
            }
        )

        return list(results)
```

### 2. สร้าง Application Metrics (app_metrics.py):
```python
from monitoring_client import MetricsClient
import time
import threading
from collections import defaultdict

class ApplicationMetrics:
    def __init__(self, project_id):
        self.metrics_client = MetricsClient(project_id)
        self.request_counts = defaultdict(int)
        self.error_counts = defaultdict(int)
        self.latencies = defaultdict(list)
        
        # Initialize metric descriptors
        self.init_metrics()
        
        # Start background reporting
        self.start_reporter()

    def init_metrics(self):
        """Initialize custom metrics."""
        self.metrics_client.create_metric_descriptor(
            "app/request_count",
            "Request Count",
            "Number of requests received",
            "1"
        )
        
        self.metrics_client.create_metric_descriptor(
            "app/error_rate",
            "Error Rate",
            "Rate of errors",
            "1"
        )
        
        self.metrics_client.create_metric_descriptor(
            "app/latency",
            "Request Latency",
            "Request processing time",
            "ms"
        )

    def record_request(self, endpoint, duration_ms, is_error=False):
        """Record a single request."""
        self.request_counts[endpoint] += 1
        self.latencies[endpoint].append(duration_ms)
        
        if is_error:
            self.error_counts[endpoint] += 1

    def report_metrics(self):
        """Report metrics to Cloud Monitoring."""
        for endpoint in self.request_counts:
            # Report request count
            self.metrics_client.write_time_series(
                "app/request_count",
                self.request_counts[endpoint],
                {"endpoint": endpoint}
            )
            
            # Report error rate
            error_rate = (
                self.error_counts[endpoint] / self.request_counts[endpoint]
                if self.request_counts[endpoint] > 0 else 0
            )
            self.metrics_client.write_time_series(
                "app/error_rate",
                error_rate,
                {"endpoint": endpoint}
            )
            
            # Report latency
            if self.latencies[endpoint]:
                avg_latency = sum(self.latencies[endpoint]) / len(self.latencies[endpoint])
                self.metrics_client.write_time_series(
                    "app/latency",
                    avg_latency,
                    {"endpoint": endpoint}
                )
            
            # Reset counters
            self.request_counts[endpoint] = 0
            self.error_counts[endpoint] = 0
            self.latencies[endpoint] = []

    def start_reporter(self):
        """Start background metric reporting."""
        def report_loop():
            while True:
                self.report_metrics()
                time.sleep(60)  # Report every minute
                
        thread = threading.Thread(target=report_loop, daemon=True)
        thread.start()
```

### 3. สร้าง Dashboard Configuration:
```bash
# Create monitoring dashboard
cat > dashboard.json << EOF
{
  "displayName": "Application Dashboard",
  "gridLayout": {
    "widgets": [
      {
        "title": "Request Rate",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"custom.googleapis.com/app/request_count\""
              },
              "aggregation": {
                "alignmentPeriod": "60s",
                "perSeriesAligner": "ALIGN_RATE"
              }
            }
          }]
        }
      },
      {
        "title": "Error Rate",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"custom.googleapis.com/app/error_rate\""
              }
            }
          }]
        }
      },
      {
        "title": "Latency",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"custom.googleapis.com/app/latency\""
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
```

### 4. ตั้งค่า Alert Policies:
```bash
# Create alert for high error rate
gcloud alpha monitoring policies create \
    --display-name="High Error Rate" \
    --condition-display-name="Error rate > 5%" \
    --condition-filter='metric.type="custom.googleapis.com/app/error_rate"' \
    --condition-threshold-value=0.05 \
    --condition-threshold-duration=300s \
    --notification-channels="projects/$PROJECT_ID/notificationChannels/$CHANNEL_ID"

# Create alert for high latency
gcloud alpha monitoring policies create \
    --display-name="High Latency" \
    --condition-display-name="Latency > 1s" \
    --condition-filter='metric.type="custom.googleapis.com/app/latency"' \
    --condition-threshold-value=1000 \
    --condition-threshold-duration=300s \
    --notification-channels="projects/$PROJECT_ID/notificationChannels/$CHANNEL_ID"

# Create alert for low request rate
gcloud alpha monitoring policies create \
    --display-name="Low Traffic" \
    --condition-display-name="Request rate < 1 req/min" \
    --condition-filter='metric.type="custom.googleapis.com/app/request_count"' \
    --condition-threshold-value=1 \
    --condition-threshold-duration=300s \
    --notification-channels="projects/$PROJECT_ID/notificationChannels/$CHANNEL_ID"
```

### 5. ตั้งค่า Uptime Checks:
```bash
# Create HTTP uptime check
gcloud monitoring uptime-checks create http app-health \
    --display-name="Application Health Check" \
    --uri="https://your-app.com/health" \
    --timeout=10s \
    --check-interval=60s \
    --success-threshold=1 \
    --failure-threshold=3

# Create alert for uptime check failures
gcloud alpha monitoring policies create \
    --display-name="Uptime Check Failed" \
    --condition-display-name="Health check failing" \
    --condition-filter='metric.type="monitoring.googleapis.com/uptime_check/check_passed"' \
    --condition-threshold-value=1 \
    --condition-threshold-duration=300s \
    --notification-channels="projects/$PROJECT_ID/notificationChannels/$CHANNEL_ID"
```

### 6. SLO Configuration (slo_config.py):
```python
from google.cloud import monitoring_v3
import datetime

def create_slo(project_id, service_id):
    client = monitoring_v3.ServiceMonitoringServiceClient()
    
    # Create availability SLO
    availability_slo = monitoring_v3.ServiceLevelObjective({
        "display_name": "Availability SLO",
        "goal": 0.999,  # 99.9% availability
        "rolling_period": {"seconds": 86400 * 28},  # 28 days
        "service_level_indicator": {
            "basic_sli": {
                "availability": {}
            }
        }
    })
    
    # Create latency SLO
    latency_slo = monitoring_v3.ServiceLevelObjective({
        "display_name": "Latency SLO",
        "goal": 0.95,  # 95% of requests under threshold
        "rolling_period": {"seconds": 86400 * 28},  # 28 days
        "service_level_indicator": {
            "basic_sli": {
                "latency": {
                    "threshold": {"seconds": 1}  # 1 second threshold
                }
            }
        }
    })
    
    service_name = f"projects/{project_id}/services/{service_id}"
    
    # Create SLOs
    client.create_service_level_objective(
        parent=service_name,
        service_level_objective=availability_slo
    )
    
    client.create_service_level_objective(
        parent=service_name,
        service_level_objective=latency_slo
    )
```

### 7. Incident Response Automation (incident_handler.py):
```python
import google.cloud.monitoring_v3.types as monitoring_types
from google.cloud import monitoring_v3
from google.cloud import error_reporting
import json
import requests

class IncidentHandler:
    def __init__(self, project_id):
        self.project_id = project_id
        self.incident_client = monitoring_v3.IncidentServiceClient()
        self.error_client = error_reporting.Client()

    def handle_incident(self, incident_json):
        """Handle monitoring incident."""
        incident = json.loads(incident_json)
        
        # Log incident
        print(f"Handling incident: {incident['incident_id']}")
        print(f"Resource: {incident['resource_name']}")
        print(f"Condition: {incident['condition_name']}")
        
        # Report to error reporting
        self.error_client.report(
            f"Monitoring incident: {incident['condition_name']}"
        )
        
        # Send notification to Slack
        self.notify_slack(incident)
        
        # Create incident ticket
        self.create_ticket(incident)

    def notify_slack(self, incident):
        """Send notification to Slack."""
        webhook_url = "https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
        
        message = {
            "text": f"🚨 Alert: {incident['condition_name']}",
            "attachments": [{
                "color": "danger",
                "fields": [
                    {
                        "title": "Resource",
                        "value": incident['resource_name'],
                        "short": True
                    },
                    {
                        "title": "Severity",
                        "value": incident['severity'],
                        "short": True
                    }
                ]
            }]
        }
        
        requests.post(webhook_url, json=message)

    def create_ticket(self, incident):
        """Create incident ticket in ticketing system."""
        # Add your ticketing system integration here
        pass

    def get_incident_metrics(self, incident_id):
        """Get metrics related to an incident."""
        name = f"projects/{self.project_id}/incidents/{incident_id}"
        incident = self.incident_client.get_incident(name=name)
        
        return {
            "start_time": incident.start_time,
            "end_time": incident.end_time,
            "severity": incident.severity,
            "condition_name": incident.condition_name,
            "resource_name": incident.resource_name
        }
```

## การทดสอบ | Testing
```python
# Test metrics collection
metrics = ApplicationMetrics('your-project-id')
metrics.record_request('/api/users', 150)
metrics.record_request('/api/orders', 200, is_error=True)

# Test SLO creation
create_slo('your-project-id', 'your-service-id')

# Test incident handling
handler = IncidentHandler('your-project-id')
test_incident = {
    'incident_id': 'test-123',
    'resource_name': 'test-resource',
    'condition_name': 'High CPU Usage',
    'severity': 'critical'
}
handler.handle_incident(json.dumps(test_incident))
```

## เพิ่มเติม | Additional Notes
- ใช้ custom metrics สำหรับ business metrics
- ตั้งค่า proper aggregation windows
- ใช้ multiple notification channels
- ทำ regular incident response drills
- พิจารณาใช้ Monitoring API สำหรับ automation
