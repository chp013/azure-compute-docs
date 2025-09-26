---
title: Soft Delete on Azure Compute Gallery
description: Learn how to enable soft delete feature on azure compute gallery and recover images from accidental deletes.
author: sandeepraichura
ms.service: azure-virtual-machines
ms.subservice: gallery
ms.topic: how-to
ms.date: 09/30/2025
ms.author: saraic
ms.reviewer: jushiman
ms.custom: template-how-to
ms.devlang: azurecli
# Customer intent: As a cloud service provider or cloud administrator, I want to enable soft delete on Azure compute gallery, so that I can recover images that were accidentally deleted.
---

# What is Soft Delete
The Soft Delete feature in Azure Compute Gallery lets you recover accidentally deleted within a 7-day retention period. After this timeframe, the platform will permanently delete the resources after the 7-day retention period. Currently, this feature is available for ACG Images (in preview), and support for VM Apps recovery is expected soon.
Soft delete is available for the following Gallery types:

-	Private
-	Direct Shared Gallery
-	Community Gallery

When a resource such as an image is deleted using Soft Delete, it is not immediately removed from the system. Instead, it enters a "soft-deleted" state, during which it remains recoverable for up to seven days. This grace period gives administrators or users time to review and restore any resources that may have been mistakenly deleted, thereby preventing permanent loss. 
After the retention window expires, however, the deleted items are automatically purged and cannot be recovered by any means. Soft Delete is particularly useful in environments where accidental deletions can disrupt workflows or cause data loss.

## Pre-requisites for using the Soft Delete Feature

- The Soft Delete feature is currently in preview and is subject to the [preview terms for Azure Compute Gallery](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). This Preview is intended for testing, evaluation, and feedback purposes only. Production workloads aren't recommended. Some aspects of this feature might change with general availability (GA).
- Register the "Azure Compute Gallery Soft Delete Feature" [Preview Feature](articles/azure-resource-manager/management/preview-features.md) from Portal.
- Alternatively, you can also register by running the following az feature register command from CLI
   ```
   az feature register --name SIGSoftDelete --namespace Microsoft.Compute 
   ```
- Minimum API version required to use Soft Delete is 2024-03-03

## Limitations

-	Only supported via Portal/RestAPI with some limitations.
-	Cannot update the retention policy
-	VM Apps Recovery is not supported
-	Deleting a Gallery/Image definition with Soft Deleted Resources is not supported
-	Only the owner of a subscription, or a user or service principal assigned to the Compute Gallery Sharing Admin role at the subscription or gallery level will be able to enable soft delete
-	Not supported in National Clouds
-	Cannot enable soft delete if Gallery/Image definition and Image version are in different locations
-	VMSS Flex is not compatible with Soft Delete feature
-	When an image in a Gallery with Soft Delete enabled has a failed provisioning state, it cannot be deleted. The recommended workaround is to temporarily disable Soft Delete on the Gallery, delete the image, and then re-enable Soft Delete.

## Billing
Soft Delete is provided at no cost. After an image is soft deleted, only one replica per region is retained, and customers incur charges for that single replica until the image is permanently deleted.


## How to enable Soft Delete
In this example, a Gallery will be enabled for Soft Delete. Customers can update an existing Gallery to have soft delete enabled

### [REST](#tab/rest)
To enable Soft Delete on a Gallery, send the following `PUT` request. As part of the request, include the `Softdeletepolicy` and `isSoftDeleteEnabled` value to `true`


```rest
PUT 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}?api-version=2024-03-03 
{
"properties": {
"softDeletePolicy": {
"isSoftDeleteEnabled": true
}
},
"location": "{location}
}
```

### [Portal](#tab/portal)

When creating a new Gallery:

1. On the Basics tab.

1. At the bottom of the page, select **Enables soft delete for Image versions**.

   :::image type="content" source="media/soft-delete/enable-soft-delete-on-create.png" alt-text="Screenshot that shows the enabling of Soft Delete on new Gallery creation.":::

1. When you're done, select **Review + Create**.

---

## How to Soft Delete an Image
Once the Soft Delete is Enabled on the Gallery, all images in the Gallery will be soft deleted.

### [REST](#tab/rest)
To Soft Delete an Image, send the following `DELETE` request on the resource you intend to delete.

```rest
DELETE 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/images/{image-defination-name}/versions/{version-name}?api-version=2024-03-03
```
### [Portal](#tab/portal)

Soft Delete an Image:

1. On the Image Definitions (or) Image Versions.

1. At the bottom of the page, select the **Image Version** to be deleted and click on **Delete** **Enables soft delete for Image versions**.

   :::image type="content" source="media/soft-delete/soft-delete-image.png" alt-text="Screenshot that shows soft deleting an image.":::

---

## List all the Soft Deleted Images
To view the list of soft deleted images in a Gallery/Image Definition. You can make a Get API call (or) go to the image definition in the Portal to list all the soft deleted images.

### [REST](#tab/rest)
To list the Soft deleted images in a Gallery. send the following `GET` request on the resource on the image definition to list all the soft deleted images.

```rest
GET 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/softDeletedArtifactTypes/Images/artifacts/{image-defination-name}/versions?api-version=2024-03-03
```

### [Portal](#tab/portal)

List all Soft Deleted Images

1. On the Image Definitions blade.

1. Switch the toggle button to select **Show soft deleted versions** to list all the soft deleted image versions

   :::image type="content" source="media/soft-delete/list-soft-deleted-images.png" alt-text="Screenshot that list all the soft deleted images in a gallery.":::

---

## Recover a Soft Deleted Image
In this example, you will see how to recover a soft deleted image.

### [REST](#tab/rest)
To recover Soft deleted images in a Gallery. send the following `PUT` request on the image version to be recovered along with the home region of the image

```rest
PUT 
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/images/{image-defination-name}/versions/{version-name}?api-version=2024-03-03
{
    "location": "eastus2euap"
}
```

### [Portal](#tab/portal)

Recover a Soft Deleted Image

1. On the Image Definitions blade.

1. Switch the toggle button to select **Show soft deleted versions** to list all the soft deleted image versions

   :::image type="content" source="media/soft-delete/list-soft-deleted-images.png" alt-text="Screenshot that list all the soft deleted images in a gallery.":::

---

## Hard Delete the Images
Hard delete will permanently delete the image without the posibility of a recovery.

### [REST](#tab/rest)
To hard delete an image in a Gallery. send the following `DELETE` request on the image version to be hard deleted

```rest
DELETE  
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{rgName}/providers/Microsoft.Compute/galleries/{gallery-name}/softDeletedArtifactTypes/Images/artifacts/{image-defination-name}/versions/{version-name}?api-version=2024-03-03
```

### [Portal](#tab/portal)

Hard delete an image

1. On the Image Definitions blade.

1. Switch the toggle button to select **Show soft deleted versions** to list all the soft deleted image versions and select the image version to hard delete and click on delete

   :::image type="content" source="media/soft-delete/hard-soft-an-image.png" alt-text="Screenshot that shows how to hard delete an image":::

---

## Frequently Asked Questions

### Does Soft Delete become effective immediately upon being enabled in the Gallery?
Yes, as long as the soft delete operation completes successfully. 

### Can I enable Soft Delete on an existing Gallery and instantly delete and recover images, or is there a delay?
Yes, as long as the soft delete operation completes successfully.

### Can I update the retention period beyond 7 days?
No, cannot update retention period but it will be supported in the future.

### Can I delete a Gallery (or) Image definition that has soft deleted images?
No, however users can either delete all the soft deleted images (or) disable soft delete on the Gallery to delete the image definition

