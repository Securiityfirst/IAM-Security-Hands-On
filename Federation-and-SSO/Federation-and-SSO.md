
# AWS Identity Federation (AIF)

AWS Identity Federation is a way to let users sign in to AWS without creating separate IAM users for them. Instead, AWS trusts an external identity provider (IdP)
such as your corporate directory, Google Workspace, Okta, Azure AD, or even Amazon Cognito — to authenticate users and then grant them temporary security credentials to access AWS resources.

It’s basically AWS saying:

“I’ll let your users in if you vouch for them, and I’ll give them temporary keys instead of permanent ones.”


How AWS Identity Federation Works

	1.	User attempts to access AWS
They start from your application, SSO portal, or a third-party IdP login screen.

	2.	Authentication at the IdP
 
The user signs in with existing credentials (e.g., Active Directory username/password, Google account, SAML-based login, etc.).

	3.	AWS trust relationship
The IdP sends an authentication response (like a SAML assertion or OIDC token) to AWS.

	4.	AWS STS issues temporary credentials
AWS Security Token Service (STS) exchanges the IdP token for short-lived AWS access keys.

	5.	User accesses AWS
 
The temporary credentials let them call AWS APIs, access the console, or interact with services — based on the IAM role policies assigned.

Common Identity Federation Methods in AWS

<img width="2778" height="760" alt="image" src="https://github.com/user-attachments/assets/6390a3f4-e2a7-4d59-b847-fc90db77e1fb" />

Key Benefits

	•	No need for IAM users — reduces account sprawl.
 
	•	Centralized identity management — rely on existing corporate or third-party IdP.
 
	•	Improved security — uses short-lived credentials instead of long-term keys.
 
	•	Easier onboarding/offboarding — when someone leaves your org, revoking access is done at the IdP.

Example Architecture

For a corporate SAML-based federation:

User → IdP Login (e.g., Okta) → SAML Assertion → AWS STS → Temporary IAM Role → Access AWS

For a mobile app with web identity federation:

User → Sign in with Google → OIDC Token → AWS STS → Temporary IAM Role → Access S3

Prepare Your Identity Provider (IdP)

	•	Example IdPs: Okta, Azure AD, ADFS, Ping Identity
 
	•	Configure an AWS application in your IdP.
 
	•	Collect:
 
	•	SAML metadata file or SAML endpoint URL
 
	•	IdP entity ID
 
	•	SAML attributes (e.g., Role, RoleSessionName)


2. Create the Identity Provider in AWS

	1.	Go to: AWS Console → IAM → Identity providers → Add provider.
 
	2.	Choose type: SAML.
 
	3.	Provider name: Choose a name (e.g., Okta-IdP).
 
	4.	Upload SAML metadata file or paste the URL from the IdP.
 
	5.	Save.


