---
title: 最佳实践
description: 了解开发 Azure Batch 解决方案的最佳做法和有用技巧。
ms.author: v-tawe
origin.date: 05/22/2020
ms.date: 06/29/2020
ms.topic: conceptual
ms.openlocfilehash: c19c836fba99d9ae6befb5b1c20fabfc3671f035
ms.sourcegitcommit: d24e12d49708bbe78db450466eb4fccbc2eb5f99
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/01/2020
ms.locfileid: "85613314"
---
# <a name="azure-batch-best-practices"></a>Azure Batch 最佳做法

本文介绍了有效且高效地使用 Azure Batch 服务的最佳做法集合，这些做法基于使用 Batch 的实际体验。 阅读本文，以避免在开发和使用 Batch 时出现设计缺陷、潜在性能问题和反模式。

## <a name="pools"></a>池

[池](nodes-and-pools.md#pools)是用于在 Batch 服务上执行作业的计算资源。 以下各部分提供了有关使用 Batch 池的建议。

### <a name="pool-configuration-and-naming"></a>池配置和命名

- **池分配模式** 创建 Batch 帐户时，可以在两种池分配模式之间进行选择：**Batch 服务**或**用户订阅**。 在大部分情况下，应使用默认的 Batch 服务模式，使用此模式时，池在幕后在 Batch 托管的订阅中分配。 在备用的“用户订阅”模式下，会在创建池后直接在订阅中创建 Batch VM 和其他资源。 用户订阅帐户主要用于实现重要但却不太多见的方案。 有关用户订阅模式的详细信息，请参阅[用户订阅模式的其他配置](batch-account-create-portal.md#additional-configuration-for-user-subscription-mode)。

- **确定用于池映射的作业时考虑作业和任务运行时间。**
    如果作业主要包括短时间运行的任务，且预期的任务总计数较小，因此作业的总预期运行时间不长，那么，请不要为每个作业分配新池。 节点的分配时间会缩减作业运行时间。

- **池应包含多个计算节点。**
    不保证各个节点始终可用。 硬件故障、操作系统更新和其他许多问题虽然不太常见，但它们可能会导致个别节点脱机。 如果 Batch 工作负荷需要具有确定性且有保证的进度，则你应该分配包含多个节点的池。

- **不要重复使用资源名称。**
    我们往往会不断地分配和解除 Batch 资源（作业、池等）。 例如，你可能会在星期一创建一个池，在星期二将它删除，然后在星期四又创建一个池。 应为创建的每个新资源指定一个以前从未用过的唯一名称。 为此，可以使用 GUID（作为整个资源名称或其中的一部分），或者在资源名称中嵌入资源的创建时间。 Batch 支持 [DisplayName](/dotnet/api/microsoft.azure.batch.jobspecification.displayname?view=azure-dotnet)，使用此属性可为资源指定一个用户可读的名称，即使实际资源 ID 不够用户友好。 使用唯一名称可以更方便地区分哪个特定资源在日志和指标中产生了影响。 如果需要针对某个资源提交支持案例，唯一名称还可以消除不明确性。

- **池维护和故障期间的连续性。**
    最好是让作业动态使用池。 如果作业将同一个池用于所有用途，在该池出现问题时，作业有可能无法运行。 这对于时间敏感型工作负载尤其重要。 若要解决此问题，请在计划每个作业时动态选择或创建池，或通过某种方式替代池名称，以便可以绕过运行不正常的池。

- **池维护和故障期间的业务连续性** 有许多可能的原因（例如内部错误、容量约束等）会阻止池缩放到所需的大小。出于此原因，应当做好相应准备，以便在必要时将作业目标重新定为不同的池（也许可以使用不同的 VM 大小 - Batch 通过 [UpdateJob](/dotnet/api/microsoft.azure.batch.protocol.joboperationsextensions.update?view=azure-dotnet) 实现此目的）。 避免使用预计永远不会删除或更改的静态池 ID。

### <a name="pool-lifetime-and-billing"></a>池生存期和计费

池生存期可能根据分配方法和应用于池配置的选项而有所不同。 池可以具有任意生存期，并且在任意时间点，池中的计算节点数可能会变化。 你需要负责显式管理池中的计算节点，或者通过服务提供的功能（自动缩放或自动汇集）进行管理。

- **使池保持最新状态。**
    应每隔几个月将池的大小调整为零，以确保获得最新的节点代理更新和 bug 修复。 除非重新创建池或者将其大小调整为 0 个计算节点，否则池不会接收节点代理更新。 重新创建池或调整池大小之前，建议根据[节点](#nodes)部分中所述，下载所有节点代理日志进行调试。

- **重新创建池** 同样需要注意的是，不建议每天删除再重新创建池。 应该创建新池，并将现有作业更新为指向新池。 将所有任务移到新池后删除旧池。

- **池效率和计费** Batch 本身不会产生额外的费用，但使用的计算资源确实会产生费用。 你需要为池中的每个计算节点付费，不管这些节点处于何种状态。 这包括运行节点所需的任何费用，例如存储和网络成本。 若要详细了解最佳做法，请参阅 [Azure Batch 的成本分析和预算](budget.md)。

### <a name="pool-allocation-failures"></a>池分配失败

在首次分配或后续的调整大小期间，可能会发生池分配失败。 可能的原因是区域中出现了暂时性的容量耗尽，或 Batch 所依赖的其他 Azure 服务发生故障。 核心配额没有保障，而且存在限制。

### <a name="unplanned-downtime"></a>计划外停机

Azure 中的 Batch 池可能会遇到停机事件。 在规划和开发 Batch 方案或工作流时，请牢记这一点。

如果某个节点发生故障，Batch 会代表你自动尝试恢复这些计算节点。 这可能会触发在恢复的节点上重新计划任何正在运行的任务。 若要详细了解中断的任务，请参阅[重试设计](#design-for-retries-and-re-execution)。

### <a name="azure-region-dependency"></a>Azure 区域依赖项

如果你有时间敏感型工作负载或生产工作负载，建议不要依赖单个 Azure 区域。 有些问题虽然极少出现，但可能会影响整个区域。 例如，如果处理需要在特定的时间开始，请考虑在开始之前的相当长一段时间纵向扩展主要区域中的池。 如果扩展该池失败，可以回退并在一个或多个备份区域中纵向扩展池。 如果另一个池出现问题，跨不同区域中多个帐户的池可提供一个现成的且易于访问的备份。 有关详细信息，请参阅[设计应用程序以实现高可用性](high-availability-disaster-recovery.md)。

## <a name="jobs"></a>作业

[作业](jobs-and-tasks.md#jobs)是可以包含数百、数千甚至数百万个任务的容器。 创建作业时，请遵循这些指导原则。

### <a name="fewer-jobs-more-tasks"></a>更少作业，更多任务

使用一个作业运行单个任务是低效的做法。 例如，使用包含 1000 个任务的单个作业，比创建 100 个作业并在每个作业中包含 10 个任务更高效。 运行 1000 个作业并在每个作业中包含单个任务是最低效、速度最慢且成本最高的做法。

因此，请确保不要设计同时需要数千个活动作业的 Batch 解决方案。 不存在针对任务的配额，因此，通过尽量少的作业执行尽量多的任务可以有效利用[作业和作业计划配额](batch-quota-limit.md#resource-quotas)。

### <a name="job-lifetime"></a>作业生存期

在从系统中删除之前，Batch 作业生存期是无限的。 其状态会指示该作业是否可以接受更多任务来进行计划。

除非显式终止作业，否则作业不会自动转换为已完成状态。 可以通过 [onAllTasksComplete](https://docs.microsoft.com/dotnet/api/microsoft.azure.batch.common.onalltaskscomplete?view=azure-dotnet) 属性或 [maxWallClockTime](https://docs.microsoft.com/rest/api/batchservice/job/add#jobconstraints) 自动触发此状态转换。

存在默认的[活动作业和作业计划配额](batch-quota-limit.md#resource-quotas)。 处于已完成状态的作业和作业计划不会计入此配额。

## <a name="tasks"></a>任务

[任务](jobs-and-tasks.md#tasks)是构成作业的单个工作单位。 任务由用户提交，并由 Batch 在计算节点上进行计划。 创建和执行任务时，需要考虑多个设计注意事项。 以下各部分介绍了常见方案，以及如何设计任务以便能够处理问题并有效地执行任务。

### <a name="save-task-data"></a>保存任务数据

计算节点具有瞬态性。 Batch 中的许多功能（例如自动汇集如自动缩放）很容易使节点消失。 当节点离开池时（由于重设大小或删除池），这些节点上的所有文件也会一并删除。 因此，在某个任务完成之前，它应将其自身的输出从运行它的节点移到持久存储。 同样，如果任务失败，则应将诊断失败问题所需的日志移到持久存储。

Batch 中集成了用于通过 [OutputFiles](batch-task-output-files.md) 上传数据的支持 Azure 存储以及各种共享文件系统，你也可以在任务中自行执行上传。

### <a name="manage-task-lifetime"></a>管理任务生存期

当不再需要这些任务时将其删除，或者设置 [retentionTime](/dotnet/api/microsoft.azure.batch.taskconstraints.retentiontime?view=azure-dotnet) 任务约束。 如果设置了 `retentionTime`，当 `retentionTime` 过期时，Batch 会自动清理该任务占用的磁盘空间。

删除任务可以实现两种目的。 它可以确保作业中不会存在积累的任务，存在积累任务会使查询/查找感兴趣的任务变得更困难（因为你必须在“已完成”的任务中筛选）。 此外，它还会清理节点上的相应任务数据（假设尚未达到 `retentionTime`）。 这有助于确保节点中不会填满任务数据且不会耗尽磁盘空间。

### <a name="submit-large-numbers-of-tasks-in-collection"></a>以集合形式提交大量任务

可以逐个提交或以集合形式提交任务。 执行批量任务提交时，每次以最多包含 100 个任务的[集合](https://docs.microsoft.com/rest/api/batchservice/task/addcollection)形式提交任务可以减少开销并缩短提交时间。

### <a name="set-max-tasks-per-node-appropriately"></a>适当地设置每个节点的最大任务数

Batch 在节点上支持超额订阅的任务（运行的任务数超过节点所具有的核心数）。 需要由你确保任务量“适合”池中的节点。 例如，如果你尝试计划 8 个任务，其中每个任务消耗 25% 的 CPU 使用率（在设置了 `maxTasksPerNode = 8` 的池中），则体验可能会下降。

### <a name="design-for-retries-and-re-execution"></a>设计重试和重新执行

Batch 可以自动重试任务。 有两种类型的重试：用户控制的重试和内部重试。 用户控制的重试由任务的 [maxTaskRetryCount](/dotnet/api/microsoft.azure.batch.taskconstraints.maxtaskretrycount?view=azure-dotnet) 指定。 如果任务中指定的程序退出并出现非零退出代码，则会将该任务重试最多 `maxTaskRetryCount` 次。

可能会由于计算节点上发生故障（例如，在运行任务时无法更新内部状态或节点上发生故障）而在内部重试任务，不过，这种情况很罕见。 将尽可能地在同一计算节点上重试任务，直到达到内部限制，重试失败后将放弃该任务，并推迟任务以让 Batch 重新对其进行计划（可能会将其安排在不同的计算节点上）。

无论是在专用节点上还是在低优先级节点上执行任务，在设计方面没有任何差异。 无论是任务在低优先级节点上运行时被抢占资源，还是在专用节点上因为故障而中断，都可以通过将任务设计为能够承受故障来进行缓解。

### <a name="build-durable-tasks"></a>生成持久任务

在设计任务时应当使其能够承受故障并提供重试机制。 对于长时间运行的任务，这一点尤其重要。 为此，请确保任务即使多次运行，也会生成同一种的结果。 实现此目的方法之一是使任务“寻找目标”。 另一种方法是确保任务是幂等的（无论任务运行多少次，都生成相同的结果）。

一个常见示例是通过某个任务将文件复制到计算节点。 简单的方法是每次运行任务时复制所有指定的文件，但这种方法非常低效，且不能承受故障。 替代方法是创建一个任务来确保文件位于计算节点上，该任务不会重新复制已存在的文件。 通过这种方式，在该任务中断时，它会从上次中断的位置继续运行。

### <a name="avoid-short-execution-time"></a>避免短执行时间

仅运行一两秒的任务并不是很理想的任务。 应该尝试在单个任务（最少运行 10 秒，最多运行几小时甚至几天）中执行大量的工作。 如果每个任务执行一分钟（或更长时间），则调度开销将仅占总体计算时间的很少一部分。


## <a name="nodes"></a>Nodes

[计算节点](nodes-and-pools.md#nodes)是专门用于处理一部分应用程序工作负载的 Azure 虚拟机 (VM) 或云服务 VM。 使用节点时，请遵循以下指导原则。

### <a name="idempotent-start-tasks"></a>幂等启动任务

就像其他任务一样，节点[启动任务](jobs-and-tasks.md#start-task)应该是幂等的，因为每次节点启动时，都要重新运行该任务。 幂等任务就是在多次运行时生成一致结果的任务。

### <a name="manage-long-running-services-via-the-operating-system-services-interface"></a>通过操作系统服务接口管理长时间运行的服务

有时，需要在节点中将 Batch 代理与另一代理一起运行。 例如，你可能想要从节点收集数据并生成相关报告。 建议将这些代理部署为 OS 服务，例如 Windows 服务或 Linux `systemd` 服务。

运行这些服务时，它们不得对节点上 Batch 托管目录中的任何文件创建文件锁，否则 Batch 会由于存在文件锁而无法删除这些目录。 例如，如果在启动任务中安装 Windows 服务，而不是直接从启动任务工作目录启动该服务，请将文件复制到其他位置（或者，如果文件已存在，则直接跳过该复制操作）。 然后从该位置安装服务。 当 Batch 重新运行启动任务时，它会删除然后重新创建启动任务工作目录。 这是可行的，因为服务在其他目录而不是在启动任务工作目录上具有文件锁。

### <a name="avoid-creating-directory-junctions-in-windows"></a>避免在 Windows 中创建目录联接

在清理任务和作业时，很难处理目录联接（有时称为目录硬链接）。 请使用符号链接（软链接），而不要使用硬链接。

### <a name="collect-the-batch-agent-logs"></a>收集 Batch 代理日志

如果发现节点的行为或节点上运行的任务出现问题，请在解除分配有问题的节点之前收集 Batch 代理日志。 可以使用“上传 Batch 服务日志”API 收集 Batch 代理日志。 可将这些日志作为支持票证的一部分提供给 Microsoft，它们将有助于排查和解决问题。

## <a name="isolation-security"></a>隔离安全性

出于隔离目的，如果方案需要将作业相互隔离，请通过将这些作业放入不同的池中进行隔离。 池是 Batch 中的安全隔离边界，默认情况下，两个池之间互不可见，相互之间也无法通信。 请避免使用单独的 Batch 帐户作为隔离方式。

## <a name="moving-batch-accounts-across-regions"></a>跨区域移动 Batch 帐户

在某些情况下，将现有 Batch 帐户从一个区域移到另一个区域可能会很有帮助。 例如在灾难恢复计划中，你可能想将帐户移到另一个区域。

Azure Batch 帐户无法直接从一个区域移到另一个区域。 但是，可以使用 Azure 资源管理器模板来导出 Batch 帐户的现有配置。 然后，可将资源暂存在另一区域，方法是：将 Batch 帐户导出到模板，根据目标区域的情况修改参数，然后将模板部署到新区域。

将模板上传到新区域后，必须重新创建证书、作业计划和应用程序包。 若要提交更改并完成 Batch 帐户的移动，请记得删除原始 Batch 帐户或资源组。

有关资源管理器和模板的详细信息，请参阅[快速入门：使用 Azure 门户创建和部署 Azure 资源管理器模板](https://docs.azure.cn/azure-resource-manager/templates/quickstart-create-templates-use-the-portal)。

## <a name="connectivity"></a>连接

在考虑 Batch 解决方案中的连接性时，请查看以下指导。

### <a name="network-security-groups-nsgs-and-user-defined-routes-udrs"></a>网络安全组 (NSG) 和用户定义的路由 (UDR)

[在虚拟网络中预配 Batch 池](batch-virtual-network.md)时，请确保严格遵循有关使用 `BatchNodeManagement` 服务标记、端口、协议和规则方向的指导原则。
强烈建议使用服务标记，而不要使用基础 Batch 服务 IP 地址。 这是因为 IP 地址会随时间而更改。 直接使用 Batch 服务 IP 地址可能会导致 Batch 池不稳定、受干扰或中断。

对于用户定义的路由 (UDR)，请确保制定一个流程来定期更新路由表中的 Batch 服务 IP 地址，因为这些地址会随时间而更改。 若要了解如何获取 Batch 服务 IP 地址列表，请参阅[本地服务标记](../virtual-network/service-tags-overview.md)。 Batch 服务 IP 地址将与 `BatchNodeManagement` 服务标记（或与你的 Batch 帐户区域匹配的区域变体）相关联。

### <a name="honoring-dns"></a>遵守 DNS

确保系统遵守 Batch 帐户服务 URL 的 DNS 生存时间 (TTL)。 此外，请确保 Batch 服务客户端以及 Batch 服务的其他连接机制不依赖于 IP 地址。

如果请求收到 5xx 级别 HTTP 响应并且响应中包含“Connection: close”标头，则 Batch 服务客户端应遵循建议关闭现有连接，重新解析 Batch 帐户服务 URL 的 DNS，然后在新的连接上尝试后续请求。

### <a name="retry-requests-automatically"></a>自动重试请求

确保 Batch 服务客户端实施了适当的重试策略来自动重试请求，即使在正常操作期间也要实施重试机制，而不仅仅是在任何服务维护时段实施。 这些重试策略的间隔时间应该至少为 5 分钟。 各种 Batch SDK（例如 [.NET RetryPolicyProvider 类](/dotnet/api/microsoft.azure.batch.retrypolicyprovider?view=azure-dotnet)）都附带了自动重试功能。

