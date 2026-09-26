---
title: "6.3. Day 2: Investigate"
date: 2026-08-22
weight: 3
chapter: false
pre: " <b> 6.3. </b> "
---

# 6.3. Day 2: Investigate — Điều Tra Dấu Vết Sự Cố Đám Mây

### 1. Bối cảnh tình huống thử thách (Scenario Problem)
Đội ngũ giám sát phát hiện chi phí truyền dữ liệu ra Internet (Data Transfer Out) tăng vọt bất thường trong 2 giờ qua:
* Một lượng lớn lưu lượng mạng xuất phát từ VPC nội bộ được truyền tới một địa chỉ IP công khai lạ (`198.51.100.45`).
* Nhiệm vụ:
  1. Xác định chính xác máy chủ nội bộ nào đang phát sinh luồng dữ liệu độc hại.
  2. Bóc tách dấu vết xâm nhập ban đầu qua nhật ký **AWS CloudTrail**.
  3. Áp dụng cơ chế cô lập an ninh mạng (**Network Isolation & Quarantine**) để ngăn chặn rò rỉ dữ liệu mà không làm mất chứng cứ pháp chứng số.

---

### 2. Kế hoạch giải quyết & Thao tác kỹ thuật

#### Bước 1: Phân tích VPC Flow Logs bằng Amazon Athena
Truy vấn nhật ký lưu lượng mạng thông qua bảng Athena để tìm kiếm giao tiếp với IP độc hại:
```sql
SELECT srcaddr, dstaddr, srcport, dstport, protocol, bytes, action
FROM "vpc_flow_logs_db"."flow_logs"
WHERE dstaddr = '198.51.100.45'
ORDER BY bytes DESC
LIMIT 10;
```
*Kết quả điều tra:* Phát hiện địa chỉ IP nội bộ `10.0.2.145` (tương ứng với EC2 instance `i-0987654321fedcba0`) đã truyền đi hơn 4.2 GB dữ liệu qua cổng 4444 (TCP Reverse Shell).

#### Bước 2: Truy vết sự kiện xâm nhập qua AWS CloudTrail
Truy vấn sự kiện API trong CloudTrail để xác định danh tính và thời điểm kẻ tấn công thực thi lệnh:
```bash
aws cloudtrail lookup-events \
    --lookup-attributes AttributeKey=ResourceName,AttributeValue=i-0987654321fedcba0 \
    --query 'Events[*].[EventTime, EventName, Username, SourceIPAddress]' \
    --output table
```
*Kết quả:* Phát hiện sự kiện `AuthorizeSecurityGroupIngress` mở cổng 22 (SSH) ra toàn cầu `0.0.0.0/0` từ IP lạ cách đó 3 giờ.

#### Bước 3: Cô lập máy chủ bằng Quarantine Security Group
Thay vì tắt nguồn máy chủ làm mất dữ liệu trong RAM, tiến hành gán một Security Group cô lập triệt để mọi luồng ra/vào (Zero Inbound/Outbound Rules):
```bash
# 1. Tạo Quarantine Security Group rỗng
QUARANTINE_SG=$(aws ec2 create-security-group \
    --group-name ForensicQuarantineSG \
    --description "Isolate compromised instance for digital forensics" \
    --vpc-id vpc-0a1b2c3d4e5f6g7h8 \
    --output text --query 'GroupId')

# 2. Xóa bỏ luật outbound mặc định
aws ec2 revoke-security-group-egress \
    --group-id $QUARANTINE_SG \
    --protocol -1 \
    --cidr 0.0.0.0/0

# 3. Gán Security Group cô lập vào máy chủ bị xâm nhập
aws ec2 modify-instance-attribute \
    --instance-id i-0987654321fedcba0 \
    --groups $QUARANTINE_SG
```

---

### 3. Kết quả đạt được & Bài học chuyên môn
* **Điểm số thử thách:** Hoàn thành xuất sắc 100% mục tiêu điều tra và cô lập chỉ trong 40 phút.
* **Bài học kinh nghiệm:** Nắm vững quy trình ứng cứu sự cố bảo mật (Incident Response): Giữ nguyên hiện trạng bộ nhớ RAM để phục vụ trích xuất forensic artifact trước khi đưa ra quyết định tiêu hủy tài nguyên.
