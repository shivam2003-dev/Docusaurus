---
sidebar_position: 10
---

# Section 1: Practice Questions (50+)

Test your knowledge of designing and planning cloud solution architecture with these scenario-based questions.

---

### Question 1
**Scenario**: Your company is migrating a 100TB MySQL database to GCP. The database must remain operational during migration with minimal downtime.
**What should you do?**

*   A. Use `mysqldump` to export and import the database
*   B. Use Database Migration Service with continuous replication
*   C. Use Transfer Appliance
*   D. Manually copy data using `gsutil`

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Database Migration Service with continuous replication)**
*   **Explanation**: DMS provides continuous replication, allowing minimal downtime. The database stays operational during migration.
*   *mysqldump* requires significant downtime for large databases.
*   *Transfer Appliance* is for offline data transfer, not databases.
</details>

---

### Question 2
**Scenario**: You need to store 50 TB of financial data for 7 years due to compliance regulations. The data will rarely be accessed (once a year at most). You want to minimize cost.
**Which storage class should you choose?**

*   A. Standard
*   B. Nearline
*   C. Coldline
*   D. Archive

<details>
<summary>Click to reveal answer</summary>

**Answer: D (Archive)**
*   **Explanation**: Archive storage is the lowest cost for data accessed less than once a year.
*   *Standard*: Hot data (frequent access).
*   *Nearline*: Once a month.
*   *Coldline*: Once a quarter.
</details>

---

### Question 3
**Scenario**: You are designing a global multiplayer game backend. You need a database that scales horizontally to handle millions of users worldwide and provides strong consistency.
**Which database service should you use?**

*   A. Cloud SQL
*   B. Cloud Spanner
*   C. Bigtable
*   D. Firestore

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Cloud Spanner)**
*   **Explanation**: Cloud Spanner is the only globally distributed, strongly consistent, relational database service.
*   *Cloud SQL*: Regional only.
*   *Bigtable*: NoSQL, eventual consistency (unless single cluster), not relational.
</details>

---

### Question 4
**Scenario**: Your application needs to process 10 million IoT sensor readings per second with sub-10ms latency. Data is time-series and will be queried by sensor ID and timestamp.
**Which storage solution should you use?**

*   A. Cloud SQL
*   B. Cloud Spanner
*   C. Bigtable
*   D. BigQuery

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Bigtable)**
*   **Explanation**: Bigtable is designed for high-throughput, low-latency workloads like IoT time-series data.
*   *BigQuery* is for analytics, not real-time ingestion.
*   *Spanner* is expensive for this use case.
</details>

---

### Question 5
**Scenario**: You need to run a batch processing job that takes 8 hours to complete. The job can tolerate interruptions and restart from checkpoints.
**What is the most cost-effective compute option?**

*   A. Standard Compute Engine VMs
*   B. Preemptible VMs
*   C. Cloud Run
*   D. App Engine Flexible

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Preemptible VMs)**
*   **Explanation**: Preemptible VMs are up to 80% cheaper and perfect for fault-tolerant batch jobs.
*   *Cloud Run* and *App Engine* are for web services, not batch processing.
</details>

---

### Question 6
**Scenario**: Your on-premises data center needs a dedicated, private connection to GCP with 10 Gbps bandwidth and an SLA.
**Which service should you use?**

*   A. Cloud VPN
*   B. Cloud Interconnect - Dedicated
*   C. Cloud Interconnect - Partner
*   D. Direct Peering

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Cloud Interconnect - Dedicated)**
*   **Explanation**: Dedicated Interconnect provides 10 Gbps or 100 Gbps with an SLA.
*   *VPN* maxes out at 3 Gbps per tunnel.
*   *Direct Peering* is for Google services (Workspace), not GCP private network.
</details>

---

### Question 7
**Scenario**: You have a web application that must serve users globally with the lowest possible latency. Traffic should be routed to the nearest healthy backend.
**Which load balancer should you use?**

*   A. External Network Load Balancer
*   B. Internal HTTP(S) Load Balancer
*   C. External HTTP(S) Load Balancer
*   D. External TCP/UDP Load Balancer

