---
sidebar_position: 2
---

# 1. Designing and Planning a Cloud Solution Architecture

This section covers the foundational skills required to design a robust cloud architecture that aligns with both business and technical requirements.

## 1.1 Designing a Solution Infrastructure that Meets Business Requirements

As a Cloud Architect, you must translate business goals into technical implementations.

### Key Considerations

#### 1. Business Use Cases
*   **Microservices**: Decoupling monolithic applications for agility. Use **GKE** or **Cloud Run**.
*   **Data Analytics**: Processing large datasets. Use **BigQuery**, **Dataflow**, **Dataproc**.
*   **Disaster Recovery**: Ensuring business continuity. Use **Multi-region Storage**, **Load Balancing**.

#### 2. Cost Optimization
*   **CAPEX to OPEX**: Moving from upfront hardware costs to pay-as-you-go.
*   **Preemptible / Spot VMs**: Use for fault-tolerant, batch processing workloads to save up to 80%.
*   **Committed Use Discounts (CUDs)**: Commit to 1 or 3 years for predictable workloads.
*   **Storage Classes**: Move infrequently accessed data to **Nearline**, **Coldline**, or **Archive**.

:::tip Exam Tip
Always look for the most **cost-effective** solution that meets the requirements. If a requirement says "minimize cost" and the workload is batch processing, think **Preemptible VMs**.
:::

#### 3. Integration with External Systems
*   **Hybrid Cloud**: Connecting on-premises to GCP.
    *   **Cloud VPN**: Low cost, quick setup, encrypted over public internet.
    *   **Cloud Interconnect**: High bandwidth, dedicated connection, SLA.
*   **Multi-Cloud**: Using Anthos (Google Kubernetes Engine Enterprise) to manage clusters across AWS, Azure, and GCP.

#### 4. Data Movement
*   **Online Transfer**: `gsutil`, Storage Transfer Service (for S3/Azure/HTTP).
*   **Offline Transfer**: **Transfer Appliance** (for petabytes of data where bandwidth is limited).

:::warning Warning
Ingress (data coming IN to GCP) is usually free. Egress (data going OUT) costs money. Keep this in mind when designing multi-cloud architectures.
:::

---

## 1.2 Designing a Solution Infrastructure that Meets Technical Requirements

### High Availability (HA) vs. Disaster Recovery (DR)
*   **High Availability**: The system remains operational during component failures (e.g., a zone going down).
    *   *Solution*: Deploy across multiple **Zones** within a Region. Use **Regional Managed Instance Groups (MIGs)**.
*   **Disaster Recovery**: Recovering from a catastrophic failure (e.g., a whole region going down).
    *   *Solution*: Replicate data to a different **Region**. Use **Global Load Balancing**.

### Scalability
*   **Vertical Scaling (Scale Up)**: Adding more CPU/RAM to a single VM. (Limited by max machine size, requires downtime).
*   **Horizontal Scaling (Scale Out)**: Adding more VMs to a group. (Unlimited scale, no downtime).
    *   *Tool*: **Managed Instance Groups (MIGs)** with Autoscaling.

### Reliability (RTO & RPO)
*   **RTO (Recovery Time Objective)**: How long can you be down?
*   **RPO (Recovery Point Objective)**: How much data can you lose?
*   *Low RTO/RPO* = Higher Cost (Hot Standby).
*   *High RTO/RPO* = Lower Cost (Backup & Restore).

---

## 1.3 Designing Network, Storage, and Compute Resources

### Compute Decision Tree

| Service | Type | Best For |
| :--- | :--- | :--- |
| **Compute Engine** | IaaS | Migrating legacy apps (Lift & Shift), custom kernels, full OS control. |
| **Google Kubernetes Engine (GKE)** | CaaS | Containerized microservices, hybrid/multi-cloud (Anthos). |
| **App Engine (Standard)** | PaaS | Web apps, scales to zero, specific languages (Python, Java, Go, etc.). |
| **App Engine (Flexible)** | PaaS | Docker containers, custom runtimes, long-running processes. |
| **Cloud Run** | Serverless | Stateless containers, event-driven, scales to zero. |
| **Cloud Functions** | Serverless | Single-purpose code, event triggers (Pub/Sub, Storage). |

:::note
**App Engine Standard** scales to zero instantly. **App Engine Flexible** always has at least one instance running (unless stopped manually), so it costs more for low traffic.
:::

### Storage Decision Tree

| Service | Type | SQL/NoSQL | Best For |
| :--- | :--- | :--- | :--- |
| **Cloud Storage (GCS)** | Object | N/A | Images, videos, backups, static websites. |
| **Cloud SQL** | Block/Relational | SQL | MySQL, PostgreSQL, SQL Server. Regional scale (up to ~64TB). |
| **Cloud Spanner** | Relational | SQL | **Global scale**, horizontal scaling, strong consistency. Expensive. |
| **Bigtable** | NoSQL (Wide-column) | NoSQL | **High throughput** (IoT, AdTech), flat data, ms latency. |
| **Firestore** | NoSQL (Document) | NoSQL | Mobile/Web apps, offline sync, hierarchical data. |
| **BigQuery** | Warehouse | SQL | **Analytics**, petabyte-scale, serverless. |

:::tip Exam Tip
If the question mentions "Global", "Horizontal Scaling", and "Relational/SQL" -> The answer is **Cloud Spanner**.
If the question mentions "IoT", "High Throughput", "Low Latency" -> The answer is **Bigtable**.
:::

### Network Design
*   **VPC (Virtual Private Cloud)**: Global resource. Subnets are Regional.
*   **Shared VPC**: Centralized network administration. Host project holds the network; Service projects use it.
*   **VPC Peering**: Connect two VPCs (decentralized).
*   **Firewall Rules**: Stateful. Allow/Deny traffic based on IP, Protocol, Ports, or Service Accounts/Tags.

---

## 1.4 Creating a Migration Plan

### The 4 Phases of Migration
1.  **Assess**: Discover existing resources (StratoZone).
2.  **Plan**: Choose the strategy.
3.  **Deploy**: Execute the migration.
4.  **Optimize**: Refactor for cloud-native benefits.

### Migration Strategies (The 6 Rs)
1.  **Rehost (Lift & Shift)**: Move VMs as-is to Compute Engine. (Fastest, least risk, least benefit).
2.  **Replatform (Move & Improve)**: Move to managed services (e.g., Self-hosted MySQL -> Cloud SQL).
3.  **Refactor (Re-architect)**: Rewrite code for microservices/serverless. (Slowest, most benefit).
4.  **Retire**: Turn off unused systems.
5.  **Retain**: Keep as-is on-prem.
6.  **Repurchase**: Switch to SaaS (e.g., Gmail).

---

## 1.5 Envisioning Future Solution Improvements

*   **CI/CD**: Move from manual deployment to automated pipelines (**Cloud Build**, **Artifact Registry**).
*   **Observability**: Implement **Cloud Operations Suite** (Logging, Monitoring, Trace) early.
*   **Security**: Shift security left. Automate policy enforcement with **Organization Policies**.

---

## 📝 Practice Questions

### Question 1
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

### Question 2
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
