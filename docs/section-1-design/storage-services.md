---
sidebar_position: 5
---

# 1.4 Storage Services

Master Google Cloud storage options and learn when to use each service for different data types and access patterns.

## Storage Decision Framework

### Key Questions

1. **Data Type**: Structured, unstructured, or semi-structured?
2. **Access Pattern**: Frequent, infrequent, or archival?
3. **Query Requirements**: SQL, NoSQL, or object access?
4. **Scale**: GB, TB, or PB?
5. **Consistency**: Strong or eventual?
6. **Global vs Regional**: Single region or worldwide access?

---

## Object Storage: Cloud Storage

### When to Use Cloud Storage

**Best For**:
- Images, videos, backups
- Static website hosting
- Data lakes
- Archive storage
- Serving user-generated content

### Storage Classes

| Class | Access Frequency | Min Storage Duration | Retrieval Cost | Use Case |
|-------|-----------------|---------------------|----------------|----------|
| **Standard** | Frequent (>1/month) | None | None | Hot data, websites, streaming |
| **Nearline** | ~1/month | 30 days | $0.01/GB | Backups, disaster recovery |
| **Coldline** | ~1/quarter | 90 days | $0.02/GB | Long-term backups, compliance |
| **Archive** | <1/year | 365 days | $0.05/GB | Regulatory archives, rarely accessed |

### Lifecycle Management

```json
{
  "lifecycle": {
    "rule": [
      {
        "action": {"type": "SetStorageClass", "storageClass": "NEARLINE"},
        "condition": {"age": 30}
      },
      {
        "action": {"type": "SetStorageClass", "storageClass": "COLDLINE"},
        "condition": {"age": 90}
      },
      {
        "action": {"type": "SetStorageClass", "storageClass": "ARCHIVE"},
        "condition": {"age": 365}
      },
      {
        "action": {"type": "Delete"},
        "condition": {"age": 2555}
      }
    ]
  }
}
```

### Versioning and Retention

```bash
# Enable versioning
gsutil versioning set on gs://my-bucket

# Set retention policy (60 days)
gsutil retention set 60d gs://my-bucket

# Lock retention policy (irreversible!)
gsutil retention lock gs://my-bucket
```

---

## Relational Databases

### Cloud SQL

**Best For**:
- Traditional relational workloads
- MySQL, PostgreSQL, SQL Server
- Regional scale (up to ~64 TB)
- Existing apps using these databases

**Features**:
- Automated backups
- Point-in-time recovery
- Read replicas
- High availability (99.95% SLA)

**Configuration**:
```bash
# Create MySQL instance with HA
gcloud sql instances create my-db \
  --database-version=MYSQL_8_0 \
  --tier=db-n1-standard-2 \
  --region=us-central1 \
  --availability-type=REGIONAL \
  --backup-start-time=03:00

# Create read replica
gcloud sql instances create my-db-replica \
  --master-instance-name=my-db \
  --tier=db-n1-standard-1 \
  --region=us-east1
```

**Pricing**: ~$50-500/month depending on size

### Cloud Spanner

**Best For**:
- Global applications
- Strong consistency required
- Horizontal scaling needed
- Mission-critical relational data

**Key Features**:
- 99.999% availability (5 nines)
- Global transactions
- Automatic sharding
- SQL interface

**When to Choose Spanner**:
```
Need global scale? YES → Spanner
Need strong consistency? YES → Spanner
Budget sensitive? NO → Cloud SQL
Data < 10 TB? → Cloud SQL probably sufficient
```

**Configuration**:
```bash
# Create Spanner instance
gcloud spanner instances create my-instance \
  --config=regional-us-central1 \
  --nodes=1 \
  --description="Production database"

# Create database
gcloud spanner databases create my-db \
  --instance=my-instance
```

**Pricing**: ~$900/month per node (minimum)

---

## NoSQL Databases

### Bigtable

**Best For**:
- Time-series data (IoT sensors, stock prices)
- High-throughput analytics
- Low-latency access (< 10ms)
- Flat data (no complex joins)

**Use Cases**:
- AdTech (billions of events/day)
- Financial data (tick data)
- IoT telemetry
- User analytics

**Key Concepts**:
- **Row Key Design**: Critical for performance
- **Column Families**: Group related columns
- **Timestamps**: Built-in versioning

**Row Key Design Examples**:
```
Bad:  user_id#timestamp  (hotspotting)
Good: reverse(timestamp)#user_id  (distributed)
Good: hash(user_id)#timestamp  (distributed)
```

**Configuration**:
```bash
# Create Bigtable instance
gcloud bigtable instances create my-bt-instance \
  --display-name="Production Bigtable" \
  --cluster=my-cluster \
  --cluster-zone=us-central1-a \
  --cluster-num-nodes=3
```

