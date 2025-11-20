---
sidebar_position: 4
---

# 2.3 Compute Provisioning

Learn how to provision and configure Compute Engine VMs, GKE clusters, and serverless compute resources.

## Compute Engine Provisioning

### Instance Templates

**Purpose**: Define VM configuration for reuse in MIGs

```bash
# Create instance template
gcloud compute instance-templates create web-template \
  --machine-type=n2-standard-2 \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --boot-disk-size=20GB \
  --boot-disk-type=pd-balanced \
  --tags=http-server,https-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl enable nginx
    systemctl start nginx' \
  --service-account=web-sa@project.iam.gserviceaccount.com \
  --scopes=cloud-platform
```

### Managed Instance Groups (MIGs)

**Regional MIG** (High Availability):
```bash
# Create regional MIG
gcloud compute instance-groups managed create web-mig \
  --base-instance-name=web \
  --template=web-template \
  --size=3 \
  --region=us-central1 \
  --target-distribution-shape=EVEN

# Configure autoscaling
gcloud compute instance-groups managed set-autoscaling web-mig \
  --region=us-central1 \
  --max-num-replicas=10 \
  --min-num-replicas=2 \
  --target-cpu-utilization=0.7 \
  --cool-down-period=60

# Set auto-healing
gcloud compute instance-groups managed set-autohealing web-mig \
  --region=us-central1 \
  --health-check=web-health-check \
  --initial-delay=300
```

### Rolling Updates

```bash
# Update instance template
gcloud compute instance-templates create web-template-v2 \
  --machine-type=n2-standard-4 \
  --image-family=debian-11 \
  --image-project=debian-cloud

# Perform rolling update
gcloud compute instance-groups managed rolling-action start-update web-mig \
  --region=us-central1 \
  --version=template=web-template-v2 \
  --max-surge=3 \
  --max-unavailable=0
```

### Custom Machine Types

```bash
# Create custom machine (6 vCPUs, 12 GB RAM)
gcloud compute instances create custom-vm \
  --custom-cpu=6 \
  --custom-memory=12GB \
  --zone=us-central1-a
```

### GPU Instances

```bash
# Create VM with NVIDIA T4 GPU
gcloud compute instances create gpu-vm \
  --zone=us-central1-a \
  --machine-type=n1-standard-4 \
  --accelerator=type=nvidia-tesla-t4,count=1 \
  --maintenance-policy=TERMINATE \
  --image-family=pytorch-latest-gpu \
  --image-project=deeplearning-platform-release
```

---

## GKE Cluster Provisioning

### Standard Cluster

```bash
# Create production-ready cluster
gcloud container clusters create prod-cluster \
  --region=us-central1 \
  --num-nodes=2 \
  --machine-type=n2-standard-4 \
  --disk-size=50 \
  --disk-type=pd-ssd \
  --enable-autoscaling \
  --min-nodes=2 \
  --max-nodes=10 \
  --enable-autorepair \
  --enable-autoupgrade \
  --maintenance-window-start=2024-01-01T00:00:00Z \
  --maintenance-window-duration=4h \
  --enable-stackdriver-kubernetes \
  --addons=HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver
```

### Private GKE Cluster

```bash
# Create private cluster
gcloud container clusters create private-cluster \
  --region=us-central1 \
  --enable-private-nodes \
  --enable-private-endpoint \
  --master-ipv4-cidr=172.16.0.0/28 \
  --enable-ip-alias \
  --network=my-vpc \
  --subnetwork=gke-subnet \
  --cluster-secondary-range-name=pods \
  --services-secondary-range-name=services
```

### Node Pools

```bash
# Add node pool for specific workloads
gcloud container node-pools create high-mem-pool \
  --cluster=prod-cluster \
  --region=us-central1 \
  --machine-type=n2-highmem-4 \
  --num-nodes=1 \
  --enable-autoscaling \
  --min-nodes=0 \
  --max-nodes=5 \
  --node-taints=workload=memory-intensive:NoSchedule

# Add preemptible node pool
gcloud container node-pools create preemptible-pool \
  --cluster=prod-cluster \
  --region=us-central1 \
  --preemptible \
  --machine-type=n2-standard-4 \
  --num-nodes=0 \
  --enable-autoscaling \
  --min-nodes=0 \
  --max-nodes=10
```

### GKE Autopilot

```bash
# Create Autopilot cluster (fully managed)
gcloud container clusters create-auto autopilot-cluster \
  --region=us-central1 \
  --network=my-vpc \
  --subnetwork=gke-subnet
```

### Workload Identity

```bash
# Enable Workload Identity on cluster
gcloud container clusters update prod-cluster \
  --region=us-central1 \
  --workload-pool=PROJECT_ID.svc.id.goog

# Configure for namespace
kubectl create serviceaccount my-ksa -n default

gcloud iam service-accounts add-iam-policy-binding \
  my-gsa@PROJECT_ID.iam.gserviceaccount.com \
  --role=roles/iam.workloadIdentityUser \
  --member="serviceAccount:PROJECT_ID.svc.id.goog[default/my-ksa]"

kubectl annotate serviceaccount my-ksa \
  iam.gke.io/gcp-service-account=my-gsa@PROJECT_ID.iam.gserviceaccount.com
```

