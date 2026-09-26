---
title: "Worklog Week 8"
date: 2026-09-21
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Worklog Week 8: Comprehensive Observability with Amazon CloudWatch & Project Synthesis

### 1. General Information & Core Objectives
* **Duration:** September 21, 2026 – September 27, 2026 (Week 8).
* **Location:** AWS Hanoi Office — 7th Floor, Grand Terra Building, 36 Cat Linh, Dong Da, Hanoi.
* **Field Supervisor:** Pham Van Phong / Do Tuan Anh (Solutions Architect).
* **Mentor Lead:** Nguyen Gia Hung (Senior Solutions Architect - AWS Vietnam).
* **Core Technical Objectives:**
  1. Construct an end-to-end **Amazon CloudWatch Observability** framework encompassing structured logs, custom metrics, and operational dashboards across the serverless topology.
  2. Author aggregated telemetry queries within **CloudWatch Logs Insights** to compute P50, P90, and P99 latency percentiles and analyze real memory consumption.
  3. Provision the centralized operations board **CloudWatch Dashboard** `PhishingDetection-Operations-Dashboard` for real-time visualization.
  4. Establish automated **CloudWatch Metric Alarms** tracking failure rates (`Errors > 0`) and anomalous durations (`Duration > 2000 ms`), integrated with Amazon SNS incident queues.
  5. Synthesize the complete 08-week intensive internship deliverables, finalize codebase documentation, produce technical reports, and successfully defend the Capstone project.

---

### 2. Implementation Schedule & Daily Work Breakdown

