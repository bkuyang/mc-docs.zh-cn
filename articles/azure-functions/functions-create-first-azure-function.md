---
title: 在 Azure 门户中创建第一个函数
description: 了解如何使用 Azure 门户创建第一个可无服务器执行的 Azure Function。
ms.assetid: 96cf87b9-8db6-41a8-863a-abb828e3d06d
ms.topic: how-to
ms.date: 06/08/2020
ms.custom: mvc, devcenter, cc996988-fb4f-47
ms.openlocfilehash: ad48b03dda4ab24957af896ae782ae31a56273f9
ms.sourcegitcommit: f1a76ee3242698123a3d77f44c860db040b48f70
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/09/2020
ms.locfileid: "84563787"
---
# <a name="create-your-first-function-in-the-azure-portal"></a>在 Azure 门户中创建第一个函数

Azure Functions 用于在无服务器环境中运行代码，无需先创建虚拟机 (VM) 或发布 Web 应用程序。 本文介绍如何使用 Azure Functions 在 Azure 门户中创建 HTTP 触发器函数“hello world”。

如果没有 Azure 订阅，可在开始前创建一个[试用帐户](https://www.azure.cn/pricing/1rmb-trial)。

如果你是 C# 开发人员，则请考虑[在 Visual Studio 2019 中创建第一个函数](functions-create-your-first-function-visual-studio.md)，而不要在门户中这样做。 

## <a name="sign-in-to-azure"></a>登录 Azure

使用 Azure 帐户登录到 [Azure 门户](https://portal.azure.cn)。

## <a name="create-a-function-app"></a>创建函数应用

必须使用 Function App 托管函数的执行。 函数应用可将函数分组为逻辑单元，以便更轻松地管理、部署、缩放和共享资源。

[!INCLUDE [Create function app Azure portal](../../includes/functions-create-function-app-portal.md)]

接下来，在新的函数应用中创建一个函数。

## <a name="create-an-http-trigger-function"></a><a name="create-function"></a>创建 HTTP 触发器函数

1. 从“Functions”**** 窗口的左侧菜单中选择“Functions”****，然后从顶部菜单中选择“添加”****。 
 
1. 在“新建函数”**** 窗口中，选择“Http 触发器”****。

    ![选择 HTTP 触发器函数](./media/functions-create-first-azure-function/function-app-select-http-trigger.png)

1. 在“新建函数”**** 窗口中，接受新函数**** 的默认名称，或输入新名称。 

1. 从“授权级别”**** 下拉列表中选择“匿名”****，然后选择“创建函数”****。

    Azure 会创建该 HTTP 触发器函数。 现在，可以通过发送 HTTP 请求来运行新函数了。

## <a name="test-the-function"></a>测试函数

1. 在新的 HTTP 触发器函数中，从左侧菜单中选择“代码 + 测试”****"，然后从顶部菜单中选择“获取函数 URL”****。

    ![选择“获取函数 URL”](./media/functions-create-first-azure-function/function-app-select-get-function-url.png)

1. 在“获取函数 URL”**** 对话框的下拉列表中选择“默认值”****，然后选择“复制到剪贴板”**** 图标。 

    ![从 Azure 门户复制函数 URL](./media/functions-create-first-azure-function/function-app-develop-tab-testing.png)

1. 将函数 URL 粘贴到浏览器的地址栏中。 将查询字符串值 `?name=<your_name>` 添加到该 URL 尾部，然后按 Enter 来运行请求。 

    以下示例显示了浏览器中的响应：

    ![浏览器中的函数响应。](./media/functions-create-first-azure-function/function-app-browser-testing.png)

    请求 URL 包含通过 HTTP 访问函数默认所需的密钥。

1. 运行函数时，会在日志中写入跟踪信息。 若要查看跟踪输出，请返回到门户中的“代码 + 测试”**** 页，并展开页面底部的“日志”**** 箭头。

   ![Azure 门户中的“函数日志”查看器。](./media/functions-create-first-azure-function/function-view-logs.png)

## <a name="clean-up-resources"></a>清理资源

[!INCLUDE [Clean-up resources](../../includes/functions-quickstart-cleanup.md)]

## <a name="next-steps"></a>后续步骤

[!INCLUDE [Next steps note](../../includes/functions-quickstart-next-steps.md)]


