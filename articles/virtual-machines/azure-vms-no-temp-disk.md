---
title: 无本地临时磁盘的 Azure VM 规格的常见问题解答
description: 本文提供有关没有本地临时磁盘的 Azure VM 规格的常见问题解答 (FAQ)。
author: rockboyfor
ms.service: virtual-machines
ms.topic: article
ms.reviewer: mimckitt
origin.date: 06/15/2020
ms.date: 07/27/2020
ms.testscope: no
ms.testdate: ''
ms.author: v-yeche
ms.openlocfilehash: 58a5644e8431ffbffea13bce525736ccfe2528d4
ms.sourcegitcommit: 2b78a930265d5f0335a55f5d857643d265a0f3ba
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/28/2020
ms.locfileid: "87254857"
---
<!--Pending GA on Q3 2020, only be suitable for China East 2 site-->
<!--RELEASE BEFORE CONFIRME AND BE CAREFULLY-->
# <a name="azure-vm-sizes-with-no-local-temporary-disk"></a>无本地临时磁盘的 Azure VM 规格 
本文提供有关没有本地临时磁盘（即无本地临时磁盘）的 Azure VM 规格的常见问题解答 (FAQ)。 有关这些 VM 规格的详细信息，请参阅 [Dv4 和 Dsv4 系列规范（常规用途工作负载）](dv4-dsv4-series.md)或 [Ev4 和 Esv4 系列规范（内存优化工作负载）](ev4-esv4-series.md)。

> [!IMPORTANT]
> Dv4、Dsv4、Ev4 和 Esv4 的 VM 规格现以公共预览版提供。 若要注册公共预览版，请填写此 [窗体](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR_Y3toRKxchLjARedqtguBRURE1ZSkdDUzg1VzJDN0cwWUlKTkcyUlo5Mi4u)。 

## <a name="what-does-no-local-temp-disk-mean"></a>无本地临时磁盘是什么意思？ 
通常，我们的 VM 规格（例如 Standard_D2s_v3、Standard_E48_v3）包含一个小型本地磁盘（例如 D:驱动器）。 现在，使用这些新的 VM 规格，该小型本地磁盘将不再存在；但你仍然可以附加标准 HDD、高级 SSD 或超级 SSD。

## <a name="what-if-i-still-want-local-temp-disk"></a>如果仍需要本地临时磁盘怎么办？
如果工作负载需要本地临时磁盘，我们也提供了新的 [Ddv4 和 Ddsv4](ddv4-ddsv4-series.md) 或 [Edv4 和 Edsv4](edv4-edsv4-series.md) VM 规格。 与以前的 v3 规格相比，这些规格提供大 50% 的临时磁盘。

> [!NOTE]
> 本地临时磁盘不是持久性的；若要确保数据是持久性数据，请使用标准 HDD、高级 SSD 或超级 SSD 选项。 

## <a name="what-are-the-differences-between-these-new-vm-sizes-and-the-general-purpose-dv3dsv3-or-the-memory-optimized-ev3esv3-vm-sizes-that-i-am-used-to"></a>这些新的 VM 规格与我使用的常规用途 Dv3/Dsv3 或内存优化 Ev3/Esv3 VM 规格之间有何区别？ 
| 具有本地临时磁盘的 v3 VM 系列   | 具有本地临时磁盘的新型 v4 系列 | 无本地临时磁盘的新型 v4 系列 |
|---|---|---|
| Dv3   | Ddv4 | Dv4 |
| Dsv3 | Ddsv4  | Dsv4 |
| Ev3   | Edv4  | Ev4 |
| Esv3 | Edsv4 |    Esv4 | 

## <a name="can-i-resize-a-vm-size-that-has-a-local-temp-disk-to-a-vm-size-with-no-local-temp-disk"></a>是否可将具有本地临时磁盘的 VM 规格调整为无本地临时磁盘的 VM 规格？  
否。 只允许下方的组合调整规格： 

1. VM（具有本地临时磁盘）-> VM（具有本地临时磁盘）；以及 
2. VM（无本地临时磁盘）-> VM（无本地临时磁盘）。 

> [!NOTE]
> 如果映像依赖于资源磁盘，或者本地临时磁盘上存在页面文件或交换文件，则无磁盘映像将不起作用，而需改用“具有磁盘”替代项。 

## <a name="do-these-vm-sizes-support-both-linux-and-windows-operating-systems-os"></a>这些 VM 规格是否支持 Linux 和 Windows 操作系统 (OS)？
是的。

## <a name="will-this-break-my-custom-scripts-custom-images-or-os-images-that-have-scratch-files-or-page-files-on-a-local-temp-disk"></a>这是否会中断自定义脚本、自定义映像或在本地临时磁盘上具有暂存文件或页面文件的 OS 映像？
如果自定义 OS 映像指向本地临时磁盘，则该映像可能无法通过此无磁盘规格正常工作。

## <a name="have-questions-or-feedback"></a>有问题或反馈？
填写[反馈窗体]( https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR_Y3toRKxchLjARedqtguBRUMzdCQkw0OVVRTldFUUtXSTlLQVBPUkVHSy4u)。 

## <a name="next-steps"></a>后续步骤 
本文档详细介绍了有关具有本地临时磁盘的 Azure VM 的最常见问题。 有关这些 VM 规格的详细信息，请参阅以下文章：

- [Dv4 和 Dsv4 系列规格（常规用途工作负载）](dv4-dsv4-series.md)
- [Ev4 和 Esv4 系列规范（内存优化工作负载）](ev4-esv4-series.md)

<!-- Update_Description: new article about azure vms no temp disk -->
<!--NEW.date: 07/27/2020-->