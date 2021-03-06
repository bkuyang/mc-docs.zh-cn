---
title: 所有预生成实体 - LUIS
titleSuffix: Azure Cognitive Services
description: 本文包含了语言理解 (LUIS) 中包括的预构建实体的列表。
services: cognitive-services
author: lingliw
manager: digimobile
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
origin.date: 10/03/2019
ms.date: 12/05/2019
ms.author: v-lingwu
ms.openlocfilehash: 93fcea0423c68e697c2a9546c6941aa1b3aa193c
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "78154545"
---
# <a name="entities-per-culture-in-your-luis-model"></a>LUIS 模型中每个区域性的实体

语言理解 (LUIS) 提供了预构建的实体。 当应用程序中包括预构建实体时，LUIS 会在终结点响应中包括对应的实体预测。 所有陈述示例都标记有实体。 **无法**修改预构建实体的行为。 除非另行说明，预构建实体在所有 LUIS 应用程序区域设置（语言区域）中都可用。 下表显示了每个语言区域支持的预构建实体。

|环境|子区域性|说明|
|--|--|--|
|中文|[zh-CN](#chinese-entity-support)||

## <a name="prediction-endpoint-runtime"></a>预测终结点运行时

特定语言的预生成实体的可用性由预测终结点运行时版本确定。 

## <a name="chinese-entity-support"></a>中文实体支持

支持以下实体：

|预生成实体|```zh-CN``` |
------|:------:|
[存在时长](luis-reference-prebuilt-age.md)：<br>year<br>月份<br>week<br>day   |    V2、V3   |
[货币（金钱）](luis-reference-prebuilt-currency.md)：<br>美元<br>分数单位（示例：便士）  |    V2、V3   |
[DatetimeV2](luis-reference-prebuilt-datetimev2.md)：<br>date<br>daterange<br>time<br>时间范围   |    V2、V3   | 
[维度](luis-reference-prebuilt-dimension.md)：<br>卷<br>面积<br>重量<br>信息（示例：位/字节）<br>长度（示例：米）<br>速度（示例：英里每小时）  |    V2、V3   | 
[电子邮件](luis-reference-prebuilt-email.md)   |    V2、V3   | 
[GeographyV2](luis-reference-prebuilt-geographyV2.md)   |    -   | 
[keyPhrase](luis-reference-prebuilt-keyphrase.md)   |    -   | 
[数字](luis-reference-prebuilt-number.md)   |    V2、V3   |  
[序号](luis-reference-prebuilt-ordinal.md)   |    V2、V3   |  
[百分比](luis-reference-prebuilt-percentage.md)   |    V2、V3   | 
[PersonName](luis-reference-prebuilt-person.md)   |    -   | 
[电话号码](luis-reference-prebuilt-phonenumber.md)   |    V2、V3   | 
[温度](luis-reference-prebuilt-temperature.md)：<br>华氏温度<br>开氏温度<br>兰金温度<br>德利尔温度<br>摄氏温度   |    V2、V3   | 
[URL](luis-reference-prebuilt-url.md)   |    V2、V3   |

## <a name="contribute-to-prebuilt-entity-cultures"></a>为预构建实体语言区域做贡献
预构建实体是在 Recognizers-Text 开发源代码项目中开发的。 [参与](https://github.com/Microsoft/Recognizers-Text)项目。 该项目包括每个语言区域的货币的示例。 

Recognizers-Text 项目中不包括 GeographyV2 和 PersonName。 

## <a name="next-steps"></a>后续步骤

了解[数字](luis-reference-prebuilt-number.md)、[datetimeV2](luis-reference-prebuilt-datetimev2.md) 和[货币](luis-reference-prebuilt-currency.md)实体。 




