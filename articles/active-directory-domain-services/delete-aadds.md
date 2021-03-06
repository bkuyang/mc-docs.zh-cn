---
title: 删除 Azure Active Directory 域服务 | Microsoft Docs
description: 了解如何使用 Azure 门户禁用或删除 Azure Active Directory 域服务托管域
services: active-directory-ds
author: iainfoulds
manager: daveba
ms.assetid: 89e407e1-e1e0-49d1-8b89-de11484eee46
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/13/2020
ms.author: v-junlch
ms.openlocfilehash: b240f5f0a1f690859dae76e9f29a03c7ec2d403d
ms.sourcegitcommit: fe9ccd3bffde0dd2b528b98a24c6b3a8cbe370bc
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/20/2020
ms.locfileid: "86472569"
---
# <a name="delete-an-azure-active-directory-domain-services-managed-domain-using-the-azure-portal"></a>使用 Azure 门户删除 Azure Active Directory 域服务托管域

如果不再需要托管域，则可以删除 Azure Active Directory 域服务 (Azure AD DS) 托管域。 没有关闭或暂时禁用 Azure AD DS 托管域的选项。 删除托管域不会删除 Azure AD 租户，否则会对 Azure AD 租户产生不利影响。 本文介绍了如何使用 Azure 门户删除托管域。

> [!WARNING]
> 删除是永久性的，无法撤消。
> 删除托管域时，将发生以下步骤：
>   * 将取消设置托管域的域控制器，并将从虚拟网络中删除。
>   * 托管域上的数据将被永久删除。 此数据包括创建的自定义 OU、GPO、自定义 DNS 记录、服务主体、GMSA 等。
>   * 已加入到托管域的计算机将丢失其与域的信任关系，需要从域中取消加入。
>       * 将无法使用公司 AD 凭据登录这些计算机。 相反，必须使用计算机的本地管理员凭据。

## <a name="delete-the-managed-domain"></a>删除托管域

要删除托管域，请完成以下步骤：

1. 在 Azure 门户中，搜索并选择“Azure AD 域服务”。
1. 选择你的托管域，例如 aaddscontoso.com。
1. 在“概览”页上，选择“删除”。 若要确认删除，请再次输入托管域的域名，然后选择“删除”。

删除托管域可能需要 15-20 分钟或更长时间。

## <a name="next-steps"></a>后续步骤

考虑[共享反馈][feedback]以获取希望在 Azure AD DS 中看到的功能。

如果要再次开始使用 Azure AD DS，请参阅[创建和配置 Azure Active Directory 域服务托管域][create-instance]。

<!-- INTERNAL LINKS -->
[feedback]: https://feedback.azure.com/forums/169401-azure-active-directory?category_id=160593
[create-instance]: tutorial-create-instance.md

