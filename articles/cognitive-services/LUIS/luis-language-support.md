---
title: 语言支持 - LUIS
titleSuffix: Azure Cognitive Services
description: LUIS 在服务中具有多种功能。 并非所有功能都会同等地以各种语言提供。 请确保你所定位的语言文化支持你感兴趣的功能。 LUIS 应用特定于区域性，一旦设置即无法更改。
services: cognitive-services
author: lingliw
manager: digimobile
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
origin.date: 12/09/2019
ms.date: 1/2/2020
ms.author: v-lingwu
ms.openlocfilehash: ef73ff1dd10440a1804de81e38a7cadf60c74b78
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "79292129"
---
# <a name="language-and-region-support-for-luis"></a>LUIS 的语言和区域支持

LUIS 在服务中具有多种功能。 并非所有功能都会同等地以各种语言提供。 请确保你所定位的语言文化支持你感兴趣的功能。 LUIS 应用特定于区域性，一旦设置即无法更改。

## <a name="multi-language-luis-apps"></a>多语言 LUIS 应用

如果需要多语言 LUIS 客户端应用程序（例如聊天机器人），可通过几种方法实现。 如果 LUIS 支持所有语言，则需面向每种语言开发一个 LUIS 应用。 每个 LUIS 应用都具有唯一的应用 ID 和终结点日志。 如果需要为 LUIS 不支持的语言提供语言理解，可使用 [Microsoft Translator API](../Translator/translator-info-overview.md) 将表述翻译成受支持的语言，将表述提交到 LUIS 终结点，然后接收生成的分数。

## <a name="languages-supported"></a>支持的语言

LUIS 理解以下语言：

| 语言 |Locale  |  预生成域 | 预生成实体 | 短语列表建议 | \**[文本分析](/cognitive-services/text-analytics/text-analytics-supported-languages)<br>（情绪和<br>关键字）|
|--|--|:--:|:--:|:--:|:--:|
| 美国英语 |`en-US` | ✔ | ✔  |✔|✔|
| 阿拉伯语（预览版 - 现代标准阿拉伯语） |`ar-AR`|-|-|-|-|
| *[中文](#chinese-support-notes) |`zh-CN` | ✔ | ✔ |✔|-|
| 荷兰语 |`nl-NL` |✔|  -   |-|✔|
| 法语（法国） |`fr-FR` |✔| ✔ |✔ |✔|
| 法语（加拿大） |`fr-CA` |-|   -   |-|✔|
| 德语 |`de-DE` |✔| ✔ |✔ |✔|
| Hindi | `hi-IN`|-|-|-|-|
| 意大利语 |`it-IT` |✔| ✔ |✔|✔|
| *[日语](#japanese-support-notes) |`ja-JP` |✔| ✔ |✔|仅关键短语|
| 韩语 |`ko-KR` |✔|   -   |-|仅关键短语|
| 葡萄牙语（巴西） |`pt-BR` |✔| ✔ |✔ |并非所有亚区域性|
| 西班牙语(西班牙) |`es-ES` |✔| ✔ |✔|✔|
| 西班牙语（墨西哥）|`es-MX` |-|  -   |✔|✔|
| 土耳其语 | `tr-TR` |✔|-|-|仅情绪|

[预生成实体](luis-reference-prebuilt-entities.md)和[预生成域](luis-reference-prebuilt-domains.md)具有不同的语言支持。

[!INCLUDE [Chinese language support notes](includes/chinese-language-support-notes.md)]

### <a name="japanese-support-notes"></a>*日语支持说明

 - 在 `zh-cn` 区域性中，LUIS 要求简体中文字符集，而不是繁体字符集。
 - 意向、实体、功能和正则表达式的名称可采用中文或罗马字符。
 - 请参阅[预生成域参考](luis-reference-prebuilt-domains.md)，了解 `zh-cn` 区域性支持的预生成域。
<!--- When writing regular expressions in Chinese, do not insert whitespace between Chinese characters.-->

## <a name="rare-or-foreign-words-in-an-application"></a>应用程序中的罕见字词或外来字词
在 `en-us` 区域性中，LUIS 可学习区分大多数英文字词，包括俚语。 在 `zh-cn` 区域性中，LUIS 可学习区分大多数中文字符。 如果在 `en-us` 或 `zh-cn` 中使用一个罕见字词或字符，并且 LUIS 似乎无法识别该字词或字符，则可将该字词或字符添加到[短语列表功能](luis-how-to-add-features.md)。 例如，应将超出应用程序区域性的字词（即外来字词）添加到短语列表功能。 应将此短语列表标记为不可互换，以指示罕见字词集组成 LUIS 应学会识别的类，但它们不是同义词，也不能彼此互换。

### <a name="hybrid-languages"></a>混合语言
混合语言混含两个区域性的字词，如英语和中文。 由于单个应用仅基于单个区域性，因此 LUIS 不支持此类语言。

## <a name="tokenization"></a>词汇切分
为了执行机器学习，LUIS 基于区域性将表述拆分成[词法单元](luis-glossary.md#token)。

|语言|  每个空格或特殊字符 | 字符级|复合词|[返回的切分后的实体](luis-concept-data-extraction.md#tokenized-entity-returned)
|--|:--:|:--:|:--:|:--:|
|阿拉伯语|||||
|中文||✔||✔|
|荷兰语|||✔|✔|
|美国英语|✔ ||||
|法语 (fr-FR)|✔||||
|法语 (fr-CA)|✔||||
|德语|||✔|✔|
| Hindi |✔|-|-|-|-|
|意大利语|✔||||
|日语||||✔|
|韩语||✔||✔|
|葡萄牙语（巴西）|✔||||
|西班牙语 (es-ES)|✔||||
|西班牙语 (es-MX)|✔||||
