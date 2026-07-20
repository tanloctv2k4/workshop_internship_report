---
title : "Triển Khai Tự Động Hóa và Xác Thực"
date : 2026-07-14
weight : 3
chapter : false
pre : " <b>5.3. </b> "
---

#### Tổng quan triển khai tự động hóa và xác thực

Trong phần này, bạn sẽ triển khai cơ chế tự động hóa dữ liệu và hệ thống xác thực người dùng cho hệ thống **PharmaCare** trên Amazon Web Services (AWS).

Quá trình triển khai bao gồm hai thành phần chính:

+ **AWS Lambda Migration** dùng để tự động khởi tạo và cập nhật cấu trúc cơ sở dữ liệu PostgreSQL.
+ **Amazon Cognito** dùng để quản lý tài khoản, xác thực và phân quyền người dùng.

#### Triển khai Lambda Migration

AWS Lambda được sử dụng để tự động hóa quá trình khởi tạo cấu trúc cơ sở dữ liệu cho hệ thống PharmaCare.

+ Hàm Lambda `pharmacare-db-migration` được triển khai bên trong VPC.
+ Lambda kết nối đến Amazon RDS PostgreSQL thông qua địa chỉ private.
+ Lambda thực thi các tập lệnh SQL để tạo bảng và cập nhật cấu trúc cơ sở dữ liệu.
+ Thông tin đăng nhập cơ sở dữ liệu được lưu trữ trong AWS Secrets Manager.
+ IAM Role cho phép Lambda đọc thông tin kết nối từ Secrets Manager mà không cần hardcode mật khẩu trong mã nguồn.
+ Cấu hình Memory và Timeout của Lambda được điều chỉnh để phù hợp với quá trình thực thi migration.
+ Kết quả thực thi và lỗi của Lambda được ghi lại trong Amazon CloudWatch Logs.

Xem hướng dẫn triển khai chi tiết tại:

+ [5.3.1. Triển Khai Lambda Migration Cơ Sở Dữ Liệu](5.3.1-create-lambda/)

#### Triển khai xác thực người dùng bằng Amazon Cognito

Amazon Cognito được sử dụng làm dịch vụ quản lý danh tính tập trung cho hệ thống PharmaCare.

+ **Cognito User Pool** quản lý quá trình đăng ký, đăng nhập, xác minh tài khoản và khôi phục mật khẩu.
+ User Pool `pharmacare-user-pool` được sử dụng làm nguồn quản lý thông tin tài khoản người dùng.
+ **App Client** cung cấp Client ID để frontend kết nối với Cognito.
+ Các nhóm `Admin` và `Customer` được tạo để phân quyền người dùng theo vai trò.
+ Sau khi đăng nhập thành công, Cognito cấp JWT Token cho người dùng.
+ Frontend gửi JWT Token đến Amazon API Gateway khi gọi các API yêu cầu xác thực.
+ Backend kiểm tra thông tin trong token để xác định danh tính và quyền truy cập của người dùng.

Xem hướng dẫn triển khai chi tiết tại:

+ [5.3.2. Triển Khai Xác Thực Amazon Cognito](5.3.2-create-cognito/)

#### Nội dung triển khai

Trong chương này, bạn sẽ thực hiện các nội dung sau:

+ Triển khai Lambda Migration trong VPC.
+ Cấu hình quyền truy cập AWS Secrets Manager cho Lambda.
+ Kiểm tra kết nối từ Lambda đến Amazon RDS PostgreSQL.
+ Tạo Cognito User Pool và App Client.
+ Tạo nhóm người dùng `Admin` và `Customer`.
+ Kiểm tra quá trình đăng ký và đăng nhập người dùng.