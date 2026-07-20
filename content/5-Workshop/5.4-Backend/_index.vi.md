---
title : "Lambda Backend"
date : 2026-07-14
weight : 4 
chapter : false
pre : " <b>5.4. </b> "
---

## Tổng quan AWS Lambda

Trong kiến trúc **PharmaCare AI**, AWS Lambda được sử dụng để xử lý nghiệp vụ backend và các chức năng AI theo mô hình serverless.

Hệ thống gồm ba Lambda chính:

- **Lambda Backend API:** Xử lý sản phẩm, danh mục, tài khoản, giỏ hàng, đơn hàng và các chức năng quản trị.
- **Lambda Chatbot Handler:** Nhận câu hỏi từ người dùng, gọi Amazon Bedrock và truy vấn dữ liệu trong Amazon OpenSearch để tạo câu trả lời.
- **Lambda Indexing:** Đọc tài liệu từ Amazon S3, tạo embedding bằng Amazon Bedrock và lưu dữ liệu vector vào OpenSearch.

Các Lambda được triển khai trong **Private App Subnet** và kết nối riêng tư đến Amazon RDS, Amazon Bedrock, Amazon OpenSearch và Amazon S3 thông qua VPC Endpoint.

Log và thông số hoạt động của Lambda được gửi đến Amazon CloudWatch để phục vụ giám sát và cảnh báo.
    
![Kiến trúc tổng thể PharmaCare AI](/workshop_internship_report/images/5-Workshop/5.1-Workshop-overview/dia02.jpg)