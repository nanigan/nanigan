## Approach

This solution will utilize [Azure's Guest Configuration feature](https://docs.microsoft.com/en-us/azure/governance/policy/concepts/guest-configuration) and [PowerShell Desired State Configuration v3](https://docs.microsoft.com/en-us/powershell/dsc/overview)

- Azure Policy's guest configuration feature provides native capability to audit or configure operating system settings as code, both for machines running in Azure and hybrid Arc-enabled machines. The feature can be used directly per-machine, or at-scale orchestrated by Azure Policy
    - The guest configuration agent checks for new or changed guest assignments every 5 minutes. Once a guest assignment is received, the settings for that configuration are rechecked on a 15-minute interval: https://docs.microsoft.com/en-us/azure/governance/policy/concepts/guest-configuration#validation-frequency

---

## Pre-Requisites

- Enable Resource Provider: Microsoft.GuestConfiguration
- [Install PowerShell 7 and required PowerShell modules](https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-windows?WT.mc_id=THOMASMAURER-blog-thmaure&view=powershell-7)
- Install GuestConfiguration PowerShell Module
- https://docs.microsoft.com/en-us/azure/virtual-machines/extensions/overview

Example:

```powershell
Install GuestConfiguration PowerShell Module
```
- Deploy the Guest Configuration VM extension To deploy the extension at scale across many machines, assign the policy initiative:
  - *"Deploy prerequisites to enable guest configuration policies on virtual machines to a management group, subscription, or resource group containing the machines that you plan to manage"*
- Ensure VM system managed Managed ID has been provisioned on each VM. If the machine doesn't currently have any managed identities, the effective policy is: 
    - *Add system-assigned managed identity to enable guest configuration assignments on virtual machines with no identities*

---

## Configuration Steps

1. Create configuration package

Example (Ensures the presence of a registry key):

```powershell
Configuration winreg {

    Import-DscResource -ModuleName PSDscResources

    Node 'localhost' {
            Registry DeviceTagging {
        
                Ensure = "Present"
                Key = "HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows Advanced Threat Protection\DeviceTagging"
                ValueName = "Group"
                ValueData = "testgroup"
                ValueType = "String" }
    }
}
```
2. Create Configuration Package

Example (calls the configuration package created in previous steps, and exports the .mof files and required modules to a directory called "winreg"):

```powershell
. .\DSC\createConfigPackage.ps1 
winreg
```
3. Compile DSC\MOF File

Example (this takes the .mof file as an input and creates the package .zip file needed to publish guest configuration):

```powershell
Start-DscConfiguration -Path .\winreg -Verbose -Wait -Force
```

4. Create a package that will audit/apply compliance

Example:

```powershell
New-GuestConfigurationPackage `
  -Name 'winregedit' `
  -Configuration '.\winreg\localhost.mof' `
  -Type AuditAndSet `
  -Force
```
5. Create Storage Account

Example (creates a storage account in a resource group, defines storage replication, creates a container, and sets the permission to "blob" ):

```powershell
New-AzStorageAccount -ResourceGroupName <name of resource group> -Name <name of storage account> -SkuName '<storage sku name>' -Location '<region>' | New-AzStorageContainer -Name <name of storage account container> -Permission Blob
```
6. Validate the configuration package meets requirements. This needs to point to the path of your .zip configuration package: https://docs.microsoft.com/en-us/azure/governance/policy/how-to/guest-configuration-create-test#validate-the-configuration-package-meets-requirements

Example:

```powershell
Get-GuestConfigurationPackageComplianceStatus <name of zip file.zip>
```
7. Set Storage Context

Example:

```powershell
$Context = New-AzStorageContext -ConnectionString "DefaultEndpointsProtocol=https;AccountName=<your storage account name>;AccountKey=<your storage account key>"

```
8. Publish Package

Example:

```powershell
Set-AzStorageBlobContent -Container "<your container name>" -File <file path and name of zip file.zip> -Blob "<name of storage account container>" -Force -Context $Context

```
9. Create an Azure Policy definition Audit/Enforce Policy

In order to Create a policy definition that deploys a configuration using a custom configuration package, in a specified path.

This policy has an effect type of *deployIfNotExists*

You will need to generate a GUID to use as the policy ID

Example:

```powershell
New-GuestConfigurationPolicy `
  -PolicyId '<your policy GUID>' `
  -ContentUri '<path to your storage account blob>' `
  -DisplayName 'Apply and Auto Correct the presence MDE Tags registry key on WIndows VMs.' `
  -Description 'This policy will Apply and Auto Correct the presence MDE Tags registry key on WIndows VMs.' `
  -Path 'policies/<your directory to output the policy>' `
  -Platform 'Windows' `
  -PolicyVersion 1.0.0 `
  -Mode 'ApplyAndAutoCorrect' `
  -Verbose
```
Parameters of the New-GuestConfigurationPolicy cmdlet:

- PolicyId: A GUID or other unique string that identifies the definition. 
    - GUIDS can be created using one of the many free GUID generator services such as https://guidgenerator.com/
- ContentUri: Public HTTP(s) URI of guest configuration content package.
- DisplayName: Policy display name.
- Description: Policy description.
- Parameter: Policy parameters provided in hashtable format.
- PolicyVersion: Policy version.
- Path: Destination path where policy definitions are created.
- Platform: Target platform (Windows/Linux) for guest configuration policy and content package.
- Mode: (ApplyAndMonitor, ApplyAndAutoCorrect, Audit) choose if the policy should audit or deploy the configuration. Default is "Audit".
- Tag adds one or more tag filters to the policy definition
- Category sets the category metadata field in the policy definition

10. Retrieve Package From Storage Account

Example:
```powershell
Set-AzStorageBlobContent -Container "<name of storage account container>" -File <file path and name of zip file.zip> -Blob "<name of storage account container>" -Context $Context
```

11. Publish the Azure Policy definition

Example (this will publish the policy definition from the directory policies\<your policy name>):

```powershell
New-AzPolicyDefinition -Name '<the display name of the policy created in step 11>' -Policy 'policies\<name of your policy>\**'
```

12. Get a reference to the subscription that is the scope of the assignment

Example:

```powershell
$sub = Get-AzSubscription -SubscriptionId '<your subscription ID>'
```
13. Get a reference to the built-in policy definition to assign

Example:

```powershell
$definition = Get-AzPolicyDefinition | Where-Object { $_.Properties.DisplayName -eq '<the display name of the policy created in step 10>' }
```

14. Create a policy assignment

Example:

```powershell
New-AzPolicyAssignment -Name '<name of your policy assignment>' -DisplayName '<the display name of the policy created in step 10>' -Scope "/subscriptions/$($sub.Id)" -PolicyDefinition $definition -IdentityType 'SystemAssigned' -Location '<region>'
```
---
## Planned Changes

Planned Approach will use [CIM Sessions](https://docs.microsoft.com/en-us/powershell/module/cimcmdlets/new-cimsession?view=powershell-7.2#example-3-create-a-cim-session-to-multiple-computers) to use a list of VMs with system properties to define which machines to deploy the tag policy to.
 
1. Create a list of VMs with properties and export to .CSV

Example:

```powershell
$reportName = "vmList.csv"
(Get-AzSubscription)|ForEach-Object{
 
 Select-AzSubscription $_

$report = @()
$vms = Get-AzVM
$publicIps = Get-AzPublicIpAddress 
$nics = Get-AzNetworkInterface | ?{ $_.VirtualMachine -NE $null}
foreach ($nic in $nics) { 
    $info = "" | Select-Object VmName, ResourceGroupName, Region, VirturalNetwork, Subnet, PrivateIpAddress, OsType, PublicIPAddress, SubscriptionName
    $vm = $vms | ? -Property Id -eq $nic.VirtualMachine.id 
    foreach($publicIp in $publicIps) { 
        if($nic.IpConfigurations.id -eq $publicIp.ipconfiguration.Id) {
            $info.PublicIPAddress = $publicIp.ipaddress
            } 
        } 
        $info.OsType = $vm.StorageProfile.OsDisk.OsType 
        $info.VMName = $vm.Name 
        $info.ResourceGroupName = $vm.ResourceGroupName 
        $info.Region = $vm.Location 
        $info.VirturalNetwork = $nic.IpConfigurations.subnet.Id.Split("/")[-3] 
        $info.Subnet = $nic.IpConfigurations.subnet.Id.Split("/")[-1] 
        $info.PrivateIpAddress = $nic.IpConfigurations.PrivateIpAddress
        $info.SubscriptionName=$_.Name 
        $report+=$info 
    } 
$report | Export-CSV ".\DSC\$reportName" -NoTypeInformation -Append

}
```
2. Use .CSV as input to policy creation
Example:

```powershell
$ComputerName = input -path <path to .CSV> object vmName and Subscription ID (any property will work)
$Session = New-CimSession -ComputerName vm-configuration-w2k16
Start-DscConfiguration -Path .\winreg -Verbose -Wait -CimSession $Session
```
3. [Using parameters in custom guest configuration policy definitions](https://docs.microsoft.com/en-us/azure/governance/policy/how-to/guest-configuration-create-definition#using-parameters-in-custom-guest-configuration-policy-definitions)

```powershell
# This DSC Resource text:
Service 'UserSelectedNameExample'
  {
    Name = 'ParameterValue'
    Ensure = 'Present'
    State = 'Running'
  }`

# Would require the following hashtable:
$PolicyParameterInfo = @(
  @{
    Name = 'ServiceName'                                           # Policy parameter name (mandatory)
    DisplayName = 'windows service name.'                          # Policy parameter display name (mandatory)
    Description = 'Name of the windows service to be audited.'     # Policy parameter description (optional)
    ResourceType = 'Service'                                       # DSC configuration resource type (mandatory)
    ResourceId = 'UserSelectedNameExample'                         # DSC configuration resource id (mandatory)
    ResourcePropertyName = 'Name'                                  # DSC configuration resource property name (mandatory)
    DefaultValue = 'winrm'                                         # Policy parameter default value (optional)
    AllowedValues = @('BDESVC','TermService','wuauserv','winrm')   # Policy parameter allowed values (optional)
  }
)

New-GuestConfigurationPolicy `
  -PolicyId 'My GUID' `
  -ContentUri '<paste the ContentUri output from the Publish command>' `
  -DisplayName 'Audit Windows Service.' `
  -Description 'Audit if a Windows Service isn't enabled on Windows machine.' `
  -Path '.\policies' `
  -Parameter $PolicyParameterInfo `
  -PolicyVersion 1.0.0
```

---

## Notes/Limitations

- Azure Policy definitions in the category 'Guest Configuration' can be assigned to Management Groups only when the effect is 'AuditIfNotExists'. Policy definitions with effect 'DeployIfNotExists' aren't supported as assignments to Management Groups
- Remote PowerShell must be enabled on target VMs
    
