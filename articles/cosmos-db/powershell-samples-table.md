---
title: 适用于 Azure Cosmos DB 的 Azure PowerShell 示例 - 表 API
description: 获取用于在 Azure Cosmos DB 表 API 帐户中执行各种常见任务的 Azure PowerShell 示例
author: rockboyfor
ms.service: cosmos-db
ms.topic: sample
origin.date: 05/13/2020
ms.date: 07/06/2020
ms.author: v-yeche
ms.openlocfilehash: dd9ba23457d877f44a04998185e1084f5d8490eb
ms.sourcegitcommit: f5484e21fa7c95305af535d5a9722b5ab416683f
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/24/2020
ms.locfileid: "85323332"
---
# <a name="azure-powershell-samples-for-azure-cosmos-db---table-api"></a>适用于 Azure Cosmos DB 的 Azure PowerShell 示例 - 表 API

下表包含用于 Azure Cosmos DB for Table API 的示例 Azure PowerShell 脚本的链接。

> [!NOTE]
> 该示例使用 [Az.CosmosDB](https://docs.microsoft.com/powershell/module/az.cosmosdb) 管理 cmdlet。 请定期检查 `Az.CosmosDB` 是否有更新。

| | |
|---|---|
|[创建帐户和表](scripts/powershell/table/ps-table-create.md)| 创建 Azure Cosmos 帐户和表。 |
|[列出或获取表](scripts/powershell/table/ps-table-list-get.md)| 列出或获取表。 |
|[获取 RU/秒](scripts/powershell/table/ps-table-ru-get.md)| 获取表的 RU/秒。 |
|[更新 RU/秒](scripts/powershell/table/ps-table-ru-update.md)| 更新表的 RU/秒。 |
|[更新帐户或添加区域](scripts/powershell/common/ps-account-update.md)| 将区域添加到 Cosmos 帐户。 也可用于修改其他帐户属性，但这些必须与对区域的更改分开。 |
|[更改故障转移优先级或触发故障转移](scripts/powershell/common/ps-account-failover-priority-update.md)| 更改 Azure Cosmos 帐户的区域故障转移优先级或触发手动故障转移。 |
|[帐户密钥或连接字符串](scripts/powershell/common/ps-account-keys-connection-strings.md)| 获取主密钥和辅助密钥、连接字符串或重新生成 Azure Cosmos 帐户的帐户密钥。 |
|[创建启用 IP 防火墙的 Cosmos 帐户](scripts/powershell/common/ps-account-firewall-create.md)| 创建启用 IP 防火墙的 Azure Cosmos 帐户。 |
|||

<!-- Update_Description: update meta properties, wording update, update link -->