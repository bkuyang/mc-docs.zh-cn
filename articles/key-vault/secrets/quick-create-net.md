---
title: 快速入门 - 适用于 .NET 的 Azure Key Vault 客户端库 (v4)
description: 了解如何使用 .NET 客户端库 (v4) 在 Azure 密钥保管库中创建、检索和删除机密
author: msmbaldwin
ms.author: v-tawe
origin.date: 03/21/2020
ms.date: 07/28/2020
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.openlocfilehash: 4ddb1a1bcdb58885c564d3ae708bb9ca1360e8c7
ms.sourcegitcommit: 0e778acf5aa5eb63ab233e07e7aecce3a9a5e6d4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/28/2020
ms.locfileid: "87296472"
---
# <a name="quickstart-azure-key-vault-client-library-for-net-sdk-v4"></a>快速入门：适用于 .NET 的 Azure Key Vault 客户端库 (SDK v4)

适用于 .NET 的 Azure Key Vault 客户端库入门。 请遵循以下步骤安装包并试用基本任务的示例代码。

Azure 密钥保管库可帮助保护云应用程序和服务使用的加密密钥和机密。 使用适用于 .NET 的 Key Vault 客户端库可以：

- 提高安全性以及控制密钥和密码。
- 在几分钟内创建并导入加密密钥。
- 通过云扩展和全局冗余减少延迟。
- 简化和自动化与 TLS/SSL 证书相关的任务。

<!-- - Use FIPS 140-2 Level 2 validated HSMs. -->

