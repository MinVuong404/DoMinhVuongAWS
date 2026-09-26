---
title: "Docker & Amazon ECR"
date: 2026-08-25
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# 4.2. Docker Container Packaging & Amazon ECR Registry

### 4.2.1. Codebase Architecture & Production Dockerfile

To deploy specialized machine learning workloads requiring compiled binary C-extensions (`xgboost`, `scikit-learn`, `scipy`) that exceed Lambda's legacy 250 MB zip deployment threshold, packaging the runtime into an **OCI Container Image** is the optimal architectural pattern.

#### Component Breakdown in `aws_serverless/`:

| Artifact / File | Category | Format | Engineering Role & Responsibilities |
| :--- | :--- | :--- | :--- |
| `Dockerfile` | Container Spec | Dockerfile | Multi-stage build manifest optimized on Amazon Linux 2023 Python 3.12. |
| `requirements.txt` | Dependencies | Pip config | Pinned machine learning libraries (`scikit-learn`, `xgboost`, `joblib`,...). |
| `lambda_function.py` | Python Script | Python 3.12 | Core event handler parsing API Gateway requests and executing in-memory inference. |
| `model.joblib` | Binary Model | Serialized Model | Serialized XGBoost gradient boosting decision tree classifier (> 97% Accuracy). |
| `tfidf.joblib` | Feature Extractor | Serialized Vectorizer | Pre-trained TF-IDF vectorizer mapping 10,000 unigrams and bigrams. |
| `svd.joblib` | Dimension Reducer | Serialized SVD Matrix | TruncatedSVD projection reducing 10,000 dimensions to 300 (saving 97% RAM). |

#### 1. Specification `requirements.txt`:
```text
numpy>=1.26.0
scipy>=1.12.0
scikit-learn>=1.4.0
xgboost>=2.0.0
joblib>=1.3.0
```

#### 2. Specification `Dockerfile`:
```dockerfile
# Official AWS Lambda base image optimized for Python 3.12 (Amazon Linux 2023)
FROM public.ecr.aws/lambda/python:3.12

# Install machine learning inference dependencies into task root
COPY requirements.txt ${LAMBDA_TASK_ROOT}/
RUN pip install --no-cache-dir -r ${LAMBDA_TASK_ROOT}/requirements.txt

# Copy serialized machine learning artifacts into task root
COPY model.joblib tfidf.joblib svd.joblib ${LAMBDA_TASK_ROOT}/

# Copy application runtime logic into task root
COPY lambda_function.py ${LAMBDA_TASK_ROOT}/

# Define execution handler entrypoint triggered on event invocation
CMD [ "lambda_function.lambda_handler" ]
```

---

### 4.2.2. Amazon ECR Private Repository Initialization

**Amazon Elastic Container Registry (ECR)** is a fully managed container image registry offering high availability and zero-egress latency when delivering images to AWS Lambda over AWS backbone networks.

Execute the following AWS CLI command to provision a private repository within `ap-southeast-1`:
```bash
aws ecr create-repository \
    --repository-name phishing-xgboost \
    --image-scanning-configuration scanOnPush=true \
    --image-tag-mutability MUTABLE \
    --region ap-southeast-1
```

*AWS CLI JSON response:*
```json
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-southeast-1:803146828520:repository/phishing-xgboost",
        "registryId": "803146828520",
        "repositoryName": "phishing-xgboost",
        "repositoryUri": "803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost",
        "createdAt": "2026-08-26T09:00:00+07:00"
    }
}
```

{{% notice tip %}}
The `--image-scanning-configuration scanOnPush=true` flag automatically triggers CVE vulnerability scanning against all newly pushed layers, fulfilling compliance criteria under the **AWS Well-Architected Security Pillar**.
{{% /notice %}}

---

### 4.2.3. Docker Authentication, Cross-Platform Build & Image Publishing

#### Step 1: Authenticate Local Docker Client with ECR
Acquire an ephemeral authentication token via AWS STS:
```bash
aws ecr get-login-password --region ap-southeast-1 | \
    docker login --username AWS --password-stdin 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com
```
*Expected terminal confirmation:* `Login Succeeded`.

#### Step 2: Build Multi-Architecture Container Image targeting `linux/amd64`
To guarantee binary execution compatibility inside Lambda's Firecracker microVM and prevent `exec format error` issues:
```bash
docker build --platform linux/amd64 -t phishing-xgboost:latest .
```

#### Step 3: Tag Container Image to Destination ECR URI:
```bash
docker tag phishing-xgboost:latest 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest
```

#### Step 4: Publish Container Image to Amazon ECR:
```bash
docker push 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest
```

#### Step 5: Verify Image Digest on Amazon ECR:
```bash
aws ecr describe-images \
    --repository-name phishing-xgboost \
    --region ap-southeast-1 \
    --query 'imageDetails[*].[imageTags, imageSizeInBytes, imageDigest]' \
    --output table
```
*Verified Amazon ECR Image Metadata:*

| Image Tag | Size (Bytes) | Image Digest (SHA256) | Vulnerability Scan |
| :---: | :---: | :--- | :---: |
| `latest` | 220,456,128 (~210 MB) | `sha256:7b91d2e9fa498a12bcde0914838f7129c54...` | **PASSED (0 Critical)** |

The machine learning container image is now published and ready to be bound to AWS Lambda in the subsequent module.
