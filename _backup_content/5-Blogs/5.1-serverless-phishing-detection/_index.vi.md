---
title: "5.1. Blog: Serverless Phishing AI"
date: 2026-08-28
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

# Triển Khai Hệ Thống AI Phát Hiện Email Lừa Đảo 100% Serverless Trên AWS: Tối Ưu Độ Trễ < 15ms và Chi Phí $0 Vận Hành

*Tác giả: Đỗ Minh Vương — Cloud Solutions Architect Intern (AWS Vietnam / FCAJ 2026)*  
*Chuyên mục: Serverless Computing, Machine Learning on Cloud, AWS Well-Architected*

---

### 1. Nghịch Lý Chi Phí Trong Triển Khai AI Cho Doanh Nghiệp Nhỏ

Khi nhắc đến việc đưa một mô hình Trí tuệ Nhân tạo (Machine Learning / Deep Learning) lên môi trường Cloud thực tế (Production), lựa chọn phổ biến mà các kỹ sư thường nghĩ tới là **Amazon SageMaker Real-Time Endpoint** hoặc một cụm máy ảo **Amazon EC2 (ví dụ `g4dn.xlarge` có GPU hoặc `c6i.xlarge` tính toán CPU)**.

Tuy nhiên, đối với một doanh nghiệp vừa và nhỏ (SME) hoặc một ứng dụng tiện ích kiểm tra email cho nhân viên văn phòng:
* Một SageMaker Endpoint instance loại nhỏ nhất (`ml.t3.medium`) chạy 24/7 tiêu tốn khoảng **$45 – $60 USD / tháng**.
* Một máy chủ ảo EC2 `t3.medium` gắn ổ cứng EBS tiêu tốn khoảng **$35 – $45 USD / tháng**.
* **Nghịch lý xảy ra:** Nhân viên văn phòng chỉ kiểm tra email trong giờ hành chính (8 tiếng/ngày), ban đêm và cuối tuần hệ thống hoàn toàn không có request. Nhưng doanh nghiệp vẫn phải trả 100% tiền thuê máy chủ nhàn rỗi!

Câu hỏi kiến trúc đặt ra: **Làm thế nào để xây dựng một hệ thống AI nhận diện email lừa đảo đạt độ chính xác > 97%, phản hồi trong chớp mắt (< 20 ms), nhưng chi phí vận hành ở trạng thái chờ là đúng 0 đồng ($0.00)?**

Câu trả lời chính là: **100% Serverless Architecture trên AWS**.

---

### 2. Thiết Kế Mô Hình AI: Tinh Gọn Thay Vì Cồng Kềnh

Thay vì sử dụng các mô hình ngôn ngữ lớn (LLMs) hàng tỷ tham số tiêu tốn hàng Gigabyte VRAM, bài toán phân loại email lừa đảo thực chất là bài toán **Phân loại Văn bản Nhị phân (Binary Text Classification)**.

Chúng tôi áp dụng kỹ thuật tinh gọn hóa mô hình gồm 3 bước:
1. **Trích xuất đặc trưng từ vựng (TF-IDF):** Lập từ điển 10.000 n-grams phản ánh các từ khóa nguy cơ cao (*"khẩn cấp", "khóa thẻ", "xác thực otp", "chuyển tiền", "trúng thưởng"*).
2. **Nén không gian ngữ nghĩa bằng TruncatedSVD:** Giảm chiều ma trận thưa từ 10.000 chiều xuống còn **300 chiều tiềm ẩn**. Kỹ thuật này giúp loại bỏ nhiễu, giữ lại 92.4% đặc trưng ngữ nghĩa và giảm kích thước mảng đầu vào hơn 95%.
3. **Mô hình suy luận XGBoost Classifier:** Cây quyết định tăng cường gradient với độ sâu cây tối ưu hóa.

**Kết quả thu được:** Tổng dung lượng 3 tệp mô hình (`model.joblib`, `tfidf.joblib`, `svd.joblib`) chỉ nặng vẻn vẹn **37.8 MB**, tiêu tốn dưới **150 MB RAM** khi thực thi suy luận và đạt độ chính xác **97.4%** trên tập dữ liệu kiểm thử độc lập.

---

### 3. Vượt Rào Cản Giới Hạn 250 MB Của AWS Lambda Bằng Docker Container

Theo mặc định, AWS Lambda chỉ cho phép tải lên tệp nén ZIP có dung lượng sau khi giải nén tối đa là **250 MB**. Khi cài đặt các gói tính toán khoa học của Python như `scipy`, `scikit-learn`, `xgboost`, tổng dung lượng `site-packages` thường xuyên chạm ngưỡng 350 – 400 MB, dẫn đến lỗi từ chối triển khai `InvalidParameterValueException`.

