---
sidebar_position: 2
---

# 6.1 Monitoring and Logging

Master Cloud Operations Suite (formerly Stackdriver) for monitoring, logging, tracing, and debugging applications on Google Cloud.

## Cloud Monitoring

### Metrics and Dashboards

**Create Custom Dashboard**:
```bash
# Via gcloud (using JSON config)
gcloud monitoring dashboards create --config-from-file=dashboard.json
```

**Dashboard JSON**:
```json
{
  "displayName": "My Dashboard",
  "dashboardFilters": [],
  "gridLayout": {
    "widgets": [{
      "title": "CPU Utilization",
      "xyChart": {
        "dataSets": [{
          "timeSeriesQuery": {
            "timeSeriesFilter": {
              "filter": "resource.type=\"gce_instance\"",
              "aggregation": {
                "alignmentPeriod": "60s",
                "perSeriesAligner": "ALIGN_MEAN"
              }
            }
          }
        }]
      }
    }]
  }
}
```

### Uptime Checks

```bash
# Create HTTP uptime check
gcloud monitoring uptime create my-check \
  --resource-type=uptime-url \
  --display-name="Website Check" \
  --http-check-path=/ \
  --monitored-resource=my-service \
  --period=60 \
  --timeout=10
```

### Alerting Policies

```bash
# Create alert for high CPU
gcloud alpha monitoring policies create \
  --notification-channels=CHANNEL_ID \
  --display-name="High CPU Alert" \
  --condition-display-name="CPU > 80%" \
  --condition-threshold-value=0.8 \
  --condition-threshold-duration=300s \
  --condition-filter='resource.type="gce_instance" AND metric.type="compute.googleapis.com/instance/cpu/utilization"'
```

### Custom Metrics

```python
from google.cloud import monitoring_v3
import time

client = monitoring_v3.MetricServiceClient()
project_name = f"projects/{PROJECT_ID}"

series = monitoring_v3.TimeSeries()
series.metric.type = "custom.googleapis.com/my_metric"
series.resource.type = "global"

point = monitoring_v3.Point()
point.value.double_value = 42.0
point.interval.end_time.seconds = int(time.time())
series.points = [point]

client.create_time_series(name=project_name, time_series=[series])
```

---

## Cloud Logging

### Log Types

1. **Admin Activity Logs** (always on, free)
2. **Data Access Logs** (must enable, costs money)
3. **System Event Logs** (always on, free)
4. **Access Transparency Logs** (enterprise only)

### Writing Logs

```python
from google.cloud import logging

logging_client = logging.Client()
logger = logging_client.logger("my-app")

logger.log_text("Application started")
logger.log_struct({
    "message": "User login",
    "user_id": "123",
    "ip_address": "203.0.113.1"
}, severity="INFO")
```

### Log Queries

```bash
# Query logs
gcloud logging read \
  'resource.type="gce_instance" AND severity="ERROR"' \
  --limit=50 \
  --format=json

# Real-time tail
gcloud logging tail \
  'resource.type="cloud_run_revision" AND resource.labels.service_name="my-service"'
```

**Advanced Query**:
```
resource.type="k8s_container"
resource.labels.namespace_name="production"
severity >= ERROR
timestamp >= "2024-01-15T00:00:00Z"
jsonPayload.user_id="123"
```

### Log-Based Metrics

```bash
# Create metric from logs
gcloud logging metrics create error_count \
  --description="Count of errors" \
  --log-filter='severity="ERROR"'
```

### Log Sinks

**Export to BigQuery**:
```bash
# Create dataset
bq mk --dataset logs_dataset

# Create sink
gcloud logging sinks create my-sink \
  bigquery.googleapis.com/projects/PROJECT_ID/datasets/logs_dataset \
  --log-filter='resource.type="gce_instance"'
```

**Export to Cloud Storage**:
```bash
gcloud logging sinks create storage-sink \
  storage.googleapis.com/my-logs-bucket \
  --log-filter='resource.type="cloud_run_revision"'
```

---

## Cloud Trace

### Distributed Tracing

**Automatic Tracing** (Cloud Run, App Engine, GKE with Istio)

**Manual Instrumentation**:
```python
from google.cloud import trace_v1

tracer = trace_v1.TraceServiceClient()
project_id = "my-project"

# Create trace
trace = {
    "project_id": project_id,
    "spans": [{
        "span_id": "1",
        "name": "my-operation",
        "start_time": "2024-01-15T10:00:00Z",
        "end_time": "2024-01-15T10:00:01Z"
    }]
}

tracer.patch_traces(project_id=project_id, traces={"traces": [trace]})
```

---

## Cloud Profiler

### CPU and Memory Profiling

```python
# Enable profiler in application
import googlecloudprofiler

googlecloudprofiler.start(
    service='my-service',
    service_version='1.0.0',
    verbose=3
)
```

**Analyze**:
- CPU usage by function
- Memory allocation
- Heap profiles
- Wall time

---

## Error Reporting

### Automatic Error Detection

**Supported**: App Engine, Cloud Functions, Cloud Run, GKE

**Manual Reporting**:
```python
from google.cloud import error_reporting

client = error_reporting.Client()

try:
    raise Exception("Something went wrong!")
except Exception:
    client.report_exception()
```

---

## Service Level Objectives (SLOs)

### Defining SLOs

**Example SLO**: 99.9% of requests complete in < 500ms

```yaml
# slo.yaml
serviceLevelIndicator:
  requestBased:
    goodTotalRatio:
      goodServiceFilter: |
        metric.type="loadbalancing.googleapis.com/https/request_count"
        metric.label.response_code_class="2xx"
        metric.label.response_code_class="3xx"
      totalServiceFilter: |
        metric.type="loadbalancing.googleapis.com/https/request_count"

goal: 0.999
rollingPeriod: 2592000s  # 30 days
```

---

## Summary

**Key Takeaways**:
1. Use **Cloud Monitoring** for metrics and alerts
2. Enable **Data Access Logs** for compliance
3. Export logs to **BigQuery** for analysis
4. Use **Cloud Trace** for distributed tracing
5. Enable **Cloud Profiler** to find performance bottlenecks
6. Define **SLOs** to measure reliability
7. Create **log-based metrics** for custom monitoring
8. Use **Error Reporting** for automatic error detection