<details>
<summary>Click to reveal answer</summary>

**Answer: C (External HTTP(S) Load Balancer)**
*   **Explanation**: This is the only global load balancer that routes HTTP(S) traffic to the nearest backend.
*   *Network* and *TCP/UDP* load balancers are regional only.
</details>

---

### Question 8
**Scenario**: Your company requires that all data stored in GCP must be encrypted with keys that you manage and can revoke at any time.
**Which encryption option should you use?**

*   A. Default encryption (Google-managed keys)
*   B. Customer-Managed Encryption Keys (CMEK)
*   C. Customer-Supplied Encryption Keys (CSEK)
*   D. Client-side encryption

<details>
<summary>Click to reveal answer</summary>

**Answer: B (CMEK)**
*   **Explanation**: CMEK allows you to manage keys in Cloud KMS and revoke them at any time.
*   *CSEK* requires you to supply keys with every API call (not managed in GCP).
</details>

---

### Question 9
**Scenario**: You need to deploy a stateless containerized application that scales to zero when not in use and handles HTTP requests.
**Which service is most appropriate?**

*   A. Compute Engine with MIG
*   B. GKE Autopilot
*   C. Cloud Run
*   D. App Engine Standard

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Cloud Run)**
*   **Explanation**: Cloud Run is serverless, scales to zero, and is designed for stateless containers.
*   *GKE* doesn't scale to zero.
*   *App Engine Standard* requires specific runtimes.
</details>

---

### Question 10
**Scenario**: Your application requires a relational database that can handle 100,000 writes per second globally with strong consistency.
**Which database should you choose?**

*   A. Cloud SQL with read replicas
*   B. Cloud Spanner
*   C. Firestore
*   D. Bigtable

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Cloud Spanner)**
*   **Explanation**: Spanner is the only relational database that scales globally with strong consistency.
*   *Cloud SQL* is regional and can't handle this write throughput.
</details>

---

### Question 11
**Scenario**: You need to migrate 500 TB of data from your on-premises data center to Cloud Storage. Your internet connection is 100 Mbps.
**What is the most efficient method?**

*   A. Use `gsutil` to upload over the internet
*   B. Use Storage Transfer Service
*   C. Use Transfer Appliance
*   D. Use Cloud Interconnect

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Transfer Appliance)**
*   **Explanation**: At 100 Mbps, uploading 500 TB would take months. Transfer Appliance is a physical device shipped to you.
*   *gsutil* and *Storage Transfer Service* use your internet connection.
</details>

---

### Question 12
**Scenario**: Your application needs to process streaming data from Pub/Sub and write aggregated results to BigQuery in real-time.
**Which service should you use?**

*   A. Cloud Functions
*   B. Dataflow
*   C. Dataproc
*   D. Cloud Run

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Dataflow)**
*   **Explanation**: Dataflow is designed for stream processing with built-in Pub/Sub and BigQuery connectors.
*   *Cloud Functions* has a 10-minute timeout limit.
*   *Dataproc* is for Spark/Hadoop batch jobs.
</details>

---

### Question 13
**Scenario**: You need to ensure that VMs in your VPC can only access specific Google APIs (like Cloud Storage) and cannot reach the public internet.
**What should you configure?**

*   A. Cloud NAT
*   B. Private Google Access
*   C. VPC Service Controls
*   D. Cloud Armor

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Private Google Access)**
*   **Explanation**: Private Google Access allows VMs without external IPs to access Google APIs.
*   *VPC Service Controls* prevents data exfiltration, not internet access.
</details>

---

### Question 14
**Scenario**: Your company wants to enforce that all Compute Engine VMs must be created in specific regions (us-central1, europe-west1) only.
**How should you implement this?**

*   A. Use IAM policies
*   B. Use Organization Policy with `gcp.resourceLocations` constraint
*   C. Use firewall rules
*   D. Use Cloud Armor

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Organization Policy with gcp.resourceLocations)**
*   **Explanation**: Organization Policies can restrict resource locations across the entire organization.
*   *IAM* controls who can do what, not where.
</details>

---

