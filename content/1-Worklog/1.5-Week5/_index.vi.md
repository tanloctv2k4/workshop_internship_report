---
title: "Worklog Tuần 5"
date: 2026-05-24
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
---

### Mục tiêu tuần 5:

- Tìm hiểu dịch vụ Amazon Simple Storage Service (Amazon S3) và cơ chế lưu trữ Object Storage.
- Thực hành triển khai Static Website Hosting trên Amazon S3.
- Hiểu cơ chế sao chép dữ liệu giữa các Region bằng S3 Cross-Region Replication (CRR).
- Tìm hiểu AWS Security Hub và cách theo dõi trạng thái bảo mật của tài khoản AWS.
- Thực hành sử dụng AWS Lambda kết hợp Amazon EventBridge để tự động hóa việc quản lý EC2 nhằm tối ưu chi phí.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Tìm hiểu tổng quan Amazon S3, S3 Bucket và Object.<br>- Tạo S3 Bucket và cấu hình Block Public Access. | 18/05/2026 | 18/05/2026 | https://000057.awsstudygroup.com/vi/ |
| 3 | - Cấu hình Static Website Hosting trên Amazon S3.<br>- Upload Object và kiểm tra khả năng truy cập Website tĩnh. | 19/05/2026 | 19/05/2026 | https://000057.awsstudygroup.com/vi/ |
| 4 | - Thực hành cấu hình S3 Cross-Region Replication (CRR).<br>- Kiểm tra việc sao chép dữ liệu giữa hai S3 Bucket ở hai Region khác nhau. | 20/05/2026 | 20/05/2026 | https://000057.awsstudygroup.com/vi/ |
| 5 | - Tìm hiểu AWS Security Hub.<br>- Kích hoạt Security Hub và theo dõi các Security Findings, Security Standards. | 21/05/2026 | 21/05/2026 | https://000018.awsstudygroup.com/vi/ |
| 6 | - Tìm hiểu AWS Lambda kết hợp Amazon EventBridge.<br>- Xây dựng Lambda Function để tự động Start/Stop EC2 theo sự kiện nhằm tối ưu chi phí. | 22/05/2026 | 22/05/2026 | https://000022.awsstudygroup.com/vi/ |
| 7 | - Kiểm thử hoạt động của Lambda và EventBridge.<br>- Kiểm tra kết quả của Static Website Hosting, S3 Cross-Region Replication và hoàn thiện cấu hình các dịch vụ đã triển khai. | 23/05/2026 | 23/05/2026 | https://000022.awsstudygroup.com/vi/ |

### Kết quả đạt được tuần 5:

| Thứ | Công việc | Kết quả đạt được | Hình ảnh |
| --- | --- | --- | --- |
| 2 | Cấu hình Amazon S3 Bucket | Tạo thành công S3 Bucket và cấu hình Block Public Access nhằm kiểm soát quyền truy cập theo đúng khuyến nghị bảo mật của AWS. | ![S3 Bucket](anh5.1.png) |
| 3 | Upload Object lên Amazon S3 | Upload thành công các Object lên S3 Bucket và kiểm tra dữ liệu được lưu trữ chính xác, sẵn sàng phục vụ Static Website Hosting. | ![Upload Object](anh5.2.png) |
| 4 | Cấu hình S3 Cross-Region Replication | Thiết lập thành công Replication Rule giữa hai S3 Bucket ở hai Region khác nhau, dữ liệu được tự động sao chép nhằm tăng khả năng dự phòng và hỗ trợ Disaster Recovery. | |
| 5 | AWS Security Hub | Kích hoạt AWS Security Hub, tìm hiểu cách tổng hợp các Security Findings và theo dõi trạng thái tuân thủ bảo mật của tài khoản AWS. | |
| 6 | Tối ưu chi phí EC2 bằng AWS Lambda | Xây dựng và kiểm thử thành công Lambda Function kết hợp Amazon EventBridge để tự động Start/Stop EC2 theo sự kiện, góp phần tối ưu chi phí vận hành hệ thống. | ![Lambda](anh5.3.png) |
| 7 | Kiểm tra và hoàn thiện hệ thống | Kiểm tra hoạt động của Static Website Hosting, xác minh dữ liệu được sao chép thành công giữa các Region thông qua Cross-Region Replication và đánh giá kết quả thực thi của Lambda Function trước khi hoàn tất bài Lab. | |