**Giải pháp:** Tận dụng tính năng **AWS Lambda Container Image Support** thông qua **Amazon ECR**.
* Sử dụng Base Image tối ưu: `public.ecr.aws/lambda/python:3.12` dựa trên hệ điều hành Amazon Linux 2023.
* AWS Lambda hỗ trợ Container Image có dung lượng lên đến **10 GB**, giải quyết triệt để vấn đề kích thước thư viện.

```dockerfile
FROM public.ecr.aws/lambda/python:3.12
COPY requirements.txt ${LAMBDA_TASK_ROOT}/
RUN pip install --no-cache-dir -r ${LAMBDA_TASK_ROOT}/requirements.txt
COPY model.joblib tfidf.joblib svd.joblib ${LAMBDA_TASK_ROOT}/
COPY lambda_function.py ${LAMBDA_TASK_ROOT}/
CMD [ "lambda_function.lambda_handler" ]
```

---

### 4. Kỹ Thuật Global Scope: Bí Quyết Đạt Độ Trễ < 15ms

Thách thức lớn nhất của Lambda Container là hiện tượng **Cold Start** (mất từ 1.2 đến 1.8 giây trong lần đầu tiên để nạp microVM và container image). Nếu trong mỗi lần chạy ta đều gọi `joblib.load()`, hàm sẽ luôn bị trễ thêm 800 ms.

**Bí quyết kỹ thuật:** Nạp toàn bộ mô hình và khởi tạo SDK client ra ngoài phạm vi hàm `lambda_handler` (**Global Scope**):

```python
# KHỞI TẠO GLOBAL SCOPE: CHỈ CHẠY 1 LẦN KHI CONTAINER KHỞI ĐỘNG
dynamodb = boto3.resource('dynamodb')
sns = boto3.client('sns')
table = dynamodb.Table('PhishingDetectionLogs')

TASK_ROOT = os.environ.get('LAMBDA_TASK_ROOT', '.')
MODEL = joblib.load(os.path.join(TASK_ROOT, 'model.joblib'))
TFIDF = joblib.load(os.path.join(TASK_ROOT, 'tfidf.joblib'))
SVD = joblib.load(os.path.join(TASK_ROOT, 'svd.joblib'))

def lambda_handler(event, context):
    # CÁC LẦN GỌI WARM START CHỈ THỰC THI SUY LUẬN TOÁN HỌC TRÊN RAM
    # THỜI GIAN XỬ LÝ CHỈ MẤT 11 - 13 MS!
    cleaned_text = clean_text(event['body'])
    vector = SVD.transform(TFIDF.transform([cleaned_text]))
    prob = MODEL.predict_proba(vector)[0][1]
    ...
```

Nhờ cơ chế giữ ấm container của AWS Lambda, đối với 99% các request tiếp theo của người dùng, container được tái sử dụng và hàm chỉ mất đúng **12 mili-giây** để hoàn tất việc suy luận!

---

### 5. Bài Toán Kinh Tế: Tối Ưu Hóa Chi Phí Tuyệt Đối

Dưới đây là bảng so sánh chi phí thực tế giữa các phương án triển khai phục vụ **100.000 lượt quét email / tháng**:

| Phương án triển khai | Cấu hình hạ tầng | Chi phí máy chủ nhàn rỗi | Chi phí xử lý 100k requests | Tổng chi phí / tháng |
| :--- | :--- | :---: | :---: | :---: |
| **Amazon SageMaker Endpoint** | 1 instance `ml.t3.medium` | $50.40 | $0.00 | **$50.40 USD** |
| **Amazon EC2 Virtual Machine** | 1 instance `t3.medium` + 30GB EBS | $33.58 | $0.00 | **$33.58 USD** |
| **Serverless Architecture (Dự án)** | **Lambda 512MB + API Gateway + DynamoDB** | **$0.00** | **$0.00 (Trong Free Tier)** | **$0.00 USD** |

> [!NOTE]
> Ngay cả khi quy mô tăng gấp 10 lần (1.000.000 requests/tháng), kiến trúc Serverless cũng chỉ tiêu tốn vỏn vẹn **$1.15 USD**, tiết kiệm hơn **97% ngân sách** so với EC2 truyền thống.

---

### 6. Lời Kết

Dự án chứng minh rằng: **Điện toán đám mây hiện đại không chỉ là việc đưa máy chủ lên Internet, mà là tư duy tối ưu hóa kiến trúc**. Việc kết hợp thông minh giữa các mô hình học máy tinh gọn và các dịch vụ phi máy chủ của AWS cho phép các kỹ sư xây dựng những giải pháp có độ sẵn sàng cao, bảo mật cấp doanh nghiệp và tối ưu chi phí đến mức hoàn hảo.
