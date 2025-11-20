---
sidebar_position: 3
---

# 1.2 Technical Requirements

This chapter focuses on designing solutions that meet technical requirements like high availability, scalability, reliability, and performance.

## High Availability (HA) vs. Disaster Recovery (DR)

### High Availability (HA)
**Definition**: The system remains operational during component failures.

**Failure Scenarios**:
- Single VM crashes
- Zone becomes unavailable
- Network partition

**Solutions**:
- Deploy across **multiple Zones** within a Region
- Use **Regional Managed Instance Groups (MIGs)**
- Implement **health checks** and auto-healing
- Use **Regional Persistent Disks**

**Example Architecture**:
```
Region: us-central1
├── Zone: us-central1-a (2 VMs)
├── Zone: us-central1-b (2 VMs)
└── Zone: us-central1-c (2 VMs)

Load Balancer distributes traffic across all zones.
If one zone fails, traffic routes to remaining zones.
```

### Disaster Recovery (DR)
**Definition**: Recovering from catastrophic failures (entire region down).

**Failure Scenarios**:
- Entire region becomes unavailable
- Data center disaster
- Major network outage

**Solutions**:
- Replicate data to **different Regions**
- Use **Global Load Balancing**
- Implement **Cross-region replication** for databases
- Regular backup and restore testing

**DR Patterns**:

| Pattern | Description | RTO | RPO | Cost |
|---------|-------------|-----|-----|------|
| **Backup & Restore** | Regular backups to Cloud Storage | Hours-Days | Hours | Low |
| **Pilot Light** | Minimal resources running, scale up on failover | Minutes-Hours | Minutes | Medium |
| **Warm Standby** | Scaled-down version running | Minutes | Minutes | Medium-High |
| **Hot Standby (Active-Active)** | Full capacity in multiple regions | Seconds | Near-zero | High |

:::tip Exam Tip
**RTO (Recovery Time Objective)**: How long can you be down?
**RPO (Recovery Point Objective)**: How much data can you lose?

Lower RTO/RPO = Higher Cost
:::

---

## Scalability

### Vertical Scaling (Scale Up)
**Definition**: Adding more CPU/RAM to a single VM.

**Pros**:
- Simple to implement
- No application changes needed

**Cons**:
- Limited by maximum machine size
- Requires downtime (stop VM, resize, start)
- Single point of failure

**When to Use**:
- Legacy applications that can't scale horizontally
- Databases (though horizontal is preferred)

### Horizontal Scaling (Scale Out)
**Definition**: Adding more VMs to a group.

**Pros**:
- Unlimited scale
- No downtime
- Fault tolerance (multiple VMs)

**Cons**:
- Application must be stateless or use external state
- More complex architecture

**Implementation**:
- **Managed Instance Groups (MIGs)** with autoscaling
- **GKE** with Horizontal Pod Autoscaler
- **Cloud Run** (automatic)

**Autoscaling Policies**:
```yaml
# Example: Scale based on CPU
- Target CPU: 70%
- Min instances: 2
- Max instances: 10
- Cool-down period: 60 seconds

When CPU > 70% for 60s → Add instances
When CPU < 70% for 60s → Remove instances
```

:::warning Cool-down Period
Always set a cool-down period to prevent "flapping" (rapid scale up/down).
:::

---

## Reliability

### Defining Reliability Metrics

#### 1. Availability (Uptime)
**Formula**: `Availability = Uptime / (Uptime + Downtime)`

**SLA Examples**:
| SLA | Downtime/Year | Downtime/Month | Downtime/Week |
|-----|---------------|----------------|---------------|
| 99% | 3.65 days | 7.2 hours | 1.68 hours |
| 99.9% | 8.76 hours | 43.2 minutes | 10.1 minutes |
| 99.95% | 4.38 hours | 21.6 minutes | 5.04 minutes |
| 99.99% | 52.6 minutes | 4.32 minutes | 1.01 minutes |
| 99.999% | 5.26 minutes | 25.9 seconds | 6.05 seconds |

**How to Achieve High Availability**:
- Eliminate single points of failure
- Use managed services (Google handles infrastructure)
- Implement health checks
- Use load balancing

#### 2. Durability (Data Loss)
**Definition**: Probability that data will NOT be lost.

**GCP Storage Durability**:
- **Cloud Storage**: 99.999999999% (11 nines)
- **Persistent Disks**: 99.9999999% (9 nines) with snapshots
- **Regional Persistent Disks**: Replicated across 2 zones

