---
title: "Worklog Week 7"
date: 2026-06-07
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
---

### Week 7 Objectives:

- Explore how to apply IAM Roles and IAM Conditions to restrict access based on the principle of Least Privilege.
- Practice granting permissions to EC2 applications using IAM Roles instead of hardcoded Access Keys.
- Study database schema conversion and migration workflows using AWS SCT and AWS DMS.
- Discover Data Lake architecture on AWS and practice analyzing data with AWS Glue, Amazon Athena, and Amazon QuickSight.

### Tasks to Implement This Week:

| Day | Task | Start Date | Completion Date | Resource |
| --- | --- | --- | --- | --- |
| Mon | - Explore IAM Roles, IAM Conditions, and the principle of Least Privilege.<br>- Practice creating IAM Groups, IAM Users, IAM Roles, and configuring access conditions based on IP and time attributes. | 01/06/2026 | 01/06/2026 | https://000044.awsstudygroup.com/vi/ |
| Tue | - Practice granting permissions to an EC2 application using an IAM Role.<br>- Compare Access Key methods versus IAM Roles when an application interacts with AWS services. | 02/06/2026 | 02/06/2026 | https://000048.awsstudygroup.com/vi/ |
| Wed | - Learn about the AWS Schema Conversion Tool (AWS SCT).<br>- Practice converting database schemas and study the migration pipeline via AWS Database Migration Service (AWS DMS). | 03/06/2026 | 03/06/2026 | https://000043.awsstudygroup.com/vi/ |
| Thu | - Understand Data Lake architecture models on AWS.<br>- Practice data ingestion, building a Data Catalog with AWS Glue, and querying datasets using Amazon Athena. | 04/06/2026 | 04/06/2026 | https://000035.awsstudygroup.com/vi/ |
| Fri | - Practice data visualization using Amazon QuickSight.<br>- Finalize the data analytics workflow within the established Data Lake infrastructure. | 05/06/2026 | 05/06/2026 | https://000035.awsstudygroup.com/vi/ |

### Week 7 Achievements:

| Day | Task | Key Achievements | Image |
| --- | --- | --- | --- |
| Mon | IAM Roles & Conditions | Successfully practiced creating IAM Groups, IAM Users, and IAM Roles, configuring precise IAM Conditions to restrict access by IP address and specific time windows, enhancing overall security posture via Least Privilege. | |
| Tue | IAM Roles for EC2 Applications | Delegated secure authorization to an application running on an EC2 instance via an IAM Role rather than utilizing hardcoded Access Keys, securing cloud service interactions and eliminating credential exposure risks. | |
| Wed | AWS SCT & AWS DMS | Successfully granted system permissions to the **DMS_USER** account on an Oracle database source to support schema transformations and replication pipelines managed by AWS SCT and AWS DMS. Also restored an Amazon RDS database cluster instance from an active Snapshot and verified its operational health post-recovery. | ![Grant Permission](anh7.2.png)<br><br>![Restore RDS](anh7.1.png) |
| Thu | Data Lake on AWS | Researched the operational paradigms of Data Lakes on AWS, registered metadata within a Data Catalog using AWS Glue, and executed analytical queries against datasets using Amazon Athena. | |
| Fri | Amazon QuickSight | Connected target Data Lake tables into Amazon QuickSight to visualize insights, assembling interactive analytical charts and dashboards for business evaluation. | |