---
title: "Bài viết 1"
date: 2026-07-07
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Data Masking trong Amazon RDS for Oracle

Trong quá trình thực tập và tìm hiểu về **AWS Database**, mình có đọc được một bài viết khá hay về **Data Masking trong Amazon RDS for Oracle**. Đây là một chủ đề mà trước đây mình chưa từng tìm hiểu, nên mình muốn chia sẻ lại những gì mình học được với mọi người.

Trước đây mình nghĩ rằng khi cần tạo môi trường **Development** hoặc **UAT** thì chỉ cần restore bản backup của **Production** là có thể sử dụng. Tuy nhiên, sau khi đọc bài viết này, mình mới nhận ra rằng cách làm đó có thể làm lộ rất nhiều dữ liệu nhạy cảm của khách hàng.

Ví dụ như:

- Họ và tên
- Email
- Số điện thoại
- Địa chỉ
- Các thông tin cá nhân khác

Nếu những dữ liệu này được đưa sang môi trường Development hoặc Testing mà không có biện pháp bảo vệ thì sẽ tiềm ẩn rất nhiều rủi ro về bảo mật.

---

## Data Masking là gì?

Theo mình hiểu, **Data Masking** là quá trình thay thế dữ liệu thật bằng dữ liệu giả nhưng vẫn giữ nguyên cấu trúc và định dạng của dữ liệu ban đầu. Nhờ đó, hệ thống vẫn có thể hoạt động bình thường trong quá trình phát triển và kiểm thử mà không làm lộ thông tin thật của khách hàng.

Ví dụ:

| Dữ liệu gốc | Sau khi Mask |
|-------------|--------------|
| Nguyễn Văn A | Customer001 |
| loc@gmail.com | user001@example.com |
| 0909123456 | 0909XXXXXX |

---

## Quy trình hoạt động

Dưới đây là sơ đồ mô tả quy trình Data Masking trên AWS.

> *Hình 1. Quy trình tự động thực hiện Data Masking trong Amazon RDS for Oracle.*

![Data Masking Workflow](imageblog1.png)

Sau khi xem sơ đồ trên, mình hiểu quy trình hoạt động như sau:

### 1. Amazon EventBridge Scheduler

Dịch vụ này sẽ tự động kích hoạt quy trình theo lịch đã cấu hình trước. Nhờ vậy, việc làm mới dữ liệu không cần phải thực hiện thủ công.

### 2. Restore Snapshot

Hệ thống sẽ lấy Snapshot từ cơ sở dữ liệu Production và khôi phục thành một Amazon RDS Instance tạm thời để tiếp tục xử lý.

### 3. AWS Secrets Manager

Thông tin đăng nhập cơ sở dữ liệu sẽ được lưu trong **AWS Secrets Manager** thay vì ghi trực tiếp trong mã nguồn hoặc script. Đây là cách làm an toàn và cũng là Best Practice mà AWS khuyến nghị.

### 4. Thực hiện Data Masking

Sau khi cơ sở dữ liệu được khôi phục, **AWS Systems Manager** sẽ chạy các script để che giấu những dữ liệu nhạy cảm như tên, email hoặc số điện thoại.

### 5. Tạo Snapshot mới

Sau khi quá trình Data Masking hoàn tất, hệ thống sẽ tạo một Snapshot mới chứa dữ liệu đã được che giấu.

### 6. Restore cho môi trường Development hoặc UAT

Cuối cùng, Snapshot đã được xử lý sẽ được sử dụng để tạo cơ sở dữ liệu dành cho Developer hoặc Tester, giúp quá trình phát triển và kiểm thử diễn ra an toàn hơn.

---

## Điều mình thấy hay

Điều mình ấn tượng nhất là AWS không chỉ hướng dẫn cách che giấu dữ liệu mà còn kết hợp nhiều dịch vụ để tự động hóa toàn bộ quy trình.

Các dịch vụ được sử dụng gồm:

- Amazon EventBridge Scheduler
- AWS Step Functions
- AWS Systems Manager
- AWS Secrets Manager
- Amazon RDS for Oracle

Mỗi dịch vụ đảm nhận một nhiệm vụ riêng và phối hợp với nhau để tạo thành một quy trình tự động, giúp giảm thao tác thủ công và hạn chế sai sót.

---

## Những điều mình học được

Sau khi tìm hiểu bài viết này, mình rút ra một số kiến thức như sau:

- Không nên sử dụng trực tiếp dữ liệu Production cho môi trường Development hoặc Testing.
- Data Masking giúp bảo vệ dữ liệu nhạy cảm nhưng vẫn đảm bảo dữ liệu có thể sử dụng để phát triển và kiểm thử.
- AWS Secrets Manager giúp quản lý thông tin đăng nhập an toàn hơn thay vì lưu trong mã nguồn hoặc script.
- Việc tự động hóa quy trình bằng các dịch vụ AWS giúp tiết kiệm thời gian và giảm nguy cơ xảy ra sai sót.

---

## Kết luận

Đây là một chủ đề khá mới đối với mình và cũng giúp mình hiểu thêm về cách AWS hỗ trợ bảo vệ dữ liệu trong các hệ thống thực tế.

Qua bài viết này, mình nhận thấy rằng khi xây dựng hệ thống trên nền tảng Cloud, việc bảo mật dữ liệu không chỉ dừng lại ở phân quyền hay mã hóa, mà còn cần đảm bảo dữ liệu được xử lý an toàn khi sử dụng trong các môi trường Development hoặc Testing.

Hy vọng bài chia sẻ này sẽ giúp mọi người có thêm một góc nhìn về **Data Masking trong Amazon RDS for Oracle** cũng như tầm quan trọng của việc bảo vệ dữ liệu trong quá trình phát triển phần mềm.

---

## Tài liệu tham khảo

AWS Database Blog – **Data Masking in Amazon RDS for Oracle**

https://aws.amazon.com/vi/blogs/database/data-masking-in-amazon-rds-for-oracle/

Bài blog được đăng lên nhóm **AWS Study Group VN** - *Ngày 7-7-2026*

https://www.facebook.com/groups/awsstudygroupfcj/permalink/2206764610088499/?rdid=6jucykm1JYcH5g00#