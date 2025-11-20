---
sidebar_position: 3
---

# 3.2 Data Security and Encryption

Master encryption strategies, key management, and data protection techniques for Google Cloud.

## Encryption at Rest

### Default Encryption

**All data in GCP is encrypted at rest by default** using Google-managed keys.

**Services with automatic encryption**:
- Cloud Storage
- Compute Engine persistent disks
- Cloud SQL
- BigQuery
- Bigtable
- All other GCP storage services

### Encryption Options

| Option | Key Management | Use Case |
|--------|----------------|----------|
| **Google-managed** | Google manages | Default, easiest |
| **CMEK** (Customer-Managed) | You manage in Cloud KMS | Compliance, key rotation control |
| **CSEK** (Customer-Supplied) | You supply with each request | Maximum control, more complex |

---

## Cloud KMS (Key Management Service)

### Key Hierarchy

```
Key Ring (regional or global)
├── Crypto Key (purpose: encryption, signing, etc.)
│   ├── Crypto Key Version 1 (active)
│   ├── Crypto Key Version 2 (primary)
│   └── Crypto Key Version 3 (destroyed)
```

### Creating Keys

```bash
# Create key ring
gcloud kms keyrings create my-keyring \
  --location=us-central1

# Create encryption key
gcloud kms keys create my-key \
  --location=us-central1 \
  --keyring=my-keyring \
  --purpose=encryption

# Create signing key
gcloud kms keys create signing-key \
  --location=us-central1 \
  --keyring=my-keyring \
  --purpose=asymmetric-signing \
  --default-algorithm=rsa-sign-pkcs1-4096-sha512
```

### Key Rotation

**Automatic Rotation**:
```bash
# Enable automatic rotation (every 90 days)
gcloud kms keys update my-key \
  --location=us-central1 \
  --keyring=my-keyring \
  --rotation-period=90d \
  --next-rotation-time=2024-04-01T00:00:00Z
```

**Manual Rotation**:
```bash
# Create new version
gcloud kms keys versions create \
  --location=us-central1 \
  --keyring=my-keyring \
  --key=my-key \
  --primary
```

### Key Destruction

```bash
# Schedule destruction (24-hour minimum)
gcloud kms keys versions destroy VERSION_ID \
  --location=us-central1 \
  --keyring=my-keyring \
  --key=my-key

# Restore before destruction completes
gcloud kms keys versions restore VERSION_ID \
  --location=us-central1 \
  --keyring=my-keyring \
  --key=my-key
```

---

## CMEK (Customer-Managed Encryption Keys)

### Cloud Storage with CMEK

```bash
# Create bucket with CMEK
gsutil mb \
  -l us-central1 \
  -k projects/PROJECT_ID/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key \
  gs://my-cmek-bucket

# Upload object with CMEK
gsutil -o \
  "GSUtil:encryption_key=projects/PROJECT_ID/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key" \
  cp file.txt gs://my-bucket/
```

### Compute Engine with CMEK

```bash
# Create disk with CMEK
gcloud compute disks create my-disk \
  --size=100GB \
  --zone=us-central1-a \
  --kms-key=projects/PROJECT_ID/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key

# Create VM with CMEK boot disk
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --boot-disk-kms-key=projects/PROJECT_ID/locations/us-central1/keyRings/my-keyring/cryptoKeys/my-key
```

### BigQuery with CMEK

```bash
# Create dataset with CMEK
bq mk --dataset \
  --location=US \
  --default_kms_key=projects/PROJECT_ID/locations/us/keyRings/my-keyring/cryptoKeys/my-key \
  my_dataset
```

---

## CSEK (Customer-Supplied Encryption Keys)

### Cloud Storage with CSEK

```bash
# Generate encryption key
python3 -c 'import base64; import os; print(base64.b64encode(os.urandom(32)).decode())'

# Upload with CSEK
gsutil -o \
  "GSUtil:encryption_key=YOUR_BASE64_KEY" \
  cp file.txt gs://my-bucket/

# Download with CSEK
gsutil -o \
  "GSUtil:decryption_key1=YOUR_BASE64_KEY" \
  cp gs://my-bucket/file.txt .
```

### Compute Engine with CSEK

```json
// csek-key.json
{
  "key": "YOUR_BASE64_KEY",
  "raw-key": "YOUR_BASE64_KEY"
}
```

```bash
# Create disk with CSEK
gcloud compute disks create my-disk \
  --size=100GB \
  --zone=us-central1-a \
  --csek-key-file=csek-key.json
```

---

## Encryption in Transit

### TLS/SSL Configuration

**Load Balancer SSL Certificate**:
```bash
# Create SSL certificate
gcloud compute ssl-certificates create my-cert \
  --certificate=cert.pem \
  --private-key=key.pem \
  --global

# Use with HTTPS load balancer
gcloud compute target-https-proxies create my-proxy \
  --ssl-certificates=my-cert \
  --url-map=my-url-map
```

