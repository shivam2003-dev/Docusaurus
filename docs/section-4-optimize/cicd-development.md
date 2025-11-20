---
sidebar_position: 2
---

# 4.1 CI/CD and Development Processes

Master continuous integration, continuous deployment, and software development lifecycle best practices for Google Cloud.

## CI/CD on Google Cloud

### Cloud Build

**Purpose**: Serverless CI/CD platform

**Build Configuration** (cloudbuild.yaml):
```yaml
steps:
# Step 1: Run tests
- name: 'gcr.io/cloud-builders/npm'
  args: ['install']
- name: 'gcr.io/cloud-builders/npm'
  args: ['test']

# Step 2: Build Docker image
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'gcr.io/$PROJECT_ID/my-app:$SHORT_SHA', '.']

# Step 3: Push to Container Registry
- name: 'gcr.io/cloud-builders/docker'
  args: ['push', 'gcr.io/$PROJECT_ID/my-app:$SHORT_SHA']

# Step 4: Deploy to Cloud Run
- name: 'gcr.io/cloud-builders/gcloud'
  args:
  - 'run'
  - 'deploy'
  - 'my-service'
  - '--image=gcr.io/$PROJECT_ID/my-app:$SHORT_SHA'
  - '--region=us-central1'
  - '--platform=managed'

images:
- 'gcr.io/$PROJECT_ID/my-app:$SHORT_SHA'

options:
  machineType: 'N1_HIGHCPU_8'
  logging: CLOUD_LOGGING_ONLY
```

**Trigger on Git Push**:
```bash
# Create trigger
gcloud builds triggers create github \
  --repo-name=my-repo \
  --repo-owner=my-org \
  --branch-pattern="^main$" \
  --build-config=cloudbuild.yaml
```

### Artifact Registry

**Purpose**: Store Docker images, npm packages, Maven artifacts

```bash
# Create repository
gcloud artifacts repositories create my-repo \
  --repository-format=docker \
  --location=us-central1 \
  --description="Docker repository"

# Configure Docker
gcloud auth configure-docker us-central1-docker.pkg.dev

# Push image
docker tag my-app us-central1-docker.pkg.dev/PROJECT_ID/my-repo/my-app:v1
docker push us-central1-docker.pkg.dev/PROJECT_ID/my-repo/my-app:v1
```

---

## Deployment Strategies

### Blue/Green Deployment

**Concept**: Run two identical environments, switch traffic instantly

```bash
# Deploy "green" version
gcloud run deploy my-service \
  --image=gcr.io/PROJECT_ID/my-app:v2 \
  --tag=green \
  --no-traffic

# Test green version
curl https://green---my-service-xxx.run.app

# Switch 100% traffic to green
gcloud run services update-traffic my-service \
  --to-tags=green=100
```

### Canary Deployment

**Concept**: Gradually shift traffic to new version

```bash
# Deploy v2 with 10% traffic
gcloud run deploy my-service \
  --image=gcr.io/PROJECT_ID/my-app:v2 \
  --tag=canary

gcloud run services update-traffic my-service \
  --to-revisions=LATEST=10,PREVIOUS=90

# Monitor metrics, then increase
gcloud run services update-traffic my-service \
  --to-revisions=LATEST=50,PREVIOUS=50

# Finally, 100% to new version
gcloud run services update-traffic my-service \
  --to-revisions=LATEST=100
```

### Rolling Update (GKE)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 10
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2        # Max 2 extra pods during update
      maxUnavailable: 1  # Max 1 pod unavailable
  template:
    spec:
      containers:
      - name: app
        image: gcr.io/PROJECT_ID/my-app:v2
```

---

## Infrastructure as Code

### Terraform

**Example**: Provision GKE cluster

```hcl
# main.tf
resource "google_container_cluster" "primary" {
  name     = "my-gke-cluster"
  location = "us-central1"
  
  initial_node_count = 1
  
  node_config {
    machine_type = "n2-standard-4"
    oauth_scopes = [
      "https://www.googleapis.com/auth/cloud-platform"
    ]
  }
  
  addons_config {
    http_load_balancing {
      disabled = false
    }
    horizontal_pod_autoscaling {
      disabled = false
    }
  }
}
```

```bash
# Deploy
terraform init
terraform plan
terraform apply
```

### Deployment Manager

```yaml
# cluster.yaml
resources:
- name: my-cluster
  type: container.v1.cluster
  properties:
    zone: us-central1-a
    cluster:
      initialNodeCount: 3
      nodeConfig:
        machineType: n2-standard-4