| Day | Technical Tasks & Objectives | Start Date | End Date | References | Outcomes & Evidence |
| :--- | :--- | :---: | :---: | :--- | :--- |
| **Mon** | • Surveyed Log Group `/aws/lambda/PhishingDetectionXGBoostContainer`.<br>• Configured a 30-day retention policy to prevent compounding storage charges. | 09/21/2026 | 09/21/2026 | [CloudWatch Logs Retention](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Working-with-log-groups-and-streams.html) | Log group successfully capped, eliminating perpetual log storage costs. |
| **Tue** | • Formulated analytical queries within CloudWatch Logs Insights: Extracted `Duration`, `Billed Duration`, and `Max Memory Used`.<br>• Evaluated 100 sample invocations: P50 was 12.1 ms, P95 was 13.8 ms, and peak memory was 154 MB. | 09/22/2026 | 09/22/2026 | [CloudWatch Logs Insights Query Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_AnalyzeLogData-discover-queries.html) | Documented quantitative empirical proof of ultra-low latency serverless execution. |
| **Wed** | • Provisioned CloudWatch Dashboard `PhishingDetection-Operations-Dashboard` via AWS CLI.<br>• Added 4 telemetry graphs: Total Invocations, Average Latency, Error Rates, and Peak Memory Usage. | 09/23/2026 | 09/23/2026 | [Using Amazon CloudWatch Dashboards](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html) | Operational dashboard operational with 1-minute auto-refresh cadence. |
| **Thu** | • Created CloudWatch Alarm `Lambda-Execution-Errors` triggering on `Errors >= 1` over 5 minutes.<br>• Created Alarm `Lambda-High-Latency` triggering on `Duration > 2000 ms`.<br>• Wired alarm state changes to SNS topic `PhishingAlertTopic`. | 09/24/2026 | 09/24/2026 | [Creating CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ConsoleAlarms.html) | Both automated guardrails tested and confirmed, meeting modern Cloud SRE operational criteria. |
| **Fri** | • Standardized codebase documentation, sanitized credentials, and polished architectural topology schematics.<br>• Defended 8-week Capstone deliverables before Mentor Lead Nguyen Gia Hung and Enterprise Mentors. | 09/25/2026 | 09/25/2026 | [AWS Well-Architected Operational Excellence](https://docs.aws.amazon.com/wellarchitected/latest/framework/oe-pillar.html) | Project awarded highest rating across technical architecture, sub-15ms latency, and $0 TCO. |
| **Sat - Sun**| • Completed academic graduation internship documentation and executive presentation slides.<br>• Formulated 4-week future development roadmap (Weeks 9 – 12) for enterprise scaling. | 09/26/2026 | 09/27/2026 | [FCAJ Workforce Bootcamp Guidelines](https://cloudjourney.awsstudygroup.com/) | 08-week intensive on-site internship successfully concluded on schedule. |

---

### 3. Practical Configurations & CloudWatch Logs Insights Queries

#### 3.1. Telemetry Query on CloudWatch Logs Insights:
```sql
fields @timestamp, @duration, @billedDuration, @maxMemoryUsed
| filter @type = "REPORT"
| stats avg(@duration) as AvgDuration,
        pct(@duration, 50) as P50_Latency,
        pct(@duration, 95) as P95_Latency,
        pct(@duration, 99) as P99_Latency,
        max(@maxMemoryUsed) as PeakMemoryMB,
        count(*) as TotalInvocations
  by bin(1h)
```
*Empirical Query Results:*

| Hourly Window (`bin(1h)`) | Avg Duration (`ms`) | P50 Latency (`ms`) | P95 Latency (`ms`) | P99 Latency (`ms`) | Peak Memory | Total Invocations |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **2026-09-22 14:00:00** | 12.35 ms | 12.10 ms | 13.82 ms | 14.90 ms | 154 MB | 100 |

#### 3.2. CloudWatch Dashboard CLI Command:
```bash
aws cloudwatch put-dashboard \
    --dashboard-name "PhishingDetection-Operations-Dashboard" \
    --dashboard-body '{
        "widgets": [
            {
                "type": "metric",
                "width": 12,
                "height": 6,
                "properties": {
                    "title": "Lambda Invocations & Errors",
                    "metrics": [
                        [ "AWS/Lambda", "Invocations", "FunctionName", "PhishingDetectionXGBoostContainer", { "stat": "Sum", "color": "#2ca02c" } ],
                        [ ".", "Errors", ".", ".", { "stat": "Sum", "color": "#d62728" } ]
                    ],
                    "period": 300,
                    "region": "ap-southeast-1"
                }
            },
            {
                "type": "metric",
                "width": 12,
                "height": 6,
                "properties": {
                    "title": "Execution Duration (Latency ms)",
                    "metrics": [
                        [ "AWS/Lambda", "Duration", "FunctionName", "PhishingDetectionXGBoostContainer", { "stat": "Average", "color": "#1f77b4" } ],
                        [ "...", { "stat": "p95", "color": "#ff7f0e" } ]
                    ],
                    "period": 300,
                    "region": "ap-southeast-1"
                }
            }
        ]
    }' \
    --region ap-southeast-1
```

---

### 4. Technical Challenges & Troubleshooting

* **Issue: Compounding CloudWatch log group accumulation posing cost overage risks.**
  * *Symptom:* Default Lambda log groups are configured with a `Never Expire` retention setting, causing cumulative storage to eventually exceed the 5 GB Free Tier allotment.
  * *Root Cause Analysis:* Default AWS behavior prioritizes log durability over storage cost minimization.
  * *Resolution:* Enforced a 30-day automated log expiration rule:
    ```bash
    aws logs put-retention-policy \
        --log-group-name /aws/lambda/PhishingDetectionXGBoostContainer \
        --retention-in-days 30 \
        --region ap-southeast-1
    ```
    Stale log streams are automatically purged, ensuring storage costs remain permanently at $0.00.

---

### 5. Deliverables & Milestones
1. **End-to-End Observability Pipeline:** Fully operational CloudWatch dashboard and metric alarms ensuring continuous health monitoring.
2. **Capstone Defense Passed with Excellence:** Received top evaluation for production-grade serverless implementation.
3. **Comprehensive Technical Documentation:** Fully documented architecture, step-by-step workshop guide, and reproduction scripts.
