---
title: 使用门户上传、编码和流式传输内容 - Azure
description: 本快速入门介绍如何使用门户通过 Azure 媒体服务上传、编码和流式传输内容。
ms.topic: quickstart
origin.date: 06/08/2020
ms.date: 07/27/2020
ms.openlocfilehash: 5a3bef365eb7cb60b684335686fc9c2f5f018b65
ms.sourcegitcommit: 091c672fa448b556f4c2c3979e006102d423e9d7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/24/2020
ms.locfileid: "87162758"
---
# <a name="quickstart-upload-encode-and-stream-content-with-portal"></a>快速入门：使用门户上传、编码和流式传输内容

> [!NOTE]
> Google Widevine 内容保护服务目前在 Azure 中国区域不可用。

本快速入门介绍如何使用 Azure 门户 通过 Azure 媒体服务上传、编码和流式传输内容。

> [!NOTE]
> 请务必查看：[媒体服务 v3 的 Azure 门户限制](frequently-asked-questions.md#what-are-the-azure-portal-limitations-for-media-services-v3)。
  
## <a name="overview"></a>概述

* 要开始管理、加密、编码和流式处理 Azure 中的媒体内容，需要创建媒体服务帐户，并将高质量的数字媒体文件上传到资产中。 
    
    > [!NOTE]
    > 如果视频以前是使用媒体服务 v3 API 上传到媒体服务帐户的，或者内容是基于实时输出生成的，则 Azure 门户中不会显示“编码”或“加密”按钮 。 使用媒体服务 v3 API 来执行这些任务。

    查看以下内容： 

  * [云上传和存储](storage-account-concept.md)
  * [资产概念](assets-concept.md)
  * [管理资产](manage-asset-concept.md)
* 将高质量的数字媒体文件上传到一个资产（输入资产）后，可对其进行处理（编码）。 处理的内容将进入另一个资产（输出资产）。 
    * 将上传的文件[编码](encoding-concept.md)为可在各种浏览器和设备上播放的格式。
* 处理完内容后，即可将媒体内容交付给客户端播放器。 若要使输出资产中的视频可供客户端进行播放，必须创建流式处理定位符。 创建流式处理定位符时，需要指定流式处理策略 。 使用流式处理策略可为流式处理定位符定义流式处理协议和加密选项（如果有） 。
    
    审阅：

    * [流式处理定位符](streaming-locators-concept.md)
    * [流式处理策略](streaming-policy-concept.md)
    * [打包和交付](dynamic-packaging-overview.md)
    * [筛选器](filters-concept.md)
* 可以通过使用高级加密标准 (AES-128) 或/和以下两种主要 DRM 系统对内容进行加密，以保护内容：Microsoft PlayReady 和 Apple FairPlay。 [使用 Azure 门户加密内容](encrypt-content-quickstart.md)快速入门介绍如何配置内容保护。
        
## <a name="prerequisites"></a>先决条件

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[创建媒体服务帐户](create-account-howto.md#use-the-azure-portal)

## <a name="upload"></a>上传

1. 登录到 [Azure 门户](https://portal.azure.cn/)。
1. 找到并单击你的媒体服务帐户。
1. 选择“资产(新)”。
1. 在该窗口顶部按“上传”。 
1. 拖放或浏览到要上传的文件。

如果导航到资产窗口，可看到一个新资产已添加到列表中：

![上载](./media/manage-assets-quickstart/upload.png)

## <a name="encode"></a>编码

1. 选择“资产(新)”。
1. 选择新资产（已在上一步中添加）。
1. 单击窗口顶部的“编码”。

    按此按钮将启动编码作业。 作业成功完成后，生成一个包含已编码内容的输出资产。

如果导航到资产窗口，可看到该输出资产已添加到列表中：

![编码](./media/manage-assets-quickstart/encode.png)

## <a name="monitor-the-job-progress"></a>监视作业进度

若要查看作业状态，请导航到“作业”。 作业通常会经历以下状态：已计划、已排队、正在处理、已完成（最终状态）。 如果作业出错，则显示“错误”状态。

![状态](./media/manage-assets-quickstart/job-status.png)

## <a name="publish-and-stream"></a>发布和流式传输

若要发布资产，现需要向资产添加流式处理定位符。

### <a name="streaming-locator"></a>流式处理定位符 

1. 在“流式处理定位符”部分，按“+ 添加流式处理定位符” 。
    该操作将发布资产并生成流式处理 URL。

    > [!NOTE]
    > 如果要对流进行加密，则需要创建内容密钥策略，并在流式处理定位符上对其进行设置。 有关详细信息，请参阅[使用 Azure 门户加密内容](encrypt-content-quickstart.md)。
1. 在“添加流式处理定位符”窗口中，选择一个预定义的流式处理策略。 有关详细信息，请参阅[流式处理策略](streaming-policy-concept.md)

    ![流式处理定位符](./media/manage-assets-quickstart/streaming-locator.png)

资产发布后，可直接在门户中对其进行流式传输。 

![显示](./media/manage-assets-quickstart/publish.png)

或者，复制流式处理 URL 并将其用于客户端播放器。

> [!NOTE]
> 确保[流式处理终结点](streaming-endpoint-concept.md)正在运行。 在你首次创建媒体服务帐户时，系统将创建默认的流式处理终结点，且该终结点处于停止状态，因此你需要启动它，然后才能流式传输内容。<br/>仅当流式处理终结点处于运行状态时才进行计费。

## <a name="cleanup-resources"></a>清理资源

如果想学习其他快速入门，请保留创建的资源。 否则，请转到 Azure 门户，浏览到资源组，选择运行本快速入门所用的资源组，并删除所有资源。

## <a name="next-steps"></a>后续步骤

[使用门户加密内容](encrypt-content-quickstart.md)
