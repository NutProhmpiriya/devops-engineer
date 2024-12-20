# แบบฝึกหัดที่ 12: การใช้งาน Cloud Spanner
# Exercise 12: Using Cloud Spanner

## โจทย์ | Problem
ออกแบบและจัดการฐานข้อมูล Cloud Spanner สำหรับแอปพลิเคชันระดับ global โดยมีความต้องการดังนี้:

1. สร้าง:
   - Instance configuration
   - Database schema
   - Indexes
   - Interleaved tables

2. จัดการ:
   - Transactions
   - Backups
   - IAM permissions
   - Monitoring

## เฉลย | Solution

### 1. สร้าง Instance:
```bash
# Create Spanner instance
gcloud spanner instances create production-instance \
    --config=regional-asia-southeast1 \
    --description="Production Instance" \
    --nodes=3

# Create database
gcloud spanner databases create myapp \
    --instance=production-instance
```

### 2. สร้าง Schema (schema.sql):
```sql
-- Create Singers table
CREATE TABLE Singers (
  SingerId    INT64 NOT NULL,
  FirstName   STRING(1024),
  LastName    STRING(1024),
  BirthDate   DATE,
  JoinedAt    TIMESTAMP,
  Country     STRING(1024),
) PRIMARY KEY (SingerId);

-- Create Albums table (interleaved in Singers)
CREATE TABLE Albums (
  SingerId    INT64 NOT NULL,
  AlbumId     INT64 NOT NULL,
  Title       STRING(1024),
  ReleaseDate DATE,
  Genre       STRING(1024),
  Sales       INT64,
) PRIMARY KEY (SingerId, AlbumId),
  INTERLEAVE IN PARENT Singers ON DELETE CASCADE;

-- Create Songs table (interleaved in Albums)
CREATE TABLE Songs (
  SingerId    INT64 NOT NULL,
  AlbumId     INT64 NOT NULL,
  TrackId     INT64 NOT NULL,
  Title       STRING(1024),
  Duration    INT64,
  Lyrics      STRING(MAX),
) PRIMARY KEY (SingerId, AlbumId, TrackId),
  INTERLEAVE IN PARENT Albums ON DELETE CASCADE;

-- Create secondary index
CREATE INDEX SingersByCountry ON Singers(Country);
CREATE INDEX AlbumsByReleaseDate ON Albums(ReleaseDate);

-- Create storing index
CREATE INDEX AlbumsBySales ON Albums(Sales) STORING (Title, ReleaseDate);
```

### 3. Python Client Code (spanner_client.py):
```python
from google.cloud import spanner
from google.cloud.spanner_v1 import param_types

def init_client():
    client = spanner.Client()
    instance = client.instance('production-instance')
    database = instance.database('myapp')
    return database

def insert_singer(database, singer_id, first_name, last_name, country):
    with database.batch() as batch:
        batch.insert(
            table='Singers',
            columns=('SingerId', 'FirstName', 'LastName', 'Country'),
            values=[(singer_id, first_name, last_name, country)]
        )

def read_singers_by_country(database, country):
    with database.snapshot() as snapshot:
        results = snapshot.execute_sql(
            'SELECT SingerId, FirstName, LastName '
            'FROM Singers '
            'WHERE Country = @country',
            params={'country': country},
            param_types={'country': param_types.STRING}
        )
        return list(results)

def insert_album_transaction(database, singer_id, album_id, title, release_date):
    def insert_album(transaction):
        # Check if singer exists
        row = transaction.execute_sql(
            'SELECT 1 FROM Singers WHERE SingerId = @id',
            params={'id': singer_id},
            param_types={'id': param_types.INT64}
        ).one_or_none()
        
        if not row:
            raise ValueError('Singer not found')
            
        # Insert album
        transaction.insert(
            table='Albums',
            columns=('SingerId', 'AlbumId', 'Title', 'ReleaseDate'),
            values=[(singer_id, album_id, title, release_date)]
        )
    
    database.run_in_transaction(insert_album)
```

### 4. ตั้งค่า Backup:
```bash
# Create backup
gcloud spanner backups create myapp-backup \
    --instance=production-instance \
    --database=myapp \
    --expiration-date=2024-12-31

# List backups
gcloud spanner backups list \
    --instance=production-instance

# Restore from backup
gcloud spanner databases restore myapp-restored \
    --instance=production-instance \
    --source-backup=myapp-backup
```

### 5. ตั้งค่า IAM:
```bash
# Create service account
gcloud iam service-accounts create spanner-app \
    --display-name="Spanner Application"

# Grant permissions
gcloud spanner instances add-iam-policy-binding production-instance \
    --member="serviceAccount:spanner-app@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/spanner.databaseUser"

# Grant backup admin role
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:spanner-app@$PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/spanner.backupAdmin"
```

### 6. ตั้งค่า Monitoring:
```bash
# Create alert policy for high CPU
gcloud alpha monitoring policies create \
    --display-name="Spanner High CPU" \
    --condition-display-name="CPU > 80%" \
    --condition-filter='resource.type="spanner_instance" AND metric.type="spanner.googleapis.com/instance/cpu/utilization"' \
    --condition-threshold-value=0.8 \
    --condition-threshold-duration=300s

# Create alert for storage usage
gcloud alpha monitoring policies create \
    --display-name="Spanner Storage" \
    --condition-display-name="Storage > 80%" \
    --condition-filter='resource.type="spanner_instance" AND metric.type="spanner.googleapis.com/instance/storage/utilization"' \
    --condition-threshold-value=0.8 \
    --condition-threshold-duration=300s
```

## การทดสอบ | Testing
```python
# Test data operations
database = init_client()

# Insert test data
insert_singer(database, 1, "John", "Doe", "USA")
insert_singer(database, 2, "Jane", "Smith", "UK")

# Test read operations
singers = read_singers_by_country(database, "USA")
print(singers)

# Test transaction
try:
    insert_album_transaction(database, 1, 1, "First Album", "2024-01-01")
    print("Album inserted successfully")
except ValueError as e:
    print(f"Error: {e}")

# Test queries
with database.snapshot() as snapshot:
    results = snapshot.execute_sql('''
        SELECT s.FirstName, s.LastName, a.Title
        FROM Singers s
        JOIN Albums a ON s.SingerId = a.SingerId
        WHERE s.Country = @country
    ''', params={'country': 'USA'}, 
        param_types={'country': param_types.STRING})
    
    for row in results:
        print(f"{row[0]} {row[1]}: {row[2]}")
```

## เพิ่มเติม | Additional Notes
- ใช้ connection pooling สำหรับ production
- ทำ performance testing ก่อน deployment
- ตั้งค่า automatic scaling
- ใช้ mutations แทน DML สำหรับ bulk operations
- พิจารณา partition queries สำหรับ large datasets
