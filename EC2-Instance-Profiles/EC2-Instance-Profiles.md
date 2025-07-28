-  Use Amazon EC2 Instance Profiles for service access

To create an EC2 instance profile in AWS, you need to:

	1.	Create an IAM Role with a trust policy for EC2.
 
	2.	Attach IAM policies to the role (e.g., S3 access, CloudWatch logs, etc.).
 
	3.	Create the instance profile and associate the IAM role with it.
 
	4.	Attach the instance profile to an EC2 instance.

You can do this using the AWS CLI, Console, or Infrastructure as Code (e.g., Terraform or CloudFormation). Below is the AWS CLI method:

Step-by-Step via AWS CLI

1. Create the IAM Role

   aws iam create-role \
  --role-name EC2InstanceRole \
  --assume-role-policy-document '{
    "Version": "2012-10-17",
    "Statement": [{
      "Effect": "Allow",
      "Principal": {
        "Service": "ec2.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }]
  }'


2. Attach a Policy to the Role

Test case: Allow EC2 access to S3

aws iam attach-role-policy \
  --role-name EC2InstanceRole \
  --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

3. Create the Instance Profile

   aws iam create-instance-profile \
  --instance-profile-name EC2InstanceProfile

4. Add the Role to the Instance Profile

   aws iam add-role-to-instance-profile \
  --instance-profile-name EC2InstanceProfile \
  --role-name EC2InstanceRole

5. Attach Instance Profile to an EC2 Instance

 Use the EC2 Console or CLI when launching or modifying the instance:

 aws ec2 associate-iam-instance-profile \
  --instance-id i-0123456789abcdef0 \
  --iam-instance-profile Name=EC2InstanceProfile

 Terraform 

-This includes:

	•	IAM role (trusted by EC2)
 
	•	IAM policy attachment
 
	•	Instance profile
 
	•	EC2 instance using the profile
 

- In the repository create main.tf file 

 provider "aws" {
  region = "us-east-1"  # Change to your region
}

resource "aws_iam_role" "ec2_role" {
  name = "ec2-instance-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17",
    Statement = [{
      Action = "sts:AssumeRole",
      Effect = "Allow",
      Principal = {
        Service = "ec2.amazonaws.com"
      }
    }]
  })
}

resource "aws_iam_role_policy_attachment" "s3_readonly" {
  role       = aws_iam_role.ec2_role.name
  policy_arn = "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
}

resource "aws_iam_instance_profile" "ec2_instance_profile" {
  name = "ec2-instance-profile"
  role = aws_iam_role.ec2_role.name
}

resource "aws_instance" "example" {
  ami           = "ami-0c02fb55956c7d316"  # Replace with latest Amazon Linux 2 AMI for your region
  instance_type = "t2.micro"

  iam_instance_profile = aws_iam_instance_profile.ec2_instance_profile.name

  tags = {
    Name = "EC2-With-Instance-Profile"
  }
}


- Deploy

terraform init

terraform apply


  

   


