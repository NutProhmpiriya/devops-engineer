# แบบฝึกหัดที่ 17: การใช้งาน Cloud Composer
# Exercise 17: Using Cloud Composer

## โจทย์ | Problem
พัฒนาและจัดการ workflow orchestration ด้วย Cloud Composer โดยมีความต้องการดังนี้:

1. สร้าง:
   - DAGs for different workflows
   - Custom operators
   - Sensors
   - Task dependencies

2. จัดการ:
   - Environment configuration
   - Monitoring
   - Error handling
   - Scheduling

## เฉลย | Solution

### 1. สร้าง Environment:
```bash
# Create Cloud Composer environment
gcloud composer environments create production-env \
    --location=asia-southeast1 \
    --image-version=composer-2.0.0 \
    --node-count=3 \
    --zone=asia-southeast1-a \
    --machine-type=n1-standard-2 \
    --network=default \
    --subnetwork=default

# Set environment variables
gcloud composer environments update production-env \
    --location=asia-southeast1 \
    --update-env-variables=MAX_PARALLEL_TASKS=10
```

### 2. สร้าง Data Processing DAG (data_processing_dag.py):
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.providers.google.cloud.operators.bigquery import BigQueryOperator
from airflow.providers.google.cloud.transfers.gcs_to_bigquery import GCSToBigQueryOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'data-team',
    'depends_on_past': False,
    'email': ['data-team@example.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5)
}

with DAG(
    'data_processing',
    default_args=default_args,
    description='Data processing workflow',
    schedule_interval='0 2 * * *',
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=['data-processing']
) as dag:

    # Task to check if data is ready
    def check_data_availability(**context):
        from google.cloud import storage
        client = storage.Client()
        bucket = client.bucket('your-data-bucket')
        blobs = list(bucket.list_blobs(prefix='data/'))
        if not blobs:
            raise Exception('No data files found')
        return True

    check_data = PythonOperator(
        task_id='check_data',
        python_callable=check_data_availability
    )

    # Load data to BigQuery
    load_to_bq = GCSToBigQueryOperator(
        task_id='load_to_bigquery',
        bucket='your-data-bucket',
        source_objects=['data/*.csv'],
        destination_project_dataset_table='your-project.dataset.table',
        schema_fields=[
            {'name': 'id', 'type': 'STRING'},
            {'name': 'value', 'type': 'FLOAT'},
            {'name': 'timestamp', 'type': 'TIMESTAMP'}
        ],
        write_disposition='WRITE_APPEND',
        create_disposition='CREATE_IF_NEEDED',
        skip_leading_rows=1,
        allow_quoted_newlines=True
    )

    # Transform data in BigQuery
    transform_data = BigQueryOperator(
        task_id='transform_data',
        sql='''
        SELECT 
            DATE(timestamp) as date,
            SUM(value) as daily_total
        FROM `your-project.dataset.table`
        GROUP BY date
        ''',
        destination_dataset_table='your-project.dataset.daily_summary',
        write_disposition='WRITE_TRUNCATE',
        create_disposition='CREATE_IF_NEEDED',
        use_legacy_sql=False
    )

    # Set task dependencies
    check_data >> load_to_bq >> transform_data
```

### 3. สร้าง Custom Operator (custom_operators.py):
```python
from airflow.models import BaseOperator
from airflow.utils.decorators import apply_defaults
from google.cloud import storage
from typing import Any, Optional

class DataValidationOperator(BaseOperator):
    @apply_defaults
    def __init__(
        self,
        project_id: str,
        bucket_name: str,
        validation_function: callable,
        *args, **kwargs
    ) -> None:
        super().__init__(*args, **kwargs)
        self.project_id = project_id
        self.bucket_name = bucket_name
        self.validation_function = validation_function

    def execute(self, context: Any) -> None:
        storage_client = storage.Client(project=self.project_id)
        bucket = storage_client.bucket(self.bucket_name)

        # Get latest data file
        blobs = list(bucket.list_blobs(prefix='data/'))
        if not blobs:
            raise ValueError('No data files found')

        latest_blob = max(blobs, key=lambda x: x.time_created)
        
        # Download and validate
        data = latest_blob.download_as_string()
        validation_result = self.validation_function(data)
        
        if not validation_result['is_valid']:
            raise ValueError(f"Data validation failed: {validation_result['errors']}")
        
        return validation_result

