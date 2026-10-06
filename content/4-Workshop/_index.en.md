---
title: "Workshop"
date: 2026-08-25
weight: 4
chapter: false
pre: " <b> 4. </b> "
aliases:
  - /5-workshop/
---

# Hands-on Technical Workshop: Building & Deploying a 100% Serverless Phishing Detection System on AWS

#### Workshop Overview & Technical Objectives
This workshop provides an exhaustive, end-to-end hands-on laboratory guiding practitioners through architectural design, containerizing machine learning models with Docker, publishing container images to **Amazon ECR**, provisioning serverless compute on **AWS Lambda**, exposing ingestion interfaces via **Amazon API Gateway v2**, persisting real-time audit trails into **Amazon DynamoDB**, fanning out incident notifications via **Amazon SNS**, embedding automated inspection within **Google Chrome Extension for Gmail**, and establishing comprehensive telemetry using **Amazon CloudWatch**.

{{% notice info %}}
* **Project Name:** AWS Serverless Real-time Phishing Detection & Alert System
* **Codebase Repository:** [https://github.com/MinVuong404/AWS-Serverless-Phishing-Detection](https://github.com/MinVuong404/AWS-Serverless-Phishing-Detection)
* **Architectural Paradigm:** 100% Serverless Architecture (Amazon ECR, AWS Lambda, Amazon API Gateway v2, Amazon DynamoDB, Amazon SNS, Amazon CloudWatch, AWS IAM) integrated with a Client Google Chrome Extension (Manifest V3).
* **Deployment Region:** `ap-southeast-1` (Singapore) — minimizing latency for Asia-Pacific traffic (< 35 ms).
* **Operational TCO:** **$0.00 USD / month** (100% covered by the AWS Free Tier lifetime and 12-month tier).
{{% /notice %}}

---

### End-to-End System Architecture Diagram

![End-to-End System Architecture](/images/architecture/serverless-system-architecture.svg)

---

#### Hands-on Modules Curriculum:

1. [**4.1. Architecture Overview & Environment Preparation**](4.1-architecture-environment/)
   * 4.1.1. Data flow analysis & Machine Learning specification (XGBoost + SVD)
   * 4.1.2. Developer toolchain validation (AWS CLI v2, Docker Desktop, Python 3.12)
   * 4.1.3. IAM Execution Role configuration (`PhishingDetectionLambdaRole`) via Least Privilege
2. [**4.2. Docker Container Packaging & Amazon ECR Registry**](4.2-docker-ecr/)
   * 4.2.1. Codebase layout & Multi-layer Dockerfile creation on Amazon Linux 2023
   * 4.2.2. Amazon ECR Private Repository initialization (`phishing-xgboost`)
   * 4.2.3. Docker authentication, Cross-platform build, and Image publishing to ECR
3. [**4.3. AWS Lambda Function Deployment from Container Image**](4.3-lambda-deployment/)
   * 4.3.1. Lambda provisioning (`PhishingDetectionXGBoostContainer`) from ECR Image URI
   * 4.3.2. Compute resource allocation (512 MB RAM, 5s Timeout) & Environment Variables
   * 4.3.3. Cold Start mitigation & Global Scope model deserialization optimization
4. [**4.4. Amazon API Gateway v2 Ingestion Ingress Setup**](4.4-api-gateway-v2/)
   * 4.4.1. HTTP API v2 instantiation & `POST /predict` Route configuration
   * 4.4.2. Lambda Proxy Integration wiring & `lambda:InvokeFunction` permission grants
   * 4.4.3. Full-suite Cross-Origin Resource Sharing (CORS) preflight enforcement
5. [**4.5. Amazon DynamoDB Audit Persistence & Amazon SNS Alerting**](4.5-dynamodb-sns/)
   * 4.5.1. DynamoDB NoSQL table initialization (`PhishingDetectionLogs`) under Free Tier limits
   * 4.5.2. Amazon SNS Topic configuration (`PhishingAlertTopic`) & Email subscriber verification
   * 4.5.3. Granular IAM Least Privilege permission attachments (`dynamodb:PutItem`, `sns:Publish`)
6. [**4.6. Chrome Extension Client & Live Gmail End-to-End Validation**](4.6-chrome-extension-testing/)
   * 4.6.1. Chrome Manifest V3 structure & Dynamic Gmail DOM parsing via `MutationObserver`
   * 4.6.2. Local extension deployment via Google Chrome Developer Mode
   * 4.6.3. Empirical test suite execution across 20 safe and malicious email scenarios
7. [**4.7. Operational Telemetry & Alerting with Amazon CloudWatch**](4.7-cloudwatch-monitoring/)
   * 4.7.1. Log Stream inspection & Performance queries via CloudWatch Logs Insights
   * 4.7.2. Centralized operations board setup (`PhishingDetection-Operations-Dashboard`)
   * 4.7.3. Metric Alarms configuration tracking execution failures and abnormal latency
8. [**4.8. Resource Clean-up Guide**](4.8-resource-cleanup/)
   * 4.8.1. Visual tear-down guide via AWS Management Console
   * 4.8.2. Automated resource teardown script via AWS CLI