### Question 15
**Scenario**: You need to run a legacy Windows application that requires specific Windows Server 2012 R2 features.
**Which compute option should you use?**

*   A. Cloud Run
*   B. GKE with Windows containers
*   C. Compute Engine with custom Windows image
*   D. App Engine Flexible

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Compute Engine with custom Windows image)**
*   **Explanation**: Compute Engine supports Windows VMs with full OS control.
*   *Cloud Run* and *GKE* are for containers.
*   *App Engine* doesn't support Windows.
</details>

---

### Question 16
**Scenario**: Your application needs to store user session data that expires after 1 hour. The data must be accessible with sub-millisecond latency.
**Which storage solution is best?**

*   A. Cloud SQL
*   B. Memorystore for Redis
*   C. Cloud Storage
*   D. Firestore

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Memorystore for Redis)**
*   **Explanation**: Redis is an in-memory cache perfect for session data with TTL support.
*   *Cloud SQL* has higher latency.
*   *Cloud Storage* is for objects, not key-value pairs.
</details>

---

### Question 17
**Scenario**: You need to analyze petabytes of historical sales data using SQL queries. The data is rarely updated.
**Which service should you use?**

*   A. Cloud SQL
*   B. Cloud Spanner
*   C. BigQuery
*   D. Bigtable

<details>
<summary>Click to reveal answer</summary>

**Answer: C (BigQuery)**
*   **Explanation**: BigQuery is a serverless data warehouse designed for analytics on massive datasets.
*   *Cloud SQL* can't handle petabytes.
*   *Bigtable* is NoSQL, not SQL.
</details>

---

### Question 18
**Scenario**: Your microservices architecture has 20 services that need to communicate securely within the same VPC. You want to avoid managing firewall rules for each service.
**What should you use?**

*   A. Firewall rules with IP ranges
*   B. Firewall rules with service accounts as targets
*   C. VPC Peering
*   D. Cloud Armor

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Firewall rules with service accounts)**
*   **Explanation**: Service account-based firewall rules are more maintainable than IP-based rules.
*   *VPC Peering* is for connecting different VPCs.
</details>

---

### Question 19
**Scenario**: You need to deploy an application that requires GPU acceleration for machine learning inference.
**Which compute option provides GPU support?**

*   A. Cloud Run
*   B. App Engine Standard
*   C. Compute Engine with GPU
*   D. Cloud Functions

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Compute Engine with GPU)**
*   **Explanation**: Compute Engine and GKE support GPUs. Cloud Run, App Engine Standard, and Cloud Functions do not.
</details>

---

### Question 20
**Scenario**: Your application needs to send notifications to mobile devices when new data is available. You want a fully managed pub/sub messaging service.
**Which service should you use?**

*   A. Cloud Tasks
*   B. Pub/Sub
*   C. Cloud Scheduler
*   D. Eventarc

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Pub/Sub)**
*   **Explanation**: Pub/Sub is designed for asynchronous messaging and event distribution.
*   *Cloud Tasks* is for task queues.
*   *Cloud Scheduler* is for cron jobs.
</details>

---

### Question 21
**Scenario**: You need to create a development environment that mirrors your production VPC network without affecting production resources.
**What should you do?**

*   A. Create a new VPC and use VPC Peering
*   B. Create a new VPC with the same CIDR ranges
*   C. Use Shared VPC
*   D. Create subnets in the production VPC

<details>
<summary>Click to reveal answer</summary>

**Answer: A (Create a new VPC and use VPC Peering)**
*   **Explanation**: VPC Peering allows communication between VPCs while keeping them isolated.
*   *Same CIDR ranges* would cause routing conflicts if peered.
</details>

---

### Question 22
**Scenario**: Your company requires all VM boot disks to be encrypted with your own encryption keys stored in Cloud KMS.
**How should you configure this?**

*   A. Enable default encryption
*   B. Use CMEK for boot disks
*   C. Use CSEK for boot disks
*   D. Enable Shielded VMs

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Use CMEK for boot disks)**
*   **Explanation**: CMEK allows you to use your own keys from Cloud KMS for disk encryption.
*   *CSEK* requires supplying keys with every API call.
*   *Shielded VMs* is for boot integrity, not encryption.
</details>

