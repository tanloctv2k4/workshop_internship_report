---
title: "Worklog Tuần 11"
date: 2026-07-03
weight: 11
chapter: false
pre: " <b> 1.11. </b> "
---

### Mục tiêu tuần 11:

- Phân tích dữ liệu bằng các truy vấn SQL nâng cao (Hàm gom nhóm, Joins, Subqueries và Window Functions).
- Thiết lập chỉ mục (Indexing) và tối ưu hóa kế hoạch thực thi truy vấn (Execution Plan) để giảm độ trễ phản hồi trên AWS RDS.
- Định dạng và đóng gói các tập dữ liệu có cấu trúc từ cơ sở dữ liệu để cung cấp ngữ cảnh cho AI Chatbot (Bedrock).

### Các công việc cần triển khai trong tuần này:

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu |
| --- | --- | --- | --- | --- |
| 2 | - Viết các truy vấn SQL phân tích doanh số, lịch sử tương tác người dùng và danh mục dược phẩm.<br>- Sử dụng JOIN, GROUP BY, HAVING và Subquery để tổng hợp dữ liệu phục vụ báo cáo. | 29/06/2026 | 29/06/2026 | https://000043.awsstudygroup.com/vi/ |
| 3 | - Phân tích kế hoạch thực thi truy vấn trên Amazon RDS.<br>- Đánh giá các truy vấn có hiệu năng chưa tối ưu và xác định các vị trí cần cải thiện. | 30/06/2026 | 30/06/2026 | https://000043.awsstudygroup.com/vi/ |
| 4 | - Thiết lập các chỉ mục (Index) cho các bảng thường xuyên truy vấn.<br>- So sánh hiệu năng truy vấn trước và sau khi tối ưu hóa chỉ mục. | 01/07/2026 | 01/07/2026 | https://000043.awsstudygroup.com/vi/ |
| 5 | - Chuẩn hóa và chuyển đổi dữ liệu từ Amazon RDS sang định dạng JSON phục vụ chatbot AI.<br>- Chuẩn bị dữ liệu đầu vào cho mô hình RAG. | 02/07/2026 | 02/07/2026 | https://000035.awsstudygroup.com/vi/ |
| 6 | - Kiểm thử việc truy xuất dữ liệu từ Amazon RDS thông qua AWS Lambda.<br>- Đánh giá tốc độ phản hồi và tối ưu dữ liệu trả về cho ứng dụng. | 03/07/2026 | 03/07/2026 | https://000048.awsstudygroup.com/vi/ |
---

### Kết quả đạt được tuần 11:

| Thứ | Công việc | Kết quả đạt được |
| --- | --- | --- |
| 2 | Phân tích dữ liệu bằng SQL | Xây dựng thành công bộ truy vấn SQL phân tích dữ liệu tổng hợp phục vụ báo cáo và tra cứu dược phẩm. |
| 3 | Phân tích Kế hoạch thực thi | Xác định được các câu lệnh SQL bị nghẽn do Table Scan và lập danh sách các cột cần đánh chỉ mục. |
| 4 | Tối ưu hóa chỉ mục (Indexing) | Tạo thành công các chỉ mục B-Tree và Composite Index, giúp giảm đáng kể thời gian truy vấn trên RDS. |
| 5 | Đóng gói dữ liệu cho AI | Chuẩn hóa toàn bộ tập dữ liệu thông tin dược phẩm sang cấu trúc JSON sẵn sàng cho AI Chatbot tra cứu. |
| 6 | Tích hợp & Đo đạc hiệu năng | Kiểm thử thành công kết nối Lambda - RDS, đảm bảo tốc độ phản hồi nhanh và dữ liệu trả về chuẩn xác. |