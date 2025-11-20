---
sidebar_position: 7
---

# 6. Ensuring Solution and Operations Reliability

This section covers the "Day 2" operations: keeping the lights on, monitoring performance, and troubleshooting issues.

## 6.1 Monitoring/Logging/Profiling/Alerting Solution

### Cloud Operations Suite (formerly Stackdriver)

#### Cloud Monitoring
*   **Metrics**: Numerical data (CPU usage, disk I/O).
*   **Dashboards**: Visualize metrics.
*   **Uptime Checks**: Ping your service from around the world to see if it's reachable.
*   **Alerting Policies**: "If CPU > 90% for 5 mins, send SMS."

#### Cloud Logging
*   **Log Router**: Ingests logs.
*   **Sinks**: Export logs to:
    *   **Cloud Storage**: Long-term retention (Audit compliance).
    *   **BigQuery**: Analytics (SQL queries on logs).
    *   **Pub/Sub**: Streaming to external SIEM (Splunk, Datadog).
*   **Exclusion Filters**: Discard noisy logs to save money.

#### Application Performance Management (APM)
*   **Cloud Trace**: Latency sampling. Find out *where* the request is slow (e.g., Database call took 2s).
*   **Cloud Profiler**: CPU/RAM profiling. Find out *which function* is consuming resources.

---

## 6.2 Deployment and Release Management

### Deployment Strategies
*   **Blue/Green**:
    *   *Blue*: Current version (Live).
    *   *Green*: New version (Idle).
    *   *Switch*: Route 100% traffic to Green. Instant rollback possible.
*   **Canary**:
    *   Roll out to a small % of users (e.g., 10%).
    *   If errors spike, roll back. If good, increase to 100%.
*   **Rolling Update**:
    *   Update instance by instance. (Standard in MIGs and GKE).

### Traffic Splitting
*   **App Engine**: Native support. "Split traffic 90/10".
*   **Cloud Run**: Native support. "Send 5% to revision-2".
*   **Istio (ASM)**: Advanced traffic splitting for Kubernetes.

---

## 6.3 Assisting with the Support of Solutions in Operation

### Troubleshooting
*   **Connectivity Issues**: Use **Network Intelligence Center** -> **Connectivity Tests**. It simulates a packet from Source to Dest and tells you where it blocked (Firewall? Route? IAM?).
*   **Permission Issues**: Use **Policy Troubleshooter**. "Why did Bob get Access Denied?"

### Escalation
*   **Google Cloud Support**:
    *   **Basic**: Billing only.
    *   **Standard**: P2 support.
    *   **Enhanced**: P1 support (1 hr response).
    *   **Premium**: P1 support (15 min response), dedicated TAM (Technical Account Manager).

---

## 📝 Practice Questions

### Question 1
**Scenario**: You need to store your application logs for 7 years for audit purposes. You want to minimize the cost of storage.
**Where should you export the logs?**

*   A. BigQuery
*   B. Cloud Storage (Archive Class)
*   C. Pub/Sub
*   D. Keep them in Cloud Logging

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Cloud Storage - Archive Class)**
*   **Explanation**: Cloud Storage Archive class is the cheapest option for long-term retention where immediate access is not required.
*   *BigQuery* is for analytics (more expensive).
*   *Cloud Logging* retention is limited (usually 30 days by default).
</details>

### Question 2
**Scenario**: Users are reporting that your application is slow. You suspect a specific database query is the bottleneck.
**Which tool should you use to debug this?**

*   A. Cloud Monitoring
*   B. Cloud Logging
*   C. Cloud Trace
*   D. Cloud Profiler

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Cloud Trace)**
*   **Explanation**: Cloud Trace captures the latency of requests as they propagate through your microservices, allowing you to see exactly how long the database call took.
*   *Cloud Profiler* is for CPU/RAM usage of code functions, not network/DB latency.
</details>
