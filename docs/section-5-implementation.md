---
sidebar_position: 6
---

# 5. Managing Implementation

This section focuses on the "How-To" of deploying and interacting with your cloud solution.

## 5.1 Advising Development/Operation Teams to Ensure Successful Deployment

### Application Migration
*   **Containerization**: Packaging apps with Docker for consistency across environments.
*   **Artifact Registry**: Store Docker images, Maven/npm packages. (Replaces Container Registry).

### API Management
*   **Apigee**: Enterprise-grade API management. Analytics, monetization, rate limiting, quota enforcement.
*   **Cloud Endpoints**: Lighter weight, NGINX-based proxy. Good for gRPC and simple HTTP APIs.
*   **API Gateway**: Fully managed, serverless gateway for Cloud Functions/Run.

### Data Migration
*   **Database Migration Service (DMS)**: Serverless, easy migration to Cloud SQL (MySQL/PostgreSQL).
*   **Datastream**: Serverless Change Data Capture (CDC) and replication service.

---

## 5.2 Interacting with Google Cloud Programmatically

### Google Cloud SDK (`gcloud`)
The command-line interface (CLI) for GCP.
*   `gcloud init`: Initialize configuration.
*   `gcloud config set project [PROJECT_ID]`: Switch context.
*   `gcloud compute instances create`: Make a VM.
*   `gcloud container clusters get-credentials`: Configure `kubectl` to talk to GKE.

### Cloud Shell
*   Interactive shell environment in the browser.
*   **5GB** persistent home directory.
*   Pre-installed tools: `gcloud`, `kubectl`, `docker`, `git`, `vim`, `nano`.
*   **Web Preview**: Preview web apps running on port 8080.

### Emulators
*   Test locally without spending money.
*   Available for: **Datastore**, **Firestore**, **Pub/Sub**, **Bigtable**, **Spanner**.

:::tip Exam Tip
If a developer wants to test a Cloud Function triggered by Pub/Sub **locally** before deploying, advise them to use the **Pub/Sub Emulator**.
:::

---

## 📝 Practice Questions

### Question 1
**Scenario**: You have a legacy monolithic application that you want to modernize. You decide to break it down into microservices and deploy them on GKE.
**Which tool should you use to store the container images?**

*   A. Cloud Storage
*   B. Artifact Registry
*   C. Cloud Build
*   D. Cloud Source Repositories

<details>
<summary>Click to reveal answer</summary>

**Answer: B (Artifact Registry)**
*   **Explanation**: Artifact Registry is the successor to Container Registry and is the standard place to store container images and language packages.
</details>

### Question 2
**Scenario**: You need to expose a set of Cloud Functions as a unified API to external clients. You need to enforce API keys and validate JWT tokens.
**Which service is best suited for this?**

*   A. Cloud Load Balancing
*   B. API Gateway
*   C. Cloud Armor
*   D. VPC Service Controls

<details>
<summary>Click to reveal answer</summary>

**Answer: B (API Gateway)**
*   **Explanation**: API Gateway is designed to sit in front of serverless backends (Functions, Run) and handle API concerns like auth (API Keys, JWT) and routing.
</details>