---

### Question 23
**Scenario**: You need to run a containerized application that requires access to the Kubernetes API and custom CRDs (Custom Resource Definitions).
**Which service should you use?**

*   A. Cloud Run
*   B. GKE Standard
*   C. App Engine Flexible
*   D. Compute Engine with Docker

<details>
<summary>Click to reveal answer</summary>

**Answer: B (GKE Standard)**
*   **Explanation**: GKE provides full Kubernetes API access and supports CRDs.
*   *Cloud Run* is serverless and doesn't expose the Kubernetes API.
</details>

---

### Question 24
**Scenario**: Your application needs to store and serve 4K video files to users worldwide with the lowest latency.
**What should you do?**

*   A. Store in Cloud Storage and use Cloud CDN
*   B. Store in Cloud SQL
*   C. Store in Bigtable
*   D. Store in Filestore

<details>
<summary>Click to reveal answer</summary>

**Answer: A (Cloud Storage + Cloud CDN)**
*   **Explanation**: Cloud Storage is designed for object storage, and Cloud CDN caches content globally for low latency.
*   *Cloud SQL* is not for large binary files.
</details>

---

### Question 25
**Scenario**: You need to ensure that your Compute Engine VMs can only be accessed via SSH from your corporate network (IP range 203.0.113.0/24).
**What should you configure?**

*   A. IAM policy
*   B. Firewall rule allowing SSH from 203.0.113.0/24
*   C. Cloud Armor security policy
*   D. VPC Service Controls

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Firewall rule allowing SSH from 203.0.113.0/24)**
*   **Explanation**: Firewall rules control network traffic based on IP ranges.
*   *IAM* controls who can do what, not network access.
</details>

---

### Question 26
**Scenario**: Your application needs to process images uploaded to Cloud Storage by resizing them and storing the results.
**Which serverless option is most appropriate?**

*   A. Cloud Functions triggered by Cloud Storage events
*   B. Cloud Run with Pub/Sub
*   C. App Engine Cron
*   D. Cloud Scheduler

<details>
<summary>Click to reveal answer</summary>

**Answer: A (Cloud Functions triggered by Cloud Storage events)**
*   **Explanation**: Cloud Functions can be directly triggered by Cloud Storage object creation events.
*   *Cloud Run* requires Pub/Sub or HTTP triggers.
</details>

---

### Question 27
**Scenario**: You need to migrate a PostgreSQL database from AWS RDS to Cloud SQL with minimal downtime.
**What should you use?**

*   A. pg_dump and pg_restore
*   B. Database Migration Service
*   C. Transfer Appliance
*   D. Cloud Interconnect

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Database Migration Service)**
*   **Explanation**: DMS supports continuous replication from external databases including AWS RDS.
*   *pg_dump* requires downtime.
</details>

---

### Question 28
**Scenario**: Your application needs to store structured data with complex queries, transactions, and joins. Data size is 500 GB.
**Which database should you use?**

*   A. Firestore
*   B. Bigtable
*   C. Cloud SQL
*   D. Cloud Storage

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Cloud SQL)**
*   **Explanation**: Cloud SQL is a relational database that supports complex queries, transactions, and joins.
*   *Firestore* and *Bigtable* are NoSQL.
</details>

---

### Question 29
**Scenario**: You need to ensure that traffic between your VMs and Cloud Storage never leaves Google's network.
**What should you enable?**

*   A. Cloud NAT
*   B. Private Google Access
*   C. VPC Peering
*   D. Cloud Interconnect

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Private Google Access)**
*   **Explanation**: Private Google Access allows VMs to reach Google APIs via Google's internal network.
</details>

---

### Question 30
**Scenario**: Your company wants to run Apache Spark jobs on-demand without managing infrastructure.
**Which service should you use?**

*   A. Dataproc
*   B. Dataflow
*   C. GKE
*   D. Compute Engine

<details>
<summary>Click to reveal answer</summary>

**Answer: A (Dataproc)**
*   **Explanation**: Dataproc is a managed Spark and Hadoop service.
*   *Dataflow* is for Apache Beam pipelines.
</details>

