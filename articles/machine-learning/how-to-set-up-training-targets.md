---
title: 使用模型训练的计算目标
titleSuffix: Azure Machine Learning
description: 为机器学习模型训练配置训练环境（计算目标）。 可以轻松地在训练环境之间切换。 在本地开始训练。 如果需要横向扩展，请切换到基于云的计算目标。
services: machine-learning
author: sdgilley
ms.author: sgilley
ms.reviewer: sgilley
ms.service: machine-learning
ms.subservice: core
ms.topic: how-to
ms.date: 06/11/2020
ms.custom: seodec18, tracking-python
ms.openlocfilehash: b47846ffe26f27b6540f5935b857a9e64500c855
ms.sourcegitcommit: 2bd0be625b21c1422c65f20658fe9f9277f4fd7c
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/17/2020
ms.locfileid: "86440888"
---
# <a name="set-up-and-use-compute-targets-for-model-training"></a>设置并使用模型训练的计算目标 
[!INCLUDE [applies-to-skus](../../includes/aml-applies-to-basic-enterprise-sku.md)]

使用 Azure 机器学习可以在不同的资源或环境（统称为[__计算目标__](concept-azure-machine-learning-architecture.md#compute-targets)）中训练模型。 计算目标可以是本地计算机，也可以是云资源，例如 Azure 机器学习计算、Azure HDInsight 或远程虚拟机。  还可以为模型部署创建计算目标，如[“部署模型的位置和方式”](how-to-deploy-and-where.md)中所述。

可以使用 Azure 机器学习 SDK、Azure 机器学习工作室、Azure CLI 或 Azure 机器学习 VS Code 扩展来创建和管理计算目标。 如果通过其他服务（例如 HDInsight 群集）创建了计算目标，可以通过将其附加到 Azure 机器学习工作区来使用它们。
 
本文介绍如何使用各种计算目标进行模型训练。  适用于所有计算目标的步骤遵循相同的工作流：
1. 如果还没有计算目标，请__创建__一个。
2. 将计算目标__附加__到工作区。
3. __配置__计算目标，使其包含脚本所需的 Python 环境和包依赖项。


>[!NOTE]
> 本文中的代码已使用 Azure 机器学习 SDK 版本 1.0.74 进行测试。

## <a name="compute-targets-for-training"></a>训练的计算目标

Azure 机器学习为不同的计算目标提供不同的支持。 典型的模型开发生命周期从开发/试验少量的数据开始。 在此阶段，我们建议使用本地环境。 例如，本地计算机或基于云的 VM。 针对更大的数据集扩展训练或执行分布式训练时，我们建议使用 Azure 机器学习计算来创建可在每次提交运行时自动缩放的单节点或多节点群集。 你也可以附加自己的计算资源，不过，为各种方案提供的支持可能有所不同，详情如下：

[!INCLUDE [aml-compute-target-train](../../includes/aml-compute-target-train.md)]


> [!NOTE]
> Azure 机器学习计算群集可以创建为持久资源，也可以在你请求某个运行时动态创建。 基于运行的创建在训练运行完成后会删除计算目标，因此无法重复使用以这种方式创建的计算目标。

## <a name="whats-a-run-configuration"></a>什么是运行配置？

训练时，通常会在本地计算机上开始，然后在不同的计算目标上运行该训练脚本。 使用 Azure 机器学习，你可以在各种计算目标上运行脚本，而无需更改脚本。

只需在运行配置中为每个计算目标定义环境即可。  然后，当你想要在不同的计算目标上运行训练试验时，可以指定该计算的运行配置。 有关指定环境并将其绑定到运行配置的详细信息，请参阅[创建和管理用于训练和部署的环境](how-to-use-environments.md)。

本文的最后详细介绍了如何[提交试验](#submit)。

## <a name="whats-an-estimator"></a>什么是评估器？

为了便于使用常用框架进行模型训练，Azure 机器学习 Python SDK 提供了另一种可选择的高级抽象：估算器类。  使用此类，你可以轻松地构造运行配置。 可以创建一个泛型[估算器](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.estimator?view=azure-ml-py)，并通过该估算器提交使用你选择的任何学习框架（例如 scikit-learn）的训练脚本。 建议使用估算器进行训练，因为它会自动构造嵌入对象（如环境或 RunConfiguration 对象）。 如果你希望能够更好地控制这些对象的创建方式并指定要为试验运行安装哪些包，请遵循[这些步骤](#amlcompute)，以使用 Azure 机器学习计算中的 RunConfiguration 对象提交训练试验。

Azure 机器学习提供适用于 [PyTorch](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.pytorch?view=azure-ml-py)、[TensorFlow](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.tensorflow?view=azure-ml-py)、[Chainer](https://docs.microsoft.com/python/api/azureml-train-core/azureml.train.dnn.chainer?view=azure-ml-py) 和 [Ray RLlib](how-to-use-reinforcement-learning.md) 的特定估算器。

有关详细信息，请参阅[使用估算器训练 ML 模型](how-to-train-ml-models.md)。

## <a name="whats-an-ml-pipeline"></a>什么是 ML 管道？

使用 ML 管道，可以在简洁性、速度、可移植性和重用性方面优化工作流。 使用 Azure 机器学习来生成管道时，你可以专注于自己的专业，即机器学习，而不需要关注基础设施和自动化。

ML 管道由多个步骤构造而成，步骤是管道中的独特计算单位。 每个步骤都可以独立运行，并使用独立的计算资源。 通过这种方法，多名数据科学家可以同时处理同一个管道且不过度消耗计算资源，此外还可以轻松对每个步骤使用不同的计算类型/大小。

> [!TIP]
> 训练模型时，ML 管道可以使用运行配置或估算器。

ML 管道不仅可以训练模型，还可以在训练之前准备数据并在训练之后部署模型。 管道的主要用例之一是批量评分。 有关详细信息，请参阅[管道：优化机器学习工作流](concept-ml-pipelines.md)。

## <a name="set-up-in-python"></a>使用 Python 进行设置

使用以下部分配置这些计算目标：

* [本地计算机](#local)
* [Azure 机器学习计算群集](#amlcompute)
* [Azure 机器学习计算实例](#instance)
* [远程虚拟机](#vm)
* [Azure HDInsight](#hdinsight)


### <a name="local-computer"></a><a id="local"></a>本地计算机

1. **创建和附加**：无需创建或附加计算目标即可将本地计算机用作训练环境。  

1. **配置**：将本地计算机用作计算目标时，训练代码将在[开发环境](how-to-configure-environment.md)中运行。  如果该环境已包含所需的 Python 包，请使用用户管理的环境。

```Python
from azureml.core.runconfig import RunConfiguration

# Edit a run configuration property on the fly.
run_local = RunConfiguration()

run_local.environment.python.user_managed_dependencies = True
```

附加计算并配置运行后，下一步是[提交训练运行](#submit)。

### <a name="azure-machine-learning-compute-cluster"></a><a id="amlcompute"></a>Azure 机器学习计算群集

Azure 机器学习计算群集是一个托管的计算基础结构，可让你轻松创建单节点或多节点计算。 该计算是在工作区区域内部创建的，是可与工作区中的其他用户共享的资源。 提交作业时，计算会自动扩展，并可以放入 Azure 虚拟网络。 计算在容器化环境中执行，将模型的依赖项打包在 [Docker 容器](https://www.docker.com/why-docker)中。

可以使用 Azure 机器学习计算在云中的 CPU 或 GPU 计算节点群集之间分配训练进程。 有关包括 GPU 的 VM 大小的详细信息，请参阅 [GPU 优化的虚拟机大小](https://docs.microsoft.com/azure/virtual-machines/linux/sizes-gpu)。 

Azure 机器学习计算对可以分配的核心数等属性实施默认限制。 有关详细信息，请参阅[管理和请求 Azure 资源的配额](https://docs.microsoft.com/azure/machine-learning/how-to-manage-quotas)。

还可以选择使用低优先级 VM 来运行部分或所有工作负载。 这些 VM 的可用性未得到保证，在使用时可能会被占用。 对于被占用的作业，在重新可用时将重新启动，而不是直接续用。  与普通 VM 相比，低优先级 VM 提供折扣费率，请参阅[计划和管理成本](https://docs.microsoft.com/azure/machine-learning/concept-plan-manage-cost)。

> [!TIP]
> 一般情况下，只要所需核心数方面的配额足够，群集就可以扩展到多达 100 个节点。 默认情况下，设置群集时会启用群集节点之间的通信（例如，为了支持 MPI 作业）。 但是，还可以将群集扩展到数千个节点，只需[提交支持票证](https://portal.azure.cn/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest)，并请求将你的订阅、工作区或特定群集加入允许列表以禁用节点间通信即可。 

可在不同的运行中重复使用 Azure 机器学习计算。 计算可与工作区中的其他用户共享，并在每次运行之后保留，它会根据提交的运行数以及群集上设置的 max_nodes 自动纵向扩展或缩减节点。 min_nodes 设置控制可用节点数的下限。

[!INCLUDE [min-nodes-note](../../includes/machine-learning-min-nodes.md)]

1. **创建和附加**：若要在 Python 中创建持久性 Azure 机器学习计算资源，请指定 **vm_size** 和 **max_nodes** 属性。 然后，Azure 机器学习将对其他属性使用智能默认值。 计算在不使用时自动缩减为零个节点。   按需创建专用 VM 来运行作业。
    
    * **vm_size**：Azure 机器学习计算创建的节点的 VM 系列。
    * **max_nodes**：在 Azure 机器学习计算中运行作业时自动扩展到的最大节点数。
    
    ```Python
    from azureml.core.compute import ComputeTarget, AmlCompute
    from azureml.core.compute_target import ComputeTargetException

    # Choose a name for your CPU cluster
    cpu_cluster_name = "cpucluster"

    # Verify that cluster does not exist already
    try:
        cpu_cluster = ComputeTarget(workspace=ws, name=cpu_cluster_name)
        print('Found existing cluster, use it.')
    except ComputeTargetException:
        compute_config = AmlCompute.provisioning_configuration(vm_size='STANDARD_D2_V2',
                                                            max_nodes=4)
        cpu_cluster = ComputeTarget.create(ws, cpu_cluster_name, compute_config)

    cpu_cluster.wait_for_completion(show_output=True)
    ```

   还可以在创建 Azure 机器学习计算时配置多个高级属性。 使用这些属性可以创建固定大小的持久性群集，或者在订阅中的现有 Azure 虚拟网络内创建持久性群集。  有关详细信息，请参阅 [AmlCompute 类](https://docs.microsoft.com/python/api/azureml-core/azureml.core.compute.amlcompute.amlcompute?view=azure-ml-py
    )。
    
   或者，可以[在 Azure 机器学习工作室](#portal-create)中创建并附加持久性 Azure 机器学习计算资源。

1. **配置**：为持久性计算目标创建运行配置。

   ```Python
   from azureml.core.runconfig import RunConfiguration
    from azureml.core.conda_dependencies import CondaDependencies
    from azureml.core.runconfig import DEFAULT_CPU_IMAGE

    # Create a new runconfig object
    run_amlcompute = RunConfiguration()

    # Use the cpu_cluster you created above. 
    run_amlcompute.target = cpu_cluster

    # Enable Docker
    run_amlcompute.environment.docker.enabled = True

    # Set Docker base image to the default CPU-based image
    run_amlcompute.environment.docker.base_image = DEFAULT_CPU_IMAGE

    # Use conda_dependencies.yml to create a conda environment in the Docker image for execution
    run_amlcompute.environment.python.user_managed_dependencies = False

    # Specify CondaDependencies obj, add necessary packages
    run_amlcompute.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'])
    ```

附加计算并配置运行后，下一步是[提交训练运行](#submit)。


### <a name="azure-machine-learning-compute-instance"></a><a id="instance"></a>Azure 机器学习计算实例

[Azure 机器学习计算实例](concept-compute-instance.md)是一个托管的计算基础结构，可让你轻松创建单个 VM。 该计算是在工作区区域内创建的，但与计算群集不同，实例不能与工作区中的其他用户共享。 此外，实例不会自动纵向缩减。  你必须停止该资源，防止连续不断地产生费用。

计算实例可以并行运行多个作业，它有一个作业队列。 

计算实例可以在[虚拟网络环境](how-to-enable-virtual-network.md#compute-instance)中安全地运行作业，无需企业打开 SSH 端口。 作业在容器化环境中执行，并将模型依赖项打包到 Docker 容器中。 

1. **创建和附加**： 
    
   ```Python
   import datetime
    import time

    from azureml.core.compute import ComputeTarget, ComputeInstance
    from azureml.core.compute_target import ComputeTargetException

    # Choose a name for your instance
    compute_name = "compute-instance"

    # Verify that instance does not exist already
    try:
        instance = ComputeInstance(workspace=ws, name=compute_name)
        print('Found existing instance, use it.')
    except ComputeTargetException:
        compute_config = ComputeInstance.provisioning_configuration(
            vm_size='STANDARD_D3_V2',
            ssh_public_access=False,
            # vnet_resourcegroup_name='<my-resource-group>',
            # vnet_name='<my-vnet-name>',
            # subnet_name='default',
            # admin_user_ssh_public_key='<my-sshkey>'
        )
        instance = ComputeInstance.create(ws, compute_name, compute_config)
        instance.wait_for_completion(show_output=True)
    ```

1. **配置**：创建运行配置。
    
    ```python
    
    from azureml.core import ScriptRunConfig
    from azureml.core.runconfig import DEFAULT_CPU_IMAGE
    
    src = ScriptRunConfig(source_directory='', script='train.py')
    
    # Set compute target to the one created in previous step
    src.run_config.target = instance
    
    # Set environment
    src.run_config.environment = myenv
     
    run = experiment.submit(config=src)
    ```

有关适用于计算实例的更多命令，请参阅笔记本 [train-on-computeinstance](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training/train-on-computeinstance/train-on-computeinstance.ipynb)。 此笔记本也可在“training/train-on-computeinstance” 中的工作室 **Samples** 文件夹中找到。

附加计算并配置运行后，下一步是[提交训练运行](#submit)


### <a name="remote-virtual-machines"></a><a id="vm"></a>远程虚拟机

Azure 机器学习还支持将自己的计算资源附加到工作区。 任意远程 VM（只要可从 Azure 机器学习访问）都是这种资源类型。 该资源可以是 Azure VM，也可以是组织内部或本地的远程服务器。 具体而言，在指定 IP 地址和凭据（用户名和密码，或 SSH 密钥）的情况下，可以使用任何可访问的 VM 进行远程运行。

可以使用系统生成的 conda 环境、现有的 Python 环境或 Docker 容器。 若要在 Docker 容器中执行，必须在 VM 上运行 Docker 引擎。 需要一个比本地计算机更灵活的基于云的开发/试验环境时，此功能特别有用。

请对此方案使用 Data Science Virtual Machine (DSVM) 作为 Azure VM。 此 VM 在 Azure 中预配置了数据科学和 AI 开发环境。 此 VM 提供精选的工具和框架用于满足整个机器学习开发生命周期的需求。 有关如何将 DSVM 与 Azure 机器学习配合使用的详细信息，请参阅[配置开发环境](https://docs.microsoft.com/azure/machine-learning/how-to-configure-environment#dsvm)。

1. **创建**：创建 DSVM，然后使用它来训练模型。 若要创建此资源，请参阅[预配适用于 Linux (Ubuntu) 的 Data Science Virtual Machine](https://docs.microsoft.com/azure/machine-learning/data-science-virtual-machine/dsvm-ubuntu-intro)。

    > [!WARNING]
    > Azure 机器学习仅支持运行 Ubuntu 的虚拟机。 创建 VM 或选择现有 VM 时，必须选择使用 Ubuntu 的 VM。
    > 
    > Azure 机器学习还要求虚拟机具有公共 IP 地址。

1. **附加**：若要附加现有虚拟机作为计算目标，必须提供虚拟机的资源 ID、用户名和密码。 可以使用订阅 ID、资源组名称和 VM 名称按以下字符串格式构造 VM 的资源 ID：`/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Compute/virtualMachines/<vm_name>`

 
   ```python
   from azureml.core.compute import RemoteCompute, ComputeTarget

   # Create the compute config 
   compute_target_name = "attach-dsvm"
   
   attach_config = RemoteCompute.attach_configuration(resource_id='<resource_id>',
                                                   ssh_port=22,
                                                   username='<username>',
                                                   password="<password>")

   # Attach the compute
   compute = ComputeTarget.attach(ws, compute_target_name, attach_config)

   compute.wait_for_completion(show_output=True)
   ```

   或者，可以[使用 Azure 机器学习工作室](#portal-reuse)将 DSVM 附加到工作区。

1. **配置**：为 DSVM 计算目标创建运行配置。 Docker 与 conda 用于在 DSVM 上创建和配置训练环境。

   ```Python
   import azureml.core
    from azureml.core.runconfig import RunConfiguration
    from azureml.core.conda_dependencies import CondaDependencies

    run_dsvm = RunConfiguration(framework = "python")

    # Set the compute target to the Linux DSVM
    run_dsvm.target = compute_target_name 

    # Use Docker in the remote VM
    run_dsvm.environment.docker.enabled = True

    # Use the CPU base image 
    # To use GPU in DSVM, you must also use the GPU base Docker image "azureml.core.runconfig.DEFAULT_GPU_IMAGE"
    run_dsvm.environment.docker.base_image = azureml.core.runconfig.DEFAULT_CPU_IMAGE
    print('Base Docker image is:', run_dsvm.environment.docker.base_image)

    # Specify the CondaDependencies object
    run_dsvm.environment.python.conda_dependencies = CondaDependencies.create(conda_packages=['scikit-learn'])
    ```


附加计算并配置运行后，下一步是[提交训练运行](#submit)。

### <a name="azure-hdinsight"></a><a id="hdinsight"></a>Azure HDInsight 

Azure HDInsight 是用于大数据分析的热门平台。 该平台提供的 Apache Spark 可用于训练模型。

1. **创建**：先创建 HDInsight 群集，然后使用它来训练模型。 若要在 HDInsight 群集中创建 Spark，请参阅[在 HDInsight 中创建 Spark 群集](https://docs.microsoft.com/azure/hdinsight/spark/apache-spark-jupyter-spark-sql)。 

    > [!WARNING]
    > Azure 机器学习要求 HDInsight 群集具有公共 IP 地址。

    创建群集时，必须指定 SSH 用户名和密码。 请记下这些值，因为在将 HDInsight 用作计算目标时需要用到这些值。
    
    创建群集后，使用主机名 \<clustername>-ssh.azurehdinsight.net 连接到该群集，其中，\<clustername> 是为该群集提供的名称。 

1. **附加**：若要将 HDInsight 群集附加为计算目标，必须提供该 HDInsight 群集的资源 ID、用户名和密码。 可以使用订阅 ID、资源组名称和 HDInsight 群集名称按以下字符串格式构造 HDInsight 群集的资源 ID：`/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.HDInsight/clusters/<cluster_name>`

    ```python
   from azureml.core.compute import ComputeTarget, HDInsightCompute
   from azureml.exceptions import ComputeTargetException

   try:
    # if you want to connect using SSH key instead of username/password you can provide parameters private_key_file and private_key_passphrase

    attach_config = HDInsightCompute.attach_configuration(resource_id='<resource_id>',
                                                          ssh_port=22, 
                                                          username='<ssh-username>', 
                                                          password='<ssh-pwd>')
    hdi_compute = ComputeTarget.attach(workspace=ws, 
                                       name='myhdi', 
                                       attach_configuration=attach_config)

   except ComputeTargetException as e:
    print("Caught = {}".format(e.message))

   hdi_compute.wait_for_completion(show_output=True)
   ```

   或者，可以[使用 Azure 机器学习工作室](#portal-reuse)将 HDInsight 群集附加到工作区。

1. **配置**：为 HDI 计算目标创建运行配置。 

   ```Python
   from azureml.core.runconfig import RunConfiguration
    from azureml.core.conda_dependencies import CondaDependencies


    # use pyspark framework
    run_hdi = RunConfiguration(framework="pyspark")

    # Set compute target to the HDI cluster
    run_hdi.target = hdi_compute.name

    # specify CondaDependencies object to ask system installing numpy
    cd = CondaDependencies()
    cd.add_conda_package('numpy')
    run_hdi.environment.python.conda_dependencies = cd
    ```


附加计算并配置运行后，下一步是[提交训练运行](#submit)。


### <a name="azure-batch"></a><a id="azbatch"></a>Azure Batch 

Azure Batch 用于在云中高效运行大规模并行高性能计算 (HPC) 应用程序。 可以在 Azure 机器学习管道中使用 AzureBatchStep 将作业提交到 Azure Batch 计算机池。

若要将 Azure Batch 附加为计算目标，必须使用 Azure 机器学习 SDK 并提供以下信息：

-    **Azure Batch 计算名称**：在工作区中用于计算的易记名称
-    **Azure Batch 帐户名称**：Azure Batch 帐户的名称
-    **资源组**：包含 Azure Batch 帐户的资源组。

以下代码演示如何将 Azure Batch 附加为计算目标：

```python
from azureml.core.compute import ComputeTarget, BatchCompute
from azureml.exceptions import ComputeTargetException

# Name to associate with new compute in workspace
batch_compute_name = 'mybatchcompute'

# Batch account details needed to attach as compute to workspace
batch_account_name = "<batch_account_name>"  # Name of the Batch account
# Name of the resource group which contains this account
batch_resource_group = "<batch_resource_group>"

try:
    # check if the compute is already attached
    batch_compute = BatchCompute(ws, batch_compute_name)
except ComputeTargetException:
    print('Attaching Batch compute...')
    provisioning_config = BatchCompute.attach_configuration(
        resource_group=batch_resource_group, account_name=batch_account_name)
    batch_compute = ComputeTarget.attach(
        ws, batch_compute_name, provisioning_config)
    batch_compute.wait_for_completion()
    print("Provisioning state:{}".format(batch_compute.provisioning_state))
    print("Provisioning errors:{}".format(batch_compute.provisioning_errors))

print("Using Batch compute:{}".format(batch_compute.cluster_resource_id))
```

## <a name="set-up-in-azure-machine-learning-studio"></a>在 Azure 机器学习工作室中设置

可以在 Azure 机器学习工作室中访问与工作区关联的计算目标。  可以使用工作室执行以下操作：

* [查看](#portal-view)附加到工作区的计算目标
* 在工作区中[创建计算目标](#portal-create)
* [附加](#portal-reuse)在工作区外部创建的计算目标


创建目标并将其附加到工作区后，可以通过一个 `ComputeTarget` 对象在运行配置中使用该目标： 

```python
from azureml.core.compute import ComputeTarget
myvm = ComputeTarget(workspace=ws, name='my-vm-name')
```

### <a name="view-compute-targets"></a><a id="portal-view"></a>查看计算目标


若要查看工作区的计算目标，请使用以下步骤：

1. 导航到 [Azure 机器学习工作室](https://ml.azure.com)。
 
1. 在“应用程序”下，选择“计算”。 

    [![查看“计算”选项卡](./media/how-to-set-up-training-targets/azure-machine-learning-service-workspace.png)](./media/how-to-set-up-training-targets/azure-machine-learning-service-workspace-expanded.png)

### <a name="create-a-compute-target"></a><a id="portal-create"></a>创建计算目标

遵循上述步骤查看计算目标的列表。 然后使用以下步骤创建计算目标： 

1. 选择加号 (+) 添加计算目标。

    ![添加计算目标](./media/how-to-set-up-training-targets/add-compute-target.png) 

1. 输入计算目标的名称。 

1. 选择“机器学习计算”作为用于训练的计算类型。 

    >[!NOTE]
    >Azure 机器学习计算是可在 Azure 机器学习工作室中创建的唯一一种托管计算资源。  创建其他所有计算资源后，可以附加这些资源。

1. 填写表单。 提供必需属性的值，尤其是“VM 系列”，以及用于运转计算的**最大节点数**。  

1. 选择“创建” 。


1. 通过在列表中选择计算目标来查看创建操作的状态：

    ![选择一个计算目标以查看创建操作的状态](./media/how-to-set-up-training-targets/View_list.png)

1. 然后会看到相应计算目标的详细信息： 

    ![查看计算目标详细信息](./media/how-to-set-up-training-targets/compute-target-details.png) 

### <a name="attach-compute-targets"></a><a id="portal-reuse"></a>附加计算目标

若要使用在 Azure 机器学习工作区外部创建的计算目标，必须附加这些计算目标。 附加计算目标会使其可供你的工作区使用。

遵循上述步骤查看计算目标的列表。 然后使用以下步骤来附加计算目标： 

1. 选择加号 (+) 添加计算目标。 
1. 输入计算目标的名称。 
1. 选择要为训练附加的计算类型：

    > [!IMPORTANT]
    > 并非所有计算类型都可以从 Azure 机器学习工作室附加。 目前，可为训练附加的计算类型包括：
    >
    > * 远程 VM
    > * Azure Databricks（在机器学习管道中使用）
    > * Azure Data Lake Analytics（在机器学习管道中使用）
    > * Azure HDInsight

1. 填写表单，并提供必需属性的值。

    > [!NOTE]
    > Microsoft 建议使用 SSH 密钥，因为它们比密码更安全。 密码很容易受到暴力破解攻击。 SSH 密钥依赖于加密签名。 若要了解如何创建用于 Azure 虚拟机的 SSH 密钥，请参阅以下文档：
    >
    > * [在 Linux 或 macOS 上创建和使用 SSH 密钥](/virtual-machines/linux/mac-create-ssh-keys)
    > * [在 Windows 上创建和使用 SSH 密钥](/virtual-machines/linux/ssh-from-windows)

1. 选择“附加”。 
1. 通过在列表中选择计算目标来查看附加操作的状态。

## <a name="set-up-with-cli"></a>使用 CLI 进行设置

可以使用适用于 Azure 机器学习的 [CLI 扩展](reference-azure-machine-learning-cli.md)访问与工作区关联的计算目标。  可以使用 CLI 执行以下操作：

* 创建托管计算目标
* 更新托管计算目标
* 附加托管计算目标

有关详细信息，请参阅[资源管理](reference-azure-machine-learning-cli.md#resource-management)。

## <a name="set-up-with-vs-code"></a>使用 VS Code 进行设置

可以使用适用于 Azure 机器学习的 [VS Code 扩展](tutorial-train-deploy-image-classification-model-vscode.md#configure-compute-targets)访问、创建和管理与工作区关联的计算目标。

## <a name="submit-training-run-using-azure-machine-learning-sdk"></a><a id="submit"></a>使用 Azure 机器学习 SDK 提交训练运行

创建运行配置后，可以使用它来运行试验。  对于所有类型的计算目标，用于提交训练运行的代码模式都是相同的：

1. 创建要运行的试验
1. 提交运行任务。
1. 等待运行任务完成。

> [!IMPORTANT]
> 提交训练运行时，将创建包含训练脚本的目录的快照，并将其发送到计算目标。 目录快照也作为试验的一部分存储在工作区中。 如果更改文件并再次提交运行，只会上传已更改的文件。
>
> [!INCLUDE [amlinclude-info](../../includes/machine-learning-amlignore-gitignore.md)]
> 
> 有关详细信息，请参阅[快照](concept-azure-machine-learning-architecture.md#snapshots)。

### <a name="create-an-experiment"></a>创建试验

首先，在工作区中创建一个试验。

```Python
from azureml.core import Experiment
experiment_name = 'my_experiment'

exp = Experiment(workspace=ws, name=experiment_name)
```

### <a name="submit-the-experiment"></a>提交试验

使用 `ScriptRunConfig` 对象提交该试验。  此对象包含：

* **source_directory**：包含训练脚本的源目录
* **script**：标识训练脚本
* **run_config**：运行配置，其中定义训练位置。

例如，若要使用[本地目标](#local)配置：

```Python
from azureml.core import ScriptRunConfig
import os 

script_folder = os.getcwd()
src = ScriptRunConfig(source_directory = script_folder, script = 'train.py', run_config = run_local)
run = exp.submit(src)
run.wait_for_completion(show_output = True)
```

使用不同的运行配置（例如 [amlcompute 目标](#amlcompute)）将同一试验切换为在不同的计算目标中运行：

```Python
from azureml.core import ScriptRunConfig

src = ScriptRunConfig(source_directory = script_folder, script = 'train.py', run_config = run_amlcompute)
run = exp.submit(src)
run.wait_for_completion(show_output = True)
```

> [!TIP]
> 此示例默认为仅使用计算目标的一个节点进行训练。 若要使用多个节点，请将运行配置的 `node_count` 设置为所需的节点数。 例如，下面的代码将用于训练的节点数设置为 4：
>
> ```python
> src.run_config.node_count = 4
> ```

或者可以：

* 根据[使用评估器训练机器学习模型](how-to-train-ml-models.md)中所述，使用 `Estimator` 对象提交试验。
* 提交用于[超参数优化](how-to-tune-hyperparameters.md)的 HyperDrive 运行。
* 通过 [VS Code 扩展](tutorial-train-deploy-image-classification-model-vscode.md#train-the-model)提交试验。

有关详细信息，请参阅 [ScriptRunConfig](https://docs.microsoft.com/python/api/azureml-core/azureml.core.scriptrunconfig?view=azure-ml-py) 和 [RunConfiguration](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfiguration?view=azure-ml-py) 文档。

## <a name="create-run-configuration-and-submit-run-using-azure-machine-learning-cli"></a>使用 Azure 机器学习 CLI 创建运行配置并提交运行

可以使用 [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 和[机器学习 CLI 扩展](reference-azure-machine-learning-cli.md)针对不同的计算目标创建运行配置并提交运行。 以下示例假设你已有 Azure 机器学习工作区，并且已使用 `az login` CLI 命令登录到 Azure。 

[!INCLUDE [select-subscription](../../includes/machine-learning-cli-subscription.md)] 

### <a name="create-run-configuration"></a>创建运行配置

创建运行配置的最简单方法是导航到包含机器学习 Python 脚本的文件夹，并使用 CLI 命令

```azurecli
az ml folder attach
```

此命令将创建子文件夹 `.azureml`，其中包含不同计算目标的模板运行配置文件。 可以复制和编辑这些文件以自定义配置，例如添加 Python 包或更改 Docker 设置。  

### <a name="structure-of-run-configuration-file"></a>运行配置文件的结构

运行配置文件的格式为 YAML，包括以下部分
 * 要运行的脚本及其参数
 * 计算目标名称，可以是“local”，也可以是工作区下的计算名称。
 * 用于执行运行的参数：框架、用于分布式运行的通信器、最长持续时间和计算节点数。
 * 环境部分。 有关本部分中各字段的详细信息，请参阅[创建和管理用于训练和部署的环境](how-to-use-environments.md)。
   * 若要为运行指定要安装的 Python 包，请创建 [conda 环境文件](https://docs.conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html#create-env-file-manually)，并设置 condaDependenciesFile 字段。
 * 运行历史记录详细信息，以指定日志文件文件夹，并指定启用或禁用输出集合及运行历史记录快照。
 * 特定于所选框架的配置详细信息。
 * 数据引用和数据存储详细信息。
 * 用于创建新群集的机器学习计算特定的配置详细信息。

若要了解完整的运行配置架构，请参阅示例 [JSON 文件](https://github.com/microsoft/MLOps/blob/b4bdcf8c369d188e83f40be8b748b49821f71cf2/infra-as-code/runconfigschema.json)。

### <a name="create-an-experiment"></a>创建试验

首先，为运行创建试验

```azurecli
az ml experiment create -n <experiment>
```

### <a name="script-run"></a>脚本运行

若要提交脚本运行，请执行命令

```azurecli
az ml run submit-script -e <experiment> -c <runconfig> my_train.py
```

### <a name="hyperdrive-run"></a>HyperDrive 运行

可以结合使用 HyperDrive 与 Azure CLI 以执行参数优化运行。 首先，创建采用以下格式的 HyperDrive 配置文件。 有关超参数优化参数的详细信息，请参阅[优化模型的超参数](how-to-tune-hyperparameters.md)一文。

```yml
# hdconfig.yml
sampling: 
    type: random # Supported options: Random, Grid, Bayesian
    parameter_space: # specify a name|expression|values tuple for each parameter.
    - name: --penalty # The name of a script parameter to generate values for.
      expression: choice # supported options: choice, randint, uniform, quniform, loguniform, qloguniform, normal, qnormal, lognormal, qlognormal
      values: [0.5, 1, 1.5] # The list of values, the number of values is dependent on the expression specified.
policy: 
    type: BanditPolicy # Supported options: BanditPolicy, MedianStoppingPolicy, TruncationSelectionPolicy, NoTerminationPolicy
    evaluation_interval: 1 # Policy properties are policy specific. See the above link for policy specific parameter details.
    slack_factor: 0.2
primary_metric_name: Accuracy # The metric used when evaluating the policy
primary_metric_goal: Maximize # Maximize|Minimize
max_total_runs: 8 # The maximum number of runs to generate
max_concurrent_runs: 2 # The number of runs that can run concurrently.
max_duration_minutes: 100 # The maximum length of time to run the experiment before cancelling.
```

将此文件与运行配置文件一起添加。 然后使用以下命令提交 HyperDrive 运行：
```azurecli
az ml run submit-hyperdrive -e <experiment> -c <runconfig> --hyperdrive-configuration-name <hdconfig> my_train.py
```

请注意运行配置中的参数部分和 HyperDrive 配置中的参数空间 。这些参数包含要传递给训练脚本的命令行参数。 在每次迭代中，运行配置中的值保持不变，而 HyperDrive 配置中的范围将会更迭。 请不要在这两个文件中指定相同的参数。

有关这些 ```az ml``` CLI 命令的详细信息，请参阅[参考文档](reference-azure-machine-learning-cli.md)。

<a id="gitintegration"></a>

## <a name="git-tracking-and-integration"></a>Git 跟踪和集成

如果以本地 Git 存储库作为源目录开始训练运行，有关存储库的信息将存储在运行历史记录中。 有关详细信息，请参阅 [Azure 机器学习的 Git 集成](concept-train-model-git-integration.md)。

## <a name="notebook-examples"></a>Notebook 示例

请参阅以下 Notebook，其中提供了有关使用各种计算目标进行训练的示例：
* [how-to-use-azureml/training](https://github.com/Azure/MachineLearningNotebooks/blob/master/how-to-use-azureml/training)
* [tutorials/img-classification-part1-training.ipynb](https://github.com/Azure/MachineLearningNotebooks/blob/master/tutorials/image-classification-mnist-data/img-classification-part1-training.ipynb)

[!INCLUDE [aml-clone-in-azure-notebook](../../includes/aml-clone-for-examples.md)]

## <a name="next-steps"></a>后续步骤

* [教程：训练模型](tutorial-train-models-with-aml.md)使用一个托管计算目标来训练模型。
* 若要构建更好的模型，请了解如何[高效地优化超参数](how-to-tune-hyperparameters.md)。
* 训练模型后，了解[如何以及在何处部署模型](how-to-deploy-and-where.md)。
* 查看 [RunConfiguration 类](https://docs.microsoft.com/python/api/azureml-core/azureml.core.runconfig.runconfiguration?view=azure-ml-py) SDK 参考。
* [通过 Azure 虚拟网络使用 Azure 机器学习](how-to-enable-virtual-network.md)
