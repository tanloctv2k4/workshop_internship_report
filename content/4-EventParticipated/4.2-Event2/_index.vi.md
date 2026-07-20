---
title: "Event 2"
date: 2026-06-06
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---

# Bài thu hoạch: "AWS First Cloud Journey: Từ Ảo hóa, Serverless đến AI trong Bảo mật"

Chào mọi người, tôi là thực tập sinh tại AWS. Vừa qua, tôi đã có cơ hội tham dự một sự kiện cực kỳ bổ ích mang tên "AWS First Cloud Journey". Đây không chỉ là một buổi chia sẻ lý thuyết mà là một hành trình thực tế đi qua các kiến trúc hạ tầng hiện đại, những thách thức khi triển khai Serverless và đặc biệt là sự giao thoa giữa AI và Bảo mật trên nền tảng AWS.

Dưới đây là phần tổng hợp và những bài học tâm đắc nhất mà tôi đúc kết được từ sự kiện này.

---

### Mục Đích Của Sự Kiện

Sự kiện được thiết kế để cung cấp một cái nhìn toàn cảnh và có chiều sâu về hệ sinh thái AWS, giúp người tham dự:
- Khám phá hành trình Cloud: Hiểu rõ con đường chuyển dịch từ hạ tầng truyền thống lên AWS Cloud.
- Mổ xẻ kiến trúc hiện đại: Phân tích chi tiết và so sánh giữa Virtualization (VM) và Containerization (Docker) để tối ưu hóa tài nguyên.
- Đối mặt với thực tế Serverless: Nhận diện các thách thức kỹ thuật và bài toán chi phí khi vận hành kiến trúc Serverless (AWS Lambda, DynamoDB).
- AI trong Bảo mật: Ứng dụng Machine Learning kết hợp với AWS WAF để xây dựng hệ thống phát hiện xâm nhập mạng (NIDS) thông minh.
- Định hướng nghề nghiệp: Lắng nghe chia sẻ thực tế về lộ trình phát triển từ Helpdesk/Sysadmin tiến tới DevOps.

### Danh Sách Diễn Giả & Thời Gian

Hành trình được dẫn dắt bởi các chuyên gia dày dạn kinh nghiệm:
- 09:20 - 10:00 | Nguyễn Quốc Bảo
- 10:00 - 10:35 | Nguyễn Huỳnh Quốc Bảo
- 10:35 - 10:50 | Việt Phát
- 10:50 - 11:10 | Lê Hoàng Gia Đại - Chuyên gia Bảo mật
    - Chủ đề: WAF + ML for Cyber Attack Detection
- 11:10 - 12:00 | Trần Trung Vinh
    - Chủ đề: Hành trình nghề nghiệp từ Sysadmin đến DevOps & Câu chuyện phỏng vấn tại Central Retail Group

*(Ghi chú: Các chủ đề kỹ thuật chuyên sâu về Container và Serverless được phân bổ trong các phiên sáng của các diễn giả Quốc Bảo, Huỳnh Quốc Bảo và Việt Phát).*

---

### Nội Dung Nổi Bật & Những Điểm Nhấn Kỹ Thuật

#### 1. Cuộc chiến Kiến trúc: Virtual Machines vs Containers

Đây là phần mở đầu cực kỳ ấn tượng, giúp làm rõ sự tiến hóa của hạ tầng.
- Bản chất khác biệt: VM (Máy ảo) mang tính "Heavyweight" vì mỗi VM chạy một hệ điều hành riêng (Guest OS) trên Hypervisor, dẫn đến khởi động chậm (vài phút) và tốn tài nguyên. Trong khi đó, Container (Docker) là "Lightweight", dùng chung Host OS, khởi động tính bằng mili-giây.
- Tối ưu hóa: Container giúp đóng gói ứng dụng (App + Bins/Lib) gọn nhẹ, mang lại hiệu năng Native, tiết kiệm RAM và cho phép chạy hàng trăm đến hàng ngàn container trên cùng một hạ tầng thay vì chỉ vài chục VM.

#### 2. Thách thức khi triển khai Serverless (AWS Lambda & DynamoDB)

Serverless không phải là "chiếc đũa thần" giải quyết mọi vấn đề. Phần này tập trung vào các bài học xương máu khi triển khai thực tế.
- Quản lý trạng thái (Stateless Lambda): AWS Lambda không lưu trữ dữ liệu trên memory giữa các request. Mọi trạng thái (ví dụ: Game state) đều phải được query và lưu trữ liên tục từ DynamoDB, đòi hỏi thiết kế kiến trúc khéo léo để tránh độ trễ.
- Bài toán chi phí (DynamoDB Scan Cost): Đây là một lưu ý cực kỳ quan trọng. Việc sử dụng lệnh ScanCommand để quét toàn bộ dữ liệu (VD: tìm player trong bảng) sẽ ngày càng chậm và đắt đỏ khi hệ thống scale up. Cần tối ưu bằng Query và Index.
- Kết nối chết (Stale Connections - GoneException): Xử lý triệt để các người chơi ngắt kết nối đột ngột để hệ thống Matchmaking không gửi message lãng phí.

#### 3. WAF & Machine Learning cho Network Intrusion Detection System (Lê Hoàng Gia Đại)

