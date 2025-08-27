---
title: Migrate from Azure Disk Encryption to encryption at host
description: Learn how to migrate your virtual machines from Azure Disk Encryption (ADE) to encryption at host
author: msmbaldwin
ms.date: 08/21/2025
ms.topic: how-to
ms.author: mbaldwin
ms.service: azure-virtual-machines
ms.subservice: security
ms.custom: references_regions
# Customer intent: As a security administrator, I need to migrate my VMs from Azure Disk Encryption to encryption at host
---

# Migrate from Azure Disk Encryption to encryption at host

This article provides step-by-step guidance for migrating your virtual machines from Azure Disk Encryption (ADE) to Encryption at Host. The migration process requires creating new disks and VMs, as in-place conversion is not supported.

## Migration overview

Azure Disk Encryption (ADE) encrypts data within the VM using BitLocker (Windows) or dm-crypt (Linux), while Encryption at Host encrypts data at the VM host level without consuming VM CPU resources. Encryption at Host enhances Azure's default server-side encryption (SSE) by providing end-to-end encryption for all VM data, including temp disks, caches, and data flows between compute and storage.

For more information, see [Overview of managed disk encryption options](/azure/virtual-machines/disk-encryption-overview) or [Enable end-to-end encryption using encryption at host](/azure/virtual-machines/disks-enable-host-based-encryption-portal).

### Migration limitations and considerations

Before starting the migration process, be aware of these important limitations and considerations that will affect your migration strategy:

- **No in-place migration**: You cannot directly convert ADE-encrypted disks to Encryption at Host. Migration requires creating new disks and VMs.

- **Linux OS disk limitation**: Disabling ADE on Linux OS disks is not supported. For Linux VMs with ADE-encrypted OS disks, you must create a new VM with a new OS disk.

- **Windows ADE encryption patterns**: On Windows VMs, Azure Disk Encryption can only encrypt the OS disk alone OR all disks (OS + data disks). It's not possible to encrypt only data disks on Windows VMs.

- **UDE flag persistence**: Disks encrypted with Azure Disk Encryption have a User Data Encryption (UDE) flag that persists even after decryption. Both snapshots and disk copies using the Copy option retain this UDE flag. The migration requires creating completely new managed disks using the Upload method and copying the VHD blob data, which creates a new disk object without any metadata from the source disk.

- **Downtime required**: The migration process requires VM downtime for disk operations and VM recreation.

- **Domain-joined VMs**: If your VMs are part of an Active Directory domain, additional steps are required:
  - The original VM must be removed from the domain before deletion
  - After creating the new VM, it must be rejoined to the domain
  - For Linux VMs, domain joining can be accomplished using Azure AD extensions

For more information, see [What is Microsoft Entra Domain Services?](/entra/identity/domain-services/overview)

### Prerequisites

Before starting the migration:

1. **Backup your data**: Create backups of all critical data before beginning the migration process.

1. **Test the process**: If possible, test the migration process on a non-production VM first.

1. **Prepare encryption resources**: Ensure your VM size supports encryption at host. Most current VM sizes support this feature.

For more information about VM size requirements, see [Enable end-to-end encryption using encryption at host](/azure/virtual-machines/disks-enable-host-based-encryption-portal).

1. **Document configuration**: Record your current VM configuration, including network settings, extensions, and attached resources.

## Migration scenarios

The migration approach depends on your VM's operating system and which disks are encrypted:

| Scenario | Supported | Migration Method |
|----------|-----------|------------------|
| Windows VM - OS disk only encrypted | ✅ Yes | Disable ADE, create new disks without encryption settings, create new VM with Encryption at Host |
| Windows VM - OS and data disks encrypted | ✅ Yes | Disable ADE, create new disks without encryption settings, create new VM with Encryption at Host |
| Linux VM - Data disks only encrypted | ✅ Yes | Disable ADE, create new disks without encryption settings, attach to new/existing VM with Encryption at Host |
| Linux VM - OS disk only encrypted | ❌ Limited | Create new VM with Encryption at Host, migrate data using new disks without encryption settings |
| Linux VM - OS disk encrypted | ❌ Limited | Create new VM with Encryption at Host, migrate data using new disks without encryption settings |
| Linux VM - OS and data disks encrypted | ❌ Limited | Create new VM with Encryption at Host, migrate data using new disks without encryption settings |

