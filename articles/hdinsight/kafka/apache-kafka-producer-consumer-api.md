---
title: '教程：使用 Apache Kafka 生成者和使用者 API - Azure HDInsight '
description: 了解如何将 Apache Kafka 生成者和使用者 API 与 Kafka on HDInsight 配合使用。 本教程介绍如何从 Java 应用程序中将这些 API 与 Kafka on HDInsight 配合使用。
services: hdinsight
documentationcenter: ''
author: Blackmist
manager: cgronlun
editor: cgronlun
tags: azure-portal
ms.service: hdinsight
ms.custom: hdinsightactive
ms.devlang: na
ms.topic: tutorial
origin.date: 05/19/2020
ms.author: v-yiso
ms.date: 07/06/2020
ms.openlocfilehash: c2539725e997023fc772d4901119aa8c17fad352
ms.sourcegitcommit: 3a8a7d65d0791cdb6695fe6c2222a1971a19f745
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2020
ms.locfileid: "85516751"
---
# <a name="tutorial-use-the-apache-kafka-producer-and-consumer-apis"></a>教程：使用 Apache Kafka 生成者和使用者 API

了解如何将 Apache Kafka 生成者和使用者 API 与 Kafka on HDInsight 配合使用。

Kafka 生成者 API 允许应用程序将数据流发送到 Kafka 群集。 Kafka 使用者 API 允许应用程序从群集读取数据流。

本教程介绍如何执行下列操作：

> [!div class="checklist"]
> * 先决条件
> * 了解代码
> * 生成并部署应用程序
> * 在群集上运行应用程序

