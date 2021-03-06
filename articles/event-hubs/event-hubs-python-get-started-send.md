---
title: 使用 Python（旧版）向/从 Azure 事件中心发送/接收事件
description: 本演练介绍如何创建和运行 Python 脚本，这些脚本使用旧的 azure-eventhub 版本 1 包向/从 Azure 事件中心发送/接收事件。
services: event-hubs
author: spelluru
manager: femila
ms.service: event-hubs
ms.workload: core
ms.topic: quickstart
origin.date: 01/15/2020
ms.date: 07/01/2020
ms.author: v-tawe
ms.custom: tracking-python
ms.openlocfilehash: 9872453a1b67de406caa91e348383b7b5a9325ae
ms.sourcegitcommit: 4f84bba7e509a321b6f68a2da475027c539b8fd3
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/01/2020
ms.locfileid: "85796191"
---
# <a name="quickstart-send-and-receive-events-with-event-hubs-using-python-azure-eventhub-version-1"></a>快速入门：使用 Python（azure-eventhub 版本 1）向/从事件中心发送/接收事件
本快速入门介绍如何使用 **azure-eventhub 版本 1** Python 包向事件中心发送事件以及从事件中心接收事件。 

> [!WARNING]
> 本快速入门使用旧的 azure-eventhub 版本 1 包。 有关使用该包的最新**版本 5** 的快速入门，请参阅[使用 azure-eventhub 版本 5 发送和接收事件](get-started-python-send-v2.md)。 若要将应用程序从使用旧包迁移到使用新包，请参阅[从 azure-eventhub 版本 1 迁移到版本 5 的指南](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub/migration_guide.md)。
 

## <a name="prerequisites"></a>先决条件
如果不熟悉 Azure 事件中心，请在阅读本快速入门之前参阅[事件中心概述](event-hubs-about.md)。 

若要完成本快速入门，需要具备以下先决条件：

