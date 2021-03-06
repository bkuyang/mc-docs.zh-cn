---
title: Azure 中的 Office 365 管理解决方案 | Azure Docs
description: 本文详细介绍如何配置和使用 Azure 中的 Office 365 解决方案。  它还详细介绍了在 Azure Monitor 中创建的 Office 365 记录。
author: lingliw
manager: digimobile
ms.subservice: ''
ms.topic: conceptual
origin.date: 12/27/2019
ms.date: 2/18/2020
ms.author: v-lingwu
ms.openlocfilehash: 5909982fbd69643bb095e69b09d830e59990d422
ms.sourcegitcommit: c1ba5a62f30ac0a3acb337fb77431de6493e6096
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/17/2020
ms.locfileid: "78850318"
---
# <a name="office-365-management-solution-in-azure-preview"></a>Azure 中的 Office 365 管理解决方案（预览版）

![Office 365 徽标](media/solution-office-365/icon.png)


> 
> Azure Sentinel 是云中原生的安全信息和事件管理解决方案，它可以引入日志并提供附加的 SIEM 功能，包括检测、调查、搜寻和机器学习驱动的见解。 Azure Sentinel 现在可以提供 Office 365 SharePoint 活动和 Exchange 管理日志的引入功能。
> 
> 收集 Azure AD 日志需要按 Azure Monitor 定价付费。  有关详细信息，请参阅 [Azure Monitor 定价](https://azure.microsoft.com/pricing/details/monitor/)。

> ## <a name="frequently-asked-questions"></a>常见问题
> 
> ### <a name="q-is-it-possible-to-on-board-the-office-365-azure-monitor-solution-between-now-and-april-30th"></a>问：在当前时间与 4 月 30 日之间是否可以加入 Office 365 Azure Monitor 解决方案？
> 不可以，Azure Monitor Office 365 解决方案加入脚本不再可用。 我们将在 4 月 30 日删除该解决方案。
> 
> ### <a name="q-will-the-tables-and-schemas-be-changed"></a>问：表和架构是否会有更改？
> **OfficeActivity** 表名称和架构将与当前解决方案相同。 可以继续在新解决方案中使用相同的查询，但不包括引用 Azure AD 数据的查询。
> 
> 以下示例将 **OfficeActivity** 中的查询转换为 **SigninLogs**：
> 
> **按用户查询失败的登录：**
> 
> ```Kusto
> OfficeActivity
> | where TimeGenerated >= ago(1d) 
> | where OfficeWorkload == "AzureActiveDirectory"                      
> | where Operation == 'UserLoginFailed'
> | summarize count() by UserId 
> ```
> 
> ```Kusto
> SigninLogs
> | where ConditionalAccessStatus == "failure" or ConditionalAccessStatus == "notApplied"
> | summarize count() by UserDisplayName
> ```
> 
> **查看 Azure AD 操作：**
> 
> ```Kusto
> OfficeActivity
> | where OfficeWorkload =~ "AzureActiveDirectory"
> | sort by TimeGenerated desc
> | summarize AggregatedValue = count() by Operation
> ```
> 
> ```Kusto
> AuditLogs
> | summarize count() by OperationName
> ```
> 
> ### <a name="q-how-can-i-on-board-azure-sentinel"></a>问：如何加入 Azure Sentinel？
> Azure Sentinel 是可以在新的或现有的 Log Analytics 工作区中启用的解决方案。 
>

> ###   <a name="q-what-do-i-need-to-change-when-moving-to-the-new-azure-ad-reporting-and-monitoring-tables"></a>问：迁移到新的 Azure AD 报告和监视表时需要做出哪些更改？
> 使用 Azure AD 数据的所有查询（包括警报、仪表板中的查询）以及使用 Office 365 Azure AD 数据创建的任何内容必须使用新表重新创建。
>
> ### <a name="q-how-i-can-use-the-azure-sentinel-out-of-the-box-security-oriented-content"></a>问：如何使用 Azure Sentinel 现成的安全导向内容？
> Azure Sentinel 基于 Office 365 和 Azure AD 日志提供现成的安全导向仪表板、自定义警报查询、搜寻查询、调查和自动响应功能。 请浏览 Azure Sentinel GitHub 和教程了解详细信息：
>
> ###   <a name="q-what-will-happen-on-april-30-do-i-need-to-offboard-beforehand"></a>问：4 月 30 日会发生什么情况？ 我是否需要提前卸载现有版本？
> 
> - 你将无法从 **Office365** 解决方案接收数据。 该解决方案将不再在市场中提供
> - 对于 Azure Sentinel 客户，Log Analytics 工作区解决方案 **Office365** 将包含在 Azure Sentinel **SecurityInsights** 解决方案中。
> - 如果不手动卸载现有解决方案，4 月 30 日数据将自动断开连接。
> 
> ### <a name="q-will-my-data-transfer-to-the-new-solution"></a>问：我的数据是否会传输到新解决方案？
> 是的。 从工作区中删除 **Office 365** 解决方案时，其数据将暂时不可用，因为架构已删除。 在 Sentinel 中启用新的 **Office 365** 连接器后，架构将还原到工作区，任何已收集的数据将可用。 
 

通过 Office 365 管理解决方案，可在 Azure Monitor 中监视 Office 365 环境。

- 监视 Office 365 帐户的用户活动，以分析使用模式并确定行为趋势。 例如，可提取特定使用方案，例如组织外共享的文件或最常用的 SharePoint 网站。
- 监视管理员活动，以跟踪配置更改或高特权操作。
- 检测并调查多余的用户行为，此操作可根据组织需求进行自定义。
- 演示审核和符合性。 例如，可监视对机密文件的文件访问操作，这对审核和符合性进程有所帮助。
- 通过对组织的 Office 365 活动数据使用[日志查询](../log-query/log-query-overview.md)，执行操作故障排除。


## <a name="uninstall"></a>卸载

可以使用[删除管理解决方案](solutions.md#remove-a-monitoring-solution)中的过程删除 Office 365 管理解决方案。 但是，这不会停止将数据从 Office 365 收集到 Azure Monitor 中。 请按照下面的过程来取消订阅 Office 365 并停止收集数据。

1. 将以下脚本保存为 office365_unsubscribe.ps1  。

    ```powershell
    param (
        [Parameter(Mandatory=$True)][string]$WorkspaceName,
        [Parameter(Mandatory=$True)][string]$ResourceGroupName,
        [Parameter(Mandatory=$True)][string]$SubscriptionId,
        [Parameter(Mandatory=$True)][string]$OfficeTennantId
    )
    $line='#-------------------------------------------------------------------------------------------------------------------------------------------------------------------------'
    
    $line
    IF ($Subscription -eq $null)
        {Login-AzAccount -ErrorAction Stop}
    $Subscription = (Select-AzSubscription -SubscriptionId $($SubscriptionId) -ErrorAction Stop)
    $Subscription
    $option = [System.StringSplitOptions]::RemoveEmptyEntries 
    $Workspace = (Set-AzOperationalInsightsWorkspace -Name $($WorkspaceName) -ResourceGroupName $($ResourceGroupName) -ErrorAction Stop)
    $Workspace
    $WorkspaceLocation= $Workspace.Location

    # Client ID for Azure PowerShell
    $clientId = "1950a258-227b-4e31-a9cf-717495945fc2"
    # Set redirect URI for Azure PowerShell
    $redirectUri = "urn:ietf:wg:oauth:2.0:oob"
    $domain='login.partner.microsoftonline.cn'
    $adTenant =  $Subscription[0].Tenant.Id
    $authority = "https://login.chinacloudapi.cn/$adTenant";
    $ARMResource ="https://management.chinacloudapi.cn/";
    $xms_client_tenant_Id ='55b65fb5-b825-43b5-8972-c8b6875867c1'

    switch ($WorkspaceLocation) {
           "USGov Virginia" { 
                             $domain='login.partner.microsoftonline.cn';
                              $authority = "https://login.chinacloudapi.cn/$adTenant";
                              $ARMResource ="https://management.usgovcloudapi.net/"; break} # China Gov Virginia
           default {
                    $domain='login.partner.microsoftonline.cn'; 
                    $authority = "https://login.chinacloudapi.cn/$adTenant";
                    $ARMResource ="https://management.chinacloudapi.cn/";break} 
                    }

    Function RESTAPI-Auth { 

    $global:SubscriptionID = $Subscription.SubscriptionId
    # Set Resource URI to Azure Service Management API
    $resourceAppIdURIARM=$ARMResource;
    # Authenticate and Acquire Token 
    # Create Authentication Context tied to Azure AD Tenant
    $authContext = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext" -ArgumentList $authority
    # Acquire token
    $platformParameters = New-Object "Microsoft.IdentityModel.Clients.ActiveDirectory.PlatformParameters" -ArgumentList "Auto"
    $global:authResultARM = $authContext.AcquireTokenAsync($resourceAppIdURIARM, $clientId, $redirectUri, $platformParameters)
    $global:authResultARM.Wait()
    $authHeader = $global:authResultARM.Result.CreateAuthorizationHeader()
    $authHeader
    }
    
    Function Office-UnSubscribe-Call{
    
    #----------------------------------------------------------------------------------------------------------------------------------------------
    $authHeader = $global:authResultARM.Result.CreateAuthorizationHeader()
    $ResourceName = "https://manage.office.com"
    $SubscriptionId   = $Subscription[0].Subscription.Id
    $OfficeAPIUrl = $ARMResource + 'subscriptions/' + $SubscriptionId + '/resourceGroups/' + $ResourceGroupName + '/providers/Microsoft.OperationalInsights/workspaces/' + $WorkspaceName + '/datasources/office365datasources_'  + $SubscriptionId + $OfficeTennantId + '?api-version=2015-11-01-preview'
    
    $Officeparams = @{
        ContentType = 'application/json'
        Headers = @{
        'Authorization'="$($authHeader)"
        'x-ms-client-tenant-id'=$xms_client_tenant_Id
        'Content-Type' = 'application/json'
        }
        Method = 'Delete'
        URI = $OfficeAPIUrl
      }
    
    $officeresponse = Invoke-WebRequest @Officeparams 
    $officeresponse
    
    }
    
    #GetDetails 
    RESTAPI-Auth -ErrorAction Stop
    Office-UnSubscribe-Call -ErrorAction Stop
    ```

2. 使用以下命令运行该脚本：

    ```
    .\office365_unsubscribe.ps1 -WorkspaceName <Log Analytics workspace name> -ResourceGroupName <Resource Group name> -SubscriptionId <Subscription ID> -OfficeTennantID <Tenant ID> 
    ```

    示例：

    ```powershell
    .\office365_unsubscribe.ps1 -WorkspaceName MyWorkspace -ResourceGroupName MyResourceGroup -SubscriptionId '60b79d74-f4e4-4867-b631-yyyyyyyyyyyy' -OfficeTennantID 'ce4464f8-a172-4dcf-b675-xxxxxxxxxxxx'
    ```

系统将提示你输入凭据。 提供 Log Analytics 工作区的凭据。

## <a name="data-collection"></a>数据收集

初次收集数据可能需要几个小时。 在开始收集后，每次创建记录时，Office 365 都会向 Azure Monitor 发送带详细数据的 [webhook 通知](https://msdn.microsoft.com/office-365/office-365-management-activity-api-reference#receiving-notifications)。 在收到此记录几分钟后，此记录将出现在 Azure Monitor 中。

## <a name="using-the-solution"></a>使用解决方案

[!INCLUDE [azure-monitor-solutions-overview-page](../../../includes/azure-monitor-solutions-overview-page.md)]

向 Log Analytics 工作区添加 Office 365 解决方案时，“Office 365”磁贴将添加到你的仪表板  。 此磁贴显示环境中计算机数量及其更新符合性的计数和图形表示形式。<br><br>
![Office 365 摘要磁贴](media/solution-office-365/tile.png)  

单击“Office 365”磁贴，打开“Office 365”仪表板   。

![Office 365 仪表板](media/solution-office-365/dashboard.png)  

仪表板包含下表中的列。 每个列按照指定范围和时间范围内符合该列条件的计数列出了前十个警报。 可通过以下方式运行提供整个列表的日志搜索：单击该列底部的“全部查看”或单击列标题。

| 列 | 说明 |
|:--|:--|
| 操作 | 提供所有监视的 Office 365 订阅中的活动用户相关信息。 还能够看到随着时间的推移发生的活动数。
| Exchange | 显示 Exchange Server 活动的明细，例如 Add-Mailbox 权限或 Set-Mailbox。 |
| SharePoint | 显示用户在 SharePoint 文档上执行次数最多的一些活动。 从此磁贴向下钻取时，搜索页会显示这些活动的详细信息，例如目标文档和此活动的位置。 比如，对于文件访问事件，将能够看到正在访问的文档、其关联的帐户名以及 IP 地址。 |
| Azure Active Directory | 包含一些最活跃的用户活动，例如重置用户密码和登录尝试。 向下钻取时，将能够看到这些活动的详细信息（例如结果状态）。 如果想要监视 Azure Active Directory 上的可疑活动，这通常很有帮助。 |




## <a name="azure-monitor-log-records"></a>Azure Monitor 日志记录

对于 Office 365 解决方案在 Azure Monitor 中的 Log Analytics 工作区中创建的所有记录，其类型都是 **OfficeActivity**。   OfficeWorkload 属性确定记录所指的 Office 365 服务 - Exchange、AzureActiveDirectory、SharePoint 或 OneDrive  。  RecordType 属性指定操作的类型  。  每种操作类型的属性都不同，详情请见下表。

### <a name="common-properties"></a>公共属性

以下属性对于所有 Office 365 记录通用。

| 属性 | 说明 |
|:--- |:--- |
| 类型 | OfficeActivity  |
| ClientIP | 记录活动时使用的设备的 IP 地址。 IP 地址以 IPv4 或 IPv6 地址格式显示。 |
| OfficeWorkload | 记录所指的 Office 365 服务。<br><br>AzureActiveDirectory<br>Exchange<br>SharePoint|
| 操作 | 用户或管理员活动的名称。  |
| OrganizationId | 组织的 Office 365 租户的 GUID。 无论发生在哪种 Office 365 服务中，组织中的此值均保持不变。 |
| RecordType | 所执行操作的类型。 |
| ResultStatus | 指示操作（在 Operation 属性中指定）是成功还是失败。 可能的值有 Succeeded、PartiallySucceeded 或 Failed。 对于 Exchange 管理员活动，值为 True 或 False。 |
| UserId | 执行使系统记下记录的操作的用户的 UPN（用户主体名称），例如 my_name@my_domain_name。 请注意，还包括系统帐户（例如 SHAREPOINT\system 或 NTAUTHORITY\SYSTEM）执行的活动的记录。 | 
| UserKey | UserId 属性中标识的用户的备用 ID。  例如，此属性由 SharePoint、OneDrive for Business 和 Exchange 中用户执行的事件的 Passport 唯一 ID (PUID) 进行填充。 此属性还可为其他服务中发生的事件以及系统帐户执行的事件指定与 UserID 属性相同的值|
| UserType | 执行操作的用户的类型。<br><br>管理员<br>应用程序<br>DcAdmin<br>常规<br>保留<br>服务主体<br>系统 |


### <a name="azure-active-directory-base"></a>Azure Active Directory Base

以下属性对于所有 Azure Active Directory 记录通用。

| 属性 | 说明 |
|:--- |:--- |
| OfficeWorkload | AzureActiveDirectory |
| RecordType     | AzureActiveDirectory |
| AzureActiveDirectory_EventType | Azure AD 事件的类型。 |
| ExtendedProperties | Azure AD 事件的扩展属性。 |


### <a name="azure-active-directory-account-logon"></a>Azure Active Directory 帐户登录

Active Directory 用户尝试登录时，将创建这些记录。

| 属性 | 说明 |
|:--- |:--- |
| `OfficeWorkload` | AzureActiveDirectory |
| `RecordType`     | AzureActiveDirectoryAccountLogon |
| `Application` | 触发帐户登录事件的应用程序，如 Office 15。 |
| `Client` | 有关客户端设备、设备操作系统和用于帐户登录事件的设备浏览器的详细信息。 |
| `LoginStatus` | 此属性直接从 OrgIdLogon.LoginStatus 获取。 可通过警报算法完成各种关注的登录失败的映射。 |
| `UserDomain` | 租户标识信息 (TII)。 | 


### <a name="azure-active-directory"></a>Azure Active Directory

更改 Azure Active Directory 对象或向其添加内容时，将创建这些记录。

| 属性 | 说明 |
|:--- |:--- |
| OfficeWorkload | AzureActiveDirectory |
| RecordType     | AzureActiveDirectory |
| AADTarget | 所执行操作（由 Operation 属性标识）针对的用户。 |
| 执行组件 | 执行操作的用户或服务主体。 |
| ActorContextId | 参与者所属的组织的 GUID。 |
| ActorIpAddress | 采用 IPV4 或 IPV6 地址格式的参与者 IP 地址。 |
| InterSystemsId | 跨 Office 365 服务内的组件跟踪操作的 GUID。 |
| IntraSystemId |   由 Azure Active Directory 生成用于跟踪操作的 GUID。 |
| SupportTicketId | “代表执行”情况下的操作的客户支持票证 ID。 |
| TargetContextId | 目标用户所属的组织的 GUID。 |


### <a name="data-center-security"></a>数据中心安全
基于数据中心安全审核数据创建这些记录。  

| 属性 | 说明 |
|:--- |:--- |
| EffectiveOrganization | 提升/cmdlet 面向的租户的名称。 |
| ElevationApprovedTime | 提升获得批准时的时间戳。 |
| ElevationApprover | Azure 管理器的名称。 |
| ElevationDuration | 提升处于活动状态的持续时间。 |
| ElevationRequestId |  提升请求的唯一标识符。 |
| ElevationRole | 为其请求提升的角色。 |
| ElevationTime | 提升的开始时间。 |
| Start_Time | cmdlet 执行的开始时间。 |


### <a name="exchange-admin"></a>Exchange 管理员

更改 Exchange 配置时，将创建这些记录。

| 属性 | 说明 |
|:--- |:--- |
| OfficeWorkload | Exchange |
| RecordType     | ExchangeAdmin |
| ExternalAccess |  指定 cmdlet 是由组织中的用户运行、由 Microsoft 数据中心人员或数据中心服务帐户运行，还是由委派的管理员运行。 值 False 标识 cmdlet 由组织中的某人运行。 值 True 表示 cmdlet 由数据中心人员、数据中心服务帐户或委派的管理员运行。 |
| ModifiedObjectResolvedName |  这是由 cmdlet 修改的对象的用户友好名称。 仅在 cmdlet 修改对象时才记录此信息。 |
| OrganizationName | 租户的名称。 |
| OriginatingServer | 从中执行 cmdlet 的服务器的名称。 |
| parameters | 与 Operations 属性中标识的 cmdlet 结合使用的所有参数的名称和值。 |


### <a name="exchange-mailbox"></a>Exchange 邮箱

更改 Exchange 邮箱或向其添加内容时，将创建这些记录。

| 属性 | 说明 |
|:--- |:--- |
| OfficeWorkload | Exchange |
| RecordType     | ExchangeItem |
| ClientInfoString | 用于执行操作的电子邮件客户端的相关信息，例如浏览器版本、Outlook 版本和移动设备信息。 |
| Client_IPAddress | 记录操作时所用的设备的 IP 地址。 IP 地址以 IPv4 或 IPv6 地址格式显示。 |
| ClientMachineName | 托管 Outlook 客户端的计算机名称。 |
| ClientProcessName | 用于访问邮箱的电子邮件客户端。 |
| ClientVersion | 电子邮件客户端的版本。 |
| InternalLogonType | 保留以供内部使用。 |
| Logon_Type | 表示访问邮箱并执行所记录的操作的用户类型。 |
| LogonUserDisplayName |    执行操作的用户的用户友好名称。 |
| LogonUserSid | 执行操作的用户的 SID。 |
| MailboxGuid | 所访问邮箱的 Exchange GUID。 |
| MailboxOwnerMasterAccountSid | 邮箱所有者帐户的主帐户 SID。 |
| MailboxOwnerSid | 邮箱所有者的 SID。 |
| MailboxOwnerUPN | 拥有所访问邮箱的人员的电子邮件地址。 |


### <a name="exchange-mailbox-audit"></a>Exchange 邮箱审核

创建邮箱审核项时，将创建这些记录。

| 属性 | 说明 |
|:--- |:--- |
| OfficeWorkload | Exchange |
| RecordType     | ExchangeItem |
| 项目 | 表示对其执行操作的项 | 
| SendAsUserMailboxGuid | 为发送电子邮件而访问的邮箱的 Exchange GUID。 |
| SendAsUserSmtp | 被模拟用户的 SMTP 地址。 |
| SendonBehalfOfUserMailboxGuid | 为代替发送邮件而访问的邮箱的 Exchange GUID。 |
| SendOnBehalfOfUserSmtp | 以其名义发送电子邮件的用户的 SMTP 地址。 |


### <a name="exchange-mailbox-audit-group"></a>Exchange 邮箱审核组

更改 Exchange 组或向其添加内容时，将创建这些记录。

| 属性 | 说明 |
|:--- |:--- |
| OfficeWorkload | Exchange |
| OfficeWorkload | ExchangeItemGroup |
| AffectedItems | 组中每个项的相关信息。 |
| CrossMailboxOperations | 表示操作是否涉及多个邮箱。 |
| DestMailboxId | 仅在 CrossMailboxOperations 参数为 True 时设置。 指定目标邮箱 GUID。 |
| DestMailboxOwnerMasterAccountSid | 仅在 CrossMailboxOperations 参数为 True 时设置。 为目标邮箱所有者的主帐户 SID 指定 SID。 |
| DestMailboxOwnerSid | 仅在 CrossMailboxOperations 参数为 True 时设置。 指定目标邮箱的 SID。 |
| DestMailboxOwnerUPN | 仅在 CrossMailboxOperations 参数为 True 时设置。 指定的目标邮箱所有者的 UPN。 |
| DestFolder | 针对“移动”等操作的目标文件夹。 |
| 文件夹 | 一组项所在的文件夹。 |
| 文件夹 |     操作中涉及的源文件夹相关信息；例如如果文件夹已选中且随后删除。 |


### <a name="sharepoint-base"></a>SharePoint Base

这些属性对于所有 SharePoint 记录通用。

| 属性 | 说明 |
|:--- |:--- |
| OfficeWorkload | SharePoint |
| OfficeWorkload | SharePoint |
| EventSource | 确定事件在 SharePoint 中发生。 可能的值有 SharePoint 或 ObjectModel。 |
| ItemType | 访问或修改的对象的类型。 请参阅 ItemType 表，详细了解对象类型。 |
| MachineDomainInfo | 有关设备同步操作的信息。 仅在此信息出现在请求中时，才对其报告。 |
| MachineId |   有关设备同步操作的信息。 仅在此信息出现在请求中时，才对其报告。 |
| Site_ | 用户访问的文件或文件夹所在的站点的 GUID。 |
| Source_Name | 触发已审核操作的实体。 可能的值有 SharePoint 或 ObjectModel。 |
| UserAgent | 用户客户端或浏览器的相关信息。 此信息由客户端或浏览器提供。 |


### <a name="sharepoint-schema"></a>SharePoint 架构

对 SharePoint 进行配置更改时，将创建这些记录。

| 属性 | 说明 |
|:--- |:--- |
| OfficeWorkload | SharePoint |
| OfficeWorkload | SharePoint |
| CustomEvent | 自定义事件的可选字符串。 |
| Event_Data |  自定义事件的可选有效负载。 |
| ModifiedProperties | 包含此属性用于管理员事件，例如将用户添加为网站成员或网站集管理员组的成员。 该属性包括已修改的属性的名称（例如网站管理员组）、已修改属性的新值（例如已添加为网站管理员的用户），以及已修改的对象之前的值。 |


### <a name="sharepoint-file-operations"></a>SharePoint 文件操作

响应 SharePoint 中的文件操作时，将创建这些记录。

| 属性 | 说明 |
|:--- |:--- |
| OfficeWorkload | SharePoint |
| OfficeWorkload | SharePointFileOperation |
| DestinationFileExtension | 复制或移动的文件的文件扩展名。 此属性仅对 FileCopied 和 FileMoved 事件显示。 |
| DestinationFileName | 复制或移动的文件的名称。 此属性仅对 FileCopied 和 FileMoved 事件显示。 |
| DestinationRelativeUrl | 复制或移动文件的目标文件夹的 URL。 SiteURL、DestinationRelativeURL 和 DestinationFileName 参数值的组合与 ObjectID 属性的值相同，即为已复制文件的完整路径名称。 此属性仅对 FileCopied 和 FileMoved 事件显示。 |
| SharingType | 分配给与其共享资源的用户的共享权限类型。 此用户由 UserSharedWith 参数标识。 |
| Site_Url | 用户访问的文件或文件夹所在的站点的 URL。 |
| SourceFileExtension | 用户访问的文件的文件扩展名。 如果所访问的对象是文件夹，则此属性为空。 |
| SourceFileName |  用户访问的文件或文件夹的名称。 |
| SourceRelativeUrl | 包含用户访问的文件的文件夹 URL。 SiteURL、SourceRelativeURL 和 SourceFileName 参数值的组合与 ObjectID 属性的值相同，即为用户访问的文件的完整路径名称。 |
| UserSharedWith |  与其共享资源的用户。 |




## <a name="sample-log-queries"></a>示例日志查询

下表提供了此解决方案收集的更新记录的示例日志查询。

| 查询 | 说明 |
| --- | --- |
|Office 365 订阅上所有操作的计数 |OfficeActivity &#124; summarize count() by Operation |
|SharePoint 网站的使用情况|OfficeActivity &#124; where OfficeWorkload =~ "sharepoint" &#124; summarize count() by SiteUrl \| sort by Count asc|
|文件访问操作数（按用户类型） | OfficeActivity &#124; summarize count() by UserType |
|监视 Exchange 上的外部操作|OfficeActivity &#124; where OfficeWorkload =~ "exchange" and ExternalAccess == true|



## <a name="next-steps"></a>后续步骤

* 使用 [Azure Monitor 中的日志查询](../log-query/log-query-overview.md)查看详细的更新数据。
* [创建警报](../platform/alerts-overview.md)，主动接收重要的 Office 365 活动通知。  




