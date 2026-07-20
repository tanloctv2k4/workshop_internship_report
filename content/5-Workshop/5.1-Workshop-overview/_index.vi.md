---
title : "Giới thiệu"
date : 2026-07-14
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Giới thiệu về PharmaCare AI

**PharmaCare AI** là hệ thống nhà thuốc trực tuyến được triển khai trên nền tảng Amazon Web Services (AWS), kết hợp giữa website thương mại điện tử và chatbot trí tuệ nhân tạo. Hệ thống hỗ trợ người dùng đăng ký, đăng nhập, xem danh sách thuốc, tìm kiếm sản phẩm, quản lý giỏ hàng và tạo đơn hàng trực tuyến. Bên cạnh đó, chatbot AI giúp người dùng tra cứu thông tin dược phẩm dựa trên các tài liệu đã được lưu trữ trong hệ thống. Kiến trúc được xây dựng theo hướng serverless nhằm giảm chi phí vận hành, tự động mở rộng khi lưu lượng truy cập tăng và giúp các thành phần frontend, backend, cơ sở dữ liệu và chatbot có thể được quản lý độc lập.

#### Tổng quan kiến trúc

Website frontend của PharmaCare AI được phát triển bằng ReactJS, lưu trữ trong Amazon S3 và phân phối đến người dùng thông qua Amazon CloudFront. Amazon Route 53 thực hiện phân giải tên miền và chuyển yêu cầu truy cập đến CloudFront, trong khi AWS WAF được sử dụng để lọc các yêu cầu bất thường và hạn chế các cuộc tấn công phổ biến vào ứng dụng web. Các yêu cầu xử lý dữ liệu từ frontend được gửi đến Amazon API Gateway thông qua giao thức HTTPS. Kiến trúc này giúp tăng tốc độ truy cập, hỗ trợ mở rộng hệ thống và hạn chế việc công khai trực tiếp các tài nguyên backend ra Internet.

#### Xác thực và Backend API

Amazon Cognito được sử dụng để quản lý tài khoản, đăng ký, đăng nhập, xác minh người dùng và cấp JWT Token sau khi xác thực thành công. Frontend gửi JWT Token kèm theo các yêu cầu cần xác thực đến Amazon API Gateway. API Gateway sử dụng Cognito JWT Authorizer để kiểm tra token trước khi chuyển yêu cầu đến AWS Lambda. Lambda Backend API xử lý các nghiệp vụ như quản lý sản phẩm, giỏ hàng, đơn hàng, tài khoản người dùng và tồn kho. Dữ liệu nghiệp vụ được lưu trữ trong Amazon RDS và chỉ được truy cập bởi các tài nguyên đã được cấp quyền bên trong VPC.

#### Chatbot AI

Lambda Chatbot Handler tiếp nhận câu hỏi của người dùng từ Amazon API Gateway và thực hiện quy trình xử lý chatbot theo mô hình Retrieval-Augmented Generation (RAG). Câu hỏi được chuyển thành vector embedding thông qua mô hình embedding của Amazon Bedrock. Amazon OpenSearch Service tìm kiếm các đoạn tài liệu có nội dung liên quan nhất trong Vector Store. Những nội dung tìm được được sử dụng làm ngữ cảnh cho mô hình ngôn ngữ lớn trên Amazon Bedrock. Câu trả lời cuối cùng được trả về Lambda, chuyển qua API Gateway và hiển thị trên giao diện chatbot của người dùng.

#### Nội dung chính của workshop

Trong workshop này, bạn sẽ triển khai VPC, Private Subnet, Security Group và các VPC Endpoint cần thiết cho hệ thống. Amazon RDS Multi-AZ được sử dụng để lưu trữ dữ liệu nghiệp vụ, trong khi Amazon OpenSearch Service được triển khai làm Vector Store cho chatbot AI. Workshop cũng hướng dẫn cấu hình Amazon Cognito, API Gateway, Lambda Backend API và xây dựng chatbot RAG bằng Amazon Bedrock, Amazon S3 và OpenSearch. Cuối cùng, các dịch vụ Amazon CloudFront, Route 53, AWS WAF, CloudWatch và Amazon SNS được cấu hình để phân phối nội dung, bảo vệ, giám sát và gửi cảnh báo cho toàn bộ hệ thống.

#### Kiến trúc tổng thể PharmaCare AI

![Kiến trúc tổng thể PharmaCare AI](/images/5-Workshop/5.1-Workshop-overview/dia02.jpg)