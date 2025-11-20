---
sidebar_position: 3
---

# 2.2 Storage Configuration

Learn how to configure and manage Cloud Storage, Cloud SQL, Bigtable, and other storage services on Google Cloud.

## Cloud Storage Configuration

### Creating and Configuring Buckets

```bash
# Create bucket with specific storage class
gsutil mb -c STANDARD -l us-central1 gs://my-bucket

# Create multi-region bucket
gsutil mb -c STANDARD -l US gs://my-multi-region-bucket

# Set default storage class
gsutil defstorageclass set NEARLINE gs://my-bucket
```

### Lifecycle Management

**Automatic Transition Example**:
```json
{
  "lifecycle": {
    "rule": [
      {
        "action": {
          "type": "SetStorageClass",
          "storageClass": "NEARLINE"
        },
        "condition": {
          "age": 30,
          "matchesStorageClass": ["STANDARD"]
        }
      },
      {
        "action": {
          "type": "SetStorageClass",
          "storageClass": "COLDLINE"
        },
        "condition": {
          "age": 90,
          "matchesStorageClass": ["NEARLINE"]
        }
      },
      {
        "action": {
          "type": "Delete"
        },
        "condition": {
          "age": 365,
          "isLive": false
        }
      }
    ]
  }
}
```

Apply lifecycle policy:
```bash
gsutil lifecycle set lifecycle.json gs://my-bucket
```

### Versioning and Object Retention

```bash
# Enable versioning
gsutil versioning set on gs://my-bucket

# Set retention policy (cannot delete for 30 days)
gsutil retention set 30d gs://my-bucket

# Lock retention policy (IRREVERSIBLE!)
gsutil retention lock gs://my-bucket

# View retention policy
gsutil retention get gs://my-bucket
```

### Access Control

**Uniform Bucket-Level Access** (Recommended):
```bash
# Enable uniform access
gsutil uniformbucketlevelaccess set on gs://my-bucket

# Grant access via IAM
gsutil iam ch user:alice@example.com:objectViewer gs://my-bucket
gsutil iam ch serviceAccount:sa@project.iam.gserviceaccount.com:objectAdmin gs://my-bucket
```

**Signed URLs** (Temporary Access):
```bash
# Generate signed URL (valid for 1 hour)
gsutil signurl -d 1h private-key.json gs://my-bucket/file.pdf
```

### CORS Configuration

```json
[
  {
    "origin": ["https://example.com"],
    "method": ["GET", "POST"],
    "responseHeader": ["Content-Type"],
    "maxAgeSeconds": 3600
  }
]
```

Apply CORS:
```bash
gsutil cors set cors.json gs://my-bucket
```

---

## Cloud SQL Configuration

### Creating Instances

```bash
# Create MySQL instance with HA
gcloud sql instances create my-mysql \
  --database-version=MYSQL_8_0 \
  --tier=db-n1-standard-2 \
  --region=us-central1 \
  --availability-type=REGIONAL \
  --backup-start-time=03:00 \
  --enable-bin-log \
  --retained-backups-count=7 \
  --retained-transaction-log-days=7

# Create PostgreSQL instance
gcloud sql instances create my-postgres \
  --database-version=POSTGRES_14 \
  --tier=db-custom-4-16384 \
  --region=us-central1 \
  --availability-type=ZONAL
```

### High Availability Configuration

**Regional HA** (Automatic Failover):
```bash
gcloud sql instances patch my-mysql \
  --availability-type=REGIONAL
```

**Architecture**:
```
Primary Instance (Zone A)
    ↓ (Synchronous Replication)
Standby Instance (Zone B)
    ↓ (Automatic Failover: ~60 seconds)
```

### Read Replicas

