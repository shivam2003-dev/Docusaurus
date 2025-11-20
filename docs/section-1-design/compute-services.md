---
sidebar_position: 4
---

# 1.3 Compute Services

Master the different compute options on Google Cloud and learn when to use each service.

## Compute Decision Framework

### Understanding Your Workload

Before choosing a compute service, ask:
1. **Control vs Convenience**: Do you need OS-level control or prefer managed services?
2. **Stateful vs Stateless**: Does your app maintain state or is it stateless?
3. **Scaling Pattern**: Predictable or unpredictable traffic?
4. **Container-ready**: Is your app containerized?

---

## Compute Engine (IaaS)

### When to Use Compute Engine

**Best For**:
- Lift-and-shift migrations
- Custom OS requirements (Windows, specific Linux distros)
- Applications requiring specific kernel modules
- GPU/TPU workloads
- Full control over networking and storage

**Not Ideal For**:
- Simple web apps (use App Engine or Cloud Run)
- Serverless workloads (use Cloud Functions)

### Machine Type Families

#### E2 (Cost-Optimized)
```
Specs: 2-32 vCPUs, 0.5-128 GB RAM
Use Case: Development, testing, small databases
Cost: Lowest
Example: e2-medium (2 vCPU, 4 GB RAM)
```

#### N2/N2D (Balanced)
```
Specs: 2-128 vCPUs, 0.5-864 GB RAM
Use Case: General-purpose workloads, web servers
Cost: Medium
Example: n2-standard-4 (4 vCPU, 16 GB RAM)
```

#### C2/C2D (Compute-Optimized)
```
Specs: 4-112 vCPUs, High CPU-to-RAM ratio
Use Case: Gaming servers, HPC, scientific computing
Cost: Higher (per core)
Example: c2-standard-8 (8 vCPU, 32 GB RAM)
```

#### M2/M3 (Memory-Optimized)
```
Specs: 96-416 vCPUs, Up to 12 TB RAM
Use Case: In-memory databases (SAP HANA, Redis)
Cost: Highest
Example: m2-ultramem-208 (208 vCPU, 5.9 TB RAM)
```

### Creating VMs

```bash
# Basic VM
gcloud compute instances create my-vm \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-11 \
  --image-project=debian-cloud

# VM with startup script
gcloud compute instances create web-server \
  --zone=us-central1-a \
  --machine-type=n2-standard-2 \
  --tags=http-server \
  --metadata=startup-script='#!/bin/bash
    apt-get update
    apt-get install -y nginx
    systemctl start nginx'

# Preemptible VM
gcloud compute instances create batch-worker \
  --zone=us-central1-a \
  --machine-type=n2-standard-4 \
  --preemptible \
  --maintenance-policy=TERMINATE
```

### Managed Instance Groups (MIGs)

**Features**:
- Auto-scaling
- Auto-healing
- Regional distribution
- Load balancing integration

```bash
# Create instance template
gcloud compute instance-templates create web-template \
  --machine-type=n2-standard-2 \
  --image-family=debian-11 \
  --image-project=debian-cloud \
  --tags=http-server \
  --metadata=startup-script-url=gs://my-bucket/startup.sh

# Create regional MIG
gcloud compute instance-groups managed create web-mig \
  --base-instance-name=web \
  --template=web-template \
  --size=3 \
  --region=us-central1

# Configure autoscaling
gcloud compute instance-groups managed set-autoscaling web-mig \
  --region=us-central1 \
  --max-num-replicas=10 \
  --min-num-replicas=2 \
  --target-cpu-utilization=0.7 \
  --cool-down-period=60
```

---

## Google Kubernetes Engine (GKE)

### When to Use GKE

**Best For**:
- Microservices architectures
- Containerized applications
- Hybrid/multi-cloud deployments (with Anthos)
- Applications requiring Kubernetes features

### Cluster Types

#### Standard Cluster
**Control**: Full control over node configuration
**Use Case**: Production workloads requiring customization

```bash
gcloud container clusters create my-cluster \
  --zone=us-central1-a \
  --num-nodes=3 \
  --machine-type=n2-standard-4 \
  --enable-autoscaling \
  --min-nodes=2 \
  --max-nodes=10
```

#### Autopilot Cluster
**Control**: Google manages nodes
**Use Case**: Simplified operations, pay-per-pod

```bash
gcloud container clusters create-auto my-autopilot \
  --region=us-central1
```

### Private GKE Cluster

**Security**: Nodes have no external IPs

```bash
gcloud container clusters create private-cluster \
  --zone=us-central1-a \
  --enable-private-nodes \
  --enable-private-endpoint \
  --master-ipv4-cidr=172.16.0.0/28 \
  --enable-ip-alias
```

### GKE Workload Example

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: gcr.io/my-project/web:v1
        ports:
        - containerPort: 8080
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 500m
            memory: 512Mi
---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: web-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: web-app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

---

## App Engine

### Standard Environment

