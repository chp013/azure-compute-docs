---
title: Share Azure Compute Gallery Resources Directly with Subscriptions and Tenants
description: Learn how to share Azure Compute Gallery resources explicitly with subscriptions and tenants.
author: sandeepraichura
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 02/14/2023
ms.author: saraic
ms.reviewer: cynthn
ms.custom: template-how-to , devx-track-azurecli
ms.devlang: azurecli
# Customer intent: As a cloud administrator, I want to share a gallery with specific subscriptions and tenants, so that I can provide controlled read-only access to images for users within my organization and facilitate image deployment efficiently.
---

# Share a gallery with all users in a subscription or tenants (preview)

This article covers how to share a gallery with specific subscriptions or tenants by using a direct shared gallery in Azure Compute Gallery. Sharing a gallery with tenants and subscriptions give them read-only access to it.

There are three main ways to share images in Compute Gallery, depending on which users you want to share with:

| Sharing with: | People | Groups | Service principal | All users in a specific subscription or tenant | Publicly with all users in Azure |
|---|---|---|---|---|---|
| [Role-based access control (RBAC) sharing](share-gallery.md) | Yes | Yes | Yes | No | No |
| RBAC + [direct shared gallery](./share-gallery-direct.md)  | Yes | Yes | Yes | Yes | No |
| RBAC + [community gallery](./share-gallery-community.md) | Yes | Yes | Yes | No | Yes |

## Important notes about direct shared galleries

