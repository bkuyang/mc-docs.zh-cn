---
title: 使用 Azure CLI 和模板部署资源
description: 使用 Azure 资源管理器和 Azure CLI 将资源部署到 Azure。 资源在 Resource Manager 模板中定义。
ms.topic: conceptual
origin.date: 06/04/2020
ms.date: 06/22/2020
ms.author: v-yeche
ms.openlocfilehash: 79e7417343931abd72735d4957abf55d9bc18858
ms.sourcegitcommit: 48b5ae0164f278f2fff626ee60db86802837b0b4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2020
ms.locfileid: "85098740"
---
# <a name="deploy-resources-with-arm-templates-and-azure-cli"></a>通过 ARM 模板和 Azure CLI 来部署资源

本文介绍了如何将 Azure CLI 与 Azure 资源管理器 (ARM) 模板配合使用，以便将资源部署到 Azure。 如果不熟悉部署和管理 Azure 解决方案的概念，请参阅[模版部署概述](overview.md)。

部署命令在 Azure CLI 版本 2.2.0 中已更改。 本文中的示例需要 Azure CLI 2.2.0 或更高版本。

[!INCLUDE [sample-cli-install](../../../includes/sample-cli-install.md)]

<!-- Not Available on Cloud Shell -->

## <a name="deployment-scope"></a>部署范围

可将部署目标设定为资源组、订阅、管理组或租户。 大多数情况下，我们会将以资源组指定为部署目标。 若要在更大范围内应用策略和角色分配，请使用订阅、管理组或租户部署。 部署到订阅时，可以创建资源组并将资源部署到该资源组。

根据部署范围使用不同的命令。

