```
Disclaimer: The scripts are not supported under any Microsoft standard support program or service. 
The scripts are provided AS IS without warranty of any kind. Microsoft further disclaims all implied 
warranties including, without limitation, any implied warranties of merchantability or of fitness for a 
particular purpose. The entire risk arising out of the use or performance of the scripts and 
documentation remains with you. In no event shall Microsoft, its authors, or anyone else involved in the 
creation, production, or delivery of the scripts be liable for any damages whatsoever (including, without 
limitation, damages for loss of business profits, business interruption, loss of business information, or 
other pecuniary loss) arising out of the use of or inability to use the scripts or documentation, 
even if Microsoft has been advised of the possibility of such damages. 
```

# GPAD Tools PowerShell Module

## Overview
The **Group Provisioning to Active Directory (GPAD) Tools PowerShell Module** streamlines the management of group provisioning from Microsoft Entra ID to Active Directory using the Microsoft Graph PowerShell SDK. This module provides a comprehensive set of cmdlets designed to:

- Ensure consistency of group memberships between Microsoft Entra ID and on-premises Active Directory.
- Manage directory extension attributes used for group writeback filtering.

To learn more, refer to the following resources:
- [Migrate Microsoft Entra Connect Sync Group Writeback v2 to Microsoft Entra Cloud Sync](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/migrate-group-writeback).
- [Provision Microsoft Entra ID to Active Directory - Configuration](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/how-to-configure-entra-to-active-directory)
- [Scenario - Using directory extensions with group provisioning to Active Directory](https://learn.microsoft.com/en-us/entra/identity/hybrid/cloud-sync/tutorial-directory-extension-group-provisioning)


## Prerequisites

- Microsoft Active Directory PowerShell Module (on Windows Server):
    ```PowerShell
    Install-WindowsFeature "RSAT-AD-Tools"
    ```
- Microsoft Graph PowerShell SDK:
    ```PowerShell
    Install-Module Microsoft.Graph
    ```
- Connect to Microsoft Graph
    ```PowerShell
    Connect-MgGraph -Scopes "Application.ReadWrite.All, Directory.ReadWrite.All, Synchronization.ReadWrite.All"
    ```
## Installing or updating

The **GpadTools PowerShell module** is available for install from [PowerShell Gallery](https://www.powershellgallery.com/packages/GpadTools/).

- Install module:
    ```PowerShell
    Install-Module GpadTools
    ```
- Update module (if already installed):
    ```PowerShell
    Uninstall-Module GpadTools
    Install-Module GpadTools
    ```
    > Note: I don't recommended using `Update-Module GpadTools`, as that will install the new version while leaving the old version. This can lead to version conflicts and unexpected behavior when loading the module.

## Usage

Get all security groups enabled for writeback
```PowerShell
Get-GpadGroupFromEntra | ft -AutoSize
```
![Results from getting all security groups](https://github.com/user-attachments/assets/6fcb976a-7080-47c7-b757-a91f10bd8022)

Check if reconciliation is needed between a Microsoft Entra group and its corresponding on-premises Active Directory group.
```PowerShell
'defc####-####-####-####-####b684b71a' | Confirm-GpadReconciliationNeeded
```
![Results from checking if reconciliation needed for one group](https://github.com/user-attachments/assets/e755d3e9-4b7f-4d8b-828c-1634e2eb9263)

Check if reconciliation is needed for all groups
```PowerShell
Get-GpadGroupFromEntra | %{Confirm-GpadReconciliationNeeded -GroupId $_.Id}
```
![Results from checking if reconciliation needed for all groups](https://github.com/user-attachments/assets/aaa33469-a851-4300-adcc-9f9d065c268a)






## Functions

### Get-GpadCustomExtensionApplication

Retrieves the Microsoft Entra ID application used for Cloud Sync custom extensions.

Returns:
Entra Application object

Example:
```PowerShell
 Get-GpadCustomExtensionApplication
```

### Get-GpadCustomExtensionAppServicePrincipal

 Retrieves the service principal for the Cloud Sync custom extensions application.

Returns:
Entra Service Principal object

Example:
```PowerShell
Get-GpadCustomExtensionAppServicePrincipal
```

### Get-GpadCustomExtensionProperty

Retrieves custom extension properties from the Cloud Sync custom extensions application.

Returns:
Custom extension property

Example:
```PowerShell
Get-GpadCustomExtensionProperty
```

### Get-GpadGroupFromAD

Retrieves groups from Active Directory

Returns:
Active Directory object

Example:
```PowerShell
Get-GpadGroupFromAD -GroupId <guid>
```

### Get-GpadGroupFromEntra

Retrieves one or all security groups from Entra ID, excluding cloud M365 / Unified groups.
Exposes the WritebackConfiguration property (group schema) and the WritebackEnabled custom extension (directory schema).

Returns:

Entra Group Object, or;

`System.Collections.ArrayList` — List of security groups from Microsoft Entra.

Example:
```PowerShell
Get-GpadGroupFromEntra
```
```PowerShell
Get-GpadGroupFromEntra -GroupId <guid>
```


### Get-GpadGroupMembersFromAD

Retrieves group members from an Active Directory Group

Returns:
List of Active Directory objects representing the group members

Example:
```PowerShell
Get-GpadGroupMembersFromAD -GroupId <guid>
```

### Get-GpadGroupMembersFromEntra

Retrieves the members of a group from Entra ID.
With onPremisesSyncEnabledMembers switch, gets only the members that are synced from on-premises AD

Returns:
`System.Collections.ArrayList` — List of group members with object ID and type.

Example:
```PowerShell
Get-GpadGroupMembersFromEntra -GroupId <guid>
```

### Get-GpadWritebackEnabledExtensionName

Retrieves the name of the custom extension property used to determine if the group is enabled for writeback.

Returns:
`System.String` — Name of the custom extension property.

Example:
```PowerShell
Get-GpadWritebackEnabledExtensionName
```

### Get-GpadSynchronizationIdentifiers

Retrieves the service principal ID, synchronization job ID, and synchronization rules ID for a given domain.

Parameters: 
`-DomainName <String>`: Domain name to retrieve identifiers for.

Returns: 
`PSCustomObject` — Contains ServicePrincipalId, SynchronizationJobId, and SynchronizationRulesId.

Example:
```PowerShell
Get-GpadSynchronizationIdentifiers -DomainName Contoso.com
```

### New-GpadCustomExtensionApplication

Creates a new Cloud Sync custom extensions application in Microsoft Entra ID.

Example:
```PowerShell
New-GpadCustomExtensionApplication
```

### New-GpadCustomExtensionAppServicePrincipal

Creates a service principal for the Cloud Sync custom extensions application.

Returns:
`System.String` — Name of...

Example:
```PowerShell
New-GpadCustomExtensionAppServicePrincipal
```

### New-GpadCustomExtensionProperty

Creates a new custom extension property on the Cloud Sync custom extensions application.

Example:
```PowerShell
New-GpadCustomExtensionProperty
```

### Remove-GpadCustomExtensionProperty

Removes a custom extension property from the Cloud Sync custom extensions application.

Example:
```PowerShell
Remove-GpadCustomExtensionProperty
```

### Set-GpadWritebackConfiguration

Set IsEnabled True/False property in WritebackConfiguration
This function sets the WritebackConfiguration.IsEnabled property (Group schema) for a group in Entra ID
To get this group property use Get-GpadGroupFromEntra

Example:
```PowerShell
Set-GpadWritebackConfiguration -GroupId "3b10####-####-####-####-####9835c3e0" -IsEnabled $true
```

### Set-GpadWritebackEnabledExtension

Sets the WritebackEnabled customer extension for a cloud group.
This function updates (True/False) the WritebackEnabled custom extension (directory schema) for a given group.
To get this group property use Get-GpadGroupFromEntra

Example:
```PowerShell
Set-GpadWritebackEnabledExtension -GroupId "3b10####-####-####-####-####9835c3e0" -Value 'True'
```


### Update-GpadWritebackEnabledExtension

Updates the WritebackEnabled extension property for all security groups based on their WritebackConfiguration.

Example:
```PowerShell
Update-GpadWritebackEnabledExtension
```

### Confirm-GpadReconciliationNeeded

Checks if reconciliation is needed between an Entra ID group and its corresponding AD group.

Parameters:

`-GroupId <String[]>`: One or more Entra ID group IDs.

`-GroupWritebackOU <String>` (Optional): Group writeback OU in Active Directory.

Returns:
`System.Collections.ArrayList` — List of group members with object ID, type and reconciliation needed flag.

Example:
```PowerShell
Confirm-GpadReconciliationNeeded -GroupId "3b10####-####-####-####-####9835c3e0"
```
```PowerShell
Confirm-GpadReconciliationNeeded -GroupId "3b10####-####-####-####-####9835c3e0" -GroupWritebackOU "OU=Groups,DC=Contoso,DC=com"
```