**Managed SSL Certificates**:
```bash
# Google manages certificate renewal
gcloud compute ssl-certificates create my-managed-cert \
  --domains=example.com,www.example.com \
  --global
```

### VPN Encryption

**Cloud VPN** automatically encrypts traffic using IPsec.

```bash
# Create VPN tunnel (encrypted by default)
gcloud compute vpn-tunnels create my-tunnel \
  --peer-address=203.0.113.1 \
  --shared-secret=SECRET \
  --ike-version=2 \
  --region=us-central1 \
  --target-vpn-gateway=my-gateway
```

---

## Secret Manager

### Storing Secrets

```bash
# Create secret
echo -n "my-database-password" | \
  gcloud secrets create db-password \
  --data-file=-

# Create secret with automatic replication
gcloud secrets create api-key \
  --replication-policy=automatic \
  --data-file=api-key.txt

# Create secret with specific locations
gcloud secrets create sensitive-data \
  --replication-policy=user-managed \
  --locations=us-central1,europe-west1 \
  --data-file=data.txt
```

### Accessing Secrets

```bash
# Add new version
echo -n "new-password" | \
  gcloud secrets versions add db-password \
  --data-file=-

# Access latest version
gcloud secrets versions access latest \
  --secret=db-password

# Access specific version
gcloud secrets versions access 2 \
  --secret=db-password
```

### IAM for Secrets

```bash
# Grant access to secret
gcloud secrets add-iam-policy-binding db-password \
  --member=serviceAccount:my-sa@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/secretmanager.secretAccessor
```

### Using Secrets in Applications

```python
from google.cloud import secretmanager

client = secretmanager.SecretManagerServiceClient()
name = "projects/PROJECT_ID/secrets/db-password/versions/latest"
response = client.access_secret_version(request={"name": name})
password = response.payload.data.decode("UTF-8")
```

---

## Data Loss Prevention (DLP)

### Inspecting Data for Sensitive Information

```python
from google.cloud import dlp_v2

dlp = dlp_v2.DlpServiceClient()
project = f"projects/{PROJECT_ID}"

# Inspect text for PII
item = {"value": "My email is alice@example.com and SSN is 123-45-6789"}
inspect_config = {
    "info_types": [
        {"name": "EMAIL_ADDRESS"},
        {"name": "US_SOCIAL_SECURITY_NUMBER"}
    ]
}

response = dlp.inspect_content(
    request={
        "parent": project,
        "inspect_config": inspect_config,
        "item": item
    }
)

for finding in response.result.findings:
    print(f"Found {finding.info_type.name}: {finding.quote}")
```

### De-identifying Data

```python
# Redact sensitive data
deidentify_config = {
    "info_type_transformations": {
        "transformations": [{
            "primitive_transformation": {
                "replace_config": {
                    "new_value": {"string_value": "[REDACTED]"}
                }
            }
        }]
    }
}

response = dlp.deidentify_content(
    request={
        "parent": project,
        "deidentify_config": deidentify_config,
        "inspect_config": inspect_config,
        "item": item
    }
)

print(response.item.value)
# Output: "My email is [REDACTED] and SSN is [REDACTED]"
```

---

## VPC Service Controls

### Creating Service Perimeter

**Purpose**: Prevent data exfiltration

```bash
# Create access policy
gcloud access-context-manager policies create \
  --organization=ORG_ID \
  --title="My Policy"

# Create service perimeter
gcloud access-context-manager perimeters create my_perimeter \
  --title="Production Perimeter" \
  --resources=projects/PROJECT_NUMBER \
  --restricted-services=storage.googleapis.com,bigquery.googleapis.com \
  --policy=POLICY_ID
```

**Effect**: Resources inside perimeter cannot be accessed from outside.

---

## Binary Authorization

### Enforcing Container Image Signatures

```bash
# Enable Binary Authorization
gcloud services enable binaryauthorization.googleapis.com

# Create attestor
gcloud container binauthz attestors create my-attestor \
  --attestation-authority-note=projects/PROJECT_ID/notes/my-note \
  --attestation-authority-note-project=PROJECT_ID

# Create policy requiring attestation
cat > policy.yaml <<EOF
admissionWhitelistPatterns:
- namePattern: gcr.io/PROJECT_ID/*
defaultAdmissionRule:
  requireAttestationsBy:
  - projects/PROJECT_ID/attestors/my-attestor
  evaluationMode: REQUIRE_ATTESTATION
  enforcementMode: ENFORCED_BLOCK_AND_AUDIT_LOG
EOF

gcloud container binauthz policy import policy.yaml
```

---

## Summary

**Key Takeaways**:
1. All GCP data is **encrypted at rest by default**
2. Use **CMEK** for compliance and key rotation control
3. Use **Secret Manager** for passwords, API keys, certificates
4. Enable **automatic key rotation** in Cloud KMS
5. Use **VPC Service Controls** to prevent data exfiltration
6. Use **DLP** to detect and redact sensitive information
7. Implement **Binary Authorization** for container security
8. Use **managed SSL certificates** for automatic renewal
