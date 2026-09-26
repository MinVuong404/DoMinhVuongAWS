---
title: "Giám sát CloudWatch"
date: 2026-08-25
weight: 7
chapter: false
pre: " <b> 4.7. </b> "
---

# 4.7. Giám sát vận hành và cảnh báo sự cố với Amazon CloudWatch

### 4.7.1. Phân tích nhật ký và truy vấn trên CloudWatch Logs Insights

Mỗi lượt thực thi hàm Lambda đều ghi nhận cấu trúc nhật ký chi tiết về Amazon CloudWatch Logs tại nhóm nhật ký /aws/lambda/PhishingDetectionXGBoostContainer.

Dòng nhật ký tiêu chuẩn của một lần thực thi:
```text
REPORT RequestId: a4f89d12-612b-4e01-9a73-8cb23e41b0fa
Duration: 13.12 ms
Billed Duration: 14 ms
Memory Size: 512 MB
Max Memory Used: 154 MB
```

Truy vấn thống kê hiệu năng trên CloudWatch Logs Insights:
Thực hiện truy vấn trong giao diện CloudWatch Logs Insights để trích xuất các chỉ số phân vị về độ trễ và dung lượng bộ nhớ tiêu thụ:

```sql
fields @timestamp, @duration, @billedDuration, @maxMemoryUsed
| filter @type = "REPORT"
| stats avg(@duration) as ThoiGianTB_ms,
        pct(@duration, 50) as P50_DoTre_ms,
        pct(@duration, 90) as P90_DoTre_ms,
        pct(@duration, 99) as P99_DoTre_ms,
        max(@maxMemoryUsed) as DinhRAM_MB,
        count(*) as TongSoRequest
  by bin(5m)
```

Kết quả truy vấn thực tế:

| Khoảng thời gian | Thời gian trung bình | P50 Độ trễ | P90 Độ trễ | P99 Độ trễ | Đỉnh RAM tiêu thụ | Tổng số yêu cầu |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 2026-08-25 14:30:00 | 12.85 ms | 12.10 ms | 13.95 ms | 14.80 ms | 154 MB | 20 |

---

### 4.7.2. Khởi tạo bảng điều khiển vận hành tập trung

Để theo dõi các thông số vận hành của hệ thống, quản trị viên khởi tạo bảng điều khiển CloudWatch Dashboard mang tên PhishingDetection-Operations-Dashboard.

Tạo Dashboard với 4 biểu đồ giám sát bằng lệnh AWS CLI:
```bash
aws cloudwatch put-dashboard \
    --dashboard-name "PhishingDetection-Operations-Dashboard" \
    --dashboard-body '{
        "widgets": [
            {
                "type": "metric",
                "x": 0, "y": 0, "width": 12, "height": 6,
                "properties": {
                    "title": "Tổng số lượt gọi Invocations và lỗi Errors",
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
                    "title": "Thời gian thực thi suy luận Duration ms",
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
                    "title": "Bộ nhớ RAM tiêu thụ MaxMemoryUsed MB",
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
                    "title": "Tần suất giới hạn tài nguyên Throttles",
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

Bảng điều khiển được khởi tạo và sẵn sàng hiển thị trên giao diện CloudWatch Management Console.

---

### 4.7.3. Thiết lập cảnh báo tự động với CloudWatch Metric Alarms

Thiết lập 2 quy tắc cảnh báo bằng các lệnh:

#### 1. Cảnh báo khi có lỗi phát sinh trong quá trình thực thi
```bash
aws cloudwatch put-metric-alarm \
    --alarm-name "Lambda-Execution-Errors-Alarm" \
    --alarm-description "Cảnh báo khi hàm Lambda phát sinh lỗi xử lý" \
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

#### 2. Cảnh báo khi thời gian thực thi vượt quá 2000 ms
```bash
aws cloudwatch put-metric-alarm \
    --alarm-name "Lambda-High-Duration-Alarm" \
    --alarm-description "Cảnh báo khi độ trễ suy luận của hàm vượt quá 2 giây" \
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

Hệ thống giám sát và cảnh báo tự động hoàn tất thiết lập, hỗ trợ theo dõi chỉ số vận hành và gửi thông báo khi có lỗi phát sinh.
