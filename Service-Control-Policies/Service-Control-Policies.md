<img width="1284" height="209" alt="image" src="https://github.com/user-attachments/assets/a5ab35bc-e8af-40b6-affb-db599938a4d7" />

<img width="1284" height="1427" alt="image" src="https://github.com/user-attachments/assets/c26ae834-0748-47a5-a68d-a6000c020b69" />

<img width="1284" height="2028" alt="image" src="https://github.com/user-attachments/assets/7eab25a6-6281-49e9-981e-5319d2632357" />

 
 ### Restrict EC2 Instance Types

  To restrict EC2 instance types using a Service Control Policy (SCP) in AWS Organizations, you can write a policy that denies the creation of EC2 instances unless the instance type is explicitly allowed.

Example: SCP to Allow Only Specific EC2 Instance Types

This SCP allows launching only t3.micro and t3.small instances. All others will be denied.

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyUnsupportedInstanceTypes",
      "Effect": "Deny",
      "Action": "ec2:RunInstances",
      "Resource": "*",
      "Condition": {
        "StringNotEqualsIfExists": {
          "ec2:InstanceType": [
            "t3.micro",
            "t3.small"
          ]
        }
      }
    }
  ]
}

Notes:

	•	This SCP must be attached to the organizational unit (OU) or account where you want the restriction to apply.
 
	•	It works only if the calling IAM principal is within an account that is governed by AWS Organizations.
 
	•	Ensure your IAM policies and SCPs are not contradicting each other. SCPs define maximum permissions.


### Restrict Based on Region or AMI

You can combine conditions, for example, to limit to certain regions:

"Condition": {
  "StringNotEqualsIfExists": {
    "ec2:InstanceType": ["t3.micro", "t3.small"]
  },
  "StringEquals": {
    "aws:RequestedRegion": "us-east-1"
  }
} 


### Prevent Deletion of All S3 Buckets

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyS3BucketDeletion",
      "Effect": "Deny",
      "Action": "s3:DeleteBucket",
      "Resource": "*"
    }
  ]
}

•	Prevents all principals in the affected accounts from deleting any S3 bucket, regardless of their IAM permissions.

	•	Does not prevent object deletion — only the bucket itself is protected.
 
	•	Applies across all regions unless scoped further with conditions.

 Allow deletion of only specific buckets (scoping):

 "Resource": [
  "arn:aws:s3:::bucket-to-allow-deletion"
]

Deny only in specific environments (e.g., prod OU):


Use AWS tag conditions or limit by account or OU placement.

### Combine with IAM policies:

Use IAM policies for fine-grained access control at the user/role level, while SCPs provide org-wide guardrails.