Đây là phiên đòi hỏi sự tập trung cao độ, kết hợp giữa kỹ thuật WAF và sức mạnh của AI để tối ưu khả năng bảo vệ trên môi trường Cloud.
- Giới hạn của Signature-based: Phân tích lý do tại sao các bộ luật dựa trên chữ ký (Signature) truyền thống của WAF là không đủ để chống lại các cuộc tấn công hiện đại (Zero-day, Behavior anomalies).
- Giải pháp NIDS với ML: Sử dụng Machine Learning để phân tích hành vi (Behavior-based detection). Mô hình được train trên tập dữ liệu chuẩn CSE-CIC-IDS2018.
- Workflow vận hành: Xây dựng Real-time dashboard để monitor, sau đó tương quan (Correlate) các dự đoán từ NIDS với các sự kiện của AWS WAF để nâng cao khả năng phát hiện mối đe dọa.

#### 4. Hành trình Nghề nghiệp: Từ Sysadmin đến DevOps (Trần Trung Vinh)

Phiên cuối cùng mang lại nhiều cảm xúc và định hướng rõ ràng cho sinh viên.
- Chuyển đổi tư duy (Mindset Shift): Cloud không chỉ là một công nghệ mới, nó là một "cách tư duy mới". Chuyển từ việc cài đặt thủ công sang tự động hóa (Infrastructure as Code).
- Kinh nghiệm phỏng vấn: Những bài học xương máu và chiến lược phát triển bản thân đúc kết từ quá trình phỏng vấn tại các tập đoàn lớn (Central Retail Group).

---

### Hình Ảnh Ghi Nhận Tại Sự Kiện

![So sánh Virtual Machines và Containers](image/event2.1.jpg)
![Nội dung Agenda WAF và Machine Learning](image/event2.2.jpg)
![Slide Network Intrusion Detection System trên AWS](image/event2.3.jpg)

---

### Những Gì Học Được (Key Takeaways)

#### Tư Duy Kiến Trúc & Cloud
- Right Tool for the Right Job: Không có công nghệ nào là hoàn hảo tuyệt đối. Tùy vào bài toán chi phí và hiệu năng để chọn VM hay Container; chọn thiết kế DB sao cho không bị dội chi phí (Scan vs Query).
- DevOps Mindset: Khái niệm về hạ tầng giờ đây đã chuyển thành "Code". Sự ổn định của hệ thống phụ thuộc vào khả năng tự động hóa và giám sát.

#### Tư Duy Bảo Mật Hiện Đại
- Hiểu sâu hơn về giới hạn của các Rule tĩnh trong Firewall/WAF. Bảo mật hiện đại bắt buộc phải có sự can thiệp của AI/ML để phân tích các pattern bất thường của mạng thay vì chỉ dựa vào các rule đã biết.

### Ứng Dụng Vào Công Việc & Học Tập (Call to Action)

- Đóng gói dự án cá nhân: Thay vì chạy các ứng dụng trực tiếp trên máy host gây xung đột môi trường, tôi sẽ bắt đầu viết Dockerfile để container hóa các web app của mình, phục vụ cho việc deploy lên server dễ dàng hơn.
- Nâng cấp tư duy Security: Kiến thức từ phiên về WAF + ML bổ trợ cực tốt cho định hướng nghiên cứu bảo mật. Tôi có thể áp dụng tư duy "Tương quan log" (Correlating events) này vào hệ thống SIEM để viết các custom rule hiệu quả hơn cho đồ án của mình.
- Kiểm soát chi phí Cloud: Nếu có triển khai các Demo nhỏ lên AWS trong tương lai, bài học về DynamoDB Scan Cost sẽ giúp tôi cẩn trọng hơn trong việc thiết kế cơ sở dữ liệu để không bị vượt quá giới hạn Free Tier.

---

### Trải nghiệm cá nhân về sự kiện

Sự kiện AWS First Cloud Journey thực sự là một trải nghiệm "mở mang tầm mắt", mang lại một góc nhìn đậm chất "Industry" (Thực tiễn công nghiệp) thay vì chỉ là lý thuyết sách vở.

- Về mặt kỹ thuật: Được nhìn thấy những kiến trúc thực tế mà các team AWS đang triển khai giúp tôi hình dung rõ ràng hơn về con đường trở thành một Kỹ sư Hệ thống/Bảo mật. Việc mổ xẻ những "điểm yếu" của Serverless (như Stateless Lambda) cho thấy sự chuyên sâu và thực tế từ các diễn giả.
- Về mặt định hướng: Session chia sẻ hành trình từ IT Helpdesk lên Sysadmin và DevOps của anh Trung Vinh truyền động lực rất lớn. Nó vạch ra một lộ trình rõ ràng, giúp sinh viên biết cần trang bị những bộ kỹ năng nào (từ OS, Network đến Cloud, Automation) để chuẩn bị cho các đợt phỏng vấn sắp tới.
- Không khí sự kiện: Chuyên nghiệp nhưng cởi mở. Các case study về ứng dụng AI vào WAF thật sự mở ra hướng đi mới cho các dự án an toàn thông tin mà tôi đang theo đuổi.

> **Tóm lại:** Sự kiện là cầu nối hoàn hảo giữa kiến thức Quản trị mạng/Bảo mật tôi đang học với các tiêu chuẩn Cloud thực tế của doanh nghiệp. Nó củng cố thêm quyết tâm của tôi trong việc làm chủ hạ tầng mạng và tích hợp AI vào quy trình bảo mật.