> [!NOTE]
> **Windows ADE VolumeType limitations**: Windows VMs cannot have only data disks encrypted with ADE - you cannot encrypt data disks without first encrypting the OS disk. The VolumeType parameter for Windows can be omitted (defaults to "All") or set to either "All" or "OS", but cannot be set to "Data" alone. Therefore, the "Windows VM - Data disks only encrypted" scenario is not possible with Azure Disk Encryption.
>
> **Linux ADE VolumeType requirements**: The VolumeType parameter is required when encrypting Linux virtual machines and must be set to a value ("Os", "Data", or "All") supported by the Linux distribution.

## Windows VM migration procedures

This procedure works for Windows VMs where you can disable Azure Disk Encryption. Follow these steps:

### Disable Azure Disk Encryption

First, disable ADE and remove the encryption extension:

# [Azure PowerShell - Disable ADE](#tab/azure-powershell-disable)

```azurepowershell
# Disable encryption on the VM
Disable-AzVMDiskEncryption -ResourceGroupName "MyResourceGroup" -VMName "MyVM" -VolumeType "All"

# Check encryption status
Get-AzVmDiskEncryptionStatus -ResourceGroupName "MyResourceGroup" -VMName "MyVM"

# Remove the encryption extension after disks are decrypted
Remove-AzVMExtension -ResourceGroupName "MyResourceGroup" -VMName "MyVM" -Name "AzureDiskEncryption"
```

# [Azure CLI - Disable ADE](#tab/azure-cli-disable)

```azurecli
# Disable encryption on the VM
az vm encryption disable --resource-group "MyResourceGroup" --name "MyVM" --volume-type "all"

# Check encryption status
az vm encryption show --resource-group "MyResourceGroup" --name "MyVM"

# Remove the encryption extension after disks are decrypted
az vm extension delete --resource-group "MyResourceGroup" --vm-name "MyVM" --name "AzureDiskEncryption"
```

---

### Create new managed disks using Upload method

To properly remove the UDE (User Data Encryption) flag, you must create new managed disks using the Upload method and copy the VHD blob data from the original disks. This process creates a completely new disk object without any metadata from the source disk, ensuring the UDE flag is not present.

> [!IMPORTANT]
> **Critical requirement**: You must use the Upload method with AzCopy to copy the VHD blob data. This creates a new managed disk object without any metadata from the source disk, which is the only way to remove the UDE flag from ADE-encrypted disks.

For more information, see [Upload a VHD to Azure or copy a managed disk to another region - Azure PowerShell](/azure/virtual-machines/windows/disks-upload-vhd-to-managed-disk-powershell) or [Upload a VHD to Azure or copy a managed disk to another region - Azure CLI](/azure/virtual-machines/linux/disks-upload-vhd-to-managed-disk-cli).

# [Azure PowerShell - Upload Method](#tab/azure-powershell-copy)

