---
title: "Project Proposal"
date: 2026-08-07
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Technical Project Proposal

### Project Title: 100% Serverless Phishing Email Detection & Automated Incident Alert System on AWS

---

### 1. Executive Summary
In the era of digital enterprise operations and remote collaboration, email remains the lifeblood of corporate communication. Concurrently, it represents the primary attack vector for cyber adversaries. **Phishing attacks** — including sophisticated financial impersonations, Business Email Compromise (BEC), and urgent credential-harvesting schemes — inflict billions of dollars in global enterprise losses annually.

This project designs and delivers a comprehensive cybersecurity solution: Integrating **Machine Learning (XGBoost + TruncatedSVD)** with a **100% Serverless Architecture** on **Amazon Web Services (AWS)**, embedded directly into corporate workflows via a **Google Chrome Extension for Gmail**.

---

### 2. Problem Statement

{{% notice warning %}}
**Enterprise Threat Reality:** Cyber adversaries continually launch sophisticated social engineering campaigns (banking impersonation, typosquatting domain lures, urgent MFA/password credential harvesting). Distracted or hurried enterprise end-users routinely fall victim directly within their Gmail clients.
{{% /notice %}}

#### Comparative Matrix: Traditional Gateway vs. 100% Serverless Architecture:

| Evaluation Dimension | Legacy Perimeter Email Gateway | 100% Serverless Architecture (This Workshop) |
| :--- | :--- | :--- |
| **Infrastructure & TCO** | Costly ($30 - $50 USD/month idle EC2 VM hosting overhead). | **$0.00 TCO** (100% covered within AWS Free Tier, scales to absolute zero). |
| **Detection Point** | Delayed batch scanning at Mail Transfer Agent (MTA) level. | **Real-Time (< 200 ms)** directly inside user's active browser Gmail interface. |
| **Maintenance & Scaling**| Requires OS patching, security kernel upgrades, fragile auto-scaling. | **Zero Server Management**: Sub-millisecond instant scaling from 0 to thousands of requests. |
| **Audit & Incident Dispatch**| Fragmented server logs, manual incident triaging. | **Amazon DynamoDB** captures 100% audit traces; **Amazon SNS** dispatches instant alerts (P ≥ 90%). |

---

### 3. Project Objectives

1. **Superior AI Classification Precision:** Train a lightweight binary classifier achieving **Accuracy ≥ 97%** and F1-Score ≥ 0.95, reliably categorizing English and Vietnamese deceptive email content.
2. **100% Serverless Topology:** Zero server maintenance, zero OS patching overhead, and automatic horizontal scaling from zero to thousands of concurrent requests.
3. **Real-time Inference Speed:** Warm start execution latency **under 15 ms**, yielding a total end-to-end browser inspection round-trip **under 200 ms**.
4. **Absolute Cost Optimization ($0.00 TCO):** Engineered entirely within the **AWS Free Tier** limits, incurring zero ongoing operational expenditure during zero-traffic intervals.
5. **Real-time Audit Logging & Rapid Incident Response:** 100% of inspection telemetry captured inside **Amazon DynamoDB** NoSQL database with automated **Amazon SNS** emergency email alerts dispatched when phishing risk exceeds **≥ 90%**.

---

### 4. Technical Solution Architecture

#### Data Flow & Service Interaction Diagram:

![Data Flow & Service Interaction Diagram](/images/architecture/serverless-system-architecture.svg)

#### Core AWS Services Inventory:

| AWS Service | Category | Architectural Role | Selection Rationale |
| :--- | :--- | :--- | :--- |
| **AWS Lambda** | Serverless Compute | Core computational brain executing real-time XGBoost inference. | Instant zero-to-scale capability, 1,000,000 free calls/month, sub-15ms warm latency. |
| **Amazon API Gateway v2** | Managed API Gateway | Public HTTPS ingestion endpoint receiving requests from Chrome Extension. | HTTP API standard provides 60% lower latency, 71% cost savings vs REST APIs, native CORS support. |
| **Amazon ECR** | Container Registry | Houses container images bundling Python 3.12, XGBoost, and model weights. | Supports container images up to 10 GB (shattering the traditional 250 MB zip barrier). |
| **Amazon DynamoDB** | Serverless NoSQL DB | Captures real-time immutable audit trails for every scan event. | Sub-10ms read/write latency at any scale, permanent 25 GB free storage under Free Tier. |
| **Amazon SNS** | Pub/Sub Messaging | Fans out emergency incident notifications via Email protocol. | Decoupled asynchronous delivery; notifies administrators in < 2 seconds without client blocking. |
| **Amazon CloudWatch** | Monitoring & Observability | Centralized telemetry, structured log analysis, memory tracking, and alarms. | Deep native Lambda integration, customized real-time dashboards, and metric-based alarms. |
| **AWS IAM** | Identity & Access Management | Enforces zero-trust role-based access control (RBAC) via Least Privilege. | Guarantees tight security boundaries limited exclusively to target resource ARNs. |

