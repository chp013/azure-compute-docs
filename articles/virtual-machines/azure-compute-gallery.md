---
title: Overview of Azure Compute Gallery
description: Learn about Azure Compute Gallery and how to share Azure resources.
author: ju-shim
ms.author: jushiman
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: overview
ms.date: 09/20/2023
ms.reviewer: cynthn, mattmcinnes
# Customer intent: As a cloud architect, I want to use Azure Compute Gallery to manage and share image resources efficiently across multiple regions, so that I can enhance deployment scalability, maintain high availability, and optimize storage for my organization's applications and virtual machines.
---

# Store and share resources in Azure Compute Gallery

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Flexible scale sets :heavy_check_mark: Uniform scale sets

Azure Compute Gallery helps you build structure and organization around your Azure resources, like images and [applications](vm-applications.md). Compute Gallery provides:

- Global replication.<sup>1</sup>
- Versioning and grouping of resources for easier management.
- Highly available resources with zone-redundant storage (ZRS) accounts, in regions that support availability zones. ZRS offers resilience against zonal failures.
- Premium storage support with locally redundant storage (LRS).
- Sharing to the community, across subscriptions, and between Active Directory tenants.
- Scaling your deployments with resource replicas in each region.

With a gallery, you can share your resources to everyone, or limit sharing to particular users, service principals, or Active Directory groups within your organization. You can replicate resources to multiple regions, for quicker scaling of your deployments.

<sup>1</sup> The Azure Compute Gallery service isn't a global resource. For disaster recovery scenarios, the best practice is to have at least two galleries in separate regions.

## Images

For more information about storing images in Azure Compute Gallery, see [Store and share images in Azure Compute Gallery](shared-image-galleries.md).

## VM apps

Although you can create an image of a virtual machine (VM) with apps preinstalled, you would need to update your image each time an application changes. Separating your application installation from your VM images means there's no need to publish a new image for every line of code change.

For more information about storing applications in Azure Compute Gallery, see [VM applications](vm-applications.md).

## Regional support

All public regions can be target regions, but certain regions require that customers go through a request process to gain access. To request the addition of a subscription to the allowlist for a region such as Australia Central or Australia Central 2, submit an [access request](/troubleshoot/azure/general/region-access-request-process).

## Limits

The following limits apply when you deploy resources by using Azure Compute Gallery:

- There's a limit of 100 galleries, per subscription, per region.
- There's a limit of 1,000 image definitions, per subscription, per region.
- There's a limit of 10,000 image versions, per subscription, per region.
- There's a limit of 100 replicas per image version. However, 50 replicas should be sufficient for most use cases.
- The image size should be less than 2 TB, but you can use shallow replication to support larger image sizes (up to 32 TB).
- Resource movement isn't supported for Azure Compute Gallery resources.

For more information and for examples of how to check your current usage, see [Check resource usage against limits](/azure/networking/check-usage-against-limits).

## Scaling

Azure Compute Gallery allows you to specify the number of replicas that you want to keep. In multiple-VM deployment scenarios, you can spread the VM deployments to other replicas. This action reduces the chance that instance creation processing is throttled due to overloading of a single replica.

With Compute Gallery, you can deploy up to a 1,000 VM instances in a scale set. You can set a different replica count in each target region, based on the scale needs for the region. Because each replica is a copy of your resource, this approach helps scale your deployments linearly with each extra replica. Although we understand that no two resources or regions are the same, here are general guidelines on how to use replicas in a region:

- For every 50 VMs that you create concurrently, we recommend that you keep one replica. For example, if you're creating 500 VMs concurrently by using the same image in a region, we suggest you keep at least 10 replicas of your image.
- For each scale set that you create concurrently, we recommend you keep one replica.

We always recommend that you overprovision the number of replicas due to factors like resource size, content, and OS type.

![Diagram that shows how you can scale images](./media/shared-image-galleries/scaling.png)

## High availability