- **Azure 订阅**。 若要使用 Azure 服务（包括 Azure 事件中心），需要一个订阅。  如果没有现有 Azure 帐户，可以注册 [1 元试用版](https://wd.azure.cn/pricing/1rmb-trial/)或[创建帐户](https://wd.azure.cn/pricing/pia/)。
- Python 3.4 或更高版本，其中已安装并更新 `pip`。
- 事件中心的 Python 包。 若要安装此包，请在路径中包含 Python 的命令提示符中运行以下命令： 
  
  ```cmd
  pip install azure-eventhub==1.3.*
  ```
- **创建事件中心命名空间和事件中心**。 第一步是使用 [Azure 门户](https://portal.azure.cn)创建事件中心类型的命名空间，并获取应用程序与事件中心进行通信所需的管理凭据。 要创建命名空间和事件中心，请按照[此文](event-hubs-create.md)中的步骤操作。 然后，按照文章中的以下说明获取事件中心访问密钥的值：[获取连接字符串](event-hubs-get-connection-string.md#get-connection-string-from-the-portal)。 你将在本快速入门中稍后编写的代码中使用访问密钥。 默认密钥名称为：RootManageSharedAccessKey。 


## <a name="send-events"></a>发送事件

若要创建将事件发送到事件中心的 Python 应用程序，请执行以下操作：

> [!NOTE]
> 可以从 GitHub 下载并运行[示例应用](https://github.com/Azure/azure-event-hubs-python/tree/master/examples)，不需通过本快速入门来进行。 将 `EventHubConnectionString` 和 `EventHubName` 字符串替换为事件中心的值。

1. 打开你常用的 Python 编辑器，例如 [Visual Studio Code](https://code.visualstudio.com/)
2. 创建名为 *send.py* 的新文件。 此脚本将向事件中心发送 100 个事件。
3. 将下列代码粘贴到“send.py”，将事件中心 \<namespace>、\<eventhub>、\<AccessKeyName> 和 \<primary key value> 替换为你的值： 
   
   ```python
   import sys
   import logging
   import datetime
   import time
   import os
   
   from azure.eventhub import EventHubClient, Sender, EventData
   
   logger = logging.getLogger("azure")
   
   # Address can be in either of these formats:
   # "amqps://<URL-encoded-SAS-policy>:<URL-encoded-SAS-key>@<namespace>.servicebus.chinacloudapi.cn/eventhub"
   # "amqps://<namespace>.servicebus.chinacloudapi.cn/<eventhub>"
   # SAS policy and key are not required if they are encoded in the URL
   
   ADDRESS = "amqps://<namespace>.servicebus.chinacloudapi.cn/<eventhub>"
   USER = "<AccessKeyName>"
   KEY = "<primary key value>"
   
   try:
       if not ADDRESS:
           raise ValueError("No EventHubs URL supplied.")
   
       # Create Event Hubs client
       client = EventHubClient(ADDRESS, debug=False, username=USER, password=KEY)
       sender = client.add_sender(partition="0")
       client.run()
       try:
           start_time = time.time()
           for i in range(100):
               print("Sending message: {}".format(i))
               message = "Message {}".format(i)
               sender.send(EventData(message))
       except:
           raise
       finally:
           end_time = time.time()
           client.stop()
           run_time = end_time - start_time
           logger.info("Runtime: {} seconds".format(run_time))
   
   except KeyboardInterrupt:
       pass
   ```
   
4. 保存文件。 

若要运行脚本，请从保存 *send.py* 的目录运行以下命令：

```cmd
start python send.py
```

祝贺！ 现在已向事件中心发送消息。

## <a name="receive-events"></a>接收事件

若要创建从事件中心接收事件的 Python 应用程序，请执行以下操作：

1. 在 Python 编辑器中，创建名为 *recv.py* 的文件。
2. 将下列代码粘贴到“recv.py”，将事件中心 \<namespace>、\<eventhub>、\<AccessKeyName> 和 \<primary key value> 替换为你的值： 
   
   ```python
   import os
   import sys
   import logging
   import time
   from azure.eventhub import EventHubClient, Receiver, Offset
   
   logger = logging.getLogger("azure")
   
   # Address can be in either of these formats:
   # "amqps://<URL-encoded-SAS-policy>:<URL-encoded-SAS-key>@<mynamespace>.servicebus.chinacloudapi.cn/myeventhub"
   # "amqps://<namespace>.servicebus.chinacloudapi.cn/<eventhub>"
   # SAS policy and key are not required if they are encoded in the URL
   
   ADDRESS = "amqps://<namespace>.servicebus.chinacloudapi.cn/<eventhub>"
   USER = "<AccessKeyName>"
   KEY = "<primary key value>"
   
   
   CONSUMER_GROUP = "$default"
   OFFSET = Offset("-1")
   PARTITION = "0"
   
   total = 0
   last_sn = -1
   last_offset = "-1"
   client = EventHubClient(ADDRESS, debug=False, username=USER, password=KEY)
   try:
       receiver = client.add_receiver(
           CONSUMER_GROUP, PARTITION, prefetch=5000, offset=OFFSET)
       client.run()
       start_time = time.time()
       for event_data in receiver.receive(timeout=100):
           print("Received: {}".format(event_data.body_as_str(encoding='UTF-8')))
           total += 1
   
       end_time = time.time()
       client.stop()
       run_time = end_time - start_time
       print("Received {} messages in {} seconds".format(total, run_time))
   
   except KeyboardInterrupt:
       pass
   finally:
       client.stop()
   ```
   
4. 保存文件。

若要运行脚本，请从保存 *recv.py* 的目录运行以下命令：

```cmd
start python recv.py
```

## <a name="next-steps"></a>后续步骤
有关事件中心的详细信息，请参阅以下文章：

- [EventProcessorHost](event-hubs-event-processor-host.md)
- [Azure 事件中心的功能和术语](event-hubs-features.md)
- [事件中心常见问题](event-hubs-faq.md)

