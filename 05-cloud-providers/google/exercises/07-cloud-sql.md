# แบบฝึกหัดที่ 7: การจัดการ Cloud SQL
# Exercise 7: Managing Cloud SQL

## โจทย์ | Problem
ตั้งค่าและจัดการ Cloud SQL instance สำหรับแอปพลิเคชัน โดยมีความต้องการดังนี้:

1. สร้าง Cloud SQL instance with:
   - High availability configuration
   - Automated backups
   - Private IP access
   - SSL/TLS encryption

2. ตั้งค่า:
   - Read replicas
   - Monitoring และ alerting
   - Database proxy
   - Maintenance windows

## เฉลย | Solution

### 1. สร้าง Cloud SQL Instance:
```bash
# Create VPC network for private IP
gcloud compute networks create sql-network \
    --subnet-mode=custom

gcloud compute networks subnets create sql-subnet \
    --network=sql-network \
    --region=asia-southeast1 \
    --range=10.0.0.0/24

# Create Cloud SQL instance
gcloud sql instances create main-instance \
    --database-version=POSTGRES_13 \
    --cpu=2 \
    --memory=4GB \
    --region=asia-southeast1 \
    --availability-type=REGIONAL \
    --storage-type=SSD \
    --storage-size=100GB \
    --backup-start-time=23:00 \
    --maintenance-window-day=SUN \
    --maintenance-window-hour=02 \
    --network=sql-network \
    --no-assign-ip \
    --require-ssl
```

### 2. ตั้งค่า Read Replicas:
```bash
# Create read replica
gcloud sql instances create read-replica-1 \
    --master-instance-name=main-instance \
    --region=asia-southeast1-b \
    --availability-type=ZONAL

# Create another read replica in different zone
gcloud sql instances create read-replica-2 \
    --master-instance-name=main-instance \
    --region=asia-southeast1-c \
    --availability-type=ZONAL
```

### 3. ตั้งค่า Cloud SQL Proxy:
```bash
# Download Cloud SQL Proxy
wget https://dl.google.com/cloudsql/cloud_sql_proxy_x64.linux

# Make it executable
chmod +x cloud_sql_proxy_x64.linux

# Create service account for proxy
gcloud iam service-accounts create sql-proxy \
    --display-name="Cloud SQL Proxy"

# Grant necessary permissions
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:sql-proxy@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/cloudsql.client"

# Create key for service account
gcloud iam service-accounts keys create key.json \
    --iam-account=sql-proxy@$PROJECT_ID.iam.gserviceaccount.com

# Run proxy
./cloud_sql_proxy_x64.linux \
    -instances=$PROJECT_ID:asia-southeast1:main-instance=tcp:5432 \
    -credential_file=key.json
```

### 4. ตั้งค่า Monitoring:
```bash
# Create alert policy for high CPU usage
gcloud beta monitoring policies create \
    --display-name="High SQL CPU Usage" \
    --condition-display-name="CPU usage above 80%" \
    --condition-filter='resource.type="cloudsql_database" AND metric.type="cloudsql.googleapis.com/database/cpu/utilization"' \
    --condition-threshold-value=0.8 \
    --condition-threshold-duration=300s \
    --notification-channels="projects/$PROJECT_ID/notificationChannels/$CHANNEL_ID"

# Create alert for storage usage
gcloud beta monitoring policies create \
    --display-name="High Storage Usage" \
    --condition-display-name="Storage usage above 85%" \
    --condition-filter='resource.type="cloudsql_database" AND metric.type="cloudsql.googleapis.com/database/disk/bytes_used"' \
    --condition-threshold-value=85 \
    --condition-threshold-duration=300s \
    --notification-channels="projects/$PROJECT_ID/notificationChannels/$CHANNEL_ID"
```

### 5. สร้าง Connection Script (connect.sh):
```bash
#!/bin/bash

# Export credentials
export GOOGLE_APPLICATION_CREDENTIALS="key.json"

# Start proxy in background
./cloud_sql_proxy_x64.linux \
    -instances=$PROJECT_ID:asia-southeast1:main-instance=tcp:5432 \
    -credential_file=$GOOGLE_APPLICATION_CREDENTIALS &

# Wait for proxy to start
sleep 5

# Connect to database
PGPASSWORD=$DB_PASSWORD psql \
    -h localhost \
    -p 5432 \
    -U $DB_USER \
    -d $DB_NAME
```

### 6. ตั้งค่า Backup และ Recovery:
```bash
# Create on-demand backup
gcloud sql backups create --instance=main-instance

# Configure point-in-time recovery
gcloud sql instances patch main-instance \
    --backup-start-time=23:00 \
    --enable-point-in-time-recovery

# List available backups
gcloud sql backups list --instance=main-instance
```

## การทดสอบ | Testing
```bash
# Test connection
./connect.sh

# Check instance status
gcloud sql instances describe main-instance

# Monitor metrics
gcloud monitoring metrics list \
    --filter="metric.type=cloudsql.googleapis.com/database/cpu/utilization"

# Test failover
gcloud sql instances failover main-instance

# Check replica lag
gcloud monitoring metrics list \
    --filter="metric.type=cloudsql.googleapis.com/database/replication/lag"
```

## เพิ่มเติม | Additional Notes
- ใช้ connection pooling สำหรับ production workloads
- ตั้งค่า automated scaling ตามความเหมาะสม
- ทำ regular backup testing
- ใช้ Cloud KMS สำหรับ encryption at rest
- พิจารณาใช้ Cloud SQL Auth Proxy ในทุก environment
