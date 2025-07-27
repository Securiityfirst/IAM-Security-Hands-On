- 📄 IAM Role use cases
  
<img width="1284" height="922" alt="image" src="https://github.com/user-attachments/assets/423fbe33-7738-4548-bb12-81767971408f" />

<img width="1284" height="2236" alt="image" src="https://github.com/user-attachments/assets/4b32ad3d-725e-4d66-a030-93d414e8276f" />

<img width="1284" height="716" alt="image" src="https://github.com/user-attachments/assets/8857654e-a54c-4d92-a0cf-050e62d0e0bb" />

<img width="1284" height="1538" alt="image" src="https://github.com/user-attachments/assets/ff279335-2c67-41b5-b55b-a8ec519bcbee" />






  
- Cross-account access to S3 with aws:AssumeRole

  To set up cross-account access to an S3 bucket using AWS:AssumeRole, follow these steps. 

Prerequisites (2 Aws Accounts A&B)

	•	Account A (ID: 246368586877) owns the S3 bucket

	•	Account B (ID: 654327875444) needs to access the bucket using AssumeRole


✅ Step 1: Create a Role in Account A (S3 Owner)


This role will be assumed by Account B.


IAM Role Trust Policy (Account A)

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::222222222222:root"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

IAM Role Permissions (Account A)

Attach a policy to allow access to the S3 bucket.

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket-name",
        "arn:aws:s3:::my-bucket-name/*"
      ]
    }
  ]
}

✅ Step 2: Allow Role Access in the S3 Bucket Policy (Account A)

Update the S3 bucket policy to allow the role access:

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "CrossAccountAssumeRoleAccess",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:role/CrossAccountS3AccessRole"
      },
      "Action": [
        "s3:GetObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::my-bucket-name",
        "arn:aws:s3:::my-bucket-name/*"
      ]
    }
  ]
}

✅ Step 3: Assume the Role in Account B

Python (Boto3)

import boto3

# Step 1: Assume the role
sts_client = boto3.client('sts')

assumed_role = sts_client.assume_role(
    RoleArn="arn:aws:iam::111111111111:role/CrossAccountS3AccessRole",
    RoleSessionName="CrossAccountS3Session"
)

credentials = assumed_role['Credentials']

# Step 2: Use the temporary credentials to access S3

s3_client = boto3.client(
    's3',
    aws_access_key_id=credentials['AccessKeyId'],
    aws_secret_access_key=credentials['SecretAccessKey'],
    aws_session_token=credentials['SessionToken']
)

# Example: List objects

response = s3_client.list_objects_v2(Bucket='my-bucket-name')
for obj in response.get('Contents', []):
    print(obj['Key'])

    
