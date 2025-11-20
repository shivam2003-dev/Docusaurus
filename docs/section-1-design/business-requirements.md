---
sidebar_position: 2
---

# 1.1 Business Requirements

As a Cloud Architect, you must translate business goals into technical implementations. This chapter covers how to design solutions that meet business needs.

## Understanding Business Use Cases

### Common Business Scenarios

#### 1. Microservices Architecture
**Business Goal**: Increase development velocity and enable independent team deployments.

**Technical Solution**:
- Decouple monolithic applications into smaller services
- Use **Google Kubernetes Engine (GKE)** for container orchestration
- Use **Cloud Run** for serverless containers
- Implement **API Gateway** for unified API management

**Benefits**:
- Faster time to market
- Independent scaling of services
- Technology diversity (different languages per service)
- Fault isolation

#### 2. Data Analytics & Business Intelligence
**Business Goal**: Make data-driven decisions with real-time insights.

**Technical Solution**:
- Use **BigQuery** for data warehousing and analytics
- Use **Dataflow** for stream processing
- Use **Dataproc** for Spark/Hadoop workloads
- Use **Looker** or **Data Studio** for visualization

**Benefits**:
- Petabyte-scale analytics
- Serverless (no infrastructure management)
- Pay only for queries run

#### 3. Disaster Recovery & Business Continuity
**Business Goal**: Ensure business operations continue during outages.

**Technical Solution**:
- Multi-region storage with **Cloud Storage**
- **Global Load Balancing** for automatic failover
- **Cloud SQL** with HA configuration
- Regular backups and tested recovery procedures

**Benefits**:
- Minimize downtime (RTO)
- Minimize data loss (RPO)
- Compliance with SLAs

---

## Cost Optimization

### CAPEX to OPEX Transformation

**Capital Expenditure (CAPEX)**:
- Upfront hardware purchases
- Depreciation over years
- Over-provisioning for peak capacity
- Maintenance costs

**Operational Expenditure (OPEX)**:
- Pay-as-you-go pricing
- Scale up/down as needed
- No upfront costs
- Predictable monthly billing

### Cost Optimization Strategies

#### 1. Committed Use Discounts (CUDs)
**What**: Commit to using resources for 1 or 3 years.
**Savings**: Up to 57% for 3-year commitment.
**Best For**: Predictable, steady-state workloads.

```
Example:
- On-demand: $0.10/hour
- 1-year CUD: $0.07/hour (30% savings)
- 3-year CUD: $0.043/hour (57% savings)
```

#### 2. Preemptible / Spot VMs
**What**: Short-lived VMs that can be terminated by Google with 30 seconds notice.
**Savings**: Up to 80% off regular price.
**Best For**: Fault-tolerant batch processing, rendering, data analysis.

**Limitations**:
- Maximum 24-hour runtime
- No SLA
- Can be preempted at any time

:::warning When NOT to Use Preemptible VMs
- Production web servers
- Databases
- Applications that can't handle interruptions
:::

#### 3. Storage Class Selection

| Class | Access Frequency | Minimum Storage | Retrieval Cost | Use Case |
|-------|-----------------|-----------------|----------------|----------|
| **Standard** | Frequent | None | None | Hot data, websites |
| **Nearline** | Once/month | 30 days | Yes | Backups, disaster recovery |
| **Coldline** | Once/quarter | 90 days | Yes | Archival, compliance |
| **Archive** | Once/year | 365 days | Yes | Long-term retention |

:::tip Exam Tip
If a question mentions "minimize cost" and "rarely accessed" → Think **Coldline** or **Archive** storage class.
:::

#### 4. Sustained Use Discounts (Automatic)
**What**: Automatic discounts for running VMs for a significant portion of the month.
**Savings**: Up to 30% for VMs running the entire month.
**Best Part**: No commitment required, applied automatically.

---

## Integration with External Systems

### Hybrid Cloud Connectivity

#### Cloud VPN
**Use Case**: Quick, cost-effective connection to on-premises.

