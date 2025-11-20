---
sidebar_position: 3
---

# 2. Managing and Provisioning a Solution Infrastructure

This section focuses on the practical implementation and configuration of the resources you designed in Section 1.

## 2.1 Configuring Network Topologies

### VPC Configuration
*   **Auto Mode**: Automatically creates a subnet in every region. Good for PoCs, bad for production (IP overlap risk).
*   **Custom Mode**: You explicitly create subnets. **Recommended for production**.
*   **IP Addressing**: Use **RFC 1918** private ranges (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16).
*   **Alias IPs**: Assign secondary IP ranges to VMs (useful for containers/Pods).

### Hybrid Connectivity
*   **Cloud VPN (HA)**:
    *   Uses **BGP** (Border Gateway Protocol) for dynamic routing.
    *   SLA: 99.99% (if configured with 2 tunnels).
    *   Max bandwidth: 3 Gbps per tunnel.
*   **Cloud Interconnect**:
    *   **Dedicated**: Physical cable to Google. 10 Gbps or 100 Gbps.
    *   **Partner**: Connect via a service provider. 50 Mbps to 10 Gbps.
    *   **Direct Peering**: Connects to Google Edge (not GCP private network directly, usually for Workspace).

### Load Balancing
| Type | Traffic | Scope | Proxy/Pass-through | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **External HTTP(S)** | HTTP/HTTPS | Global | Proxy | Web apps, content delivery, DDoS protection (Cloud Armor). |
| **Internal HTTP(S)** | HTTP/HTTPS | Regional | Proxy | Internal microservices. |
| **External TCP/UDP** | TCP/UDP | Regional | Pass-through | Gaming, non-HTTP protocols. |
| **Internal TCP/UDP** | TCP/UDP | Regional | Pass-through | Internal databases, legacy apps. |

:::tip Exam Tip
If you need to distribute traffic across **multiple regions** (Global), you MUST use **HTTP(S)**, **SSL Proxy**, or **TCP Proxy** Load Balancing. The "Network" load balancers are Regional only.
:::

---

## 2.2 Configuring Individual Storage Systems

### Cloud Storage (GCS)
*   **Versioning**: Protects against accidental overwrites/deletes.
*   **Lifecycle Management**: Automate moving objects to cheaper classes (e.g., "Move to Coldline after 30 days").
*   **Retention Policies**: "Locked" Bucket Policy prevents deletion for a set time (Compliance).
*   **Access Control**:
    *   **Uniform**: Applies to the whole bucket (Recommended).
    *   **Fine-grained (ACLs)**: Applies to individual objects (Legacy).

### Cloud SQL
*   **High Availability**: Creates a standby instance in a different zone. Failover is automatic.
*   **Read Replicas**: Offload read traffic from the primary instance. **Not** for DR (unless promoted).
*   **Backups**: Automated (daily) and On-demand. Point-in-time recovery (PITR) enabled by binary logs.

---

## 2.3 Configuring Compute Systems

### Compute Engine (GCE)
*   **Preemptible VMs**: Last max 24 hours. Can be terminated with 30s warning.
*   **Shielded VMs**: Verify boot integrity (Secure Boot, vTPM).
*   **Startup Scripts**: Run commands when the VM boots (install software, configure OS).
*   **Metadata Server**: `http://metadata.google.internal/computeMetadata/v1/`. Used to get VM info or service account tokens.

### Google Kubernetes Engine (GKE)
*   **Cluster Types**:
    *   **Zonal**: Single zone. Low availability.
    *   **Regional**: Control plane and nodes replicated across 3 zones. High availability.
    *   **Private Cluster**: Nodes have no public IPs. Access control plane via private endpoint or authorized networks.
*   **Autoscaling**:
    *   **HPA (Horizontal Pod Autoscaler)**: Adds more Pods based on CPU/RAM.
    *   **Cluster Autoscaler**: Adds more Nodes when Pods are pending.

### Cloud Run
*   **Concurrency**: Handle multiple requests per container instance (unlike AWS Lambda which is 1:1).
*   **Revisions**: Immutable snapshots of your code/config. Allows traffic splitting (Canary).

---

## 📝 Practice Questions

### Question 1
**Scenario**: You are deploying a web application that must be accessible globally. You want to route users to the closest region and provide SSL termination.
**Which Load Balancer should you choose?**

*   A. External TCP/UDP Network Load Balancer
*   B. Internal HTTP(S) Load Balancer
*   C. Global External HTTP(S) Load Balancer
*   D. SSL Proxy Load Balancer

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Global External HTTP(S) Load Balancer)**
*   **Explanation**: HTTP(S) LB is global, handles Layer 7 traffic (Web), and terminates SSL.
*   *SSL Proxy* is for non-HTTP SSL traffic.
</details>

### Question 2
**Scenario**: You have a regulatory requirement to ensure that objects in a Cloud Storage bucket cannot be deleted or overwritten for 5 years.
**What should you configure?**

*   A. Enable Object Versioning.
*   B. Configure a Lifecycle Management rule.
*   C. Set a Retention Policy and Lock it.
*   D. Remove the `storage.objects.delete` permission from all users.

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Set a Retention Policy and Lock it)**
*   **Explanation**: A Locked Retention Policy enforces WORM (Write Once, Read Many) compliance. Even administrators cannot delete the objects until the retention period expires.
</details>
