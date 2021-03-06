---
title: 快速入门 - 创建异地复制的注册表 - 资源管理器模板
description: 了解如何使用 Azure 资源管理器模板创建异地复制 Azure 容器注册表。
services: azure-resource-manager
author: rockboyfor
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
origin.date: 05/26/2020
ms.date: 07/27/2020
ms.testscope: yes
ms.testdate: 08/03/2020
ms.author: v-yeche
ms.openlocfilehash: c13959018ec491e0ab5b638a1f7958f4fb8319c8
ms.sourcegitcommit: 2b78a930265d5f0335a55f5d857643d265a0f3ba
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/28/2020
ms.locfileid: "87244707"
---
<!--Verified successfully-->
# <a name="quickstart-create-a-geo-replicated-container-registry-by-using-a-resource-manager-template"></a>快速入门：使用资源管理器模板创建异地复制容器注册表

本快速入门介绍如何使用 Azure 资源管理器模板创建 Azure 容器注册表实例。 该模板会设置一个[异地复制](container-registry-geo-replication.md)注册表，使其自动在多个 Azure 区域之间同步注册表内容。 借助异地复制，可以从区域部署对映像进行近网络访问，同时提供单一管理体验。 这是[高级](container-registry-skus.md)注册表服务层级的一项功能。 

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

