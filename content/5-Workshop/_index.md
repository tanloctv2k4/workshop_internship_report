---
title: "Workshop"
date: 2026-07-19
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

#### Overview

In the **PharmaCare AI** project, the system is designed using a Cloud Native architecture. The backend Lambda functions and AI Lambda functions are deployed inside the **Private App Subnets** of an Amazon VPC. These workloads do not need direct access to the public Internet. Instead, they privately connect to Amazon S3, Amazon Bedrock, Amazon OpenSearch Serverless, and Amazon RDS through internal AWS endpoints.

Using **VPC Endpoints** and **AWS PrivateLink** keeps service-to-service traffic within the AWS network, reduces the risk of data exposure, minimizes dependence on a NAT Gateway, and strengthens the overall security of the system.

The PharmaCare AI architecture uses two types of VPC Endpoints:

- **Gateway VPC Endpoint:** Used by the Indexing Lambda function to access the **S3 Knowledge Documents** bucket. The endpoint is associated with the route table of the Private Subnets, allowing the function to read documents from Amazon S3 without sending traffic through the Internet.

- **Interface VPC Endpoint:** Used by the AI Lambda functions to privately connect to **Amazon Bedrock Runtime** and **Amazon OpenSearch Serverless** through Elastic Network Interfaces inside the VPC. Service addresses are resolved through Private DNS.

The main application workflow is described below:

```text
User
    ↓
Route 53 → CloudFront → S3 Frontend
    ↓
API Gateway
    ↓
Backend Lambda in Private App Subnet
    ↓
Amazon RDS in Private DB Subnet
```

For the AI chatbot:

```text
S3 Knowledge Documents
    ↓ S3 Gateway Endpoint
Indexing Lambda
    ↓ Bedrock Runtime Interface Endpoint
Amazon Bedrock Embedding Model
    ↓
OpenSearch Serverless Vector Store
```

When a user submits a question, the **Chatbot Lambda** creates a vector embedding for the question through Amazon Bedrock, retrieves relevant documents from OpenSearch Serverless, and uses an AI model to generate an appropriate response. The entire process is monitored through Amazon CloudWatch. When an error occurs or a configured threshold is exceeded, CloudWatch Alarm sends a notification to the administrator through Amazon SNS.

#### Content

1. [Workshop Overview](5.1-Workshop-overview/)
2. [VPC and Database Preparation](5.2-VPC-and-Database/)
3. [Lambda and Amazon Cognito](5.3-Lambda-and-Cognitu/)
4. [Lambda Backend](5.4-Backend/)
5. [AI Chatbot](5.5-Chat-aiy/)
6. [Website Deployment](5.6-Deploy-web/)
7. [Monitoring, Backup, and Security for PharmaCare](5.7-Monitoring-backup-security/)
8. [Soure Code And Demo Web](5.8-SoureCode-Demo/)