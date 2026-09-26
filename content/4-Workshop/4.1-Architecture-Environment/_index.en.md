---
title: "Architecture & Environment"
date: 2026-08-25
weight: 1
chapter: false
pre: " <b> 4.1. </b> "
---

# 4.1. Architecture Overview & Environment Preparation

### 4.1.1. Data Flow Analysis & Machine Learning Model Specification

The phishing detection system is engineered to adhere strictly to a **100% Serverless** paradigm and **Zero-Trust** security architecture on AWS. The comprehensive data lifecycle operates across 5 seamlessly integrated phases:

![Data Flow Analysis & Machine Learning Inference Pipeline](/images/architecture/phishing-detection-dataflow-en.svg)

#### Natural Language Processing & Inference Pipeline:
1. **Regex Normalization:** Extracts clean text from raw email payloads, strips script/style markup, replaces hyperlinks with token `tagurl`, normalizes email addresses with `tagemail`, and removes erratic punctuation.
2. **TF-IDF Transformation:** Vectorizes sanitized strings across a vocabulary size of 10,000 unigrams and bigrams.
3. **TruncatedSVD Dimensionality Reduction:** Projects high-dimensional sparse representations into a dense **300-dimensional Latent Semantic Analysis (LSA)** space, capturing >92% explained variance while slashing inference RAM usage by 97%.
4. **XGBoost Inference:** Generates binary probability estimates ($P \in [0.0, 1.0]$). Payloads with $P \ge 0.5$ are labeled as `Phishing`; otherwise categorized as `Safe`.
5. **Persistence & Alerting:** Writes structured audit records into DynamoDB and publishes emergency SNS broadcasts when $P \ge 0.90$.

---

### 4.1.2. Local Toolchain & Environment Standardization

Before initiating cloud deployments, verify and standardize local developer workstation dependencies:

#### 1. Validate AWS CLI v2 Installation:
```bash
aws --version
```
*Required output:* `aws-cli/2.x.x` or higher.

#### 2. Configure AWS CLI Authentication:
```bash
aws configure set default.region ap-southeast-1
aws configure set default.output json

# Verify caller identity
aws sts get-caller-identity
```
*Expected JSON output:*
```json
{
    "UserId": "AIDA4TESTVUONGFCAJ01",
    "Account": "803146828520",
    "Arn": "arn:aws:iam::803146828520:user/dev-vuong"
}
```

#### 3. Verify Docker Engine:
```bash
docker --version
docker ps
```
*Requirement:* Docker daemon is operational in the background.

#### 4. Verify Python 3.12:
```bash
python --version
```
*Requirement:* Python `3.12.x` matching the official Lambda container runtime.

---

### 4.1.3. IAM Execution Role Setup via Least Privilege

AWS Lambda requires an execution identity (**IAM Role**) authorizing CloudWatch log ingestion, DynamoDB item insertion, and SNS message publishing.

#### Step 1: Create Trust Policy Allowing Lambda Service Principal
Save `trust-policy.json` locally:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "lambda.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

#### Step 2: Provision IAM Role via AWS CLI:
```bash
aws iam create-role \
    --role-name PhishingDetectionLambdaRole \
    --assume-role-policy-document file://trust-policy.json \
    --description "IAM Execution Role for Serverless Phishing Detection Lambda Function"
```

#### Step 3: Attach Basic Execution Policy for CloudWatch Logging:
```bash
aws iam attach-role-policy \
    --role-name PhishingDetectionLambdaRole \
    --policy-arn arn:aws:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole
```

#### Step 4: Retrieve IAM Role ARN:
```bash
aws iam get-role --role-name PhishingDetectionLambdaRole --query 'Role.Arn' --output text
```
*Record ARN:* `arn:aws:iam::803146828520:role/PhishingDetectionLambdaRole` (required in module 4.3).

{{% notice warning %}}
Adhering to **Least Privilege**, only basic logging privileges are granted at this phase. Granular DynamoDB and SNS access permissions will be scoped to exact resource ARNs in [Hands-on Module 4.5](4.5-dynamodb-sns/) once those resources are created.
{{% /notice %}}