3. Create an IAM Role for Federated Access
   
	1.	Go to: AWS Console → IAM → Roles → Create role.
  2. Trusted entity type: SAML 2.0 federation.

	3.	Select the SAML provider you created.
 
	4.	Attribute-based role assumption: Choose Allow programmatic and AWS Management Console access.
 
	5.	Permissions policy: Attach what the federated user should have (e.g., AmazonS3ReadOnlyAccess).
 
	6.	Role name: Something meaningful like SAML-Federated-ReadOnly.
 
	7.	Trust policy will look like:

 {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<ACCOUNT_ID>:saml-provider/Okta-IdP"
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

4. Map IdP Users/Groups to AWS Roles
   
	•	In your IdP, assign AWS roles to users or groups.

	•	Pass them via the SAML assertion attributes

Attribute Name: https://aws.amazon.com/SAML/Attributes/Role
Attribute Value: arn:aws:iam::<ACCOUNT_ID>:role/SAML-Federated-ReadOnly,arn:aws:iam::<ACCOUNT_ID>:saml-provider/Okta-IdP

Test the Federation Login

	1.	Log into your IdP (Okta, Azure AD, etc.).
 
	2.	Click the AWS application tile.
 
	3.	You should be redirected to AWS without typing a separate AWS password.
 
	4.	Test permissions to ensure they match the IAM policy.


6. (Optional) Enable Multi-Account Access

	•	If you have multiple AWS accounts, repeat steps 2–4 in each account and assign roles accordingly.
 
	•	Users can choose accounts/roles during login.


# End Result

	•	Users authenticate via your IdP → get temporary AWS credentials via STS → access AWS with IAM role permissions.
 
	•	No IAM user passwords or long-term access keys needed.


# IAM Identity Federation (IAMIF)

IAM Identity Federation in AWS is a way to let users sign in to your AWS environment using credentials from an external identity provider (IdP) instead of creating and managing IAM users directly in AWS.

In other words:

Instead of giving everyone in your organization their own IAM username/password in AWS, you connect AWS to a trusted identity source (like your company’s Microsoft Entra ID / Azure AD, Okta, Google Workspace, or even another AWS account) and federate their identities.


Key Ideas

	•	Federation = Trust relationship between AWS and an external IdP.
 
	•	No AWS permanent credentials for the user — they get temporary credentials via AWS Security Token Service (STS).
 
	•	Users authenticate with the IdP, and AWS trusts that authentication.
 
	•	Good for SSO (Single Sign-On) and avoiding duplicated account management.


How It Works (High Level Flow)

	1.	User logs in to your external IdP (e.g., Okta, Entra ID) using their existing credentials.
 
	2.	The IdP authenticates the user and sends a security assertion (SAML or OIDC token) to AWS.
 
	3.	AWS STS exchanges that assertion for temporary credentials.
 
	4.	User uses these credentials to access AWS resources based on assigned IAM roles and policies.
 
<img width="2778" height="790" alt="image" src="https://github.com/user-attachments/assets/3c811b71-98fc-414d-ba06-9c18fd3f8e59" />


⸻

Why Use IAM Federation?

	•	Centralized user management in your existing IdP.
 
	•	Avoids AWS credential sprawl.
 
	•	Supports MFA from your IdP.
 
	•	Simplifies compliance by having one source of identity truth.
 

Common AWS Services Involved

	•	AWS IAM → Role creation, trust policies.
 
	•	AWS STS → Issues temporary credentials.
 
	•	AWS SSO (IAM Identity Center) → Easier setup for workforce federation.
 
	•	Amazon Cognito → Federation for applications you build.

practical example of AWS IAM Identity Federation using Microsoft Entra ID (Azure AD) as the Identity Provider via SAML 2.0.



Scenario

You have an organization using Microsoft 365 with Entra ID as the IdP, and you want employees to log in to the AWS Management Console without AWS-native IAM users — instead, they’ll use their Entra ID credentials.


Step-by-Step Setup

1. Plan Roles & Permissions

Decide what roles you’ll need in AWS:

	•	AWS-Admin → Full admin access.
 
	•	AWS-ReadOnly → Read-only access.


2. Create IAM Roles in AWS
   
	1.	Go to IAM → Roles → Create Role.
    
	2.	Select SAML 2.0 federation.
    
	3.	If no IdP exists yet, click Create New SAML Provider:
    
	•	Name: EntraID

	•	Upload the federation metadata XML file from Entra ID.

	4.	Create a role (e.g., AWS-Admin) with the required policy (e.g., AdministratorAccess).
    
	5.	In the Trust Policy, AWS will trust arn:aws:iam::<account_id>:saml-provider/EntraID.


3. Configure Microsoft Entra ID
   
	1.	Go to Azure Portal → Enterprise Applications → New Application.
 	
	2.	Add an AWS Single Sign-On app from the gallery.
    
	3.	Configure SAML:
 
	•	Identifier (Entity ID): urn:amazon:webservices

	•	Reply URL (ACS URL): https://signin.aws.amazon.com/saml

	4.	Upload AWS’s metadata XML (from the AWS SAML provider you created earlier).
    
	5.	In Attributes & Claims:
    
	•	Add https://aws.amazon.com/SAML/Attributes/Role → role_arn,principal_arn

Example:

arn:aws:iam::<account_id>:role/AWS-Admin,arn:aws:iam::<account_id>:saml-provider/EntraID

	6.	Assign users/groups to the AWS app in Entra ID.


4. Test the Federation
   
	1.	Sign in to your Microsoft Entra MyApps portal.
    
	2.	Click the AWS application.
    
	3.	Entra ID authenticates you and sends a SAML assertion to AWS.
    
	4.	AWS STS issues temporary credentials mapped to the IAM role you were assigned.
    
	5.	You land in the AWS Management Console — no AWS password required.

Flow Summary

User → Entra ID login → SAML Assertion → AWS STS → Temporary Credentials → AWS Console/CLI

Security Notes

	•	Always enable MFA on your IdP for extra protection.
 
	•	Use least privilege when assigning AWS roles.
 
	•	Monitor AWS CloudTrail for AssumeRoleWithSAML events.

<img width="1284" height="766" alt="image" src="https://github.com/user-attachments/assets/477c5a8c-7aa2-4362-8700-81868c0c1a50" />



# IAM Identity Center (IAM ID Center)

IAM Identity Center (formerly called AWS Single Sign-On) is AWS’s managed service for centralized access management across AWS accounts and supported cloud apps.

Instead of juggling separate IAM users and credentials in each AWS account, you can manage who has access to what in one place, and those people can sign in once to access everything they’re allowed to use.

NB:

To enable IAM Identity Center in an alternative Region, you must first delete the current IAM Identity Center configuration in the US East (Ohio) Region(default region)

Key Features

	1.	Centralized user management
 
	•	Integrates with external identity providers (Microsoft Entra ID/Azure AD, Okta, Google Workspace, etc.) or use its built-in identity store.
 
	•	Supports SCIM for automated user provisioning.
 
	2.	Single Sign-On (SSO)
 
	•	Users sign in once and access multiple AWS accounts, AWS applications, and SAML/OIDC-compatible business apps.
 
	3.	Granular permission assignments
 
	•	Uses Permission Sets (based on IAM roles and policies) to define what a user can do in each account.
 
	•	Assigns access to AWS accounts, specific services, or even single resources.
 
	4.	Integration with AWS Organizations
 
	•	Automatically aware of all accounts in your org.
 
	•	Lets you quickly assign roles across multiple accounts without creating duplicate IAM roles everywhere.
 
	5.	Auditing & security
 
	•	Central logging in AWS CloudTrail.
 
	•	MFA support for stronger authentication.


How It Works

	1.	Connect an Identity Source
 
	•	Use the built-in IAM Identity Center directory or connect to an external IdP.
 
	2.	Create Permission Sets
 
	•	Define access levels (Administrator, ReadOnly, custom).
 
	3.	Assign Users/Groups to Accounts
 
	•	Select AWS accounts (or OUs) and link them to permission sets.
 
	4.	User Sign-In
 
	•	Users go to the IAM Identity Center portal and see all their assigned AWS accounts and apps.

Benefits vs Traditional IAM Users

<img width="1284" height="822" alt="image" src="https://github.com/user-attachments/assets/8d2789e8-13ed-4302-833a-6daeed6745cf" />


1. Prerequisites

	•	You must be AWS Organizations management account admin (or have equivalent permissions).
 
	•	AWS Organizations already set up with all your AWS accounts linked.
 
	•	Decide whether you’ll use the built-in Identity Center directory or an external Identity Provider (IdP) such as Microsoft Entra ID, Okta, Google Workspace, etc.


2. Enable IAM Identity Center

	1.	Go to:
AWS Console → IAM Identity Center.

	2.	Click Enable.
 
	3.	Choose Region (select one — IAM Identity Center is a regional service, but works org-wide).
 
	4.	Click Enable IAM Identity Center.


3. Configure Your Identity Source

You have two main options:

	•	Option A: Built-in Directory
 
	•	Default choice.
 
	•	You manually create users and groups inside IAM Identity Center.
 
	•	Option B: External IdP
 
	•	Go to: Settings → Identity Source → Change.
 
	•	Choose External Identity Provider.
 
	•	Follow the provided metadata file/URL to configure your IdP (via SAML or SCIM).
 
	•	Enable automatic provisioning if your IdP supports SCIM.

4. Create Permission Sets

	1.	In IAM Identity Center, go to Permission Sets → Create permission set.
 
	2.	Choose a template (e.g., AdministratorAccess, ReadOnlyAccess) or Custom.
 
	3.	Set session duration (e.g., 1 hour).
 
	4.	(Optional) Attach customer-managed IAM policies for fine-grained access.


5. Assign Users/Groups to Accounts

	1.	Go to AWS Accounts in IAM Identity Center.
 
	2.	Select one or multiple accounts.
 
	3.	Click Assign users/groups.
 
	4.	Pick your users/groups and permission set.
 
	5.	Save.


6. Test the User Experience

	1.	Get the IAM Identity Center user portal URL:
 
Settings → User portal → Copy URL.

	2.	Send the link to your test user.
 
	3.	Log in as that user — you should see:
 
	•	All AWS accounts they have access to.
 
	•	Any SSO-enabled external apps you’ve assigned.


7. Enable MFA (Highly Recommended)

	1.	Go to Settings → Multi-Factor Authentication.
 
	2.	Require MFA for all sign-ins.
 
	3.	Choose TOTP (authenticator app) or security key.
    

8. Monitor & Audit
   
	•	Use AWS CloudTrail to log all sign-ins and assignments.

	•	Check IAM Identity Center → Activity reports for quick summaries.


  
  # Configure AWS SSO using AWS Managed AD


1. Prerequisites
   
	•	An AWS Managed Microsoft AD directory already deployed in your VPC (via AWS Directory Service).

	•	Network connectivity between AWS Managed AD and the IAM Identity Center service:

	•	Correct VPC, subnets, and security groups.

	•	Allow TCP 389 (LDAP), TCP 636 (LDAPS), TCP/UDP 88 (Kerberos), TCP/UDP 464 (Kerberos password change), and TCP/UDP 53 (DNS).

	•	An Active Directory user account with permissions to read users and groups in your directory (for binding IAM Identity Center).


3. Enable IAM Identity Center
   
	1.	Go to AWS Console → IAM Identity Center.
    
	2.	Click Enable (if not already enabled).
    
	3.	Select the Region where you want IAM Identity Center to run.
(Usually choose the same region as your AWS Managed AD.)


4. Change Identity Source to AWS Managed Microsoft AD
   
	1.	In IAM Identity Center, go to Settings → Identity Source.
    
	2.	Choose Change identity source.
    
	3.	Select AWS Managed Microsoft AD.
    
	4. Pick your directory from the dropdown.
    
	5.	Enter the AD connector service account credentials (bind account).
    
    6. Save changes.


4. Assign Users and Groups from AD
   
	1.	In IAM Identity Center → Users and Groups, you should now see AD-synced objects.
    
	2.	Assign groups or users to AWS accounts or IAM roles:
    
	•	AWS Accounts: Assign permissions via Permission Sets.

	•	Applications: Assign access to SAML-enabled apps.


5. Test Access
   
	1.	Go to your AWS access portal URL (found in IAM Identity Center settings).
    
	2.  Log in with an Active Directory username and password.
    
	3.	Verify that:
    
	•	The portal shows assigned AWS accounts and apps.

	•	MFA prompts appear if configured.


6. (Optional) Enable MFA for AD Users
   
	•	Go to IAM Identity Center → Settings → Multi-Factor Authentication.

	•	Enforce MFA for sign-in.

	•	This works in addition to AD authentication.


Common Pitfalls

	•	Network issues: IAM Identity Center needs to reach AWS Managed AD over the right ports — check your security groups and VPC peering.
 
	•	Time sync: AD and AWS must have synchronized clocks (NTP).
 
	•	Permissions: The AD bind account must have read permissions on all required OUs.

<img width="1284" height="773" alt="image" src="https://github.com/user-attachments/assets/f471dbbe-a7b3-4aa2-89b4-352a93344860" />



