---
title: "Worklog Week 1"
date: 2026-08-03
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
---

# Worklog Week 1: Environment Setup, Identity Governance with IAM & AWS Cloud Foundations

### 1. General Information & Core Objectives
* **Duration:** August 3, 2026 – August 9, 2026 (Week 1).
* **Location:** AWS Hanoi Office — 7th Floor, Grand Terra Building, 36 Cat Linh, Dong Da, Hanoi.
* **Field Supervisor:** Pham Van Phong / Do Tuan Anh (Solutions Architect).
* **Mentor Lead:** Nguyen Gia Hung (Senior Solutions Architect - AWS Vietnam).
* **Core Technical Objectives:**
  1. Set up a secure AWS practice account adhering strictly to the **AWS Well-Architected Security Pillar**: Enable Multi-Factor Authentication (MFA) on the root account and eliminate root access keys completely.
  2. Implement an enterprise-grade identity governance architecture using **AWS IAM (Identity and Access Management)** following the *Principle of Least Privilege*.
  3. Install and standardize the **AWS CLI v2** development environment on the local workstation, configuring default region `ap-southeast-1` (Singapore).
  4. Establish automated cost guardrails: Configure **AWS Budgets** and **CloudWatch Billing Alarms** integrated with **Amazon SNS** for instant email notification whenever estimated charges exceed $5 USD.
  5. Gain deep mastery over the **Shared Responsibility Model** and the core pillars of the AWS Well-Architected Framework.

---

### 2. Implementation Schedule & Daily Work Breakdown

