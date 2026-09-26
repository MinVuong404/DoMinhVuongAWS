---
title: "Nhật ký Tuần 4"
date: 2026-08-24
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

# Nhật ký Tuần 4: Đóng gói mô hình học máy bằng Docker Container và Amazon ECR

### 1. Thông tin chung và mục tiêu
* Thời gian thực hiện: Từ 24/08/2026 đến 30/08/2026.
* Địa điểm làm việc: Văn phòng AWS Hà Nội, Tầng 7, Tòa nhà Grand Terra, 36 Cát Linh, Đống Đa, Hà Nội.
* Cán bộ hướng dẫn cơ sở: Phạm Văn Phóng và Đỗ Tuấn Anh, chức vụ Solutions Architect.
* Cán bộ phụ trách đơn vị: Nguyễn Gia Hưng, chức vụ Senior Solutions Architect, AWS Việt Nam.
* Mục tiêu kỹ thuật:
  1. Đóng gói 3 tệp trọng số mô hình gồm model.joblib, tfidf.joblib, svd.joblib và tệp mã nguồn xử lý lambda_function.py.
  2. Viết Dockerfile tối ưu phân tầng dựa trên Base Image chính thức của AWS là public.ecr.aws/lambda/python:3.12.
  3. Khởi tạo kho lưu trữ Amazon Elastic Container Registry riêng tư mang tên phishing-xgboost tại khu vực ap-southeast-1.
  4. Thực hiện xác thực kết nối giữa Docker Client trên máy trạm và Amazon ECR bằng chuỗi mã tạm thời qua lệnh aws ecr get-login-password.
  5. Biên dịch, gán nhãn và tải Container Image lên Amazon ECR, kiểm tra tính toàn vẹn của mã băm Image Digest.

---

### 2. Kế hoạch triển khai và nhật ký công việc chi tiết