```azurepowershell
# Get the VM and its disks
$vm = Get-AzVM -ResourceGroupName "MyResourceGroup" -Name "MyVM"

# Create new OS disk using Upload method
$osDisk = Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName $vm.StorageProfile.OsDisk.Name
# Add 512 bytes offset as required by Azure
$osDiskSizeBytes = $osDisk.DiskSizeBytes + 512
$osDiskConfig = New-AzDiskConfig -SkuName 'Standard_LRS' -OsType 'Windows' -UploadSizeInBytes $osDiskSizeBytes -Location $vm.Location -CreateOption 'Upload'
$newOsDisk = New-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName "MyVM-OS-New" -Disk $osDiskConfig

# Generate SAS for source and target OS disks
$sourceDiskSas = Grant-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName $osDisk.Name -DurationInSecond 86400 -Access 'Read'
$targetDiskSas = Grant-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName "MyVM-OS-New" -DurationInSecond 86400 -Access 'Write'

# Copy VHD blob data using AzCopy
azcopy copy $sourceDiskSas.AccessSAS $targetDiskSas.AccessSAS --blob-type PageBlob

# Revoke access
Revoke-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName $osDisk.Name
Revoke-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName "MyVM-OS-New"

# Create new data disks using Upload method
foreach ($dataDisk in $vm.StorageProfile.DataDisks) {
    $sourceDisk = Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName $dataDisk.Name
    $dataDiskSizeBytes = $sourceDisk.DiskSizeBytes + 512
    $diskConfig = New-AzDiskConfig -SkuName 'Standard_LRS' -UploadSizeInBytes $dataDiskSizeBytes -Location $vm.Location -CreateOption 'Upload'
    $newDisk = New-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName "$($dataDisk.Name)-New" -Disk $diskConfig
    
    # Generate SAS and copy data
    $sourceDataSas = Grant-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName $dataDisk.Name -DurationInSecond 86400 -Access 'Read'
    $targetDataSas = Grant-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName "$($dataDisk.Name)-New" -DurationInSecond 86400 -Access 'Write'
    
    azcopy copy $sourceDataSas.AccessSAS $targetDataSas.AccessSAS --blob-type PageBlob
    
    Revoke-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName $dataDisk.Name
    Revoke-AzDiskAccess -ResourceGroupName "MyResourceGroup" -DiskName "$($dataDisk.Name)-New"
}
```

# [Azure CLI - Upload Method](#tab/azure-cli-copy)

```azurecli
# Get VM information
vm_info=$(az vm show --resource-group "MyResourceGroup" --name "MyVM")
location=$(echo $vm_info | jq -r '.location')

# Get OS disk information
os_disk_name=$(echo $vm_info | jq -r '.storageProfile.osDisk.name')
os_disk_size_bytes=$(az disk show --resource-group "MyResourceGroup" --name $os_disk_name --query "diskSizeBytes" -o tsv)
# Add 512 bytes offset as required by Azure
os_upload_size=$((os_disk_size_bytes + 512))

# Create new OS disk using Upload method
az disk create \
    --resource-group "MyResourceGroup" \
    --name "MyVM-OS-New" \
    --location $location \
    --upload-size-bytes $os_upload_size \
    --upload-type Upload \
    --os-type Windows \
    --sku Standard_LRS

# Generate SAS for source and target disks
source_sas=$(az disk grant-access --resource-group "MyResourceGroup" --name $os_disk_name --duration-in-seconds 86400 --access-level Read --query "accessSas" -o tsv)
target_sas=$(az disk grant-access --resource-group "MyResourceGroup" --name "MyVM-OS-New" --duration-in-seconds 86400 --access-level Write --query "accessSas" -o tsv)

# Copy VHD blob data using AzCopy
azcopy copy "$source_sas" "$target_sas" --blob-type PageBlob

# Revoke access
az disk revoke-access --resource-group "MyResourceGroup" --name $os_disk_name
az disk revoke-access --resource-group "MyResourceGroup" --name "MyVM-OS-New"

# Create new data disks using Upload method
data_disks=$(echo $vm_info | jq -r '.storageProfile.dataDisks[].name')
for disk in $data_disks; do
    disk_size_bytes=$(az disk show --resource-group "MyResourceGroup" --name $disk --query "diskSizeBytes" -o tsv)
    upload_size=$((disk_size_bytes + 512))
    
    az disk create \
        --resource-group "MyResourceGroup" \
        --name "${disk}-New" \
        --location $location \
        --upload-size-bytes $upload_size \
        --upload-type Upload \
        --sku Standard_LRS
    
    # Generate SAS and copy data
    data_source_sas=$(az disk grant-access --resource-group "MyResourceGroup" --name $disk --duration-in-seconds 86400 --access-level Read --query "accessSas" -o tsv)
    data_target_sas=$(az disk grant-access --resource-group "MyResourceGroup" --name "${disk}-New" --duration-in-seconds 86400 --access-level Write --query "accessSas" -o tsv)
    
    azcopy copy "$data_source_sas" "$data_target_sas" --blob-type PageBlob
    
    az disk revoke-access --resource-group "MyResourceGroup" --name $disk
    az disk revoke-access --resource-group "MyResourceGroup" --name "${disk}-New"
done
```