**Characteristics**:
- Scales to zero
- Fast startup (milliseconds)
- Specific runtimes (Python, Java, Go, Node.js, PHP, Ruby)
- Sandboxed environment

**Limitations**:
- No SSH access
- No custom binaries
- Request timeout: 10 minutes (60 seconds for automatic scaling)

```yaml
# app.yaml
runtime: python39
instance_class: F2

automatic_scaling:
  target_cpu_utilization: 0.65
  min_instances: 2
  max_instances: 10

env_variables:
  DATABASE_URL: "postgresql://..."
```

### Flexible Environment

**Characteristics**:
- Runs in Docker containers
- Any runtime/language
- SSH access available
- Slower startup (minutes)

**Use Case**: Custom runtimes, long-running processes

```yaml
# app.yaml
runtime: custom
env: flex

automatic_scaling:
  min_num_instances: 1
  max_num_instances: 5
  cpu_utilization:
    target_utilization: 0.7
```

---

## Cloud Run

### When to Use Cloud Run

**Best For**:
- Stateless HTTP services
- APIs and microservices
- Event-driven applications
- Serverless containers

**Key Features**:
- Scales to zero
- Pay per request
- Any language/runtime (containerized)
- Automatic HTTPS

### Deploying to Cloud Run

```bash
# Deploy from source (Cloud Run builds the container)
gcloud run deploy my-service \
  --source=. \
  --region=us-central1 \
  --allow-unauthenticated

# Deploy from container image
gcloud run deploy my-service \
  --image=gcr.io/my-project/my-app:v1 \
  --region=us-central1 \
  --platform=managed \
  --memory=512Mi \
  --cpu=1 \
  --max-instances=100 \
  --min-instances=0 \
  --concurrency=80
```

### Cloud Run Service YAML

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: my-service
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/maxScale: '100'
        autoscaling.knative.dev/minScale: '0'
    spec:
      containerConcurrency: 80
      containers:
      - image: gcr.io/my-project/my-app:v1
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

---

## Cloud Functions

### When to Use Cloud Functions

**Best For**:
- Event-driven processing
- Webhooks
- Scheduled tasks
- Lightweight APIs

**Triggers**:
- HTTP
- Pub/Sub
- Cloud Storage
- Firestore
- Firebase

### Example Functions

```python
# HTTP Function
def hello_http(request):
    name = request.args.get('name', 'World')
    return f'Hello, {name}!'

# Pub/Sub Function
def process_message(event, context):
    import base64
    message = base64.b64decode(event['data']).decode('utf-8')
    print(f'Processing: {message}')

# Cloud Storage Function
def process_file(event, context):
    file_name = event['name']
    bucket = event['bucket']
    print(f'File {file_name} uploaded to {bucket}')
```

### Deployment

```bash
# Deploy HTTP function
gcloud functions deploy hello-http \
  --runtime=python39 \
  --trigger-http \
  --allow-unauthenticated \
  --entry-point=hello_http

# Deploy Pub/Sub function
gcloud functions deploy process-message \
  --runtime=python39 \
  --trigger-topic=my-topic \
  --entry-point=process_message

# Deploy Storage function
gcloud functions deploy process-file \
  --runtime=python39 \
  --trigger-resource=my-bucket \
  --trigger-event=google.storage.object.finalize
```

---

## Compute Service Comparison

| Feature | Compute Engine | GKE | App Engine | Cloud Run | Cloud Functions |
|---------|----------------|-----|------------|-----------|-----------------|
| **Control** | Full | High | Medium | Low | Lowest |
| **Scaling** | Manual/Auto | Auto | Auto | Auto | Auto |
| **Scale to Zero** | No | No | Standard: Yes | Yes | Yes |
| **Startup Time** | Minutes | Seconds | Milliseconds | Seconds | Seconds |
| **Container Support** | Yes | Native | Flexible: Yes | Native | No |
| **Pricing Model** | Per VM | Per node | Per instance-hour | Per request | Per invocation |
| **Max Request Time** | Unlimited | Unlimited | 10 min | 60 min | 9 min (1st gen), 60 min (2nd gen) |

---

## Decision Tree

```
Need full OS control? → Compute Engine
Need Kubernetes? → GKE
Need simple web app with zero config? → App Engine Standard
Need custom runtime web app? → App Engine Flexible or Cloud Run
Need stateless HTTP service? → Cloud Run
Need event-driven function? → Cloud Functions
Need GPU/TPU? → Compute Engine or GKE
Need to scale to zero? → App Engine Standard, Cloud Run, or Cloud Functions
```

---

## Summary

**Key Takeaways**:
1. **Compute Engine** for full control and lift-and-shift
2. **GKE** for Kubernetes and microservices
3. **App Engine** for simple web apps with minimal ops
4. **Cloud Run** for stateless containers with auto-scaling
5. **Cloud Functions** for event-driven, single-purpose code
6. Choose based on control vs convenience trade-off