有关这些 API 的详细信息，请参阅有关[生成者 API](https://kafka.apache.org/documentation/#producerapi) 和[使用者 API](https://kafka.apache.org/documentation/#consumerapi) 的 Apache 文档。

## <a name="prerequisites"></a>先决条件

* Apache Kafka on HDInsight 群集。 若要了解如何创建该群集，请参阅 [Apache Kafka on HDInsight 入门](apache-kafka-get-started.md)。
* [Java Developer Kit (JDK) 版本 8](https://aka.ms/azure-jdks) 或等效工具，例如 OpenJDK。

* 根据 Apache 要求正确[安装](https://maven.apache.org/install.html)的 [Apache Maven](https://maven.apache.org/download.cgi)。  Maven 是 Java 项目的项目生成系统。

* 一个 SSH 客户端，例如 Putty。 有关详细信息，请参阅[使用 SSH 连接到 HDInsight (Apache Hadoop)](../hdinsight-hadoop-linux-use-ssh-unix.md)。

## <a name="understand-the-code"></a>了解代码

示例应用程序位于 `Producer-Consumer` 子目录的 [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started) 中。 该应用程序主要包含四个文件：

该应用程序主要包含四个文件：
* `pom.xml`：此文件定义项目依赖项、Java 版本和打包方法。
* `Producer.java`：此文件使用生成者 API 将随机句子发送到 Kafka。
* `Consumer.java`：此文件使用使用者 API 从 Kafka 读取数据并将其发出到 STDOUT。
* `AdminClientWrapper.java`：此文件使用管理 API 来创建、描述和删除 Kafka 主题。
* `Run.java`：用于运行生成者和使用者代码的命令行接口。

### <a name="pomxml"></a>Pom.xml

在 `pom.xml` 文件中要了解的重要事项：

* 依赖项：此项目依赖于 `kafka-clients` 包提供的 Kafka 生成者和使用者 API。 以下 XML 代码定义此依赖项：

    ```xml
    <!-- Kafka client for producer/consumer operations -->
    <dependency>
      <groupId>org.apache.kafka</groupId>
      <artifactId>kafka-clients</artifactId>
      <version>${kafka.version}</version>
    </dependency>
    ```

    `${kafka.version}` 条目在 `pom.xml` 的 `<properties>..</properties>` 部分进行声明，并配置为 HDInsight 群集的 Kafka 版本。

* 插件：Maven 插件提供各种功能。 此项目使用了以下插件：

    * `maven-compiler-plugin`：用于将项目使用的 Java 版本设置为 8。 这是 HDInsight 3.6 的 Java 版本。
    * `maven-shade-plugin`：用于生成包含此应用程序以及任何依赖项的 uber jar。 它还用于设置应用程序的入口点，以便直接运行 Jar 文件，而无需指定主类。

### <a name="producerjava"></a>Producer.java

生成者与 Kafka 中转站主机（辅助角色节点）进行通信，并将数据发送到 Kafka 主题。 以下代码片段摘自 [GitHub 存储库](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started)中的 [Producer.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Producer-Consumer/src/main/java/com/microsoft/example/Producer.java) 文件，演示了如何设置生成者属性：

```java
Properties properties = new Properties();
// Set the brokers (bootstrap servers)
properties.setProperty("bootstrap.servers", brokers);
// Set how to serialize key/value pairs
properties.setProperty("key.serializer","org.apache.kafka.common.serialization.StringSerializer");
properties.setProperty("value.serializer","org.apache.kafka.common.serialization.StringSerializer");
KafkaProducer<String, String> producer = new KafkaProducer<>(properties);
```

### <a name="consumerjava"></a>Consumer.java

使用者与 Kafka 代理主机（辅助角色节点）进行通信，并在循环中读取记录。 [Consumer.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Producer-Consumer/src/main/java/com/microsoft/example/Consumer.java) 文件中的以下代码片段设置了使用者属性：

```java
KafkaConsumer<String, String> consumer;
// Configure the consumer
Properties properties = new Properties();
// Point it to the brokers
properties.setProperty("bootstrap.servers", brokers);
// Set the consumer group (all consumers must belong to a group).
properties.setProperty("group.id", groupId);
// Set how to serialize key/value pairs
properties.setProperty("key.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
properties.setProperty("value.deserializer","org.apache.kafka.common.serialization.StringDeserializer");
// When a group is first created, it has no offset stored to start reading from. This tells it to start
// with the earliest record in the stream.
properties.setProperty("auto.offset.reset","earliest");

consumer = new KafkaConsumer<>(properties);
```

在此代码中，使用者配置为从该主题的开头读取（`auto.offset.reset` 设置为 `earliest`）。

### <a name="runjava"></a>Run.java

[Run.java](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/blob/master/Producer-Consumer/src/main/java/com/microsoft/example/Run.java) 文件提供了命令行接口，可运行生成者或使用者代码。 必须提供 Kafka 代理主机信息作为参数。 可以选择包括组 ID 值，该值由使用者进程使用。 如果使用相同的组 ID 创建多个使用者实例，它们将对主题的读取进行负载均衡。

## <a name="build-and-deploy-the-example"></a>生成并部署示例

### <a name="use-pre-built-jar-files"></a>使用预建 JAR 文件

从 [Kafka 入门 Azure 示例](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started/tree/master/Prebuilt-Jars)下载 jar。 如果你的群集启用了**企业安全性套餐 (ESP)** ，请使用 kafka-producer-consumer-esp.jar。 使用以下命令可将 jar 复制到群集。

```cmd
scp kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.cn:kafka-producer-consumer.jar
```

### <a name="build-the-jar-files-from-code"></a>从代码生成 JAR 文件

如果要跳过此步骤，可以从 `Prebuilt-Jars` 子目录下载预构建的 jar。 下载 kafka-producer-consumer.jar。 如果你的群集启用了**企业安全性套餐 (ESP)** ，请使用 kafka-producer-consumer-esp.jar。 执行步骤 3 来将该 jar 复制到你的 HDInsight 群集。

1. 从 [https://github.com/Azure-Samples/hdinsight-kafka-java-get-started](https://github.com/Azure-Samples/hdinsight-kafka-java-get-started) 下载并提取示例。

2. 将当前目录设置为 `hdinsight-kafka-java-get-started\Producer-Consumer` 目录的位置，然后执行以下命令：

    ```cmd
    mvn clean package
    ```

    此命令创建名为 `target` 的目录，其中包含名为 `kafka-producer-consumer-1.0-SNAPSHOT.jar` 的文件。

3. 将 `sshuser` 替换为群集的 SSH 用户，并将 `CLUSTERNAME` 替换为群集的名称。 输入以下命令，将 `kafka-producer-consumer-1.0-SNAPSHOT.jar` 文件复制到 HDInsight 群集。 出现提示时，请输入 SSH 用户的密码。

    ```cmd
    scp ./target/kafka-producer-consumer*.jar SSHUSER@CLUSTERNAME-ssh.azurehdinsight.cn:kafka-producer-consumer.jar
    ```

## <a name="run-the-example"></a><a id="run"></a> 运行示例

1. 将 `sshuser` 替换为群集的 SSH 用户，并将 `CLUSTERNAME` 替换为群集的名称。 输入以下命令，打开到群集的 SSH 连接。 出现提示时，输入 SSH 用户帐户的密码。

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.cn
    ```

1. 若要获取 Kafka 代理主机，请替换以下命令中 `<clustername>` 和 `<password>` 的值并执行该命令。 对于 `<clustername>`，请使用如 Azure 门户中所示相同的大小写。 将 `<password>` 替换为群集登录密码，然后执行：

    ```bash
    sudo apt -y install jq
    export clusterName='<clustername>'
    export password='<password>'
    export KAFKABROKERS=$(curl -sS -u admin:$password -G https://$clusterName.azurehdinsight.cn/api/v1/clusters/$clusterName/services/KAFKA/components/KAFKA_BROKER | jq -r '["\(.host_components[].HostRoles.host_name):9092"] | join(",")' | cut -d',' -f1,2);
    ```

    > [!Note]  
    > 此命令需要 Ambari 访问权限。 如果群集位于 NSG 后面，请在可访问 Ambari 的计算机上运行此命令。

1. 通过输入以下命令创建 Kafka 主题 `myTest`：

    ```bash
    java -jar kafka-producer-consumer.jar create myTest $KAFKABROKERS
    ```

7. 若要运行生成者并将数据写入到主题，请使用以下命令：

    ```bash
    java -jar kafka-producer-consumer.jar producer myTest $KAFKABROKERS
    ```

8. 在生成者完成后，使用以下命令从主题中读取：

    ```bash
    java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS
    scp ./target/kafka-producer-consumer*.jar sshuser@CLUSTERNAME-ssh.azurehdinsight.cn:kafka-producer-consumer.jar
    ```

    会显示已读取的记录以及记录计数。

9. 使用 __Ctrl + C__ 退出使用者。

### <a name="multiple-consumers"></a>多个使用者

在读取记录时，Kafka 使用者使用使用者组。 对多个使用者使用相同的组会导致从主题进行负载均衡读取。 组中的每个使用者都会接收一部分记录。

使用者应用程序接受一个用作组 ID 的参数。 例如，以下命令使用组 ID `myGroup` 启动一个使用者：

```bash
java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS myGroup
```

使用 __Ctrl + C__ 退出使用者。
若要在操作中了解此过程，请使用以下命令：

```bash
tmux new-session 'java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS myGroup' \
\; split-window -h 'java -jar kafka-producer-consumer.jar consumer myTest $KAFKABROKERS myGroup' \
\; attach
```

此命令使用 `tmux` 将终端拆分为两列。 每列中都会启动使用者，且具有相同的组 ID 值。 使用者完成读取后，请注意，每个使用者仅读取记录的一部分。 按 Ctrl+C 两次以退出 `tmux`。

同一个组中客户端的使用方式由主题的分区处理。 在此代码示例中，之前创建的 `test` 主题有 8 个分区。 如果启动 8 个使用者，则每个使用者都从主题的单个分区读取记录。

> [!IMPORTANT]
> 使用者组中的使用者实例数不能超过分区数。 此示例中，一个使用者组最多可包含八个使用者，因为这是本主题中的分区数。 也可拥有多个使用者组，每个组的使用者不能超过八个。

Kafka 中存储的记录将按接收顺序存储在分区中。 若要 *在分区中*实现有序的记录传送，可以创建使用者实例数与分区数相匹配的使用者组。 若要 *在主题中*实现有序的记录传送，可以创建仅包含一个使用者实例的使用者组。

## <a name="clean-up-resources"></a>清理资源

若要清理本教程创建的资源，可以删除资源组。 删除资源组也会删除相关联的 HDInsight 群集，以及与资源组相关联的任何其他资源。

若要使用 Azure 门户删除资源组，请执行以下操作：

1. 在 Azure 门户中展开左侧的菜单，打开服务菜单，然后选择“资源组”以显示资源组的列表。
2. 找到要删除的资源组，然后右键单击列表右侧的“更多”按钮 (...)。
3. 选择“删除资源组”，然后进行确认。

## <a name="next-steps"></a>后续步骤

本文档介绍了如何将 Apache Kafka 生成者和使用者 API 与 Kafka on HDInsight 配合使用。 请参阅以下资源了解有关使用 Kafka 的详细信息：

* [使用 Kafka REST 代理](rest-proxy.md)
* [分析 Apache Kafka 日志](apache-kafka-log-analytics-operations-management.md)
