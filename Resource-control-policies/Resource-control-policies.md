
<img width="1284" height="1702" alt="image" src="https://github.com/user-attachments/assets/ba7130ef-6854-4265-8cc8-b2d374031365" />

<img width="1284" height="2045" alt="image" src="https://github.com/user-attachments/assets/a02f07d3-58bd-4947-bb76-9c14d1117e2d" />


A Resource Control Policy (RCP) in AWS Organizations is used to manage access to specific AWS resources in trusted access scenarios. It’s mostly applied in Service-Linked Role (SLR) or delegated administrator use cases, often with AWS services like AWS Resource Access Manager (RAM) or Control Tower.


# Example Use Case: AWS RAM with Resource Control Policy

If you want to restrict which resources can be shared with other accounts using AWS RAM, you would create an RCP that looks like this:

# Sample Resource Control Policy

 {
  "Effect": "Allow",
  "Action": "*",
  "Resource": [
    "arn:aws:ec2:us-east-1:111122223333:subnet/subnet-abc12345",
    "arn:aws:ec2:us-east-1:111122223333:vpc/vpc-98765432"
  ]
}

This allows only the specified subnet and VPC to be shared across accounts using RAM.

# How to Apply an RCP

	1.	Enable Trusted Access for a service like AWS RAM:

aws organizations enable-aws-service-access --service-principal ram.amazonaws.com

2.	Attach a Resource Policy:
Use the AWS CLI or API to attach the policy to your organization or OU:

aws ram put-resource-policy \
  --policy "<YOUR_JSON_POLICY>" \
  --resource-arn arn:aws:ram::org-id:resource-share/share-id

  Note:

  RCPs typically require IAM administrative privileges and only work for specific services that support resource-level sharing in AWS Organizations.

  

 


