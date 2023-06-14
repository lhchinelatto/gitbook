---
description: >-
  https://learn.microsoft.com/en-us/training/paths/az-104-manage-identities-governance/
---

# Manage identities and governance in Azure

## Configure Azure Active Directory <a href="#configure-azure-active-directory" id="configure-azure-active-directory"></a>

Azure Active Directory is a multi-tenant cloud-based directory and identity management service.

Can be used to authenticate services like Microsoft 365, Azure apps and external applications.

AAD Features:

* Single sign-on (SSO) - Use same credentials for different apps (internal and third-party)
* Supports multiple devices.
* Security - MFA, access policies and group-based access management, security reports, notifications, remediation, and risk-based policies
* Self-service support (password reset for example)

AAD Components:

* Identity - An object that can be authenticated (user or application)
* Account - An identity that has data associated with it
* Azure AD Account - An identity created using Azure AD or another MS cloud service
* Azure Tenant (directory) - Single dedicated instance of AAD. Represents a single organization.
* Azure Subscription - Used to pay for Azure services. Each subscription is joined to a single tenant.

Comparison between AAD and Active Directory Domain Services (AD DS) which is the traditional deployment of AD on Windows Server:

* AAD is a full identity solution while AD DS is primarily a directory service
* AAD doesn't use Kerberos auth. AAD uses HTTP and HTTPS protocols such as SAML, WS-Federation and OpenID Connect for authentication and OAuth for authorization
* AAD users and groups are created in a flat structure
* AAD is a managed service, you only manage users, groups and policies

AAD is available in 4 versions

* AAD Free - The Free edition provides user and group management, on-premises directory synchronization, and basic reports. Single sign-on access is supported across Azure, Microsoft 365, and many popular SaaS apps
* AAD Microsoft 365 - Everything from free edition + IAM for MS 365 apps, MFA and self-service pw reset
* AAD Premium P1 - Everything from 365 + hybrid user access (on-prem and cloud), advanced administration like dynamic groups, self-service group management and cloud write-back capabilities
* AAD Premium P2 - Everything from P1 + Azure AD Identity Protection which provides risk-based conditional access and Privileged Identity Management (PIM) used to discover, restrict and monitor administrators and their access to resources and provide just-in-time access when needed

AAD Join is used to join devices on AAD (used with notebooks and cellphones) giving benefits like using SSO to logon, enterprise state roaming (configure new device), access to defined apps on MS Store for business and provide restriction so defined apps can only be used from joined devices
