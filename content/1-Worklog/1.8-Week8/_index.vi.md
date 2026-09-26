---
title: "Nhật ký Tuần 8"
date: 2026-09-21
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

# Nhật ký Tuần 8: Thiết lập hệ thống giám sát với CloudWatch và đóng gói dự án

### 1. Thông tin chung và mục tiêu
* Thời gian thực hiện: Từ 21/09/2026 đến 27/09/2026.
* Địa điểm làm việc: Văn phòng AWS Hà Nội, Tầng 7, Tòa nhà Grand Terra, 36 Cát Linh, Đống Đa, Hà Nội.
* Cán bộ hướng dẫn cơ sở: Phạm Văn Phóng và Đỗ Tuấn Anh, chức vụ Solutions Architect.
* Cán bộ phụ trách đơn vị: Nguyễn Gia Hưng, chức vụ Senior Solutions Architect, AWS Việt Nam.
* Mục tiêu kỹ thuật:
  1. Xây dựng giải pháp giám sát tập trung với Amazon CloudWatch gồm Logs, Metrics và Dashboard cho toàn bộ kiến trúc Serverless.
  2. Viết câu truy vấn thống kê trên CloudWatch Logs Insights để trích xuất các chỉ số P50, P90 và P99 về thời gian thực thi cùng dung lượng bộ nhớ tiêu thụ thực tế.
  3. Tạo bảng điều khiển CloudWatch Dashboard PhishingDetection-Operations-Dashboard trực quan hóa các biểu đồ vận hành.
  4. Cấu hình cảnh báo CloudWatch Metric Alarm giám sát số lượng lỗi Errors lớn hơn 0 và độ trễ Duration vượt quá 2000 ms, kết nối với Amazon SNS để gửi thông báo.
  5. Đóng gói mã nguồn, viết tài liệu kỹ thuật và hoàn thiện báo cáo giai đoạn 8 tuần thực tập.

---

### 2. Kế hoạch triển khai và nhật ký công việc chi tiết

