# GPAD Tools PowerShell Module

> **Disclaimer:** The scripts are not supported under any Microsoft standard support program or service. The scripts are provided AS IS without warranty of any kind. Microsoft further disclaims all implied warranties including, without limitation, any implied warranties of merchantability or of fitness for a particular purpose. The entire risk arising out of the use or performance of the scripts and documentation remains with you. In no event shall Microsoft, its authors, or anyone else involved in the creation, production, or delivery of the scripts be liable for any damages whatsoever (including, without limitation, damages for loss of business profits, business interruption, loss of business information, or other pecuniary loss) arising out of the use of or inability to use the scripts or documentation, even if Microsoft has been advised of the possibility of such damages.

## Overview

The Group Provisioning to Active Directory (GPAD) Tools PowerShell Module streamlines the management of group provisioning from Microsoft Entra ID to Active Directory using the Microsoft Graph PowerShell SDK. This module provides a comprehensive set of cmdlets designed to:

• Ensure consistency of group memberships between Microsoft Entra ID and on-premises Active Directory.
• Manage directory extension attributes used for group writeback filtering.
• Perform reconciliation checks between cloud and on-premises groups.
• Configure writeback settings for individual groups or in bulk.

To learn more, refer to the following resources:

• [Migrate Microsoft Entra Connect Sync Group Writeback v2 to Microsoft Entra Cloud Sync](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/migrate-group-writeback)

• [Provision Microsoft Entra ID to Active Directory - Configuration](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/how-to-configure-entra-to-active-directory)

• [Scenario - Using directory extensions with group provisioning to Active Directory](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/tutorial-directory-extension-group-provisioning)

## Prerequisites

• **Microsoft Active Directory PowerShell Module** (on Windows Server): `Install-WindowsFeature "RSAT-AD-Tools"`

• **Microsoft Graph PowerShell SDK**: `Install-Module Microsoft.Graph`

• **Connect to Microsoft Graph**: `Connect-MgGraph -Scopes "Application.ReadWrite.All, Directory.ReadWrite.All, Synchronization.ReadWrite.All"`

## Installing or updating

