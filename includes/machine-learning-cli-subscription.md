---
author: Blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 03/26/2020
ms.author: larryfr
ms.openlocfilehash: 682b9b7124a89bc83acce4b99a1770801ebffb08
ms.sourcegitcommit: d210eb03ed6432aeefd3e9b1c77d2c92a6a8dbca
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/30/2020
ms.locfileid: "82591446"
---
> [!TIP]
> 登录后，你将看到与你的 Azure 帐户关联的订阅列表。 `isDefault: true` 的订阅信息是当前已激活的 Azure CLI 命令订阅。 此订阅必须与包含 Azure 机器学习工作区的订阅相同。 通过访问工作区的概述页，可以从 [Azure 门户](https://portal.azure.cn)中找到订阅 ID。 还可以使用 SDK 从工作区对象获取订阅 ID。 例如，`Workspace.from_config().subscription_id`。
> 
> 若要选择另一个订阅，请使用 `az account set -s <subscription name or ID>` 命令，并指定要切换到的订阅名称或 ID。 有关订阅选择的详细信息，请参阅[使用多个 Azure 订阅](/cli/manage-azure-subscriptions-azure-cli?view=azure-cli-latest)。