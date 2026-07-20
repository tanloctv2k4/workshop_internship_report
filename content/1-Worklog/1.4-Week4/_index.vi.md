---
title: "Worklog Tuần 4"
date: 2026-05-16
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
---

### Mục tiêu tuần 4:

- Tìm hiểu dịch vụ lưu trữ Amazon S3 và các tính năng quan trọng.
- Thực hành cấu hình S3 Cross-Region Replication để sao chép dữ liệu giữa các Region.
- Nghiên cứu dịch vụ AWS Backup và quy trình sao lưu, khôi phục dữ liệu.
- Tìm hiểu cơ chế VM Import/Export và triển khai hệ thống lưu trữ Amazon FSx for Windows.

### Các công việc thực hiện trong tuần:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --------- | ------------ | --------------- | --------------- |
| 2 | - Tìm hiểu tổng quan về Amazon S3.<br>- Nghiên cứu Bucket, Object và các lớp lưu trữ trên S3. | 11/05/2026 | 11/05/2026 | https://000013.awsstudygroup.com/vi/ |
| 3 | - Thực hành tạo Bucket, Upload Object.<br>- Tìm hiểu Versioning và Lifecycle Management. | 12/05/2026 | 12/05/2026 | https://000014.awsstudygroup.com/vi/ |
| 4 | - Thực hành cấu hình S3 Cross-Region Replication.<br>- Thiết lập Replication Rule để sao chép dữ liệu giữa hai Region. | 13/05/2026 | 13/05/2026 | https://000057.awsstudygroup.com/vi/ |
| 5 | - Tìm hiểu AWS Backup.<br>- Tạo Backup Plan và Backup Vault. | 14/05/2026 | 14/05/2026 | https://000025.awsstudygroup.com/vi/ |
| 6 | - Thực hành Restore dữ liệu từ AWS Backup.<br>- Tìm hiểu AWS SNS gửi thông báo sau khi Backup hoàn tất. | 15/05/2026 | 15/05/2026 | https://000025.awsstudygroup.com/vi/ |
| 7 | - Tìm hiểu VM Import/Export.<br>- Thực hành triển khai Amazon FSx for Windows. | 16/05/2026 | 16/05/2026 | https://000025.awsstudygroup.com/vi/ |

---

### Kết quả đạt được tuần 4:

| Thứ | Công việc | Kết quả đạt được | Hình ảnh |
| --- | --------- | ---------------- | -------- |
| 2 | Tìm hiểu Amazon S3 | Hiểu kiến trúc lưu trữ Object Storage của Amazon S3, nắm được sự khác nhau giữa Bucket và Object, các Storage Class và các tính năng bảo mật cơ bản. |  |
| 3 | Thực hành quản lý Bucket | Tạo thành công Bucket, Upload Object, cấu hình Versioning và làm quen với Lifecycle Management để quản lý dữ liệu hiệu quả. | |
| 4 | Cấu hình S3 Cross-Region Replication | Thiết lập thành công Replication Rules giúp tự động sao chép dữ liệu giữa hai Region, hiểu được cơ chế dự phòng dữ liệu và Disaster Recovery trên AWS. |  |
| 5 |Triển khai Amazon CloudFront | Tải thành công tệp máy ảo (.ova) lên Amazon S3 để chuẩn bị cho quá trình VM Import/Export. Hiểu được quy trình lưu trữ image máy ảo trên S3 trước khi import thành Amazon EC2 Instance.| ![CloudFront Distribution](anh4.1.png) |
| 6 | Thực hành VM Import/Export | Tải thành công tệp máy ảo (.ova) lên Amazon S3 để chuẩn bị cho quá trình VM Import/Export. Hiểu được quy trình lưu trữ image máy ảo trên S3 trước khi import thành Amazon EC2 Instance.| ![](anh4.2.png) |
| 7 | VM Import/Export và Amazon FSx | Hiểu quy trình Import máy ảo lên AWS EC2 bằng VM Import/Export. Tìm hiểu Amazon FSx for Windows, cấu hình File Share, quản lý lưu trữ dùng chung và các tính năng mở rộng của dịch vụ. | |