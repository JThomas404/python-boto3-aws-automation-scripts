# Automating EC2 Snapshots with AWS Lambda and EventBridge

This project demonstrates the use of AWS Lambda, Boto3, and EventBridge to automate the creation of EC2 volume snapshots. The project applies infrastructure automation principles, security best practices, and event-driven design using native AWS services. It serves as a practical example of integrating Python automation with AWS serverless architecture.

---

## Project Overview

### Objective

Create a fully automated backup solution by configuring a Lambda function that creates a snapshot of an EC2 instance's attached volume once every 24 hours.

### Motivation

Routine backups are essential for disaster recovery and business continuity. Automating this process eliminates manual intervention, reduces human error, and ensures consistent backups.

---

## Initial Setup and Resources

### AWS Services Used:

- **EC2**: A running instance named `boto3-ec2-instance-01` with an EBS volume.
- **Lambda**: A function named `LambdaEC2Snapshot` written in Python using the Boto3 SDK.
- **IAM**: A custom role associated with the Lambda function for permissions.
- **EventBridge**: Configured to trigger the Lambda function every 24 hours.
- **CloudWatch**: Used to verify and debug Lambda execution.

### Local Development Environment:

- Python virtual environment with `boto3` installed.
- Code downloaded from AWS Lambda console into VS Code for editing and testing.

---

## Detailed Workflow

1. **EC2 Instance Setup**

   - A volume ID was identified from the running EC2 instance `boto3-ec2-instance-01`.
   - This volume ID was hardcoded into the Lambda function for snapshot operations.

2. **Lambda Function Implementation**

   The following Python script was implemented:

   ```python
   import json
   import boto3
   import logging
   from datetime import datetime

   logger = logging.getLogger()
   logger.setLevel(logging.INFO)

   def lambda_handler(event, context):
       ec2 = boto3.client('ec2')
       current_date = datetime.now().strftime("%Y-%m-%d")

       try:
           response = ec2.create_snapshot(
               VolumeId='vol-0c7a2ee934c239a59',
               Description='My EC2 Snapshot',
               TagSpecifications=[
                   {
                       'ResourceType': 'snapshot',
                       'Tags': [
                           {
                               'Key': 'Name',
                               'Value': f"My EC2 snapshot {current_date}"
                           }
                       ]
                   }
               ]
           )
           logger.info(f"Successfully created snapshot: {json.dumps(response, default=str)}")
       except Exception as e:
           logger.error(f"Error creating snapshot: {str(e)}")
   ```

   **Script Breakdown:**

   - `boto3.client('ec2')` instantiates the EC2 client for AWS API interaction.
   - `create_snapshot()` is used to back up the volume.
   - Tags help identify the snapshot by date.
   - Structured logging ensures observability via CloudWatch.

3. **EventBridge Configuration**

   - Accessed EventBridge from the AWS Console.
   - Created a schedule rule with the following parameters:

     - Frequency: Every 24 hours
     - Time Zone: Configured appropriately
     - Target: `LambdaEC2Snapshot`

4. **Initial Execution & Error Encounter**

   - The function failed with the following CloudWatch error:

   ```
   [ERROR] Error creating snapshot: An error occurred (UnauthorizedOperation) when calling the CreateSnapshot operation:
   You are not authorized to perform this operation. User: arn:aws:sts::533267010082:assumed-role/LambdaEC2Snapshot-role-hoaaikpg/LambdaEC2Snapshot
   is not authorized to perform: ec2:CreateSnapshot on resource: arn:aws:ec2:us-east-1::snapshot/* because no identity-based policy allows the ec2:CreateSnapshot action.
   ```

   - The issue was traced to missing IAM permissions.

5. **IAM Policy Update**

   - The execution role attached to Lambda was updated with the following least-privilege inline policy:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Action": [
           "ec2:CreateSnapshot",
           "ec2:CreateTags",
           "ec2:DescribeVolumes"
         ],
         "Resource": "*"
       }
     ]
   }
   ```

6. **Final Execution**

   - After updating permissions, the Lambda function executed successfully.
   - Snapshots were created and tagged with the current date.
   - Verified in the EC2 console that the snapshot existed with appropriate metadata.

---

## Folder Structure

```
python-boto3-aws-automation-scripts/
├── LambdaEC2Snapshot/
│   └── lambda_function.py
├── README.md
└── venv/
```

---

## Key Takeaways

- Demonstrates hands-on experience with AWS Lambda, IAM, EC2, and EventBridge.
- Shows the ability to troubleshoot AWS permissions using CloudWatch.
- Follows best practices by implementing the principle of least privilege.
- Highlights experience with Boto3 scripting and Python automation.
- Provides a working example of serverless architecture in action.

---
