---
title: "Deploy an update using the Windows Update for Business deployment service"
description: "With the Windows Update for Business deployment service, you can deploy Windows updates to sets of devices in an Azure AD tenant."
author: "Alice-at-Microsoft"
localization_priority: Normal
ms.prod: "w10"
doc_type: conceptualPageType
---

# Deploy an update using the Windows Update for Business deployment service

With the Windows Update for Business deployment service, you can deploy Windows updates to sets of devices in an Azure AD tenant.

When you deploy an update to a device, Windows Update will offer the specified update to the device if it has not yet taken the update. For example, if you deploy a feature update (e.g. version 20H2), the device will move to the specified version if it is enrolled in feature update management and currently on an older version of Windows 10. If the device is already at or above the specified version, it will stay on its current version. As long as it remains enrolled in feature update management, the device will not receive any other feature updates from Windows Update unless explicitly deployed using the deployment service.

Today, the deployment service supports deployments of Windows 10 feature updates. (See also: [Deploy an expedited update](windowsupdates-deploy-expedited-update.md))

## Prerequisites

* [deployment service prerequisites]
* Before you can use the deployment service to deploy updates of a given category (e.g. feature updates), devices must be [enrolled in management](windowsupdates-enroll.md) by the deployment service for that update category.

## Step 1: (Optional) Get a list of deployable updates

You can query the deployment service catalog to get a list of updates that can be deployed to devices as content in a deployment.

Below is an example of querying for all Windows 10 feature updates that are deployable by the deployment service.

### Request

```http
GET https://graph.microsoft.com/beta/admin/windows/updates/catalog/entries?$filter=isof('microsoft.graph.windowsUpdates.featureUpdateCatalogEntry')
```

### Response

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
    "value": [
        {
            "@odata.type": "#microsoft.graph.windowsUpdates.featureUpdateCatalogEntry",
            "id": "560a186a-1434-4364-8330-deb944b494ff",
            "displayName": "Windows 10, version 20H2",
            "releaseDate": "String (timestamp)",
            "deployableUntilDateTime": "String (timestamp)",
            "version": "20H2"
        },
        {
            "@odata.type": "#microsoft.graph.windowsUpdates.featureUpdateCatalogEntry",
            "id": "5e436dae-56bd-4925-bf8b-acf550e07227",
            "displayName": "Windows 10, version 2004",
            "releaseDate": "String (timestamp)",
            "deployableUntilDateTime": "String (timestamp)",
            "version": "2004"
        }
    ]
}
```

## Step 2: Create a deployment

A deployment specifies content to deploy, how and when to deploy the content, and the targeted devices. When a deployment is created, a deployment audience is automatically created as a relationship.

Below is an example of creating a deployment of a feature update, with optional settings configuring the [deployment schedule](windowsupdates-schedule-deployment.md) and [monitoring rules](windowsupdates-manage-monitoring-rules.md). The targeted devices will be specified in the next step.

### Request

```http
POST https://graph.microsoft.com/beta/admin/windows/updates/deployments
Content-type: application/json

{
    "@odata.type": "#microsoft.graph.windowsUpdates.deployment",
    "content": {
        "@odata.type": "microsoft.graph.windowsUpdates.featureUpdateReference",
        "version": "20H2"
    },
    "settings": {
        "@odata.type": "microsoft.graph.windowsUpdates.windowsDeploymentSettings",
        "rollout": {
            "devicesPerOffer": 100,
            "durationBetweenOffers": "P7D"
        },
        "monitoring": {
            "monitoringRules": [
                {
                    "@odata.type": "#microsoft.graph.windowsUpdates.monitoringRule",
                    "signal": "rollback",
                    "threshold": 5,
                    "action": "pauseDeployment"
                }
            ]
        }
    }
}
```

### Response

```http
HTTP/1.1 201 Created
Content-Type: application/json

{
    "@odata.type": "#microsoft.graph.windowsUpdates.deployment",
    "id": "b5171742-1742-b517-4217-17b5421717b5",
    "state": {
        "@odata.type": "microsoft.graph.windowsUpdates.deploymentState",
        "value": "offering",
        "reasons": [
            {
                "@odata.type": "microsoft.graph.windowsUpdates.deploymentStateReason",
                "value": "offeringByRequest"
            }
        ],
        "requestedValue": "none",
        "effectiveSinceDate": "String (timestamp)"
    },
    "content": {
        "@odata.type": "microsoft.graph.windowsUpdates.featureUpdateReference",
        "version": "20H2"
    },
    "settings": {
        "@odata.type": "microsoft.graph.windowsUpdates.windowsDeploymentSettings",
        "rollout": {
            "devicesPerOffer": 100,
            "durationBetweenOffers": "P7D",
            "startDateTime": null,
            "endDateTime": null
        },
        "monitoring": {
            "monitoringRules": [
                {
                    "@odata.type": "#microsoft.graph.windowsUpdates.monitoringRule",
                    "signal": "rollback",
                    "threshold": 5,
                    "action": "pauseDeployment"
                }
            ]
        },
        "userExperience": null
    },
    "createdDateTime": "String (timestamp)",
    "lastModifiedDateTime": "String (timestamp)"
}
```

## Step 3: Assign devices to the deployment audience

After a deployment is created, you can assign devices to the deployment audience. Devices can be assigned directly, or via updatable asset groups. Once the deployment audience is successfully updated, Windows Update will start offering the update to the relevant devices according to the deployment's settings.

Devices are automatically registered with the service when added to the members or exclusions collections of a deployment audience (i.e. an azureADDevice object is automatically created if it does not already exist).

Below is an example of adding updatable asset groups and Azure AD devices as members of the deployment audience, while also excluding a specific Azure AD device.

### Request

```http
POST https://graph.microsoft.com/beta/admin/windows/updates/deployments/{deploymentId}/audience/updateAudience
Content-type: application/json

{
    "addMembers": [
        {
            "@odata.type": "#microsoft.graph.windowsUpdates.updatableAssetGroup",
            "id": "String (identifier)"
        },
        {
            "@odata.type": "#microsoft.graph.windowsUpdates.azureADDevice",
            "id": "String (identifier)"
        },
        {
            "@odata.type": "#microsoft.graph.windowsUpdates.azureADDevice",
            "id": "String (identifier)"
        }
    ],
    "addExclusions": [
        {
            "@odata.type": "#microsoft.graph.windowsUpdates.azureADDevice",
            "id": "String (identifier)"
        }
    ]
}
```

### Response

```http
HTTP/1.1 202 Accepted
```

## During a deployment

While a deployment is in progress, you can pause the deployment by updating its state, as well as update its audience members and exclusions.

## After a deployment

After all devices assigned to a deployment's audience have been initially offered the update, it is possible that not all devices have started or completed the update, due to factors like device connectivity. As long as the deployment still exists, it will continue to make sure that Windows Update is offering the update to the assigned devices whenever they reconnect.
