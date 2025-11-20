---
sidebar_position: 2
---

# 2.1 Network Configuration

Master VPC networking, hybrid connectivity, load balancing, and DNS configuration.

## VPC (Virtual Private Cloud)

### VPC Modes

#### Auto Mode VPC
**Characteristics**:
- Automatically creates one subnet in **every region**
- Predefined IP ranges (10.128.0.0/9)
- New regions get subnets automatically

**Pros**:
- Quick to set up
- Good for demos/PoCs

**Cons**:
- No control over IP ranges
- Risk of IP overlap with on-premises networks
- **Not recommended for production**

#### Custom Mode VPC
**Characteristics**:
- You explicitly create subnets
- You choose IP ranges (RFC 1918)
- Full control over network design

**Pros**:
- Production-ready
- No IP conflicts
- Flexible design

**Cons**:
- Requires planning

:::tip Exam Tip
Always choose **Custom Mode VPC** for production environments.
:::

### Creating a Custom VPC

```bash
# Create VPC (global resource)
gcloud compute networks create my-vpc \
  --subnet-mode=custom \
  --bgp-routing-mode=regional

# Create subnet in us-central1 (regional resource)
gcloud compute networks subnets create my-subnet-us \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.1.0/24

# Create subnet in europe-west1
gcloud compute networks subnets create my-subnet-eu \
  --network=my-vpc \
  --region=europe-west1 \
  --range=10.0.2.0/24
```

### IP Addressing Best Practices

**RFC 1918 Private Ranges**:
- `10.0.0.0/8` (16.7 million addresses)
- `172.16.0.0/12` (1 million addresses)
- `192.168.0.0/16` (65,536 addresses)

**Subnet Sizing**:
```
/24 = 256 addresses (251 usable, 5 reserved by GCP)
/20 = 4,096 addresses
/16 = 65,536 addresses
```

**Reserved IPs per Subnet**:
- Network address (.0)
- Gateway (.1)
- Second-to-last (.254 in /24)
- Broadcast (.255 in /24)

### Alias IP Ranges (Secondary Ranges)

**Use Case**: Assign multiple IP addresses to a single VM (for containers/Pods).

```bash
# Create subnet with secondary range for GKE Pods
gcloud compute networks subnets create gke-subnet \
  --network=my-vpc \
  --region=us-central1 \
  --range=10.0.10.0/24 \
  --secondary-range pods=10.1.0.0/16,services=10.2.0.0/20
```

**Why Use Alias IPs**:
- Native GKE networking (VPC-native clusters)
- Better performance (no NAT)
- Firewall rules can target Pod IPs directly

---

## Hybrid Connectivity

### Cloud VPN

#### HA VPN (High Availability VPN)
**Architecture**:
- 2 VPN gateways (99.99% SLA)
- 2 tunnels (active-active)
- BGP for dynamic routing

**Setup**:
```bash
# 1. Create HA VPN gateway
gcloud compute vpn-gateways create my-ha-vpn \
  --network=my-vpc \
  --region=us-central1

# 2. Create external VPN gateway (on-prem)
gcloud compute external-vpn-gateways create on-prem-gateway \
  --interfaces 0=203.0.113.1,1=203.0.113.2

# 3. Create Cloud Router for BGP
gcloud compute routers create my-router \
  --region=us-central1 \
  --network=my-vpc \
  --asn=65001

# 4. Create VPN tunnels
gcloud compute vpn-tunnels create tunnel-1 \
  --peer-external-gateway=on-prem-gateway \
  --peer-external-gateway-interface=0 \
  --region=us-central1 \
  --ike-version=2 \
  --shared-secret=SECRET_KEY \
  --router=my-router \
  --vpn-gateway=my-ha-vpn \
  --interface=0
```

**Performance**:
- Max 3 Gbps per tunnel
- Can create multiple tunnels for higher throughput

**Pricing**:
- ~$0.05/hour per tunnel
- Egress charges apply

