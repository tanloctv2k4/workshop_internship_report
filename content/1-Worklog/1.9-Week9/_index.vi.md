---
title: "Worklog Tuần 9"
date: 2026-06-19
weight: 9
chapter: false
pre: " <b> 1.9. </b> "
---

### Mục tiêu tuần 9:

- Khảo sát các quy trình nghiệp vụ thực tế của hệ thống PharmaCare AI để xác định danh mục dữ liệu cần quản lý.
- Phân tích và thiết kế Mô hình dữ liệu khái niệm (Conceptual Data Model) và Mô hình dữ liệu logic (Logical Data Model).
- Xây dựng bản phác thảo Sơ đồ Thực thể Mối quan hệ (ERD - Entity Relationship Diagram) và xác định các quy tắc toàn vẹn dữ liệu.
- Thiết lập từ điển dữ liệu (Data Dictionary) ban đầu cho các thực thể người dùng, dược phẩm, đơn hàng và truy vấn AI.

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Khảo sát quy trình quản lý dược phẩm, đơn hàng và lịch sử tương tác AI của dự án PharmaCare AI.<br>- Liệt kê danh sách các tập dữ liệu đầu vào, đầu ra và các luồng thông tin nghiệp vụ chính. | 15/06/2026 | 15/06/2026 | Tài liệu đặc tả yêu cầu dự án PharmaCare AI |
| 3 | - Xây dựng Mô hình dữ liệu khái niệm (Conceptual Data Model) đại diện cho các thực thể cốt lõi.<br>- Xác định mối quan hệ giữa các thực thể (1-1, 1-N, N-N) trong sơ đồ nghiệp vụ. | 16/06/2026 | 16/06/2026 | Dbdiagram.io |
| 4 | - Phân tích và chuẩn hóa dữ liệu đạt dạng chuẩn 1NF, 2NF và 3NF để giảm thiểu trùng lặp dữ liệu.<br>- Chuyển đổi các mối quan hệ nhiều - nhiều (N-N) thành các bảng trung gian (Junction Tables). | 17/06/2026 | 17/06/2026 | Giáo trình Cơ sở dữ liệu; Microsoft SQL Server Documentation |
| 5 | - Vẽ lược đồ quan hệ sơ bộ chi tiết bằng công cụ thiết kế cơ sở dữ liệu (draw.io/DbDiagram).<br>- Xác định khóa chính (Primary Key), khóa ngoại (Foreign Key) và các ràng buộc dữ liệu (Not Null, Unique, Check). | 18/06/2026 | 18/06/2026 | https://dbdiagram.io |
| 6 | - Soạn thảo tài liệu Từ điển dữ liệu (Data Dictionary) chi tiết bao gồm kiểu dữ liệu, độ dài và mô tả trường.<br>- Tham vấn và thống nhất thiết kế cơ sở dữ liệu với bộ phận phát triển Backend. | 19/06/2026 | 19/06/2026 | https://000043.awsstudygroup.com/vi/ |

---

### Kết quả đạt được tuần 9:

| Thứ | Công việc | Kết quả đạt được |
| --- | --- | --- |
| 2 | Phân tích quy trình nghiệp vụ | Tổng hợp đầy đủ danh mục dữ liệu nghiệp vụ và các luồng thông tin cần quản lý trong PharmaCare AI. |
| 3 | Thiết kế mô hình khái niệm | Khởi tạo thành công Mô hình dữ liệu khái niệm và xác định các mối quan hệ thực thể nền tảng. |
| 4 | Chuẩn hóa dữ liệu 3NF | Hoàn thành việc chuẩn hóa cấu trúc dữ liệu lên dạng chuẩn 3NF, loại bỏ các bất thường khi thêm/sửa/xóa. |
| 5 | Xây dựng Sơ đồ ERD sơ bộ | Hoàn thiện thiết kế bản phác thảo Sơ đồ ERD bao gồm đầy đủ các bảng, khóa chính, khóa ngoại và ràng buộc. |
| 6 | Hoàn thiện Từ điển dữ liệu | Xây dựng bộ tài liệu Từ điển dữ liệu chuẩn hóa, sẵn sàng cho bước viết script DDL ở tuần tiếp theo. |