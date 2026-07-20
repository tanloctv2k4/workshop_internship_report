---
title : "Deploy Automation and Authentication"
date : 2026-07-14
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

#### Overview of automation and authentication deployment

In this section, you will deploy the data automation process and user authentication system for the **PharmaCare** platform on Amazon Web Services (AWS).

The deployment includes two main components:

+ **AWS Lambda Migration** is used to automatically initialize and update the PostgreSQL database schema.
+ **Amazon Cognito** is used to manage user accounts, authentication, and authorization.

#### Deploy Lambda Migration

AWS Lambda is used to automate the database initialization process for the PharmaCare system.

+ The `pharmacare-db-migration` Lambda function is deployed inside the VPC.
+ Lambda connects to the Amazon RDS PostgreSQL database through a private endpoint.
+ Lambda executes SQL scripts to create tables and update the database schema.
+ Database credentials are securely stored in AWS Secrets Manager.
+ The Lambda IAM Role allows the function to retrieve database credentials from Secrets Manager without hardcoding passwords in the source code.
+ The Lambda Memory and Timeout settings are configured to support the migration process.
+ Execution results and errors are recorded in Amazon CloudWatch Logs.

See the detailed deployment guide:

+ [5.3.1. Deploy Lambda Database Migration](5.3.1-create-lambda/)

#### Deploy user authentication with Amazon Cognito

Amazon Cognito is used as the centralized identity management service for the PharmaCare system.

+ **Cognito User Pool** manages user registration, login, account verification, and password recovery.
+ The `pharmacare-user-pool` User Pool is used as the primary source for managing user account information.
+ The **App Client** provides a Client ID that allows the frontend application to connect to Cognito.
+ The `Admin` and `Customer` groups are created to implement role-based access control.
+ After a successful login, Cognito issues JWT tokens to the authenticated user.
+ The frontend sends the JWT token to Amazon API Gateway when calling protected APIs.
+ The backend verifies the token information to determine the user's identity and access permissions.

See the detailed deployment guide:

+ [5.3.2. Deploy Amazon Cognito Authentication](5.3.2-create-cognito/)

#### Deployment contents

In this chapter, you will complete the following tasks:

+ Deploy the Lambda Migration function inside the VPC.
+ Configure AWS Secrets Manager permissions for Lambda.
+ Verify the connection between Lambda and Amazon RDS PostgreSQL.
+ Create a Cognito User Pool and App Client.
+ Create the `Admin` and `Customer` user groups.
+ Test the user registration and login processes.