### Cloud Interconnect

#### Dedicated Interconnect
**Use Case**: High-bandwidth, low-latency connection to on-premises.

**Requirements**:
- Colocation in supported facility
- Physical cross-connect to Google

**Capacity**:
- 10 Gbps or 100 Gbps per connection
- Can bundle multiple connections

**SLA**:
- 99.9% (single connection)
- 99.99% (dual connections)

**Setup Process**:
1. Request Dedicated Interconnect in Console
2. Google provides LOA-CFA (Letter of Authorization)
3. Colocation provider creates cross-connect
4. Create VLAN attachments
5. Configure BGP

#### Partner Interconnect
**Use Case**: Need Interconnect but can't collocate.

**How It Works**:
- Connect to a service provider
- Provider has existing connection to Google

**Capacity**:
- 50 Mbps to 10 Gbps
- Flexible bandwidth

**Pricing**:
- Pay service provider + Google attachment fee

### Decision Matrix: VPN vs Interconnect

| Requirement | Solution |
|-------------|----------|
| < 3 Gbps, cost-sensitive | **Cloud VPN** |
| 10-100 Gbps, can collocate | **Dedicated Interconnect** |
| 50 Mbps - 10 Gbps, no colocation | **Partner Interconnect** |
| Encrypted over internet | **Cloud VPN** |
| Private connection, no internet | **Interconnect** |

---

## Load Balancing

### Load Balancer Types

#### External HTTP(S) Load Balancer
**Scope**: Global
**Use Case**: Web applications, APIs, content delivery

**Features**:
- Anycast IP (single global IP)
- SSL termination
- URL-based routing
- Cloud CDN integration
- Cloud Armor (DDoS protection)

**Configuration**:
```bash
# 1. Create instance group
gcloud compute instance-groups managed create web-mig \
  --base-instance-name=web \
  --size=3 \
  --template=web-template \
  --region=us-central1

# 2. Create health check
gcloud compute health-checks create http web-health-check \
  --port=80 \
  --request-path=/health

# 3. Create backend service
gcloud compute backend-services create web-backend \
  --protocol=HTTP \
  --health-checks=web-health-check \
  --global

# 4. Add instance group to backend
gcloud compute backend-services add-backend web-backend \
  --instance-group=web-mig \
  --instance-group-region=us-central1 \
  --global

# 5. Create URL map
gcloud compute url-maps create web-map \
  --default-service=web-backend

# 6. Create target HTTP proxy
gcloud compute target-http-proxies create web-proxy \
  --url-map=web-map

# 7. Create forwarding rule (global IP)
gcloud compute forwarding-rules create web-forwarding-rule \
  --global \
  --target-http-proxy=web-proxy \
  --ports=80
```

#### Internal HTTP(S) Load Balancer
**Scope**: Regional
**Use Case**: Internal microservices

**Features**:
- Private IP only
- Layer 7 routing
- No internet exposure

#### Network Load Balancer (TCP/UDP)
**Scope**: Regional
**Use Case**: Non-HTTP protocols, gaming, streaming

**Features**:
- Pass-through (no proxy)
- Preserves source IP
- Lower latency

### Load Balancer Comparison

| Feature | External HTTP(S) | Internal HTTP(S) | Network (TCP/UDP) |
|---------|------------------|------------------|-------------------|
| **Scope** | Global | Regional | Regional |
| **Layer** | 7 (Application) | 7 (Application) | 4 (Transport) |
| **Protocol** | HTTP/HTTPS | HTTP/HTTPS | TCP/UDP |
| **SSL Termination** | Yes | Yes | No (pass-through) |
| **URL Routing** | Yes | Yes | No |
| **Cloud CDN** | Yes | No | No |
| **Cloud Armor** | Yes | No | No |

:::tip Exam Tip
**Global** traffic distribution = HTTP(S), SSL Proxy, or TCP Proxy Load Balancer
**Regional** only = Network Load Balancer or Internal Load Balancers
:::