The GpadTools PowerShell module is available for install from [PowerShell Gallery](https://www.powershellgallery.com/packages/GpadTools/).

• **Install module**: `Install-Module GpadTools`
• **Update module** (if already installed): 
  ```powershell
  Uninstall-Module GpadTools
  Install-Module GpadTools
  ```

> **Note:** I don't recommend using `Update-Module GpadTools`, as that will install the new version while leaving the old version. This can lead to version conflicts and unexpected behavior when loading the module.

## Usage

### Get all security groups enabled for writeback

```powershell
Get-GpadGroupFromEntra | Format-Table -AutoSize
```

### Set writeback configuration for a group

```powershell
Set-GpadWritebackConfiguration -GroupId "3b10####-####-####-####-####9835c3e0" -IsEnabled $true
```

### Set writeback enabled extension for a group

```powershell
Set-GpadWritebackEnabledExtension -GroupId "3b10####-####-####-####-####9835c3e0" -IsEnabled $true
```

### Check if reconciliation is needed between a Microsoft Entra group and its corresponding on-premises Active Directory group

```powershell
'defc####-####-####-####-####b684b71a' | Confirm-GpadReconciliationNeeded
```

### Check if reconciliation is needed for all groups

```powershell
Get-GpadGroupFromEntra | ForEach-Object { Confirm-GpadReconciliationNeeded -GroupId $_.Id }
```
```powershell
Get-GpadGroupFromEntra | select -ExpandProperty Id | Confirm-GpadReconciliationNeeded
```

> **Important**
> 
> If there's a mismatch between the group in Entra and its corresponding on-premises group—typically caused by changes made directly on-premises—this function can help identify missing members to re-add or extra members to remove. If the cause of the inconsistency is unclear, please open a support case for further investigation.

## Functions

### Get-GpadCustomExtensionApplication

Retrieves the Microsoft Entra ID application used for Cloud Sync custom extensions.

**Returns:** Entra Application object

**Example:**
```powershell
Get-GpadCustomExtensionApplication
```

### Get-GpadCustomExtensionAppServicePrincipal

Retrieves the service principal for the Cloud Sync custom extensions application.

**Returns:** Entra Service Principal object

**Example:**
```powershell
Get-GpadCustomExtensionAppServicePrincipal
```

### Get-GpadCustomExtensionProperty

Retrieves custom extension properties from the Cloud Sync custom extensions application.

**Returns:** Custom extension property

**Example:**
```powershell
Get-GpadCustomExtensionProperty
```

### Get-GpadGroupFromAD

Retrieves groups from Active Directory.

**Parameters:**
- `-GroupId <String>`: The GUID of the group to retrieve from Active Directory.

**Returns:** Active Directory object

**Example:**
```powershell
Get-GpadGroupFromAD -GroupId <guid>
```

### Get-GpadGroupFromEntra

Retrieves one or all security groups from Entra ID, excluding cloud M365 / Unified groups. Exposes the WritebackConfiguration property (group schema) and the WritebackEnabled custom extension (directory schema).

**Parameters:**
- `-GroupId <String>` (Optional): The GUID of a specific group to retrieve.

**Returns:** Entra Group Object, or `System.Collections.ArrayList` — List of security groups from Microsoft Entra.

**Examples:**
```powershell
Get-GpadGroupFromEntra
```
```powershell
Get-GpadGroupFromEntra -GroupId <guid>
```

### Get-GpadGroupMembersFromAD

Retrieves group members from an Active Directory Group.

**Parameters:**
- `-GroupFromAD <Object>`: The group object from Active Directory, including the member property.

**Returns:** List of Active Directory objects representing the group members

**Example:**
```powershell
Get-GpadGroupMembersFromAD -GroupFromAD <group object>
```

### Get-GpadGroupMembersFromEntra

Retrieves the members of a group from Entra ID. With `OnPremisesSyncEnabledMembers` switch, gets only the members that are synced from on-premises AD.

**Parameters:**
- `-GroupId <String>`: The GUID of the group to get members for.
- `-OnPremisesSyncEnabledMembers` (Switch): Only return members that are synced from on-premises AD.

**Returns:** `System.Collections.ArrayList` — List of group members with object ID and type.

**Example:**
```powershell
Get-GpadGroupMembersFromEntra -GroupId <guid>
```

### Get-GpadWritebackEnabledExtensionName

Retrieves the name of the custom extension property used to determine if the group is enabled for writeback.

**Returns:** `System.String` — Name of the custom extension property.

**Example:**
```powershell
Get-GpadWritebackEnabledExtensionName
```

### Get-GpadSynchronizationIdentifiers

Retrieves the service principal ID, synchronization job ID, and synchronization rules ID for a given domain.

**Parameters:**
- `-DomainName <String>`: Domain name to retrieve identifiers for.

**Returns:** `PSCustomObject` — Contains ServicePrincipalId, SynchronizationJobId, and SynchronizationRulesId.

**Example:**
```powershell
Get-GpadSynchronizationIdentifiers -DomainName "Contoso.com"
```

### New-GpadCustomExtensionApplication

Creates a new Cloud Sync custom extensions application in Microsoft Entra ID.

**Example:**
```powershell
New-GpadCustomExtensionApplication
```

### New-GpadCustomExtensionAppServicePrincipal

Creates a service principal for the Cloud Sync custom extensions application.

**Returns:** `System.String` — Service principal object ID.

**Example:**
```powershell
New-GpadCustomExtensionAppServicePrincipal
```

### New-GpadCustomExtensionProperty

Creates a new custom extension property on the Cloud Sync custom extensions application.

**Example:**
```powershell
New-GpadCustomExtensionProperty
```

### Remove-GpadCustomExtensionProperty

Removes a custom extension property from the Cloud Sync custom extensions application.

**Example:**
```powershell
Remove-GpadCustomExtensionProperty
```

### Set-GpadWritebackConfiguration

Sets the WritebackConfiguration.IsEnabled property (Group schema) for a group in Entra ID.

**Parameters:**
- `-GroupId <String[]>`: The identifier(s) of the Entra ID group(s) to set the WritebackConfiguration for.
- `-IsEnabled <Boolean>`: The boolean value (True/False) to set for the WritebackConfiguration.IsEnabled property.

**Examples:**
```powershell
Set-GpadWritebackConfiguration -GroupId "3b10####-####-####-####-####9835c3e0" -IsEnabled $true
```
```powershell
'b92b####-####-####-####-####8c6fba8d', 'defc####-####-####-####-####b684b71a' | Set-GpadWritebackConfiguration -IsEnabled $true
```

### Set-GpadWritebackEnabledExtension

Sets the WritebackEnabled custom extension (directory schema) for a cloud group.

**Parameters:**
- `-GroupId <String[]>`: The identifier(s) of the Entra ID group(s) to set the WritebackEnabled extension for.
- `-IsEnabled <Boolean>`: The boolean value (True/False) to set for the WritebackEnabled extension.

**Examples:**
```powershell
Set-GpadWritebackEnabledExtension -GroupId "3b10####-####-####-####-####9835c3e0" -IsEnabled $true
```
```powershell
'b92b####-####-####-####-####8c6fba8d', 'defc####-####-####-####-####b684b71a' | Set-GpadWritebackEnabledExtension -IsEnabled $true
```

### Update-GpadWritebackEnabledExtension

Updates the WritebackEnabled extension property for all security groups based on their WritebackConfiguration.

**Example:**
```powershell
Update-GpadWritebackEnabledExtension
```

### Confirm-GpadReconciliationNeeded

Checks if reconciliation is needed between an Entra ID group and its corresponding AD group.

**Parameters:**
- `-GroupId <String[]>`: One or more Entra ID group IDs.
- `-GroupWritebackOU <String>` (Optional): Group writeback OU in Active Directory.

**Returns:** `System.Collections.ArrayList` — List of group members with object ID, type and reconciliation needed flag.

**Examples:**
```powershell
Confirm-GpadReconciliationNeeded -GroupId "3b10####-####-####-####-####9835c3e0"
```
```powershell
Confirm-GpadReconciliationNeeded -GroupId "3b10####-####-####-####-####9835c3e0" -GroupWritebackOU "OU=Groups,DC=Contoso,DC=com"
```

## Additional Links

- [Code](https://github.com/NuAlex/GpadTools/tree/v0.0.3)
- [Issues](https://github.com/NuAlex/GpadTools/issues)
- [Pull requests](https://github.com/NuAlex/GpadTools/pulls)
- [Discussions](https://github.com/NuAlex/GpadTools/discussions)
- [Actions](https://github.com/NuAlex/GpadTools/actions)
- [Projects](https://github.com/NuAlex/GpadTools/projects)
- [Security](https://github.com/NuAlex/GpadTools/security)
- [PowerShell Gallery](https://www.powershellgallery.com/packages/GpadTools/)
