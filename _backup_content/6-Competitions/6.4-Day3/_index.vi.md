---
title: "6.4. Day 3: Prove It Arena"
date: 2026-08-23
weight: 4
chapter: false
pre: " <b> 6.4. </b> "
---

# 6.4. Day 3: Prove It Arena — Đấu Trường Trực Tiếp & Phản Ứng Khẩn Cấp

### 1. Bối cảnh tình huống chung kết (Grand Finale Arena)
Vòng chung kết **Prove It Arena** đưa thí sinh vào một kịch bản khủng hoảng hạ tầng nghiêm trọng theo thời gian thực:
* Hệ thống thanh toán trực tuyến của ngân hàng đang hứng chịu một đợt tấn công từ chối dịch vụ phân tán (**DDoS Layer 7**) với hơn 50.000 requests/giây từ mạng botnet.
* Tầng API Gateway bị nghẽn (HTTP 429 Too Many Requests), các hàm Lambda bị cạn kiệt hạn mức đồng thời (Concurrency Exhaustion), và cơ sở dữ liệu bị thắt cổ chai IOPS.
* **Mục tiêu:** Trong vòng 90 phút, thí sinh phải khôi phục dịch vụ trở lại trạng thái `Healthy`, chặn đứng luồng tấn công độc hại và duy trì độ trễ dịch vụ cho khách hàng hợp lệ dưới 100 ms.

---

### 2. Kế hoạch giải quyết & Thao tác khắc phục khủng hoảng

```
[ Lưu lượng tấn công Botnet 50k req/s ]
                  │
                  ▼
  [ 1. Kích hoạt AWS WAF Shield ] ──► (Chặn 98% IP Botnet rác bằng Rate-limiting)
                  │ (Lưu lượng hợp lệ 500 req/s)
                  ▼
   [ 2. API Gateway HTTP API ]
                  │
                  ▼
      [ 3. AWS Lambda Compute ] ◄── (Cấu hình Reserved Concurrency = 500)
                  │
                  ▼
  [ 4. Amazon DynamoDB Database ] ◄── (Chuyển chế độ sang On-Demand Capacity Mode)
```

#### Bước 1: Kích hoạt AWS WAF và Thiết lập Rate-limiting khẩn cấp
Tạo Web ACL với luật Rate-based Rule chặn mọi IP gửi quá 100 requests trong vòng 5 phút:
```bash
aws wafv2 create-web-acl \
    --name EmergencyDDoSMitigationACL \
    --scope REGIONAL \
    --default-action Allow={} \
    --rules '[{
        "Name": "RateLimitRule",
        "Priority": 1,
        "Statement": {
            "RateBasedStatement": {
                "Limit": 100,
                "AggregateKeyType": "IP"
            }
        },
        "Action": { "Block": {} },
        "VisibilityConfig": {
            "SampledRequestsEnabled": true,
            "CloudWatchMetricsEnabled": true,
            "MetricName": "RateLimitRuleMetric"
        }
    }]' \
    --visibility-config SampledRequestsEnabled=true,CloudWatchMetricsEnabled=true,MetricName=WebACLMetric \
    --region ap-southeast-1
```
*Kết quả:* Trong vòng 2 phút sau khi gắn WAF, 98.5% lượng request rác bị chặn đứng ngay tại biên mạng (Edge), giải phóng hoàn toàn gánh nặng tính toán cho Lambda.

#### Bước 2: Phân bổ Reserved Concurrency bảo vệ Lambda
Để ngăn chặn tình trạng một hàm bị tấn công làm cạn kiệt hạn mức Concurrency của toàn bộ tài khoản AWS:
```bash
aws lambda put-function-concurrency \
    --function-name PhishingDetectionXGBoostContainer \
    --reserved-concurrent-executions 500 \
    --region ap-southeast-1
```

#### Bước 3: Chuyển đổi DynamoDB sang chế độ On-Demand Capacity
Chuyển đổi bảng CSDL NoSQL từ Provisioned sang On-Demand để tự động mở rộng thông lượng tức thì:
```bash
aws dynamodb update-table \
    --table-name PhishingDetectionLogs \
    --billing-mode PAY_PER_REQUEST \
    --region ap-southeast-1
```

---

### 3. Kết quả đạt được & Bài học chuyên môn
* **Chỉ số phục hồi hệ thống:**
  * Tỷ lệ lỗi (Error Rate) giảm từ 89.4% xuống **0.02%**.
  * Độ trễ trung bình của người dùng hợp lệ giảm từ 4800 ms xuống **14.2 ms**.
  * Hoàn thành toàn bộ kịch bản phục hồi trong 52 phút (thời gian cho phép là 90 phút).
* **Bài học kinh nghiệm:** Trong thiết kế kiến trúc đám mây hiện đại, an ninh bảo mật nhiều lớp (**Defense-in-Depth**) với sự kết hợp của AWS WAF ở tầng biên và cơ chế co giãn tự động ở tầng dữ liệu là chìa khóa duy nhất để đảm bảo tính sẵn sàng cao (High Availability).