---

## Cloud DNS

### DNS Zone Types

#### Public Zone
**Use Case**: Resolve domain names from the internet.

```bash
# Create public zone
gcloud dns managed-zones create my-zone \
  --dns-name=example.com. \
  --description="Public zone for example.com"

# Add A record
gcloud dns record-sets create www.example.com. \
  --zone=my-zone \
  --type=A \
  --ttl=300 \
  --rrdatas=203.0.113.1
```

#### Private Zone
**Use Case**: Internal DNS resolution within VPC.

```bash
# Create private zone
gcloud dns managed-zones create internal-zone \
  --dns-name=internal.example.com. \
  --description="Private zone" \
  --visibility=private \
  --networks=my-vpc

# Add A record
gcloud dns record-sets create db.internal.example.com. \
  --zone=internal-zone \
  --type=A \
  --ttl=300 \
  --rrdatas=10.0.1.5
```

### DNS Policies

**Server Policy**: Forward DNS queries to on-premises DNS servers.

```bash
gcloud dns policies create forward-to-onprem \
  --networks=my-vpc \
  --enable-inbound-forwarding \
  --alternative-name-servers=192.168.1.1,192.168.1.2
```

---

## Firewall Rules

### Firewall Rule Structure

**Components**:
- **Direction**: Ingress (incoming) or Egress (outgoing)
- **Priority**: 0-65535 (lower = higher priority)
- **Action**: Allow or Deny
- **Target**: All instances, tags, or service accounts
- **Source/Destination**: IP ranges, tags, or service accounts
- **Protocol/Port**: TCP, UDP, ICMP, etc.

### Common Firewall Rules

```bash
# Allow SSH from specific IP
gcloud compute firewall-rules create allow-ssh \
  --network=my-vpc \
  --allow=tcp:22 \
  --source-ranges=203.0.113.0/24 \
  --description="Allow SSH from office"

# Allow HTTP/HTTPS from anywhere
gcloud compute firewall-rules create allow-web \
  --network=my-vpc \
  --allow=tcp:80,tcp:443 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=web-server

# Allow internal communication
gcloud compute firewall-rules create allow-internal \
  --network=my-vpc \
  --allow=tcp:0-65535,udp:0-65535,icmp \
  --source-ranges=10.0.0.0/8

# Deny all egress (explicit)
gcloud compute firewall-rules create deny-all-egress \
  --network=my-vpc \
  --direction=EGRESS \
  --action=DENY \
  --rules=all \
  --destination-ranges=0.0.0.0/0 \
  --priority=65535
```

### Firewall Best Practices

1. **Use service accounts** instead of tags when possible
2. **Least privilege**: Only open necessary ports
3. **Use priority** to override default rules
4. **Log firewall hits** for security auditing
5. **Test with Connectivity Tests** before deploying

---

## Shared VPC

### Architecture

**Host Project**: Owns the VPC network
**Service Projects**: Use the network from host project

**Use Case**: Centralized network administration.

```bash
# 1. Enable Shared VPC on host project
gcloud compute shared-vpc enable HOST_PROJECT_ID

# 2. Attach service project
gcloud compute shared-vpc associated-projects add SERVICE_PROJECT_ID \
  --host-project=HOST_PROJECT_ID

# 3. Grant IAM permissions
gcloud projects add-iam-policy-binding HOST_PROJECT_ID \
  --member=user:admin@example.com \
  --role=roles/compute.networkAdmin
```

**Benefits**:
- Centralized firewall management
- Shared IP address space
- Simplified billing

---

## Summary

**Key Takeaways**:
1. Use **Custom Mode VPC** for production
2. **HA VPN** for < 3 Gbps, **Interconnect** for higher bandwidth
3. **Global HTTP(S) LB** for global traffic, **Network LB** for regional TCP/UDP
4. Use **Private DNS zones** for internal resolution
5. **Shared VPC** for centralized network management

