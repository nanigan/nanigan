# HOW TO PROVIDE ACCESS TO CUSTOMER'S TENANT USING ALTERNATE MFA
*Shane Neff*

*Senior Cloud Solution Architect- Microsoft*

*6/2/2022*

---

# TABLE OF CONTENTS

- [HOW TO PROVIDE ACCESS TO CUSTOMER'S TENANT USING ALTERNATE MFA](#how-to-provide-access-to-customers-tenant-using-alternate-mfa)
- [TABLE OF CONTENTS](#table-of-contents)
- [PROBLEM STATEMENT](#problem-statement)
  - [Problem statement](#problem-statement)
  - [Supplemental Details](#supplemental-details)
  - [Additional Requirements/Requests](#additional-requirementsrequests)
- [CUSTOMER ENVIRONMENT](#customer-environment)
- [OPTIONS](#options)
  - [B2C Tenant](#b2c-tenant)
  - [Guest users in the customers tenant (B2B)](#guest-users-in-the-customers-tenant-b2b)
  - [Exporting SharePoint Data to Azure Storage Account](#exporting-sharepoint-data-to-azure-storage-account)
  - [Cross-Tenant Access with Azure AD External IDs (Preview)](#cross-tenant-access-with-azure-ad-external-ids-preview)
- [AUTHENTICATION METHODS](#authentication-methods)
- [RECOMMENDATIONS](#recommendations)
  - [MFA Authentication Method Recommendations](#mfa-authentication-method-recommendations)
  - [External User Recommendation](#external-user-recommendation)
  - [Solution Details](#solution-details)

---

# PROBLEM STATEMENT

## Problem statement
How to enforce MFA for external users using an alternate authentication method than that of internal associates, while also enabling authenticated access to the resources residing in SharePoint.

---

## Supplemental Details

The customer has many partners, vendors, and contractors that they collaborate with, or provide services to. In the past, these external IDs were invited to the customer's production Azure Active Directory tenant. 

The customer's tenant is configured to require MFA for every external login attempt. Permitted authentication methods are configured at the tenant level and cannot be modified differently for a subset of users or groups. The authentication methods include the Microsoft Authenticator, which is dependent upon the user possessing a mobile device in order to complete the MFA challenge.

The customer would like to maintain their current MFA solution for their internal users, but have requested a solution that can enforce MFA for their external partners, but utilize alternate authentication methods that do not require a mobile device.

---

## Additional Requirements/Requests

In addition to enforcing MFA for external partners, these users must be permitted to access various SharePoint sites residing in the customer's production tenant SharePoint Online instance. Furthermore, many of the documents accessed are considered confidential and include sensitive internal documentation.


---

# CUSTOMER ENVIRONMENT

- Conditional access policies enforcing MFA for all users
- MFA authentication method: Microsoft Authenticator App
- SharePoint Online tenant

---

# OPTIONS

## B2C Tenant

The B2C (Business to Consumer) service is typically (not always) geared toward providing access to external users to consume applications as consumers or customers for internally developed business applications without any identity management related to the customer's production tenant

- PRO
    - External user IDs are provisioned in a separate tenant than the customer's
    - External user IDs are federated, which results in no user ID management
    - Supports multiple identity providers (Facebook, Twitter, email, etc.)
    - Ability to leverage user flows to allow external users sign up through a self-registration process
    - Customize MFA configuration separately for external users
    
- CON
    - Since the IDs are not provisioned in the customer's tenant, they will not have access to internal SharePoint sites (unless publicly accessible)
    - Additional administrative tasks management an additional Azure Active Directory tenant

- DOCUMENTATION
    - [B2C Overview](https://docs.microsoft.com/en-us/azure/active-directory-b2c/overview)
    - [Compare external identity options](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/external-identities-overview?bc=%2Fazure%2Factive-directory-b2c%2Fbread%2Ftoc.json&toc=%2Fazure%2Factive-directory-b2c%2FTOC.json)
    - [Identity Protection and Conditional Access for Azure AD B2C](https://docs.microsoft.com/en-us/azure/active-directory-b2c/conditional-access-identity-protection-overview)

---

## Guest users in the customers tenant (B2B)

Business to Consumer (also known as B2B) refers to guest or external users that ae added to another tenant to enable cross-tenant collaboration, RBAC, conditional access, etc.) 

- PRO
    - Fewer administrative tasks since there will not be a separate tenant
    - Able to leverage M365, etc. licenses already purchased for external IDs

- CON
    - Unable to define different MFA authentication methods for external IDs

- DOCUMENTATION
    - [B2B Overview](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/what-is-b2b)
    - [Integrate with Identity providers](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/identity-providers)
    - [Authentication and Conditional Access for External Identities](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/authentication-conditional-access)
    
---

## Exporting SharePoint Data to Azure Storage Account 

- PRO
    - Ability to provide non-tenant access to the files
    - Enhanced security options (SAS Tokens providing time-based access)

- CON
    - Customer has stated that many of their partner relationships require Azure Information Protection/DLP to ensure document integrity is maintained
        - Potential workaround: storage account immutability

- DOCUMENTATION
    - Not applicable, as this would be a custom solution

---

## Cross-Tenant Access with Azure AD External IDs (Preview)

- PRO
    - Can be configured to [trust external users identity provider's MFA solution](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/cross-tenant-access-settings-b2b-direct-connect#to-change-inbound-trust-settings-for-mfa-and-device-state)
    - Enables customer to maintain their current MFA/Microsoft Authenticator authentication method, while also ensuring all external users connecting to resources in their tenant have been challenged for MFA
    - Enables external access to internal SharePoint sites

- CON
    - Users need to have the ability to invite external users to the tenant
        - Mitigated by limiting who can invite external users
    - External users are still added as guests in the customer's Azure Active Directory tenant
    - Some initial manual configuration tasks and administration may impact customer

- DOCUMENTATION
    - [Configure cross-tenant access settings for B2B direct connect](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/cross-tenant-access-settings-b2b-direct-connect#to-change-inbound-trust-settings-for-mfa-and-device-state)
    - [B2B Direct Connect](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/b2b-direct-connect-overview)
    - [inbound trust settings for MFA and device state](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/cross-tenant-access-settings-b2b-direct-connect#to-change-inbound-trust-settings-for-mfa-and-device-state)
    - [External collaboration settings](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/external-collaboration-settings-configure)

---

# AUTHENTICATION METHODS

Below are the lists of authentication methods supported and whether or not they are a supported method of secondary authentication




<table>
<tr>
<th>Method</th>
<th style="text-align: center;">Primary authentication</th>
<th style="text-align: center;">Secondary authentication</th>
</tr>
</thead>
<tbody>
<tr>
<td>Windows Hello for Business</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">MFA</td>
</tr>
<tr>
<td>Microsoft Authenticator app</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">MFA and SSPR</td>
</tr>
<tr>
<td>FIDO2 security key</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">MFA</td>
</tr>
<tr>
<td>OATH hardware tokens (preview)</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">MFA and SSPR</td>
</tr>
<tr>
<td>OATH software tokens</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">MFA and SSPR</td>
</tr>
<tr>
<td>SMS</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">MFA and SSPR</td>
</tr>
<tr>
<td>Voice call</td>
<td style="text-align: center;">No</td>
<td style="text-align: center;">MFA and SSPR</td>
</tr>
<tr>
<td>Password</td>
<td style="text-align: center;">Yes</td>
<td style="text-align: center;">N/A</td>
</tr>
</tbody>
</table>

Reference: 
[Supported authentication methods for login type](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-authentication-methods#how-each-authentication-method-works)

---

# RECOMMENDATIONS

Below are the recommendations for cross tenant MFA leveraging different authentication methods


## MFA Authentication Method Recommendations

- **Internal users:**
    - *Primary authentication*: This will remain- unchanged
    - *MFA authentication*: This will remain- unchanged


- **External users**
    - Primary authentication: Password
    - *MFA authentication*: FIDO2 security key

---

## External User Recommendation

Leverage [Cross-tenant access with Azure AD External Identities (Preview) using B2B direct connect](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/cross-tenant-access-overview) to honor service MFA claims from users' home directory by the resource directory

This will accomplish the following:

- External user can access SharePoint Online files they've been granted access to
- External user will be challenged for MFA, but the MFA challenge will be against the external user's home Azure Active Directory tenant utilizing their home directory's MFA authentication methods

---

## Solution Details

In an Azure AD cross-tenant scenario, the resource organization can create Conditional Access policies that require MFA or device compliance for all guest and external users. Generally, an external user accessing a resource is required to set up their Azure AD MFA with the resource tenant. However, Azure AD now offers the capability to trust MFA, compliant device claims, and hybrid Azure AD joined device claims from external Azure AD organizations, making for a more streamlined sign-in experience for the external user. As the resource tenant, you can use cross-tenant access settings to trust the MFA and device claims from external Azure AD tenants. Trust settings can apply to all Azure AD organizations, or just selected Azure AD organizations.

When trust settings are enabled, Azure AD will check a user's credentials during authentication for an MFA claim or a device ID to determine if the policies have already been met in their home tenant. If so, the external user will be granted seamless sign-on to your shared resource. Otherwise, an MFA or device challenge will be initiated in the user's home tenant. If trust settings aren't enabled, or if the user's credentials don't contain the required claims, the external user will be presented with an MFA or device challenge.

*Figure: Authentication flow for external Azure AD users*
![Authentication flow for external Azure AD users](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/media/authentication-conditional-access/cross-tenant-auth.png)

*Reference*: [Authentication flow for external Azure AD users](https://docs.microsoft.com/en-us/azure/active-directory/external-identities/authentication-conditional-access#authentication-flow-for-external-azure-ad-users)
