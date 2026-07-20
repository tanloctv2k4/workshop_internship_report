---
title: "Worklog Week 8"
date: 2026-06-14
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
---

### Week 8 Objectives:
- Explore and migrate the monolithic application architecture to a serverless/serviceful microservices model.
- Deploy an identity management and access control system (AAA) using Amazon Cognito.
- Configure and work with two popular cloud database systems: Amazon RDS (Relational) and Amazon DynamoDB (NoSQL).
- Build, package, and automatically distribute data processing functions (Python/Node.js) via AWS SAM CLI and API Gateway.
- Operate Static Website Hosting (SPA) on S3 and establish a real-time performance monitoring and tracing framework using AWS X-Ray.

---

### Tasks to Implement This Week (In System Logical Order):

| Day | Task | Start Date | Completion Date | Resource |
| --- | --- | --- | --- | --- |
| Mon | AAA Infrastructure & NoSQL Database<br>- Create an Amazon Cognito User Pool (cognito-fcj-book-shop) and Identity Pool to set up the sign-up/sign-in system.<br>- Create the NoSQL database table TravelBuddyTripSectors on Amazon DynamoDB to support high I/O response rates. | 08/06/2026 | 08/06/2026 | https://000081.awsstudygroup.com/<br>https://000055.awsstudygroup.com/ |
| Tue | Backend Development & Serverless API<br>- Write Lambda functions to handle authentication logic (Register, Confirm_user, Login) using Python.<br>- Configure routing, set up CORS mechanisms, and automatically deploy serverless resources using the AWS SAM CLI tool. | 09/06/2026 | 09/06/2026 | https://000081.awsstudygroup.com/ |
| Wed | Frontend SPA Deployment<br>- Configure and build the Single Page Application (SPA) source code for the TravelBuddy project.<br>- Deploy the static files to Amazon S3 as Static Website Hosting.<br>- Integrate the Bearer Token (JWT from Cognito) into the Client to secure all backend endpoints routed through API Gateway. | 10/06/2026 | 10/06/2026 | https://000055.awsstudygroup.com/ |
| Thu | Relational Database Infrastructure (RDS)<br>- Study an overview of the Amazon RDS (MySQL) managed relational database service.<br>- Set up a DB Subnet group spanning multiple Availability Zones (AZ) to ensure high availability and disaster recovery.<br>- Create and configure a dedicated Security Group for the DB Instance to enforce strict network access control. | 11/06/2026 | 11/06/2026 | https://000005.awsstudygroup.com/ |
| Fri | Full-Stack App Operations & RDS Connection<br>- SSH into the EC2 instance using MobaXterm, and install Node.js alongside the MySQL Client environment.<br>- Clone the management application source code, and configure the .env environment variables to connect directly to the RDS Endpoint.<br>- Run the SQL script to define the user table structure, populate sample records, and fire up the Web Server on port 5000. | 12/06/2026 | 12/06/2026 | https://000005.awsstudygroup.com/ |
| Sat | RDS Backup & Recovery Testing<br>- Configure automated scheduling and define a specific Maintenance Window for the RDS cluster.<br>- Practice creating a manual DB Snapshot to securely capture the running database state.<br>- Execute a complete Restore Snapshot pipeline to spin up a separate, independent DB Instance and verify data integrity. | 13/06/2026 | 13/06/2026 | https://000005.awsstudygroup.com/ |
| Sun | System Performance Tracing with AWS X-Ray<br>- Enable Active Tracing within the AWS Lambda Console for the deployed microservices.<br>- Leverage the CloudWatch Service Map to visually track the end-to-end flow of distributed requests.<br>- Analyze detailed Trace graphs to identify performance bottlenecks (such as pinpointing a DynamoDB table Scan taking 1.67s out of a total 5.98s request cycle). | 14/06/2026 | 14/06/2026 | https://000055.awsstudygroup.com/ |

---

### Week 8 Achievements:

| Day | Task | Key Achievements |
| --- | --- | --- |
| Mon | Configuring Cognito & DynamoDB | Successfully established a highly secure, centralized email-based authentication gateway and initiated the distributed NoSQL data storage engine. |
| Tue | Provisioning Serverless APIs via SAM | Successfully packaged and updated the deployment stages for the API Gateway and Lambda integration. All active stacks successfully achieved the **CREATE_COMPLETE** state. |
| Wed | Deploying Static Website SPAs | The frontend application runs reliably via the S3 website endpoint, processing the new user sign-up pipeline smoothly and enforcing token-based request authorization rules. |
| Thu | Configuring Secure RDS Networking | Successfully isolated the database cluster inside a private VPC subnet framework, preventing direct public internet access exposures. |
| Fri | Synchronizing Full-Stack Apps & RDS | Populated 18 structured user profiles into the relational database cluster. The production Node.js engine running on port 5000 accurately retrieves and renders the live rows onto the web interface. |
| Sat | Executing Disaster Recovery (DR) Operations | Created a stable manual DB Snapshot on S3 storage. Completed the structural database restoration workflow, bringing the backup instance online in an **Available** operational status. |
| Sun | Performance Tuning with AWS X-Ray | Generated a live, end-to-end Service Map graph linking all cloud microservice layers. Effectively evaluated the distinct impacts of Cold Starts and table Scans to plan optimized global index strategies. |