---
title: 快速入门 - 创建异地复制的注册表 - 资源管理器模板
description: 在本快速入门中，你将使用 Azure 资源管理器模板快速部署在独立 Azure 容器实例中运行的容器化 Web 应用。
services: azure-resource-manager
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
origin.date: 04/30/2020
ms.date: 07/27/2020
ms.testscope: no
ms.testdate: 06/08/2020
ms.author: v-yeche
ms.openlocfilehash: 4a9f78df1a58cdb00294521ab66e149736ae6d50
ms.sourcegitcommit: 5726d3b2e694f1f94f9f7d965676c67beb6ed07c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/21/2020
ms.locfileid: "86863181"
---
<!--Verified successfully-->
# <a name="quickstart-deploy-a-container-instance-in-azure-using-a-resource-manager-template"></a>快速入门：使用资源管理器模板在 Azure 中部署容器实例

使用 Azure 容器实例在 Azure 中快速方便地运行无服务器 Docker 容器。 当你不需要像 AzureKubernetes 服务这样的完整容器业务流程平台时，可以按需将应用程序部署到容器实例。 在本快速入门中，你将使用 Azure 资源管理器模板部署一个独立的 Docker 容器，并使其 Web 应用程序可通过公共 IP 地址使用。

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

如果没有 Azure 订阅，请在开始之前创建一个[免费](https://www.azure.cn/pricing/1rmb-trial/)帐户。

## <a name="prerequisites"></a>先决条件

无。

## <a name="create-a-container-instance"></a>创建容器实例

### <a name="review-the-template"></a>查看模板

本快速入门中使用的模板来自 [Azure 快速入门模板](https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-linuxcontainer-public-ip/)。

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "name": {
      "type": "string",
      "metadata": {
        "description": "Name for the container group"
      },
      "defaultValue": "acilinuxpublicipcontainergroup"
    },
    "image": {
      "type": "string",
      "metadata": {
        "description": "Container image to deploy. Should be of the form repoName/imagename:tag for images stored in public Docker Hub, or a fully qualified URI for other registries. Images from private registries require additional registry credentials."
      },
      "defaultValue": "mcr.microsoft.com/azuredocs/aci-helloworld"
    },
    "port": {
      "type": "string",
      "metadata": {
        "description": "Port to open on the container and the public IP address."
      },
      "defaultValue": "80"
    },
    "cpuCores": {
      "type": "string",
      "metadata": {
        "description": "The number of CPU cores to allocate to the container."
      },
      "defaultValue": "1.0"
    },
    "memoryInGb": {
      "type": "string",
      "metadata": {
        "description": "The amount of memory to allocate to the container in gigabytes."
      },
      "defaultValue": "1.5"
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    },
    "restartPolicy": {
      "type": "string",
      "defaultValue": "always",
      "metadata": {
        "description": "The behavior of Azure runtime if container has stopped."
      },
      "allowedValues": [
        "never",
        "always",
        "onfailure"
      ]
    }
  },
  "variables": {},
  "resources": [
    {
      "name": "[parameters('name')]",
      "type": "Microsoft.ContainerInstance/containerGroups",
      "apiVersion": "2019-12-01",
      "location": "[parameters('location')]",
      "properties": {
        "containers": [
          {
            "name": "[parameters('name')]",
            "properties": {
              "image": "[parameters('image')]",
              "ports": [
                {
                  "port": "[parameters('port')]"
                }
              ],
              "resources": {
                "requests": {
                  "cpu": "[parameters('cpuCores')]",
                  "memoryInGb": "[parameters('memoryInGb')]"
                }
              }
            }
          }
        ],
        "osType": "Linux",
        "restartPolicy": "[parameters('restartPolicy')]",
        "ipAddress": {
          "type": "Public",
          "ports": [
            {
              "protocol": "Tcp",
              "port": "[parameters('port')]"
            }
          ]
        }
      }
    }
  ],
  "outputs": {
    "containerIPv4Address": {
      "type": "string",
      "value": "[reference(resourceId('Microsoft.ContainerInstance/containerGroups/', parameters('name'))).ipAddress.ip]"
    }
  }
}
```

模板中定义了以下资源：

* **Microsoft.ContainerInstance/containerGroups**：创建 Azure 容器组。 此模板定义一个组，其中包含单个容器实例。
    
    <!--Not Available on [Microsoft.ContainerInstance/containerGroups](https://docs.microsoft.com/azure/templates/microsoft.containerinstance/containergroups)-->
    
可以在[快速入门模板库](https://github.com/Azure/azure-quickstart-templates/?resourceType=Microsoft.Containerinstance&pageNumber=1&sort=Popular)中找到更多 Azure 容器实例模板示例。

### <a name="deploy-the-template"></a>部署模板

1. 选择下图登录到 Azure 并打开一个模板。 该模板将在另一位置创建注册表和副本。

    [![部署到 Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.cn/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-aci-linuxcontainer-public-ip%2Fazuredeploy.json)

2. 选择或输入以下值。

    * 订阅：选择一个 Azure 订阅。
    * **资源组**：选择“新建”，为资源组输入一个独一无二的名称，然后选择“确定”。 
    * **位置**：选择资源组的位置。 示例：中国北部 2。
    * **名称**：接受为实例生成的名称，或者输入一个名称。
    * **映像**：接受默认映像名称。 此示例 Linux 映像打包了一个用 Node.js 编写的小型 Web 应用，该应用提供静态 HTML 页面。 

    对于剩余的属性，请接受默认值。

    查看条款和条件。 如果你同意，请选择“我同意上述条款和条件”。

    ![模板属性](media/container-instances-quickstart-template/template-properties.png)

3. 成功创建实例后，你会收到通知：

    ![门户通知](media/container-instances-quickstart-template/deployment-notification.png)

使用 Azure 门户部署模板。 除了 Azure 门户之外，还可以使用 Azure PowerShell、Azure CLI 和 REST API。 若要了解其他部署方法，请参阅[部署模板](../azure-resource-manager/templates/deploy-cli.md)。

## <a name="review-deployed-resources"></a>查看已部署的资源

使用 Azure 门户或诸如 [Azure CLI](container-instances-quickstart.md) 之类的工具来查看容器实例的属性。

1. 在门户中，搜索“容器实例”，然后选择你创建的容器实例。

1. 在“概览”页上，记下实例的“状态”及其“IP 地址” 。

    ![实例概览](media/container-instances-quickstart-template/aci-overview.png)

2. 在其状态为“正在运行”后，在浏览器中导航到 IP 地址。 

    ![在浏览器中显示的使用 Azure 容器实例部署的应用](media/container-instances-quickstart-template/view-application-running-in-an-azure-container-instance.png)

### <a name="view-container-logs"></a>查看容器日志

当排查容器或其运行的应用程序的问题时，查看容器实例的日志非常有用。

若要查看容器的日志，请在“设置”下选择“容器” > “日志”。  应当会看到在浏览器中查看应用程序时生成的 HTTP GET 请求。

![Azure 门户中的容器日志](media/container-instances-quickstart-template/aci-logs.png)

## <a name="clean-up-resources"></a>清理资源

使用完容器后，在容器实例的“概览”页上选择“删除”。  出现提示时，确认删除。

## <a name="next-steps"></a>后续步骤

在本快速入门中，你已基于公共 Azure 映像创建了一个 Azure 容器实例。 若要基于专用 Azure 容器注册表生成容器映像并部署它，请继续学习 Azure 容器实例教程。

> [!div class="nextstepaction"]
> [教程：创建要部署到 Azure 容器实例的容器映像](./container-instances-tutorial-prepare-app.md)

有关引导你完成模板创建过程的分步教程，请参阅：

> [!div class="nextstepaction"]
> [教程：创建和部署你的第一个 Azure 资源管理器模板](../azure-resource-manager/templates/template-tutorial-create-first-template.md)

<!-- Update_Description: update meta properties, wording update, update link -->