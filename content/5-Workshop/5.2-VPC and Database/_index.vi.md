---
title : "Triển Khai Hạ Tầng Lõi (VPC and Database)"
date : 2026-07-14
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---
#### Triển khai Hạ tầng Lõi VPC và Database

Trong phần này, bạn sẽ triển khai và kiểm tra kiến trúc phân tán đa vùng (**Multi-AZ**) cho cơ sở dữ liệu quan hệ **Amazon RDS (Pharmacy Database)** và kho dữ liệu vector **Amazon OpenSearch Service (Vector Store)**.

Các tài nguyên này được triển khai bên trong các phân mạng riêng tư thuộc `VPC-Cloud 10.0.0.0/16`.

#### Tại sao nên sử dụng kiến trúc Multi-AZ trong Private Subnet?

+ **Đảm bảo tính sẵn sàng cao và khả năng chịu lỗi:** Cơ chế **Multi-AZ Replication** tự động đồng bộ và sao chép dữ liệu giữa hai vùng sẵn sàng `AZ-A` và `AZ-B`.

+ Khi xảy ra sự cố tại một Availability Zone, hệ thống có thể tự động chuyển hoạt động sang tài nguyên dự phòng tại Availability Zone còn lại, giúp hạn chế gián đoạn dịch vụ và giảm nguy cơ mất dữ liệu.

+ **Cô lập và bảo mật dữ liệu:** Cơ sở dữ liệu giao dịch **Amazon RDS** được triển khai trong các `RDS DB Subnet`, thuộc lớp mạng riêng tư và không cho phép truy cập trực tiếp từ Internet.

+ **Bảo vệ kho dữ liệu vector:** Amazon OpenSearch Service được triển khai trong các `OpenSearch VPC Subnet`, giúp giới hạn truy cập chỉ từ các tài nguyên được cấp phép bên trong VPC.

+ **Kiểm soát truy cập:** Security Group được sử dụng để kiểm soát kết nối từ Lambda đến Amazon RDS và Amazon OpenSearch Service thông qua các cổng dịch vụ cần thiết.

![Kiến trúc VPC và Database](/workshop_internship_report/images/5-Workshop/5.2-vpc-data/dia1.png)