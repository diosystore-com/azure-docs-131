---
title: Azure Virtual Desktop personal desktop assignment type - Azure
description: How to configure automatic or direct assignment for an Azure Virtual Desktop personal desktop host pool.
author: Heidilohr
ms.topic: how-to
ms.date: 03/03/2023
ms.author: helohr 
ms.custom: devx-track-azurepowershell
manager: femila
---
# Configure personal desktop host pool assignment

>[!IMPORTANT]
>This content applies to Azure Virtual Desktop with Azure Resource Manager Azure Virtual Desktop objects. If you're using Azure Virtual Desktop (classic) without Azure Resource Manager objects, see [this article](./virtual-desktop-fall-2019/configure-host-pool-personal-desktop-assignment-type-2019.md).

You can configure the assignment type of your personal desktop host pool to adjust your Azure Virtual Desktop environment to better suit your needs. In this topic, we'll show you how to configure automatic or direct assignment for your users.

>[!NOTE]
> The instructions in this article only apply to personal desktop host pools, not pooled host pools, since users in pooled host pools aren't assigned to specific session hosts.

## Prerequisites

This article assumes you've already downloaded and installed the Azure Virtual Desktop PowerShell module. If you haven't, follow the instructions in [Set up the PowerShell module](powershell-module.md).

### Define variables

The PowerShell commands listed in this article require defining the following variables with the placeholder values replaced with the values relevant to your account and deployment:

```powershell
#Define variables
$subscriptionId = <00000000-0000-0000-0000-000000000000>
$resourceGroupName = <MyResourceGroupName>
$hostPoolName = <MyHostPoolName>
$sessionHostName = <SessionHostName>
```

## Personal host pools overview

A personal host pool is a type of host pool that has personal desktops. Personal desktops have one-to-one mapping, which means a single user can only be assigned to a single personal desktop. Every time the user signs in, their user session is directed to their assigned personal desktop session host. This host pool type is ideal for customers with resource-intensive workloads because user experience and session performance will improve if there's only one session on the session host. Another benefit of this host pool type is that user activities, files, and settings persist on the virtual machine operating system (VM OS) disk after the user signs out.

Users must be assigned to a personal desktop to start their session. There are two types of assignments in a personal host pool: automatic assignment and direct assignment.

## Configure automatic assignment

Automatic assignment is the default assignment type for new personal desktop host pools created in your Azure Virtual Desktop environment. Automatically assigning users doesn't require a specific session host.

To automatically assign users, first assign them to the personal desktop host pool so that they can see the desktop in their feed. When an assigned user launches the desktop in their feed, their user session will be load-balanced to an available session host if they haven't already connected to the host pool. You can still [assign a user directly to a session host](#configure-direct-assignment) before they connect, even if the assignment type is set automatic.

To configure a host pool to automatically assign users to VMs, run the following PowerShell cmdlet:

```powershell
Update-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -PersonalDesktopAssignmentType Automatic
```

To assign a user to the personal desktop host pool, run the following PowerShell cmdlet:

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

## Configure direct assignment

Unlike automatic assignment, when you use direct assignment, you must assign the user to both the personal desktop host pool and a specific session host before they can connect to their personal desktop. If the user is only assigned to a host pool without a session host assignment, they won't be able to access resources and will see an error message that says, "No resources available."

To configure a host pool to require direct assignment of users to session hosts, run the following PowerShell cmdlet:

```powershell
Update-AzWvdHostPool -ResourceGroupName $resourceGroupName -Name $hostPoolName -PersonalDesktopAssignmentType Direct
```

To assign a user to the personal desktop host pool, run the following PowerShell cmdlet:

```powershell
New-AzRoleAssignment -SignInName <userupn> -RoleDefinitionName "Desktop Virtualization User" -ResourceName <appgroupname> -ResourceGroupName $resourceGroupName -ResourceType 'Microsoft.DesktopVirtualization/applicationGroups'
```

To assign a user to a specific session host, run the following PowerShell cmdlet:

```powershell
Update-AzWvdSessionHost -HostPoolName $hostPoolName -Name $sessionHostName -ResourceGroupName $resourceGroupName -AssignedUser <userupn>
```

To directly assign a user to a session host in the Azure portal:

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Enter **Azure Virtual Desktop** into the search bar.
3. Under **Services**, select **Azure Virtual Desktop**.
4. At the Azure Virtual Desktop page, go the menu on the left side of the window and select **Host pools**.
5. Select the host pool you want to assign users to.
6. Next, go to the menu on the left side of the window and select **Application groups**.
7. Select the name of the app group you want to assign users to, then select **Assignments** in the menu on the left side of the window.
8. Select **+ Add**, then select the users or user groups you want to assign to this app group.
9. Select **Assign VM** in the Information bar to assign a session host to a user.
10. Select the session host you want to assign to the user, then select **Assign**. You can also select **Assignment** > **Assign user**.
11. Select the user you want to assign the session host to from the list of available users.
12. When you're done, select **Select**.