> [!IMPORTANT]
> This method creates new managed disks using the Upload option and copies the VHD blob data using AzCopy. This creates a completely new disk object without any metadata from the source disk, ensuring the UDE flag is not present while maintaining Azure's default server-side encryption (SSE). The 512-byte offset is required by Azure when copying managed disks. For OS disks, the UDE flag is also stored at the VM metadata level, so the VM must be recreated regardless.

---

### Verify new managed disks

Verify that the new managed disks have been created successfully and are ready for use with Encryption at Host.

# [Azure PowerShell - Verify Disks](#tab/azure-powershell-verify)

```azurepowershell
# Verify OS disk was created successfully
Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName "MyVM-OS-New" | Select-Object Name, DiskState, DiskSizeGB

# Verify data disks were created successfully
foreach ($dataDisk in $vm.StorageProfile.DataDisks) {
    $newDiskName = "$($dataDisk.Name)-New"
    Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName $newDiskName | Select-Object Name, DiskState, DiskSizeGB
}
```

# [Azure CLI - Verify Disks](#tab/azure-cli-verify)

```azurecli
# Verify OS disk was created successfully
az disk show --resource-group "MyResourceGroup" --name "MyVM-OS-New" --query "{name:name, state:diskState, sizeGB:diskSizeGb}"

# Verify data disks were created successfully
for disk in $data_disks; do
    az disk show --resource-group "MyResourceGroup" --name "${disk}-New" --query "{name:name, state:diskState, sizeGB:diskSizeGb}"
done
```

---

### Create new VM with encrypted disks

# [Azure PowerShell - New VM](#tab/azure-powershell-vm)

```azurepowershell
# Create new VM configuration
$vmConfig = New-AzVMConfig -VMName "MyVM-New" -VMSize $vm.HardwareProfile.VmSize

# Set OS disk
$vmConfig = Set-AzVMOSDisk -VM $vmConfig -ManagedDiskId $newOsDisk.Id -CreateOption Attach -Windows

# Add data disks
foreach ($dataDisk in $vm.StorageProfile.DataDisks) {
    $newDiskName = "$($dataDisk.Name)-New"
    $newDisk = Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName $newDiskName
    $vmConfig = Add-AzVMDataDisk -VM $vmConfig -ManagedDiskId $newDisk.Id -Lun $dataDisk.Lun -CreateOption Attach
}

# Enable encryption at host (required for migration from ADE)
$vmConfig = Set-AzVMSecurityProfile -VM $vmConfig -EncryptionAtHost $true

# Set network configuration
$nic = Get-AzNetworkInterface -ResourceId $vm.NetworkProfile.NetworkInterfaces[0].Id
$vmConfig = Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id

# Create the new VM
New-AzVM -ResourceGroupName "MyResourceGroup" -Location $vm.Location -VM $vmConfig
```

# [Azure CLI - New VM](#tab/azure-cli-vm)

```azurecli
# Get original VM size and NIC
vm_size=$(echo $vm_info | jq -r '.hardwareProfile.vmSize')
nic_id=$(echo $vm_info | jq -r '.networkProfile.networkInterfaces[0].id')

# Create new VM with Encryption at Host enabled
os_disk_id=$(az disk show --resource-group "MyResourceGroup" --name "MyVM-OS-New" --query "id" -o tsv)

az vm create \
    --resource-group "MyResourceGroup" \
    --name "MyVM-New" \
    --location $location \
    --size $vm_size \
    --nics $nic_id \
    --attach-os-disk $os_disk_id \
    --os-type Windows \
    --encryption-at-host true

# Attach new data disks
for disk in $data_disks; do
    disk_id=$(az disk show --resource-group "MyResourceGroup" --name "${disk}-New" --query "id" -o tsv)
    # Get original LUN number
    lun=$(echo $vm_info | jq -r ".storageProfile.dataDisks[] | select(.name==\"$disk\") | .lun")
    az vm disk attach \
        --resource-group "MyResourceGroup" \
        --vm-name "MyVM-New" \
        --disk $disk_id \
        --lun $lun
done
```