---

### Question 31
**Scenario**: You need to deploy a web application that automatically scales based on CPU usage and serves traffic across multiple zones.
**What should you use?**

*   A. Single Compute Engine VM
*   B. Regional Managed Instance Group with autoscaling
*   C. Cloud Run
*   D. App Engine Standard

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Regional MIG with autoscaling)**
*   **Explanation**: Regional MIGs distribute VMs across zones and support autoscaling.
*   *Cloud Run* and *App Engine* are also valid but the question implies VM-based deployment.
</details>

---

### Question 32
**Scenario**: Your application needs to store time-series metrics from 1 million devices, with queries by device ID and time range.
**Which database is most suitable?**

*   A. Cloud SQL
*   B. Bigtable
*   C. Firestore
*   D. BigQuery

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Bigtable)**
*   **Explanation**: Bigtable is optimized for time-series data with high write throughput.
*   *BigQuery* is for analytics, not real-time ingestion.
</details>

---

### Question 33
**Scenario**: You need to connect two VPCs in different projects within the same organization.
**What should you use?**

*   A. VPC Peering
*   B. Shared VPC
*   C. Cloud VPN
*   D. Cloud Interconnect

<details>
<summary>Click to reveal answer</summary>

**Answer: A (VPC Peering)**
*   **Explanation**: VPC Peering connects VPCs across projects.
*   *Shared VPC* is for centralized network management within one host project.
</details>

---

### Question 34
**Scenario**: Your application needs to execute code in response to HTTP requests with automatic scaling and no server management.
**Which service is most appropriate?**

*   A. Compute Engine
*   B. GKE
*   C. Cloud Functions (2nd gen)
*   D. Cloud Run

<details>
<summary>Click to reveal answer</summary>

**Answer: D (Cloud Run)**
*   **Explanation**: Cloud Run is serverless, scales automatically, and handles HTTP requests.
*   *Cloud Functions 2nd gen* is also valid but Cloud Run is more flexible for HTTP services.
</details>

---

### Question 35
**Scenario**: You need to store application logs for 10 years for compliance. Logs will never be accessed unless required by auditors.
**What is the most cost-effective solution?**

*   A. Keep logs in Cloud Logging
*   B. Export to Cloud Storage Archive class
*   C. Export to BigQuery
*   D. Export to Bigtable

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Cloud Storage Archive class)**
*   **Explanation**: Archive class is the cheapest for long-term retention of rarely accessed data.
*   *Cloud Logging* has retention limits.
</details>

---

### Question 36
**Scenario**: Your application needs to perform full-text search on millions of documents.
**Which service should you use?**

*   A. Cloud SQL
*   B. Firestore
*   C. Elasticsearch on GKE
*   D. BigQuery

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Elasticsearch on GKE)**
*   **Explanation**: Elasticsearch is designed for full-text search. GCP doesn't have a native managed search service.
*   *BigQuery* supports some text search but isn't optimized for it.
</details>

---

### Question 37
**Scenario**: You need to ensure that all API calls to your Cloud Storage bucket are logged for security auditing.
**What should you enable?**

*   A. Cloud Monitoring
*   B. Data Access audit logs
*   C. Admin Activity audit logs
*   D. VPC Flow Logs

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Data Access audit logs)**
*   **Explanation**: Data Access logs record who read/wrote data in Cloud Storage.
*   *Admin Activity* logs only record configuration changes.
</details>

---

### Question 38
**Scenario**: Your company requires that all VM instances must have Secure Boot enabled to prevent rootkit attacks.
**What should you configure?**

*   A. Enable Shielded VM with Secure Boot
*   B. Enable OS Login
*   C. Use CMEK encryption
*   D. Enable VPC Service Controls

<details>
<summary>Click to reveal answer</summary>

**Answer: A (Shielded VM with Secure Boot)**
*   **Explanation**: Shielded VMs provide Secure Boot, vTPM, and integrity monitoring.
*   *OS Login* is for SSH key management.
</details>

---

