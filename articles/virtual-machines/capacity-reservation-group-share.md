---
title: Share a Capacity Reservation Group in Azure
description: Learn how to share a Capacity Reservation Group.
author: tferdou1
ms.author: tferdous
ms.service: azure-virtual-machines
ms.topic: how-to
ms.date: 09/23/2024
ms.reviewer: jushiman, bidefore
ms.custom: template-how-to, devx-track-azurecli, devx-track-azurepowershell
---

# Share a Capacity Reservation Group (Preview)

**Applies to:** :heavy_check_mark: Linux VMs :heavy_check_mark: Windows VMs :heavy_check_mark: Uniform scale set :heavy_check_mark: Flexible scale sets

> [!IMPORTANT]
> This feature is currently in **Preview**, see the [Preview Terms of Use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability. 

On-demand Capacity Reservation Group (CRG) can be shared with other subscriptions. Using this option can make it easier to manage some common configuration needs: 

1. Reuse of capacity reserved if there is disaster recovery –  customers often want assurance of capacity in another region or zone if their main region or zones become unavailable. On-demand capacity reservation is the primary method to gain such assurance. Now the reserved capacity can be shared with subscriptions hosting less critical workloads such as development and testing or batch. When the capacity is not needed for disaster recovery, reuse by less critical workloads saves on total capacity costs and drives more value from the reserved capacity. 


2. Central management of capacity – a central operations team often administers quota requests and term commitments as part of cost management. Now reserved capacity needs can be assessed and managed more centrally to align all three aspects of capacity cost management. 

 
3. Separate security and capacity concerns - applications implemented with multiple subscriptions for security reasons can operate from a common pool of capacity. This pattern is common with service providers serving their own end customers. 
 

4. More cost-effective scale-out with capacity assurance – applications that scale at different rates and times can share one pool of reserved capacity.


## How to share a Capacity Reservation Group

Sharing a Capacity Reservation Group with other subscriptions requires configuration of rights to share the CRG to a target subscription and then rights in the target subscription to see and use a shared CRG. Once completed, a user in the target subscription can reference the shared Capacity Reservation Group in a virtual machine (VM) or virtual machine scale set (VMSS) deployment to obtain Capacity Reservation Service Level Agreement (SLA).

> [!NOTE]
> There are no extra charges for using the shared Capacity Reservation Group feature. Unused reservations are charged to the subscription that owns the reservation. VM usage is charged to the subscription that uses the capacity reservation as it does today. For details on how Reserved Instance (RI) applies to the feature, see [Use of Reserved Instances with shared Capacity Reservation Groups](#use-of-reserved-instances-with-shared-capacity-reservation-groups) section.

Example: 

Consider this example: 

User 'A' creates and manages Capacity Reservation Groups in Subscription A. 

User 'B' creates and deploys Virtual Machines in Subscription B.  

The goal is for User 'A' to share Capacity Reservation Group X with Subscription B such that User 'B' can deploy VMs using Capacity Reservation Group X. 

 

Step 1: Share Capacity Reservation Group X 

1. A rights administrator in Subscription B must grant User 'A' capacity reservation group **share** permissions. The specific access required is Microsoft.Compute/capacityReservationGroups/share/action. 

 :::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-grant-access-share.png" alt-text="A screenshot showing subscription B rights admin granting share permissions to User A in Subscription A to share a Capacity Reservation Group with Subscription B.":::


2. User 'A' must then update the 'sharing profile' of CRG X to include Subscription B. The specific access right required by User 'A' to perform such action is Microsoft.Compute/capacityReservationGroups/write. 

  :::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-subscription-share.png" alt-text="A screenshot showing subscription A sharing a Capacity Reservation Group with Subscription B.":::


At this point, CRG X and all member Capacity Reservations are visible to Subscription B. 

Step 2: Grant user 'B' access to CRG X 

1. A rights administrator in Subscription A must grant user 'B' read and deploy rights to CRG X.  

The specific access rights are: 

- Microsoft.Compute/capacityReservationGroups/read
- Microsoft.Compute/capacityReservationGroups/deploy
- Microsoft.Compute/capacityReservationGroups/capacityReservations/read
- Microsoft.Compute/capacityReservationGroups/capacityReservations/deploy 

 :::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-grant-access-read-deploy.png" alt-text="A screenshot showing subscription A rights admin granting User B read and deploy rights for making deployments in the shared Capacity Reservation Group.":::


2. User 'B' can now add CRG X as the capacityReservationGroup property on Virtual Machines. The usual rules on the VM matching a Capacity Reservation apply. VM B must match a capacity reservation in CRG X on region, size, and zone (if zone is specified).  

  :::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-deploy-vm-shared.png" alt-text="A screenshot showing subscription B deploying Virtual Machine in the shared Capacity Reservation Group.":::


User 'B' was not granted write permissions to CRG X. User 'B' is not allowed to create more reservations in Capacity Reservation Group X, change reserved quantities, or make any other changes to the definition of Capacity Reservation Group X. 

## Usage patterns: 

The subscription sharing a Capacity Reservation Group can allow: 
- Cross subscriptions to access a specific Capacity reservation Group in the subscription
- Cross subscription to access all Capacity reservation Groups created in the subscription
- All subscriptions in a specific Management Group to access the Capacity Reservation Group

> [!NOTE]

> Azure strongly recommends using one master subscription to host Capacity Reservation Groups for each application, workload or usage scope to share across other subscriptions. Creating Capacity Reservation Groups in many different subscriptions and then cross-sharing in a matrix fashion will create management challenges and lead to confusion at VM deployment.


## Prerequisites for sharing CRG:  

- A subscription must have sufficient rights to be able to share a CRG
- A Subscription with whom a CRG is shared must have sufficient rights to be able to make deployments in capacity reservation (CR) in shared CRG
- A VM being deployed in the shared CRG must match the VM SKU, region, and zone if applicable 

## Limitations of sharing a Capacity Reservation Group

Limitations by design:
- Sharing works with an explicit list of target subscriptions. Azure does not support wildcard or tenant level sharing
- A CRG can be shared with a maximum of 100 subscriptions
- Sharing is per Capacity Reservation Group which grants access to all member Capacity Reservations. Individual Capacity Reservations cannot be shared. To isolate specific Capacity Reservations, create multiple Capacity Reservation Groups and share only those CRs that contain shared capacity. 
- By default, Capacity Reservation Group administrators in the subscription owning a Capacity Reservation Group cannot modify VM instances deployed by other subscriptions. If such VM access is desired such as to force VM deallocation, more rights to VMs on the shared subscriptions must be granted separately. 
  
Limitations for Public Preview:
- Portal support is not yet available; API and other Azure clients are available.  
- Reprovisioning of Virtual Machine Scale Set VMs using a shared Capacity Reservation Group is not supported during a zone outage
- Azure Kubernetes Service does not currently support deployment to a shared Capacity Reservation Group. 

## Share a Capacity Reservation Group: 

Capacity Reservation Groups can be shared by adding subscriptions in the sharing profile of new or existing Capacity Reservation Groups. Once shared, the subscriptions which are part of the sharing profile can deploy Virtual Machines or Virtual Machine Scale Sets in the shared Capacity Reservation Group. 

The Capacity Reservation Group can be shared with subscriptions who are in the same or different Entra ID of the subscription who creates the Capacity Reservation Group. 

### Add sharing profile on CRG creation

You can share a Capacity Reservation Group on creation by adding subscriptions in the sharing profile.

#### [API](#tab/api-1)

To share a Capacity Reservation group on creation, construct the following PUT request on Microsoft.Compute provider: 

```rest
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/capacityReservationGroups/myCapacityReservationGroup?api-version=2024-03-01 
``` 
    
In the request body, include the subscription ID to share the Capacity Reservation Group in the `sharing profile`
    
```json
{
    "location": "westus",
    "tags": {
        "department": "finance"
    },
    "zones": [
        "1"
    ],
    "properties": {
        "sharingProfile": {
            "subscriptionIds": [{
                    "id": "/subscriptions/{subscription-id1}"
                }, {
                    "id": "/subscriptions/{subscription-id2}"
                }
            ]
        }
    }
}
```

This example is to create shared CRG in zone 1 of Region West US. 

To learn more, see [Capacity Reservation Groups - Create Or Update](/rest/api/compute/capacity-reservation-groups/create-or-update)

 
#### [CLI](#tab/cli-1)

Create a Capacity Reservation group with sharing profile using `az capacity reservation group create`. 

The following example creates a shared capacity reservation group in the West Europe location: 

 ```azurecli-interactive
 az capacity reservation group create 
 -n reservationGroupName 
 -g myResourceGroup
 --sharing-profile "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" "subscriptions/yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy" -l westEurope 
 ```

To learn more, go to Azure PowerShell command [AzCapacityReservation](/cli/azure/capacity/reservation/group)


#### [PowerShell](#tab/powershell-1)

Create a shared Capacity Reservation group with `New-AzCapacityReservationGroup`. The following example creates a shared Capacity Reservation group in the East US location. 


```powershell-interactive
New-AzCapacityReservationGroup 
-ResourceGroupName myRG
-Location eastus
-Name myCapacityReservationGroup
-SharingProfile "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "/subscriptions/yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy" 
```

To learn more, see Azure PowerShell command [New-AzCapacityReservation](/powershell/module/az.compute/new-azcapacityreservationgroup)
To learn more on how to create a Capacity Reservation, see [Create a Capacity Reservation in Azure](capacity-reservation-create.md)


#### [ARM Template](#tab/armtemplatesample-1)

An [ARM template](/azure/azure-resource-manager/templates/overview) is a JavaScript Object Notation (JSON) file that defines the infrastructure and configuration for your project. The template uses declarative syntax. In declarative syntax, you describe your intended deployment without writing the sequence of programming commands to create the deployment. 

ARM templates let you deploy groups of related resources. In a single template, you can share Capacity Reservation group. You can deploy templates through the Azure portal, Azure CLI, or Azure PowerShell, or from continuous integration/continuous delivery (CI/CD) pipelines. 

The following ARM template creates a capacity reservation group shared with subscriptions SharedSubscriptionID1 and SharedSubscriptionID2. Alternatively, you can remove the *zone* information if you would like to deploy a regional shared capacity reservation group.

If your environment meets the prerequisites and you're familiar with using ARM templates, use the following template.


 ```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "resourcename": {
            "defaultValue": "SharedZonalCRGResourceName",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Compute/capacityReservationGroups",
            "apiVersion": "2023-09-01",
            "name": "[parameters('resourcename')]",
            "location": "SharedZonalCRGRegionName",
             "zones": [
                "1",
                "2",
                "3"
            ],
            "properties": {
				"sharingProfile" :
				{
					"subscriptionIds" : [
                        {
                          "id": "/subscriptions/SharedSubscriptionID1"
                        },
                        {
                          "id": "/subscriptions/SharedSubscriptionID2"
                        }
					]
				}
			}
        }
    ]
}

```

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Modify sharing profile of an existing Capacity Reservation Group 

After creating a Capacity Reservation group, you may want to modify the sharing profile. This article explains how to do the following actions using API, CLI, and PowerShell. 

### Add sharing profile to an existing CRG 

Add sharing profile and share with subscriptions for an existing Capacity Reservation Group. 

#### [API](#tab/api-2)

To add sharing profile to an existing Capacity Reservation group, construct the following PUT request on Microsoft.Compute provider.  

Following example adds sharing profile to an existing Capacity Reservation group called 'myCapacityReservationGroup' and shared with three subscription IDs: 

```rest
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/capacityReservationGroups/myCapacityReservationGroup?api-version=2024-03-01 
``` 
    
In the request body, include the subscription ID to share the Capacity Reservation Group in the `sharing profile`
    
```json
{
    "location": "westus",
    "tags": {
        "department": "finance"
    },
    "zones": [
        "1"
    ],
    "properties": {
        "sharingProfile": {
            "subscriptionIds": [
                {
                    "id": "/subscriptions/{subscription-id1}"
                },
                {
                    "id": "/subscriptions/{subscription-id2}"
                },
                {
                    "id": "/subscriptions/{subscription-id3}"
                }
            ]
        }
    }
}
```


To learn more, see [Capacity Reservation Groups - Create Or Update](/rest/api/compute/capacity-reservation-groups/create-or-update)

 
#### [CLI](#tab/cli-2)

Update a Capacity Reservation group with sharing profile using `az capacity reservation group update` with following command: 

 ```azurecli-interactive
 az capacity reservation group update
 -n ReservationGroupName 
 -g myResourceGroup
 --sharing-profile "subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
 ```

To learn more, go to Azure PowerShell command [AzCapacityReservationGroupUpdate](/cli/azure/capacity/reservation/group)


#### [PowerShell](#tab/powershell-2)

Update Capacity Reservation group with sharing profile using `Update-AzCapacityReservationGroup`
The following example is to update an existing capacity reservation group named 'myCapacityReservationGroup' with sharing profile having one subscription ID and later adding a new subscription ID. 

```powershell-interactive
Update-AzCapacityReservationGroup
-ResourceGroupName myRG
-Name myCapacityReservationGroup
-SharingProfile "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx", "/subscriptions/yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy" 
```

To learn more, see Azure PowerShell command [Update-AzCapacityReservation](/powershell/module/az.compute/update-azcapacityreservationgroup)


--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Stop sharing a Capacity Reservation Group

The Capacity Reservation Group owner can stop sharing a Capacity Reservation Group with a subscription or all subscriptions anytime. 

Unsharing of Capacity Reservation Group with cross subscription ID can be done while a Virtual Machine or Virtual Machine Scale Set from cross subscriptions remain associated with the Capacity Reservation Group. The associated resources get SLA until deallocated or reallocated. 

Once unsharing happens, any virtual machine or virtual machine scale set previously associated to CRG would fail to associate upon deallocation or reallocation. You can avoid this failure by removing the association from the capacity reservation group.

### Unsharing a Capacity Reservation Group with a subscription

To unshare a capacity Reservation Group with a subscription from Sharing profile, the subscription has to be removed from sharing profile. 

Consider an example where a Capacity Reservation Group was shared with Subscription ID 1, Subscription ID 2, and Subscription ID 3. The goal is to remove Subscription ID 3 only from the sharing profile. 
When updating the sharing profile of the Capacity Reservation Group, only Subscription ID 3 must be removed and not Subscription ID 1 and Subscription ID 2.

#### [API](#tab/api-3)

To remove a subscription from the sharing profile of an existing Capacity Reservation group, construct the following PUT request on Microsoft.Compute provider.  

Following example removes subscription ID3 from sharing profile to an existing Capacity Reservation group called 'myCapacityReservationGroup' that was shared with three subscription IDs previously: 

```rest
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/capacityReservationGroups/myCapacityReservationGroup?api-version=2024-03-01
``` 
    
In the request body, include the subscription ID to share the Capacity Reservation Group in the `sharing profile`
    
```json
{
    "location": "westus",
    "tags": {
        "department": "finance"
    },
    "zones": [
        "1"
    ],
    "properties": {
        "sharingProfile": {
            "subscriptionIds": [
                {
                    "id": "/subscriptions/{subscription-id1}"
                },
                {
                    "id": "/subscriptions/{subscription-id2}"
                }
            ]
        }
    }
}
```


To learn more, see [Capacity Reservation Groups - Create Or Update](/rest/api/compute/capacity-reservation-groups/create-or-update)

 
#### [CLI](#tab/cli-3)

You can remove a subscription ID from the sharing profile of an existing Capacity Reservation group  using `az capacity reservation group update`. 
Following example removes two subscription IDs from sharing profile of an existing Capacity Reservation  that was shared with three subscription IDs: 

 ```azurecli-interactive
 az capacity reservation group update
 -n ReservationGroupName 
 -g myResourceGroup
 --sharing-profile "subscriptions/yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy"
 ```

To learn more, go to [AzCapacityReservationGroupUpdate](/cli/azure/capacity/reservation/group).


#### [PowerShell](#tab/powershell-3)

You can remove a subscriptions ID from the sharing profile of an existing capacity reservation group using  `Update-AzCapacityReservationGroup`.  

The following example is to remove 2 subscriptions IDs from the sharing profile of an existing capacity reservation group named 'myCapacityReservationGroup' that was shared with three subscriptions IDs. 

```powershell-interactive
Update-AzCapacityReservationGroup
-ResourceGroupName myRG
-Name myCapacityReservationGroup
-SharingProfile "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

To learn more, see [Update-AzCapacityReservation](/powershell/module/az.compute/update-azcapacityreservationgroup).


--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

### Unsharing a Capacity Reservation Group with all subscriptions

To unshare a Capacity Reservation Group with all subscriptions, remove all subscriptions from the sharing profile. 

#### [API](#tab/api-4)

To remove all subscriptions from the sharing profile of an existing Capacity Reservation group, construct the following PUT request on Microsoft.Compute provider.  

Following example removes all subscriptions from sharing profile to an existing Capacity Reservation group called 'myCapacityReservationGroup'

```rest
PUT https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/myResourceGroup/providers/Microsoft.Compute/capacityReservationGroups/myCapacityReservationGroup?api-version=2024-03-01 
``` 
    
In the request body, include the subscription ID to share the Capacity Reservation Group in the `sharing profile`
    
```json
{
    "location": "westus",
    "tags": {
        "department": "finance"
    },
    "zones": [
        "1"
    ],
    "properties": {
        "sharingProfile": {
            "subscriptionIds": []
        }
    }
}
```

To learn more, see [Capacity Reservation Groups - Create Or Update](/rest/api/compute/capacity-reservation-groups/create-or-update).

#### [ARM Template](#tab/armtemplatesample-4)

An [ARM template ](/azure/azure-resource-manager/templates/overview) is a JavaScript Object Notation (JSON) file that defines the infrastructure and configuration for your project. The template uses declarative syntax. In declarative syntax, you describe your intended deployment without writing the sequence of programming commands to create the deployment. 

ARM templates let you deploy groups of related resources. In a single template, you can share Capacity Reservation group. You can deploy templates through the Azure portal, Azure CLI, or Azure PowerShell, or from continuous integration/continuous delivery (CI/CD) pipelines. 

If your environment meets the prerequisites and you're familiar with using ARM templates, use any of the following templates: 

The following ARM template unshared a capacity reservation group shared with all subscriptions. If your environment meets the prerequisites and you're familiar with using ARM templates, use the following template.


 ```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "resourcename": {
            "defaultValue": "SharedCRGResourceName",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Compute/capacityReservationGroups",
            "apiVersion": "2023-09-01",
            "name": "[parameters('resourcename')]",
            "location": "SharedCRGRegionName",
            "properties": {
				"sharingProfile" :
				{
					"subscriptionIds" : []
				}
			}
        }
    ]
}

