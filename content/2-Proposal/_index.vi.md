---
title: "Bản kế hoạch"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
## Website Nhà Thuốc Online & Hệ Thống Chatbot AI Tư Vấn Y Tế dựa trên AWS Serverless

### 1. Tóm tắt điều hành 
PharmaCare AI là một hệ thống quản lý nhà thuốc trực tuyến thông minh, được thiết kế để kết hợp nền tảng thương mại điện tử dược phẩm với trợ lý ảo (AI Chatbot). Dự án được triển khai hoàn toàn trên kiến trúc Serverless của AWS, tích hợp công nghệ RAG (Retrieval-Augmented Generation) nhằm cung cấp trải nghiệm mua sắm mượt mà và khả năng giải đáp, tư vấn thông tin y tế, thuốc men tự nhiên, chính xác dựa trên kho tri thức nội bộ. Giải pháp hướng tới việc tối ưu hóa hiệu suất vận hành, giảm tải cho đội ngũ dược sĩ và nâng cao trải nghiệm người dùng cuối.

### 2. Tuyên bố vấn đề 
*Vấn đề hiện tại* Các nhà thuốc truyền thống và các website bán thuốc cơ bản hiện nay đang gặp khó khăn trong việc đáp ứng nhu cầu tra cứu thông tin y khoa, tác dụng phụ, hoặc tương tác thuốc từ khách hàng một cách nhanh chóng và chính xác 24/7. Việc tư vấn thủ công làm quá tải đội ngũ dược sĩ, trong khi các hệ thống dùng máy chủ vật lý (hoặc máy ảo truyền thống) thường tốn kém chi phí duy trì, khó mở rộng khi lượng truy cập tăng đột biến.

*Giải pháp* PharmaCare AI giải quyết vấn đề này bằng cách xây dựng một kiến trúc 100% Serverless trên AWS. Frontend (ReactJS) được phân phối siêu tốc qua CloudFront và S3. Backend xử lý logic bằng API Gateway và Lambda, kết nối an toàn với cơ sở dữ liệu quan hệ RDS (Multi-AZ) nằm kín trong VPC. Điểm nhấn của giải pháp là hệ thống Chatbot AI ứng dụng Amazon Bedrock kết hợp cùng Amazon OpenSearch Service (Vector Store) theo mô hình RAG. Điều này cho phép chatbot truy xuất tài liệu chuyên ngành từ S3, tạo ra câu trả lời có ngữ cảnh, chính xác và bám sát dữ liệu thực tế của nhà thuốc.

*Lợi ích và hoàn vốn đầu tư (ROI)* Hệ thống giúp tự động hóa khâu chăm sóc khách hàng mức độ 1 (giải đáp FAQ, tra cứu thông tin thuốc), tiết kiệm đáng kể thời gian và chi phí nhân sự. Việc sử dụng kiến trúc Serverless (trả tiền theo mức sử dụng - Pay-as-you-go) giúp loại bỏ chi phí máy chủ nhàn rỗi. Cơ chế RDS Multi-AZ đảm bảo hệ thống luôn sẵn sàng cao (High Availability), không gián đoạn hoạt động kinh doanh.

### 3. Kiến trúc giải pháp 
Hệ thống được thiết kế chia thành 5 lớp (Layers) rõ ràng để tối ưu hóa bảo mật, hiệu suất và dễ dàng bảo trì.

![AWS Architecture - Pharmacare AI](/workshop_internship_report/images/5-Workshop/5.1-Workshop-overview/dia.jpg)

*Dịch vụ AWS sử dụng* - *Amazon Route 53 & CloudFront*: Quản lý DNS và phân phối nội dung tĩnh (CDN) toàn cầu.
- *Amazon S3*: Lưu trữ Frontend tĩnh (Web UI) và tài liệu tri thức y khoa thô (Knowledge Docs).
- *AWS WAF*: Tường lửa bảo vệ ứng dụng web khỏi các cuộc tấn công phổ biến.
- *Amazon Cognito*: Quản lý định danh, đăng nhập và cấp phát JWT Token.
- *Amazon API Gateway & AWS Lambda*: Xử lý logic API backend và tích hợp Chatbot.
- *Amazon RDS (Multi-AZ)*: Lưu trữ dữ liệu có cấu trúc (sản phẩm, đơn hàng, user) trong Private VPC.
- *Amazon OpenSearch Service*: Cơ sở dữ liệu Vector (Vector Store) phục vụ tìm kiếm ngữ nghĩa cho Chatbot.
- *Amazon Bedrock*: Nền tảng LLM (Large Language Model) tạo phản hồi cho Chatbot.
- *AWS Secrets Manager*: Quản lý bảo mật thông tin đăng nhập Database.
- *CloudWatch, SNS, IAM, Backup*: Giám sát, cảnh báo, quản lý quyền hạn và sao lưu dữ liệu.