如果没有 Azure 订阅，请在开始之前创建一个[免费](https://www.azure.cn/pricing/1rmb-trial/)帐户。

## <a name="prerequisites"></a>先决条件

无。

## <a name="create-a-geo-replicated-registry"></a>创建异地复制注册表

### <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速启动模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-container-registry-geo-replication/)。 该模板会设置注册表和其他区域副本。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "acrName": {
      "type": "string",
      "defaultValue": "[concat('acr', uniqueString(resourceGroup().id))]",
      "minLength": 5,
      "maxLength": 50,
      "metadata": {
        "description": "Globally unique name of your Azure Container Registry"
      }
    },
    "acrAdminUserEnabled": {
      "type": "bool",
      "defaultValue": false,
      "metadata": {
        "description": "Enable admin user that has push / pull permission to the registry."
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for registry home replica."
      }
    },
    "acrSku": {
      "type": "string",
      "metadata": {
        "description": "Tier of your Azure Container Registry. Geo-replication requires Premium SKU."
      },
      "defaultValue": "Premium",
      "allowedValues": [
        "Premium"
      ]
    },
    "acrReplicaLocation": {
      "type": "string",
      "metadata": {
        "description": "Short name for registry replica location."
      }
    }
  },
  "resources": [
    {
      "name": "[parameters('acrName')]",
      "type": "Microsoft.ContainerRegistry/registries",
      "apiVersion": "2019-05-01",
      "location": "[parameters('location')]",
      "comments": "Container registry for storing docker images",
      "tags": {
        "displayName": "Container Registry",
        "container.registry": "[parameters('acrName')]"
      },
      "sku": {
        "name": "[parameters('acrSku')]",
        "tier": "[parameters('acrSku')]"
      },
      "properties": {
        "adminUserEnabled": "[parameters('acrAdminUserEnabled')]"
      }
    },
    {
      "name": "[concat(parameters('acrName'), '/', parameters('acrReplicaLocation'))]",
      "type": "Microsoft.ContainerRegistry/registries/replications",
      "apiVersion": "2019-05-01",
      "location": "[parameters('acrReplicaLocation')]",
      "properties": {},
      "dependsOn": [
        "[concat('Microsoft.ContainerRegistry/registries/', parameters('acrName'))]"
      ]
    }
  ],
  "outputs": {
    "acrLoginServer": {
      "value": "[reference(resourceId('Microsoft.ContainerRegistry/registries',parameters('acrName')),'2019-05-01').loginServer]",
      "type": "string"
    }
  }
}
```

该模板中定义了以下资源：

* **Microsoft.ContainerRegistry/registries**：创建 Azure 容器注册表
    
    <!--Not Available on (https://docs.microsoft.com/azure/templates/microsoft.containerregistry/registries)-->
    
* **Microsoft.ContainerRegistry/registries/replications**：创建容器注册表副本

    <!--Not Available on (https://docs.microsoft.com/azure/templates/microsoft.containerregistry/registries/replications)-->

可以在[快速入门模板库](https://github.com/Azure/azure-quickstart-templates/?resourceType=Microsoft.Containerregistry&pageNumber=1&sort=Popular)中找到更多 Azure 容器注册表模板示例。

### <a name="deploy-the-template"></a>部署模板

 1. 选择下图登录到 Azure 并打开一个模板。

    [![部署到 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-container-registry-geo-replication%2Fazuredeploy.json)

 2. 选择或输入以下值。

    * 订阅：选择一个 Azure 订阅。
    * **资源组**：选择“新建”，为资源组输入一个独一无二的名称，然后选择“确定”。
    * **位置**：选择资源组的位置。 示例：**中国北部**。
    * **Acr 名称**：接受为注册表生成的名称，或者输入一个名称。 它必须全局唯一。
    * **位置**：接受为注册表的主副本生成的位置，或输入一个位置，例如“中国北部”。 
    * **Acr 副本位置**：使用区域的短名称输入注册表副本的位置。 该位置必须与主注册表的位置不同。 示例：chinanorth。
    * **我同意上述条款和条件**：选中。

        :::image type="content" source="media/container-registry-get-started-geo-replication-template/template-properties.png" alt-text="模板属性":::

 3. 如果接受条款和条件，请选择“购买”。 成功创建注册表后，你会收到通知：

    :::image type="content" source="media/container-registry-get-started-geo-replication-template/deployment-notification.png" alt-text="门户通知":::

 使用 Azure 门户部署模板。 除了 Azure 门户之外，还可以使用 Azure PowerShell、Azure CLI 和 REST API。 若要了解其他部署方法，请参阅[部署模板](../azure-resource-manager/templates/deploy-cli.md)。

## <a name="review-deployed-resources"></a>查看已部署的资源

使用 Azure 门户或诸如 Azure CLI 之类的工具来查看容器注册表的属性。

1. 在门户中，搜索“容器注册表”，然后选择所创建的容器注册表。

1. 在“概述”页上，记下注册表的“登录服务器” 。 使用 Docker 标记映像并将其推送到注册表时，请使用此 URI。 有关信息，请参阅[使用 Docker CLI 推送第一个映像](container-registry-get-started-docker-cli.md)。

    :::image type="content" source="media/container-registry-get-started-geo-replication-template/registry-overview.png" alt-text="注册表概述":::

1. 在“复制”页上，确认主副本和通过该模板添加的副本的位置。 如果需要，可在此页上添加更多副本。

    :::image type="content" source="media/container-registry-get-started-geo-replication-template/registry-replications.png" alt-text="注册表复制":::

## <a name="clean-up-resources"></a>清理资源

如果不再需要资源组、注册表和注册表副本，请将其删除。 为此，请访问 Azure 门户，选择包含注册表的资源组，然后选择“删除资源组”。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你使用资源管理器模板创建了 Azure 容器注册表，并在其他位置配置了注册表副本。 请继续阅读 Azure 容器注册表教程，以更深入地了解 ACR。

> [!div class="nextstepaction"]
> [Azure 容器注册表教程](container-registry-tutorial-prepare-registry.md)

有关引导你完成模板创建过程的分步教程，请参阅：

> [!div class="nextstepaction"]
> [教程：创建和部署你的第一个 Azure 资源管理器模板](../azure-resource-manager/templates/template-tutorial-create-first-template.md)

<!-- Update_Description: new article about container registry get started geo replication template -->
<!--NEW.date: 07/27/2020-->