```


#### [CLI](#tab/cli-4)

You can remove all subscription ID from the sharing profile of an existing Capacity Reservation group  using `az capacity reservation group update`.
Following example removes all subscription IDs from sharing profile of an existing Capacity Reservation  that was previously shared with one or more subscription IDs:

 ```azurecli-interactive
 az capacity reservation group update
 -n reservationGroupName 
 -g myResourceGroup
 --sharing-profile 
 ```

To learn more, [AzCapacityReservationGroup](/cli/azure/capacity/reservation/group).


#### [PowerShell](#tab/powershell-4)

You can remove all subscriptions ID from the sharing profile of an existing capacity reservation group using  `Update-AzCapacityReservationGroup`.

The following example removes all subscriptions IDs from the sharing profile of an existing capacity reservation group named 'myCapacityReservationGroup' that was shared with one or more subscriptions IDs.  


```powershell-interactive
Update-AzCapacityReservationGroup 
-ResourceGroupName myRG
-Name myCapacityReservationGroup
-SharingProfile "" 
```

To learn more, see [Update-AzCapacityReservation](/powershell/module/az.compute/update-azcapacityreservationgroup).


--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## Deletion of shared Capacity Reservation Group

- Users with sufficient rights only within the subscription creating the shared Capacity Reservation Group can delete the Capacity Reservation Group
- Users from shared subscriptions cannot delete the shared Capacity Reservation Group
- Azure allows a capacity reservation group to be deleted when all the member capacity reservations are deleted
- Azure allows a capacity reservation to be deleted when no VMs are associated to the CR
- Unsharing of CRG with shared subscription happens as part of shared Capacity Reservation Group deletion process

See [Modify a capacity reservation](/azure/virtual-machines/capacity-reservation-modify) for deletion guidance.

## Using shared Capacity Reservation Group

Once the Capacity Reservation Group is successfully shared, users with sufficient rights from shared subscription can deploy Virtual Machines or Virtual Machine Scale Set in shared Capacity Reservation Group.

> [!NOTE]
> The subscription deploying the shared CRG will need to hold their own quota for deploying the CRG. Subscription making a deployment in the shared reservation will need to hold its own quota


### Availability zone mapping with shared zonal Capacity Reservation Groups 

The availability zones visible to each Azure subscription represent a logical mapping to underlying groups of physical data centers that constitute a physical zone. To promote efficient distribution of resources across availability zones, each Azure subscription gets a random logical to physical mapping of zones.
For example, subscription A and subscription B can have different logical mappings. 

Consider the following example: 

If Sub A deploys to AZ1, the deployment goes to physical zone 1. But if Sub B deploys to AZ1, the deployment goes to physical zone 2:
:::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-a-b-mapping.png" alt-text="A screenshot showing subscription A and subscription B having different physical to logical zone mapping.":::

Now consider a Capacity Reservation deployed by Subscription A to logical AZ1. The result is reserved capacity in physical zone 1.  
:::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-a-reservation.png" alt-text="A screenshot showing subscription A creating a capacity reservation in logical zone 1.":::

If Subscription B deployed a VM to logical AZ1 using the shared Capacity Reservation Group, the deployment would fail because Sub B AZ1 resolves to physical zone 2. And there is no reserved capacity in physical zone 2. 
:::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-a-b-not-aligned.png" alt-text="A screenshot showing subscription A capacity reservation and subscription B VM are in two different logical zones resulting in failure.":::


The solution is to reconcile the logical to physical mappings for Subscription A and Subscription B. Which shows Subscription B should deploy to AZ2 to access reserved capacity in physical zone 1. 
:::image type="content" source="./media/capacity-reservation-group-share/capacity-reservation-group-a-b-aligned.png" alt-text="A screenshot showing subscription A capacity reservation and subscription B VM are in same logical zones resulting in success.":::

When using a shared Capacity Reservation Group with zones, all subscriptions have the logical view of availability zones from the CRG subscription which is likely different than the logical view of availability zones from the target subscription. When deploying a VM or VMSS to a shared CRG, the user must remap the zones. To ensure that the Azure resources are efficiently distributed across Availability zones in a Region, each subscription has independent logical zone mapping for Availability zones. This means that logical to physical zone mapping may or may not be the same across subscriptions. 


To check the Physical Zone and Logical Zone mapping for your subscription: 

- Open [Subscriptions - List Locations - REST API (Azure Resource Management)](/rest/api/resources/subscriptions/list-locations)
- Enter your subscription name in the field 'subscriptionId' in the Parameters section
- Execute 'Run'
- Check the logical to physical zone mapping information in the JSON returned 

For more information, see [Physical and Logical availability zones](/azure/reliability/availability-zones-overview?tabs=azure-cli#physical-and-logical-availability-zones).

### Use of Reserved Instances with shared Capacity Reservation Groups 

Sharing a Capacity Reservation Group does not alter the scope of any Reserved Instances or Savings Plans. If either the CRG or the VM is deployed from a scope not covered by prepaid discounts, the pay-as-you-go price is charged. 

To share Reserved Instance discounts between a Capacity Reservation Group and VMs deployed from another subscription, the CRG subscription and the VM subscription must share the same RI scope. If the two subscriptions share an enrollment or a management group, then Reserved Instances set to the corresponding scope works automatically. 

#### Associate or create a single Virtual Machine with shared Capacity Reservation Group

Single Virtual Machine can be deployed in shared Capacity Reservation Group using Powershell, CLI, or REST API. See [Associate a virtual machine to a Capacity Reservation group](/azure/virtual-machines/capacity-reservation-associate-vm).

#### Remove a single Virtual Machine from Shared Capacity Reservation Group 

Single Virtual Machine can be removed from Shared Capacity Reservation Group using Powershell, CLI, or REST API. See [Remove a virtual machine association from a Capacity Reservation group](/azure/virtual-machines/capacity-reservation-remove-vm)

#### Associate or create a Virtual Machine Scale Set with shared Capacity Reservation Group 

VMSS Flex and Uniform can be deployed in shared Capacity Reservation Group using Powershell, CLI, or REST API. To learn more, see [Associate a scale set-Flexible](/azure/virtual-machines/capacity-reservation-associate-virtual-machine-scale-set-flex) and [Associate a scale set-Uniform](/azure/virtual-machines/capacity-reservation-associate-virtual-machine-scale-set)

#### Remove Virtual Machine Scale Set from Shared Capacity Reservation Group

Virtual Machine Scale Set- Flex and Uniform can be removed from shared Capacity Reservation Group using Powershell, CLI, or REST API. To learn more, see [Remove a scale set](/azure/virtual-machines/capacity-reservation-remove-virtual-machine-scale-set).

## View shared Capacity Reservation Group

Once a Capacity Reservation Group is shared successfully, the reservations are immediately available for use with single Virtual Machines and Virtual Machine Scale Sets. 

You can view the subscription IDs the Capacity Reservation Group are shared with from the sharing profile. 

To learn more, see [Create a Capacity Reservation](/azure/virtual-machines/capacity-reservation-create).


### [API](#tab/api-5)

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Compute/capacityReservationGroups/{capacityReservationGroupName}?api-version=2023-07-01
``` 
    
    
```json
{
    "name": "SharedCRG",
    "id": "/subscriptions/{subscriptionID1}/resourceGroups/{MyRG}/providers/Microsoft.Compute/capacityReservationGroups/SharedCRG",
    "type": "Microsoft.Compute/capacityReservationGroups",
    "location": "eastus2",
    "properties": {
        "capacityReservations": [{
                "id": "/subscriptions/{subscriptionID1}/resourceGroups/{MyRG}/providers/Microsoft.Compute/capacityReservationGroups/{SharedCRG}/capacityReservations/{CR1}"
            },
        ],
        "sharingProfile": {
            "subscriptionIds": [{
                    "id": "/subscriptions/{subscriptionID2}"
                }
            ]
        },
        "provisioningState": "Succeeded"
    }
}
```