---

## App Engine Deployment

### Standard Environment

```yaml
# app.yaml
runtime: python39
instance_class: F2

automatic_scaling:
  target_cpu_utilization: 0.65
  min_instances: 2
  max_instances: 10
  min_pending_latency: 30ms
  max_pending_latency: automatic
  max_concurrent_requests: 50

env_variables:
  DATABASE_URL: "postgresql://..."
  
handlers:
- url: /static
  static_dir: static
  
- url: /.*
  script: auto
  secure: always
```

**Deploy**:
```bash
gcloud app deploy app.yaml \
  --project=PROJECT_ID \
  --version=v1 \
  --promote
```

### Flexible Environment

```yaml
# app.yaml
runtime: custom
env: flex

automatic_scaling:
  min_num_instances: 1
  max_num_instances: 5
  cool_down_period_sec: 120
  cpu_utilization:
    target_utilization: 0.7

resources:
  cpu: 2
  memory_gb: 4
  disk_size_gb: 10

network:
  instance_tag: app-engine-flex
```

### Traffic Splitting

```bash
# Split traffic between versions
gcloud app services set-traffic default \
  --splits=v1=0.9,v2=0.1 \
  --split-by=ip

# Migrate traffic gradually
gcloud app services set-traffic default \
  --splits=v2=1 \
  --migrate
```

---

## Cloud Run Deployment

### Deploy from Source

```bash
# Deploy directly from source code
gcloud run deploy my-service \
  --source=. \
  --region=us-central1 \
  --platform=managed \
  --allow-unauthenticated \
  --memory=512Mi \
  --cpu=1 \
  --min-instances=0 \
  --max-instances=100 \
  --concurrency=80 \
  --timeout=300 \
  --set-env-vars="DATABASE_URL=postgresql://..." \
  --vpc-connector=my-connector
```

### Deploy from Container

```bash
# Build and push container
docker build -t gcr.io/PROJECT_ID/my-app:v1 .
docker push gcr.io/PROJECT_ID/my-app:v1

# Deploy
gcloud run deploy my-service \
  --image=gcr.io/PROJECT_ID/my-app:v1 \
  --region=us-central1 \
  --platform=managed
```

### Service YAML

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: my-service
  annotations:
    run.googleapis.com/ingress: all
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: '100'
        autoscaling.knative.dev/minScale: '1'
        run.googleapis.com/vpc-access-connector: my-connector
    spec:
      containerConcurrency: 80
      timeoutSeconds: 300
      containers:
      - image: gcr.io/PROJECT_ID/my-app:v1
        ports:
        - containerPort: 8080
        resources:
          limits:
            memory: 512Mi
            cpu: '1'
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-secret
              key: url
```

### Traffic Management

```bash
# Deploy new revision without traffic
gcloud run deploy my-service \
  --image=gcr.io/PROJECT_ID/my-app:v2 \
  --no-traffic \
  --tag=canary

# Split traffic
gcloud run services update-traffic my-service \
  --to-revisions=LATEST=90,canary=10

# Rollback
gcloud run services update-traffic my-service \
  --to-revisions=PREVIOUS=100
```

---

## Cloud Functions Deployment

### HTTP Function

```python
# main.py
def hello_http(request):
    name = request.args.get('name', 'World')
    return f'Hello, {name}!'
```

```bash
# Deploy
gcloud functions deploy hello-http \
  --runtime=python39 \
  --trigger-http \
  --allow-unauthenticated \
  --entry-point=hello_http \
  --memory=256MB \
  --timeout=60s \
  --max-instances=100 \
  --set-env-vars="API_KEY=xxx"
```

### Event-Driven Functions

**Pub/Sub Trigger**:
```bash
gcloud functions deploy process-message \
  --runtime=python39 \
  --trigger-topic=my-topic \
  --entry-point=process_message
```

**Cloud Storage Trigger**:
```bash
gcloud functions deploy process-upload \
  --runtime=python39 \
  --trigger-resource=my-bucket \
  --trigger-event=google.storage.object.finalize \
  --entry-point=process_file
```

**Firestore Trigger**:
```bash
gcloud functions deploy on-user-create \
  --runtime=python39 \
  --trigger-event=providers/cloud.firestore/eventTypes/document.create \
  --trigger-resource='projects/PROJECT_ID/databases/(default)/documents/users/{userId}'
```

---

## Serverless VPC Access

### Create VPC Connector

```bash
# Create connector
gcloud compute networks vpc-access connectors create my-connector \
  --region=us-central1 \
  --network=my-vpc \
  --range=10.8.0.0/28 \
  --min-instances=2 \
  --max-instances=10
```

**Use in Cloud Run**:
```bash
gcloud run deploy my-service \
  --vpc-connector=my-connector \
  --vpc-egress=private-ranges-only
```

---

## Summary

**Key Takeaways**:
1. Use **instance templates** for consistent VM configuration
2. Configure **autoscaling** and **auto-healing** for MIGs
3. Use **regional MIGs** for high availability
4. Enable **Workload Identity** for GKE security
5. Use **private clusters** for production GKE
6. Deploy **Cloud Run** with VPC connectors for private access
7. Use **traffic splitting** for canary deployments
8. Configure **min instances** to reduce cold starts
