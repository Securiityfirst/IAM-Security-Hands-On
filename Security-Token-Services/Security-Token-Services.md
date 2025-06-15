- AWS STS capabilities (AssumeRole, GetSessionToken, Federated Access)

AWS STS (AWS Security Token Service) is a web service that enables you to request temporary, limited-privilege credentials for AWS Identity and Access Management (IAM) users or for users that you authenticate (federated users).

⸻

🔑 Core Use Cases
-	1.	Temporary access for IAM users or roles
→ Use AssumeRole to switch to a different role with specific permissions.
-	2.	Federated user access (SAML, web identity)
→ Use AssumeRoleWithSAML or AssumeRoleWithWebIdentity for temporary credentials.
-	3.	Programmatic access with temporary credentials
→ Use GetSessionToken to generate temporary credentials for MFA-enabled IAM users.
-	4.	Cross-account access
→ Use AssumeRole to allow secure access to resources in another AWS account.

![image](https://github.com/user-attachments/assets/5b75928c-28bd-451d-be35-909f70f10467)

🛡️ Security Considerations
	
 •	Temporary credentials are short-lived (15 mins to 12 hours).
	
 •	Use MFA for GetSessionToken to enforce strong authentication.
	
 •	Control access with trust policies on roles.
	
 •	Use condition keys like aws:RequestedRegion, aws:SourceIp, aws:MultiFactorAuthPresent.

📌 Example: AssumeRole CLI

aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/demo-role \
  --role-session-name demoSession

  aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/demo-role \
  --role-session-name demoSession


AssumeRole

The AWS Security Token Service (STS) AssumeRole API operation is a powerful mechanism that enables cross-account access, temporary access to resources, and privilege escalation (within scoped boundaries). Here’s a concise summary of its capabilities:

⸻

🔐 AssumeRole - Capabilities

![image](https://github.com/user-attachments/assets/2ac24f52-3570-431c-8c53-2cf0a00e3705)

🔄 Basic Workflow (How AssumeRole Works)
-	1.	Caller must have permission to call sts:AssumeRole on the target role.
-	2.	Role’s Trust Policy must allow the principal (caller) to assume the role.
-	3.	STS issues temporary credentials.
-	4.	These credentials can be used to access AWS resources as per the role’s permissions.

🧾 Example CLI Command

aws sts assume-role \
  --role-arn arn:aws:iam::123456789012:role/MyRole \
  --role-session-name my-session

 📦 Returned Credentials

{
  "Credentials": {
    "AccessKeyId": "ASIA....",
    "SecretAccessKey": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYzEXAMPLEKEY",
    "SessionToken": "AQoDYXdzEJr...<remainder of security token>",
    "Expiration": "2025-06-08T15:04:56Z"
  },
  "AssumedRoleUser": {
    "AssumedRoleId": "AROAIEXAMPLEID:my-session",
    "Arn": "arn:aws:sts::123456789012:assumed-role/MyRole/my-session"
  }
}

![image](https://github.com/user-attachments/assets/a0c43cb9-3180-426f-bab3-1f106ccf8879)


GetSessionToken

To retrieve a session token using aws sts get-session-token in a YAML-based automation (e.g., using a GitHub Actions workflow or AWS CodeBuild buildspec), you’ll need to integrate the AWS CLI into your YAML script.

Examples:

⸻

✅ Option 1: GitHub Actions YAML

This GitHub Actions workflow retrieves temporary credentials using aws sts get-session-token.

name: Get Session Token

on: [push]

jobs:
  retrieve-token:
    runs-on: ubuntu-latest

 steps:
    - name: Configure AWS Credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

 - name: Get Session Token
      run: |
        SESSION_OUTPUT=$(aws sts get-session-token --duration-seconds 3600)
        echo "Session Token Output:"
        echo "$SESSION_OUTPUT"

⚠️ Requires long-term AWS credentials to be stored in GitHub Secrets (AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY).

⸻

✅ Option 2: AWS CodeBuild YAML (buildspec.yml)

This is for CodeBuild, which uses a buildspec.yml file.

version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.11
  build:
    commands:
      - echo Getting temporary credentials using STS
      - TOKEN_JSON=$(aws sts get-session-token --duration-seconds 3600)
      - echo "Session Token:"
      - echo "$TOKEN_JSON"
      - export AWS_ACCESS_KEY_ID=$(echo $TOKEN_JSON | jq -r '.Credentials.AccessKeyId')
      - export AWS_SECRET_ACCESS_KEY=$(echo $TOKEN_JSON | jq -r '.Credentials.SecretAccessKey')
      - export AWS_SESSION_TOKEN=$(echo $TOKEN_JSON | jq -r '.Credentials.SessionToken')

✅ Install jq if not already available in the environment.

⸻

🔐 Why get-session-token?
	•	Used to retrieve temporary security credentials.
	•	Returns:
	•	AccessKeyId
	•	SecretAccessKey
	•	SessionToken
	•	Expiration

Use these for multi-factor-authenticated (MFA) sessions or short-term programmatic access.

How to use aws sts get-session-token with MFA in a YAML automation—specifically for two common setups:

⸻

✅ GitHub Actions: Get Session Token with MFA

📝 Prerequisites:
	•	Store these GitHub secrets:
	•	AWS_ACCESS_KEY_ID
	•	AWS_SECRET_ACCESS_KEY
	•	AWS_MFA_SERIAL (ARN of your MFA device)
	•	AWS_MFA_CODE (code from your virtual MFA device like Authy or Google Authenticator – or pass it via input/secure method)

⚠️ Real MFA codes expire quickly; use only for local testing or rotate dynamically.

name: Get Session Token with MFA

on: [workflow_dispatch]  # Trigger manually for MFA input

jobs:
  get-mfa-token:
    runs-on: ubuntu-latest
    steps:
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v4
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: us-east-1

- name: Get Session Token with MFA
      run: |
        echo "Requesting temporary session token with MFA"
        TOKEN_JSON=$(aws sts get-session-token \
          --serial-number ${{ secrets.AWS_MFA_SERIAL }} \
          --token-code ${{ secrets.AWS_MFA_CODE }} \
          --duration-seconds 3600)

   echo "$TOKEN_JSON"
        export AWS_ACCESS_KEY_ID=$(echo $TOKEN_JSON | jq -r '.Credentials.AccessKeyId')
        export AWS_SECRET_ACCESS_KEY=$(echo $TOKEN_JSON | jq -r '.Credentials.SecretAccessKey')
        export AWS_SESSION_TOKEN=$(echo $TOKEN_JSON | jq -r '.Credentials.SessionToken')

   echo "Temporary credentials ready to use."

✅ AWS CodeBuild: With MFA (Advanced)

MFA isn’t normally used in CodeBuild roles, but you can script this in a secure CI/CD if necessary using pre-built session tokens.

Simulate it locally in a buildspec.yml:

version: 0.2

phases:
  install:
    runtime-versions:
      python: 3.11
  build:
    commands:
      - echo "Get MFA session token"
      - export MFA_CODE=123456  
      
 # ⚠️ Replace with real-time input for actual use
- export MFA_ARN=arn:aws:iam::123456789012:mfa/your-user
      - TOKEN_JSON=$(aws sts get-session-token \
          --serial-number $MFA_ARN \
          --token-code $MFA_CODE \
          --duration-seconds 3600)
      - export AWS_ACCESS_KEY_ID=$(echo $TOKEN_JSON | jq -r '.Credentials.AccessKeyId')
      - export AWS_SECRET_ACCESS_KEY=$(echo $TOKEN_JSON | jq -r '.Credentials.SecretAccessKey')
      - export AWS_SESSION_TOKEN=$(echo $TOKEN_JSON | jq -r '.Credentials.SessionToken')
      - echo "Credentials set for temporary session"

🔐 Notes on MFA Automation
	•	MFA is designed to require human interaction.
	•	For automated pipelines, prefer IAM roles with assume-role and sts:assume-role instead of get-session-token with MFA.
	•	To automate get-session-token with MFA in a secure pipeline, use a script/tool that pulls the MFA code from a secure OTP source (e.g., AWS CLI + oathtool + encrypted secrets).

Reusable shell script that prompts for an MFA code and retrieves temporary session credentials using aws sts get-session-token. It’s great for local development, short-lived automation, or scripting into other tools.

⸻

✅ get_aws_session_token.sh

#!/bin/bash

# Prompt for MFA token
read -p "Enter your MFA code: " MFA_CODE

# Define your AWS user MFA ARN (change this to your actual ARN)
MFA_SERIAL="arn:aws:iam::123456789012:mfa/your-iam-username"

# Optional: specify profile, region, and duration
AWS_PROFILE="default"
AWS_REGION="us-east-1"
DURATION=3600

# Call STS to get session token
echo "Requesting session token from AWS STS..."
TOKEN_JSON=$(aws sts get-session-token \
  --serial-number $MFA_SERIAL \
  --token-code $MFA_CODE \
  --duration-seconds $DURATION \
  --profile $AWS_PROFILE \
  --region $AWS_REGION)

# Parse credentials
export AWS_ACCESS_KEY_ID=$(echo $TOKEN_JSON | jq -r '.Credentials.AccessKeyId')
export AWS_SECRET_ACCESS_KEY=$(echo $TOKEN_JSON | jq -r '.Credentials.SecretAccessKey')
export AWS_SESSION_TOKEN=$(echo $TOKEN_JSON | jq -r '.Credentials.SessionToken')

# Show masked output
echo "✅ Temporary credentials exported:"
echo "AWS_ACCESS_KEY_ID=${AWS_ACCESS_KEY_ID}"
echo "AWS_SECRET_ACCESS_KEY=****"
echo "AWS_SESSION_TOKEN=****"

# Optional: write to a temporary profile
aws configure set aws_access_key_id "$AWS_ACCESS_KEY_ID" --profile temp-session
aws configure set aws_secret_access_key "$AWS_SECRET_ACCESS_KEY" --profile temp-session
aws configure set aws_session_token "$AWS_SESSION_TOKEN" --profile temp-session
echo "✅ Credentials saved to AWS profile: temp-session"

# Usage example: aws s3 ls --profile temp-session

🧰 Prerequisites
	•	AWS CLI installed and configured (aws configure)
	•	jq installed (sudo apt install jq or brew install jq)
	•	Your IAM user has MFA enabled and permission to call sts:get-session-token

⸻

🚀 How to Use

  1. Save the script as get_aws_session_token.sh
  2. run it 

chmod +x get_aws_session_token.sh
./get_aws_session_token.sh

3.	Use the new credentials in your session or switch to the temp-session profile:

aws s3 ls --profile temp-session

✅ Option 1: Save AWS Session Credentials to .env File

Useful for local development or apps that load from .env (e.g., Flask, Node.js, Terraform via dotenv).

🔧 Modified Shell Script to Output .env

#!/bin/bash

read -p "Enter your MFA code: " MFA_CODE

MFA_SERIAL="arn:aws:iam::123456789012:mfa/your-iam-username"
AWS_PROFILE="default"
AWS_REGION="us-east-1"
DURATION=3600

echo "Getting session token..."

TOKEN_JSON=$(aws sts get-session-token \
  --serial-number "$MFA_SERIAL" \
  --token-code "$MFA_CODE" \
  --duration-seconds $DURATION \
  --profile "$AWS_PROFILE" \
  --region "$AWS_REGION")

export AWS_ACCESS_KEY_ID=$(echo $TOKEN_JSON | jq -r '.Credentials.AccessKeyId')
export AWS_SECRET_ACCESS_KEY=$(echo $TOKEN_JSON | jq -r '.Credentials.SecretAccessKey')
export AWS_SESSION_TOKEN=$(echo $TOKEN_JSON | jq -r '.Credentials.SessionToken')

# Save to .env

cat <<EOF > .env
AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID
AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY
AWS_SESSION_TOKEN=$AWS_SESSION_TOKEN
EOF

echo "✅ Temporary credentials saved to .env"

You can then load them in any script:

source .env

✅ Option 2: Use in Python with boto3

Use the credentials dynamically in your Python scripts:

import boto3
import os
import json
import subprocess

# Load token from .env or call STS directly
session = boto3.Session(
    aws_access_key_id=os.getenv("AWS_ACCESS_KEY_ID"),
    aws_secret_access_key=os.getenv("AWS_SECRET_ACCESS_KEY"),
    aws_session_token=os.getenv("AWS_SESSION_TOKEN"),
    region_name="us-east-1"
)

s3 = session.client('s3')
response = s3.list_buckets()
for bucket in response['Buckets']:
    print(f' {bucket["Name"]}')

Or dynamically from aws sts get-session-token output (e.g., from subprocess).

⸻

✅ Option 3: Use in Terraform via dotenv

Terraform will read .env if you load it before calling:

source .env
terraform plan

Or you can export directly from the script.

⸻

✅ Option 4: GitHub Actions with Temporary Token (via .env Upload)

You can modify the GitHub workflow to pull these credentials dynamically and load them:

  - name: Load credentials from .env
      run: |
        source .env
        echo "Running with temporary session credentials"
        aws s3 ls --region us-east-1

Python script version of aws-mfa-login that:
	•	Prompts for MFA code
	•	Calls aws sts get-session-token
	•	Saves the credentials to:
	•	Environment variables
	•	A .env file (for Flask, Terraform, Node.js, etc.)
	•	An optional temporary AWS CLI profile

✅ aws_mfa_login.py

#!/usr/bin/env python3
import os
import json
import subprocess

import boto3
from botocore.exceptions import ClientError

from dotenv import dotenv_values, set_key

def get_env_or_input(var_name, prompt=None):
    return os.getenv(var_name) or input(prompt or f"Enter {var_name}: ")

def get_session_token(mfa_serial, mfa_code, duration=3600, profile_name="default", region="us-east-1"):
    try:
        session = boto3.Session(profile_name=profile_name, region_name=region)
        sts_client = session.client('sts')

response = sts_client.get_session_token(
            DurationSeconds=duration,
            SerialNumber=mfa_serial,
            TokenCode=mfa_code
        )

 creds = response['Credentials']
        return {
            "AWS_ACCESS_KEY_ID": creds['AccessKeyId'],
            "AWS_SECRET_ACCESS_KEY": creds['SecretAccessKey'],
            "AWS_SESSION_TOKEN": creds['SessionToken']
        }

except ClientError as e:
        print(f"❌ Failed to get session token: {e}")
        exit(1)

def save_to_env_file(creds, env_path=".env"):
    print(f"📦 Writing credentials to {env_path}")
    for key, val in creds.items():
        set_key(env_path, key, val)

def save_to_temp_profile(creds, profile="temp-session"):
    print(f"🧾 Writing to AWS CLI profile: {profile}")
    subprocess.run(["aws", "configure", "set", "aws_access_key_id", creds["AWS_ACCESS_KEY_ID"], "--profile", profile])
    subprocess.run(["aws", "configure", "set", "aws_secret_access_key", creds["AWS_SECRET_ACCESS_KEY"], "--profile", profile])
    subprocess.run(["aws", "configure", "set", "aws_session_token", creds["AWS_SESSION_TOKEN"], "--profile", profile])

def main():
    print("🔐 AWS MFA Session Login")

mfa_serial = get_env_or_input("MFA_SERIAL", "Enter MFA ARN (e.g., arn:aws:iam::123456789012:mfa/your-user): ")
    mfa_code = get_env_or_input("MFA_CODE", "Enter MFA code: ")

 profile = os.getenv("AWS_PROFILE", "default")
    region = os.getenv("AWS_REGION", "us-east-1")

creds = get_session_token(mfa_serial, mfa_code, profile_name=profile, region=region)

 print("✅ Got temporary session credentials")

save_to_env_file(creds)
    save_to_temp_profile(creds)

 print("\n✅ All done!")
    print("👉 Use these in your terminal:`source .env`")
    print("👉 Or use AWS CLI with: `--profile temp-session`")

if __name__ == "__main__":
    main()

🛠️ Setup Instructions
	1.	Install dependencies:

pip install boto3 python-dotenv

2.	Save the script:
Save as aws_mfa_login.py and make it executable:

chmod +x aws_mfa_login.py

3. run it

./aws_mfa_login.py

🌍 Environment Variables You Can Set

Set these before running to avoid prompts:

export MFA_SERIAL=arn:aws:iam::123456789012:mfa/your-user
export AWS_PROFILE=your-iam-profile
export AWS_REGION=us-east-1

🚀 Use Cases
	•	Terraform: source .env && terraform plan
	•	Flask App: .env automatically loads credentials
	•	AWS CLI: aws s3 ls --profile temp-session

✅ Step-by-Step: Package aws-mfa-login as a CLI Tool

1. 📁 Project Structure

Create a new folder for your CLI tool:

mkdir aws_mfa_login
cd aws_mfa_login

structure it like this:

aws_mfa_login/

├── aws_mfa_login/

│   └── __main__.py         # Entry point

├── setup.py                # Packaging 

script

├── README.md               # Docs 

(optional)

├── requirements.txt        # Dependencies

2. 🧠 Your CLI Code (aws_mfa_login/__main__.py)

# aws_mfa_login/__main__.py

import os
import json
import subprocess
import boto3
from botocore.exceptions import ClientError
from dotenv import set_key


def get_env_or_input(var_name, prompt=None):
    return os.getenv(var_name) or input(prompt or f"Enter {var_name}: ")

def get_session_token(mfa_serial, mfa_code, duration=3600, profile_name="default", region="us-east-1"):
    try:
        session = boto3.Session(profile_name=profile_name, region_name=region)
        sts_client = session.client('sts')

response = sts_client.get_session_token(
            DurationSeconds=duration,
            SerialNumber=mfa_serial,
            TokenCode=mfa_code
        )

 creds = response['Credentials']
        return {
            "AWS_ACCESS_KEY_ID": creds['AccessKeyId'],
            "AWS_SECRET_ACCESS_KEY": creds['SecretAccessKey'],
            "AWS_SESSION_TOKEN": creds['SessionToken']
        }

except ClientError as e:
        print(f"❌ Failed to get session token: {e}")
        exit(1)

def save_to_env_file(creds, env_path=".env"):
    print(f"📦 Writing credentials to {env_path}")
    for key, val in creds.items():
        set_key(env_path, key, val)

def save_to_temp_profile(creds, profile="temp-session"):
    print(f"🧾 Writing to AWS CLI profile: {profile}")
    subprocess.run(["aws", "configure", "set", "aws_access_key_id", creds["AWS_ACCESS_KEY_ID"], "--profile", profile])
    subprocess.run(["aws", "configure", "set", "aws_secret_access_key", creds["AWS_SECRET_ACCESS_KEY"], "--profile", profile])
    subprocess.run(["aws", "configure", "set", "aws_session_token", creds["AWS_SESSION_TOKEN"], "--profile", profile])

def main():
    print("🔐 AWS MFA Session Login")

 mfa_serial = get_env_or_input("MFA_SERIAL", "Enter MFA ARN (e.g., arn:aws:iam::123456789012:mfa/your-user): ")
    mfa_code = get_env_or_input("MFA_CODE", "Enter MFA code: ")

 profile = os.getenv("AWS_PROFILE", "default")
    region = os.getenv("AWS_REGION", "us-east-1")

 creds = get_session_token(mfa_serial, mfa_code, profile_name=profile, region=region)

print("✅ Got temporary session credentials")

save_to_env_file(creds) save_to_temp_profile(creds)

print("\n✅ All done!")
print("👉 Use: `source .env` or `--profile temp-session`")

if __name__ == "__main__":
    main()


3. 📦 setup.py for Packaging.

from setuptools import setup, find_packages

setup(
    name='aws-mfa-login',
    version='0.1.0',
    description='AWS MFA session token retriever and environment loader',
    author='Your Name',
    packages=find_packages(),
    install_requires=[
        'boto3',
        'python-dotenv'
    ],
    entry_points={
        'console_scripts': [
            'aws-mfa-login=aws_mfa_login.__main__:main',
        ],
    },
)

4. 📄 requirements.txt

boto3
python-dotenv

5. 🚀 Install It Locally

In the root of the project:

pip install .

Now you can run:

aws-mfa-login

6. Optional: Create a Virtual Environment

python3 -m venv venv
source venv/bin/activate
pip install .

7. 🌍 Want to Publish to PyPI?
 Create an account on https://pypi.org

• Install the tools:

pip install twine build

• Build & upload:

 python -m build
twine upload dist/*

Federated Access

Federated Access with AWS STS, which is commonly used to grant temporary access to AWS for users from external identity providers (IdPs) such as Microsoft Entra ID (formerly Azure AD), Google, Facebook, or a custom SAML provider.

🌐 AWS STS Federated Access Overview

Federated access enables users outside of AWS to assume a role in your AWS account using one of the following:

✅ STS APIs for Federation:

![image](https://github.com/user-attachments/assets/c561c4ef-5593-4197-b138-f7534ce05863)

🔐 SAML Federation Example (Enterprise SSO with Azure AD)

🔁 Flow Summary:

User → Login to Azure AD → Gets SAML Assertion
 → AWS STS AssumeRoleWithSAML → Temporary AWS Credentials
 → Access AWS resources (S3, EC2, etc.)

✅ Steps to Implement:
-	1.	Create IAM Role for SAML
In AWS IAM:
	•	Trust policy must include SAML provider.
	•	Permissions policy defines allowed AWS actions.
	•	Example trust relationship:

 {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:saml-provider/AzureAD"
      },
      "Action": "sts:AssumeRoleWithSAML",
      "Condition": {
        "StringEquals": {
          "SAML:aud": "https://signin.aws.amazon.com/saml"
        }
      }
    }
  ]
}

-    2.	Configure IdP (e.g., Azure AD / Okta)
	•	Register AWS as an enterprise application.
	•	Upload AWS metadata to IdP and vice versa.
	•	Map IdP groups → AWS roles via SAML assertions.
	3.	User Logs into IdP → Gets SAML Assertion
	4.	Browser posts SAML to AWS SSO endpoint
AWS automatically calls AssumeRoleWithSAML and returns temporary credentials.

🌐 OIDC / Web Identity Federation Example (e.g., Google, Cognito)

🔁 Flow Summary:

User → Login with Google/Facebook → Gets OIDC Token
 → AWS STS AssumeRoleWithWebIdentity → Temporary AWS Credentials
 → Access AWS

 Example Boto3 Call:

import boto3

client = boto3.client('sts')

response = client.assume_role_with_web_identity(
    RoleArn='arn:aws:iam::123456789012:role/WebIdentityRole',
    RoleSessionName='web-identity-session',
    WebIdentityToken='ID_TOKEN_FROM_PROVIDER'
)

credentials = response['Credentials']


✅ Benefits of Federated Access

	
 •	🔐 No need to create IAM users
	
 •	⏳ Temporary credentials reduce risk
	
 •	🔄 SSO experience for enterprise users
	
 •	🔍 Can use attribute-based access control (ABAC)


 





 