To learn more, see [Capacity Reservation Group-GET](/rest/api/compute/capacity-reservation-groups/get)

 
### [CLI](#tab/cli-5)

See [az capacity reservation group show](/cli/azure/capacity/reservation/group).


### [PowerShell](#tab/powershell-5)


See [Get-AzCapacityReservationGroup](/powershell/module/az.compute/get-azcapacityreservationgroup).

--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->

## View the list of Capacity Reservation Groups for a subscription

The list of all Capacity Reservation Groups that are created locally or shared with by other subscriptions, can be viewed for a given subscription. Extra parameter 'resourceIdsonly' needs to be passed to view the shared Capacity Reservation Groups.

### [API](#tab/api-6)

Enables fetching Resource Ids for all capacity reservation group resources shared with the subscription.  

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/capacityReservationGroups?api-version=2023-09-01&resourceIdsOnly=sharedwithsubscription 
``` 

Enables fetching Resource Ids for all capacity reservation group resources shared with the subscription and created in the subscription: 

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/capacityReservationGroups?api-version=2023-09-01&resourceIdsOnly=All 
``` 

Enables fetching Resource Ids for all capacity reservation group resources shared with the subscription and created in the subscription: 

```rest
GET https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Compute/capacityReservationGroups?api-version=2023-09-01&resourceIdsOnly=CreatedInSubscription 
``` 

    
```json
{
    "value": [
        {
            "id": "/subscriptions/{subscriptionId1}/resourceGroups/{resourceGroupName1} /providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName1} ",
            "type": "Microsoft.Compute/capacityReservationGroups",
            "location": "EastUS2"
        },
        {
            "id": "/subscriptions/{subscriptionId2}/resourceGroups/{resourceGroupName2} /providers/Microsoft.Compute/capacityReservationGroups/{CapacityReservationGroupName2} ",
            "type": "Microsoft.Compute/capacityReservationGroups",
            "location": "EastUS2"
        }
    ]
}
```

