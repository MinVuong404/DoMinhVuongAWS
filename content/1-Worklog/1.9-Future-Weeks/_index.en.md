---
title: "Weeks 9 - 12 Roadmap"
date: 2026-09-28
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

# Enterprise-Grade System Expansion & Final Review Roadmap (Weeks 9 – 12)

To fulfill the comprehensive 12-week academic evaluation criteria and establish a clear engineering roadmap transitioning the phishing detection platform from a working prototype to an **Enterprise-Grade** solution, the following 4-week advanced sprint plan (Weeks 9 – 12) is established:

---

### High-Level 4-Week Sprint Progression

#### 4-Week Advanced Roadmap Matrix:

| Week | Date Interval | Core Architectural Focus | Key AWS Stack | Expected Technical Deliverable |
| :---: | :---: | :--- | :--- | :--- |
| **Week 9** | 28/09 – 04/10/2026 | **Perimeter Security & Rate Limiting** | AWS WAF, API Gateway v2 | 20 req/5m throttling rules, zero botnet impact. |
| **Week 10** | 05/10 – 11/10/2026 | **Enterprise Data Lake & SQL Analytics** | Amazon S3, DynamoDB Streams, AWS Glue, Athena | Snappy-compressed Parquet lake with serverless Athena queries. |
| **Week 11** | 12/10 – 18/10/2026 | **Automated MLOps Training Lifecycle** | AWS Step Functions, Amazon SageMaker, EventBridge | Triggered continuous training upon detecting data drift. |
| **Week 12** | 19/10 – 25/10/2026 | **Well-Architected Assessment & Defense** | AWS Well-Architected Tool, CloudWatch | Comprehensive 6-Pillars WAR scorecard & thesis readiness. |

---

### Detailed Weekly Objectives & Technical Work Breakdown:

#### 1. Week 9 (28/09/2026 – 04/10/2026): Perimeter Security with AWS WAF & Rate Limiting
* **Context & Problem:** Public API endpoints face Denial-of-Service (DoS) attacks, brute-force probes, and resource exhaustion threats from malicious web crawlers.
* **Technical Objectives:**
  * Deploy **AWS WAF (Web Application Firewall)** directly upstream of Amazon API Gateway v2.
  * Implement an automated **Rate-based Rule**: Enforce a strict ceiling of 20 requests per 5 minutes per client IP address.
  * Attach AWS Managed Rule Groups: Core Rule Set (CRS), SQL Database protection, and Amazon IP Reputation List.
* **Expected Outcome:** 100% mitigation against automated bot traffic and denial-of-wallet exploitation.

#### 2. Week 10 (05/10/2026 – 11/10/2026): Data Lake Ingestion with Amazon S3 & AWS Glue
* **Context & Problem:** DynamoDB is optimized for operational hot storage (30-90 days). Preserving millions of historical email records for compliance and retraining requires cost-effective object storage.
* **Technical Objectives:**
  * Enable **DynamoDB Streams** to capture data change events in real time.
  * Leverage **Amazon Kinesis Data Firehose** to buffer and flush audit events to an **Amazon S3 Document Lake**.
  * Convert incoming JSON records into **Apache Parquet** columnar format with Snappy compression, saving up to 80% on storage.
  * Register schemas in **AWS Glue Data Catalog** and execute ad-hoc serverless SQL queries using **Amazon Athena**.
* **Expected Outcome:** Robust, scalable analytical data lake operating at pennies per month on S3 Standard and S3 Glacier lifecycle tiers.

#### 3. Week 11 (12/10/2026 – 18/10/2026): Automated MLOps Pipeline with AWS Step Functions & SageMaker
* **Context & Problem:** Phishing tactics undergo continuous evolution (Data Drift). Machine learning classifiers require continuous retraining to sustain peak precision.
* **Technical Objectives:**
  * Build an event-driven workflow using **AWS Step Functions (State Machine)** triggered monthly or upon accumulating 50,000 newly validated samples in S3.
  * Dispatch an **Amazon SageMaker Training Job** utilizing EC2 Spot GPU instances (saving 70% in compute cost) to retrain the XGBoost model.
  * Model evaluation gateway: If validation F1-score surpasses the production model, trigger **AWS CodeBuild** to recompile the Docker image and push to Amazon ECR.
  * Seamlessly update Lambda runtime code to the new image digest via `aws lambda update-function-code`.
* **Expected Outcome:** Fully autonomous Continuous Training and Continuous Deployment (CT/CD) MLOps loop.

#### 4. Week 12 (19/10/2026 – 25/10/2026): AWS Well-Architected Review & Graduation Defense
* **Context & Problem:** Rigorous multi-dimensional audit of the end-to-end cloud architecture prior to enterprise handover and final graduation thesis defense.
* **Technical Objectives:**
  * Conduct a formal **AWS Well-Architected Review (WAR)** across all 6 core pillars:
    1. *Operational Excellence*
    2. *Security*
    3. *Reliability*
    4. *Performance Efficiency*
    5. *Cost Optimization*
    6. *Sustainability (Carbon footprint reduction via Serverless)*
  * Remediate High Risk Issues (HRI) and document architectural trade-offs.
  * Finalize all academic deliverables, mentor evaluations, and slide presentations for the graduation thesis committee at HUCE.
* **Expected Outcome:** Certified, enterprise-grade cloud architecture recognized with highest honors.
