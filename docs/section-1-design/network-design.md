---
sidebar_position: 6
---

# 1.5 Network Design

Master VPC networking, hybrid connectivity patterns, and network security design for Google Cloud.

## VPC Network Architecture

### VPC Fundamentals

**Key Concepts**:
- VPC is a **global resource**
- Subnets are **regional resources**
- Firewall rules are **global** (apply to entire VPC)
- Routes are **global**

### Subnet Design Best Practices

#### IP Address Planning

**RFC 1918 Private Ranges**:
```
10.0.0.0/8     → 16,777,216 addresses
172.16.0.0/12  → 1,048,576 addresses
192.168.0.0/16 → 65,536 addresses
```

**Subnet Sizing Strategy**:
```
/24 = 256 addresses (251 usable)
  - Small environments, single service
  
/20 = 4,096 addresses
  - Medium environments, multiple services
  
/16 = 65,536 addresses
  - Large environments, GKE clusters
```

**Example Design**:
```
VPC: production-vpc (10.0.0.0/8)
├── us-central1
│   ├── web-subnet: 10.1.0.0/24
│   ├── app-subnet: 10.1.1.0/24
│   └── data-subnet: 10.1.2.0/24
├── europe-west1
│   ├── web-subnet: 10.2.0.0/24
│   ├── app-subnet: 10.2.1.0/24
│   └── data-subnet: 10.2.2.0/24
```

### VPC Peering vs Shared VPC

#### VPC Peering
**Use Case**: Connect VPCs across projects (decentralized)

**Characteristics**:
- Transitive peering NOT supported
- Each side manages their own network
- No overlapping IP ranges

```bash
# Create peering from VPC-A to VPC-B
gcloud compute networks peerings create peer-a-to-b \
  --network=vpc-a \
  --peer-project=project-b \
  --peer-network=vpc-b
```

#### Shared VPC
**Use Case**: Centralized network management

**Architecture**:
```
Host Project (owns VPC)
├── Service Project 1 (uses VPC)
├── Service Project 2 (uses VPC)
└── Service Project 3 (uses VPC)
```

**Benefits**:
- Centralized firewall management
- Shared IP address space
- Simplified billing

---

## Hybrid Connectivity Patterns

### Pattern 1: VPN for Basic Connectivity

**Scenario**: Small office needs secure connection to GCP

**Solution**: HA VPN with BGP

```
On-Premises (192.168.0.0/16)
    ↓ (VPN Tunnel)
GCP VPC (10.0.0.0/8)
```

**Configuration**:
```bash
# Create Cloud Router
gcloud compute routers create vpn-router \
  --network=my-vpc \
  --region=us-central1 \
  --asn=65001

# Create HA VPN Gateway
gcloud compute vpn-gateways create ha-vpn-gw \
  --network=my-vpc \
  --region=us-central1

# Create tunnels (2 for HA)
gcloud compute vpn-tunnels create tunnel-1 \
  --peer-external-gateway=on-prem-gw \
  --peer-external-gateway-interface=0 \
  --region=us-central1 \
  --ike-version=2 \
  --shared-secret=SECRET \
  --router=vpn-router \
  --vpn-gateway=ha-vpn-gw \
  --interface=0
```

### Pattern 2: Interconnect for High Bandwidth

**Scenario**: Data center needs 10+ Gbps connection

**Solution**: Dedicated Interconnect

```
Data Center
    ↓ (10 Gbps Physical Connection)
Google Edge Location
    ↓ (VLAN Attachment)
GCP VPC
```

**When to Use**:
- Bandwidth > 3 Gbps
- Consistent low latency required
- Cost-effective for high data transfer

### Pattern 3: Multi-Cloud Connectivity

**Scenario**: Connect AWS and GCP

**Solution**: VPN between cloud providers

```
AWS VPC (172.16.0.0/16)
    ↓ (VPN)
GCP VPC (10.0.0.0/8)
```

---

## Load Balancing Architecture

### Global Load Balancing

**Use Case**: Serve users worldwide with lowest latency

**Architecture**:
```
User (Asia) ────┐
User (Europe) ──┼──→ Global LB (Anycast IP)
User (US) ──────┘         ├──→ Backend (us-central1)
                          ├──→ Backend (europe-west1)
                          └──→ Backend (asia-east1)
```

**Configuration**:
```bash
# Create health check
gcloud compute health-checks create http global-health \
  --port=80 \
  --request-path=/health

# Create backend service
gcloud compute backend-services create global-backend \
  --protocol=HTTP \
  --health-checks=global-health \
  --global

# Add backends from multiple regions
gcloud compute backend-services add-backend global-backend \
  --instance-group=us-mig \
  --instance-group-region=us-central1 \
  --global

gcloud compute backend-services add-backend global-backend \
  --instance-group=eu-mig \
  --instance-group-region=europe-west1 \
  --global
```

### Internal Load Balancing

**Use Case**: Microservices communication

**Architecture**:
```
Frontend Service
    ↓ (Internal LB: 10.1.0.100)
Backend Service Pool
    ├── Backend-1 (10.1.1.10)
    ├── Backend-2 (10.1.1.11)
    └── Backend-3 (10.1.1.12)
```

---

## Network Security Design

### Defense in Depth Strategy

**Layers**:
1. **Perimeter**: Cloud Armor, VPC Service Controls
2. **Network**: Firewall rules, Private Google Access
3. **Application**: Identity-Aware Proxy
4. **Data**: Encryption in transit

