---
sidebar_position: 2
---

# 3.1 IAM and Access Control

Master Identity and Access Management, resource hierarchy, and access control best practices for Google Cloud.

## IAM Fundamentals

### IAM Policy Structure

**Who** + **Can Do What** + **On Which Resource**

```
Member (Who): user:alice@example.com
Role (Can Do What): roles/storage.objectViewer
Resource (On Which): gs://my-bucket
```

### Member Types

| Type | Format | Example |
|------|--------|---------|
| **Google Account** | user:email | user:alice@example.com |
| **Service Account** | serviceAccount:email | serviceAccount:sa@project.iam.gserviceaccount.com |
| **Google Group** | group:email | group:devs@example.com |
| **Domain** | domain:domain | domain:example.com |
| **All Users** | allUsers | allUsers (public access) |
| **All Authenticated** | allAuthenticatedUsers | allAuthenticatedUsers |

### Role Types

#### Primitive Roles (Avoid in Production)
```
roles/owner    → Full access (dangerous!)
roles/editor   → Modify resources
roles/viewer   → Read-only access
```

:::warning
Primitive roles grant access across ALL services. Use predefined or custom roles instead.
:::

#### Predefined Roles (Recommended)
```
roles/compute.admin           → Full Compute Engine access
roles/storage.objectViewer    → Read Cloud Storage objects
roles/bigquery.dataEditor     → Edit BigQuery data
roles/iam.serviceAccountUser  → Use service accounts
```

#### Custom Roles
```bash
# Create custom role
gcloud iam roles create customRole \
  --project=PROJECT_ID \
  --title="Custom Role" \
  --description="Specific permissions" \
  --permissions=compute.instances.get,compute.instances.list \
  --stage=GA
```

---

## Resource Hierarchy

### Hierarchy Structure

```
Organization (example.com)
├── Folder: Production
│   ├── Project: prod-web
│   ├── Project: prod-api
│   └── Project: prod-data
├── Folder: Development
│   ├── Project: dev-web
│   └── Project: dev-api
└── Folder: Shared Services
    └── Project: shared-vpc
```

### Policy Inheritance

**Policies flow DOWN the hierarchy**:
```
Organization Policy
    ↓ (inherited)
Folder Policy
    ↓ (inherited)
Project Policy
    ↓ (inherited)
Resource Policy
```

**Example**:
```bash
# Grant at organization level (applies to ALL projects)
gcloud organizations add-iam-policy-binding ORG_ID \
  --member=group:admins@example.com \
  --role=roles/viewer

# Grant at folder level
gcloud resource-manager folders add-iam-policy-binding FOLDER_ID \
  --member=group:devs@example.com \
  --role=roles/editor

# Grant at project level
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:alice@example.com \
  --role=roles/compute.admin
```

---

## Service Accounts

### Types of Service Accounts

#### User-Managed Service Accounts
```bash
# Create service account
gcloud iam service-accounts create my-sa \
  --display-name="My Service Account" \
  --description="For application authentication"

# Grant roles to service account
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=serviceAccount:my-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/storage.objectViewer
```

#### Default Service Accounts
```
Compute Engine: PROJECT_NUMBER-compute@developer.gserviceaccount.com
App Engine: PROJECT_ID@appspot.gserviceaccount.com
```

:::warning
Default service accounts have Editor role by default. Create custom service accounts with minimal permissions.
:::

### Service Account Keys

**Best Practice**: Avoid keys when possible!

**Alternatives**:
1. **Workload Identity** (GKE)
2. **Service Account Impersonation**
3. **Metadata Server** (Compute Engine)

**If keys are necessary**:
```bash
# Create key (avoid!)
gcloud iam service-accounts keys create key.json \
  --iam-account=my-sa@PROJECT_ID.iam.gserviceaccount.com

# List keys
gcloud iam service-accounts keys list \
  --iam-account=my-sa@PROJECT_ID.iam.gserviceaccount.com

# Delete key
gcloud iam service-accounts keys delete KEY_ID \
  --iam-account=my-sa@PROJECT_ID.iam.gserviceaccount.com
```

### Service Account Impersonation

```bash
# Grant impersonation permission
gcloud iam service-accounts add-iam-policy-binding \
  target-sa@PROJECT_ID.iam.gserviceaccount.com \
  --member=user:alice@example.com \
  --role=roles/iam.serviceAccountTokenCreator

# Use impersonation
gcloud compute instances list \
  --impersonate-service-account=target-sa@PROJECT_ID.iam.gserviceaccount.com
```

