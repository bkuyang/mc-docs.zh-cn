---
title: 创建自定义 Azure 资源管理器角色并将其分配给服务主体 - Azure
description: 本文提供有关如何使用 Azure CLI 创建自定义 Azure 资源管理器角色，并将其分配给 IoT Edge 上实时视频分析的服务主体的指南。
ms.topic: how-to
origin.date: 05/27/2020
ms.date: 07/27/2020
ms.openlocfilehash: da62f0734a3254e0d2bd29185aa0519b22613637
ms.sourcegitcommit: 091c672fa448b556f4c2c3979e006102d423e9d7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/24/2020
ms.locfileid: "87162828"
---
# <a name="create-custom-azure-resource-manager-role-and-assign-to-service-principal"></a>创建自定义 Azure 资源管理器角色并将其分配给服务主体

IoT Edge 模块实例上的实时视频分析需要可用的 Azure 媒体服务帐户，才能正常工作。 IoT Edge 模块上的实时视频分析与 Azure 媒体服务帐户之间的关系是通过一组模块孪生属性建立的。 其中一个孪生属性是[服务主体](/active-directory/develop/app-objects-and-service-principals#service-principal-object)，它使模块实例能够与媒体服务帐户进行通信并触发必要的操作。 为了最大程度地减少来自边缘设备滥用和/或意外数据泄露的可能性，此服务主体应拥有最少的权限。

本文介绍创建自定义 Azure 资源管理器角色的步骤，该角色用于创建服务主体。

## <a name="prerequisites"></a>先决条件  

本文的先决条件如下所示：

* 具有所有者订阅的 Azure 订阅。
* 有权创建应用并将服务主体分配给角色的 Azure Active Directory。

检查帐户是否有足够权限的最简方法是使用门户。 请参阅[检查所需的权限](/active-directory/develop/howto-create-service-principal-portal#required-permissions)。

## <a name="overview"></a>概述  

我们将按照以下顺序介绍创建自定义角色并将其与服务主体关联的步骤：

1. 如果你没有媒体服务帐户，请创建一个。
1. 创建服务主体。
1. 创建具有有限权限的自定义 Azure 资源管理器角色。
1. 使用创建的自定义角色“限制”服务主体权限。
1. 运行一个简单的测试，查看是否能够成功地限制服务主体。
1. 捕获将在 IoT Edge 部署清单中使用的参数。

### <a name="create-a-media-services-account"></a>创建媒体服务帐户  

如果你没有媒体服务帐户，请使用以下步骤创建一个。

1. 使用以下命令模板将 Azure 订阅设置为默认帐户：
    
    ```
    az account set --subscription " <yourSubscriptionName or yourSubscriptionId>"
    ```
1. 创建[资源组](/cli/group?view=azure-cli-latest#az-group-create)和[存储帐户](/cli/storage/account?view=azure-cli-latest#az-storage-account-create)。
1. 现在，通过在 Cloud Shell 中使用以下命令模板来创建 Azure 媒体服务帐户：

    ```
    az ams account create --name <yourAMSAccountName>  --resource-group <yourResouceGroup>  --storage-account <yourStorageAccountName>
    ```

### <a name="create-service-principal"></a>创建服务主体  

现在，我们将创建一个新的服务主体并将其关联到媒体服务帐户。

如果没有任何身份验证参数，则将基于密码的身份验证与服务主体的随机密码配合使用。 

```
az ams account sp create --account-name < yourAMSAccountName > --resource-group < yourResouceGroup >
```

此命令会生成如下响应：

```
{
  "AadClientId": "00000000-0000-0000-0000-000000000000",
  "AadEndpoint": "https://login.chinacloudapi.cn",
  "AadSecret": "<yourServicePrincipalPassword>",
  "AadTenantId": "00000000-0000-0000-0000-000000000000",
  "AccountName": " <yourAMSAccountName >",
  "ArmAadAudience": "https://management.core.chinacloudapi.cn/",
  "ArmEndpoint": "https://management.chinacloudapi.cn/",
  "Region": "China East 2",
  "ResourceGroup": " <yourResouceGroup >",
  "SubscriptionId": "<yourSubscriptionId>"
}

```
1. 具有密码身份验证的服务主体的输出包括密码密钥，在本例中，此密码密钥为“AadSecret”参数。 

    请确保复制此值 - 它不可检索。 如果忘记了密码，请[重置服务主体凭据](/cli/create-an-azure-service-principal-azure-cli?view=azure-cli-latest#reset-credentials)。
1. appId 和租户密钥分别在输出中显示为“AadClientId”和“AadTenantId”。 它们用于服务主体身份验证。 请记录其值，但它们随时可以通过 [az ad sp list](/cli/ad/sp?view=azure-cli-latest#az-ad-sp-list) 检索。

### <a name="create-a-custom-role-definition"></a>创建自定义角色定义  

若要创建自定义角色，请按照以下步骤操作：

1. 在本地系统上创建角色定义 JSON 文件，并将以下文本保存在该文件中。 
    1. 将 < yourSubscriptionId> 替换为你的 Azure 订阅 ID
    1. 此角色仅允许执行以下操作：
        * listContainerSas - 帮助模块列出具有共享访问签名 (SAS) 的存储容器 URL，以便上传和下载资产内容。
        * 写入资产 - 帮助模块创建或更新任何资产
        * listEdgePolicies - 列出应用于边缘设备的策略  
        
        ```
        {
          "Name": "LVAEdge User",
          "IsCustom": true,
          "Description": "Can create assets, view list of containers and view Edge policies",
          "Actions": [
            "Microsoft.Media/mediaservices/assets/listContainerSas/action",
            "Microsoft.Media/mediaservices/assets/write",
            "Microsoft.Media/mediaservices/listEdgePolicies/action"
          ],
          "NotActions": [],
          "DataActions": [],
          "NotDataActions": [],
          "AssignableScopes": [
            "/subscriptions/<yourSubscriptionId>"
          ]
        }
        ```  
          
1. 创建完成后，运行以下命令模板以在订阅中创建新的角色定义：
    
    ```
    az role definition create --role-definition "<location of the Role Definition JSON file >"
    ```

    成功执行命令后，你将看到以下输出：
    
    ```
    {
      "assignableScopes": [
      "/subscriptions/<yourSubscriptionId>"
      ],
      "description": "Can create assets, view list of containers and view Edge policies",
      "id": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<unique name>",
      "name": "<unique name>",
      "permissions": [
        {
          "actions": [
            "Microsoft.Media/mediaservices/assets/listContainerSas/action",
            "Microsoft.Media/mediaservices/assets/write",
            "Microsoft.Media/mediaservices/listEdgePolicies/action",
          ],
          "dataActions": [],
          "notActions": [],
          "notDataActions": []
        }
      ],
      "roleName": " LVAEdge User ",
      "roleType": "CustomRole",
      "type": "Microsoft.Authorization/roleDefinitions"
    }
    ```

### <a name="create-role-assignment"></a>创建角色分配  

若要添加角色分配，你需要服务主体的 objectId，才能向该主体分配刚刚创建的自定义角色。

在 Cloud Shell 中使用以下命令获取 objectId：

```
az ad sp show --id "<appId>" | Select-String "objectId"
```

> [!NOTE]
> 可以从[创建服务主体](#create-service-principal)步骤的输出中检索 `<appId>`。

上述命令将输出服务主体的 objectId。 

```
“objectId” : “<yourObjectId>”,
```

使用 [az role assignment create 命令](/cli/role/assignment?view=azure-cli-latest#az-role-assignment-create)模板将自定义角色与服务主体关联：

```
az role assignment create --role “LVAEdge User” --assignee-object-id < objectId>    
```

参数：

|参数|说明| 
|---|---|
|--role |自定义角色名称或 ID。 在本例中为：“LVAEdge User”。|
|--assignee-object-id|将使用的服务主体的对象 ID。|

结果将如下所示：

```
{
  "canDelegate": null,
  "id": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleAssignments/<yourCustomRoleId>",
  "name": "<yourCustomRoleId>",
  "principalId": "<yourServicePrincipalId>",
  "principalType": "ServicePrincipal",
  "roleDefinitionId": "/subscriptions/<yourSubscriptionId>/providers/Microsoft.Authorization/roleDefinitions/<yourRoleDefinitionId>",
  "scope": "/subscriptions/<yourSubscriptionId>",
  "type": "Microsoft.Authorization/roleAssignments"
} 
```

### <a name="confirm-that-role-assignment-happened"></a>确认角色分配已发生

若要确认服务主体现在是否已与刚刚创建的自定义角色关联，请运行以下命令：

```
az role assignment list  --assignee < objectId>
```

结果应如下所示：

```
[
  {
    "canDelegate": null,
    "id": "/subscriptions/xxx/providers/Microsoft.Authorization/roleAssignments/00000000-0000-0000-0000-000000000000",
    "name": "00000000-0000-0000-0000-000000000000",
    "principalId": "<yourServicePrincipalID>",
    "principalName": "<yourServicePrincipalName>",
    "principalType": "ServicePrincipal",
    "roleDefinitionId": "/subscriptions/xxx/providers/Microsoft.Authorization/roleDefinitions/zzz",
    "roleDefinitionName": "LVAEdge User",
    "scope": "/subscriptions/<yourSubscription ID>",
    "type": "Microsoft.Authorization/roleAssignments
  }
]  
```
 
查找“roleDefinitionName”，并查看其值是否设置为“LVAEdge User”。 

这可以确认我们已将自定义用户角色与用于应用程序的服务主体关联。

### <a name="test-the-service-principal-rbac"></a>测试服务主体 RBAC  

1. 使用服务主体登录。 为此，我们将需要 3 条信息，以便 Azure Active Directory 授予我们适当的访问令牌，这些令牌可以从[创建服务主体](#create-service-principal)步骤的输出中获取：
    1. AadClientID 
    1. AadSecret
    1. AadTenantId
1. 现在，让我们尝试使用下面的命令模板登录：
    
    ```
    az login --service-principal --username "< AadClientID>" --password " <AadSecret>" --tenant "<AadTenantId>"
    ```
3. 现在，让我们通过尝试创建资源组并确保操作失败，来确定登录仅限于具有“LVAEdge User”角色的服务主体。 在 Cloud Shell 中运行以下命令：

    ```
    az group create --location "china east 2" --name "testresourcegroup"
    ```

    此命令应会失败，并将如下所示：
    
    ```
    The client '<AadClientId>' with object id '<AadClientId>' does not have authorization to perform action 'Microsoft.Resources/subscriptions/resourcegroups/write' over scope '/subscriptions/<yourSubscriptionId>/resourcegroups/testresourcegroup' or the scope is invalid. If access was recently granted, please refresh your credentials.
    ```

## <a name="next-steps"></a>后续步骤  

请记下本文中的以下值。 你需要使用这些值配置 IoT Edge 模块上的实时视频分析的孪生属性。有关信息，请参阅[模块孪生 JSON 模式](module-twin-configuration-schema.md)。

| 本文中的变量|IoT Edge 上的实时视频分析的孪生属性名称|
|---|---|
|AadSecret |    aadServicePrincipalPassword|
|AadTenantId |  aadTenantId|
|AadClientId    |aadServicePrincipalAppId|