```bash
# Create read replica in same region
gcloud sql instances create my-mysql-replica \
  --master-instance-name=my-mysql \
  --tier=db-n1-standard-1 \
  --region=us-central1

# Create cross-region read replica
gcloud sql instances create my-mysql-replica-eu \
  --master-instance-name=my-mysql \
  --tier=db-n1-standard-1 \
  --region=europe-west1

# Promote replica to standalone
gcloud sql instances promote-replica my-mysql-replica
```

### Backup and Recovery

```bash
# Create on-demand backup
gcloud sql backups create \
  --instance=my-mysql \
  --description="Before migration"

# Restore from backup
gcloud sql backups restore BACKUP_ID \
  --backup-instance=my-mysql \
  --backup-id=1234567890

# Point-in-time recovery
gcloud sql instances clone my-mysql my-mysql-clone \
  --point-in-time='2024-01-15T10:30:00.000Z'
```

### Connection Options

**Private IP** (Recommended for Production):
```bash
gcloud sql instances patch my-mysql \
  --network=projects/PROJECT_ID/global/networks/my-vpc \
  --no-assign-ip
```

**Cloud SQL Proxy**:
```bash
# Download proxy
curl -o cloud-sql-proxy https://storage.googleapis.com/cloud-sql-connectors/cloud-sql-proxy/v2.8.0/cloud-sql-proxy.linux.amd64

# Run proxy
./cloud-sql-proxy PROJECT:REGION:INSTANCE
```

**Application Connection**:
```python
import sqlalchemy

# Create connection pool
pool = sqlalchemy.create_engine(
    sqlalchemy.engine.url.URL.create(
        drivername="mysql+pymysql",
        username="root",
        password="password",
        database="mydb",
        query={"unix_socket": "/cloudsql/PROJECT:REGION:INSTANCE"}
    ),
    pool_size=5,
    max_overflow=2,
    pool_timeout=30,
)
```

---

## Bigtable Configuration

### Creating Instances and Clusters

```bash
# Create production instance with SSD
gcloud bigtable instances create my-bt-instance \
  --display-name="Production Bigtable" \
  --cluster=my-cluster \
  --cluster-zone=us-central1-a \
  --cluster-num-nodes=3 \
  --cluster-storage-type=SSD

# Add replication cluster
gcloud bigtable clusters create my-cluster-replica \
  --instance=my-bt-instance \
  --zone=us-east1-b \
  --num-nodes=3
```

### Table and Column Family Design

```bash
# Create table using cbt CLI
echo "project = my-project
instance = my-bt-instance" > ~/.cbtrc

# Create table
cbt createtable events

# Create column families
cbt createfamily events metadata
cbt createfamily events data

# Set garbage collection policy (keep last 7 days)
cbt setgcpolicy events metadata maxage=7d
```

### Row Key Design Patterns

**Bad Design** (Hotspotting):
```
user123#2024-01-15T10:00:00
user123#2024-01-15T10:01:00
user456#2024-01-15T10:02:00
```

**Good Design** (Distributed):
```
# Reverse timestamp
20240115100000#user123
20240115100100#user123
20240115100200#user456

# Or hash prefix
a3f2#user123#2024-01-15T10:00:00
b7e1#user123#2024-01-15T10:01:00
```

### Performance Optimization

**Scaling**:
```bash
# Scale up nodes
gcloud bigtable clusters update my-cluster \
  --instance=my-bt-instance \
  --num-nodes=10

# Enable autoscaling
gcloud bigtable clusters update my-cluster \
  --instance=my-bt-instance \
  --autoscaling-min-nodes=3 \
  --autoscaling-max-nodes=10 \
  --autoscaling-cpu-target=70
```

**Key Visualizer**: Monitor row key distribution
```bash
# Enable in Console: Bigtable > Instance > Key Visualizer
```

---

## Firestore Configuration

### Creating Database

```bash
# Create Firestore database (Native mode)
gcloud firestore databases create \
  --location=us-central \
  --type=firestore-native

# Create index
gcloud firestore indexes composite create \
  --collection-group=users \
  --field-config field-path=age,order=ASCENDING \
  --field-config field-path=city,order=ASCENDING
```

### Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users can only read/write their own data
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    
    // Public read, authenticated write
    match /posts/{postId} {
      allow read: if true;
      allow write: if request.auth != null;
    }
    
    // Admin only
    match /admin/{document=**} {
      allow read, write: if request.auth.token.admin == true;
    }
  }
}
```

### Data Modeling

**Example Structure**:
```
users (collection)
  ├── user123 (document)
  │   ├── name: "John Doe"
  │   ├── email: "john@example.com"
  │   └── orders (subcollection)
  │       ├── order456 (document)
  │       │   ├── total: 99.99
  │       │   └── items: [...]
  │       └── order789 (document)
```

**Querying**:
```javascript
// Simple query
db.collection('users')
  .where('age', '>=', 18)
  .where('city', '==', 'NYC')
  .orderBy('age')
  .limit(10)
  .get();

// Real-time listener
db.collection('users').doc('user123')
  .onSnapshot((doc) => {
    console.log('Current data:', doc.data());
  });
```

---

## BigQuery Configuration

### Creating Datasets and Tables

```bash
# Create dataset
bq mk --dataset \
  --location=US \
  --description="Analytics data" \
  my_project:analytics

# Create partitioned table
bq mk --table \
  --time_partitioning_field=timestamp \
  --clustering_fields=user_id,event_type \
  --schema=schema.json \
  my_project:analytics.events
```

**Schema Example** (schema.json):
```json
[
  {"name": "timestamp", "type": "TIMESTAMP", "mode": "REQUIRED"},
  {"name": "user_id", "type": "STRING", "mode": "REQUIRED"},
  {"name": "event_type", "type": "STRING", "mode": "REQUIRED"},
  {"name": "properties", "type": "JSON", "mode": "NULLABLE"}
]
```

### Loading Data

```bash
# Load from Cloud Storage
bq load \
  --source_format=NEWLINE_DELIMITED_JSON \
  --autodetect \
  analytics.events \
  gs://my-bucket/data/*.json

# Load from local file
bq load \
  --source_format=CSV \
  --skip_leading_rows=1 \
  analytics.users \
  users.csv

# Streaming insert (via API)
```

```python
from google.cloud import bigquery

client = bigquery.Client()
table_id = "project.dataset.table"

rows_to_insert = [
    {"timestamp": "2024-01-15T10:00:00", "user_id": "123", "event": "click"},
    {"timestamp": "2024-01-15T10:01:00", "user_id": "456", "event": "view"},
]

errors = client.insert_rows_json(table_id, rows_to_insert)
if errors:
    print(f"Errors: {errors}")
```

### Query Optimization

**Partitioning**:
```sql
-- Query only recent data (reduces cost)
SELECT *
FROM `project.analytics.events`
WHERE DATE(timestamp) >= DATE_SUB(CURRENT_DATE(), INTERVAL 7 DAY)
  AND user_id = '123';
```

**Clustering**:
```sql
-- Clustering on user_id improves performance
SELECT user_id, COUNT(*) as event_count
FROM `project.analytics.events`
WHERE user_id IN ('123', '456', '789')
GROUP BY user_id;
```

**Materialized Views**:
```sql
CREATE MATERIALIZED VIEW `project.analytics.daily_summary`
AS
SELECT
  DATE(timestamp) as date,
  user_id,
  COUNT(*) as event_count
FROM `project.analytics.events`
GROUP BY date, user_id;
```

---

## Summary

**Key Takeaways**:
1. Use **lifecycle policies** to automatically transition Cloud Storage objects
2. Enable **HA** for production Cloud SQL instances
3. Design **row keys carefully** in Bigtable to avoid hotspotting
4. Use **partitioning and clustering** in BigQuery to reduce costs
5. Implement **security rules** in Firestore to protect data
6. Use **Cloud SQL Proxy** for secure database connections
7. Enable **versioning** for critical Cloud Storage buckets
