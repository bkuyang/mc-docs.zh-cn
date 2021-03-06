---
title: 排查 Azure 自动化混合 Runbook 辅助角色问题
description: 本文介绍如何排查和解决 Azure 自动化混合 Runbook 辅助角色出现的问题。
services: automation
ms.service: automation
ms.subservice: ''
author: WenJason
ms.author: v-jay
origin.date: 11/25/2019
ms.date: 08/10/2020
ms.topic: conceptual
manager: digimobile
ms.openlocfilehash: 5d36fca1f31207823d3e6223318c5eab54c751da
ms.sourcegitcommit: e6b216b180734783219378410e13192e314a4497
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/05/2020
ms.locfileid: "87788298"
---
# <a name="troubleshoot-hybrid-runbook-worker-issues"></a>排查混合 Runbook 辅助角色问题

本文介绍如何排查和解决 Azure 自动化混合 Runbook 辅助角色的问题。 如需常规信息，请参阅[混合 Runbook 辅助角色概述](../automation-hybrid-runbook-worker.md)。

## <a name="general"></a>常规

混合 Runbook 辅助角色依靠代理与 Azure 自动化帐户通信，以注册辅助角色、接收 Runbook 作业和报告状态。 对于 Windows，此代理是适用于 Windows 的 Log Analytics 代理。 对于 Linux，此代理是适用于 Linux 的 Log Analytics 代理。

### <a name="scenario-runbook-execution-fails"></a><a name="runbook-execution-fails"></a>场景：Runbook 执行失败

#### <a name="issue"></a>问题

Runbook 执行失败并出现以下错误消息：

```error
"The job action 'Activate' cannot be run, because the process stopped unexpectedly. The job action was attempted three times."
```

Runbook 在尝试执行三次后很快暂停。 在某些情况下，Runbook 可能会中断，无法正常完成。 相关错误消息可能不包括任何附加信息。

#### <a name="cause"></a>原因

下面是可能的原因：

* Runbook 无法使用本地资源进行身份验证。
* 混合辅助角色在代理或防火墙后面。
* 配置为运行混合 Runbook 辅助角色的计算机不满足最低硬件要求。

#### <a name="resolution"></a>解决方法

确保计算机在端口 443 上对 * **.azure-automation.cn** 有出站访问权限。

运行混合 Runbook 辅助角色的计算机应满足最低硬件要求，然后才能将辅助角色配置为托管此功能。 它们使用的 Runbook 和后台进程可能会导致系统过度使用，造成 Runbook 作业延迟或超时。

确认将要运行混合 Runbook 辅助角色功能的计算机满足最低硬件要求。 如果满足，请监视 CPU 和内存使用，以确定混合 Runbook 辅助角色进程的性能和 Windows 之间的任何关联。 出现内存或 CPU 压力可能意味着需要升级资源。 也可以选择其他支持最低要求的计算资源，并在工作负荷需求指示需要增加时进行扩展。

在 **Microsoft-SMA** 事件日志中检查附带“`Win32 Process Exited with code [4294967295]`”说明的相应事件。 此错误的原因是尚未在 Runbook 中配置身份验证，或者未为混合 Runbook 辅助角色组指定运行方式凭据。 在[在混合 Runbook 辅助角色上运行 Runbook](../automation-hrw-run-runbooks.md) 中查看 Runbook 权限，确认是否已正确配置 Runbook 的身份验证。

### <a name="scenario-event-15011-in-the-hybrid-runbook-worker"></a><a name="cannot-connect-signalr"></a>场景：混合 Runbook 辅助角色中发生事件 15011

#### <a name="issue"></a>问题