| Thứ | Nội dung công việc và mục tiêu kỹ thuật | Ngày bắt đầu | Ngày hoàn thành | Tài liệu tham khảo | Kết quả và ghi nhận thực tế |
| :--- | :--- | :---: | :---: | :--- | :--- |
| Thứ 2 | Kiểm tra tính toàn vẹn của 3 tệp mô hình đã huấn luyện gồm model.joblib phân loại XGBoost, tfidf.joblib trích xuất đặc trưng văn bản và svd.joblib giảm chiều không gian 300 thành phần. Tổng dung lượng ghi nhận là 37.8 MB. | 24/08/2026 | 24/08/2026 | [Joblib Persistence Documentation](https://joblib.readthedocs.io/) | Các tệp trọng số được kiểm tra mã băm MD5 và lưu trữ trong thư mục làm việc. |
| Thứ 3 | Viết tệp requirements.txt khai báo các thư viện gồm numpy, scipy, scikit-learn, xgboost và joblib. Soạn thảo Dockerfile khai báo biến môi trường LAMBDA_TASK_ROOT và chỉ định lệnh thực thi lambda_function.lambda_handler. | 25/08/2026 | 25/08/2026 | [Creating Lambda container images](https://docs.aws.amazon.com/lambda/latest/dg/images-create.html) | Hoàn thành Dockerfile với thứ tự khai báo tận dụng bộ nhớ đệm khi chạy lệnh cài đặt thư viện. |
| Thứ 4 | Tạo kho lưu trữ Amazon ECR riêng tư bằng lệnh aws ecr create-repository với tham số tên phishing-xgboost. Kích hoạt tính năng tự động quét lỗ hổng bảo mật khi tải ảnh lên scanOnPush. | 26/08/2026 | 26/08/2026 | [Amazon ECR Repositories](https://docs.aws.amazon.com/AmazonECR/latest/userguide/Repositories.html)<br>[ECR Image Scanning](https://docs.aws.amazon.com/AmazonECR/latest/userguide/image-scanning.html) | Kho lưu trữ ECR được tạo với định danh 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost. |
| Thứ 5 | Đăng nhập Docker Client với ECR bằng lệnh aws ecr get-login-password. Thực hiện biên dịch Container Image trên máy trạm bằng lệnh docker build gán nhãn phishing-xgboost:latest. | 27/08/2026 | 27/08/2026 | [Docker CLI Documentation](https://docs.docker.com/engine/reference/commandline/cli/) | Quá trình biên dịch hoàn thành sau 115 giây, Container Image khởi chạy bình thường trên nền tảng Amazon Linux 2023. |
| Thứ 6 | Gán thẻ Container Image theo đường dẫn kho lưu trữ trên ECR và thực hiện lệnh docker push để tải ảnh lên đám mây. | 28/08/2026 | 28/08/2026 | [Pushing a Docker image to ECR](https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html) | Tải Container Image lên ECR hoàn tất với dung lượng nén 210 MB, kích thước sau giải nén là 540 MB. |
| Thứ 7 và Chủ nhật | Kiểm tra kết quả quét lỗ hổng ECR Scan ghi nhận 0 lỗ hổng nghiêm trọng và 0 lỗ hổng mức cao. Đọc tài liệu cấu hình quyền truy cập IAM cho phép dịch vụ Lambda lấy ảnh từ ECR. | 29/08/2026 | 30/08/2026 | [IAM Permissions for Lambda Container Images](https://docs.aws.amazon.com/lambda/latest/dg/images-permissions.html) | Chuẩn bị đầy đủ Container Image cho bước khởi tạo hàm Lambda. |

---

### 3. Thao tác kỹ thuật và mã nguồn cấu hình Dockerfile

#### 3.1. Mã nguồn tệp Dockerfile
```dockerfile
# Sử dụng Base Image chính thức của AWS Lambda dành cho Python 3.12 trên Amazon Linux 2023
FROM public.ecr.aws/lambda/python:3.12

# Cài đặt các thư viện Machine Learning cần thiết
COPY requirements.txt ${LAMBDA_TASK_ROOT}/
RUN pip install --no-cache-dir -r ${LAMBDA_TASK_ROOT}/requirements.txt

# Sao chép các tệp trọng số mô hình vào thư mục làm việc của Lambda
COPY model.joblib tfidf.joblib svd.joblib ${LAMBDA_TASK_ROOT}/

# Sao chép mã nguồn xử lý logic hàm Lambda
COPY lambda_function.py ${LAMBDA_TASK_ROOT}/

# Thiết lập điểm vào thực thi
CMD [ "lambda_function.lambda_handler" ]
```

#### 3.2. Quy trình biên dịch và tải Image lên Amazon ECR
```bash
# 1. Khởi tạo ECR Repository
aws ecr create-repository \
    --repository-name phishing-xgboost \
    --image-scanning-configuration scanOnPush=true \
    --region ap-southeast-1

# 2. Xác thực Docker CLI với kho lưu trữ ECR
aws ecr get-login-password --region ap-southeast-1 | \
    docker login --username AWS --password-stdin 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com

# 3. Đóng gói Container Image cục bộ
docker build -t phishing-xgboost:latest .

# 4. Gán thẻ theo địa chỉ ECR đích
docker tag phishing-xgboost:latest 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest

# 5. Đẩy Image lên Amazon ECR
docker push 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest
```

---

### 4. Vấn đề kỹ thuật và cách thức xử lý

* Sự cố: Lỗi exec format error do khác biệt kiến trúc phần cứng khi chạy Container trên Lambda.
  * Hiện tượng: Sau khi cấu hình hàm Lambda từ Image ECR, kiểm thử gọi hàm trả về thông báo lỗi fork exec format error cùng mã trạng thái 502 Bad Gateway.
  * Nguyên nhân: Máy tính trạm sử dụng chip kiến trúc ARM64, khi chạy lệnh docker build tạo ra định dạng nhị phân linux/arm64. Trong khi đó, hàm Lambda được khởi tạo mặc định theo kiến trúc x86_64. Sự không tương thích giữa tập lệnh kiến trúc khiến hệ điều hành không nạp được các tệp nhị phân của thư viện vào môi trường microVM.
  * Giải pháp xử lý: Thêm tham số chỉ định kiến trúc phần cứng khi biên dịch Container Image:
    ```bash
    docker build --platform linux/amd64 -t phishing-xgboost:latest .
    ```
    Dự án chuẩn hóa cấu hình biên dịch theo kiến trúc linux/amd64 để đảm bảo tương thích với các tệp thư viện của XGBoost.

---

### 5. Kết quả hoàn thành

1. Tải Container Image lên kho lưu trữ Amazon ECR tại địa chỉ 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest.
2. Hoàn thành quy trình đóng gói môi trường Python 3.12, các thư viện phụ thuộc và 3 tệp mô hình với dung lượng nén 210 MB.
3. Kết quả quét an toàn bảo mật trên Amazon ECR ghi nhận không có lỗ hổng nghiêm trọng.
