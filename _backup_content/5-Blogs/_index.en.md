---
title: "Technical Blogs"
date: 2026-08-28
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Community Technical Publications & Architectural Blogs

During the internship at AWS Vietnam, contributing domain knowledge and sharing empirical engineering insights with the broader cloud community represents a core value. Below is the published architectural deep-dive article:

---

### [5.1. Deploying a 100% Serverless AI Phishing Detection Pipeline on AWS: Sub-15ms Latency and $0 Operating TCO](5.1-serverless-phishing-detection/)
* **Domain:** Serverless Machine Learning, Docker on AWS Lambda, Cold Start Elimination, and Zero-Cost Architecture under AWS Free Tier.
* **Publication Channel:** Featured on the **AWS Study Group (FCJ)** community portal and university research forums.
* **Core Takeaways:**
  * Exhaustive architectural trade-off comparing costly 24/7 EC2 GPU instances / SageMaker real-time endpoints vs. Lightweight XGBoost + SVD inside Lambda Containers.
  * Global Scope pre-loading methodology eliminating container initialization overhead.
  * Provisioned Capacity optimization for permanent DynamoDB Free Tier qualification.