*Thiết kế thành phần (Các luồng chính)* - *Frontend Layer*: Người dùng truy cập qua Route 53, tải web từ S3/CloudFront, xác thực qua Cognito và được bảo vệ bởi WAF.
- *Backend API Layer*: API Gateway định tuyến request đến Lambda. Lambda gọi Secrets Manager lấy thông tin và kết nối vào RDS để xử lý nghiệp vụ bán hàng.
- *Database Layer (VPC)*: Dữ liệu nhạy cảm bị cô lập hoàn toàn trong Private Subnet với cơ chế dự phòng Multi-AZ.
- *AI Chatbot / RAG Layer*: 
  - *Luồng Index*: Tài liệu từ S3 -> Lambda Indexing -> Bedrock (Embedding) -> OpenSearch.
  - *Luồng Query*: User hỏi -> Lambda Chatbot -> Tìm kiếm ngữ cảnh tại OpenSearch -> Bedrock sinh câu trả lời -> Trả về User.

### 4. Triển khai kỹ thuật 
*Các giai đoạn triển khai* Dự án được chia làm 4 giai đoạn chính:
1. *Thiết lập Hạ tầng & Mạng (Infrastructure & Networking)*: Cấu hình VPC, Private/Public Subnets, triển khai RDS Multi-AZ và OpenSearch.
2. *Phát triển Lớp Dữ liệu & Tri thức (Data & Knowledge)*: Tạo các bucket S3, thiết lập luồng Indexing tài liệu (Lambda + Bedrock) đưa vào OpenSearch.
3. *Xây dựng API & Logic nghiệp vụ (Backend)*: Viết code cho các hàm Lambda xử lý đơn hàng, sản phẩm và Chatbot Handler. Cấu hình API Gateway và Cognito.
4. *Tích hợp Frontend & Bảo mật (Frontend & Security)*: Build ReactJS đẩy lên S3, phân phối qua CloudFront. Thiết lập WAF, CloudWatch Alarms và IAM Roles.

*Yêu cầu kỹ thuật* - *Frontend*: ReactJS, giao tiếp API qua chuẩn RESTful.
- *Backend*: Ngôn ngữ hỗ trợ AWS Lambda (Node.js/Python), sử dụng AWS SDK (Boto3/AWS SDK for JS) để tương tác giữa các dịch vụ.
- *AI/ML*: Nắm vững khái niệm Embedding, Vector Database (OpenSearch), Prompt Engineering và cách sử dụng Amazon Bedrock API.
- *Hạ tầng*: Sử dụng AWS CDK hoặc Terraform để tự động hóa việc triển khai hạ tầng (IaC).

### 5. Lộ trình & Mốc triển khai 
- *Tuần 1 (Thiết kế & Nền tảng)*: Vẽ kiến trúc chi tiết, thiết lập môi trường AWS (VPC, IAM, S3). Hoàn thiện Database schema trên RDS.
- *Tuần 2 (Backend & RAG Pipeline)*: Lập trình các API cốt lõi. Xây dựng thành công luồng xử lý tài liệu (Indexing) và truy vấn Chatbot với Amazon Bedrock.
- *Tuần 3 (Frontend & Tích hợp hoàn thiện)*: Phát triển giao diện web, tích hợp xác thực Cognito, ghép nối giao diện Chatbot với API.
- *Tuần 4 (Kiểm thử, Tối ưu & Bàn giao)*: Chạy stress-test, kiểm tra bảo mật WAF, thiết lập hệ thống cảnh báo CloudWatch/SNS và viết tài liệu bàn giao.

### 6. Ước tính ngân sách 
Hệ thống được thiết kế theo hướng Serverless, tuy nhiên do yêu cầu kiến trúc sử dụng RDS Multi-AZ và OpenSearch (những dịch vụ yêu cầu instance chạy nền), chi phí sẽ được chia thành hai nhóm chính. Các mức giá dưới đây được ước tính dựa trên cấu hình tối thiểu (phù hợp cho môi trường Dev/Test) tại khu vực tiêu chuẩn (ví dụ: us-east-1 hoặc ap-southeast-1).