- The feature of direct shared galleries is currently in preview and is subject to the [preview terms for Azure Compute Gallery](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

- To publish images to a direct shared gallery during the preview, you need to [register for the preview](https://aka.ms/directsharedgallery-preview). No additional access is required to consume images.

- Creating virtual machines (VMs) from a direct shared gallery is open to all Azure users in the target subscription or tenant that the gallery is shared with.

- Access to the feature of direct shared galleries is gated because in most scenarios, RBAC or app sharing is sufficient for same-subscription and cross-tenant scenarios. Request access to the feature only if you want to share images widely with all users in the subscription or tenant, and if your business case requires access to direct shared galleries.

- You can use images with read permissions on them to deploy VMs and disks.

- When you use a direct shared gallery, images are distributed widely to all users in a subscription/tenant. The community gallery distributes images publicly. When you share images that contain intellectual property, use caution to prevent widespread distribution.

## Limitations

During the preview:

- You can share up to 30 subscriptions and 5 tenants.

- Only images can be shared. You can't directly share a [VM application](vm-applications.md) during the preview.

- A direct shared gallery can't contain encrypted image versions. You can't create encrypted images within a gallery that's directly shared.

- Only the owner of a subscription, or a user or service principal assigned to the Compute Gallery Sharing Admin role at the subscription or gallery level, can enable group-based sharing.

- Only REST API, Azure CLI, and Azure portal support is available in preview. PowerShell and Terraform support is currently not available.

- Although portal support is available for this feature, consumption of images in the portal is available only from the pane for creating a VM or virtual machine scale set. There's no way to browse through direct shared images in the portal.

- The image version region in the gallery should be the same as the home region. The creation of cross-region versions where the home region is different from the gallery is not supported. However, after the image is in the home region, it can be replicated to other regions.

- The feature is not available in government clouds.

- *Known issue*: When you create a VM from a direct shared image by using the Azure portal, if you select a region and select an image, and then change the region, you get an error message: "You can only create VM in the replication regions of this image." The message appears even when the image is replicated to that region.

  To eliminate the error, select a different region, and then switch back to the region that you want. If the image is available, it should clear the error message.

## Prerequisites

You need to create a [new direct shared gallery](./create-gallery.md#create-a-direct-shared-gallery). During the preview, a direct shared gallery has the `sharingProfile.permissions` property set to `Groups`. When you use the Azure CLI to create a gallery, use the `--permissions groups` parameter. You can't use an existing gallery, and the property can't currently be updated.

## Share by using a direct shared gallery

Here's an overview of the process:

1. You create a gallery under `Microsoft.Compute/Galleries` and choose `groups` as a sharing option.
1. When you're ready, you share your gallery with subscriptions and tenants. Only the owner of a subscription, or a user or service principal with the Compute Gallery Sharing Admin role at the subscription or gallery level, can share the gallery.
1. The Azure infrastructure creates proxy read-only regional resources under `Microsoft.Compute/SharedGalleries`.

Only subscriptions and tenants that you've shared with can interact with the proxy resources. They never interact with your private resources.

As the publisher of the private resource, you should consider the private resource as your handle to the public proxy resources. The subscriptions and tenants that you've shared your gallery with see the gallery name as the subscription ID where the gallery was created, followed by the gallery name.

### [Portal](#tab/portaldirect)

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Enter **Azure Compute Gallery** in the search box, and select **Azure Compute Gallery** in the results.

1. On the **Azure Compute Gallery** page, select **Add**.

1. On the **Create Azure compute gallery** pane, on the **Basics** tab, select the correct subscription.

1. Complete all of the details on the **Basics** tab.

1. At the bottom of the pane, select **Next: Sharing method**.

   :::image type="content" source="media/create-gallery/create-gallery.png" alt-text="Screenshot that shows the tab where you enter basic details for a new gallery.":::

1. On the **Sharing** tab, select **RBAC + share directly**.

   :::image type="content" source="media/create-gallery/share-direct.png" alt-text="Screenshot that shows the option to both share by using role-based access control and share directly.":::

1. When you finish, select **Review + create**.

1. After validation passes, select **Create**.

1. When the deployment finishes, select **Go to resource**.

> [!NOTE]
> In the Azure portal, If you get an error "Failed to update Azure compute gallery", please verify if you have owner (or) compute gallery sharing admin permission on the gallery.

To share the gallery:

1. On the page for the gallery, on the left menu, select **Sharing**.

1. Under **Direct sharing settings**, select **Add**.

   :::image type="content" source="media/create-gallery/direct-share-add.png" alt-text="Screenshot that shows the option to share with a subscription or tenant.":::

1. If you want to share with someone within your organization, for **Type**, select **Subscription** or **Tenant**. Then choose the appropriate item in the **Tenants and subscriptions** dropdown list.

   If you want to share with someone outside your organization, select either **Subscription outside of my organization** or **Tenant outside of my organization**. Then paste or enter the ID into the text box.

   When a gallery is shared with the tenant, all subscriptions within the tenant get access to the image. They don't have to share it with individual subscriptions in the tenant.

1. When you finish adding items, select **Save**.

### [CLI](#tab/clidirect)

To create a direct shared gallery, you need to create the gallery with the `--permissions` parameter set to `groups`:

```azurecli-interactive
az sig create \
   --gallery-name myGallery \
   --permissions groups \
   --resource-group myResourceGroup  
```

To start sharing the gallery with a subscription or tenant, use [az sig share add](/cli/azure/sig#az-sig-share-add):

```azurecli-interactive
sub=<subscription-id>
tenant=<tenant-id>
gallery=<gallery-name>
rg=<resource-group-name>
az sig share add \
   --subscription-ids $sub \
   --tenant-ids $tenant \
   --gallery-name $gallery \
   --resource-group $rg
```

Remove access for a subscription or tenant using [az sig share remove](/cli/azure/sig#az-sig-share-remove):

```azurecli-interactive
sub=<subscription-id>
tenant=<tenant-id>
gallery=<gallery-name>
rg=<resource-group-name>

az sig share remove \
   --subscription-ids $sub \
   --tenant-ids $tenant \
   --gallery-name $gallery \
   --resource-group $rg
```

### [REST](#tab/restdirect)

Create a gallery for subscription or tenant-level sharing using the Azure REST API:

```rest
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}?api-version=2020-09-30

{
  "properties": {
    "sharingProfile": {
      "permissions": "Groups"
    }
  },
  "location": "{location}
}
 
```

Share a gallery to subscription or tenant:

```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{galleryName}/share?api-version=2020-09-30

{
  "operationType": "Add",
  "groups": [
    {
      "type": "Subscriptions",
      "ids": [
        "{SubscriptionID}"
      ]
    },
    {
      "type": "AADTenants",
      "ids": [
        "{tenantID}"
      ]
    }
  ]
}

```

Remove access for a subscription or tenant:

```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{galleryName}/share?api-version=2020-09-30

{
  "operationType": "Remove",
  "groups":[ 
    {
      "type": "Subscriptions",
      "ids": [
        "{subscriptionId1}",
        "{subscriptionId2}"
      ],
},
{
      "type": "AADTenants",
      "ids": [
        "{tenantId1}",
        "{tenantId2}"
      ]
    }
  ]
}

```

Reset sharing to clear everything in the `sharingProfile`:

```rest
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{galleryName}/share?api-version=2020-09-30 

{ 
 "operationType" : "Reset", 
} 
```

---

## Related content

- Create an [image definition and an image version](image-version.md).
- Create a VM from a [generalized](vm-generalized-image-version.md#direct-shared-gallery) or [specialized](vm-specialized-image-version.md#direct-shared-gallery) image from a direct shared image in the target subscription or tenant.