* 若要部署到资源组****，请使用 [az deployment group create](https://docs.microsoft.com/cli/azure/deployment/group?view=azure-cli-latest#az-deployment-group-create)：

  ```azurecli
  az deployment group create --resource-group <resource-group-name> --template-file <path-to-template>
  ```

* 若要部署到订阅****，请使用 [az deployment sub create](https://docs.azure.cn/cli/deployment/sub?view=azure-cli-latest#az-deployment-sub-create)：

  ```azurecli
  az deployment sub create --location <location> --template-file <path-to-template>
  ```

有关订阅级部署的详细信息，请参阅[在订阅级别创建资源组和资源](deploy-to-subscription.md)。

* 若要部署到管理组****，请使用 [az deployment mg create](https://docs.microsoft.com/cli/deployment/mg?view=azure-cli-latest#az-deployment-mg-create)：

  ```azurecli
  az deployment mg create --location <location> --template-file <path-to-template>
  ```

  有关管理组级部署的详细信息，请参阅[在管理组级别创建资源](deploy-to-management-group.md)。

* 若要部署到租户****，请使用 [az deployment tenant create](https://docs.microsoft.com/cli/azure/deployment/tenant?view=azure-cli-latest#az-deployment-tenant-create)：

  ```azurecli
  az deployment tenant create --location <location> --template-file <path-to-template>
  ```

  有关租户级别部署的详细信息，请参阅[在租户级别创建资源](deploy-to-tenant.md)。

本文中的示例使用资源组部署。

## <a name="deploy-local-template"></a>部署本地模板

将资源部署到 Azure 时，执行以下操作：

1. 登录到 Azure 帐户
2. 创建用作已部署资源的容器的资源组。 资源组名称只能包含字母数字字符、句点、下划线、连字符和括号。 它最多可以包含 90 个字符。 它不能以句点结尾。
3. 将定义了要创建的资源的模板部署到资源组

模板可以包括可用于自定义部署的参数。 例如，可以提供为特定环境（如开发环境、测试环境和生产环境）定制的值。 示例模板定义了存储帐户 SKU 的参数。

[!INCLUDE [azure-cli-2-azurechinacloud-environment-parameter](../../../includes/azure-cli-2-azurechinacloud-environment-parameter.md)]

以下示例将创建一个资源组，并从本地计算机部署模板：

```azurecli
az group create --name ExampleGroup --location "China North"
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters storageAccountType=Standard_GRS
```

部署可能需要几分钟才能完成。 完成之后，会看到一条包含以下结果的消息：

```output
"provisioningState": "Succeeded",
```

## <a name="deploy-remote-template"></a>部署远程模板

你可能更愿意将 ARM 模板存储在外部位置，而不是存储在本地计算机上。 可以将模板存储在源控件存储库（例如 GitHub）中。 另外，还可以将其存储在 Azure 存储帐户中，以便在组织中共享访问。

若要部署外部模板，请使用 **template-uri** 参数。 使用示例中的 URI 从 GitHub 部署示例模板。

```azurecli
az group create --name ExampleGroup --location "China North"
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json" \
  --parameters storageAccountType=Standard_GRS
```

前面的示例要求模板的 URI 可公开访问，它适用于大多数情况，因为模板应该不会包含敏感数据。 如果需要指定敏感数据（如管理员密码），请以安全参数的形式传递该值。 但是，如果不希望模板可公开访问，可以通过将其存储在专用存储容器中来保护它。 若要了解如何部署需要共享访问签名 (SAS) 令牌的模板，请参阅[部署具有 SAS 令牌的专用模板](secure-template-with-sas-token.md)。

## <a name="preview-changes"></a>预览更改

在部署模板之前，可以预览模板将对环境做出的更改。 使用[模拟操作](template-deploy-what-if.md)验证模板是否会进行你所期望的更改。 模拟操作还验证模板是否有错误。


<!--MOONCAKE: Not Available on Cloud Shell and corresponding code -->
<!--Not Available on [!INCLUDE [resource-manager-cloud-shell-deploy.md](../../includes/resource-manager-cloud-shell-deploy.md)]-->
在本地 Shell 中使用以下命令
```azurecli
az group create --name examplegroup --location "China East"
az deployment group create --resource-group examplegroup \
  --template-uri <copied URL> \
  --parameters storageAccountType=Standard_GRS
```

## <a name="parameters"></a>parameters

若要传递参数值，可以使用内联参数或参数文件。

### <a name="inline-parameters"></a>内联参数。

若要传递内联参数，请在 `parameters` 中提供值。 例如，若要在 Bash shell 中将字符串和数组传递给模板，请使用：

```azurecli
az deployment group create \
  --resource-group testgroup \
  --template-file demotemplate.json \
  --parameters exampleString='inline string' exampleArray='("value1", "value2")'
```

如果要将 Azure CLI 与 Windows 命令提示符 (CMD) 或 PowerShell 配合使用，请以以下格式传递数组：`exampleArray="['value1','value2']"`。

还可以获取文件的内容并将该内容作为内联参数提供。

```azurecli
az deployment group create \
  --resource-group testgroup \
  --template-file demotemplate.json \
  --parameters exampleString=@stringContent.txt exampleArray=@arrayContent.json
```

当需要提供配置值时，从文件中获取参数值非常有用。 例如，可以[为 Linux 虚拟机提供 cloud-init 值](../../virtual-machines/linux/using-cloud-init.md)。

arrayContent.json 格式为：

```json
[
    "value1",
    "value2"
]
```

### <a name="parameter-files"></a>参数文件

你可能会发现，与在脚本中以内联值的形式传递参数相比，使用包含参数值的 JSON 文件更为容易。 参数文件必须是本地文件。 Azure CLI 不支持外部参数文件。

有关参数文件的详细信息，请参阅[创建资源管理器参数文件](parameter-files.md)。

若要传递本地参数文件，请使用 `@` 指定名为 storage.parameters.json 的本地文件。

```azurecli
az deployment group create \
  --name ExampleDeployment \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters @storage.parameters.json
```

## <a name="handle-extended-json-format"></a>处理扩展 JSON 格式

若要使用 Azure CLI 2.3.0 或更低版本部署包含多行字符串或注释的模板，必须使用 `--handle-extended-json-format` 开关。  例如：

```json
{
  "type": "Microsoft.Compute/virtualMachines",
  "apiVersion": "2018-10-01",
  "name": "[variables('vmName')]", // to customize name, change it in variables
  "location": "[
    parameters('location')
    ]", //defaults to resource group location
  /*
    storage account and network interface
    must be deployed first
  */
  "dependsOn": [
    "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
    "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
  ],
```

## <a name="test-a-template-deployment"></a>测试模板部署

若要测试模板和参数值而不实际部署任何资源，请使用 [az group deployment validate](https://docs.azure.cn/cli/group/deployment?view=azure-cli-latest#az-group-deployment-validate)。

```azurecli
az group deployment validate \
  --resource-group ExampleGroup \
  --template-file storage.json \
  --parameters @storage.parameters.json
```

如果未检测到错误，则该命令将返回有关测试部署的信息。 需要特别注意的是，**error** 值为 null。

```output
{
  "error": null,
  "properties": {
      ...
```

如果检测到错误，则该命令将返回一条错误消息。 例如，如果为存储帐户 SKU 传递不正确的值，将返回以下错误：

```output
{
  "error": {
    "code": "InvalidTemplate",
    "details": null,
    "message": "Deployment template validation failed: 'The provided value 'badSKU' for the template parameter
      'storageAccountType' at line '13' and column '20' is not valid. The parameter value is not part of the allowed
      value(s): 'Standard_LRS,Standard_ZRS,Standard_GRS,Standard_RAGRS,Premium_LRS'.'.",
    "target": null
  },
  "properties": null
}
```

如果模板有语法错误，该命令将返回一个错误，指示它无法分析该模板。 该消息会指出分析错误的行号和位置。

```output
{
  "error": {
    "code": "InvalidTemplate",
    "details": null,
    "message": "Deployment template parse failed: 'After parsing a value an unexpected character was encountered:
      \". Path 'variables', line 31, position 3.'.",
    "target": null
  },
  "properties": null
}
```

## <a name="next-steps"></a>后续步骤

- 若要在出错时回退到成功的部署，请参阅[出错时回退到成功的部署](rollback-on-error.md)。
- 若要指定如何处理存在于资源组中但未在模板中定义的资源，请参阅 [Azure 资源管理器部署模式](deployment-modes.md)。
- 若要了解如何在模板中定义参数，请参阅[了解 ARM 模板的结构和语法](template-syntax.md)。
- 有关解决常见部署错误的提示，请参阅[排查使用 Azure Resource Manager 时的常见 Azure 部署错误](common-deployment-errors.md)。
- 有关部署需要 SAS 令牌的模板的信息，请参阅[使用 SAS 令牌部署专用模板](secure-template-with-sas-token.md)。

<!-- Update_Description: update meta properties, wording update, update link -->
