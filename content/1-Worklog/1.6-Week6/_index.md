---
title: "Worklog Week 6"
date: 2026-05-31
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
---

### Week 6 Objectives:

- Explore resource management using AWS Tags and Resource Groups.
- Practice controlling EC2 access permissions via IAM Policies combined with Resource Tags.
- Understand the mechanics of IAM Permission Boundaries to restrict the maximum permissions of users.
- Practice encrypting data on Amazon S3 using AWS KMS, logging activities with CloudTrail, and testing data sharing via Presigned URLs.

### Tasks to Implement This Week:

| Day | Task | Start Date | Completion Date | Resource |
| --- | --- | --- | --- | --- |
| Mon | - Overview of AWS Tags and Resource Groups.<br>- Practice assigning tags to resources and creating Resource Groups for collective management. | 25/05/2026 | 25/05/2026 | https://000027.awsstudygroup.com/vi/ |
| Tue | - Study EC2 permission management using IAM combined with Resource Tags.<br>- Practice creating IAM Policies, IAM Roles, and verifying access permissions based on tags. | 26/05/2026 | 26/05/2026 | https://000028.awsstudygroup.com/vi/ |
| Wed | - Learn about IAM Permission Boundaries.<br>- Practice creating a Permission Boundary and testing the maximum permission limits of an IAM User. | 27/05/2026 | 27/05/2026 | https://000030.awsstudygroup.com/vi/ |
| Thu | - Understand AWS KMS and data encryption on Amazon S3.<br>- Create a KMS Key and configure an S3 bucket to utilize the encryption key. | 28/05/2026 | 28/05/2026 | https://000033.awsstudygroup.com/vi/ |
| Fri | - Practice using CloudTrail to log activities and generating Presigned URLs to share files stored on Amazon S3.<br>- Verify the accessibility of encrypted objects. | 29/05/2026 | 29/05/2026 | https://000033.awsstudygroup.com/vi/ |
| Sat | - Clean up resources after completing the labs.<br>- Terminate EC2 instances and review created services to prevent extra charges on AWS. | 30/05/2026 | 30/05/2026 | AWS Study Group |

### Week 6 Achievements:

| Day | Task | Key Achievements | Image |
| --- | --- | --- | --- |
| Mon | Managing Resources with Tags and Resource Groups | Grasped how to utilize tags to categorize AWS resources and created Resource Groups to manage multiple resources collectively, supporting efficient tracking and administration. | |
| Tue | Access Control using Resource Tags | Successfully created IAM Policies and IAM Roles using Resource Tags to control EC2 access permissions following the principle of Least Privilege. | |
| Wed | IAM Permission Boundary | Understood the mechanics of Permission Boundaries and practiced restricting the maximum permissions of an IAM User to prevent unauthorized privilege escalation. | |
| Thu | AWS KMS and Data Encryption | Created a customer-managed KMS Key, configured an Amazon S3 bucket to use the encryption key, and explored how CloudTrail logs activities related to the encrypted data. | |
| Fri | Sharing Data via Presigned URL | Successfully generated a Presigned URL to share a file stored on Amazon S3 while ensuring the data remains protected by AWS KMS with a time-limited access window. | ![Presigned URL](anh6.2.png) |
| Sat | Resource Cleanup | Successfully terminated the EC2 instance and deleted associated resources utilized during the practical exercises to prevent unnecessary billing on the AWS account. | ![Delete EC2](anh6.1.png) |