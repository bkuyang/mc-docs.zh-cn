---
title: include 文件
description: include 文件
services: event-grid
author: tfitzmac
ms.service: event-grid
ms.topic: include
ms.date: 11/06/2018
ms.author: tomfitz
ms.custom: include file
ms.openlocfilehash: f8e00ba9d4bba4b0c135ada057749ee281ce894e
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "70014689"
---
此功能为预览版。 若要使用它，必须安装预览扩展或模块。

### <a name="install-extension-for-azure-cli"></a>安装适用于 Azure CLI 的扩展

对于 Azure CLI，需要[事件网格扩展](/cli/azure/azure-cli-extensions-list)。

在 [CloudShell](/azure/cloud-shell/quickstart) 中：

* 如果之前已安装此扩展，请使用 `az extension update -n eventgrid` 进行更新
* 如果之前尚未安装此扩展，请使用 `az extension add -n eventgrid` 安装

对于本地安装：

1. [安装 Azure CLI](/cli/install-azure-cli)。 通过使用 `az --version` 检查，确保已安装最新版本。
1. 卸载早期版本的扩展 `az extension remove -n eventgrid`
1. 使用 `az extension add -n eventgrid` 安装 `eventgrid` 扩展

### <a name="install-module-for-powershell"></a>安装适用于 PowerShell 的模块

对于 PowerShell，需要 [AzureRM.EventGrid 模块](https://www.powershellgallery.com/packages/AzureRM.EventGrid/0.4.1-preview)。

对于本地安装：

1. 以管理员身份打开 PowerShell 控制台
1. 安装模块 `Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery`

如果 `-AllowPrerelease` 参数不可用，请使用以下步骤：

1. `Install-Module PowerShellGet -Force`运行 {2}
1. `Update-Module PowerShellGet`运行 {2}
1. 关闭 PowerShell 控制台
1. 以管理员身份重启 PowerShell
1. 安装模块 `Install-Module -Name AzureRM.EventGrid -AllowPrerelease -Force -Repository PSGallery`
