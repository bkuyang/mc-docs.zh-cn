---
title: 使用 Azure 媒体服务创建多比特率流时配置本地编码器 | Microsoft Docs
description: 本主题列出的本地实时编码器可用于捕获直播活动，并将单比特率实时流发送到 AMS 频道（已启用实时编码）以供进一步处理。 本主题列出了演示如何配置所列编码器的教程链接。
services: media-services
documentationcenter: ''
author: WenJason
manager: digimobile
editor: ''
ms.assetid: 0ec6f046-0841-4673-9057-883bdbc30d5c
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
origin.date: 03/18/2019
ms.date: 06/08/2020
ms.author: v-jay
ms.openlocfilehash: fc2eed0958513e742fa23913a7b9883847230d57
ms.sourcegitcommit: 79c99a9ea013b3c74706a1038a505f4eea2aaac4
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/05/2020
ms.locfileid: "84439537"
---
# <a name="how-to-configure-on-premises-encoders-when-using-azure-media-services-to-create-multi-bitrate-streams"></a>如何在使用 Azure 媒体服务时配置本地编码器以创建多比特率流
本主题列出的本地实时编码器可用于捕获直播活动，并将单比特率实时流发送到 AMS 频道（已启用实时编码）以供进一步处理。 本主题还列出了演示如何配置所列编码器的教程链接。

> [!NOTE]
> 通过 RTMP 流式处理时，请检查防火墙和/或代理设置，确认出站 TCP 端口 1935 和 1936 已打开。

## <a name="haivision-kb-encoder"></a>Haivision KB 编码器
有关如何配置 [Haivision KB 编码器](https://www.haivision.com/products/kb-series/)以将单比特率实时流发送到 AMS 通道的信息，请参阅[配置 Haivision KB 编码器](media-services-configure-kb-live-encoder.md)。

## <a name="telestream-wirecast"></a>Telestream Wirecast
有关如何配置 [Telestream Wirecast](https://www.telestream.net/wirecast/overview.htm) 编码器以将单比特率实时流发送到 AMS 通道的信息，请参阅[配置 Wirecast](media-services-configure-wirecast-live-encoder.md)。

## <a name="elemental-live"></a>Elemental Live
有关详细信息，请参阅 [Elemental Live](https://www.elemental.com/products/aws-elemental-appliances-software/#elemental-live)。

## <a name="media-services-learning-paths"></a>媒体服务学习路径
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="next-steps"></a>后续步骤

[使用 Azure 媒体服务实时传送视频流以创建多比特率流](media-services-manage-live-encoder-enabled-channels.md)。

