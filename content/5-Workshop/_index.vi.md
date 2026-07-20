---
title: "Workshop"
date: 2026-07-19
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

#### Tổng quan

Trong dự án **PharmaCare AI**, hệ thống được thiết kế theo mô hình Cloud Native, trong đó các Lambda backend và Lambda AI được triển khai bên trong **Private App Subnet** của Amazon VPC. Các workload này không cần truy cập Internet công cộng mà kết nối riêng tư đến Amazon S3, Amazon Bedrock, Amazon OpenSearch Serverless và Amazon RDS thông qua endpoint nội bộ của AWS.

Việc sử dụng **VPC Endpoint** và **AWS PrivateLink** giúp dữ liệu trao đổi giữa các dịch vụ luôn nằm trong mạng AWS, hạn chế nguy cơ lộ thông tin, giảm phụ thuộc vào NAT Gateway và tăng cường bảo mật cho hệ thống.

Trong kiến trúc PharmaCare AI, hai loại VPC Endpoint được sử dụng:

- **Gateway VPC Endpoint:** Được sử dụng để Lambda Indexing truy cập bucket **S3 Knowledge Documents**. Endpoint được liên kết với route table của Private Subnet, cho phép đọc tài liệu từ S3 mà không đi qua Internet.

- **Interface VPC Endpoint:** Được sử dụng để các Lambda AI kết nối riêng tư đến **Amazon Bedrock Runtime** và **Amazon OpenSearch Serverless** thông qua các Elastic Network Interface trong VPC. Việc phân giải địa chỉ dịch vụ được thực hiện bằng Private DNS.

Luồng hoạt động chính của hệ thống được mô tả như sau:

```text
Người dùng
    ↓
Route 53 → CloudFront → S3 Frontend
    ↓
API Gateway
    ↓
Lambda Backend trong Private App Subnet
    ↓
Amazon RDS trong Private DB Subnet
```

Đối với chatbot AI:

```text
S3 Knowledge Documents
    ↓ S3 Gateway Endpoint
Lambda Indexing
    ↓ Bedrock Runtime Interface Endpoint
Amazon Bedrock Embedding Model
    ↓
OpenSearch Serverless Vector Store
```

Khi người dùng gửi câu hỏi, **Lambda Chatbot** tạo vector cho câu hỏi bằng Amazon Bedrock, truy vấn các tài liệu liên quan trong OpenSearch Serverless và sử dụng mô hình AI để tạo câu trả lời phù hợp. Toàn bộ quá trình được giám sát thông qua Amazon CloudWatch; khi phát hiện lỗi hoặc vượt ngưỡng, CloudWatch Alarm gửi cảnh báo đến quản trị viên thông qua Amazon SNS.

#### Nội dung

1. [Tổng quan về workshop](5.1-Workshop-overview/)
2. [Chuẩn bị](5.2-VPC-and-Database/)
3. [Truy cập đến S3 từ VPC](5.3-Lambda-and-Cognitu/)
4. [Lambda Backend](5.4-Backend/)
5. [VPC Endpoint Policies (làm thêm)](5.5-Chat-aiy/)
6. [Dọn dẹp tài nguyên](5.6-Deploy-web/)
7. [Giám Sát, Sao Lưu Và Bảo Mật Hệ Thống PharmaCare](5.7-Monitoring-backup-security/)
8. [Soure Code And Demo Web](5.8-SoureCode-Demo/)