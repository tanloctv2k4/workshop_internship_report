---
title : "Lambda Backend"
date : 2026-07-14
weight : 4
chapter : false
pre : " <b>5.4. </b> "
---

## AWS Lambda Overview

In the **PharmaCare AI** architecture, AWS Lambda is used to process backend business logic and AI-related functions following a serverless model.

The system includes three main Lambda functions:

- **Lambda Backend API:** Handles products, categories, user accounts, shopping carts, orders, and administrative functions.
- **Lambda Chatbot Handler:** Receives user questions, invokes Amazon Bedrock, and retrieves relevant data from Amazon OpenSearch to generate responses.
- **Lambda Indexing:** Reads documents from Amazon S3, generates embeddings using Amazon Bedrock, and stores vector data in Amazon OpenSearch.

The Lambda functions are deployed inside the **Private App Subnet** and privately connect to Amazon RDS, Amazon Bedrock, Amazon OpenSearch, and Amazon S3 through VPC Endpoints.

Lambda logs and operational metrics are sent to Amazon CloudWatch for monitoring and alerting.

![PharmaCare AI Overall Architecture](/workshop_internship_report/images/5-Workshop/5.1-Workshop-overview/dia.jpg)
