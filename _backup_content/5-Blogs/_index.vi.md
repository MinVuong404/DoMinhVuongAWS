---
title: "Bài viết kỹ thuật (Blog)"
date: 2026-08-28
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Danh Mục Bài Viết Kỹ Thuật Chia Sẻ Cộng Đồng (Technical Blogs)

Trong quá trình thực tập tại AWS Vietnam, việc đóng góp tri thức và chia sẻ kinh nghiệm kỹ thuật thực chiến đến cộng đồng là một trong những giá trị cốt lõi được khuyến khích. Dưới đây là bài viết kiến trúc chuyên sâu được biên soạn và công bố:

---

### [5.1. Triển Khai Hệ Thống AI Phát Hiện Email Lừa Đảo 100% Serverless Trên AWS: Tối Ưu Độ Trễ < 15ms và Chi Phí $0 Vận Hành](5.1-serverless-phishing-detection/)
* **Chủ đề chuyên môn:** Serverless Machine Learning, Docker Container trên AWS Lambda, Tối ưu hóa Cold Start và Kiến trúc Zero-Cost trên AWS Free Tier.
* **Kênh công bố:** Xuất bản trên nhóm chuyên môn **AWS Study Group (FCJ)** và cổng thông tin sinh viên nghiên cứu khoa học.
* **Điểm nhấn kỹ thuật:**
  * So sánh toàn diện giữa việc thuê máy ảo GPU EC2 / SageMaker Endpoint đắt đỏ vs. Triển khai mô hình nén XGBoost + TruncatedSVD trong Container Lambda.
  * Kỹ thuật tải mô hình trước ra Global Scope triệt tiêu độ trễ nạp dữ liệu.
  * Chiến lược cấu hình Provisioned Capacity DynamoDB để vận hành trọn đời miễn phí.
