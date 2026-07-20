---
title: "Proposal"
date: 2026-07-06
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
## Online Pharmacy Website & AI Medical Consultation Chatbot System based on AWS Serverless

### 1. Executive Summary 
PharmaCare AI is a smart online pharmacy management system designed to combine a pharmaceutical e-commerce platform with a virtual assistant (AI Chatbot). The project is deployed entirely on an AWS Serverless architecture, integrating RAG (Retrieval-Augmented Generation) technology to provide a seamless shopping experience alongside the ability to accurately and naturally answer medical and medication inquiries based on an internal knowledge base. The solution aims to optimize operational efficiency, reduce the workload for the pharmacist team, and enhance the end-user experience.

### 2. Problem Statement 
*Current Problem* Traditional pharmacies and basic online drugstores currently struggle to meet customer demands for quick, accurate 24/7 information regarding medical conditions, side effects, or drug interactions. Manual consultation overloads pharmacists, while systems relying on physical servers (or traditional virtual machines) often incur high maintenance costs and are difficult to scale during traffic spikes.

*Solution* PharmaCare AI solves this problem by building a 100% Serverless architecture on AWS. The Frontend (ReactJS) is delivered at high speed via CloudFront and S3. The Backend handles logic using API Gateway and Lambda, securely connecting to a relational database RDS (Multi-AZ) isolated within a VPC. The highlight of the solution is the AI Chatbot system utilizing Amazon Bedrock combined with Amazon OpenSearch Service (Vector Store) under the RAG model. This allows the chatbot to retrieve specialized documents from S3, generating contextual, accurate responses that strictly adhere to the pharmacy's actual data.

*Benefits and Return on Investment (ROI)* The system automates level-1 customer care (answering FAQs, looking up medication information), saving significant time and personnel costs. Utilizing a Serverless architecture (pay-as-you-go pricing) eliminates idle server costs. The RDS Multi-AZ mechanism ensures High Availability, preventing any disruption to business operations.

### 3. Solution Architecture 
The system is designed into 5 distinct layers to optimize security, performance, and maintainability.

![AWS Architecture - Pharmacare AI](/images/5-Workshop/5.1-Workshop-overview/dia02.jpg)

*AWS Services Used* - *Amazon Route 53 & CloudFront*: DNS management and global static content delivery (CDN).
- *Amazon S3*: Hosting the static Frontend (Web UI) and raw medical knowledge documents (Knowledge Docs).
- *AWS WAF*: Web application firewall protecting against common web exploits.
- *Amazon Cognito*: Identity management, user login, and JWT Token issuance.
- *Amazon API Gateway & AWS Lambda*: Backend API logic processing and Chatbot integration.
- *Amazon RDS (Multi-AZ)*: Structured data storage (products, orders, users) within a Private VPC.
- *Amazon OpenSearch Service*: Vector database (Vector Store) for semantic search serving the Chatbot.
- *Amazon Bedrock*: LLM (Large Language Model) platform to generate Chatbot responses.
- *AWS Secrets Manager*: Secure management of Database credentials.
- *CloudWatch, SNS, IAM, Backup*: Monitoring, alerting, access management, and data backup.

*Component Design (Main Flows)* - *Frontend Layer*: Users access via Route 53, load the web from S3/CloudFront, authenticate via Cognito, and are protected by WAF.
- *Backend API Layer*: API Gateway routes requests to Lambda. Lambda calls Secrets Manager to retrieve credentials and connects to RDS to process business logic (sales, orders).
- *Database Layer (VPC)*: Sensitive data is completely isolated in a Private Subnet with a Multi-AZ fallback mechanism.
- *AI Chatbot / RAG Layer*: 
  - *Indexing Flow*: Documents from S3 -> Lambda Indexing -> Bedrock (Embedding) -> OpenSearch.
  - *Query Flow*: User asks a question -> Lambda Chatbot -> Retrieves context from OpenSearch -> Bedrock generates the answer -> Returns to User.

### 4. Technical Implementation 
*Implementation Phases* The project is divided into 4 main phases:
1. *Infrastructure & Networking*: Configure VPC, Private/Public Subnets, deploy RDS Multi-AZ, and OpenSearch.
2. *Data & Knowledge Layer Development*: Create S3 buckets, set up the document Indexing pipeline (Lambda + Bedrock) feeding into OpenSearch.
3. *API & Business Logic Construction (Backend)*: Write code for Lambda functions handling orders, products, and the Chatbot Handler. Configure API Gateway and Cognito.
4. *Frontend Integration & Security*: Build the ReactJS app, push to S3, and distribute via CloudFront. Set up WAF, CloudWatch Alarms, and IAM Roles.

*Technical Requirements* - *Frontend*: ReactJS, RESTful API communication.
- *Backend*: Programming languages supported by AWS Lambda (Node.js/Python), using AWS SDK (Boto3/AWS SDK for JS) to interact across services.
- *AI/ML*: Solid understanding of Embeddings, Vector Databases (OpenSearch), Prompt Engineering, and Amazon Bedrock API usage.
- *Infrastructure*: Use AWS CDK or Terraform to automate infrastructure deployment (IaC).