**Best Practices**:
- Enable versioning on Cloud Storage
- Regular automated backups
- Test restore procedures
- Use multi-region storage for critical data

---

## Performance Optimization

### Compute Performance

#### Machine Types

**General Purpose** (E2, N2, N2D):
- Balanced CPU/RAM
- Best for web servers, app servers
- Cost-effective

**Compute-Optimized** (C2, C2D):
- High CPU-to-RAM ratio
- Best for compute-intensive workloads (gaming, scientific computing)
- Higher cost per core

**Memory-Optimized** (M2, M3):
- High RAM-to-CPU ratio
- Best for in-memory databases (SAP HANA, Redis)
- Most expensive

**Accelerator-Optimized** (A2, A3):
- GPU/TPU attached
- Best for ML training, inference, rendering
- Specialized pricing

:::tip Choosing Machine Types
Start with **E2** (cheapest) and benchmark.
Only upgrade to specialized types if you have proven bottlenecks.
:::

### Storage Performance

#### Persistent Disk Types

| Type | IOPS (Read/Write) | Throughput | Use Case | Cost |
|------|-------------------|------------|----------|------|
| **Standard (HDD)** | Low | 120-180 MB/s | Bulk storage, backups | Lowest |
| **Balanced (SSD)** | Medium | 240 MB/s | General purpose | Medium |
| **SSD** | High | 1,200 MB/s | Databases, high I/O | Higher |
| **Extreme (SSD)** | Very High | 2,400 MB/s | Mission-critical databases | Highest |

**Performance Scaling**:
- IOPS and throughput scale with disk size
- Larger disks = better performance (up to limits)

**Local SSD**:
- Physically attached to the VM
- **Highest performance** (up to 2.4 million IOPS)
- **Ephemeral** (data lost when VM stops)
- Use for temporary data, caches

### Network Performance

#### Network Tiers

**Premium Tier** (Default):
- Traffic routes through Google's global network
- Lower latency
- Better performance
- Higher cost

**Standard Tier**:
- Traffic routes through public internet
- Higher latency
- Lower performance
- Lower cost

:::tip Exam Tip
For global applications serving users worldwide → **Premium Tier**
For regional applications or cost-sensitive workloads → **Standard Tier**
:::

---

## Health Checks and Auto-Healing

### Health Check Types

#### 1. HTTP(S) Health Check
```yaml
Check: GET /health
Interval: 10 seconds
Timeout: 5 seconds
Healthy threshold: 2 consecutive successes
Unhealthy threshold: 3 consecutive failures
```

**When to Use**: Web applications with health endpoints.

#### 2. TCP Health Check
```yaml
Check: TCP connection on port 3306
Interval: 10 seconds
Timeout: 5 seconds
```

**When to Use**: Databases, non-HTTP services.

### Auto-Healing

**How It Works**:
1. Health check fails for an instance
2. Instance marked as unhealthy
3. MIG automatically deletes the unhealthy instance
4. MIG creates a new instance to maintain desired count

**Configuration**:
```bash
gcloud compute instance-groups managed set-autohealing my-mig \
  --health-check=my-health-check \
  --initial-delay=300
```

:::warning Initial Delay
Set `initial-delay` to allow time for application startup.
If too short, instances may be killed during boot.
:::

---

## Designing for Failure

### Chaos Engineering Principles

**Philosophy**: Proactively inject failures to test system resilience.

**Common Experiments**:
1. **Kill random VMs** → Does the system recover?
2. **Introduce network latency** → Do timeouts work correctly?
3. **Simulate zone failure** → Does traffic route to other zones?
4. **Fill up disk** → Do alerts fire? Does app handle gracefully?

**Tools**:
- **Chaos Monkey** (Netflix OSS)
- **Gremlin**
- Custom scripts

### Graceful Degradation

**Concept**: When a dependency fails, the system continues with reduced functionality.

**Example**:
```
E-commerce site:
- Recommendation engine fails → Show generic products instead
- Payment gateway slow → Queue orders for later processing
- Image CDN down → Show placeholder images
```

**Implementation**:
- Circuit breakers
- Fallback mechanisms
- Timeouts and retries

---

## Summary

**Key Takeaways**:
1. **HA** = Multi-zone deployment, **DR** = Multi-region replication
2. Horizontal scaling is preferred over vertical scaling
3. Choose machine types based on workload (start with E2)
4. Use SSD for high I/O, HDD for bulk storage
5. Implement health checks and auto-healing for reliability
6. Design for failure with graceful degradation

