# IAM-Security-Hands-On

# AWS IAM Security Hands-On Practice

Welcome to the **AWS IAM Security Hands-On Practice** lab repository. This training is designed for individuals seeking practical, scenario-based learning around AWS Identity and Access Management (IAM), with a strong focus on real-world security best practices.

# Prerequisites

- AWS Free Tier Account
- Basic knowledge of IAM and AWS Console
- AWS CLI installed and configured (optional but recommended)
  
# Training Topics Covered  

# AWS Account Setup(AAS)

-  Create a new AWS Account

https://signin.aws.amazon.com/signup?request_type=register

-  Save and delete root token securely(Make sure all security recommendations are followed)

<img width="1284" height="1038" alt="image" src="https://github.com/user-attachments/assets/9c2a5db3-0bb0-4fb2-b326-0ba46c806330" />

# Cost and Billing(C&B)

-  Setup and display billing alarms(AWS Cost Anomaly Detection)

Configure AWS Cost Anomaly Detection for your account. Cost Anomaly Detection is a cost management service that helps you reduce cost surprises and enhance control without slowing innovation. It uses advanced machine learning models to detect and alert on anomalous spend patterns across your deployed AWS services. Monitors and sends daily email alerts about your Aws resources.

<img width="1284" height="2382" alt="image" src="https://github.com/user-attachments/assets/27c732e2-4f38-4903-b16d-caa0fcb1a667" />


# AWS Authentication Methods(AAM)
  
![image](https://github.com/user-attachments/assets/d37e7db4-a2c1-4f37-81b6-e59d74087e61)

![image](https://github.com/user-attachments/assets/07a53bd9-2d1b-4deb-9d1f-233a7171661c)

# IAM Core Components(IAMCC)
-  Create IAM Users
-  Create and attach IAM Policies
-  Create IAM Roles
-  Create IAM User Groups

# IAM Permissions Management(IAMPM)
-  Grant custom policies
-  Grant IAM roles
-  Grant AWS Managed Policies (AMP)

# Security Token Services(STS)
-  AWS STS capabilities (AssumeRole, GetSessionToken, Federated Access)

# IAM Policies Overview(IAMPO)
-  Inline and AWS managed policies
-  custom managed policies
-  Identity-Based vs Resource-Based Policies (IBP vs RBP)
-  Access Control: Role-Based Access Control (RBAC) and Attribute-Based Access Control (ABAC)
-  Apply Permission Boundaries (PB)
-  IAM Policy Evaluation Logic (IPE)
-  IAM Policy Structure (IPS)

# Policy Tools(PT)
-  AWS Policy Generator (APG)
-  IAM Policy Simulator (IPSI)


# AWS Organizations & SCP(AO&SCP)

# Organizations & Management
-  Create AWS Organization
-  Create Organizational Units (OU)

# Service Control Policies (SCP)
-  Restrict EC2 Instance Types
-  Prevent S3 Bucket Deletion

# Resource Control Policies

- Restrict Identity based access on Aws resources (delegated administrator)

# IAM Advanced Use Cases(IAMUC)

# IAM Roles: Use Cases
-  IAM Role use cases
-  Cross-account access to S3 with `aws:AssumeRole`

# EC2 Instance Profiles(EIP)
-  Use Amazon EC2 Instance Profiles for service access


# AWS Directory and Identity Services (AD&IS)

# Directory Services
-  AWS Managed Microsoft Active Directory

# Federation & SSO
-  AWS Identity Federation (AIF)
-  IAM Identity Federation (IAMIF)
-  IAM Identity Center (IAM ID Center)
-  Configure AWS SSO using AWS Managed AD


# Diagrams & Visuals

- IAM Architecture Diagrams
- Identity Federation Flow Diagrams
- Access Control Models (RBAC, ABAC)
- Permission Boundaries & Policy Evaluation Flows


# Screenshots & Examples

- IAM Role Creation & Assignment
- MFA Configuration
- STS in Action (Temporary Credentials)
- Policy Simulation Results
- SCP Enforcement Examples


# Disclaimer

This repository is for educational purposes only. Please ensure you follow your organization's cloud usage policies and avoid deploying sensitive workloads during practice.


# Contributing

Feel free to contribute to this repository by:
- Adding real-world use cases
- Improving diagrams and visuals
- Sharing best practices or security tips


# Contact

If you encounter issues or want to collaborate, feel free to open an issue or reach out via GitHub Discussions.

---

Happy learning! 🚀