```

```bash
gcloud deployment-manager deployments create my-deployment \
  --config=cluster.yaml
```

---

## Testing Strategies

### Unit Testing

```python
# test_app.py
import unittest
from app import calculate_total

class TestApp(unittest.TestCase):
    def test_calculate_total(self):
        self.assertEqual(calculate_total([10, 20, 30]), 60)
        
if __name__ == '__main__':
    unittest.main()
```

### Integration Testing

```yaml
# cloudbuild.yaml
steps:
- name: 'gcr.io/cloud-builders/docker'
  args: ['build', '-t', 'test-image', '.']
  
- name: 'test-image'
  args: ['python', '-m', 'pytest', 'tests/integration/']
```

### Load Testing

```bash
# Use Cloud Load Testing (based on Locust)
gcloud alpha builds submit \
  --config=loadtest.yaml \
  --substitutions=_TARGET_URL=https://my-app.run.app
```

---

## Disaster Recovery Planning

### RTO and RPO

**RTO (Recovery Time Objective)**: How long can you be down?
**RPO (Recovery Point Objective)**: How much data can you lose?

| Strategy | RTO | RPO | Cost |
|----------|-----|-----|------|
| Backup & Restore | Hours-Days | Hours | Low |
| Pilot Light | Minutes-Hours | Minutes | Medium |
| Warm Standby | Minutes | Minutes | Medium-High |
| Hot Standby | Seconds | Near-zero | High |

### Backup Strategy

**Cloud SQL**:
```bash
# Automated backups (enabled by default)
gcloud sql instances patch my-instance \
  --backup-start-time=03:00 \
  --retained-backups-count=7

# On-demand backup
gcloud sql backups create \
  --instance=my-instance \
  --description="Before migration"
```

**Cloud Storage**:
```bash
# Enable versioning
gsutil versioning set on gs://my-bucket

# Set lifecycle to keep versions for 30 days
cat > lifecycle.json <<EOF
{
  "lifecycle": {
    "rule": [{
      "action": {"type": "Delete"},
      "condition": {
        "age": 30,
        "isLive": false
      }
    }]
  }
}
EOF

gsutil lifecycle set lifecycle.json gs://my-bucket
```

### Multi-Region Replication

**Cloud Storage**:
```bash
# Create multi-region bucket
gsutil mb -c STANDARD -l US gs://my-multi-region-bucket
```

**Cloud Spanner**:
```bash
# Create multi-region instance
gcloud spanner instances create my-instance \
  --config=nam-eur-asia1 \
  --nodes=3 \
  --description="Multi-region instance"
```

---

## Cost Optimization

### Committed Use Discounts

```bash
# Purchase 1-year commitment
gcloud compute commitments create my-commitment \
  --region=us-central1 \
  --resources=vcpu=100,memory=400GB \
  --plan=12-month
```

**Savings**: Up to 57% for 3-year commitment

### Rightsizing Recommendations

```bash
# Get recommendations
gcloud recommender recommendations list \
  --project=PROJECT_ID \
  --recommender=google.compute.instance.MachineTypeRecommender \
  --location=us-central1
```

### Budget Alerts

```bash
# Create budget
gcloud billing budgets create \
  --billing-account=BILLING_ACCOUNT_ID \
  --display-name="Monthly Budget" \
  --budget-amount=1000USD \
  --threshold-rule=percent=50 \
  --threshold-rule=percent=90 \
  --threshold-rule=percent=100
```

---

## Summary

**Key Takeaways**:
1. Use **Cloud Build** for serverless CI/CD
2. Implement **canary deployments** for safe rollouts
3. Use **Infrastructure as Code** (Terraform/Deployment Manager)
4. Define **RTO/RPO** requirements for DR planning
5. Enable **automated backups** for all databases
6. Use **committed use discounts** for predictable workloads
7. Monitor **cost recommendations** regularly
8. Implement **multi-region replication** for critical data