### Firewall Rule Hierarchy

**Priority Order** (0 = highest, 65535 = lowest):
```
Priority 1000: Allow SSH from office (203.0.113.0/24)
Priority 2000: Allow HTTP/HTTPS from anywhere
Priority 3000: Allow internal communication (10.0.0.0/8)
Priority 65534: Deny all (implicit)
```

**Best Practices**:
```bash
# Use service accounts (not tags)
gcloud compute firewall-rules create allow-web \
  --network=my-vpc \
  --allow=tcp:80,tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-service-accounts=web-sa@project.iam.gserviceaccount.com

# Enable logging for security analysis
gcloud compute firewall-rules create allow-ssh \
  --network=my-vpc \
  --allow=tcp:22 \
  --source-ranges=203.0.113.0/24 \
  --enable-logging
```

### Private Google Access

**Use Case**: VMs without external IPs need to access Google APIs

**Configuration**:
```bash
# Enable on subnet
gcloud compute networks subnets update my-subnet \
  --region=us-central1 \
  --enable-private-ip-google-access
```

**Flow**:
```
VM (no external IP: 10.1.0.5)
    ↓ (Private Google Access)
Google APIs (Cloud Storage, BigQuery, etc.)
```

### Cloud NAT

**Use Case**: VMs without external IPs need internet access

**Configuration**:
```bash
# Create Cloud Router
gcloud compute routers create nat-router \
  --network=my-vpc \
  --region=us-central1

# Create NAT configuration
gcloud compute routers nats create my-nat \
  --router=nat-router \
  --region=us-central1 \
  --nat-all-subnet-ip-ranges \
  --auto-allocate-nat-external-ips
```

---

## DNS Architecture

### Private DNS Zones

**Use Case**: Internal service discovery

**Example**:
```
Service: database.internal.company.com → 10.1.2.5
Service: cache.internal.company.com → 10.1.2.6
```

**Configuration**:
```bash
# Create private zone
gcloud dns managed-zones create internal-zone \
  --dns-name=internal.company.com. \
  --visibility=private \
  --networks=my-vpc

# Add A record
gcloud dns record-sets create db.internal.company.com. \
  --zone=internal-zone \
  --type=A \
  --ttl=300 \
  --rrdatas=10.1.2.5
```

### DNS Forwarding

**Use Case**: Resolve on-premises DNS from GCP

**Architecture**:
```
GCP VM queries "server.onprem.local"
    ↓
Cloud DNS (forwarding policy)
    ↓
On-Premises DNS (192.168.1.10)
```

**Configuration**:
```bash
# Create forwarding policy
gcloud dns policies create forward-to-onprem \
  --networks=my-vpc \
  --alternative-name-servers=192.168.1.10,192.168.1.11 \
  --private-alternative-name-servers
```

---

## Network Monitoring

### VPC Flow Logs

**Use Case**: Network troubleshooting, security analysis

**Configuration**:
```bash
# Enable on subnet
gcloud compute networks subnets update my-subnet \
  --region=us-central1 \
  --enable-flow-logs \
  --logging-aggregation-interval=interval-5-sec \
  --logging-flow-sampling=0.5
```

**Analysis**:
```sql
-- BigQuery query for top talkers
SELECT
  jsonPayload.connection.src_ip,
  jsonPayload.connection.dest_ip,
  SUM(CAST(jsonPayload.bytes_sent AS INT64)) as total_bytes
FROM `project.dataset.vpc_flows`
WHERE DATE(timestamp) = CURRENT_DATE()
GROUP BY 1, 2
ORDER BY total_bytes DESC
LIMIT 10;
```

### Connectivity Tests

**Use Case**: Verify network paths before deployment

```bash
# Test connectivity
gcloud network-management connectivity-tests create test-web \
  --source-instance=vm-1 \
  --destination-ip-address=10.1.0.5 \
  --protocol=TCP \
  --destination-port=80
```

---

## Network Design Patterns

### Pattern 1: Three-Tier Architecture

```
Internet
    ↓
External Load Balancer
    ↓
Web Tier (DMZ Subnet: 10.1.0.0/24)
    ↓
Internal Load Balancer
    ↓
App Tier (App Subnet: 10.1.1.0/24)
    ↓
Data Tier (Data Subnet: 10.1.2.0/24)
```

### Pattern 2: Hub-and-Spoke

```
Hub VPC (Shared Services)
    ├── VPC Peering → Spoke 1 (Dev)
    ├── VPC Peering → Spoke 2 (Staging)
    └── VPC Peering → Spoke 3 (Prod)
```

### Pattern 3: Multi-Region Active-Active

```
Global Load Balancer
    ├──→ Region 1 (us-central1)
    │     ├── Web Tier
    │     ├── App Tier
    │     └── Data Tier (Cloud Spanner)
    │
    └──→ Region 2 (europe-west1)
          ├── Web Tier
          ├── App Tier
          └── Data Tier (Cloud Spanner)
```

---

## Summary

**Key Takeaways**:
1. Plan IP address space carefully (avoid overlaps)
2. Use **Shared VPC** for centralized management
3. **HA VPN** for < 3 Gbps, **Interconnect** for higher
4. Use **Global Load Balancer** for worldwide traffic
5. Implement **defense in depth** with multiple security layers
6. Enable **VPC Flow Logs** for troubleshooting
7. Use **Private Google Access** for VMs without external IPs