---

## IAM Best Practices

### Principle of Least Privilege

**Bad**:
```bash
# Granting Owner role (too broad!)
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:dev@example.com \
  --role=roles/owner
```

**Good**:
```bash
# Grant specific role needed
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:dev@example.com \
  --role=roles/compute.instanceAdmin.v1
```

### Use Groups

**Bad** (Managing individual users):
```bash
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:alice@example.com \
  --role=roles/viewer

gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:bob@example.com \
  --role=roles/viewer
```

**Good** (Using groups):
```bash
# Create group: viewers@example.com with Alice and Bob
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=group:viewers@example.com \
  --role=roles/viewer
```

### Separate Environments

```
Folder: Production
  - Strict IAM policies
  - Minimal access
  - Audit logging enabled

Folder: Development
  - Relaxed policies
  - Developer access
  - Separate billing
```

---

## IAM Conditions

### Time-Based Access

```bash
# Grant access only during business hours
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:contractor@example.com \
  --role=roles/compute.viewer \
  --condition='expression=request.time < timestamp("2024-12-31T00:00:00Z"),
               title=Temporary Access,
               description=Access until end of year'
```

### Resource-Based Conditions

```bash
# Grant access only to specific resources
gcloud projects add-iam-policy-binding PROJECT_ID \
  --member=user:dev@example.com \
  --role=roles/compute.instanceAdmin \
  --condition='expression=resource.name.startsWith("projects/PROJECT_ID/zones/us-central1-a/instances/dev-"),
               title=Dev Instances Only'
```

---

## Organization Policies

### Common Policies

**Restrict VM External IPs**:
```bash
# Deny external IPs on VMs
gcloud resource-manager org-policies set-policy \
  --organization=ORG_ID \
  policy.yaml
```

```yaml
# policy.yaml
constraint: compute.vmExternalIpAccess
listPolicy:
  deniedValues:
  - "*"
```

**Restrict Resource Locations**:
```yaml
constraint: gcp.resourceLocations
listPolicy:
  allowedValues:
  - in:us-locations
  - in:eu-locations
```

**Require OS Login**:
```yaml
constraint: compute.requireOsLogin
booleanPolicy:
  enforced: true
```

---

## Access Transparency and Audit Logs

### Cloud Audit Logs

**Types**:
1. **Admin Activity** (Always on, free)
   - Who created/deleted/modified resources
   
2. **Data Access** (Must enable, costs money)
   - Who read/wrote data
   
3. **System Events** (Always on, free)
   - Google system actions

**Enable Data Access Logs**:
```bash
# Get current policy
gcloud projects get-iam-policy PROJECT_ID > policy.yaml

# Edit policy.yaml to add:
auditConfigs:
- auditLogConfigs:
  - logType: DATA_READ
  - logType: DATA_WRITE
  service: storage.googleapis.com

# Set policy
gcloud projects set-iam-policy PROJECT_ID policy.yaml
```

### Access Transparency

**What**: Logs when Google Support accesses your data

**Enable**:
```bash
gcloud organizations add-iam-policy-binding ORG_ID \
  --member=user:admin@example.com \
  --role=roles/accessapproval.approver
```

---

## IAM Troubleshooting

### Policy Troubleshooter

```bash
# Check why user has/doesn't have access
gcloud policy-troubleshoot iam \
  //cloudresourcemanager.googleapis.com/projects/PROJECT_ID \
  --permission=compute.instances.create \
  --principal-email=user@example.com
```

### Common Issues

**Issue**: User can't access resource
**Check**:
1. IAM policy at resource level
2. IAM policy at project level
3. IAM policy at folder/org level
4. Organization policies (may deny access)
5. VPC Service Controls (may block access)

---

## Summary

**Key Takeaways**:
1. Use **predefined or custom roles**, avoid primitive roles
2. Grant permissions at the **lowest level** possible
3. Use **groups** instead of individual users
4. **Avoid service account keys**, use Workload Identity
5. Enable **audit logs** for compliance
6. Use **IAM conditions** for temporary/conditional access
7. Implement **organization policies** for governance
8. Use **Policy Troubleshooter** for debugging