| Day | Technical Tasks & Objectives | Start Date | End Date | References | Outcomes & Evidence |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **Mon** | • Onboarded at AWS Hanoi Office.<br>• Initialized AWS practice account for First Cloud AI Journey (FCAJ).<br>• Enabled Virtual MFA for Root User via Google Authenticator.<br>• Verified zero active Root Access Keys. | 08/03/2026 | 08/03/2026 | [Cloud Journey Training Portal](https://cloudjourney.awsstudygroup.com/)<br>[AWS Account Setup & Root MFA Best Practices](https://docs.aws.amazon.com/accounts/latest/reference/manage-acct-root-user.html) | Account successfully secured according to Level 1 of CIS AWS Foundations Benchmark. |
| **Tue** | • Installed AWS CLI v2 on local workstation.<br>• Provisioned IAM development user `dev-vuong`, attached to IAM group `CloudArchitects`.<br>• Configured local credential profiles in `~/.aws/credentials` and `~/.aws/config`. | 08/04/2026 | 08/04/2026 | [AWS CLI v2 Installation Guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)<br>[IAM Best Practices & Least Privilege](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) | Verified identity with `aws sts get-caller-identity`, confirming region `ap-southeast-1`. |
| **Wed** | • Activated *Receive CloudWatch Billing Alerts* in Billing Preferences.<br>• Configured AWS Budget with a strict $10/month threshold.<br>• Created CloudWatch Metric Alarm tracking `EstimatedCharges` at $5.0 USD threshold in `us-east-1`. | 08/05/2026 | 08/05/2026 | [AWS Budgets Documentation](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)<br>[CloudWatch Billing Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/monitor_estimated_charges_with_cloudwatch.html) | Configured Amazon SNS topic `Billing-Alerts-Topic` with confirmed email subscription to `vuong.dmforwork@gmail.com`. |
| **Thu** | • Studied AWS Global Infrastructure: Regions, Availability Zones (AZs), and Edge Locations.<br>• Evaluated network Round-Trip Time (RTT) from Vietnam to neighboring regions (`ap-southeast-1`, `ap-east-1`).<br>• Reviewed *AWS Well-Architected Framework: Security & Cost Pillars*. | 08/06/2026 | 08/06/2026 | [AWS Global Infrastructure](https://aws.amazon.com/about-aws/global-infrastructure/)<br>[AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) | Selected Singapore (`ap-southeast-1`) as the primary region offering minimal latency (~32 ms). |
| **Fri** | • Participated in technical orientation meeting with Mentor Lead and Senior Solutions Architects.<br>• Presented Capstone Proposal: *100% Serverless Phishing Email Detection & Alert System on AWS*.<br>• Received feedback on containerizing ML inference workloads for AWS Lambda execution. | 08/07/2026 | 08/07/2026 | [Serverless Machine Learning on AWS](https://aws.amazon.com/blogs/compute/)<br>[Working with Lambda container images](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html) | Topic officially approved; finalized high-level design combining ECR, Lambda Container, API Gateway v2, DynamoDB, and SNS. |
| **Sat - Sun**| • Completed foundational knowledge quizzes on Cloud Journey training portal.<br>• Analyzed IAM policy evaluation logic, resource policies, and assume-role flows. | 08/08/2026 | 08/09/2026 | [AWS Skill Builder Training](https://explore.skillbuilder.aws/)<br>[IAM Policies Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html) | Achieved 100% score on the internship baseline assessment. |

---

### 3. Practical CLI Configurations & Hands-on Lab

#### 3.1. AWS CLI v2 Environment Standardization:
```bash
# Set default region and structured output format
aws configure set default.region ap-southeast-1
aws configure set default.output json

# Verify caller identity and execution permissions
aws sts get-caller-identity
```
*Output JSON response:*
```json
{
    "UserId": "AIDA4TESTVUONGFCAJ01",
    "Account": "803146828520",
    "Arn": "arn:aws:iam::803146828520:user/dev-vuong"
}
```

#### 3.2. IAM User & Group Setup via Least Privilege:
```bash
# 1. Create development architects group
aws iam create-group --group-name CloudArchitects

# 2. Attach required development permissions (avoiding full AdministratorAccess)
aws iam attach-group-policy --group-name CloudArchitects \
    --policy-arn arn:aws:iam::aws:policy/PowerUserAccess

# 3. Create user and assign to group
aws iam create-user --user-name dev-vuong
aws iam add-user-to-group --user-name dev-vuong --group-name CloudArchitects
```

#### 3.3. Automated Cost Monitoring with CloudWatch & SNS:
```bash
# Create SNS notification topic (global billing metrics reside in us-east-1)
aws sns create-topic --name Billing-Alerts-Topic --region us-east-1

# Subscribe email endpoint
aws sns subscribe \
    --topic-arn arn:aws:sns:us-east-1:803146828520:Billing-Alerts-Topic \
    --protocol email \
    --notification-endpoint vuong.dmforwork@gmail.com \
    --region us-east-1

# Create CloudWatch metric alarm tracking estimated charges > $5 USD
aws cloudwatch put-metric-alarm \
    --alarm-name "Billing-Threshold-Over-5USD" \
    --metric-name EstimatedCharges \
    --namespace AWS/Billing \
    --statistic Maximum \
    --period 21600 \
    --threshold 5.0 \
    --comparison-operator GreaterThanThreshold \
    --dimensions Name=Currency,Value=USD \
    --evaluation-periods 1 \
    --alarm-actions arn:aws:sns:us-east-1:803146828520:Billing-Alerts-Topic \
    --region us-east-1
```

---

### 4. Technical Challenges & Troubleshooting

* **Issue 1: `SignatureDoesNotMatch` on local AWS CLI executions.**
  * *Symptom:* Executing `aws ec2 describe-regions` failed with: `An error occurred (SignatureDoesNotMatch) when calling the GetCallerIdentity operation: Signature expired: 20260804T071520Z is now earlier than 20260804T072045Z`.
  * *Root Cause Analysis:* AWS Signature Version 4 drops requests where the client timestamp drifts by more than 15 minutes compared to AWS NTP authoritative servers to prevent replay attacks. Workstation local clock had drifted by > 5 minutes.
  * *Resolution:* Resynchronized the Windows Time Service via elevated PowerShell:
    ```powershell
    net start w32time
    w32tm /resync /force
    ```
    Following resynchronization, CLI commands executed with 100% consistency.

* **Issue 2: Missing `EstimatedCharges` metric in CloudWatch.**
  * *Symptom:* CLI alarm creation failed due to absent metric in namespace `AWS/Billing`.
  * *Root Cause:* Billing metric ingestion is disabled by default. Furthermore, global billing metrics are aggregated exclusively in `us-east-1` (N. Virginia), not in `ap-southeast-1`.
  * *Resolution:* Logged in via Root User → Enabled `Receive CloudWatch Billing Alerts` under Billing Preferences → Specified `--region us-east-1` in the CLI alarm creation command. Metric appeared within 15 minutes with alarm transitioning to `OK`.

---

### 5. Deliverables & Milestones
1. **Fully Secured AWS Practice Account:** 100% compliant with CIS Foundations Benchmark (Root MFA enabled, no root access keys, bounded IAM user `dev-vuong`).
2. **Automated Cost Guardrails:** Budget ceiling in place with zero risk of unexpected overages.
3. **Standardized Tooling:** AWS CLI v2, Python 3.12, and Docker ready for containerization workflows.
4. **Capstone Architecture Approval:** Formal validation of the *AWS Serverless Phishing Detection* solution design.
