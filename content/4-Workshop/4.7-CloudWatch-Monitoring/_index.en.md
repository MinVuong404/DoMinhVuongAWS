---
title: "CloudWatch Telemetry"
date: 2026-08-25
weight: 7
chapter: false
pre: " <b> 4.7. </b> "
---

# 4.7. Operational Telemetry & Alerting with Amazon CloudWatch

### 4.7.1. Log Stream Diagnostics & CloudWatch Logs Insights

Every Lambda invocation automatically records structured diagnostics into **Amazon CloudWatch Logs** under Log Group: `/aws/lambda/PhishingDetectionXGBoostContainer`.

#### Standard Execution Diagnostic Line (`REPORT`):
```text
REPORT RequestId: a4f89d12-612b-4e01-9a73-8cb23e41b0fa
Duration: 13.12 ms
Billed Duration: 14 ms
Memory Size: 512 MB
Max Memory Used: 154 MB
```

#### Analytical Query on CloudWatch Logs Insights:
Execute the following query inside the **CloudWatch → Logs Insights** console to compute latency percentiles and memory usage trends:

```sql
fields @timestamp, @duration, @billedDuration, @maxMemoryUsed
| filter @type = "REPORT"
| stats avg(@duration) as AvgDuration_ms,
        pct(@duration, 50) as P50_Latency_ms,
        pct(@duration, 90) as P90_Latency_ms,
        pct(@duration, 99) as P99_Latency_ms,
        max(@maxMemoryUsed) as PeakRAM_MB,
        count(*) as TotalRequests
  by bin(5m)
```

*Empirical Query Results:*

| Aggregation Window (`bin(5m)`) | Avg Duration (`ms`) | P50 Latency (`ms`) | P90 Latency (`ms`) | P99 Latency (`ms`) | Peak RAM Footprint | Total Requests |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **2026-08-25 14:30:00** | 12.85 ms | 12.10 ms | 13.95 ms | 14.80 ms | 154 MB | 20 |

---

### 4.7.2. Centralized Operations Dashboard Setup

To provide operational visibility across the serverless topology, we provision **CloudWatch Dashboard** `PhishingDetection-Operations-Dashboard`.

Execute the following AWS CLI command to provision 4 visual widgets:
```bash
aws cloudwatch put-dashboard \
    --dashboard-name "PhishingDetection-Operations-Dashboard" \
    --dashboard-body '{
        "widgets": [
            {
                "type": "metric",
                "x": 0, "y": 0, "width": 12, "height": 6,
                "properties": {
                    "title": "API Gateway Invocations & Errors",
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
                "x": 12, "y": 0, "width": 12, "height": 6,
                "properties": {
                    "title": "Inference Latency Duration (ms)",
                    "metrics": [
                        [ "AWS/Lambda", "Duration", "FunctionName", "PhishingDetectionXGBoostContainer", { "stat": "Average", "color": "#1f77b4" } ],
                        [ "...", { "stat": "p95", "color": "#ff7f0e" } ]
                    ],
                    "period": 300,
                    "region": "ap-southeast-1"
                }
            },
            {
                "type": "metric",
                "x": 0, "y": 6, "width": 12, "height": 6,
                "properties": {
                    "title": "Peak Memory Consumed (MB)",
                    "metrics": [
                        [ "AWS/Lambda", "MaxMemoryUsed", "FunctionName", "PhishingDetectionXGBoostContainer", { "stat": "Maximum", "color": "#9467bd" } ]
                    ],
                    "period": 300,
                    "region": "ap-southeast-1"
                }
            },
            {
                "type": "metric",
                "x": 12, "y": 6, "width": 12, "height": 6,
                "properties": {
                    "title": "Concurrency Throttling Events",
                    "metrics": [
                        [ "AWS/Lambda", "Throttles", "FunctionName", "PhishingDetectionXGBoostContainer", { "stat": "Sum", "color": "#8c564b" } ]
                    ],
                    "period": 300,
                    "region": "ap-southeast-1"
                }
            }
        ]
    }' \
    --region ap-southeast-1
```

*Verification:* The operational dashboard is visible within the AWS CloudWatch Management Console.

---

### 4.7.3. Automated CloudWatch Metric Alarms

To maintain production service level agreements (SLAs), configure 2 automated metric alarms:

#### 1. Invocation Errors Alarm (`Errors >= 1`):
```bash
aws cloudwatch put-metric-alarm \
    --alarm-name "Lambda-Execution-Errors-Alarm" \
    --alarm-description "Triggers when Lambda function encounters unhandled exceptions" \
    --metric-name Errors \
    --namespace AWS/Lambda \
    --statistic Sum \
    --dimensions Name=FunctionName,Value=PhishingDetectionXGBoostContainer \
    --period 300 \
    --threshold 1 \
    --comparison-operator GreaterThanOrEqualToThreshold \
    --evaluation-periods 1 \
    --alarm-actions arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic \
    --region ap-southeast-1
```

#### 2. Anomalous Duration Latency Alarm (`Duration > 2000 ms`):
```bash
aws cloudwatch put-metric-alarm \
    --alarm-name "Lambda-High-Duration-Alarm" \
    --alarm-description "Triggers when compute duration exceeds 2000 ms" \
    --metric-name Duration \
    --namespace AWS/Lambda \
    --statistic Average \
    --dimensions Name=FunctionName,Value=PhishingDetectionXGBoostContainer \
    --period 300 \
    --threshold 2000 \
    --comparison-operator GreaterThanThreshold \
    --evaluation-periods 1 \
    --alarm-actions arn:aws:sns:ap-southeast-1:803146828520:PhishingAlertTopic \
    --region ap-southeast-1
```

The monitoring and alerting infrastructure is fully established, providing automated failure notifications directly to system administrators.