---

### 5. Implementation Timeline

#### 12-Week Implementation Breakdown Matrix:

| Phase | Duration | Core Engineering Objectives | AWS Technologies & Stack | Key Deliverables |
| :--- | :---: | :--- | :--- | :--- |
| **Foundations & ML** | Weeks 1 - 2 | Problem formulation, train XGBoost + TruncatedSVD pipeline, configure root AWS credentials. | Python, Scikit-learn, XGBoost, IAM | Serialized model (`.joblib`), initialized AWS environment. |
| **Packaging & Network**| Weeks 3 - 4 | Build OCI Container image, configure Amazon ECR, provision HTTP API on API Gateway v2. | Docker, Amazon ECR, Amazon API Gateway v2 | Production image in ECR, active endpoint `POST /predict`. |
| **Inference & State** | Weeks 5 - 6 | Deploy Lambda Container, optimize warm latency < 15ms, provision DynamoDB Table & SNS Topic. | AWS Lambda, Amazon DynamoDB, Amazon SNS | Operational Lambda function with automated DynamoDB/SNS flows. |
| **Client & Telemetry** | Weeks 7 - 8 | Develop Manifest V3 Chrome Extension, end-to-end testing in Gmail, establish CloudWatch Alarms. | JavaScript, Chrome Extensions, Amazon CloudWatch | Fully functional Gmail extension, unified observability dashboard. |
| **Enterprise Scale** | Weeks 9 - 12 | Deploy AWS WAF rate limiting, construct S3 Data Lake & Glue catalog, automate MLOps lifecycle. | AWS WAF, Amazon S3, AWS Glue, AWS Step Functions | Complete technical thesis, Well-Architected verified platform. |

---

### 6. Budget Estimation & Free Tier Cost Analysis

Assuming a production volume of **100,000 email scans per month**:

| AWS Service | Monthly Usage Estimate | AWS Free Tier Monthly Allowance | Net Incurred Cost |
| :--- | :--- | :--- | :---: |
| **AWS Lambda** | 100,000 requests × 15 ms × 512 MB | 1,000,000 requests + 3,200,000 GB-seconds/month | **$0.00** |
| **Amazon API Gateway v2** | 100,000 HTTP API calls | 1,000,000 calls/month (First 12 months) | **$0.00** |
| **Amazon ECR** | 1 Repository (Image ~210 MB) | 500 MB private storage/month | **$0.00** |
| **Amazon DynamoDB** | 100,000 writes (~50 MB data), 1 RCU / 1 WCU | 25 GB storage + 25 RCU / 25 WCU permanently | **$0.00** |
| **Amazon SNS** | ~500 emergency notification emails | 1,000 email notifications/month | **$0.00** |
| **Amazon CloudWatch** | 1 Log Group (~100 MB), 4 Metrics, 2 Alarms | 5 GB Log ingestion + 10 Alarms permanently | **$0.00** |
| **TOTAL TCO** | **Continuous 24/7 Protection** | **100% Covered by AWS Free Tier** | **$0.00 USD / month** |

{{% notice tip %}}
Even after scaling to **1,000,000 monthly email scans**, total system operating expenses remain approximately **$1.15 USD / month** (primarily driven by API Gateway request metering at $1.00/million). This offers unprecedented cost-efficiency compared to running a dedicated EC2 instance costing $35 – $50 USD/month.
{{% /notice %}}

---

### 7. Risk Management & Technical Mitigation Strategy

| Identified Risk | Severity | Operational Impact | Mitigation & Engineering Resolution |
| :--- | :---: | :--- | :--- |
| **Cold Start Latency** | Medium | Initial invocation may require > 1 second as the microVM unpacks the ECR image. | • Pre-load all model weights into **Global Scope** to ensure subsequent warm invocations complete in **under 15 ms**.<br>• Set Lambda timeout to 5 seconds to eliminate startup dropouts. |
| **CORS Policy Restrictions** | High | Google Chrome blocks scan results due to strict cross-origin resource sharing policies. | • Enable wildcard (`*`) CORS configuration at the API Gateway HTTP API v2 layer.<br>• Inject explicit `Access-Control-Allow-Origin: *` headers in all Lambda responses. |
| **Zip Package Size Limit** | Critical | Scientific C-extension packages (`xgboost`, `scipy`) exceed Lambda's 250 MB zip ceiling. | • Adopted **OCI Docker Container Images on Amazon ECR**, which natively supports packages up to **10 GB**. |
| **Data Drift (Model Decay)** | Medium | Emerging phishing tactics degrade model classification accuracy over time. | • Schema design preserves full raw email previews in DynamoDB to feed future retraining.<br>• Established MLOps roadmap with AWS Step Functions and SageMaker. |
| **Denial-of-Service (DoS)** | Low | Adversaries flood public endpoints with traffic to deplete resources. | • Outlined perimeter protection via **AWS WAF** with Rate-based Rules (20 req/5 min/IP) in Week 9. |
