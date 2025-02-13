
# Basic Data Backup System on AWS
This repository contains an automated data backup solution using AWS services, including S3, Lambda, CloudWatch, and IAM. The system enables you to back up data from a source S3 bucket to a backup S3 bucket at scheduled intervals, ensuring your data is safely stored with minimal effort.

## Table of Contents
- [Introduction](#introduction)
- [AWS Services Used](#aws-services-used)
- [Setup Instructions](#setup-instructions)
  - [1. Create an S3 Bucket](#1-create-an-s3-bucket)
  - [2. Create an IAM Role for Lambda](#2-create-an-iam-role-for-lambda)
  - [3. Create a Lambda Function](#3-create-a-lambda-function)
  - [4. Set Up CloudWatch Events for Scheduling](#4-set-up-cloudwatch-events-for-scheduling)
  - [5. Test Your Backup System](#5-test-your-backup-system)
- [Monitoring](#monitoring)
- [License](#license)
## Introduction
This project provides a practical, serverless solution to back up data from an AWS S3 bucket to another S3 bucket using AWS Lambda. It uses CloudWatch Events to schedule the backups, so data is copied at predefined intervals. IAM roles are used to ensure that the Lambda function has appropriate permissions to access the S3 buckets.

## AWS Services Used
- **Amazon S3**: For storing data and backups.
- **AWS Lambda**: For automating the backup process.
- **Amazon CloudWatch**: For scheduling the Lambda function to run periodically.
- **AWS IAM**: For granting the Lambda function the necessary permissions.

- backup_lambda_function.py: The Lambda function code that copies data from one S3 bucket to another.



## Setup Instructions

Follow the steps below to set up the basic data backup system on AWS.

### 1. **Create an S3 Bucket**
First, create two S3 buckets: one for the source data and another for the backup.

- **Source Bucket**: Store the data you want to back up.
- **Backup Bucket**: Store the backup copies of your data.

### 2. **Create an IAM Role for Lambda**
Create an IAM role that grants your Lambda function the permissions needed to read from the source S3 bucket and write to the backup S3 bucket.

1. Navigate to **IAM** > **Roles** in the AWS Console.
2. Click **Create Role** and choose **Lambda** as the trusted entity.
3. Attach the **AmazonS3FullAccess** policy to the role.
4. Name the role `LambdaS3BackupRole` and create it.

### 3. **Create a Lambda Function**
Next, create a Lambda function that automates the backup process.

1. Navigate to **Lambda** in the AWS Console and click **Create Function**.
2. Choose **Author from scratch** and give the function a name (e.g., `BackupLambda`).
3. Choose the runtime (e.g., Python 3.9) and assign the `LambdaS3BackupRole` IAM role to the function.
4. Paste the Lambda function code (`backup_lambda_function.py`) into the function editor.
5. Save the function.
python
import boto3
import os
import datetime

def lambda_handler(event, context):
    s3 = boto3.client('s3')
    
    source_bucket = 'your-source-bucket-name'
    destination_bucket = 'your-backup-bucket-name'
    
    current_date = datetime.datetime.now().strftime('%Y-%m-%d_%H-%M-%S')
    
    objects = s3.list_objects_v2(Bucket=source_bucket)
    
    for obj in objects.get('Contents', []):
        copy_source = {'Bucket': source_bucket, 'Key': obj['Key']}
        destination_key = f'backup/{current_date}/{obj["Key"]}'
        
        s3.copy_object(CopySource=copy_source, Bucket=destination_bucket, Key=destination_key)
    
    return {
        'statusCode': 200,
        'body': f'Backup completed successfully to {destination_bucket}'
    }


### 4. **Set Up CloudWatch Events for Scheduling**
To automate the backups, schedule the Lambda function using CloudWatch Events.

1. Go to **CloudWatch** > **Rules** > **Create Rule**.
2. Choose **Schedule** as the event source.
3. Define the cron expression to schedule your backup (e.g., `cron(0 0 * * ? *)` for daily backups at midnight).
4. Set the target to the Lambda function you created (`BackupLambda`).
5. Create the rule.

### 5. **Test Your Backup System**
Test the Lambda function by manually invoking it from the Lambda console. Verify that the objects from the source bucket are copied to the backup bucket.
## Monitoring
You can monitor the Lambda function's performance and logs using **CloudWatch Logs**.
- After the Lambda function runs, check the logs in **CloudWatch** to verify that the backup process completed successfully.
- You can also create CloudWatch Alarms to notify you in case of failures.
---
## summary
This project automates data backup on AWS using S3, Lambda, CloudWatch, and IAM. It allows you to back up data from a source S3 bucket to a backup S3 bucket at scheduled intervals. The Lambda function is responsible for copying objects from the source bucket to the backup bucket. CloudWatch Events are used to trigger the Lambda function on a defined schedule (e.g., daily). IAM roles are set up to provide the necessary permissions for the Lambda function. This system ensures that your data is securely and automatically backed up without manual intervention.