混合 Runbook 辅助角色收到表示查询结果无效的事件 15011。 当辅助角色尝试与 [SignalR 服务器](https://docs.microsoft.com/aspnet/core/signalr/introduction?view=aspnetcore-3.1)建立连接时出现以下错误。

```error
[AccountId={c7d22bd3-47b2-4144-bf88-97940102f6ca}]
[Uri=https://cc-jobruntimedata-prod-su1.azure-automation.cn/notifications/hub][Exception=System.TimeoutException: Transport timed out trying to connect
   at System.Runtime.ExceptionServices.ExceptionDispatchInfo.Throw()
   at System.Runtime.CompilerServices.TaskAwaiter.HandleNonSuccessAndDebuggerNotification(Task task)
   at JobRuntimeData.NotificationsClient.JobRuntimeDataServiceSignalRClient.<Start>d__45.MoveNext()
```

#### <a name="cause"></a>原因

未为自动化功能部署（例如更新管理）正确配置混合 Runbook 辅助角色。 部署包含的某个部件会将 VM 连接到 Log Analytics 工作区。 PowerShell 脚本将在订阅中查找具有所提供名称的工作区。 在此示例中，该 Log Analytics 工作区位于其他订阅中。 脚本找不到该工作区，因此尝试创建一个工作区，但该名称已被占用。 因此，部署失败。

#### <a name="resolution"></a>解决方法

可以采用两种做法来解决此问题：

* 修改 PowerShell 脚本，以在另一个订阅中查找 Log Analytics 工作区。 如果你将来打算部署许多混合 Runbook 辅助角色计算机，则非常适合使用此解决方法。

* 手动将辅助角色计算机配置为在业务流程协调程序沙盒中运行。 然后在该辅助角色上运行在 Azure 自动化帐户中创建的 Runbook，以测试功能。

### <a name="scenario-windows-azure-vms-automatically-dropped-from-a-hybrid-worker-group"></a><a name="vm-automatically-dropped"></a>场景：从混合辅助角色组中自动删除 Windows Azure VM

#### <a name="issue"></a>问题

当辅助角色计算机长期关闭时，看不到混合 Runbook 辅助角色或 VM。

#### <a name="cause"></a>原因

混合 Runbook 辅助角色计算机有 30 天以上无法 ping 通 Azure 自动化。 因此，自动化清除了混合 Runbook 辅助角色组或系统辅助角色组。 

#### <a name="resolution"></a>解决方法

启动辅助角色计算机，然后将其重新注册到 Azure 自动化。 有关如何安装 Runbook 环境和连接到 Azure 自动化的说明，请参阅[部署 Windows 混合 Runbook 辅助角色](../automation-windows-hrw-install.md)。

### <a name="scenario-no-certificate-was-found-in-the-certificate-store-on-the-hybrid-runbook-worker"></a><a name="no-cert-found"></a>场景：在混合 Runbook 辅助角色上的证书存储中找不到证书

#### <a name="issue"></a>问题

混合 Runbook 辅助角色上运行的 runbook 失败并显示以下错误消息：

```error
Connect-AzAccount : No certificate was found in the certificate store with thumbprint 0000000000000000000000000000000000000000
At line:3 char:1
+ Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -Appl ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : CloseError: (:) [Connect-AzAccount], ArgumentException
    + FullyQualifiedErrorId : Microsoft.Azure.Commands.Profile.ConnectAzAccountCommand
```
#### <a name="cause"></a>原因

尝试在混合 Runbook 辅助角色上运行的 Runbook 中使用[运行方式帐户](../manage-runas-account.md)时，如果运行方式帐户证书不存在，则会发生此错误。 默认情况下，混合 Runbook 辅助角色在本地没有证书资产。 运行方式帐户需要此资产才能正常运行。

#### <a name="resolution"></a>解决方法

如果混合 Runbook 辅助角色是 Azure VM，则可改用[使用托管标识的 Runbook 身份验证](../automation-hrw-run-runbooks.md#runbook-auth-managed-identities)。 此方案允许使用 Azure VM 的托管标识而非运行方式帐户向 Azure 资源进行身份验证，从而简化了身份验证。 如果混合 Runbook 辅助角色是本地计算机，需要在此计算机上安装运行方式帐户证书。 若要了解如何安装证书，请参阅[在混合 Runbook 辅助角色上运行 Runbook](../automation-hrw-run-runbooks.md) 中有关如何运行 PowerShell Runbook **Export-RunAsCertificateToHybridWorker** 的步骤。

### <a name="scenario-error-403-during-registration-of-a-hybrid-runbook-worker"></a><a name="error-403-on-registration"></a>场景：在注册混合 Runbook 辅助角色期间发生错误 403

#### <a name="issue"></a>问题

辅助角色的初始注册阶段失败并出现以下错误 (403)：

```error
"Forbidden: You don't have permission to access / on this server."
```

#### <a name="cause"></a>原因

以下问题是可能的原因：

* 在代理设置中错误键入了工作区 ID 或工作区密钥（主密钥）。 
* 混合 Runbook 辅助角色无法下载配置，导致帐户链接错误。 当 Azure 在计算机上启用功能时，它仅支持特定的区域链接 Log Analytics 工作区和自动化帐户。 此外，还可能在计算机上设置了错误的日期或时间。 如果时间比当前时间快/慢 15 分钟，则功能部署会失败。

#### <a name="resolution"></a>解决方法

##### <a name="mistyped-workspace-id-or-key"></a>错误键入了工作区 ID 或密钥
若要验证是否错误键入了代理的工作区 ID 或工作区密钥，请参阅[添加或删除工作区 – Windows 代理](../../azure-monitor/platform/agent-manage.md#windows-agent)（适用于 Windows 代理）或者[添加或删除工作区 – Linux 代理](../../azure-monitor/platform/agent-manage.md#linux-agent)（适用于 Linux 代理）。 确保从 Azure 门户中选择完整字符串，然后小心复制并粘贴该字符串。

##### <a name="configuration-not-downloaded"></a>未下载配置

Log Analytics 工作区和自动化帐户必须位于链接的区域中。 有关支持的区域列表，请参阅 [Azure 自动化和 Log Analytics 工作区映射](../how-to/region-mappings.md)。

可能还需要更新计算机的日期或时区。 如果选择自定义时间范围，请确保该范围采用 UTC，它可能与你的本地时区不同。

## <a name="linux"></a>Linux

Linux 混合 Runbook 辅助角色依靠[适用于 Linux 的 Log Analytics 代理](../../azure-monitor/platform/log-analytics-agent.md)与自动化帐户通信，以注册辅助角色、接收 Runbook 作业和报告状态。 如果辅助角色注册失败，以下是一些可能导致此错误的原因。

### <a name="scenario-linux-hybrid-runbook-worker-receives-prompt-for-a-password-when-signing-a-runbook"></a><a name="prompt-for-password"></a>场景：Linux 混合 Runbook 辅助角色在为 Runbook 签名时收到密码提示

#### <a name="issue"></a>问题

针对 Linux 混合 Runbook 辅助角色运行 `sudo` 命令时检索到意外的密码提示。

#### <a name="cause"></a>原因

未在 **sudoers** 文件中正确配置适用于 Linux 的 Log Analytics 代理的 **nxautomationuser** 帐户。 需要为混合 Runbook 辅助角色适当配置帐户权限和其他数据，才能让它为 Linux Runbook 辅助角色中的 Runbook 签名。

#### <a name="resolution"></a>解决方法

* 确保混合 Runbook 辅助角色在计算机上具有 GnuPG (GPG) 可执行文件。

* 验证 **sudoers** 文件中的 **nxautomationuser** 帐户配置。 参阅[在混合 Runbook 辅助角色上运行 Runbook](../automation-hrw-run-runbooks.md)。

### <a name="scenario-log-analytics-agent-for-linux-isnt-running"></a><a name="oms-agent-not-running"></a>场景：适用于 Linux 的 Log Analytics 代理未运行

#### <a name="issue"></a>问题

适用于 Linux 的 Log Analytics 代理未运行。

#### <a name="cause"></a>原因

如果该代理未运行，会导致 Linux 混合 Runbook 辅助角色无法与 Azure 自动化通信。 该代理可能会出于各种原因而未运行。

#### <a name="resolution"></a>解决方法

 输入命令 `ps -ef | grep python` 以验证该代理是否正在运行。 应该会看到与下面类似的输出。 使用 **nxautomation** 用户帐户的 Python 进程。 如果未启用 Azure 自动化功能，则以下任何进程都不会运行。

```bash
nxautom+   8567      1  0 14:45 ?        00:00:00 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/main.py /var/opt/microsoft/omsagent/state/automationworker/oms.conf rworkspace:<workspaceId> <Linux hybrid worker version>
nxautom+   8593      1  0 14:45 ?        00:00:02 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/hybridworker.py /var/opt/microsoft/omsagent/state/automationworker/worker.conf managed rworkspace:<workspaceId> rversion:<Linux hybrid worker version>
nxautom+   8595      1  0 14:45 ?        00:00:02 python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/worker/hybridworker.py /var/opt/microsoft/omsagent/<workspaceId>/state/automationworker/diy/worker.conf managed rworkspace:<workspaceId> rversion:<Linux hybrid worker version>
```

以下列表显示针对 Linux 混合 Runbook 辅助角色启动的进程。 这些进程全部位于 /var/opt/microsoft/omsagent/state/automationworker/ 目录中。

* **oms.conf**：辅助角色管理器进程。 它直接从 DSC 启动。
* **worker.conf**：自动注册的混合辅助角色进程。 它由辅助角色管理器启动。 此进程由更新管理使用且对用户而言是透明的。 如果未在计算机上启用更新管理，则不会显示此进程。
* **diy/worker.conf**：DIY 混合辅助角色进程。 DIY 混合辅助角色进程用于执行混合 Runbook 辅助角色的用户 Runbook。 它仅与使用不同配置的自动注册混合辅助角色进程在主要细节上有所不同。 如果禁用 Azure 自动化，并且 DIY Linux 混合辅助角色未注册，则不会显示此进程。

如果该代理未运行，请运行以下命令以启动该服务：`sudo /opt/microsoft/omsagent/bin/service_control restart`。

### <a name="scenario-the-specified-class-doesnt-exist"></a><a name="class-does-not-exist"></a>场景：指定的类不存在

如果在 **/var/opt/microsoft/omsconfig/omsconfig.log** 中看到错误消息 `The specified class does not exist..`，则需要更新适用于 Linux 的 Log Analytics 代理。 运行以下命令以重新安装该代理。

```bash
wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh && sh onboard_agent.sh -w <WorkspaceID> -s <WorkspaceKey>
```

## <a name="windows"></a>Windows

Windows 混合 Runbook 辅助角色依靠[适用于 Windows 的 Log Analytics 代理](../../azure-monitor/platform/log-analytics-agent.md)与自动化帐户通信，以注册辅助角色、接收 Runbook 作业和报告状态。 如果辅助角色注册失败，请参考本部分所述的一些可能原因。

### <a name="scenario-the-log-analytics-agent-for-windows-isnt-running"></a><a name="mma-not-running"></a>场景：适用于 Windows 的 Log Analytics 代理未运行。

#### <a name="issue"></a>问题

`healthservice` 未在混合 Runbook 辅助角色计算机上运行。

#### <a name="cause"></a>原因

如果适用于 Windows 的 Log Analytics 服务未运行，则混合 Runbook 辅助角色无法与 Azure 自动化通信。

#### <a name="resolution"></a>解决方法

在 PowerShell 中输入以下命令，验证代理是否正在运行：`Get-Service healthservice`。 如果该服务已停止，请在 PowerShell 中输入以下命令启动该服务：`Start-Service healthservice`。

### <a name="scenario-event-4502-in-the-operations-manager-log"></a><a name="event-4502"></a>场景：Operations Manager 日志中出现事件 4502

#### <a name="issue"></a>问题

在 **Application and Service Logs\Operations Manager** 事件日志中看到事件 4502，以及包含 `Microsoft.EnterpriseManagement.HealthService.AzureAutomation.HybridAgent` 并附带以下说明的事件消息：<br>`The certificate presented by the service \<wsid\>.oms.opinsights.azure.cn was not issued by a certificate authority used for Azure services. Please contact your network administrator to see if they are running a proxy that intercepts TLS/SSL communication.`

#### <a name="cause"></a>原因

此问题可能是由于代理或网络防火墙阻止与 Azure 的通信造成的。 确保计算机在端口 443 上对 * **.azure-automation.cn** 有出站访问权限。

#### <a name="resolution"></a>解决方法

日志存储在每个混合辅助角色本地的 C:\ProgramData\Microsoft\System Center\Orchestrator\7.2\SMA\Sandboxes 中。 可以在 **Application and Services Logs\Microsoft-SMA\Operations** 和 **Application and Services Logs\Operations Manager** 事件日志中验证是否出现了任何警告或错误事件。 这些日志会指出发生影响了在 Azure 自动化中启用角色的连接问题或其他类型的问题，或者在正常操作时遇到的问题。 在排查 Log Analytics 代理问题时如需更多帮助，请参阅[排查 Log Analytics Windows 代理的问题](../../azure-monitor/platform/agent-windows-troubleshoot.md)。

混合辅助角色将 [Runbook 输出和消息](../automation-runbook-output-and-messages.md)发送到 Azure 自动化，其发送方式与云中运行的 Runbook 作业发送输出和消息的方式相同。 可以像使用 Runbook 时一样启用“详细”流和“进度”流。

### <a name="scenario-orchestratorsandboxexe-cant-connect-to-office-365-through-proxy"></a><a name="no-orchestrator-sandbox-connect-O365"></a>场景：Orchestrator.Sandbox.exe 无法通过代理连接到 Office 365

### <a name="scenario-hybrid-runbook-worker-not-reporting"></a><a name="corrupt-cache"></a>场景：混合 Runbook 辅助角色未提供报告

#### <a name="issue"></a>问题

混合 Runbook 辅助角色计算机在运行，但是在工作区中未看到该计算机的检测信号数据。

以下示例查询显示了工作区中的计算机及其上次检测信号：

```loganalytics
// Last heartbeat of each computer
Heartbeat
| summarize arg_max(TimeGenerated, *) by Computer
```

#### <a name="cause"></a>原因

此问题可能是由于混合 Runbook 辅助角色上的高速缓存损坏导致的。

#### <a name="resolution"></a>解决方法

若要解决此问题，请登录到混合 Runbook 辅助角色并运行以下脚本。 此脚本将停止适用于 Windows 的 Log Analytics 代理，删除其缓存，并重启服务。 此操作会强制混合 Runbook 辅助角色从 Azure 自动化重新下载其配置。

```powershell
Stop-Service -Name HealthService

Remove-Item -Path 'C:\Program Files\Microsoft Monitoring Agent\Agent\Health Service State' -Recurse

Start-Service -Name HealthService
```

### <a name="scenario-you-cant-add-a-hybrid-runbook-worker"></a><a name="already-registered"></a>场景：无法添加混合 Runbook 辅助角色

#### <a name="issue"></a>问题

尝试使用 `Add-HybridRunbookWorker` cmdlet 添加混合 Runbook 辅助角色时收到以下消息：

```error
Machine is already registered
```

#### <a name="cause"></a>原因

如果计算机已注册到一个不同的自动化帐户，或者在将混合 Runbook 辅助角色从计算机中删除后尝试重新添加它，则可能会导致此问题。

#### <a name="resolution"></a>解决方法

若要解决此问题，请删除以下注册表项，重启 `HealthService`，然后再次尝试运行 `Add-HybridRunbookWorker` cmdlet。

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\HybridRunbookWorker`

## <a name="next-steps"></a>后续步骤

如果你的问题未在本文中列出，或者无法解决问题，请尝试通过以下渠道之一获取更多支持：

* 通过 [Azure 论坛](https://social.msdn.microsoft.com/Forums/zh-cn/home)获取 Azure 专家的解答
* 提出 Azure 支持事件。 转到 [Azure 支持站点](https://support.azure.cn/zh-cn/support/contact/)。
