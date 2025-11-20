---
sidebar_position: 4
---

# 3. Designing for Security and Compliance

Security is "Job Zero" in the cloud. This section covers how to secure your architecture and meet regulatory requirements.

## 3.1 Designing for Security

### Identity and Access Management (IAM)
*   **Who**: Google Account, Service Account, Google Group, Cloud Identity domain.
*   **What**: Roles (Primitive, Predefined, Custom).
*   **Which Resource**: Organization, Folder, Project, Resource.

#### Best Practices
*   **Least Privilege**: Grant only the necessary permissions.
*   **Use Groups**: Assign roles to Groups, not individual users.
*   **Service Accounts**:
    *   Use strictly for applications/machines.
    *   **Do not** check keys into source code. Use **Workload Identity Federation** for external workloads.
*   **Custom Roles**: Use when Predefined roles are too broad. (Note: You cannot add permissions to a Custom Role that are not supported for custom roles).

:::warning Warning
Avoid using **Primitive Roles** (Owner, Editor, Viewer) in production. They are too broad and apply across the entire project.
:::

### Resource Hierarchy
1.  **Organization**: Root node. Policies set here trickle down.
2.  **Folders**: Group projects by department (HR, Finance) or environment (Dev, Prod).
3.  **Projects**: Trust boundary. Billing and APIs are enabled here.
4.  **Resources**: VMs, Buckets, etc.

### Data Security
*   **Encryption at Rest**:
    *   **Default**: Google-managed keys (GMK). On by default.
    *   **CMEK (Customer-Managed Encryption Keys)**: You manage the key in Cloud KMS. Google uses it.
    *   **CSEK (Customer-Supplied Encryption Keys)**: You keep the key. Google never sees it. (Highest security, highest risk of data loss if key is lost).
*   **Encryption in Transit**: Google encrypts traffic between data centers automatically. Use TLS/SSL for client-to-server.

### Network Security
*   **Firewall Rules**: Control traffic to/from VMs.
*   **Cloud Armor**: WAF (Web Application Firewall) and DDoS protection at the edge. Protects HTTP(S) Load Balancers.
*   **VPC Service Controls**: Prevents data exfiltration. Creates a "perimeter" around managed services (like GCS, BigQuery) so data cannot be copied to unauthorized projects.
*   **Identity-Aware Proxy (IAP)**: Zero-trust access to VMs (SSH/RDP) and Web Apps without VPNs.

---

## 3.2 Designing for Compliance

### Regulatory Standards
*   **GDPR**: Data privacy for EU citizens.
*   **HIPAA**: Healthcare data (US). Requires a BAA (Business Associate Agreement).
*   **PCI-DSS**: Credit card data.

### Auditing & Logging
*   **Cloud Audit Logs**:
    *   **Admin Activity**: Who changed what? (Always on, free).
    *   **Data Access**: Who read what data? (Must be enabled, costs money).
    *   **System Event**: Google system actions.
*   **Access Transparency**: Logs when Google Support accesses your data (for troubleshooting).

### Data Sovereignty
*   **Resource Location**: You must choose the specific Regions (e.g., `europe-west3`) to store data to comply with local laws.
*   **Organization Policies**: Use the `gcp.resourceLocations` constraint to RESTRICT which regions developers can use.

---

## 📝 Practice Questions

### Question 1
**Scenario**: You have a team of developers who need access to a specific Cloud Storage bucket. You want to follow the principle of least privilege.
**What should you do?**

*   A. Grant the "Storage Admin" role to the team at the Project level.
*   B. Grant the "Storage Object Viewer" role to the team at the Bucket level.
*   C. Create a Service Account, grant it access, and give the keys to the developers.
*   D. Add the developers to the "Project Editor" role.

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Grant the "Storage Object Viewer" role to the team at the Bucket level)**
*   **Explanation**: This grants access ONLY to that specific bucket and ONLY for reading objects.
*   *A* grants admin access to ALL buckets in the project.
*   *D* is way too broad.
</details>

### Question 2
**Scenario**: Your company handles sensitive financial data. Security policy dictates that Google must NOT have the ability to decrypt your data without a key that you explicitly control and manage within GCP.
**Which encryption option should you use?**

*   A. Default Encryption
*   B. Customer-Managed Encryption Keys (CMEK)
*   C. Customer-Supplied Encryption Keys (CSEK)
*   D. Client-side encryption

<details>
<summary>Click to reveal answer</summary>

**Answer: B (CMEK)**
*   **Explanation**: CMEK allows you to manage the keys in Cloud KMS. Google uses the key to encrypt/decrypt, but you control the key lifecycle (rotate, disable, destroy).
*   *CSEK* means you hold the key *outside* of GCP (Google doesn't manage it at all), which fits the description but usually "manage within GCP" implies KMS. However, if the requirement is "Google must not have the ability to decrypt... without a key you control", CMEK fits best for "manage within GCP". If the requirement was "Google must never persist the key", it would be CSEK.
</details>