### Question 39
**Scenario**: You need to deploy a stateful application that requires persistent storage and can tolerate zone failures but not region failures.
**What should you use?**

*   A. Zonal persistent disk
*   B. Regional persistent disk
*   C. Local SSD
*   D. Cloud Storage

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Regional persistent disk)**
*   **Explanation**: Regional persistent disks replicate data across two zones in the same region.
*   *Zonal* disks are lost if the zone fails.
*   *Local SSD* is ephemeral.
</details>

---

### Question 40
**Scenario**: Your application needs to call an external API that requires authentication. You want to avoid storing credentials in your code.
**What should you use?**

*   A. Store credentials in Cloud Storage
*   B. Use Secret Manager
*   C. Store credentials in environment variables
*   D. Hardcode credentials

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Secret Manager)**
*   **Explanation**: Secret Manager is designed to securely store and access secrets like API keys.
*   *Environment variables* can be exposed in logs.
</details>

---

### Question 41
**Scenario**: You need to run a batch job every day at 2 AM UTC.
**Which service should you use?**

*   A. Cloud Functions
*   B. Cloud Scheduler
*   C. Pub/Sub
*   D. Cloud Tasks

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Cloud Scheduler)**
*   **Explanation**: Cloud Scheduler is a cron service for scheduling jobs.
*   *Cloud Tasks* is for asynchronous task queues.
</details>

---

### Question 42
**Scenario**: Your application needs to process messages from a queue with guaranteed delivery and at-least-once processing.
**Which service should you use?**

*   A. Pub/Sub
*   B. Cloud Tasks
*   C. Memorystore
*   D. Cloud Storage

<details>
<summary>Click to reveal answer</summary>

**Answer: A (Pub/Sub)**
*   **Explanation**: Pub/Sub guarantees at-least-once delivery of messages.
*   *Cloud Tasks* is for task queues, not pub/sub messaging.
</details>

---

### Question 43
**Scenario**: You need to deploy a machine learning model for real-time inference with auto-scaling.
**Which service is most appropriate?**

*   A. Vertex AI Prediction
*   B. BigQuery ML
*   C. Dataflow
*   D. Dataproc

<details>
<summary>Click to reveal answer</summary>

**Answer: A (Vertex AI Prediction)**
*   **Explanation**: Vertex AI Prediction provides managed model serving with auto-scaling.
*   *BigQuery ML* is for training and batch prediction.
</details>

---

### Question 44
**Scenario**: Your application needs to store user-uploaded files with public read access but private write access.
**How should you configure Cloud Storage?**

*   A. Make bucket public, use IAM for write access
*   B. Use signed URLs for all access
*   C. Use allUsers for read, IAM for write
*   D. Use bucket-level ACLs

<details>
<summary>Click to reveal answer</summary>

**Answer: C (allUsers for read, IAM for write)**
*   **Explanation**: Grant `allUsers` the Storage Object Viewer role for public read, and use IAM for write permissions.
</details>

---

### Question 45
**Scenario**: You need to ensure that your GKE cluster nodes do not have external IP addresses.
**What should you configure?**

*   A. Private GKE cluster
*   B. VPC-native cluster
*   C. Autopilot cluster
*   D. Zonal cluster

<details>
<summary>Click to reveal answer</summary>

**Answer: A (Private GKE cluster)**
*   **Explanation**: Private clusters have nodes without external IPs.
*   *VPC-native* refers to IP aliasing, not private nodes.
</details>

---

### Question 46
**Scenario**: Your application needs to store shopping cart data that must be accessible across multiple sessions and devices.
**Which storage solution is best?**

*   A. Memorystore (ephemeral)
*   B. Firestore
*   C. Local browser storage
*   D. Compute Engine local SSD

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Firestore)**
*   **Explanation**: Firestore is a NoSQL database that syncs across devices and sessions.
*   *Memorystore* is for caching, not persistent storage.
</details>

---

### Question 47
**Scenario**: You need to analyze real-time clickstream data from your website and detect anomalies.
**Which combination of services should you use?**

*   A. Pub/Sub → Dataflow → BigQuery
*   B. Cloud Storage → BigQuery
*   C. Pub/Sub → Cloud Functions → Cloud SQL
*   D. Dataproc → Bigtable

