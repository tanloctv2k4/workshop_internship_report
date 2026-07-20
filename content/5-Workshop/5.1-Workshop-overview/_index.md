---
title : "Introduction"
date : 2026-07-14
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Introduction to PharmaCare AI

**PharmaCare AI** is an online pharmacy system deployed on Amazon Web Services (AWS), combining an e-commerce website with an artificial intelligence chatbot. The system allows users to register, log in, browse and search for medicines, manage their shopping carts, and place online orders. In addition, the AI chatbot helps users retrieve pharmaceutical information from documents stored in the system. The architecture follows a serverless approach to reduce operational costs, automatically scale with increasing traffic, and allow the frontend, backend, database, and chatbot components to be managed independently.

#### Architecture overview

The PharmaCare AI frontend is developed with ReactJS, hosted in Amazon S3, and delivered to users through Amazon CloudFront. Amazon Route 53 performs domain name resolution and directs user requests to CloudFront, while AWS WAF filters suspicious requests and protects the web application against common attacks. Data processing requests from the frontend are sent to Amazon API Gateway over HTTPS. This architecture improves website performance, supports system scalability, and prevents backend resources from being directly exposed to the public Internet.

#### Authentication and Backend API

Amazon Cognito is used to manage user accounts, registration, login, user verification, and JWT token issuance after successful authentication. The frontend includes the JWT token in requests sent to protected Amazon API Gateway routes. API Gateway uses a Cognito JWT Authorizer to validate the token before forwarding the request to AWS Lambda. The Lambda Backend API handles business functions such as product management, shopping carts, orders, user accounts, and inventory. Business data is stored in Amazon RDS and can only be accessed by authorized resources inside the VPC.

#### AI Chatbot

The Lambda Chatbot Handler receives user questions from Amazon API Gateway and processes them using a Retrieval-Augmented Generation (RAG) workflow. Each question is converted into a vector embedding by an Amazon Bedrock embedding model. Amazon OpenSearch Service searches the Vector Store for the most relevant document sections. The retrieved information is then provided as context to a large language model hosted on Amazon Bedrock. The final answer is returned through Lambda and API Gateway and displayed in the chatbot interface.

#### Main workshop contents

In this workshop, you will deploy the VPC, private subnets, security groups, and VPC endpoints required by the system. Amazon RDS Multi-AZ will be used to store business data, while Amazon OpenSearch Service will be deployed as the Vector Store for the AI chatbot. The workshop also covers the configuration of Amazon Cognito, API Gateway, the Lambda Backend API, and the development of a RAG chatbot using Amazon Bedrock, Amazon S3, and OpenSearch. Finally, Amazon CloudFront, Route 53, AWS WAF, CloudWatch, and Amazon SNS will be configured to distribute content, protect the application, monitor the infrastructure, and deliver system alerts.

#### PharmaCare AI overall architecture

![PharmaCare AI Overall Architecture](/workshop_internship_report/images/5-Workshop/5.1-Workshop-overview/dia02.jpg)
