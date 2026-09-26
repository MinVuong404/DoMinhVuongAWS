---
title: "Worklog Week 4"
date: 2026-08-24
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Worklog Week 4: Machine Learning Containerization with Docker & Amazon ECR

### 1. General Information & Core Objectives
* **Duration:** August 24, 2026 – August 30, 2026 (Week 4).
* **Location:** AWS Hanoi Office — 7th Floor, Grand Terra Building, 36 Cat Linh, Dong Da, Hanoi.
* **Field Supervisor:** Pham Van Phong / Do Tuan Anh (Solutions Architect).
* **Mentor Lead:** Nguyen Gia Hung (Senior Solutions Architect - AWS Vietnam).
* **Core Technical Objectives:**
  1. Package the trio of pre-trained Machine Learning artifacts (`model.joblib`, `tfidf.joblib`, `svd.joblib`) alongside the core runtime handler `lambda_function.py`.
  2. Author an optimized multi-layer **Dockerfile** built on the official AWS base image: `public.ecr.aws/lambda/python:3.12`.
  3. Provision an **Amazon ECR (Elastic Container Registry)** Private Repository titled `phishing-xgboost` within region `ap-southeast-1`.
  4. Perform secure authentication between the local Docker daemon and Amazon ECR using ephemeral authentication tokens generated via `aws ecr get-login-password`.
  5. Build, tag, and publish the production container image to Amazon ECR, validating cryptographic image digest integrity.

---

### 2. Implementation Schedule & Daily Work Breakdown

| Day | Technical Tasks & Objectives | Start Date | End Date | References | Outcomes & Evidence |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **Mon** | • Verified data integrity of the 3 trained model weights: `model.joblib` (XGBoost), `tfidf.joblib` (TF-IDF Vectorizer), and `svd.joblib` (300-component TruncatedSVD).<br>• Measured cumulative footprint: 37.8 MB. | 08/24/2026 | 08/24/2026 | [Joblib Persistence Documentation](https://joblib.readthedocs.io/) | Validated MD5 checksums; assets prepared for container bundling. |
| **Tue** | • Authored trimmed `requirements.txt`: `numpy`, `scipy`, `scikit-learn`, `xgboost`, `joblib`.<br>• Crafted Dockerfile setting `${LAMBDA_TASK_ROOT}` and `CMD ["lambda_function.lambda_handler"]`. | 08/25/2026 | 08/25/2026 | [Creating Lambda container images](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html) | Completed Dockerfile with optimal layer caching for `pip install` commands. |
| **Wed** | • Created private Amazon ECR repository via AWS CLI: `aws ecr create-repository --repository-name phishing-xgboost`.<br>• Enabled Image Scan On Push to enforce continuous CVE vulnerability scanning. | 08/26/2026 | 08/26/2026 | [Amazon ECR Repositories](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Repositories.html)<br>[ECR Image Scanning](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html) | Provisioned repository with URI: `803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost`. |
| **Thu** | • Executed Docker CLI authentication handshake against ECR using `aws ecr get-login-password`.<br>• Built container image locally via `docker build -t phishing-xgboost:latest .`. | 08/27/2026 | 08/27/2026 | [Docker CLI Documentation](https://docs.docker.com/engine/reference/commandline/cli/) | Local image built in 115 seconds, verified operational on Amazon Linux 2023 base. |
| **Fri** | • Tagged image to target ECR repository URI: `docker tag phishing-xgboost:latest 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest`.<br>• Published image to ECR via `docker push`. | 08/28/2026 | 08/28/2026 | [Pushing a Docker image to ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html) | Container successfully uploaded to ECR with compressed size of 210 MB (540 MB uncompressed). |
| **Sat - Sun**| • Evaluated Amazon ECR vulnerability scan findings: 0 Critical, 0 High vulnerabilities.<br>• Reviewed IAM service trust policy requirements for Lambda service principal (`lambda.amazonaws.com`). | 08/29/2026 | 08/30/2026 | [IAM Permissions for Lambda Container Images](https://docs.aws.amazon.com/lambda/latest/dg/images-permissions.html) | Prerequisites met for frictionless Lambda function provisioning. |

---

### 3. Practical Configurations & Docker Source

#### 3.1. Production `Dockerfile` Definition:
```dockerfile
# Official AWS Lambda base image for Python 3.12 (Amazon Linux 2023)
FROM public.ecr.aws/lambda/python:3.12

# Install machine learning inference dependencies
COPY requirements.txt ${LAMBDA_TASK_ROOT}/
RUN pip install --no-cache-dir -r ${LAMBDA_TASK_ROOT}/requirements.txt

# Copy serialized machine learning artifacts into task root
COPY model.joblib tfidf.joblib svd.joblib ${LAMBDA_TASK_ROOT}/

# Copy application runtime logic
COPY lambda_function.py ${LAMBDA_TASK_ROOT}/

# Define execution handler entrypoint
CMD [ "lambda_function.lambda_handler" ]
```

#### 3.2. ECR Provisioning & Build Commands:
```bash
# 1. Provision ECR repository with automated vulnerability scanning
aws ecr create-repository \
    --repository-name phishing-xgboost \
    --image-scanning-configuration scanOnPush=true \
    --region ap-southeast-1

# 2. Authenticate Docker CLI against ECR endpoint
aws ecr get-login-password --region ap-southeast-1 | \
    docker login --username AWS --password-stdin 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com

# 3. Build container image locally
docker build -t phishing-xgboost:latest .

# 4. Tag container image to match ECR registry URI
docker tag phishing-xgboost:latest 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest

# 5. Push container image to Amazon ECR
docker push 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest
```

---

### 4. Technical Challenges & Troubleshooting

* **Issue: CPU architecture mismatch triggering `exec format error` inside Lambda microVM.**
  * *Symptom:* After deploying the function from the ECR image, invocations failed with `fork/exec /var/rapid/init: exec format error` and HTTP 502 Bad Gateway.
  * *Root Cause Analysis:* When building images on an Apple Silicon (ARM64) workstation without explicit platform flags, Docker builds images targeting `linux/arm64`. The Lambda function was initially created with an `x86_64` execution architecture.
  * *Resolution:* Explicitly forced cross-platform emulation during the container build step:
    ```bash
    docker build --platform linux/amd64 -t phishing-xgboost:latest .
    ```
    This ensured strict binary compatibility with Lambda's standard x86_64 Firecracker runtime.

---

### 5. Deliverables & Milestones
1. **Verified ECR Container Image:** `803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest`.
2. **Automated Docker Packaging:** Bundled Python 3.12, C-libraries, and 3 ML weight files within an optimized 210 MB compressed image.
3. **Security Assurance:** 100% clean CVE security scan on Amazon ECR with zero critical/high alerts.
