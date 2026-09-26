---
title: "Worklog Week 2"
date: 2026-08-10
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
---

# Worklog Week 2: AWS Core Services Survey & Serverless Computing Deep Dive

### 1. General Information & Core Objectives
* **Duration:** August 10, 2026 – August 16, 2026 (Week 2).
* **Location:** AWS Hanoi Office — 7th Floor, Grand Terra Building, 36 Cat Linh, Dong Da, Hanoi.
* **Field Supervisor:** Pham Van Phong / Do Tuan Anh (Solutions Architect).
* **Mentor Lead:** Nguyen Gia Hung (Senior Solutions Architect - AWS Vietnam).
* **Core Technical Objectives:**
  1. Survey and benchmark 8 foundational AWS services: **Amazon EC2, Amazon S3, AWS IAM, AWS Lambda, Amazon API Gateway, Amazon DynamoDB, Amazon SNS, and Amazon CloudWatch**.
  2. Construct a rigorous architectural trade-off matrix comparing traditional virtual machines (**EC2 + RDS/PostgreSQL**) against a pure serverless topology (**Serverless: Lambda + DynamoDB**).
  3. Perform a Total Cost of Ownership (TCO) analysis under the **AWS Free Tier**: Prove that an enterprise-grade phishing detection platform can operate at $0.00 idle cost for small-to-medium throughput.
  4. Investigate AWS Lambda resource constraints (6MB synchronous payload limit, 512MB-10GB ephemeral storage `/tmp`, 250MB uncompressed zip limit vs. 10GB container image limit via Amazon ECR).
  5. Configure a clean Python 3.12 development baseline and validate initial natural language preprocessing pipelines.

---

### 2. Implementation Schedule & Daily Work Breakdown

| Day | Technical Tasks & Objectives | Start Date | End Date | References | Outcomes & Evidence |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **Mon** | • Researched Amazon EC2 virtualization architectures: Nitro Enclaves, instance classes (T4g Graviton, C7g, M7g).<br>• Provisioned a secured S3 Bucket featuring SSE-S3 default encryption and comprehensive Block Public Access. | 08/10/2026 | 08/10/2026 | [Amazon EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)<br>[Amazon S3 Security Best Practices](https://docs.aws.amazon.com/AmazonS3/latest/userguide/security-best-practices.html) | Provisioned project documentation storage bucket with zero public exposure risks. |
| **Tue** | • Explored AWS Lambda execution lifecycle: Firecracker microVMs, Execution Environments, Cold vs Warm starts.<br>• Authored a test Lambda handler logging raw event schemas and context structures. | 08/11/2026 | 08/11/2026 | [AWS Lambda Execution Environment](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-context.html)<br>[Firecracker MicroVM Whitepaper](https://firecracker-microvm.github.io/) | Mastered container global scope reuse, reducing warm invocation latency from 1200ms to <15ms. |
| **Wed** | • Attempted building a conventional Lambda zip package containing `scikit-learn`, `scipy`, and `xgboost`.<br>• Encountered package size wall: Uncompressed bundle exceeded 380 MB, violating the 250 MB zip limit. | 08/12/2026 | 08/12/2026 | [Lambda Deployment Packages](https://docs.aws.amazon.com/lambda/latest/dg/gettingstarted-package.html)<br>[AWS Lambda Container Image Support](https://aws.amazon.com/blogs/aws/new-for-aws-lambda-container-image-support/) | Decided to adopt OCI Docker container images supported up to 10 GB via Amazon ECR. |
| **Thu** | • Developed TCO model: EC2 `t3.medium` running 24/7 costs ~$30-$40/month, whereas Lambda + DynamoDB incurs zero charges under 1,000,000 requests/month.<br>• Reviewed findings with mentor. | 08/13/2026 | 08/13/2026 | [AWS Pricing Calculator](https://calculator.aws/)<br>[AWS Free Tier Terms](https://aws.amazon.com/free/) | Architectural decision ratified: 100% Serverless architecture selected for maximum cost-performance efficiency. |
| **Fri** | • Evaluated open-source phishing email corpora (SpamAssassin and Enron Phishing datasets).<br>• Trained baseline binary classifier (TF-IDF + 300-dimension TruncatedSVD + XGBoost) on local development workstation. | 08/14/2026 | 08/14/2026 | [Scikit-learn Documentation](https://scikit-learn.org/)<br>[XGBoost Python API](https://xgboost.readthedocs.io/) | Model achieved 97.4% test accuracy with 0.96 F1-score; total serialized weight size was compact (~38 MB). |
| **Sat - Sun**| • Studied Docker packaging techniques for Python Lambda runtimes utilizing `public.ecr.aws/lambda/python:3.12`.<br>• Authored regex cleaning utilities to strip HTML noise, normalize URLs (`tagurl`), and tag email tokens (`tagemail`). | 08/15/2026 | 08/16/2026 | [AWS Lambda Base Images for Docker](https://gallery.ecr.aws/lambda/python)<br>[Python Regex Module Docs](https://docs.python.org/3/library/re.html) | Built reusable preprocessing pipeline and validated feature extraction speed. |

---

### 3. Architecture Comparison Matrix

| Evaluation Criteria | Virtual Machine Architecture (EC2 + RDS) | Serverless Architecture (Lambda + DynamoDB) | Project Advantage |
| :--- | :--- | :--- | :--- |
| **Idle Cost** | Flat 24/7 server rental fee (~$35 – $50 USD/month) regardless of incoming traffic. | **$0.00 USD/month** (Billed exclusively per millisecond of compute, zero cost during idle). | 100% budget savings during testing and low-traffic phases. |
| **Elastic Scalability** | Requires Auto Scaling Groups; instance launch latency takes 2 to 5 minutes. | **Instantaneous scaling** from zero to hundreds of concurrent executions in milliseconds. | Effortlessly absorbs email batch scanning spikes without latency spikes. |
| **Operational Overhead** | Manual OS patching, kernel security updates, firewall rules, and SSH key management. | **Zero Maintenance**: Fully managed runtime environment by AWS hypervisor. | Engineering focus stays entirely on ML model accuracy and business logic. |
| **Deployment Package Limit** | Constrained only by attached EBS volume size. | **Up to 10 GB** when utilizing OCI Docker container images on Amazon ECR. | Comfortably accommodates heavy C-extension ML packages (`xgboost`, `scipy`). |

---

### 4. Technical Challenges & Troubleshooting

* **Issue: `InvalidParameterValueException: Unzipped size must be smaller than 262144000 bytes` on Lambda upload.**
  * *Symptom:* Uploading a zipped `site-packages` artifact (82 MB compressed) resulted in an immediate rejection by AWS Lambda due to an uncompressed footprint of 312 MB (> 250 MB).
  * *Root Cause Analysis:* AWS Lambda enforces a strict 250 MB ceiling on direct zip deployments. Scientific Python libraries compile heavy binary C/Fortran extensions that unpack to substantial disk footprints.
  * *Resolution:* Migrated deployment architecture to **OCI Container Images (Docker)**. AWS Lambda supports container images up to **10 GB** fetched from **Amazon ECR**, eliminating the 250 MB barrier entirely.

---

### 5. Deliverables & Milestones
1. **Architectural Feasibility Analysis:** Validated that Serverless is the superior choice for high-burst, cost-sensitive classification workloads.
2. **Production-Ready ML Pipeline:** Trained XGBoost classifier achieving 97.4% accuracy with lightweight serialized artifacts (`model.joblib`, `tfidf.joblib`, `svd.joblib` totaling ~38 MB).
3. **Data Preprocessing Utility:** Production-grade text normalization module handling HTML stripping and regex token tagging.
