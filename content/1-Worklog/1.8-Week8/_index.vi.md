---
title: "Worklog Tuần 8"
date: 2026-06-14
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Mục tiêu tuần 8:
- Tìm hiểu và chuyển đổi cấu trúc ứng dụng từ Monolith sang Serverless/Serviceful Microservices.
- Triển khai hệ thống quản lý danh tính và phân quyền bảo mật (AAA) với Amazon Cognito.
- Cấu hình và làm việc với hai hệ thống cơ sở dữ liệu phổ biến trên đám mây: Amazon RDS (Quan hệ) và Amazon DynamoDB (NoSQL).
- Xây dựng, đóng gói và phân phối tự động các hàm xử lý dữ liệu (Python/Node.js) qua AWS SAM CLI và API Gateway.
- Vận hành Static Website Hosting (SPA) trên S3 và thiết lập cơ chế giám sát, phân tích hiệu năng thời gian thực bằng AWS X-Ray.

---

### Các công việc triển khai trong tuần (Theo thứ tự logic hệ thống):

| Thứ | Nội dung công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | Hạ tầng AAA & Database NoSQL<br>- Khởi tạo Amazon Cognito User Pool (cognito-fcj-book-shop) và Identity Pool để thiết lập hệ thống đăng ký/đăng nhập.<br>- Khởi tạo bảng cơ sở dữ liệu NoSQL TravelBuddyTripSectors trên Amazon DynamoDB đáp ứng tốc độ phản hồi I/O cao. | 08/06/2026 | 08/06/2026 | https://000081.awsstudygroup.com/<br>https://000055.awsstudygroup.com/ |
| 3 | Phát triển Backend & Serverless API<br>- Viết các Lambda function xử lý logic xác thực (Register, Confirm_user, Login) bằng Python.<br>- Cấu hình định tuyến, thiết lập cơ chế CORS và phân phối tự động toàn bộ tài nguyên Serverless bằng công cụ AWS SAM CLI. | 09/06/2026 | 09/06/2026 | https://000081.awsstudygroup.com/ |
| 4 | Triển khai Giao diện Frontend SPA<br>- Cấu hình build mã nguồn Single Page Application (SPA) của dự án TravelBuddy.<br>- Triển khai lưu trữ tĩnh lên Amazon S3 dưới dạng Static Website Hosting.<br>- Tích hợp Bearer Token (JWT từ Cognito) vào Client để bảo mật toàn bộ các điểm cuối gọi về API Gateway. | 10/06/2026 | 10/06/2026 | https://000055.awsstudygroup.com/ |
| 5 | Thiết lập Hạ tầng Database Quan hệ (RDS)<br>- Nghiên cứu tổng quan dịch vụ quản lý cơ sở dữ liệu quan hệ Amazon RDS (MySQL).<br>- Thiết lập DB Subnet group trải rộng trên nhiều Availability Zone (AZ) để đảm bảo khả năng dự phòng thảm họa.<br>- Tạo và cấu hình Security Group riêng cho DB Instance để kiểm soát quyền truy cập chặt chẽ. | 11/06/2026 | 11/06/2026 | https://000005.awsstudygroup.com/ |
| 6 | Vận hành Ứng dụng Toàn phần kết nối RDS<br>- SSH vào máy chủ EC2 qua MobaXterm, cài đặt Node.js và môi trường Client MySQL.<br>- Clone source code ứng dụng quản lý, cấu hình biến môi trường .env kết nối trực tiếp đến Endpoint của RDS.<br>- Chạy SQL script định nghĩa cấu trúc bảng user, nạp dữ liệu mẫu và khởi chạy Web Server trên cổng 5000. | 12/06/2026 | 12/06/2026 | https://000005.awsstudygroup.com/ |
| 7 | Kiểm thử Backup & Khôi phục dữ liệu RDS<br>- Cấu hình tự động hóa và thời gian thực hiện khung giờ bảo trì (Maintenance Window) trên RDS.<br>- Thực hành tạo Snapshot thủ công để lưu giữ trạng thái an toàn của hệ thống cơ sở dữ liệu.<br>- Thực hiện quy trình Restore Snapshot để dựng lại một DB Instance mới độc lập và kiểm tra tính toàn vẹn dữ liệu. | 13/06/2026 | 13/06/2026 | https://000005.awsstudygroup.com/ |
| CN | Giám sát hiệu năng hệ thống với AWS X-Ray<br>- Kích hoạt tính năng Active Tracing trên AWS Lambda Console cho các vi dịch vụ.<br>- Sử dụng CloudWatch Service Map để trực quan hóa toàn bộ luồng đi của request phân tán.<br>- Phân tích biểu đồ Trace chi tiết nhằm phát hiện điểm nghẽn (như thời gian tiến trình Scan bảng DynamoDB chiếm 1.67s trên tổng 5.98s phản hồi). | 14/06/2026 | 14/06/2026 | https://000055.awsstudygroup.com/ |

---

### Kết quả đạt được trong tuần 8:

| Thứ | Công việc | Kết quả đạt được |
| --- | --- | --- |
| 2 | Cấu hình Cognito & DynamoDB | Thiết lập hoàn chỉnh cổng xác thực tập trung bảo mật cao dựa trên Email và khởi tạo thành công bảng lưu trữ dữ liệu NoSQL phân tán. |
| 3 | Khởi tạo Serverless API qua SAM | Đóng gói và cập nhật thành công deployment stage cho cụm API Gateway kết hợp Lambda. Toàn bộ các stack đạt trạng thái **CREATE_COMPLETE**. |
| 4 | Triển khai Static Website SPA | Ứng dụng Front-end chạy ổn định trên S3 Endpoint, xử lý mượt mà luồng đăng ký người dùng mới và áp dụng cơ chế chặn request không đính kèm Token. |
| 5 | Cấu hình mạng an toàn cho RDS | Cấu hình thành công cụm mạng cơ sở dữ liệu biệt lập trong VPC, cô lập tài nguyên DB khỏi môi trường internet trực tiếp. |
| 6 | Đồng bộ hóa Ứng dụng & RDS | Nạp thành công 18 bản ghi người dùng vào cơ sở dữ liệu quan hệ. Hệ thống ứng dụng Node.js trên cổng 5000 hiển thị dữ liệu chuẩn xác trên giao diện web. |
| 7 | Quản lý Phục hồi Thảm họa (DR) | Bản snapshot thủ công được khởi tạo an toàn trên S3. Quá trình dựng lại database từ backup diễn ra hoàn tất, hệ thống cơ sở dữ liệu mới hoạt động ở trạng thái **Available**. |
| CN | Tối ưu hóa hiệu năng qua X-Ray | Sở hữu sơ đồ luồng dữ liệu (Service Map) trực quan xuyên suốt các thành phần. Xác định rõ ảnh hưởng của tình trạng Cold Start và thao tác quét bảng (Scan) để lên phương án tối ưu index. |