---
title: Azure 中继混合连接 - 节点中的 WebSocket
description: 为 Azure 中继混合连接 Websocket 编写 Node.js 控制台应用程序
ms.topic: conceptual
origin.date: 06/23/2020
ms.date: 07/27/2020
ms.testscope: yes
ms.testdate: 07/20/2020
ms.author: v-yeche
author: rockboyfor
ms.openlocfilehash: 5fbd22b4c6c1b38bb13542ae91e2c24fcdcb669c
ms.sourcegitcommit: 091c672fa448b556f4c2c3979e006102d423e9d7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/24/2020
ms.locfileid: "87162420"
---
# <a name="get-started-with-relay-hybrid-connections-websockets-in-nodejs"></a>开始在 Node.js 中使用中继混合连接 WebSocket

[!INCLUDE [relay-selector-hybrid-connections](../../includes/relay-selector-hybrid-connections.md)]

在本快速入门中，请创建 Node.js 发送者和接收者应用程序，以便在 Azure 中继中通过混合连接 WebSocket 发送和接收消息。 若要了解 Azure 中继的常规信息，请参阅 [Azure 中继](relay-what-is-it.md)。 

在本快速入门中，你将执行以下步骤： 

1. 使用 Azure 门户创建中继命名空间。
2. 使用 Azure 门户在该命名空间中创建混合连接。
3. 编写服务器（侦听器）控制台应用程序，用于接收消息。
4. 编写客户端（发送方）控制台应用程序，用于发送消息。
5. 运行应用程序。 

## <a name="prerequisites"></a>必备条件

- [Node.js](https://nodejs.org/en/)。
- Azure 订阅。 如果没有订阅，请在开始之前[创建一个试用帐户](https://www.azure.cn/pricing/1rmb-trial/)。

## <a name="create-a-namespace"></a>创建命名空间
[!INCLUDE [relay-create-namespace-portal](../../includes/relay-create-namespace-portal.md)]

## <a name="create-a-hybrid-connection"></a>创建混合连接
[!INCLUDE [relay-create-hybrid-connection-portal](../../includes/relay-create-hybrid-connection-portal.md)]

## <a name="create-a-server-application-listener"></a>创建服务器应用程序（侦听程序）
若要侦听和接收来自中继的消息，请编写 Node.js 控制台应用程序。

[!INCLUDE [relay-hybrid-connections-node-get-started-server](../../includes/relay-hybrid-connections-node-get-started-server.md)]

## <a name="create-a-client-application-sender"></a>创建客户端应用程序（发送程序）
若要将消息发送到中继，请编写 Node.js 控制台应用程序。

[!INCLUDE [relay-hybrid-connections-node-get-started-client](../../includes/relay-hybrid-connections-node-get-started-client.md)]

## <a name="run-the-applications"></a>运行应用程序

1. 运行服务器应用程序：在 Node.js 命令提示符处，键入 `node listener.js`。
2. 运行客户端应用程序：在 Node.js 命令提示符处键入 `node sender.js`，然后输入某些文本。
3. 确保服务器应用程序控制台输出了客户端应用程序中输入的文本。

    ![running-applications](./media/relay-hybrid-connections-node-get-started/running-applications.png)

祝贺你，现已使用 Node.js 创建端到端混合连接应用程序！

## <a name="next-steps"></a>后续步骤
在本快速入门中，你创建了 Node.js 客户端和服务器应用程序，此类程序使用 WebSocket 来发送和接收消息。 Azure 中继的混合连接功能也支持使用 HTTP 发送和接收消息。 若要了解如何将 HTTP 与 Azure 中继混合连接配合使用，请参阅 [Node.js HTTP 快速入门](relay-hybrid-connections-http-requests-node-get-started.md)。

在本快速入门中，你使用了 Node.js 来创建客户端和服务器应用程序。 若要了解如何使用 .NET Framework 编写客户端和服务器应用程序，请参阅 [.NET WebSocket 快速入门](relay-hybrid-connections-dotnet-get-started.md)或 [.NET HTTP 快速入门](relay-hybrid-connections-http-requests-dotnet-get-started.md)。

<!-- Update_Description: update meta properties, wording update, update link -->