## Unassign a personal desktop using the Azure portal

To unassign a personal desktop in the Azure portal:

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Enter **Azure Virtual Desktop** into the search bar.
3. Under **Services**, select **Azure Virtual Desktop**.
4. At the Azure Virtual Desktop page, go the menu on the left side of the window and select **Host pools**.
5. Select the host pool you want to modify user assignment for.
6. Next, go to the menu on the left side of the window and select **Session hosts**.
7. Select the checkbox next to the session host you want to unassign a user from, select the ellipses at the end of the row, and then select **Unassign user**. You can also select **Assignment** > **Unassign user**.

    > [!div class="mx-imgBorder"]
    > ![A screenshot of the unassign user menu option from the ellipses menu for unassigning a personal desktop.](media/unassign.png)

    > [!div class="mx-imgBorder"]
    > ![A screenshot of the unassign user menu option from the assignment menu for unassigning a personal desktop.](media/unassign-2.png)

8. Select **Unassign** when prompted with the warning.

## Unassign a personal desktop using PowerShell

To unassign a personal desktop in PowerShell, run the following command:

```powershell
$unassignDesktopParams = @{
  Path = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName/sessionHosts/$($sessionHostName)?api-version=2022-02-10-preview&force=true"
  Payload = @{
    properties = @{
      assignedUser = ''
    }} | ConvertTo-Json
  Method = 'PATCH'
}
Invoke-AzRestMethod @unassignDesktopParams
```

## Reassign a personal desktop using the Azure portal

To reassign a personal desktop in the Azure portal:
1. Sign in to the [Azure portal](https://portal.azure.com).
2. Enter **Azure Virtual Desktop** into the search bar.
3. Under **Services**, select **Azure Virtual Desktop**.
4. At the Azure Virtual Desktop page, go the menu on the left side of the window and select **Host pools**.
5. Select the host pool you want to modify user assignment for.
6. Next, go to the menu on the left side of the window and select **Session hosts**.
7. Select the checkbox next to the session host you want to reassign to a different user, select the ellipses at the end of the row, and then select **Assign to a different user**. You can also select **Assignment** > **Assign to a different user**.

    > [!div class="mx-imgBorder"]
    > ![A screenshot of the assign to a different user menu option from the ellipses menu for reassigning a personal desktop.](media/reassign-doc.png)

    > [!div class="mx-imgBorder"]
    > ![A screenshot of the assign to a different user menu option from the assignment menu for reassigning a personal desktop.](media/reassign.png)

8. Select the user you want to assign the session host to from the list of available users.
9. When you're done, select **Select**.

## Reassign a personal desktop using PowerShell

Before you start, first define the `$reassignUserUpn` variable by running the following command:

```powershell
$reassignUserUpn = <UPN of user you are reassigning the desktop to>
```

To reassign a personal desktop, run this command:

```powershell
$reassignDesktopParams = @{
  Path = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName/sessionHosts/$($sessionHostName)?api-version=2022-02-10-preview&force=true"
  Payload = @{
    properties = @{
      assigneduser = $reassignUserUpn
    }} | ConvertTo-Json
  Method = 'PATCH'
}
Invoke-AzRestMethod @reassignDesktopParams
```

## Give session hosts in a personal host pool a friendly name

You can give personal desktops you create *friendly names* to help users distinguish them in their feeds.

To give a session host a friendly name, run the following command in PowerShell:

```powershell
$body = '{ "properties": {
"friendlyName": "friendlyName"
} }'

$parameters = @{
    Method = 'Patch'
    Path = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName/sessionHosts/$($sessionHostName)?api-version=2022-02-10-preview"
    Payload = $body
}

Invoke-AzRestMethod @parameters
```

>[!NOTE]
>You can also set the friendly name by using a [REST API](/rest/api/desktopvirtualization/session-hosts/update?tabs=HTTP).

### Get the session host friendly name

To get the session host friendly name, run this command in PowerShell:

```powershell
$getParams = @{
  Path = '/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.DesktopVirtualization/hostPools/$hostPoolName/sessionHosts/$($sessionHostName)?api-version=2022-02-10-preview'
  Method = 'GET'
}
Invoke-AzRestMethod @getParams
```

## Next steps

Now that you've configured the personal desktop assignment type and given your session host a friendly name, you can sign in to an Azure Virtual Desktop client to test it as part of a user session. These articles will show you how to connect to a session using the client of your choice:

- [Connect with the Windows Desktop client](./users/connect-windows.md)
- [Connect with the web client](./users/connect-web.md)
- [Connect with the Android client](./users/connect-android-chrome-os.md)
- [Connect with the iOS client](./users/connect-ios-ipados.md)
- [Connect with the macOS client](./users/connect-macos.md)