---
title: "Đóng gói Docker & Amazon ECR"
date: 2026-08-25
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# 4.2. Đóng gói Container Docker và lưu trữ trên Amazon ECR

### 4.2.1. Cấu trúc mã nguồn và soạn thảo Dockerfile

Để triển khai mô hình học máy chứa các thư viện C-extension gồm xgboost, scikit-learn và scipy vượt quá giới hạn 250 MB của tệp Zip Lambda, phương án được chọn là đóng gói môi trường thực thi vào OCI Container Image.

#### Danh mục tệp mã nguồn trong thư mục aws_serverless:

| Tên tệp hoặc cấu phần | Phân loại | Định dạng | Vai trò và chức năng kỹ thuật |
| :--- | :--- | :--- | :--- |
| `Dockerfile` | Khai báo container | Dockerfile | Khai báo các bước biên dịch môi trường trên nền tảng Amazon Linux 2023 Python 3.12. |
| `requirements.txt` | Thư viện phụ thuộc | Pip config | Khai báo danh sách thư viện cần cài đặt gồm scikit-learn, xgboost và joblib. |
| `lambda_function.py` | Mã nguồn Python | Python 3.12 | Tiếp nhận dữ liệu từ API Gateway, thực hiện tiền xử lý và chạy suy luận trong bộ nhớ. |
| `model.joblib` | Tệp nhị phân mô hình | Serialized Model | Trọng số mô hình phân loại nhị phân XGBoost đã huấn luyện với độ chính xác trên 97%. |
| `tfidf.joblib` | Trích xuất đặc trưng | Serialized Vectorizer | Từ điển đặc trưng TF-IDF với 10.000 unigrams và bigrams. |
| `svd.joblib` | Giảm chiều dữ liệu | Serialized SVD Matrix | Ma trận nén giảm chiều TruncatedSVD từ 10.000 xuống 300 chiều tiềm ẩn. |

#### 1. Tệp requirements.txt
```text
numpy>=1.26.0
scipy>=1.12.0
scikit-learn>=1.4.0
xgboost>=2.0.0
joblib>=1.3.0
```

#### 2. Tệp Dockerfile
```dockerfile
# Sử dụng Base Image chính thức của AWS Lambda dành cho Python 3.12 trên Amazon Linux 2023
FROM public.ecr.aws/lambda/python:3.12

# Cài đặt các thư viện Machine Learning vào thư mục chuẩn của tác vụ Lambda
COPY requirements.txt ${LAMBDA_TASK_ROOT}/
RUN pip install --no-cache-dir -r ${LAMBDA_TASK_ROOT}/requirements.txt

# Sao chép các tệp trọng số mô hình vào thư mục gốc của hàm Lambda
COPY model.joblib tfidf.joblib svd.joblib ${LAMBDA_TASK_ROOT}/

# Sao chép mã nguồn xử lý logic vào thư mục gốc
COPY lambda_function.py ${LAMBDA_TASK_ROOT}/

# Thiết lập điểm vào hàm thực thi khi có sự kiện kích hoạt
CMD [ "lambda_function.lambda_handler" ]
```

---

### 4.2.2. Khởi tạo kho lưu trữ Amazon ECR Private Repository

Dịch vụ Amazon Elastic Container Registry quản lý và lưu trữ Docker Container Image, kết nối trực tiếp với AWS Lambda qua hạ tầng mạng nội bộ.

Khởi tạo kho lưu trữ riêng tư tại khu vực ap-southeast-1:
```bash
aws ecr create-repository \
    --repository-name phishing-xgboost \
    --image-scanning-configuration scanOnPush=true \
    --image-tag-mutability MUTABLE \
    --region ap-southeast-1
```

Dữ liệu JSON phản hồi từ AWS ECR:
```json
{
    "repository": {
        "repositoryArn": "arn:aws:ecr:ap-southeast-1:803146828520:repository/phishing-xgboost",
        "registryId": "803146828520",
        "repositoryName": "phishing-xgboost",
        "repositoryUri": "803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost",
        "createdAt": "2026-08-26T09:00:00+07:00"
    }
}
```

{{% notice tip %}}
Tùy chọn scanOnPush kích hoạt tính năng tự động quét lỗ hổng bảo mật mã nguồn mở khi có Container Image mới được đẩy lên, hỗ trợ kiểm soát an toàn theo khuyến nghị của AWS.
{{% /notice %}}

---

### 4.2.3. Xác thực Docker CLI, biên dịch ảnh và đẩy Image lên ECR

#### Bước 1: Xác thực Docker Client với Amazon ECR
Sử dụng mã xác thực tạm thời từ lệnh aws ecr get-login-password:
```bash
aws ecr get-login-password --region ap-southeast-1 | \
    docker login --username AWS --password-stdin 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com
```
Kết quả ghi nhận: Login Succeeded.

#### Bước 2: Đóng gói Container Image với cờ kiến trúc linux/amd64
Chỉ định rõ cờ nền tảng để đảm bảo tương thích với môi trường chạy mặc định của Lambda:
```bash
docker build --platform linux/amd64 -t phishing-xgboost:latest .
```

#### Bước 3: Gán thẻ Image theo định dạng URI của ECR
```bash
docker tag phishing-xgboost:latest 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest
```

#### Bước 4: Đẩy Container Image lên Amazon ECR
```bash
docker push 803146828520.dkr.ecr.ap-southeast-1.amazonaws.com/phishing-xgboost:latest
```

#### Bước 5: Kiểm tra thông tin Image trên ECR
```bash
aws ecr describe-images \
    --repository-name phishing-xgboost \
    --region ap-southeast-1 \
    --query 'imageDetails[*].[imageTags, imageSizeInBytes, imageDigest]' \
    --output table
```
Kết quả kiểm tra ảnh trên kho lưu trữ:

| Nhãn ảnh | Kích thước byte | Mã băm Image Digest | Trạng thái quét |
| :---: | :---: | :--- | :---: |
| `latest` | 220456128 | `sha256:7b91d2e9fa498a12bcde0914838f7129c54...` | PASSED, 0 Critical |

Container Image hoàn tất lưu trữ trong kho ECR và sẵn sàng tích hợp với AWS Lambda.
