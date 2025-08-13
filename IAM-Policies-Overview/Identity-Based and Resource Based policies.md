1. Identity-Based Policies

What they are:

Policies attached to an IAM identity — a user, group, or role — to specify what actions they can perform on which AWS resources.

Where they’re stored:

Inside the IAM service.

Key features:

	•	Define permissions for the identity regardless of which resource they act on.
 
	•	Can be AWS-managed, customer-managed, or inline.
 
	•	Examples:
 
	•	Allow an IAM user to s3:GetObject from any bucket in the account.
 
	•	Allow an IAM role to start EC2 instances in a specific region.
 

Example JSON (identity-based):

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}

2. Resource-Based Policies

What they are:

Policies attached directly to an AWS resource, allowing other principals (identities in the same account or another account) to access that resource.

Where they’re stored:

With the resource itself — not in IAM.

Key features:

	•	The resource grants permissions to specific identities (AWS account, user, role).
 
	•	Common in cross-account access scenarios.
 
	•	Examples:
 
	•	S3 bucket policy allowing another AWS account to read objects.
 
	•	AWS KMS key policy allowing specific roles to use the key.
 
	•	Lambda function resource policy allowing another account to invoke it.

Example JSON (resource-based, S3 bucket policy):

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": { "AWS": "arn:aws:iam::123456789012:user/John" },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::my-bucket/*"
    }
  ]
}

<img width="2778" height="1284" alt="image" src="https://github.com/user-attachments/assets/e24dd720-6056-44d3-8a82-8ecddd4d6f7d" />

AWS evaluates access by checking:

	1.	Identity-based policy (does the caller have permission?)
 
	2.	Resource-based policy (does the resource allow it?)
 
	3.	Explicit denies always override allows.