---

## Linux VM migration procedures

For Linux VMs, the approach depends on whether the OS disk is encrypted.

For more information about Azure Disk Encryption on Linux, see [Azure Disk Encryption for Linux VMs](/azure/virtual-machines/linux/disk-encryption-overview).

### Data disks only encrypted

If only data disks are encrypted, follow a similar procedure as Windows VMs:

1. [Disable ADE on data disks](linux/disk-encryption-linux.md#disable-encryption-and-remove-the-encryption-extension)
2. Create new managed disks without encryption settings (Platform Managed Keys)
3. Copy data from the original disks to the new disks
4. Create a new VM with Encryption at Host enabled and attach the new disks

### OS disk encrypted (Linux)

Since disabling ADE on Linux OS disks is not supported, you must create a new VM:

1. Create a new VM with a fresh OS disk and Encryption at Host enabled
2. [Disable ADE on the data disks](linux/disk-encryption-linux.md#disable-encryption-and-remove-the-encryption-extension) of the original VM (if applicable)
3. Create new data disks without encryption settings (Platform Managed Keys)
4. Copy data from the original disks to the new disks
5. Attach the new data disks to the new VM with Encryption at Host
6. Migrate your data and configuration from the old VM to the new VM

## VHD blob copying process

The migration process uses AzCopy to copy the VHD blob data from the source ADE-encrypted disk to a new managed disk created with the Upload method. This approach creates a completely new disk object without any metadata from the source disk.

### Key advantages of the Upload method

1. **Complete metadata removal**: Creates a new disk object without any metadata from the source disk
2. **UDE flag elimination**: The new disk will not have the UDE flag that prevents Encryption at Host, while still maintaining Azure's default server-side encryption (SSE)
3. **Direct VHD blob transfer**: Uses AzCopy for efficient blob-level copying
4. **No intermediate steps**: Eliminates the need for temporary VMs or manual file copying

### Process overview

1. **Disable ADE**: Ensure the source disks are fully decrypted
2. **Create Upload disks**: Create new managed disks using the Upload method with the exact size (plus 512-byte offset)
3. **Generate SAS tokens**: Create read access for source disk and write access for target disk
4. **Copy VHD blobs**: Use AzCopy to transfer the VHD blob data between disks
5. **Revoke access**: Remove SAS access from both source and target disks
6. **Create new VM**: Build a new VM with Encryption at Host using the new disks

### Prerequisites for copying

- **AzCopy v10**: Download and install the latest version of AzCopy
- **Proper permissions**: Ensure you have the necessary RBAC permissions for disk operations
- **Sufficient bandwidth**: Large disks may take considerable time to copy
- **Source disk decryption**: ADE must be fully disabled and disks decrypted before starting

For more information about AzCopy and VHD uploads, see [Get started with AzCopy](/azure/storage/common/storage-use-azcopy-v10).

### Important considerations

- The 512-byte offset is mandatory when copying managed disks in Azure
- SAS tokens are limited to 60 days maximum (recommendation: use shorter durations)
- Copy operations should be performed in the same region for optimal performance
- Cross-region copying is supported but will take longer and may incur additional costs

## Domain-joined VM considerations

If your VMs are members of an Active Directory domain, additional steps are required during the migration process:

### Pre-migration domain steps

1. **Document domain membership**: Record the current domain, organizational unit (OU), and any special group memberships
2. **Note computer account**: The computer account in Active Directory will need to be managed
3. **Backup domain-specific configurations**: Save any domain-specific settings, group policies, or certificates

### Domain removal process

1. **Remove from domain**: Before deleting the original VM, remove it from the domain using one of these methods:
   - Use `Remove-Computer` PowerShell cmdlet on Windows
   - Use the System Properties dialog to change to workgroup
   - Manually delete the computer account from Active Directory Users and Computers

2. **Clean up Active Directory**: Remove any orphaned computer accounts or DNS entries

### Post-migration domain rejoining

1. **Join new VM to domain**: After creating the new VM with Encryption at Host:
   - For Windows: Use `Add-Computer` PowerShell cmdlet or System Properties
   - For Linux: Use Azure AD domain join extension or manual configuration

2. **Restore domain settings**: Reapply any domain-specific configurations, group policies, or certificates

3. **Verify domain functionality**: Test domain authentication, group policy application, and network resource access

### Linux domain joining

For Linux VMs, you can use the Azure AD Domain Services VM extension:

```azurecli
az vm extension set \
    --resource-group "MyResourceGroup" \
    --vm-name "MyLinuxVM-New" \
    --name "AADSSHLoginForLinux" \
    --publisher "Microsoft.Azure.ActiveDirectory"
```

For more information, see [What is Microsoft Entra Domain Services?](/entra/identity/domain-services/overview)

### Important domain considerations

- The new VM will have a different computer SID, which may affect some applications
- Kerberos tickets and cached credentials will need to be refreshed
- Some domain-integrated applications may require reconfiguration
- Plan for potential temporary loss of domain services during migration

## Post-migration verification

After completing the migration, verify that Encryption at Host is working correctly:

1. **Check Encryption at Host status**: Verify that Encryption at Host is enabled:

   ```azurecli
   az vm show --resource-group "MyResourceGroup" --name "MyVM-New" --query "securityProfile.encryptionAtHost"
   ```

2. **Test VM functionality**: Ensure your applications and services are working correctly.

3. **Verify disk encryption**: Confirm that disks are properly encrypted:

   ```azurepowershell
   Get-AzDisk -ResourceGroupName "MyResourceGroup" -DiskName "MyVM-OS-New" | Select-Object Name, DiskState
   ```

4. **Monitor performance**: Compare performance before and after migration to confirm the expected improvements.

For more information about encryption verification, see [Enable end-to-end encryption using encryption at host](/azure/virtual-machines/disks-enable-host-based-encryption-portal).

## Cleanup

After successful migration and verification:

1. **Delete old VM**: Remove the original ADE-encrypted VM
2. **Delete old disks**: Remove the original encrypted disks
3. **Clean up resources**: Remove any temporary resources created during migration
4. **Update documentation**: Update your infrastructure documentation to reflect the migration to Encryption at Host

## Common issues and solutions

### VM size doesn't support Encryption at Host

**Solution**: Check the [list of supported VM sizes](disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data) and resize your VM if necessary

### VM fails to start after migration

**Solution**: Check that all disks are properly attached and that the OS disk is set as the boot disk

### Encryption at Host not enabled

**Solution**: Verify that the VM was created with the `--encryption-at-host true` parameter and that your subscription supports this feature

### Performance issues persist

**Solution**: Verify that encryption at host is properly enabled and that the VM size supports the expected performance.

## Next steps

- [Overview of managed disk encryption options](/azure/virtual-machines/disk-encryption-overview)
- [Enable end-to-end encryption using encryption at host - Azure portal](/azure/virtual-machines/disks-enable-host-based-encryption-portal)
- [Enable end-to-end encryption using encryption at host - Azure PowerShell](/azure/virtual-machines/windows/disks-enable-host-based-encryption-powershell)
- [Enable end-to-end encryption using encryption at host - Azure CLI](/azure/virtual-machines/linux/disks-enable-host-based-encryption-cli)
- [Server-side encryption of Azure Disk Storage](/azure/virtual-machines/disk-encryption)
- [Azure Disk Encryption for Windows VMs](/azure/virtual-machines/windows/disk-encryption-overview)
- [Azure Disk Encryption for Linux VMs](/azure/virtual-machines/linux/disk-encryption-overview)
- [Azure Disk Encryption FAQ](/azure/virtual-machines/linux/disk-encryption-faq)
- [Upload a VHD to Azure or copy a managed disk to another region](/azure/virtual-machines/windows/disks-upload-vhd-to-managed-disk-powershell)