**Pricing**: ~$2,000/month for 3 nodes

### Firestore

**Best For**:
- Mobile/web applications
- Real-time synchronization
- Offline support
- Hierarchical data

**Data Model**:
```
Collection: users
  Document: user123
    name: "John"
    email: "john@example.com"
    SubCollection: orders
      Document: order456
        total: 99.99
        items: [...]
```

**Modes**:
- **Native Mode**: New applications, better scaling
- **Datastore Mode**: Backward compatible with Datastore

**Example Query**:
```javascript
// JavaScript SDK
const usersRef = db.collection('users');
const query = usersRef
  .where('age', '>=', 18)
  .where('city', '==', 'NYC')
  .orderBy('age')
  .limit(10);
```

---

## Data Warehouse: BigQuery

**Best For**:
- Analytics on massive datasets
- Business intelligence
- Data science / ML
- Petabyte-scale queries

**Key Features**:
- Serverless (no infrastructure)
- Columnar storage
- SQL interface
- Automatic scaling

**Pricing Model**:
- **On-demand**: $5/TB scanned
- **Flat-rate**: $2,000/month for 100 slots

**Example Query**:
```sql
-- Analyze website traffic
SELECT
  DATE(timestamp) as date,
  COUNT(*) as pageviews,
  COUNT(DISTINCT user_id) as unique_users
FROM `project.dataset.pageviews`
WHERE timestamp >= TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 30 DAY)
GROUP BY date
ORDER BY date DESC;
```

**Partitioning** (reduces cost):
```sql
CREATE TABLE `project.dataset.events`
PARTITION BY DATE(timestamp)
AS SELECT * FROM source_table;
```

---

## File Storage: Filestore

**Best For**:
- Applications requiring POSIX filesystem
- Lift-and-shift of file-based apps
- Shared storage for VMs/GKE

**Tiers**:
- **Basic HDD**: 1-63.9 TB, ~$0.20/GB/month
- **Basic SSD**: 2.5-63.9 TB, ~$0.30/GB/month
- **Enterprise**: High availability, ~$0.35/GB/month

**Use Case**: Shared NFS mount for multiple VMs

```bash
# Create Filestore instance
gcloud filestore instances create my-filestore \
  --zone=us-central1-a \
  --tier=BASIC_HDD \
  --file-share=name=share1,capacity=1TB \
  --network=name=default
```

---

## In-Memory Cache: Memorystore

### Memorystore for Redis

**Best For**:
- Session storage
- Caching layer
- Real-time analytics
- Leaderboards, counters

**Tiers**:
- **Basic**: Single node, no replication
- **Standard**: HA with automatic failover

```bash
# Create Redis instance
gcloud redis instances create my-cache \
  --size=5 \
  --region=us-central1 \
  --tier=STANDARD \
  --redis-version=redis_6_x
```

### Memorystore for Memcached

**Best For**:
- Simple key-value caching
- Lower cost than Redis
- No persistence needed

---

## Storage Decision Matrix

| Requirement | Solution |
|-------------|----------|
| Object storage (images, videos) | **Cloud Storage** |
| Relational, regional, < 64 TB | **Cloud SQL** |
| Relational, global, strong consistency | **Cloud Spanner** |
| Time-series, high throughput, low latency | **Bigtable** |
| Mobile/web app, real-time sync | **Firestore** |
| Analytics, petabyte-scale | **BigQuery** |
| File system (NFS) | **Filestore** |
| Caching, session storage | **Memorystore (Redis)** |

---

## Cost Optimization Tips

### Cloud Storage
1. Use lifecycle policies to move to cheaper classes
2. Enable object versioning only if needed
3. Use Coldline/Archive for compliance data
4. Delete old versions automatically

### Cloud SQL
1. Use committed use discounts (1-3 years)
2. Stop instances during non-business hours (dev/test)
3. Use read replicas to offload read traffic
4. Choose appropriate machine type (don't over-provision)

### BigQuery
1. Use partitioned tables
2. Use clustered tables for common filters
3. Avoid `SELECT *` (specify columns)
4. Use flat-rate pricing for predictable workloads
5. Set table expiration for temporary data

---

## Summary

**Key Takeaways**:
1. **Cloud Storage** for unstructured data with lifecycle policies
2. **Cloud SQL** for regional relational databases
3. **Cloud Spanner** for global relational databases
4. **Bigtable** for high-throughput time-series data
5. **Firestore** for mobile/web apps with real-time sync
6. **BigQuery** for analytics and data warehousing
7. Choose based on data type, access pattern, and scale requirements