| Thứ | Nội dung công việc và mục tiêu kỹ thuật | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo | Kết quả và ghi nhận thực tế |
| :--- | :--- | :---: | :---: | :--- | :--- |
| Thứ 2 | Khảo sát nhóm nhật ký /aws/lambda/PhishingDetectionXGBoostContainer. Thiết lập thời hạn lưu trữ nhật ký Retention Period ở mức 30 ngày để tối ưu chi phí lưu trữ CloudWatch. | 21/09/2026 | 21/09/2026 | [CloudWatch Logs Retention](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Working-with-log-groups-and-streams.html) | Nhóm nhật ký được áp dụng chính sách giới hạn thời gian lưu trữ tự động. |
| Thứ 3 | Soạn thảo truy vấn phân tích trên CloudWatch Logs Insights trích xuất các chỉ số Duration, Billed Duration và Max Memory Used. Phân tích dữ liệu 100 lần gọi: P50 đạt 12.1 ms, P95 đạt 13.8 ms, dung lượng bộ nhớ sử dụng tối đa là 154 MB trên mức cấp phát 512 MB. | 22/09/2026 | 22/09/2026 | [CloudWatch Logs Insights Query Syntax](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CWL_AnalyzeLogData-discover-queries.html) | Thu thập đầy đủ số liệu đo lường hiệu năng thực tế của hệ thống. |
| Thứ 4 | Khởi tạo bảng điều khiển CloudWatch Dashboard PhishingDetection-Operations-Dashboard bằng AWS CLI. Thêm 4 biểu đồ gồm Tổng số lượt gọi Invocations, Độ trễ trung bình Average Latency, Số lượng lỗi Error Count và Bộ nhớ tiêu thụ Max Memory Used. | 23/09/2026 | 23/09/2026 | [Using Amazon CloudWatch Dashboards](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/CloudWatch_Dashboards.html) | Bảng điều khiển giám sát hiển thị dữ liệu trực quan theo chu kỳ cập nhật 1 phút. |
| Thứ 5 | Tạo cảnh báo CloudWatch Alarm Lambda-Execution-Errors kích hoạt khi Errors từ 1 trở lên trong 5 phút. Tạo cảnh báo Lambda-High-Latency kích hoạt khi Duration vượt quá 2000 ms. Kết nối hành động cảnh báo tới SNS Topic PhishingAlertTopic. | 24/09/2026 | 24/09/2026 | [Creating CloudWatch Alarms](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/ConsoleAlarms.html) | Hai quy tắc cảnh báo hoạt động bình thường trên hệ thống. |
| Thứ 6 | Chuẩn hóa tài liệu mã nguồn, tệp README, hoàn thiện sơ đồ luồng dữ liệu. Báo cáo tổng kết giai đoạn 8 tuần với cán bộ phụ trách đơn vị và cán bộ hướng dẫn cơ sở. | 25/09/2026 | 25/09/2026 | [AWS Well-Architected Operational Excellence](https://docs.aws.amazon.com/wellarchitected/latest/framework/oe-pillar.html) | Báo cáo nghiệm thu giai đoạn 8 tuần được cán bộ hướng dẫn thông qua. |
| Thứ 7 và Chủ nhật | Hoàn thiện bản báo cáo thực tập và tài liệu trình chiếu. Xây dựng kế hoạch công tác mở rộng từ tuần 9 đến tuần 12. | 26/09/2026 | 27/09/2026 | [FCAJ Workforce Bootcamp Guidelines](https://cloudjourney.awsstudygroup.com/) | Hoàn thành kế hoạch thực tập 8 tuần theo đúng tiến độ. |

---

### 3. Thao tác kỹ thuật và truy vấn CloudWatch Logs Insights

#### 3.1. Cú pháp truy vấn trên CloudWatch Logs Insights
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
Kết quả trích xuất số liệu thực tế:

| Khung thời gian | Thời gian trung bình | P50 Độ trễ | P95 Độ trễ | P99 Độ trễ | Đỉnh RAM tiêu thụ | Tổng lượt gọi |
| :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| 2026-09-22 14:00:00 | 12.35 ms | 12.10 ms | 13.82 ms | 14.90 ms | 154 MB | 100 |

#### 3.2. Lệnh tạo CloudWatch Dashboard bằng AWS CLI
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

### 4. Vấn đề kỹ thuật và cách thức xử lý

* Sự cố: Dung lượng lưu trữ nhóm nhật ký CloudWatch Log Groups tích lũy gây nguy cơ phát sinh chi phí khi vận hành lâu dài.
  * Hiện tượng: Mặc định khi tạo Lambda, nhóm nhật ký /aws/lambda/PhishingDetectionXGBoostContainer được áp dụng chính sách lưu trữ không bao giờ hết hạn Never Expire. Khi hệ thống xử lý lượng lớn yêu cầu qua thời gian, dung lượng nhật ký có thể vượt mức 5 GB miễn phí của Free Tier.
  * Nguyên nhân: Thiết lập mặc định của AWS ưu tiên giữ trọn vẹn dữ liệu nhật ký nhưng chưa tối ưu về mặt chi phí lưu trữ cho môi trường thử nghiệm.
  * Giải pháp xử lý: Áp dụng lệnh cấu hình giới hạn thời gian lưu trữ thành 30 ngày:
    ```bash
    aws logs put-retention-policy \
        --log-group-name /aws/lambda/PhishingDetectionXGBoostContainer \
        --retention-in-days 30 \
        --region ap-southeast-1
    ```
    Dữ liệu nhật ký quá 30 ngày được hệ thống tự động xóa bỏ, duy trì tài nguyên nằm trong hạn mức miễn phí.

---

### 5. Kết quả hoàn thành

1. Thiết lập bảng điều khiển CloudWatch Dashboard và 2 quy tắc Metric Alarms phục vụ giám sát hệ thống.
2. Hoàn thành nội dung báo cáo thực tập và sản phẩm thực tế theo tiến độ 8 tuần thực tập tập trung.
3. Tổng hợp mã nguồn, Dockerfile và tài liệu hướng dẫn kỹ thuật của dự án.
