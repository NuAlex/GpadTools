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

# GPAD Tools PowerShell Module (GpadTools)

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
## Installing or updating GpadTools PowerShell Module

- Install module:
    ```PowerShell
    Install-Module GpadTools
    ```
- Update module (if already installed):
    ```PowerShell
    Uninstall-Module GpadTools
    Install-Module GpadTools
    ```
    > Note: It is not recommended to use Update-Module GpadTools directly, as it may install the new version while leaving the old version. This can lead to version conflicts and unexpected behavior when loading the module.

## Usage

Get all security groups enabled for writeback
```PowerShell
Get-GpadCloudSecurityGroups | ft -AutoSize
```
![Results from getting all security groups](https://github.com/user-attachments/assets/6fcb976a-7080-47c7-b757-a91f10bd8022)

Check if reconciliation is needed between a Microsoft Entra group and its corresponding on-premises Active Directory group.
```PowerShell
'defc####-####-####-####-####b684b71a' | Confirm-GpadReconciliationNeeded
```
![Results from checking if reconciliation needed for one group](https://github.com/user-attachments/assets/e755d3e9-4b7f-4d8b-828c-1634e2eb9263)

Check if reconciliation is needed for all groups
```PowerShell
Get-GpadCloudSecurityGroups | %{Confirm-GpadReconciliationNeeded -GroupId $_.Id}
```
![Results from checking if reconciliation needed for all groups](https://github.com/user-attachments/assets/aaa33469-a851-4300-adcc-9f9d065c268a)

Get Synchronization Job Identifiers
```PowerShell
$onDemandProv = Get-GpadSynchronizationIdentifiers -DomainName 'Contoso.com'
```

Call On-Demand provisioning for an Entra ID group (group reconciliation)
```PowerShell
$onDemandProv = Get-GpadSynchronizationIdentifiers -DomainName 'Contoso.com'
$groupId = 'defc####-####-####-####-####b684b71a'

Start-GpadOnDemandProvisionGroup -GroupId $groupId `
    -ServicePrincipalId $onDemandProv.ServicePrincipalId  `
    -SynchronizationJobId $onDemandProv.SynchronizationJobId  `
    -SynchronizationRulesId $onDemandProv.SynchronizationRulesId
``` 
![Results from On-Demand provisioning for an Entra ID group](https://github.com/user-attachments/assets/35911db4-2d2b-4ce6-9f98-d0dd2a272aee)

Call On-Demand provisioning for all groups needing reconciliation (group reconciliation in bulk)
```PowerShell
$onDemandProv = Get-GpadSynchronizationIdentifiers -DomainName 'Contoso.com'
$groups = Get-GpadCloudSecurityGroups | %{Confirm-GpadReconciliationNeeded -GroupId $_.Id}

$results = $groups | Where-Object {$_.ReconciliationNeeded -eq $true} | select -ExpandProperty Id |
    Start-GpadOnDemandProvisionGroup `
        -ServicePrincipalId $onDemandProv.ServicePrincipalId  `
        -SynchronizationJobId $onDemandProv.SynchronizationJobId  `
        -SynchronizationRulesId $onDemandProv.SynchronizationRulesId
```
![Results from On-Demand provisioning for all groups needing reconciliation](https://github.com/user-attachments/assets/99d36c2d-9ed0-4fb7-869a-e18671e916ab)

## Functions

### Get-GpadWritebackEnabledExtensionName

Retrieves the name of the custom extension property that indicates if writeback is enabled.

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

### Get-GpadCloudSecurityGroups

Retrieves all cloud security groups excluding M365/Unified groups. Includes writeback configuration and extension.

Returns:
`System.Collections.ArrayList` — List of security groups from Microsoft Entra.

Example:
```PowerShell
Get-GpadCloudSecurityGroups
```

### Get-GpadSyncedGroupMembers

Retrieves members of a specified group that are synchronized from on-premises AD.

Parameters:
`-GroupId <String>`: ID of the group.

Returns:
`System.Collections.ArrayList` — List of group members with object ID and type.

Example:
```PowerShell
Get-GpadSyncedGroupMembers -GroupId '3b10####-####-####-####-####9835c3e0'
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

### Start-GpadOnDemandProvisionGroup

Triggers synchronization job on-demand for Entra ID groups.

Parameters:

`-GroupId <String[]>`: One or more Entra ID group IDs.

`-ServicePrincipalId <String>`

`-SynchronizationJobId <String>`

`-SynchronizationRulesId <String>`

`-GroupWritebackOU <String> (Optional)`

Example:
```PowerShell
$prov = Get-GpadSynchronizationIdentifiers -DomainName 'Contoso.com'`

Start-GpadOnDemandProvisionGroup -GroupId '3b10####-####-####-####-####9835c3e0' `
-ServicePrincipalId $prov.ServicePrincipalId `
-SynchronizationJobId $prov.SynchronizationJobId `
-SynchronizationRulesId $prov.SynchronizationRulesId `
-GroupWritebackOU 'OU=GWB,OU=SYNC,DC=Contoso,DC=com'
```

### Set-GpadWritebackEnabledExtension

Sets the WritebackEnabled property and extension for a cloud group.

Parameters:

`-GroupId <String[]>`: One or more Entra ID group IDs.

`-Value <String>`: Set WritebackEnabled True or False

Example:
```PowerShell
Set-GpadWritebackEnabledExtension -GroupId '3b10####-####-####-####-####9835c3e0' -Value $true
```

### Get-GpadToolsGraphInvoke (Internal)

Internal function to call Microsoft Graph API with optional filters and headers.

### Import-GpadToolsGraphModule (Internal)

Internal function to import required Microsoft Graph modules.

## Release Notes

### Version 0.0.1 (2024-10-14)
- Beta Release:

    Get-GpadGroups

    Get-GpadSyncedGroupMembers

    Get-GpadSynchronizationIdentifiers

    Get-GpadWritebackEnabledExtensionName

    Get-GpadWritebackEnabledGroups

    Set-GpadWritebackEnabledExtension

    Start-GpadOnDemandProvisionGroup

    Confirm-GpadReconciliationNeeded

    Update-GpadWritebackEnabledExtension

### Version 0.0.2 (2025-05-17)
- Removed depency on Microsoft.Graph.Beta module
- Removed depency on Writeback extension attribute when synchronizing all groups
- New and updated funtions:
    
    Get-GpadToolsGraphInvoke (internal)
    
    Get-GpadCloudSecurityGroups
    
    Update-GpadWritebackEnabledExtension

- Removed functions replaced by Get-GpadCloudSecurityGroups:
    
    Get-GpadGroups

    Get-GpadWritebackEnabledGroups


