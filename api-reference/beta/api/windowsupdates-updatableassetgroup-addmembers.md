---
title: "updatableAssetGroup: addMembers"
description: "Add members to an updatable asset group."
author: "Alice-at-Microsoft"
localization_priority: Normal
ms.prod: "w10"
doc_type: apiPageType
---

# updatableAssetGroup: addMembers
Namespace: microsoft.graph.windowsUpdates

[!INCLUDE [beta-disclaimer](../../includes/beta-disclaimer.md)]

Add members to an updatable asset group.

An [azureADDevice](../resources/windowsupdates-azureaddevice.md) can be added as a member; an [updatableAssetGroup](../resources/windowsupdates-updatableassetgroup.md) cannot be added as a member.

Adding an Azure AD device as a member of an updatable asset group automatically creates an Azure AD device object if it does not already exist.

You can also use the method [addMembersById](windowsupdates-updatableassetgroup-addmembersbyid.md).

## Permissions
One of the following permissions is required to call this API. To learn more, including how to choose permissions, see [Permissions](/graph/permissions-reference).

|Permission type|Permissions (from least to most privileged)|
|:---|:---|
|Delegated (work or school account)|WindowsUpdates.ReadWrite.All|
|Delegated (personal Microsoft account)|WindowsUpdates.ReadWrite.All|
|Application|WindowsUpdates.ReadWrite.All|

## HTTP request

<!-- {
  "blockType": "ignored"
}
-->
``` http
POST /updatableAssetGroup/{updatableAssetGroupId}/addMembers
```

## Request headers
|Name|Description|
|:---|:---|
|Authorization|Bearer {token}. Required.|
|Content-Type|application/json. Required.|

## Request body
In the request body, supply JSON representation of the parameters.

The following table shows the parameters that can be used with this action.

|Parameter|Type|Description|
|:---|:---|:---|
|assets|[updatableAsset](../resources/windowsupdates-updatableasset.md) collection|List of updatable assets to add as members of the updatable asset group.|



## Response

If successful, this action returns a `202 Accepted` response code.

## Examples

### Request
<!-- {
  "blockType": "request",
  "name": "updatableassetgroup_addmembers"
}
-->
``` http
POST https://graph.microsoft.com/beta/updatableAssetGroup/{updatableAssetGroupId}/addMembers

Content-Type: application/json
Content-length: 145

{
  "assets": [
    {
      "@odata.type": "#microsoft.graph.windowsUpdates.updatableAsset",
      "id": "String (identifier)"
    }
  ]
}
```


### Response
**Note:** The response object shown here might be shortened for readability.
<!-- {
  "blockType": "response",
  "truncated": true
}
-->
``` http
HTTP/1.1 202 Accepted
```