### 5. Roadmap & Milestones 
- *Week 1 (Design & Foundation)*: Draw detailed architecture, set up AWS environments (VPC, IAM, S3). Finalize Database schema on RDS.
- *Week 2 (Backend & RAG Pipeline)*: Program core APIs. Successfully build the document processing flow (Indexing) and Chatbot queries using Amazon Bedrock.
- *Week 3 (Frontend & Final Integration)*: Develop the web interface, integrate Cognito authentication, and connect the Chatbot UI with the API.
- *Week 4 (Testing, Optimization & Handover)*: Run stress-tests, verify WAF security, set up CloudWatch/SNS alerting systems, and write handover documentation.

### 6. Budget Estimation 
The system is designed with a Serverless approach; however, due to architectural requirements utilizing RDS Multi-AZ and OpenSearch (services requiring underlying running instances), costs are divided into two main categories. The prices below are estimated based on minimal configurations (suitable for Dev/Test environments) in a standard region (e.g., us-east-1 or ap-southeast-1).

*Fixed Infrastructure Costs (Monthly Estimate)*
- **Amazon RDS (Multi-AZ): ~$30.00 - $35.00/month**. (Using a `db.t3.micro` instance with basic storage. This is the most expensive component as it maintains 2 database instances running in parallel across 2 different Availability Zones for redundancy).
- **Amazon OpenSearch Service: ~$25.00 - $28.00/month**. (Using 1 `t3.small.search` instance and 10GB EBS as the Vector Store for AI data).
- **AWS WAF: ~$6.00/month**. ($5.00 fixed fee for 1 Web ACL and ~$1.00 for custom frontend protection rules).
- **Amazon Route 53: ~$0.50/month**. (Maintenance fee for 1 Hosted Zone).
- **AWS Secrets Manager: ~$0.40/month**. (Storage fee for 1 database credential secret).
- **TOTAL FIXED COSTS: ~ $61.90 - $69.90/month**.

*Variable Costs (Serverless & Pay-as-you-go)*
- **Amazon Bedrock: ~$2.00 - $5.00/month**. (Costs based on input/output Tokens. For a test project scale, query volume is relatively low).
- **AWS Lambda, API Gateway, S3, CloudFront, Cognito: ~$0.00 - $2.00/month**. (By leveraging the AWS Free Tier with millions of free requests per month, the team incurs almost no cost for this architectural layer).
- **TOTAL VARIABLE COSTS: ~ $2.00 - $7.00/month**.

**=> ESTIMATED TOTAL COST: Approximately $65.00 - $75.00/month (If running continuously 24/7).**

*Feasibility Analysis with a $200 Budget*
With a projected budget of $200, the PharmaCare AI system can sustain operations under the following scenarios:
**Scenario 1 - Full-time Operation (24/7):** If the team leaves all services (especially RDS and OpenSearch) running non-stop, the system will consume about $75/month. The $200 budget will last for **about 2.5 to nearly 3 months**. This duration is just enough to cover the entire development, testing, and final presentation phases.
**Scenario 2 - Optimized Start/Stop (Recommended):** During the development phase (Month 1 to Month 3), the team can write automated scripts (or use Lambda cronjobs) to **temporarily stop** the Amazon RDS and OpenSearch instances at night or on non-coding days. This practice can reduce fixed infrastructure costs by up to 50-60%. Consequently, the monthly cost drops to around $30 - $40, allowing the $200 budget to stretch up to **5 - 6 months**.

*(The team will set up AWS Budgets and CloudWatch Billing Alarms at $50, $100, and $150 milestones to proactively monitor and prevent unexpected charges).*

### 7. Risk Assessment 
*Risk Matrix* - *Uncontrolled AWS Costs*: High impact, medium probability (especially with RDS and OpenSearch).
- *AI Chatbot Hallucination*: High impact, medium probability (providing incorrect medical information).
- *Database Credential Leakage*: Critical impact, low probability.

*Mitigation Strategies* - *Costs*: Establish automated budget alerts (AWS Budgets). Pause RDS/OpenSearch instances during idle times if 24/7 uptime is not required for coding.
- *AI*: Optimize the System Prompt on Bedrock, strictly limit the response data source to OpenSearch (Strict RAG), and consistently display a warning: "This Chatbot does not replace a doctor."
- *Security*: Use AWS Secrets Manager, never hardcode credentials in the source code, and apply the Principle of Least Privilege for IAM Roles.

### 8. Expected Outcomes 
*Technical Improvements*: A highly stable pharmaceutical e-commerce website with fast response times powered by CloudFront. A smart AI Chatbot with strong bilingual comprehension, capable of consulting and accurately extracting information from internal medical documents.
*Long-term Value*: An infrastructure system that complies with modern cloud design principles: Highly secure, automatically scalable, and easily maintainable, creating a solid foundation for future real-world business expansion.