*Chi phí hạ tầng cố định (Ước tính hàng tháng)*
- **Amazon RDS (Multi-AZ): ~$30.00 - $35.00/tháng**. (Sử dụng cấu hình `db.t3.micro` cùng dung lượng lưu trữ cơ bản. Đây là khoản tốn kém nhất vì phải duy trì 2 database instance chạy song song ở 2 Availability Zone khác nhau để đảm bảo dự phòng).
- **Amazon OpenSearch Service: ~$25.00 - $28.00/tháng**. (Sử dụng 1 instance `t3.small.search` và 10GB EBS để làm Vector Store lưu trữ dữ liệu AI).
- **AWS WAF: ~$6.00/tháng**. ($5.00 phí cố định cho 1 Web ACL và khoảng $1.00 cho các custom rules bảo vệ frontend).
- **Amazon Route 53: ~$0.50/tháng**. (Phí duy trì 1 Hosted Zone cho tên miền).
- **AWS Secrets Manager: ~$0.40/tháng**. (Phí lưu trữ 1 secret credential cho Database).
- **TỔNG CHI PHÍ CỐ ĐỊNH: ~ $61.90 - $69.90/tháng**.

*Chi phí biến đổi (Serverless & Pay-as-you-go)*
- **Amazon Bedrock: ~$2.00 - $5.00/tháng**. (Chi phí tính theo số lượng Token đầu vào/đầu ra. Ở quy mô đồ án thử nghiệm, lượng truy vấn không quá lớn).
- **AWS Lambda, API Gateway, S3, CloudFront, Cognito: ~$0.00 - $2.00/tháng**. (Nhờ tận dụng chính sách AWS Free Tier với hàng triệu lượt request miễn phí mỗi tháng, nhóm gần như không mất tiền cho lớp kiến trúc này).
- **TỔNG CHI PHÍ BIẾN ĐỔI: ~ $2.00 - $7.00/tháng**.

**=> TỔNG CHI PHÍ ƯỚC TÍNH: Khoảng $65.00 - $75.00/tháng (Nếu bật liên tục 24/7).**

*Phân tích tính khả thi với ngân sách $200*
Với tổng ngân sách dự kiến là $200, hệ thống PharmaCare AI có thể duy trì hoạt động theo các kịch bản sau:
**Kịch bản 1 - Chạy toàn thời gian (24/7):** Nếu nhóm để tất cả các dịch vụ (đặc biệt là RDS và OpenSearch) chạy liên tục không ngừng, hệ thống sẽ tiêu tốn khoảng $75/tháng. Ngân sách $200 sẽ duy trì được **khoảng 2.5 đến gần 3 tháng**. Thời gian này vừa đủ để cover toàn bộ quá trình phát triển, kiểm thử và bảo vệ trước hội đồng.
**Kịch bản 2 - Tối ưu hóa bật/tắt (Khuyên dùng):** Trong quá trình phát triển (Tháng 1 đến Tháng 3), nhóm có thể viết các script tự động (hoặc dùng Lambda cronjob) để **tắt tạm thời (Stop)** các instance của Amazon RDS và OpenSearch vào ban đêm hoặc những ngày không code. Việc này có thể giảm tới 50-60% chi phí hạ tầng cố định. Khi đó, chi phí hàng tháng chỉ còn khoảng $30 - $40, giúp ngân sách $200 có thể kéo dài hệ thống lên đến **5 - 6 tháng**.

*(Nhóm sẽ thiết lập AWS Budget và CloudWatch Billing Alarm ở các mốc $50, $100, $150 để chủ động theo dõi và tránh phát sinh chi phí ngoài ý muốn).*

### 7. Đánh giá rủi ro 
*Ma trận rủi ro* - *Chi phí AWS vượt kiểm soát*: Ảnh hưởng cao, xác suất trung bình (đặc biệt với RDS và OpenSearch).
- *AI Chatbot bị "ảo giác" (Hallucination)*: Ảnh hưởng cao, xác suất trung bình (trả lời sai thông tin y tế).
- *Lộ lọt thông tin kết nối Database*: Ảnh hưởng nghiêm trọng, xác suất thấp.

*Chiến lược giảm thiểu* - *Chi phí*: Thiết lập cảnh báo ngân sách tự động (AWS Budgets). Tạm dừng các instance RDS/OpenSearch khi không code nếu không cần online 24/7.
- *AI*: Tối ưu hóa System Prompt trên Bedrock, giới hạn nguồn dữ liệu trả lời chỉ nằm trong OpenSearch (RAG strict), luôn hiển thị cảnh báo "Chatbot không thay thế bác sĩ".
- *Bảo mật*: Sử dụng AWS Secrets Manager, không bao giờ hardcode credentials trong source code, áp dụng nguyên tắc đặc quyền tối thiểu (Least Privilege) cho IAM Role.

### 8. Kết quả kỳ vọng 
*Cải tiến kỹ thuật*: Một website thương mại điện tử nhà thuốc hoạt động ổn định, tốc độ phản hồi nhanh nhờ CloudFront. Chatbot AI thông minh, hiểu tiếng Việt tốt, có khả năng tư vấn và trích xuất