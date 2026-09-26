---
title: "5.1. Blog: Serverless Phishing AI"
date: 2026-08-28
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Deploying a 100% Serverless AI Phishing Detection Pipeline on AWS: Sub-15ms Latency and $0 Operating TCO

*Author: Do Minh Vuong — Cloud Solutions Architect Intern (AWS Vietnam / FCAJ 2026)*  
*Categories: Serverless Computing, Machine Learning on Cloud, AWS Well-Architected Framework*

---

### 1. The Enterprise AI Cost Paradox for Small-to-Medium Businesses

When engineering teams evaluate deploying Machine Learning (ML) workloads into cloud production environments, the conventional reflex is to reach for **Amazon SageMaker Real-Time Inference Endpoints** or long-running **Amazon EC2 Virtual Machines (e.g., `g4dn.xlarge` GPU instances or `c6i.xlarge` compute clusters)**.

However, for small-to-medium enterprises (SMEs) or internal enterprise security utilities:
* A baseline SageMaker Endpoint instance (`ml.t3.medium`) running 24/7 accrues **$45 – $60 USD / month**.
* A baseline EC2 `t3.medium` instance backed by general-purpose EBS storage costs **$35 – $45 USD / month**.
* **The Paradox Emerges:** Employees only interact with their email inboxes during working business hours (8 hours/day). During nights and weekends, the infrastructure sits completely idle, yet the business continues to pay 100% of the provisioned server rental rate!

The central architectural challenge: **How do we engineer an AI email security pipeline delivering >97% accuracy, responding in milliseconds (<20 ms), while incurring an exact idle operational cost of zero dollars ($0.00)?**

The definitive architectural solution is: **A 100% Serverless Architecture on AWS**.

---

### 2. Model Architecture: Lean Engineering vs. Monolithic Bloat

Rather than deploying multi-billion parameter Generative Large Language Models (LLMs) requiring tens of gigabytes of expensive GPU VRAM, real-time email security triage is fundamentally a **Binary Text Classification** task.

We engineered a 3-stage model compression pipeline:
1. **Feature Extraction via TF-IDF:** Tokenized a vocabulary of 10,000 n-grams capturing deceptive intent markers (*"urgent", "account locked", "verify otp", "wire transfer", "lottery claim"*).
2. **Latent Semantic Compression with TruncatedSVD:** Projected high-dimensional sparse representations into a dense **300-dimensional Latent Semantic Analysis (LSA)** space. This eliminated noise, preserved 92.4% of total variance, and slashed feature array dimensions by >95%.
3. **Inference with XGBoost Classifier:** Evaluated gradient-boosted decision trees optimized for rapid probability calculation.

**The Engineering Milestone:** The cumulative size of the 3 serialized weight files (`model.joblib`, `tfidf.joblib`, `svd.joblib`) was compressed to just **37.8 MB**, consuming less than **150 MB of memory** during active execution while achieving **97.4% accuracy** on independent validation benchmarks.

---

### 3. Shattering Lambda's 250 MB Zip Limit with OCI Docker Containers

Historically, AWS Lambda enforced a strict **250 MB uncompressed limit** on uploaded zip deployment archives. Packaging heavy compiled scientific C-libraries (`scipy`, `scikit-learn`, `xgboost`) routinely expands `site-packages` past 350 MB, causing immediate `InvalidParameterValueException` deployment failures.

**The Solution:** Transition to **AWS Lambda Container Image Support** backed by **Amazon ECR**.
* Official base image: `public.ecr.aws/lambda/python:3.12` built on Amazon Linux 2023.
* AWS Lambda natively supports container images up to **10 GB**, completely eliminating library size friction.

```dockerfile
FROM public.ecr.aws/lambda/python:3.12
COPY requirements.txt ${LAMBDA_TASK_ROOT}/
RUN pip install --no-cache-dir -r ${LAMBDA_TASK_ROOT}/requirements.txt
COPY model.joblib tfidf.joblib svd.joblib ${LAMBDA_TASK_ROOT}/
COPY lambda_function.py ${LAMBDA_TASK_ROOT}/
CMD [ "lambda_function.lambda_handler" ]
```

---

### 4. Global Scope Pre-loading: The Secret to Sub-15ms Latency

The primary caveat of containerized Lambda functions is **Cold Start Latency** (requiring 1.2 to 1.8 seconds on initial launch while the microVM pulls and unpacks the container image). If model deserialization (`joblib.load()`) occurs inside the invocation handler, every subsequent execution suffers an additional 800 ms penalty.

**Architectural Technique:** Deserialize all models and instantiate SDK clients in the **Global Execution Scope** outside `lambda_handler`:

```python
# GLOBAL SCOPE INITIALIZATION: EXECUTED ONCE PER CONTAINER LIFECYCLE
dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')
table = dynamodb.Table('PhishingDetectionLogs')

TASK_ROOT = os.environ.get('LAMBDA_TASK_ROOT', '.')
MODEL = joblib.load(os.path.join(TASK_ROOT, 'model.joblib'))
TFIDF = joblib.load(os.path.join(TASK_ROOT, 'tfidf.joblib'))
SVD = joblib.load(os.path.join(TASK_ROOT, 'svd.joblib'))

def lambda_handler(event, context):
    # WARM INVOCATIONS REUSE CACHED MEMORY FOR PURE IN-MEMORY TENSOR MATH
    # EXECUTION LATENCY DROPS TO JUST 11 - 13 MS!
    cleaned_text = clean_text(event['body'])
    vector = SVD.transform(TFIDF.transform([cleaned_text]))
    prob = MODEL.predict_proba(vector)[0][1]
    ...
```

Thanks to AWS Lambda's execution environment reuse, for 99% of user requests, the container remains warm, delivering mathematical inference in just **12 milliseconds**!

---

### 5. Architectural TCO Analysis: Absolute Cost Optimization

Below is the verified monthly cost comparison modeling **100,000 email scans per month**:

| Architecture Topology | Infrastructure Profile | Idle Server Hosting Cost | Incurred Cost for 100k Invocations | Total Monthly TCO |
| :--- | :--- | :---: | :---: | :---: |
| **Amazon SageMaker Endpoint** | 1 instance `ml.t3.medium` | $50.40 | $0.00 | **$50.40 USD** |
| **Amazon EC2 Virtual Machine** | 1 instance `t3.medium` + 30GB EBS | $33.58 | $0.00 | **$33.58 USD** |
| **Serverless Architecture (Project)** | **Lambda 512MB + API Gateway + DynamoDB** | **$0.00** | **$0.00 (AWS Free Tier Covered)** | **$0.00 USD** |

> [!NOTE]
> Even under a 10x traffic surge (1,000,000 requests/month), the serverless topology costs only **$1.15 USD / month**, delivering over **97% financial savings** compared to traditional dedicated computing instances.

---

### 6. Conclusion

This project demonstrates that **modern cloud engineering is not merely about hosting virtual machines in the cloud, but about architectural discipline and cost efficiency**. Combining lightweight machine learning models with AWS serverless primitives empowers engineers to build ultra-responsive, enterprise-resilient, and cost-optimized solutions.
