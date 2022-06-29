# Defender for Endpoint VM Scope Limitations

*Authored by:*

*Shane Neff*

*Senior Cloud Solution Architect- Microsoft*

*6/29/2022*

---

## PROBLEM STATEMENT

Customer has a large number of subscriptions containing thousands of virtual machines. The customer hosts workloads for their customers, which results in issues with managing Endpoint Protection for these WMs in the Microsoft Defender for Endpoint (MDE) portal. Currently, any user with the Azure Active Directory (AAD) Security Reader and Security Administrator roles are able to view VM details and manage the endpoint protection on any VM across the entire tenant. 

## SCOPE

The customer has requested a solution that solves this problem by restricting the scope of what the users (SOC teams, support teams, security teams, etc.) can see in the MDE portal to only the VMs that that team is responsible for, or to restrict the ability of other organizations from viewing or managing those VMs.

## OPTIONS

There are several ways this can be accomplished; each with their advantages and disadvantages. The only way to limit this scope begins with creating [device groups](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/machine-groups?view=o365-worldwide) in the context of role-based access (RBAC) and then assign permissions to AAD groups to the device groups. After access has been migrated from the directory role-based to directory security group granular assignments, only users in those groups assigned the AAD Security Reader and Security Administrator roles that is also assigned permissions to that device group will have the ability to view/manage endpoint devices in the MDE portal.

Device groups can be created and VMs can be added to them using one or many of the multiple available machine property values:

- Name of the machine
- Domain membership
- Tags
- Operating System

This demonstration will use the Tag Values for each machine, but any of the above filtering values would provide the same results.

---

### Defender for Endpoint API Calls

There are many [Defender for Endpoint APIs](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/management-apis?view=o365-worldwide) that can be leveraged to onboard and manage machines and many other MDE configurations. 

This approach uses the [Get machine by ID API](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/get-machine-by-id?view=o365-worldwide) to retrieve an exported list of machines registered and on-boarded in the MDE portal, and then using the exported list of machines retrieved to add/remove machine tags using the [Remove Machine Tags API](https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/add-or-remove-machine-tags?view=o365-worldwide)

#### Pros

- No host changes are required
- All changes are made to the meta data for the services
- Short deployment times
- Ability to deploy/use the solution to update machine tag values quickly 

#### Cons

- Limited customer experience with working with APIs
- Determining tag values will require teams to determine which machines should have which tag
- Rate limitations for this API are 100 calls per minute and 1500 calls per hour

These scripts can be run using the following example methods:

- Automation Runbook
- Function or Logic app that uses either a HTTP Hook trigger or on a set schedule
- CI/CD Pipeline that can be triggered using set schedule, HTTP web hook, pull request, commit, etc.

---

### AUTOMATED POWERSHELL/BASH

This method would utilize PowerShell and Linux Bash scripts to configure device tags at the operating system level.

Windows VMs:
- Add device tags using a small PowerShell script to add/set a Windows registry key value

```powershell
New-ItemProperty -Path "Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection\DeviceTagging\" -Name Group -PropertyType String -Value <value of the tag>

This script could be deployed using the following example methods:

- Desired State Configuration
- Custom script extension
- Automation Runbook
- Function or Logic app that uses either a HTTP Hook trigger or on a set schedule

```

Linux VMs:
- Add device tags utilizing the [mde_installer.sh installer script](https://github.com/microsoft/mdatp-xplat/tree/master/linux/installation) with the flag for VM tagging with the tag values to be assigned.

```bash
 sudo ~/mde_installer.sh --install --channel prod --onboard ~/linux_onboarding_script.py --tag GROUP Coders --min_req -y
```

#### Pros

- Customer has extensive PowerShell experience
- Can be integrated into any machine onboarding scripts
- Creating tags using VM property values in Azure to assist in tagging

#### Cons

- Using PowerShell to make OS calls/updates take a long time, and often time out before completion
- Customer has limited Linux Bash experience

## APPENDIX

MDE portal RBAC permissions: https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/rbac?view=o365-worldwide

MDE Device Groups: https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/machine-groups?view=o365-worldwide

MDE Management APIs: https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/management-apis?view=o365-worldwide

Get machine by ID API: https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/get-machine-by-id?view=o365-worldwide

Remove Machine Tags API: https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/add-or-remove-machine-tags?view=o365-worldwide

Add device tags utilizing the mde_installer.sh installer script: https://github.com/microsoft/mdatp-xplat/tree/master/linux/installation

Add device tags using PowerShell to add/set a Windows registry key value: https://docs.microsoft.com/en-us/microsoft-365/security/defender-endpoint/machine-tags?view=o365-worldwide#add-device-tags-by-setting-a-registry-key-value



