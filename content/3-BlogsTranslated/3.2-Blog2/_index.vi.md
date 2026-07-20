---
title: "Bài viết 2"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

# Tối ưu hóa bảo mật Kubernetes với Session Policies trong Amazon EKS Pod Identity

Trong quá trình tìm hiểu về Amazon EKS, mình có đọc được một bài viết giới thiệu cách tối ưu hóa việc quản lý quyền truy cập cho các ứng dụng chạy trên Kubernetes bằng cách kết hợp **Amazon EKS Pod Identity** và **Session Policies**. Đây là một giải pháp giúp đơn giản hóa việc phân quyền, giảm số lượng IAM Role cần quản lý và tăng cường bảo mật cho hệ thống.

Trước đây, việc cấp quyền cho từng Pod thường được thực hiện bằng **IAM Roles for Service Accounts (IRSA)**. Tuy nhiên, khi số lượng ứng dụng và microservices ngày càng nhiều thì cách làm này bộc lộ nhiều hạn chế trong quá trình vận hành.

---
  
## Những hạn chế của cơ chế IRSA

Theo mình tìm hiểu, cơ chế IRSA mang lại khả năng cấp quyền chi tiết cho từng Pod nhưng cũng gặp phải một số vấn đề như:

- Phải tạo rất nhiều IAM Role để đảm bảo nguyên tắc **Least Privilege**.
- Việc ánh xạ giữa Kubernetes Service Account và IAM Role phải thực hiện thủ công.
- Các nhóm phát triển thường có xu hướng dùng chung IAM Role có quyền rộng để giảm công sức quản lý, làm tăng nguy cơ mất an toàn thông tin.
- Khi hệ thống có nhiều microservices, việc kiểm tra và quản lý IAM Role trở nên khá phức tạp.

---

## Kiến trúc giải pháp

Giải pháp sử dụng **Amazon EKS Pod Identity** kết hợp với **Session Policies** để quản lý quyền truy cập theo mô hình ba lớp.

> *Hình 1. Kiến trúc Amazon EKS Pod Identity kết hợp Session Policies.*

![Kiến trúc giải pháp](imageblog2.jpg)

### 1. Lớp định danh (Identity Layer)

Các ứng dụng chạy dưới dạng Pod trong Amazon EKS.

Mỗi Pod sử dụng một **Kubernetes Service Account** để định danh thay vì tham chiếu trực tiếp đến IAM Role. Việc liên kết này được quản lý bởi **Amazon EKS Pod Identity Agent**.

### 2. Lớp chính sách và điều khiển (Policy & Control Layer)

Kubernetes Service Account sẽ được liên kết với một IAM Role thông qua **Pod Identity Association**.

Khi Pod cần truy cập tài nguyên AWS, **AWS Security Token Service (STS)** sẽ cấp thông tin xác thực tạm thời và đồng thời áp dụng **Session Policy**.

Quyền truy cập thực tế của Pod sẽ là phần giao giữa:

- Quyền của IAM Role.
- Quyền được quy định trong Session Policy.

Nhờ vậy, quyền của từng Pod được thu hẹp đúng theo nhu cầu sử dụng mà không cần tạo thêm nhiều IAM Role.

### 3. Lớp tài nguyên đích (Target Resource Layer)

Sau khi được cấp quyền, Pod có thể truy cập các dịch vụ AWS như:

- Amazon S3
- Amazon DynamoDB
- Amazon SQS

Các dịch vụ này chỉ nhận những yêu cầu đã được giới hạn quyền phù hợp mà không cần biết ứng dụng đang chạy trên Kubernetes.

---

## Điều mình thấy hay

Điều mình ấn tượng nhất là AWS đã tách riêng việc quản lý danh tính và quản lý quyền truy cập.

Thay vì phải tạo một IAM Role cho từng ứng dụng như trước đây, quản trị viên chỉ cần sử dụng một số lượng IAM Role phù hợp rồi kết hợp với **Session Policies** để giới hạn quyền cho từng Pod khi thực thi.

Cách làm này giúp hệ thống vừa dễ quản lý vừa đảm bảo tính bảo mật.

---

## Những điều mình học được

Sau khi tìm hiểu bài viết này, mình rút ra được một số kiến thức như sau:

- Amazon EKS Pod Identity giúp đơn giản hóa việc cấp quyền cho các ứng dụng chạy trên Kubernetes.
- Session Policies giúp giới hạn quyền truy cập ngay trong quá trình Pod thực thi.
- Có thể giảm đáng kể số lượng IAM Role cần quản lý.
- Việc quản lý quyền truy cập trở nên tập trung và dễ bảo trì hơn.
- Hệ thống vẫn đảm bảo tuân thủ nguyên tắc **Least Privilege** mà không làm tăng độ phức tạp trong quá trình vận hành.

---

## Kết luận

Qua bài viết này, mình hiểu thêm một giải pháp mới giúp quản lý quyền truy cập hiệu quả hơn trong Amazon EKS.

So với cơ chế IRSA truyền thống, việc kết hợp **Amazon EKS Pod Identity** và **Session Policies** không chỉ giúp giảm chi phí quản trị mà còn tăng cường bảo mật cho hệ thống Kubernetes. Đây là một giải pháp phù hợp với những hệ thống có nhiều ứng dụng hoặc microservices cần truy cập vào các dịch vụ AWS.

---

## Tài liệu tham khảo

AWS Containers Blog – **Optimize Kubernetes Security with Session Policies in Amazon EKS Pod Identity**

https://aws.amazon.com/blogs/containers/

Bài blog được đăng lên nhóm **AWS Study Group VN** - *Ngày 06-07-2026*

https://www.facebook.com/groups/awsstudygroupfcj/permalink/2206037506827876/?rdid=aJJTfyKFrzCzEko6#