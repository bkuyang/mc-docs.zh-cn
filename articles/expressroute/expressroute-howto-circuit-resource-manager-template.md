---
title: Azure ExpressRoute 模板：创建 ExpressRoute 线路
description: 创建、预配、删除和取消预配 ExpressRoute 线路。
services: expressroute
author: charwen
ms.service: expressroute
ms.topic: article
origin.date: 11/13/2019
ms.date: 06/08/2020
ms.author: v-yiso
ms.reviewer: ganesr
ms.openlocfilehash: 0711ef9c6e4260c44f155164b933d03b0a27e977
ms.sourcegitcommit: 0130a709d934d89db5cccb3b4997b9237b357803
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/29/2020
ms.locfileid: "84186595"
---
# <a name="create-an-expressroute-circuit-by-using-azure-resource-manager-template"></a>使用 Azure 资源管理器模板创建 ExpressRoute 线路

> [!div class="op_single_selector"]
> * [Azure 门户](expressroute-howto-circuit-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-circuit-arm.md)
> * [Azure CLI](howto-circuit-cli.md)
> * [Azure Resource Manager 模板](expressroute-howto-circuit-resource-manager-template.md)
> * [PowerShell（经典）](expressroute-howto-circuit-classic.md)
>

了解如何使用 Azure PowerShell 部署 Azure 资源管理器模板，以便创建 ExpressRoute 线路。 有关开发资源管理器模板的详细信息，请参阅[资源管理器文档](/azure-resource-manager/)和[模板参考](https://docs.microsoft.com/en-us/azure/templates/microsoft.network/expressroutecircuits)。

## <a name="before-you-begin"></a>准备阶段

* 在开始配置之前，请查看[先决条件](expressroute-prerequisites.md)和[工作流](expressroute-workflows.md)。
* 确保有权创建新的网络资源。 如果没有适当的权限，请与帐户管理员联系。
## <a name="create-and-provision-an-expressroute-circuit"></a><a name="create"></a>创建和预配 ExpressRoute 线路

[Azure 快速入门模板](https://azure.microsoft.com/resources/templates/)有许多资源管理器模板。 请使用某个[现有模板](https://azure.microsoft.com/resources/templates/101-expressroute-circuit-create/)来创建 ExpressRoute 线路。

<!--[!code-json[create-azure-expressroute-circuit](~/quickstart-templates/101-expressroute-circuit-create/azuredeploy.json)]-->

若要查看更多相关的模板，请在[此处](https://azure.microsoft.com/resources/templates/?term=expressroute)进行选择。

若要通过部署模板来创建 ExpressRoute 线路，请执行以下操作：

1. 按说明登录到 Azure。

    ```azurepowershell
    $circuitName = Read-Host -Prompt "Enter a circuit name"
    $location = Read-Host -Prompt "Enter the location (i.e. chinaeast)"
    $resourceGroupName = "${circuitName}rg"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-expressroute-circuit-create/azuredeploy.json"

    $serviceProviderName = "Shanghai Telecom Ethernet"
    $peeringLocation = "Shanghai"
    $bandwidthInMbps = 500
    $sku_tier = "Premium"
    $sku_family = "MeteredData"

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -circuitName $circuitName -serviceProviderName $serviceProviderName -peeringLocation $peeringLocation -bandwidthInMbps $bandwidthInMbps -sku_tier $sku_tier -sku_family $sku_family

    Write-Host "Press [ENTER] to continue ..."
    ```

   * **SKU 层**确定 ExpressRoute 线路是[本地](expressroute-faqs.md#expressroute-local)线路、标准线路还是[高级](expressroute-faqs.md#expressroute-premium)线路。 可以指定“本地”  、“标准”  或“高级”  。
   * **SKU 系列**确定计费类型。 可以指定“Metereddata”  以获取数据流量套餐，指定“Unlimiteddata”  以获取无限制流量套餐。 可以将计费类型从“Metereddata”更改为“Unlimiteddata”，但不能将类型从“Unlimiteddata”更改为“Metereddata”。     “本地”  线路仅为 Unlimiteddata  。
   * “对等互连位置”  是与 Microsoft 建立对等互连的实际位置。

     > [!IMPORTANT]
     > “对等互连位置”指明了与 Microsoft 建立对等互连的[实际位置](expressroute-locations.md)。 此位置与“Location”属性 **没有** 关系，后者指的是 Azure 网络资源提供商所在的地理位置。 尽管两者之间没有关系，但最好是选择地理上与线路对等互连位置靠近的网络资源提供商。

    资源组名称是追加了 **rg** 的服务总线命名空间名称。

2. 选择“复制”以复制 PowerShell 脚本。 
3. 右键单击 shell 控制台，然后选择“粘贴”  。

创建事件中心需要一些时间。

在本教程中，请使用 Azure PowerShell 来部署模板。 如需其他模板部署方法，请参阅：

* [使用 Azure 门户](../azure-resource-manager/templates/deploy-portal.md)。
* [使用 Azure CLI](../azure-resource-manager/templates/deploy-cli.md)。
* [使用 REST API](../azure-resource-manager/templates/deploy-rest.md)。

## <a name="deprovisioning-and-deleting-an-expressroute-circuit"></a><a name="delete"></a>取消设置和删除 ExpressRoute 线路

可以通过选择“删除”  图标来删除 ExpressRoute 线路。 请注意以下信息：

* 必须取消所有虚拟网络与 ExpressRoute 线路的链接。 如果此操作失败，请检查是否有虚拟网络链接到了该线路。
* 如果 ExpressRoute 线路服务提供商预配状态为“正在预配”  或“已预配”  ，则必须与服务提供商合作，在他们一端取消预配线路。 在服务提供商取消对线路的预配并通知我们之前，我们会继续保留资源并收费。
* 如果服务提供商已取消设置线路（服务提供商预配状态设置为“未预配”  ），可以删除线路。 这样就会停止对线路的计费。

可以通过运行以下 PowerShell 命令删除 ExpressRoute 线路：

```azurepowershell
$circuitName = Read-Host -Prompt "Enter the same circuit name that you used earlier"
$resourceGroupName = "${circuitName}rg"

Remove-AzExpressRouteCircuit -ResourceGroupName $resourceGroupName -Name $circuitName
```

## <a name="next-steps"></a>后续步骤

创建线路后，请继续执行以下后续步骤：

* [创建和修改 ExpressRoute 线路的路由](expressroute-howto-routing-portal-resource-manager.md)
* [将虚拟网络链接到 ExpressRoute 线路](expressroute-howto-linkvnet-arm.md)
