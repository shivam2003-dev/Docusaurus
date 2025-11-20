---
sidebar_position: 5
---

# 4. Analyzing and Optimizing Technical and Business Processes

This section bridges the gap between technical implementation and business value, focusing on optimization and process improvement.

## 4.1 Analyzing and Defining Technical Processes

### Software Development Life Cycle (SDLC)
*   **Continuous Integration (CI)**: Automatically build and test code changes.
    *   *Tool*: **Cloud Build**. Triggers on Git push.
*   **Continuous Delivery (CD)**: Automatically deploy to staging/production.
    *   *Tool*: **Cloud Deploy** (for GKE/Cloud Run) or **Spinnaker**.
*   **Testing Strategies**:
    *   **Unit Testing**: Test individual functions.
    *   **Integration Testing**: Test interaction between services.
    *   **Load Testing**: Simulate high traffic (e.g., using JMeter on GKE).

### Disaster Recovery (DR) Planning
*   **RTO (Recovery Time Objective)**: Max acceptable downtime.
*   **RPO (Recovery Point Objective)**: Max acceptable data loss.
*   **DR Patterns**:
    *   **Cold**: Backup and Restore (Cheapest, High RTO).
    *   **Warm**: Standby resources running but scaled down (Medium Cost, Medium RTO).
    *   **Hot**: Active-Active (Most Expensive, Near-Zero RTO).

---

## 4.2 Analyzing and Defining Business Processes

### Cost Management
*   **Budgets & Alerts**: Set a budget (e.g., $1000/month) and get emails when you reach 50%, 90%, 100%.
    *   *Note*: Budgets **DO NOT** stop spending. They only alert. To stop spending, you need a Cloud Function triggered by Pub/Sub to disable billing (risky!).
*   **Billing Export**: Export billing data to **BigQuery** for detailed analysis (SQL queries).
*   **Labels**: Tag resources (e.g., `env:prod`, `dept:finance`) to break down costs.

### Stakeholder Management
*   **Post-Mortems**: Blameless documents written after an incident to prevent recurrence.
*   **Capacity Planning**: Forecasting future growth to reserve capacity (CUDs).

---

## 4.3 Developing Procedures to Ensure Reliability of Operations in Production

### Chaos Engineering
*   Proactively breaking things (killing VMs, adding latency) to test system resilience.
*   *Goal*: Find weaknesses before they cause a real outage.

### Penetration Testing
*   Simulating cyberattacks to find vulnerabilities.
*   *Note*: You generally do **not** need Google's permission to pen-test your own projects, but you must abide by the Acceptable Use Policy.

---

## 📝 Practice Questions

### Question 1
**Scenario**: You want to analyze your monthly Google Cloud spend by department. You have already applied labels like `department:marketing` and `department:engineering` to all resources.
**What is the most effective way to generate this report?**

*   A. Download the invoice PDF and manually calculate the totals.
*   B. Use the Pricing Calculator.
*   C. Enable Billing Export to BigQuery and run a SQL query grouping by label.
*   D. Create a separate Project for each department.

<details>
<summary>Click to reveal answer</summary>

**Answer: C (Enable Billing Export to BigQuery)**
*   **Explanation**: BigQuery allows granular analysis of billing data using SQL, including grouping by labels.
*   *D* is a valid strategy (Project-based accounting), but if resources are already in one project, labels are the way to go.
</details>

### Question 2
**Scenario**: Your manager wants to ensure that if the monthly budget of $5,000 is exceeded, all compute resources are automatically shut down to prevent further charges.
**How can you achieve this?**

*   A. Configure a Budget Alert to send an email to the admin.
*   B. Configure a Budget Alert to publish a message to Pub/Sub, which triggers a Cloud Function to disable billing.
*   C. Use the "Cap Billing" feature in the Billing Console.
*   D. Set a quota of $0 on the project.

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Pub/Sub -> Cloud Function)**
*   **Explanation**: There is no native "Hard Cap" feature in GCP Billing. You must build an automation using Pub/Sub and Cloud Functions to programmatically disable billing or stop resources.
</details>