To learn more, see [Capacity Reservation Group-List by subscription](/rest/api/compute/capacity-reservation-groups/list-by-subscription).

 
### [CLI](#tab/cli-6)

Enables fetching Resource Ids for all capacity reservation group resources shared with the subscription and created in the subscription

 ```azurecli-interactive
 az capacity reservation group list
 --resource-ids-only all 
 ```

Enables fetching Resource Ids for all capacity reservation group resources created in the subscription 

 ```azurecli-interactive
 az capacity reservation group list
 --resource-ids-only CreatedInSubscription
 ```

Enables fetching Resource Ids for all capacity reservation group resources shared with the subscription 

 ```azurecli-interactive
 az capacity reservation group list
 --resource-ids-only SharedWithSubscription
 ```

To learn more, go to [AzCapacityReservationGroupList](/cli/azure/capacity/reservation/group).


### [PowerShell](#tab/powershell-6)

Enables fetching Resource Ids for all capacity reservation group resources shared with the subscription and created in the subscription

```powershell-interactive
Get-AzCapacityReservationGroup
-ResourceIdsOnly All 
```

Enables fetching Resource Ids for all capacity reservation group resources created in the subscription 

```powershell-interactive
Get-AzCapacityReservationGroup
-ResourceIdsOnly CreatedInSubscription 
```

Enables fetching Resource Ids for all capacity reservation group resources shared with the subscription 

```powershell-interactive
Get-AzCapacityReservationGroup
-ResourceIdsOnly SharedWithSubscription
```

To learn more, see Azure PowerShell command [Update-AzCapacityReservation](/powershell/module/az.compute/update-azcapacityreservationgroup).


--- 
<!-- The three dashes above show that your section of tabbed content is complete. Don't remove them :) -->


## Next steps

> [!div class="nextstepaction"]
> [Learn how to associate a VM to a capacity reservation group](capacity-reservation-associate-vm.md)