<details>
<summary>Click to reveal answer</summary>

**Answer: A (Pub/Sub → Dataflow → BigQuery)**
*   **Explanation**: Pub/Sub ingests streams, Dataflow processes in real-time, BigQuery stores for analysis.
</details>

---

### Question 48
**Scenario**: Your company wants to enforce that all Cloud Storage buckets must have versioning enabled.
**How should you implement this?**

*   A. Use IAM policies
*   B. Use Organization Policy
*   C. Manually enable on each bucket
*   D. Use Cloud Functions to check

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Organization Policy)**
*   **Explanation**: Organization Policies can enforce constraints like requiring versioning.
</details>

---

### Question 49
**Scenario**: You need to migrate 50 TB of data from AWS S3 to Cloud Storage.
**What is the most efficient method?**

*   A. Use `gsutil` with `rsync`
*   B. Use Storage Transfer Service
*   C. Use Transfer Appliance
*   D. Download and re-upload manually

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Storage Transfer Service)**
*   **Explanation**: Storage Transfer Service is designed for cloud-to-cloud transfers and is more efficient than `gsutil` for large datasets.
</details>

---

### Question 50
**Scenario**: Your application needs to send emails to users when certain events occur.
**Which service should you use?**

*   A. Cloud Functions with SendGrid API
*   B. Pub/Sub
*   C. Cloud Tasks
*   D. Cloud Scheduler

<details>
<summary>Click to reveal answer</summary>

**Answer: A (Cloud Functions with SendGrid API)**
*   **Explanation**: GCP doesn't have a native email service. Use Cloud Functions to call a third-party email API like SendGrid.
</details>

---

### Question 51
**Scenario**: You need to ensure high availability for your application across multiple regions with automatic failover.
**What should you configure?**

*   A. Regional Managed Instance Group
*   B. Global HTTP(S) Load Balancer with backends in multiple regions
*   C. Cloud DNS with health checks
*   D. VPC Peering

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Global HTTP(S) Load Balancer with multi-region backends)**
*   **Explanation**: Global load balancer automatically routes traffic to healthy backends in different regions.
</details>

---

### Question 52
**Scenario**: Your application needs to store and query hierarchical data (e.g., organizational charts).
**Which database is most suitable?**

*   A. Cloud SQL
*   B. Firestore
*   C. Bigtable
*   D. Cloud Spanner

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Firestore)**
*   **Explanation**: Firestore is a document database that naturally supports hierarchical data structures.
</details>

---

### Question 53
**Scenario**: You need to provide temporary access to a Cloud Storage object without requiring the user to have a Google account.
**What should you use?**

*   A. Make the bucket public
*   B. Generate a signed URL
*   C. Use IAM policy
*   D. Use VPC Service Controls

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Signed URL)**
*   **Explanation**: Signed URLs provide time-limited access to specific objects without authentication.
</details>

---

### Question 54
**Scenario**: Your application needs to process large CSV files uploaded to Cloud Storage by transforming and loading them into BigQuery.
**Which service should you use?**

*   A. Cloud Functions
*   B. Dataflow
*   C. Dataproc
*   D. BigQuery Load Job

<details>
<summary>Click to reveal answer</summary>

**Answer: D (BigQuery Load Job)**
*   **Explanation**: BigQuery can directly load CSV files from Cloud Storage.
*   *Dataflow* is for complex transformations.
</details>

---

### Question 55
**Scenario**: You need to ensure that your Compute Engine VMs can communicate with each other using internal DNS names.
**What should you configure?**

*   A. Cloud DNS private zone
*   B. Hosts file on each VM
*   C. VPC internal DNS (automatic)
*   D. External DNS provider

<details>
<summary>Click to reveal answer</summary>

**Answer: C (VPC internal DNS - automatic)**
*   **Explanation**: VPC automatically provides internal DNS resolution for VM instances.
*   *Cloud DNS private zones* are for custom domains.
</details>

---

**Congratulations!** You've completed 55 practice questions for Section 1. Review the explanations and continue to the next sections.
