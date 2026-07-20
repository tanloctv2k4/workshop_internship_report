---
title: "Worklog Week 3"
date: 2026-05-10
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
---

### Week 3 Objectives:

- Explore connectivity solutions and network infrastructure management on AWS.
- Practice deploying Hybrid DNS using Amazon Route 53 Resolver.
- Practice verifying network connectivity using Reachability Analyzer.
- Learn about AWS Backup and perform data backup and restoration tasks.
- Deploy AWS Storage Gateway to connect storage environments between On-premises and Amazon S3.

### Tasks to Implement This Week:

| Day | Task | Start Date | Completion Date | Resource |
| --- | --- | --- | --- | --- |
| Mon | - Explore Hybrid DNS architecture and Amazon Route 53 Resolver.<br>- Practice deploying infrastructure templates using AWS CloudFormation. | 04/05/2026 | 04/05/2026 | https://000010.awsstudygroup.com/vi/ |
| Tue | - Learn about AWS Reachability Analyzer.<br>- Practice path analysis and verify network connectivity between resources inside the VPC. | 05/05/2026 | 05/05/2026 | https://000057.awsstudygroup.com/vi/ |
| Wed | - Explore AWS Backup functionalities.<br>- Create a Backup Plan and successfully restore an EC2 instance from a designated Recovery Point. | 06/05/2026 | 06/05/2026 | https://000013.awsstudygroup.com/vi/ |
| Thu | - Learn about AWS Storage Gateway models.<br>- Deploy a File Storage Gateway and configure an SMB File Share connected to Amazon S3. | 07/05/2026 | 07/05/2026 | https://000014.awsstudygroup.com/vi/ |
| Fri | - Practice mounting and connecting the On-premises machine to the SMB File Share.<br>- Test data read, write, and synchronization capabilities to Amazon S3. | 08/05/2026 | 08/05/2026 | https://000014.awsstudygroup.com/vi/ |
| Sat | - Inspect the operational health and status of the Storage Gateway.<br>- Verify objects stored in Amazon S3 and clean up provisioned lab resources to prevent extra costs. | 09/05/2026 | 09/05/2026 | https://000014.awsstudygroup.com/vi/ |

### Week 3 Achievements:

| Day | Task | Key Achievements | Image |
| --- | --- | --- | --- |
| Mon | Deploying Hybrid DNS with Amazon Route 53 | Successfully deployed a Hybrid DNS architecture model using AWS CloudFormation. All Nested Stacks reached the **CREATE_COMPLETE** state, and generated output parameters like RDPURL and Remote Desktop Gateway Security Groups were successfully captured. | ![Hybrid DNS](anh3.1.png) |
| Tue | Verifying Connectivity with Reachability Analyzer | Conducted network path analysis between targeted VPC resources using AWS Reachability Analyzer. The tool evaluated the path as **Reachable** and status as **Succeeded**, confirming the network configuration operates accurately. | ![Reachability Analyzer](anh3.2.png) |
| Wed | EC2 Backup and Restoration using AWS Backup | Created and successfully executed an EC2 instance restoration workflow from an active Recovery Point. The system finalized the restore job with a **Completed** status, rebuilding an 8 GB capacity EC2 instance. | ![AWS Backup](anh3.3.png) |
| Thu | Deploying AWS Storage Gateway | Successfully initiated an SMB File Share backed by AWS Storage Gateway and mapped it to an Amazon S3 bucket for operational data storage. | ![Storage Gateway](anh3.4.png) |
| Fri | Connecting File Shares | Successfully mounted the SMB File Share to the local On-premises environment, executed real-time file read and write operations, and confirmed that data synchronized seamlessly to Amazon S3. | |
| Sat | Verifying and Resource Cleanup | Monitored the operational status of the Storage Gateway, audited the uploaded data files inside the target Amazon S3 bucket, and terminated all deployed resources to prevent unexpected billing. | |