**Features**:
- Encrypted over public internet (IPsec)
- Up to 3 Gbps per tunnel
- **HA VPN**: 99.99% SLA with 2 tunnels
- Dynamic routing with **BGP**

**Pricing**: ~$0.05/hour per tunnel + egress charges.

#### Cloud Interconnect
**Use Case**: High-bandwidth, low-latency connection to on-premises.

**Types**:

1. **Dedicated Interconnect**:
   - Physical cable to Google
   - 10 Gbps or 100 Gbps
   - Requires colocation in supported facility
   - 99.9% or 99.99% SLA

2. **Partner Interconnect**:
   - Connect via service provider
   - 50 Mbps to 10 Gbps
   - No colocation required
   - 99.9% or 99.99% SLA

**Decision Matrix**:
```
Need < 3 Gbps? → Cloud VPN
Need 10+ Gbps? → Dedicated Interconnect
Need 50 Mbps - 10 Gbps without colocation? → Partner Interconnect
```

:::warning Egress Costs
Ingress (data coming INTO GCP) is usually **free**.
Egress (data going OUT of GCP) costs money.

Plan your architecture to minimize cross-region and internet egress.
:::

---

## Data Movement Strategies

### Online Transfer

#### 1. gsutil (Command-line tool)
**Best For**: Small to medium datasets (< 1 TB).
**Features**:
- Parallel uploads
- Resumable transfers
- Compression

```bash
# Upload a directory
gsutil -m cp -r /local/path gs://bucket-name/

# Sync directories
gsutil -m rsync -r /local/path gs://bucket-name/
```

#### 2. Storage Transfer Service
**Best For**: Large cloud-to-cloud or on-premises-to-cloud transfers.
**Features**:
- Scheduled transfers
- Bandwidth throttling
- Filtering by file age/name
- Transfer from AWS S3, Azure Blob, HTTP/HTTPS

**Sources Supported**:
- AWS S3
- Azure Blob Storage
- HTTP/HTTPS endpoints
- On-premises (via agent)

### Offline Transfer

#### Transfer Appliance
**Best For**: Petabyte-scale data where network transfer would take too long.

**How It Works**:
1. Google ships you a physical appliance (up to 1 PB capacity)
2. You copy data to the appliance
3. Ship it back to Google
4. Google uploads data to Cloud Storage

**When to Use**:
- Data size > 20 TB
- Limited bandwidth
- Network transfer would take > 1 week

**Calculation**:
```
Transfer time = Data size / Bandwidth

Example:
100 TB / 100 Mbps = 100 days
100 TB via Transfer Appliance = ~1 week (including shipping)
```

---

## Trade-offs and Decision Making

### CAP Theorem
In distributed systems, you can only guarantee 2 out of 3:
- **C**onsistency: All nodes see the same data
- **A**vailability: System remains operational
- **P**artition Tolerance: System works despite network failures

**GCP Services**:
- **Cloud Spanner**: CP (Consistency + Partition Tolerance) with high availability
- **Firestore**: AP (Availability + Partition Tolerance) with eventual consistency
- **Cloud SQL**: CA (Consistency + Availability) within a region

### Performance vs. Cost
**High Performance** (Expensive):
- SSD persistent disks
- High-memory machine types
- Cloud Spanner for global databases

**Cost-Optimized** (Lower Performance):
- HDD persistent disks
- Shared-core machine types (e2-micro)
- Cloud SQL for regional databases

:::tip Exam Strategy
Always look for the **most cost-effective** solution that **meets the requirements**. Don't over-engineer.
:::

---

## Summary

**Key Takeaways**:
1. Understand the business problem before designing the solution
2. Use CUDs for predictable workloads, Preemptible VMs for batch jobs
3. Choose the right storage class based on access frequency
4. Use Cloud VPN for < 3 Gbps, Interconnect for higher bandwidth
5. Use Transfer Appliance for petabyte-scale offline transfers
6. Balance performance, cost, and availability based on requirements

**Next Chapter**: [1.2 Technical Requirements →](./technical-requirements)
