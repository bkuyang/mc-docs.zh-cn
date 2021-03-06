---
title: Azure 服务总线故障排除指南
description: 本文提供了 Azure 服务总线消息传送异常以及发生异常时建议采取的措施的列表。
ms.topic: article
origin.date: 06/23/2020
ms.date: 07/27/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
author: rockboyfor
ms.openlocfilehash: 60cafa3dc8a5678a7da2a027bcfb8fae9160a049
ms.sourcegitcommit: 091c672fa448b556f4c2c3979e006102d423e9d7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/24/2020
ms.locfileid: "87162261"
---
# <a name="troubleshooting-guide-for-azure-service-bus"></a>Azure 服务总线故障排除指南
本文提供的故障排除技巧和建议适用于你在使用 Azure 服务总线时可能会遇到的一些问题。 

## <a name="connectivity-certificate-or-timeout-issues"></a>连接性、证书或超时问题
以下步骤可帮助排查 *.servicebus.chinacloudapi.cn 下所有服务的连接性/证书/超时问题。 

- 浏览至 `https://<yournamespace>.servicebus.chinacloudapi.cn/` 或使用 [wget](https://www.gnu.org/software/wget/)。 它可用于检查你是否有 IP 筛选、虚拟网络或证书链问题（在使用 Java SDK 时最常见）。

    成功消息的示例：

    ```xml
    <feed xmlns="http://www.w3.org/2005/Atom"><title type="text">Publicly Listed Services</title><subtitle type="text">This is the list of publicly-listed services currently available.</subtitle><id>uuid:27fcd1e2-3a99-44b1-8f1e-3e92b52f0171;id=30</id><updated>2019-12-27T13:11:47Z</updated><generator>Service Bus 1.1</generator></feed>
    ```

    失败错误消息的示例：

    ```json
    <Error>
        <Code>400</Code>
        <Detail>
            Bad Request. To know more visit https://aka.ms/sbResourceMgrExceptions. . TrackingId:b786d4d1-cbaf-47a8-a3d1-be689cda2a98_G22, SystemTracker:NoSystemTracker, Timestamp:2019-12-27T13:12:40
        </Detail>
    </Error>
    ```
- 运行以下命令，检查防火墙是否阻止了任何端口。 所用的端口为 443 (HTTPS)、5671 (AMQP) 和 9354 (Net Messaging/SBMP)。 根据使用的库，还会使用其他端口。 下面是用于检查 5671 端口是否被阻止的示例命令。 

    ```powershell
    tnc <yournamespacename>.servicebus.chinacloudapi.cn -port 5671
    ```

    在 Linux 上：

    ```shell
    telnet <yournamespacename>.servicebus.chinacloudapi.cn 5671
    ```
- 存在间歇性连接问题时，请运行以下命令，检查是否有丢弃的数据包。 此命令会尝试通过服务每隔 1 秒建立 25 个不同的 TCP 连接。 然后，可以检查其中有多少成功/失败，还可以查看 TCP 连接延迟。 可以从[此处](https://docs.microsoft.com/sysinternals/downloads/psping)下载 `psping` 工具。

    ```shell
    .\psping.exe -n 25 -i 1 -q <yournamespace>.servicebus.chinacloudapi.cn:5671 -nobanner     
    ```
    如果使用的是其他工具（如 `tnc`、`ping` 等），则可以使用等效的命令。 
- 如果上述步骤没有帮助，请获取网络跟踪，并使用 [Wireshark](https://www.wireshark.org/) 之类的工具对其进行分析。 如果需要，请联系 [Azure 支持部门](https://support.microsoft.com/)。 

## <a name="issues-that-may-occur-with-service-upgradesrestarts"></a>服务升级/重启时可能出现的问题
后端服务升级和重启可能会对应用程序造成以下影响：

- 可能会暂时限制请求。
- 传入的消息/请求可能会减少。
- 日志文件可能包含错误消息。
- 应用程序可能会在几秒内断开与服务的连接。

如果应用程序代码使用 SDK，则重试策略已内置且处于活动状态。 应用程序会重新连接，此操作不会对应用程序/工作流产生重大影响。

## <a name="unauthorized-access-send-claims-are-required"></a>未授权访问：需要发送声明
尝试使用具有发送权限的用户分配的托管标识从本地计算机上的 Visual Studio 访问服务总线主题时，可能会出现此错误。

```bash
Service Bus Error: Unauthorized access. 'Send' claim\(s\) are required to perform this operation.
```

要解决此错误，请安装 [Microsoft.Azure.Services.AppAuthentication](https://www.nuget.org/packages/Microsoft.Azure.Services.AppAuthentication/) 库。  有关详细信息，请参阅[本地开发身份验证](..\key-vault\service-to-service-authentication.md#local-development-authentication)。 

要了解如何将权限分配给角色，请参阅[使用 Azure Active Directory 对托管标识进行身份验证，以便访问 Azure 服务总线资源](service-bus-managed-service-identity.md)。

## <a name="next-steps"></a>后续步骤
请参阅以下文章： 

- [Azure 资源管理器异常](service-bus-resource-manager-exceptions.md)。 这篇文章列出了使用 Azure 资源管理器（通过模板或直接调用）与 Azure 服务总线进行交互时生成的异常。
- [消息传送异常](service-bus-messaging-exceptions.md)。 这篇文章列出了 Azure 服务总线的 .NET Framework 生成的异常。

<!-- Update_Description: update meta properties, wording update, update link -->