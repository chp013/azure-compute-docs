---
title: Manage, Update and Delete VM Application on Azure
description: Learn how to manage, update and delete VM applications on Azure Virtual Machine (VM) and Virtual Machine Scale Sets (VMSS) using Azure Compute Gallery.
author: tanmaygore
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 09/08/2023
ms.author: tagore
ms.reviewer: jushiman
ms.custom: devx-track-azurepowershell, devx-track-azurecli
---

# Manage Azure VM Applications

This article talks about how to monitor, update and delete deployed VM application resource on Azure Virtual Machine (VM) or Virtual Machine Scale Sets (VMSS).

## Monitor the deployed VM Applications
#### [Portal](#tab/portal5)
To show the VM application status, go to the Extensions + applications tab/settings and check the status of the VMAppExtension:

:::image type="content" source="media/vmapps/select-app-status.png" alt-text="Screenshot showing VM application status.":::

To show the VM application status for scale set, go to the Azure portal Virtual Machine Scale Sets page, then the Instances section, select one of the scales sets listed, then go to **VMAppExtension**:

:::image type="content" source="media/vmapps/select-apps-status-vmss-portal.png" alt-text="Screenshot showing virtual machine scale sets application status.":::

#### [CLI](#tab/cli5)

To verify application deployment status on a VM, use ['az vm get-instance-view'](/cli/azure/vm/#az-vm-get-instance-view):

```azurecli-interactive
az vm get-instance-view -g myResourceGroup -n myVM --query "instanceView.extensions[?name == 'VMAppExtension']"
```
To verify application deployment status on Virtual Machine Scale Set, use ['az vmss get-instance-view'](/cli/azure/vmss/#az-vmss-get-instance-view):

```azurecli-interactive
az vmss get-instance-view --ids (az vmss list-instances -g myResourceGroup -n myVmss --query "[*].id" -o tsv) --query "[*].extensions[?name == 'VMAppExtension']"
```
> [!NOTE]
> The previous Virtual Machine Scale Sets deployment status command doesn't list the instance ID with the result. To show the instance ID with the status of the extension in each instance, some more scripting is required. Refer to the following CLI example that contains PowerShell syntax:

```azurepowershell-interactive
$ids = az vmss list-instances -g myResourceGroup -n myVmss --query "[*].{id: id, instanceId: instanceId}" | ConvertFrom-Json
$ids | Foreach-Object {
    $iid = $_.instanceId
    Write-Output "instanceId: $iid" 
    az vmss get-instance-view --ids $_.id --query "extensions[?name == 'VMAppExtension']" 
}
```

#### [PowerShell](#tab/powershell5)

Verify the application succeeded:

```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmName = "myVM"
$result = Get-AzVM -ResourceGroupName $rgName -VMName $vmName -Status
$result.Extensions | Where-Object {$_.Name -eq "VMAppExtension"} | ConvertTo-Json
```
To verify on your Virtual Machine Scale Set:
```azurepowershell-interactive
$rgName = "myResourceGroup"
$vmssName = "myVMss"
$result = Get-AzVmssVM -ResourceGroupName $rgName -VMScaleSetName $vmssName -InstanceView
$resultSummary  = New-Object System.Collections.ArrayList
$result | ForEach-Object {
    $res = @{ instanceId = $_.InstanceId; vmappStatus = $_.InstanceView.Extensions | Where-Object {$_.Name -eq "VMAppExtension"}}
    $resultSummary.Add($res) | Out-Null
}
$resultSummary | ConvertTo-Json -Depth 5
```

#### [REST](#tab/rest5)

If the VM application isn't installed on the VM, the value is empty. 

To get the result of VM instance view:

```rest
GET
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/virtualMachines/{VMName}/instanceView?api-version=2024-03-03
```

The result looks like this:

```rest
{
    ...
    "extensions"  [
    ...
        {
            "name":  "VMAppExtension",
            "type":  "Microsoft.CPlat.Core.VMApplicationManagerLinux",
            "typeHandlerVersion":  "1.0.9",
            "statuses":  [
                            {
                                "code":  "ProvisioningState/succeeded",
                                "level":  "Info",
                                "displayStatus":  "Provisioning succeeded",
                                "message":  "Enable succeeded: {\n \"CurrentState\": [\n  {\n   \"applicationName\": \"doNothingLinux\",\n   \"version\": \"1.0.0\",\n   \"result\": \"Install SUCCESS\"\n  },\n  {
        \n   \"applicationName\": \"badapplinux\",\n   \"version\": \"1.0.0\",\n   \"result\": \"Install FAILED Error executing command \u0027exit 1\u0027: command terminated with exit status=1\"\n  }\n ],\n \"ActionsPerformed\": []\n}
        "
                            }
                        ]
        }
    ...
    ]
}
```
The VM App status is in the status message of the result of the VM App extension in the instance view.

To get the status for the application on the Virtual Machine Scale Set:

```rest
GET
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/ virtualMachineScaleSets/{VMSSName}/virtualMachines/{instanceId}/instanceView?api-version=2019-03-01
```
The output is similar to the VM example earlier.

---

# Delete the VM Application
To delete the VM Application resource, you need to first delete all its versions. Deleting the application version causes deletion of the application version resource from Azure Compute Gallery and all its replicas. The application blob in Storage Account used to create the application version is unaffected. After deleting the application version, if any VM is using that version, then reimage operation on those VMs will fail. Use 'latest' keyword as the version number in the 'applicationProfile' instead of hard coding the version number to address this failure.  
However if the application is deleted, then VM fails during reimage operation since there are no versions available for Azure to install. The VM profile needs to be updated to not use the VM Application. 

#### [REST](#tab/rest6)
Delete the VM Application version:
```rest
DELETE
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{galleryApplicationName}/versions/{galleryApplicationVersionName}?api-version=2024-03-03
```

Delete the VM Application after all its versions are deleted:
```rest
DELETE
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/galleries/{galleryName}/applications/{galleryApplicationName}?api-version=2024-03-03
```

#### [CLI](#tab/cli6)
Delete the VM Application version:
```azurecli-interactive
az sig gallery-application version delete --resource-group $rg-name --gallery-name $gallery-name --application-name $app-name --version-name $version-name
```

Delete the VM Application after all its versions are deleted:
```azurecli-interactive
az sig gallery-application delete --resource-group $rg-name --gallery-name $gallery-name --application-name $app-name
```

#### [PowerShell](#tab/powershell6)
Delete the VM Application version: 
```azurepowershell-interactive
Remove-AzGalleryApplicationVersion -ResourceGroupName $rgNmae -GalleryName $galleryName -GalleryApplicationName $galleryApplicationName -Name $name
```

Delete the VM Application after all its versions are deleted:
```azurepowershell-interactive
Remove-AzGalleryApplication -ResourceGroupName $rgNmae -GalleryName $galleryName -Name $name
```
---

## Next steps
Learn more about [Azure VM Applications](vm-applications.md).
Learn to [create Azure VM applications](vm-applications-how-to.md).