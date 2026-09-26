---
title: "6.2. Day 1: Govern"
date: 2026-08-21
weight: 2
chapter: false
pre: " <b> 6.2. </b> "
---

# 6.2. Day 1: Govern — Quản Trị Danh Tính & Ranh Giới Bảo Mật

### 1. Bối cảnh tình huống thử thách (Scenario Problem)
Hệ thống giả lập của một tập đoàn tài chính bị cảnh báo bảo mật:
* Một máy chủ EC2 đặt tại Public Subnet bị gán nhầm IAM Instance Profile có quyền quản trị tối cao `AdministratorAccess`.
* Tồn tại 3 IAM Users chưa được kích hoạt MFA và có các Access Key đã phát hành hơn 180 ngày không được đổi mới (Key Rotation).
* Nguy cơ: Kẻ tấn công có thể khai thác lỗ hổng SSRF (Server-Side Request Forgery) trên máy chủ web để đánh cắp metadata token (`http://169.254.169.254/latest/meta-data/iam/security-credentials/`) và chiếm toàn quyền kiểm soát tài khoản AWS.

---

### 2. Kế hoạch giải quyết & Thao tác kỹ thuật

#### Bước 1: Rà soát và cô lập IAM Role bị gán quyền thừa
Sử dụng AWS CLI truy vấn danh sách policy đính kèm vào IAM Role của máy chủ:
```bash
aws iam list-attached-role-policies --role-name CompromisedWebServerRole
```
*Phát hiện:* Policy `arn:aws:iam::aws:policy/AdministratorAccess` đang được gán trực tiếp.

Thực hiện tách quyền quản trị và gán quyền tối thiểu chỉ cho phép đọc dữ liệu tĩnh trên S3:
```bash
# 1. Tách quyền AdministratorAccess
aws iam detach-role-policy \
    --role-name CompromisedWebServerRole \
    --policy-arn arn:aws:iam::aws:policy/AdministratorAccess

# 2. Gán chính sách giới hạn quyền đọc S3
aws iam attach-role-policy \
    --role-name CompromisedWebServerRole \
    --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

#### Bước 2: Bật giao thức IMDSv2 để phòng ngừa tấn công SSRF
Cưỡng chế máy chủ EC2 chuyển sang sử dụng Instance Metadata Service Version 2 (IMDSv2) yêu cầu token phiên làm việc:
```bash
aws ec2 modify-instance-metadata-options \
    --instance-id i-0123456789abcdef0 \
    --http-tokens required \
    --http-endpoint enabled
```

#### Bước 3: Thu hồi và vô hiệu hóa các Access Keys quá hạn
Liệt kê và vô hiệu hóa các Access Keys đã tạo hơn 90 ngày:
```bash
aws iam update-access-key \
    --user-name finance-analyst-01 \
    --access-key-id AKIAIOSFODNN7EXAMPLE \
    --status Inactive
```

---

### 3. Kết quả đạt được & Bài học chuyên môn
* **Điểm số thử thách:** Hoàn thành 100% mục tiêu của Day 1 trong 35 phút (vượt thời gian quy định 60 phút).
* **Bài học kinh nghiệm:** Luôn kích hoạt bắt buộc **IMDSv2** trên mọi máy chủ EC2 và duy trì quy trình kiểm toán tự động IAM Access Analyzer để đảm bảo tuân thủ nghiêm ngặt nguyên tắc Least Privilege.