[Azure ZRS](https://azure.microsoft.com/blog/azure-zone-redundant-storage-in-public-preview/) provides resilience against the failure of an availability zone in the region. With the general availability of Azure Compute Gallery, you can choose to store your images in ZRS accounts in regions that have availability zones.

You can also choose the account type for each of the target regions. The default storage account type is Standard LRS, but you can choose Standard ZRS for regions that have availability zones. For more information on the regional availability of ZRS, see [Azure Storage redundancy](/azure/storage/common/storage-redundancy).

![Diagram that shows zone-redundant storage.](./media/shared-image-galleries/zrs.png)

## Replication

Azure Compute Gallery also allows you to replicate your resources to other Azure regions automatically. You can replicate each image version to different regions, depending on what makes sense for your organization. One example is to always replicate the latest image in multiple regions while all older image versions are available in only one region. This approach can help save on storage costs.

After creation time, you can update the regions that a resource is replicated to. The time that replicatation to other regions takes depends on the amount of data that you're copying and the number of regions that the version is replicated to. This process can take a few hours in some cases.

While the replication is happening, you can view its status per region. After the image replication is complete in a region, you can deploy a VM or scale set by using that resource in the region.

![Diagram that shows the replication of an image across regions.](./media/shared-image-galleries/replication.png)

<a name=community></a>

## Trusted Launch validation for Azure Compute Gallery images (preview)

Trusted Launch validation for Azure Compute Gallery images is currently in preview. This preview is intended for testing, evaluation, and feedback purposes only. We don't recommend it for production workloads. When you register to preview, you agree to the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). Some aspects of this feature might change with general availability (GA).

### What is changing?

Starting with API 2025-03-03, all new Compute Gallery image definitions will default to:

- Hyper-V Generation: `V2`
- Security Type: `TrustedLaunchSupported`

### How Trusted Launch validation works

When you specify `TrustedLaunchSupported` or `TrustedLaunchandConfidentialVMSupported` in the Compute Gallery image definition, the platform automatically validates that the image is Trusted Launch capable. The platform adds the validation result to the image version property. This action ensures that the VM and virtual machine scale set deployments that use these images can default to Trusted Launch if the validation is successful.

### Enable the preview

To try out Trusted Launch validation for Compute Gallery images, complete the following steps:

1. Register for [Trusted Launch as Default Feature](/azure/virtual-machines/trusted-launch.md#preview-trusted-launch-as-default)
2. Register for [Trusted Launch Validation Preview](https://aka.ms/ACGTLValidationPreview)

Once the two features are enabled, all new VM and scale set deployments using Compute Gallery image versions and validated successfully for Trusted Launch will default to the Trusted Launch security type.

### VM deployments from Azure Compute Gallery images

The following compares the current and new behaviors of VM deployments from Compute Gallery images, depending on Trusted Launch validation.

#### Current behavior without Trusted Launch validation

To create a Trusted launch supported Gen2 Compute Gallery OS image definition, you need to add the following `features` element in your deployment:

```json
"features": [
    {
      "name": "SecurityType",
      "value": "TrustedLaunchSupported"
    }
],
"hyperVGeneration": "V2"
```

#### New behavior with Trusted Launch validation

`TrustedLaunchSupported` is enabled by default on new Compute Gallery image definitions if any of the following criteria is met:

- Using API version 2025-03-03 or above for `Microsoft.Compute/galleries` resource
- The absence of `securityType` feature from deployment
- A `null` value for `securityType` feature in deployment

Additionally, the Azure platform will trigger validation for the OS image to ensure it supports Trusted launch capabilities. The validation will take a minimum of 1 hour and results will be available as the image version property:

```json
"validationsProfile": {
  "executedValidations": [
      {
        "type": "TrustedLaunch",
        "status": "Succeeded",
        "version": "0.0.2",
        "executionTime": "2025-07-10T21:27:33.0113984+00:00"
      }
    ],
}
```

You can choose to explicitly bypass default for new Compute Gallery image definitions by setting `Standard` as the value for `securityType` under features:

```json
"features": [
    {
      "name": "SecurityType",
      "value": "Standard"
    }
],
"hyperVGeneration": "V2"
```

## Sharing

There are three main ways to share images in Azure Compute Gallery, depending on which users you want to share with:

| Sharing with: | People | Groups | Service principal | All users in a specific subscription or tenant | Publicly with all users in Azure |
|---|---|---|---|---|---|
| [RBAC sharing](#rbac) | Yes | Yes | Yes | No | No |
| RBAC + [direct shared gallery](#shared-directly-to-a-tenant-or-subscription)  | Yes | Yes | Yes | Yes | No |
| RBAC + [community gallery](#community-gallery) | Yes | Yes | Yes | No | Yes |

> [!NOTE]
> You can use images with read permissions on them to deploy virtual machines and disks.
>
> When you use the direct shared gallery, images are distributed widely to all users in a subscription or tenant. The community gallery distributes images publicly. When you share images that contain intellectual property, use caution to prevent widespread distribution.

### RBAC

Because the gallery, definition, and version are all resources, they can be shared using the built-in native Azure Roles-based Access Control (RBAC) roles. Using Azure RBAC roles you can share these resources to other users, service principals, and groups. You can even share access to individuals outside of the tenant they were created within. Once a user has access to the resource version, they can use it to deploy a VM or a Virtual Machine Scale Set.  Here's the sharing matrix that helps understand what the user gets access to:

| Shared with User     | Compute Gallery | Image Definition | Image version |
|----------------------|----------------------|--------------|----------------------|
| Compute Gallery | Yes                  | Yes          | Yes                  |
| Image definition     | No                   | Yes          | Yes                  |

We recommend sharing at the Gallery level for the best experience. We don't recommend sharing individual image versions. For more information about Azure RBAC, see [Assign Azure roles](/azure/role-based-access-control/role-assignments-portal).

For more information, see [Share using RBAC](./share-gallery.md).

### Shared directly to a tenant or subscription

Give specific subscriptions or tenants access to a direct shared gallery. Sharing a gallery with tenants and subscriptions give them read-only access to your gallery. For more information, see [Share a gallery with subscriptions or tenants](./share-gallery-direct.md).

> [!IMPORTANT]
> An Azure Compute Gallery direct shared gallery is currently in preview and is subject to the [preview terms for Azure Compute Gallery](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>
> To publish images to a direct shared gallery during the preview, you need to register at [https://aka.ms/directsharedgallery-preview](https://aka.ms/directsharedgallery-preview). Creating VMs from a direct shared gallery is open to all Azure users.
>
> During the preview, you need to create a new gallery, with the property `sharingProfile.permissions` set to `Groups`. When using the CLI to create a gallery, use the `--permissions groups` parameter. You can't use an existing gallery, the property can't currently be updated.
>
> You can't currently create a Flexible virtual machine scale set from an image shared to you by another tenant.

#### Limitations

During the preview:

- You can only share to subscriptions that are also in the preview.
- You can only share to 30 subscriptions and 5 tenants.
- A direct shared gallery can't contain encrypted image versions. Encrypted images can't be created within a gallery that is directly shared.
- Only the owner of a subscription, or a user or service principal assigned to the `Compute Gallery Sharing Admin` role at the subscription or gallery level will be able to enable group-based sharing.
- You need to create a new gallery,  with the property `sharingProfile.permissions` set to `Groups`. When using the CLI to create a gallery, use the `--permissions groups` parameter. You can't use an existing gallery, the property can't currently be updated.

### Community gallery

To share a gallery with all Azure users, you can create a community gallery. Community galleries can be used by anyone with an Azure subscription. Someone creating a VM can browse images shared with the community using the portal, REST, or the Azure CLI.

Sharing images to the community is a new capability in Compute Gallery. You can make your image galleries public, and share them to all Azure customers. When a gallery is marked as a community gallery, all images under the gallery become available to all Azure customers as a new resource type under Microsoft.Compute/communityGalleries. All Azure customers can see the galleries and use them to create VMs. Your original resources of the type `Microsoft.Compute/galleries` are still under your subscription, and private.

For more information, see [Share images using a community gallery](./share-gallery-community.md).

> [!IMPORTANT]
> You can't currently create a Flexible virtual machine scale set from an image shared by another tenant.

## Activity Log

The [Activity log](/azure/azure-monitor/essentials/activity-log) displays recent activity on the gallery, image, or version including any configuration changes and when it was created and deleted.  View the activity log in the Azure portal, or create a [diagnostic setting to send it to a Log Analytics workspace](/azure/azure-monitor/essentials/activity-log#send-to-log-analytics-workspace), where you can view events over time or analyze them with other collected data

The following table lists a few example operations that relate to gallery operations in the activity log. For a complete list of possible log entries, see [Microsoft.Compute Resource Provider options](/azure/role-based-access-control/resource-provider-operations#compute)

| Operation | Description |
|:---|:---|
| Microsoft.Compute/galleries/write | Creates a new Gallery or updates an existing one |
| Microsoft.Compute/galleries/delete  | Deletes the Gallery |
| Microsoft.Compute/galleries/share/action | Shares a Gallery to different scopes |
| Microsoft.Compute/galleries/images/read  | Gets the properties of Gallery Image |
| Microsoft.Compute/galleries/images/write  | Creates a new Gallery Image or updates an existing one |
| Microsoft.Compute/galleries/images/versions/read  | Gets the properties of Gallery Image Version |

## Billing

There is no extra charge for using the Azure Compute Gallery service. You'll be charged for the following resources:

- Storage costs of storing each replica. For images, the storage cost is charged as a snapshot and is based on the occupied size of the image version, the number of replicas of the image version and the number of regions the version is replicated to.
- Network egress charges for replication of the first resource version from the source region to the replicated regions. Subsequent replicas are handled within the region, so there are no additional charges.

For example, let's say you have an image of a 127 GB OS disk, that only occupies 10GB of storage, and one empty 32 GB data disk. The occupied size of each image would only be 10 GB. The image is replicated to 3 regions and each region has two replicas. There will be six total snapshots, each using 10GB. you'll be charged the storage cost for each snapshot based on the occupied size of 10 GB. you'll pay network egress charges for the first replica to be copied to the additional two regions. For more information on the pricing of snapshots in each region, see [Managed disks pricing](https://azure.microsoft.com/pricing/details/managed-disks/). For more information on network egress, see [Bandwidth pricing](https://azure.microsoft.com/pricing/details/bandwidth/).

## Best practices

- To prevent images from being accidentally deleted, use resource locks at the Gallery level. For more information, see [Protect your Azure resources with a lock](/azure/azure-resource-manager/management/lock-resources).

- Use ZRS wherever available for high availability. You can configure ZRS in the replication tab when you create a version of the image or VM application.
 For more information about which regions support ZRS, see [Azure regions with availability zones](/azure/reliability/availability-zones-region-support).

- Keep a minimum of 3 replicas for production images. For every 20 VMs that you create concurrently, we recommend you keep one replica.  For example, if you create 1000 VMs concurrently, you should keep 50 replicas (you can have a maximum of 50 replicas per region).  To update the replica count, please go to the gallery > Image Definition > Image Version > Update replication.

- Maintain separate galleries for production and test images, don't put them in a single gallery.

- For disaster recovery scenarios, it's best practice to have at least two galleries, in different regions. You can still use image versions in other regions, but if the region your gallery is in goes down, you can't create new gallery resources or update existing ones.

- When creating an image definition, keep the Publisher/Offer/SKU consistent with Marketplace images to easily identify OS versions.  For example, if you're customizing a Windows server 2019 image from Marketplace and store it as a Compute gallery image, please use the same Publisher/Offer/SKU that is used in the Marketplace image in your Compute Gallery image.

- Use `excludeFromLatest` when publishing images if you want to exclude a specific image version during VM or scale set creation. See [Gallery Image Versions - Create Or Update](/rest/api/compute/gallery-image-versions/create-or-update#galleryimageversionpublishingprofile).

- If you want to exclude a version in a specific region, use `regionalExcludeFromLatest`   instead of the global `excludeFromLatest`.  You can set both global and regional `excludeFromLatest` flag, but the regional flag will take precedence when both are specified.

   ```
    "publishingProfile": {
      "targetRegions": [
        {
          "name": "brazilsouth",
          "regionalReplicaCount": 1,
          "regionalExcludeFromLatest": false,
          "storageAccountType": "Standard_LRS"
        },
        {
          "name": "canadacentral",
          "regionalReplicaCount": 1,
          "regionalExcludeFromLatest": true,
          "storageAccountType": "Standard_LRS"
        }
      ],
      "replicaCount": 1,
      "excludeFromLatest": true,
      "storageAccountType": "Standard_LRS"
    }
   ```

- Set `safetyProfile.allowDeletionOfReplicatedLocations` to false on Image versions to prevent accidental deletion of replicated regions and prevent outage. You can also set this using CLI [allow-replicated-location-deletion](/cli/azure/sig/image-version#az-sig-image-version-create)

   ```
   {
   "properties": { 
    "publishingProfile": { 
      "targetRegions": [ 
        { 
          "name": "West US", 
          "regionalReplicaCount": 1, 
          "storageAccountType": "Standard_LRS", 
          // encryption info         
        }
      ], 
       "replicaCount": 1, 
       "publishedDate": "2018-01-01T00:00:00Z", 
       "storageAccountType": "Standard_LRS" 
    }, 
    "storageProfile": { 
       "source": { 
         "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Compute/images/{imageName}" 
      }, 
    }, 
    "safetyProfile": { 
       "allowDeletionOfReplicatedLocations" : false 
     }, 
   }, 
   "location": "West US", 
   "name": "1.0.0" 
   } 
   ```

- Set `BlockDeletionBeforeEndOfLife` to block deletion of the image before it's *end of life* date, ensuring protection against accidental deletion. Set this feature through [Rest API `blockdeletionbeforeendoflife`](/rest/api/compute/gallery-image-versions/create-or-update?view=rest-compute&preserve-view=true&tabs=HTTP#galleryimageversionsafetyprofile).

## SDK support

The following SDKs support creating Azure Compute Galleries:

- [.NET](/dotnet/api/overview/azure/virtualmachines#management-apis)
- [Java](/java/azure/)
- [Node.js](/javascript/api/overview/azure/arm-compute-readme)
- [Python](/python/api/overview/azure/virtualmachines)
- [Go](/azure/go/)

## Templates

You can create Azure Compute Gallery resources by using templates. There are several quickstart templates available:

- [Create a gallery](https://azure.microsoft.com/resources/templates/sig-create/)
- [Create an image definition in a gallery](https://azure.microsoft.com/resources/templates/sig-image-definition-create/)
- [Create an image version in a gallery](https://azure.microsoft.com/resources/templates/sig-image-version-create/)

## Related content

Learn how to deploy [images](shared-image-galleries.md) and [VM apps](vm-applications.md) by using Azure Compute Gallery.
