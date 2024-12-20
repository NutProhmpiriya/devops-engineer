# แบบฝึกหัดที่ 16: การใช้งาน Cloud Dataflow
# Exercise 16: Using Cloud Dataflow

## โจทย์ | Problem
พัฒนา data pipeline ด้วย Cloud Dataflow โดยมีความต้องการดังนี้:

1. สร้าง:
   - Batch processing pipeline
   - Streaming pipeline
   - Data transformations
   - Custom functions

2. จัดการ:
   - Pipeline monitoring
   - Error handling
   - Testing
   - Performance optimization

## เฉลย | Solution

### 1. สร้าง Batch Processing Pipeline (batch_pipeline.py):
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions
from apache_beam.io import ReadFromText, WriteToText

class SalesRecord(beam.DoFn):
    def process(self, element):
        # Parse CSV record
        fields = element.split(',')
        return [{
            'date': fields[0],
            'product': fields[1],
            'amount': float(fields[2])
        }]

class CalculateTotal(beam.PTransform):
    def expand(self, pcoll):
        return (
            pcoll
            | 'ExtractProduct' >> beam.Map(lambda x: (x['product'], x['amount']))
            | 'SumPerProduct' >> beam.CombinePerKey(sum)
        )

def run_pipeline():
    pipeline_options = PipelineOptions([
        '--project=your-project-id',
        '--job_name=sales-analysis',
        '--region=asia-southeast1',
        '--runner=DataflowRunner',
        '--temp_location=gs://your-bucket/temp',
        '--staging_location=gs://your-bucket/staging'
    ])

    with beam.Pipeline(options=pipeline_options) as p:
        # Read input data
        sales = (
            p 
            | 'ReadSales' >> ReadFromText('gs://your-bucket/sales/*.csv')
            | 'ParseRecords' >> beam.ParDo(SalesRecord())
        )

        # Calculate total sales per product
        product_totals = sales | 'CalculateTotals' >> CalculateTotal()

        # Calculate daily sales
        daily_sales = (
            sales
            | 'ExtractDate' >> beam.Map(lambda x: (x['date'], x['amount']))
            | 'SumPerDay' >> beam.CombinePerKey(sum)
        )

        # Write results
        product_totals | 'WriteProductTotals' >> WriteToText(
            'gs://your-bucket/output/product_totals'
        )
        daily_sales | 'WriteDailySales' >> WriteToText(
            'gs://your-bucket/output/daily_sales'
        )
```

### 2. สร้าง Streaming Pipeline (streaming_pipeline.py):
```python
import apache_beam as beam
from apache_beam.options.pipeline_options import PipelineOptions
from apache_beam.transforms.window import FixedWindows
import json

class ParsePubSubMessage(beam.DoFn):
    def process(self, element):
        """Parse PubSub message and extract data."""
        message = json.loads(element.decode('utf-8'))
        return [message]

class CalculateMetrics(beam.DoFn):
    def process(self, element, window=beam.DoFn.WindowParam):
        timestamp = window.start.to_utc_datetime()
        return [{
            'timestamp': timestamp.isoformat(),
            'value': element[1],
            'metric': element[0]
        }]

def run_streaming_pipeline():
    pipeline_options = PipelineOptions([
        '--project=your-project-id',
        '--job_name=metrics-streaming',
        '--region=asia-southeast1',
        '--runner=DataflowRunner',
        '--streaming',
        '--temp_location=gs://your-bucket/temp',
        '--staging_location=gs://your-bucket/staging'
    ])

    with beam.Pipeline(options=pipeline_options) as p:
        # Read from PubSub
        messages = (
            p 
            | 'ReadPubSub' >> beam.io.ReadFromPubSub(
                subscription='projects/your-project/subscriptions/metrics-sub'
            )
            | 'ParseMessages' >> beam.ParDo(ParsePubSubMessage())
        )

        # Process in windows
        windowed_metrics = (
            messages
            | 'Window' >> beam.WindowInto(FixedWindows(60))  # 1-minute windows
            | 'ExtractMetrics' >> beam.Map(
                lambda x: (x['metric_name'], float(x['value']))
            )
            | 'CombineMetrics' >> beam.CombinePerKey(beam.combiners.MeanCombineFn())
            | 'FormatOutput' >> beam.ParDo(CalculateMetrics())
        )

        # Write to BigQuery
        windowed_metrics | 'WriteToBigQuery' >> beam.io.WriteToBigQuery(
            'your-project:dataset.metrics_table',
            schema={
                'fields': [
                    {'name': 'timestamp', 'type': 'TIMESTAMP'},
                    {'name': 'metric', 'type': 'STRING'},
                    {'name': 'value', 'type': 'FLOAT'}
                ]
            },
            write_disposition=beam.io.BigQueryDisposition.WRITE_APPEND,
            create_disposition=beam.io.BigQueryDisposition.CREATE_IF_NEEDED
        )
