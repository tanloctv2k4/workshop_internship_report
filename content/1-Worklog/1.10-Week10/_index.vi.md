---
title: "Worklog Tuần 10"
date: 2026-06-26
weight: 10
chapter: false
pre: " <b> 1.10. </b> "
---

### Mục tiêu tuần 10:

- Tối ưu hóa Sơ đồ Thực thể Mối quan hệ (ERD) và chuyển đổi thành Mô hình Dữ liệu Vật lý (Physical Data Model).
- Viết các tập lệnh SQL DDL (Data Definition Language) để khởi tạo bảng, thiết lập khóa chính, khóa ngoại và các chỉ mục (Index).
- Khởi tạo cơ sở dữ liệu trên thực thể AWS RDS và nạp các tập dữ liệu mẫu (Seed Data) để phục vụ chạy kiểm thử.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Rà soát bản vẽ ERD và chuyển đổi sang Mô hình dữ liệu vật lý (Physical Data Model).<br>- Xác định kiểu dữ liệu, độ dài và thuộc tính cho từng cột trong cơ sở dữ liệu. | 22/06/2026 | 22/06/2026 | https://000043.awsstudygroup.com/vi/ |
| 3 | - Soạn thảo các tập lệnh SQL DDL tạo bảng `users`, `products`, `categories`, `orders`, `chat_histories`.<br>- Thiết lập Primary Key, Foreign Key, Unique, Default và các ràng buộc dữ liệu. | 23/06/2026 | 23/06/2026 | https://000043.awsstudygroup.com/vi/ |
| 4 | - Khởi tạo cơ sở dữ liệu trên Amazon RDS.<br>- Thực thi các tập lệnh DDL và kiểm tra cấu trúc các bảng sau khi tạo. | 24/06/2026 | 24/06/2026 | https://000043.awsstudygroup.com/vi/ |
| 5 | - Xây dựng các tập lệnh SQL DML để nạp dữ liệu mẫu (Seed Data).<br>- Thực hiện kiểm tra dữ liệu sau khi nạp vào Amazon RDS. | 25/06/2026 | 25/06/2026 | https://000043.awsstudygroup.com/vi/ |
| 6 | - Kiểm thử các truy vấn SQL, kiểm tra ràng buộc khóa ngoại và tính toàn vẹn dữ liệu.<br>- Đánh giá khả năng tích hợp cơ sở dữ liệu với API Backend. | 26/06/2026 | 26/06/2026 | https://000043.awsstudygroup.com/vi/ |

### Kết quả đạt được tuần 10:

| Thứ | Công việc | Kết quả đạt được |
| --- | --- | --- |
| 2 | Chuyển đổi Mô hình Vật lý | Hoàn thiện Mô hình Dữ liệu Vật lý, chuẩn hóa chính xác kiểu dữ liệu và thuộc tính cho toàn bộ hệ thống. |
| 3 | Soạn thảo tập lệnh DDL | Viết hoàn chỉnh file script SQL DDL tạo bảng và thiết lập đầy đủ các ràng buộc khóa chính, khóa ngoại. |
| 4 | Khởi tạo bảng trên AWS RDS | Thực thi DDL thành công trên AWS RDS, tạo lập hệ thống bảng dữ liệu hoàn chỉnh không phát sinh lỗi. |
| 5 | Nạp dữ liệu mẫu (Seed Data) | Nạp thành công tập dữ liệu mẫu về danh mục dược phẩm và thông tin người dùng vào RDS để kiểm thử. |
| 6 | Kiểm thử toàn vẹn dữ liệu | Xác minh các ràng buộc khóa ngoại hoạt động chuẩn xác, đảm bảo tính nhất quán dữ liệu trước khi tích hợp API. |