class GCSToSpannerOperator(BaseOperator):
    @apply_defaults
    def __init__(
        self,
        project_id: str,
        instance_id: str,
        database_id: str,
        table_name: str,
        gcs_bucket: str,
        gcs_object: str,
        *args, **kwargs
    ) -> None:
        super().__init__(*args, **kwargs)
        self.project_id = project_id
        self.instance_id = instance_id
        self.database_id = database_id
        self.table_name = table_name
        self.gcs_bucket = gcs_bucket
        self.gcs_object = gcs_object

    def execute(self, context: Any) -> None:
        from google.cloud import spanner
        from google.cloud import storage
        import csv
        import io

        # Get data from GCS
        storage_client = storage.Client(project=self.project_id)
        bucket = storage_client.bucket(self.gcs_bucket)
        blob = bucket.blob(self.gcs_object)
        content = blob.download_as_string().decode('utf-8')
        
        # Process CSV data
        csv_reader = csv.DictReader(io.StringIO(content))
        rows = list(csv_reader)
        
        # Write to Spanner
        spanner_client = spanner.Client(project=self.project_id)
        instance = spanner_client.instance(self.instance_id)
        database = instance.database(self.database_id)
        
        with database.batch() as batch:
            batch.insert(
                table=self.table_name,
                columns=[k for k in rows[0].keys()],
                values=[[v for v in row.values()] for row in rows]
            )
```

### 4. สร้าง Error Handling DAG (error_handling_dag.py):
```python
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.dummy import DummyOperator
from airflow.operators.email import EmailOperator
from datetime import datetime, timedelta

def task_failure_callback(context):
    """Handle task failures."""
    task_instance = context['task_instance']
    error_message = context.get('exception')
    
    # Log error
    print(f"Task {task_instance.task_id} failed: {error_message}")
    
    # Send notification
    email_op = EmailOperator(
        task_id='send_error_email',
        to='team@example.com',
        subject=f'Task {task_instance.task_id} failed',
        html_content=f'''
            <p>Task failed with error:</p>
            <pre>{error_message}</pre>
            <p>DAG: {task_instance.dag_id}</p>
            <p>Execution Time: {task_instance.execution_date}</p>
        '''
    )
    email_op.execute(context)

default_args = {
    'owner': 'data-team',
    'depends_on_past': False,
    'email': ['data-team@example.com'],
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'on_failure_callback': task_failure_callback
}

with DAG(
    'error_handling_example',
    default_args=default_args,
    schedule_interval='0 * * * *',
    start_date=datetime(2024, 1, 1),
    catchup=False
) as dag:

    start = DummyOperator(task_id='start')
    
    def risky_operation():
        import random
        if random.random() < 0.5:
            raise ValueError("Random failure")
        return "Success"

    task1 = PythonOperator(
        task_id='risky_task',
        python_callable=risky_operation
    )

    end = DummyOperator(task_id='end')

    start >> task1 >> end
```

### 5. ตั้งค่า Monitoring:
```bash
# Create monitoring dashboard
cat > composer_dashboard.json << EOF
{
  "displayName": "Composer Monitoring",
  "gridLayout": {
    "widgets": [
      {
        "title": "DAG Success Rate",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"composer.googleapis.com/workflow/run_status\""
              }
            }
          }]
        }
      },
      {
        "title": "Task Duration",
        "xyChart": {
          "dataSets": [{
            "timeSeriesQuery": {
              "timeSeriesFilter": {
                "filter": "metric.type=\"composer.googleapis.com/workflow/task_duration\""
              }
            }
          }]
        }
      }
    ]
  }
}
EOF

gcloud monitoring dashboards create --config-from-file=composer_dashboard.json

# Create alerts
gcloud alpha monitoring policies create \
    --display-name="DAG Failure Alert" \
    --condition-display-name="DAG failure rate > 10%" \
    --condition-filter='metric.type="composer.googleapis.com/workflow/run_status"' \
    --condition-threshold-value=0.1 \
    --condition-threshold-duration=300s
```

## การทดสอบ | Testing
```python
# Test custom operator
from custom_operators import DataValidationOperator

def test_validation_function(data):
    # Add your validation logic here
    return {'is_valid': True, 'errors': []}

validation_task = DataValidationOperator(
    task_id='validate_data',
    project_id='your-project-id',
    bucket_name='your-bucket',
    validation_function=test_validation_function
)

# Test DAG
from airflow.models import DagBag
dag_bag = DagBag(include_examples=False)
dag_bag.process_file('data_processing_dag.py')
dag = dag_bag.get_dag('data_processing')

# Test task dependencies
assert dag.has_task('check_data')
assert dag.has_task('load_to_bigquery')
assert dag.has_task('transform_data')
```

## เพิ่มเติม | Additional Notes
- ใช้ XCom สำหรับ task communication
- ทำ unit testing สำหรับ custom operators
- ตั้งค่า alerting สำหรับ DAG failures
- ใช้ variables สำหรับ configuration
- พิจารณาใช้ Cloud Composer 2.0