[API 参考文档](https://docs.microsoft.com/dotnet/api/azure.security.keyvault.secrets?view=azure-dotnet) | [库源代码](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault) | [包 (NuGet)](https://www.nuget.org/packages/Azure.Security.KeyVault.Secrets/)

## <a name="prerequisites"></a>先决条件

* Azure 订阅 - [创建试用订阅](https://www.azure.cn/pricing/1rmb-trial/)。
* [.NET Core 3.1 SDK 或更高版本](https://dotnet.microsoft.com/download/dotnet-core/3.1)。
* [Azure CLI](/cli/install-azure-cli?view=azure-cli-latest) 或 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/overview)

本快速入门假设在 Windows 终端（例如 [PowerShell Core](https://docs.microsoft.com/powershell/scripting/install/installing-powershell-core-on-windows?view=powershell-6) 或 [Windows PowerShell](https://docs.microsoft.com/powershell/scripting/install/installing-windows-powershell?view=powershell-6)）中运行 `dotnet`、[Azure CLI](/cli/install-azure-cli?view=azure-cli-latest) 和 Windows 命令。

## <a name="setting-up"></a>设置

### <a name="create-new-net-console-app"></a>创建新的 .NET 控制台应用

在控制台窗口中，使用 `dotnet new` 命令新建名为 `key-vault-console-app` 的 .NET 控制台应用。

```console
dotnet new console -n key-vault-console-app
```

将目录更改为新创建的应用文件夹。 可使用以下代码生成应用程序：

```console
dotnet build
```

生成输出不应包含警告或错误。

```console
Build succeeded.
 0 Warning(s)
 0 Error(s)
```

### <a name="install-the-package"></a>安装包

在控制台窗口中，安装适用于 .NET 的 Azure Key Vault 客户端库：

```console
dotnet add package Azure.Security.KeyVault.Secrets
```

本快速入门还需要安装以下包：

```console
dotnet add package Azure.Identity
```

### <a name="create-a-resource-group-and-key-vault"></a>创建资源组和 Key Vault

本快速入门使用预先创建的 Azure Key Vault。 可以遵循 [Azure CLI 快速入门](quick-create-cli.md)、[Azure PowerShell 快速入门](quick-create-powershell.md)或 [Azure 门户快速入门](quick-create-portal.md)中的步骤创建 Key Vault。 或者，只需运行以下 Azure CLI 命令。

> [!Important]
> 每个密钥保管库必须具有唯一的名称。 在以下示例中，将 <your-unique-keyvault-name> 替换为密钥保管库的名称。

```azurecli
az group create --name "myResourceGroup" -l "chinanorth"

az keyvault create --name <your-unique-keyvault-name> -g "myResourceGroup"
```

```azurepowershell
New-AzResourceGroup -Name myResourceGroup -Location chinanorth

New-AzKeyVault -Name <your-unique-keyvault-name> -ResourceGroupName myResourceGroup -Location chinanorth
```

### <a name="create-a-service-principal"></a>创建服务主体

对基于云的 .NET 应用程序进行身份验证的最简单方法是使用托管标识；有关详细信息，请参阅[使用应用服务托管标识访问 Azure Key Vault](../general/managed-identity.md)。 

不过，为了简单起见，本快速入门创建了一个需要使用服务主体和访问控制策略的 .NET 控制台应用程序。 服务主体要求使用格式为“http://&lt;my-unique-service-principal-name&gt;”的唯一名称。

使用 Azure CLI [az ad sp create-for-rbac](/cli/ad/sp?view=azure-cli-latest#az-ad-sp-create-for-rbac) 命令创建服务主体：

```azurecli
az ad sp create-for-rbac -n "http://&lt;my-unique-service-principal-name&gt;" --sdk-auth
```

此操作将返回一系列键/值对。 

```console
{
  "clientId": "7da18cae-779c-41fc-992e-0527854c6583",
  "clientSecret": "b421b443-1669-4cd7-b5b1-394d5c945002",
  "subscriptionId": "443e30da-feca-47c4-b68f-1636b75e16b3",
  "tenantId": "35ad10f1-7799-4766-9acf-f2d946161b77",
  "activeDirectoryEndpointUrl": "https://login.chinacloudapi.cn",
  "resourceManagerEndpointUrl": "https://management.chinacloudapi.cn/",
  "sqlManagementEndpointUrl": "https://management.core.chinacloudapi.cn:8443/",
  "galleryEndpointUrl": "https://gallery.chinacloudapi.cn/",
  "managementEndpointUrl": "https://management.core.chinacloudapi.cn/"
}
```

使用 Azure PowerShell 的 [New-AzADServicePrincipal](https://docs.microsoft.com/powershell/module/az.resources/new-azadserviceprincipal) 命令创建服务主体：

```azurepowershell
# Create a new service principal
$spn = New-AzADServicePrincipal -DisplayName "http://&lt;my-unique-service-principal-name&gt;"

# Get the tenant ID and subscription ID of the service principal
$tenantId = (Get-AzContext).Tenant.Id
$subscriptionId = (Get-AzContext).Subscription.Id

# Get the client ID
$clientId = $spn.ApplicationId

# Get the client Secret
$bstr = [System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($spn.Secret)
$clientSecret = [System.Runtime.InteropServices.Marshal]::PtrToStringAuto($bstr)
```

若要更详细地了解如何使用 Azure PowerShell 来创建服务主体，请参阅[使用 Azure PowerShell 创建 Azure 服务主体](https://docs.microsoft.com/powershell/azure/create-azure-service-principal-azureps)。

请记下 clientId、clientSecret 和 tenantId，因为我们将在后面的步骤中使用它们。


#### <a name="give-the-service-principal-access-to-your-key-vault"></a>为服务主体授予对 Key Vault 的访问权限

通过将 clientId 传递给 [az keyvault set-policy](/cli/keyvault?view=azure-cli-latest#az-keyvault-set-policy) 命令，为密钥保管库创建授予服务主体权限的访问策略。 授予服务主体对密钥和机密的 get、list 和 set 权限。

```azurecli
az keyvault set-policy -n <your-unique-keyvault-name> --spn <clientId-of-your-service-principal> --secret-permissions list get set delete purge
```

```azurepowershell
Set-AzKeyVaultAccessPolicy -VaultName <your-unique-keyvault-name> -ServicePrincipalName <clientId-of-your-service-principal> -PermissionsToSecrets list,get,set,delete,purge
```

#### <a name="set-environmental-variables"></a>设置环境变量

应用程序中的 DefaultAzureCredential 方法依赖于三个环境变量：`AZURE_CLIENT_ID`、`AZURE_CLIENT_SECRET` 和 `AZURE_TENANT_ID`。 使用将这些变量设置为在上述[创建服务主体](#create-a-service-principal)步骤中记下的 clientId、clientSecret 和 tenantId 值。

还需要将密钥保管库名称另存为名为 `KEY_VAULT_NAME` 的环境变量；

```console
setx AZURE_CLIENT_ID <your-clientID>

setx AZURE_CLIENT_SECRET <your-clientSecret>

setx AZURE_TENANT_ID <your-tenantId>

setx KEY_VAULT_NAME <your-key-vault-name>
````

每次调用 `setx` 时，都应会收到响应“成功:已保存指定的值。”

```shell
AZURE_CLIENT_ID=<your-clientID>

AZURE_CLIENT_SECRET=<your-clientSecret>

AZURE_TENANT_ID=<your-tenantId>

KEY_VAULT_NAME=<your-key-vault-name>
```

## <a name="object-model"></a>对象模型

使用适用于 .NET 的 Azure Key Vault 客户端库可以管理密钥和相关的资产（例如证书和机密）。 以下代码示例演示如何创建客户端以及设置、检索和删除机密。

[https://github.com/Azure-Samples/key-vault-dotnet-core-quickstart/tree/master/akvdotnet](https://github.com/Azure-Samples/key-vault-dotnet-core-quickstart/tree/master/key-vault-console-app ) 中提供了整个控制台应用。

## <a name="code-examples"></a>代码示例

### <a name="add-directives"></a>添加指令

将以下指令添加到代码的顶部：

```csharp
using System;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;
```

### <a name="authenticate-and-create-a-client"></a>进行身份验证并创建客户端

向密钥保管库进行身份验证和创建密钥保管库客户端，依赖于上面[设置环境变量](#set-environmental-variables)步骤中的环境变量。 密钥保管库的名称将扩展为密钥保管库 URI，格式为“https://\<your-key-vault-name\>.vault.azure.cn”。 下面的代码使用[“DefaultAzureCredential()”](https://docs.microsoft.com/dotnet/api/azure.identity.defaultazurecredential?view=azure-dotnet)向密钥保管库进行身份验证，该密钥保管库会读取环境变量来检索访问令牌。 

```csharp
string keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
var kvUri = "https://" + keyVaultName + ".vault.azure.cn";
var credentialOption = new DefaultAzureCredentialOptions();
credentialOption.AuthorityHost = new Uri("https://login.partner.microsoftonline.cn");

var client = new SecretClient(new Uri(kvUri), new DefaultAzureCredential(credentialOption));
```

### <a name="save-a-secret"></a>保存机密

应用程序通过身份验证后，你可以使用 [client.SetSecret 方法](/dotnet/api/microsoft.azure.keyvault.keyvaultclientextensions.setsecretasync)将机密放入密钥保管库。此操作需要使用机密的名称，本示例中使用“mySecret”。  

```csharp
client.SetSecret(secretName, secretValue);
```

可以使用 [az keyvault secret show](/cli/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show) 命令来验证是否设置了机密：

```azurecli
az keyvault secret show --vault-name <your-unique-keyvault-name> --name mySecret
```

```azurepowershell
(Get-AzKeyVaultSecret -VaultName <your-unique-keyvault-name> -Name mySecret).SecretValueText
```

### <a name="retrieve-a-secret"></a>检索机密

现在，可以使用 [client.GetSecret 方法](/dotnet/api/microsoft.azure.keyvault.keyvaultclientextensions.getsecretasync)检索以前设置的值。

```csharp
KeyVaultSecret secret = client.GetSecret(secretName);
```

机密现已保存为 `secret.Value`。

### <a name="delete-a-secret"></a>删除机密

最后，使用 [client.DeleteSecret 方法](/dotnet/api/microsoft.azure.keyvault.keyvaultclientextensions.getsecretasync)从密钥保管库中删除机密。

```csharp
client.StartDeleteSecret(secretName);
```

可以使用 [az keyvault secret show](/cli/keyvault/secret?view=azure-cli-latest#az-keyvault-secret-show) 命令来验证是否已删除机密：

```azurecli
az keyvault secret show --vault-name <your-unique-keyvault-name> --name mySecret
```

```azurepowershell
(Get-AzKeyVaultSecret -VaultName <your-unique-keyvault-name> -Name mySecret).SecretValueText
```

## <a name="clean-up-resources"></a>清理资源

可以使用 Azure CLI 或 Azure PowerShell 来删除不再需要的 Key Vault 和相应的资源组。

### <a name="delete-a-key-vault"></a>删除 Key Vault
```azurecli
az keyvault delete --name <your-unique-keyvault-name>
```

```powershell
Remove-AzKeyVault -VaultName <your-unique-keyvault-name>
```

### <a name="purge-a-key-vault"></a>清除 Key Vault
```azurecli
az keyvault purge --location chinaeast --name <your-unique-keyvault-name>
```

```powershell
Remove-AzKeyVault -VaultName <your-unique-keyvault-name> -InRemovedState -Location chinaeast
```

### <a name="delete-a-resource-group"></a>删除资源组

```azurecli
az group delete -g "myResourceGroup"
```

```azurepowershell
Remove-AzResourceGroup -Name "myResourceGroup"
```

## <a name="sample-code"></a>示例代码

```csharp
using System;
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

namespace key_vault_console_app
{
    class Program
    {
        static void Main(string[] args)
        {
            string secretName = "mySecret";

            string keyVaultName = Environment.GetEnvironmentVariable("KEY_VAULT_NAME");
            var kvUri = "https://" + keyVaultName + ".vault.azure.cn";
            var credentialOption = new DefaultAzureCredentialOptions();
            credentialOption.AuthorityHost = new Uri("https://login.partner.microsoftonline.cn");

            var client = new SecretClient(new Uri(kvUri), new DefaultAzureCredential(credentialOption));

            Console.Write("Input the value of your secret > ");
            string secretValue = Console.ReadLine();

            Console.Write("Creating a secret in " + keyVaultName + " called '" + secretName + "' with the value '" + secretValue + "` ...");

            client.SetSecret(secretName, secretValue);

            Console.WriteLine(" done.");

            Console.WriteLine("Forgetting your secret.");
            secretValue = "";
            Console.WriteLine("Your secret is '" + secretValue + "'.");

            Console.WriteLine("Retrieving your secret from " + keyVaultName + ".");

            KeyVaultSecret secret = client.GetSecret(secretName);

            Console.WriteLine("Your secret is '" + secret.Value + "'.");

            Console.Write("Deleting your secret from " + keyVaultName + " ...");

            client.StartDeleteSecret(secretName);

            System.Threading.Thread.Sleep(5000);
            Console.WriteLine(" done.");

        }
    }
}
```


## <a name="next-steps"></a>后续步骤

在本快速入门中，你创建了一个 Key Vault、存储了一个机密，然后检索了该机密。 请查看 [GitHub 中的整个控制台应用](https://github.com/Azure-Samples/key-vault-dotnet-core-quickstart/tree/master/key-vault-console-app)。

若要详细了解 Key Vault 以及如何将其与应用程序集成，请继续阅读以下文章。

- 实现[使用 .NET 对 Azure Key Vault 进行服务到服务的身份验证](../general/service-to-service-authentication.md)
- 阅读 [Azure Key Vault 概述](../general/overview.md)
- 参阅 [Azure Key Vault 开发人员指南](../general/developers-guide.md)
- 查看 [Azure Key Vault 最佳做法](../general/best-practices.md)