```

### 3. สร้าง Custom Transforms (transforms.py):
```python
import apache_beam as beam
from apache_beam import window
import datetime

class CustomWindowFn(window.WindowFn):
    def assign(self, context):
        timestamp = context.timestamp
        return [window.IntervalWindow(
            timestamp,
            timestamp + datetime.timedelta(minutes=5)
        )]

class EnrichData(beam.DoFn):
    def __init__(self, project_id):
        self.project_id = project_id

    def setup(self):
        """Initialize resources."""
        from google.cloud import storage
        self.storage_client = storage.Client(project=self.project_id)

    def process(self, element):
        # Add enrichment data
        element['processed_at'] = datetime.datetime.now().isoformat()
        element['project'] = self.project_id
        return [element]

class ValidateData(beam.DoFn):
    def process(self, element):
        required_fields = ['timestamp', 'metric', 'value']
        
        # Validate required fields
        if all(field in element for field in required_fields):
            if isinstance(element['value'], (int, float)):
                yield element
            else:
                yield beam.pvalue.TaggedOutput('errors', {
                    'error': 'Invalid value type',
                    'element': element
                })
        else:
            yield beam.pvalue.TaggedOutput('errors', {
                'error': 'Missing required fields',
                'element': element
            })
```

### 4. ตั้งค่า Testing (test_pipeline.py):
```python
import unittest
import apache_beam as beam
from apache_beam.testing.test_pipeline import TestPipeline
from apache_beam.testing.util import assert_that, equal_to

class TestDataflowPipeline(unittest.TestCase):
    def test_sales_record_parsing(self):
        with TestPipeline() as p:
            input_data = ['2024-01-01,Product1,100.50']
            expected_output = [{
                'date': '2024-01-01',
                'product': 'Product1',
                'amount': 100.50
            }]

            output = (
                p 
                | beam.Create(input_data)
                | beam.ParDo(SalesRecord())
            )

            assert_that(output, equal_to(expected_output))

    def test_calculate_total(self):
        with TestPipeline() as p:
            input_data = [
                {'product': 'A', 'amount': 100},
                {'product': 'A', 'amount': 200},
                {'product': 'B', 'amount': 300}
            ]
            expected_output = [
                ('A', 300),
                ('B', 300)
            ]

            output = (
                p 
                | beam.Create(input_data)
                | CalculateTotal()
            )

            assert_that(output, equal_to(expected_output))

if __name__ == '__main__':
    unittest.main()
```

### 5. ตั้งค่า Monitoring:
```bash
# Create monitoring dashboard
cat > dataflow_dashboard.json << EOF
{
  "displayName": "Dataflow Monitoring",
  "gridLayout": {
    "widgets": [
      {
        "title": "System Lag",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"dataflow.googleapis.com/job/system_lag\" resource.type=\"dataflow_job\""
              }
            }
          }]
        }
      },
      {
        "title": "Element Count",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"dataflow.googleapis.com/job/element_count\" resource.type=\"dataflow_job\""
              }
            }
          }]
        }
      }
    ]
  }
}
EOF

gcloud monitoring dashboards create --config-from-file=dataflow_dashboard.json

# Create alerts
gcloud alpha monitoring policies create \
    --display-name="High System Lag" \
    --condition-display-name="System lag > 5min" \
    --condition-filter='metric.type="dataflow.googleapis.com/job/system_lag"' \
    --condition-threshold-value=300 \
    --condition-threshold-duration=300s
```

## การทดสอบ | Testing
```bash
# Run unit tests
python -m unittest test_pipeline.py

# Run batch pipeline locally
python batch_pipeline.py \
    --runner=DirectRunner \
    --project=your-project-id

# Run streaming pipeline
python streaming_pipeline.py \
    --runner=DataflowRunner \
    --project=your-project-id \
    --streaming

# Monitor job
gcloud dataflow jobs list

# View job details
gcloud dataflow jobs describe JOB_ID

# Cancel job
gcloud dataflow jobs cancel JOB_ID
```

## เพิ่มเติม | Additional Notes
- ใช้ Dataflow SQL สำหรับ simple transformations
- ทำ performance testing ก่อน production
- ใช้ flex templates สำหรับ reusable pipelines
- ตั้งค่า auto-scaling
- พิจารณาใช้ Dataflow Prime
