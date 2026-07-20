---
title: "Source Code và Video Demo"
date: 2026-07-09
weight: 10
chapter: false
pre: " <b> 5.8. </b> "
---

Phần này cung cấp source code và video demo của dự án **PharmaCare AI** sau khi hệ thống đã được triển khai trên AWS.

Source code và video demo được lưu trữ trên Google Drive để thuận tiện cho việc truy cập và đánh giá.

---

## 1. Source Code

Source code bao gồm các thành phần chính của hệ thống **PharmaCare AI**:

* Source code frontend sử dụng ReactJS.
* Source code backend API sử dụng AWS Lambda với Node.js 22.x.
* Script migration database cho Amazon RDS PostgreSQL.
* Source code AI/RAG cho Lambda Indexing và Lambda Chatbot.
* Các file cấu hình sử dụng trong quá trình phát triển và triển khai.

> Lưu ý: Các thông tin nhạy cảm như AWS credentials, mật khẩu database, secret value và các key theo môi trường triển khai không được đưa vào source code chia sẻ.

<div style="margin: 20px 0;">
  <a href="https://drive.google.com/drive/folders/1NeFCJBzDqJ6o5xJXiv3p9vyerpdatSDt?usp=drive_link/" target="_blank" rel="noopener noreferrer" style="display: inline-block; padding: 10px 18px; background-color: #2563eb; color: white; text-decoration: none; border-radius: 6px;">
    Xem Source Code trên Google Drive
  </a>
</div>

---

## 2. Video Demo

Video demo trình bày các chức năng chính của hệ thống **PharmaCare AI**, bao gồm:

* Truy cập website đã deploy thông qua CloudFront.
* Xem danh mục sản phẩm và chi tiết sản phẩm.
* Xác thực người dùng bằng Amazon Cognito.
* Chức năng khách hàng như giỏ hàng và đặt hàng.
* Chức năng admin như quản lý sản phẩm, người dùng và đơn hàng.
* Chatbot AI tư vấn sử dụng kiến trúc RAG.
* Tổng quan cấu hình monitoring, backup và security.
<div style="margin: 20px 0;">
  <a href="https://drive.google.com/file/d/1iurCvMBdtWCHTxBh87mCg6jfsYnTHvRs/view?usp=sharing/" target="_blank" rel="noopener noreferrer" style="display: inline-block; padding: 10px 18px; background-color: #16a34a; color: white; text-decoration: none; border-radius: 6px;">
    Xem Video Demo trên Google Drive
  </a>
</div>

---

## 3. Link Website đã Deploy

Website đã triển khai có thể truy cập tại:

<div style="margin: 20px 0;">
  <a href="https://d3tm5364zrtmpq.cloudfront.net/" target="_blank" rel="noopener noreferrer" style="display: inline-block; padding: 10px 18px; background-color: #ea580c; color: white; text-decoration: none; border-radius: 6px;">
    Mở Website PharmaCare AI
  </a>
</div>

---

## Tổng kết

Sau khi hoàn thành dự án thực tập, source code, video demo và website đã deploy được chuẩn bị để minh chứng cho kết quả cuối cùng của hệ thống **PharmaCare AI**. Các tài liệu này giúp người xem hiểu rõ quá trình triển khai và sản phẩm hoàn thiện.