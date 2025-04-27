# AWS Data Lakehouse Project - Spark and Human Balance

The third project in the "Data Engineering with AWS" course provided by Udemy.

The background of the task is STEDI is working on development of a hardware STEDI Step Trainer. The aim is to extract the data produced by the STEDI Step Trainer sensors and the mobile app, and curate them into a data lakehouse solution on AWS. Privacy is an essential consideration when it comes to customer data. Some of the customers have given permission for their data to be used for training the model purposes. Hence, the data used will be limited to the customers` data that gave their permission. This will be done by ensuring 


For these project, the following environments are required:
- Python and Spark
- AWS Glue
- AWS Athena
- AWS S3

As part of the project, creating Python scripts using AWS Glue and Glue Studio.Python editor can be used to locally run the code, but to test or run the Glue Jobs, they weill need to be submitted to an AWS Glue environment.

Project data:
The project consists of 3 datasets:
- *customer*
- *step_trainer*
- *accelerometer*

Using AWS Glue, AWS S3, Python, and Spark, create or generate Python scripts to build a lakehouse solution in AWS. The image below shows the requirements:
![image](https://github.com/user-attachments/assets/273f32f1-3a47-48ad-9056-3cfe63ad3719)

Relational diagram is given below:
![image](https://github.com/user-attachments/assets/32921352-9175-489d-957a-15563e1d9cfb)

Below are the starter lines to run in the AWS terminal, before starting creation of Glue jobs:

 - *aws s3 mb s3://kgolovko-lake-house*
 - *aws s3 ls s3://kgolovko-lake-house/*
 - *aws ec2 describe-vpcs*
 - *aws ec2 describe-vpcs --query "Vpcs[*].VpcId - to get VPC id if it is not shown in JSON
 - *aws ec2 describe-route-tables*
 - *aws ec2 create-vpc-endpoint --vpc-id vpc-09a86f342aad49043 --service-name com.amazonaws.us-east-1.s3 --route-table-ids rtb-03ce74bb95ad4ce13*

```bash
aws iam create-role --role-name my-glue-service-role --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "glue.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}'
```
```bash
aws iam put-role-policy --role-name my-glue-service-role --policy-name S3Access --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "ListObjectsInBucket",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::kgolovko-lake-house"
            ]
        },
        {
            "Sid": "AllObjectActions",
            "Effect": "Allow",
            "Action": "s3:*Object",
            "Resource": [
                "arn:aws:s3:::kgolovko-lake-house/*"
            ]
        }
    ]
}'
```
```bash
aws iam put-role-policy --role-name my-glue-service-role --policy-name GlueAccess --policy-document '{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "glue:*",
                "s3:GetBucketLocation",
                "s3:ListBucket",
                "s3:ListAllMyBuckets",
                "s3:GetBucketAcl",
                "ec2:DescribeVpcEndpoints",
                "ec2:DescribeRouteTables",
                "ec2:CreateNetworkInterface",
                "ec2:DeleteNetworkInterface",
                "ec2:DescribeNetworkInterfaces",
                "ec2:DescribeSecurityGroups",
                "ec2:DescribeSubnets",
                "ec2:DescribeVpcAttribute",
                "iam:ListRolePolicies",
                "iam:GetRole",
                "iam:GetRolePolicy",
                "cloudwatch:PutMetricData"
            ],
            "Resource": [
                "*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:CreateBucket",
                "s3:PutBucketPublicAccessBlock"
            ],
            "Resource": [
                "arn:aws:s3:::aws-glue-*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:DeleteObject"
            ],
            "Resource": [
                "arn:aws:s3:::aws-glue-*/*",
                "arn:aws:s3:::*/*aws-glue-*/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::crawler-public*",
                "arn:aws:s3:::aws-glue-*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:PutLogEvents",
                "logs:AssociateKmsKey"
            ],
            "Resource": [
                "arn:aws:logs:*:*:/aws-glue/*"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "ec2:CreateTags",
                "ec2:DeleteTags"
            ],
            "Condition": {
                "ForAllValues:StringEquals": {
                    "aws:TagKeys": [
                        "aws-glue-service-resource"
                    ]
                }
            },
            "Resource": [
                "arn:aws:ec2:*:*:network-interface/*",
                "arn:aws:ec2:*:*:security-group/*",
                "arn:aws:ec2:*:*:instance/*"
            ]
        }
    ]
}'
```

- *git clone https://github.com/udacity/nd027-Data-Engineering-Data-Lakes-AWS-Exercises.git*
- *cd nd027-Data-Engineering-Data-Lakes-AWS-Exercises/starter/project/starter/customer/landing/*
- *aws s3 cp ./customer-1691348231425.json s3://kgolovko-lake-house/customer_landing/*
- *aws s3 cp ./ s3://kgolovko-lake-house/accelerometer_landing/ --recursive*
