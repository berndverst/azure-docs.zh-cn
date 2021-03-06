---
title: "Azure 安全中心的自适应应用程序控制 | Microsoft Docs"
description: "本文档介绍如何在 Azure 安全中心使用自适应应用程序控制将在 Azure VM 中运行的应用程序加入允许列表。"
services: security-center
documentationcenter: na
author: YuriDio
manager: mbaldwin
editor: 
ms.assetid: 9268b8dd-a327-4e36-918e-0c0b711e99d2
ms.service: security-center
ms.devlang: na
ms.topic: hero-article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/20/2017
ms.author: yurid
ms.openlocfilehash: 9c3a9a7255bbbdab8f4c356eb07022d7f1d242d7
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="adaptive-application-controls-in-azure-security-center-preview"></a>Azure 安全中心（预览版）的自适应应用程序控制
了解如何通过本演练在 Azure 安全中心配置应用程序控制。

## <a name="what-are-adaptive-application-controls-in-security-center"></a>安全中心的自适应应用程序控制是什么？
可以通过自适应应用程序控制来控制哪些应用程序能够在位于 Azure 中的 VM 上运行，这样有很多好处，其中之一是能够增强 VM 对恶意软件的抵抗力。 安全中心使用机器学习来分析在 VM 中运行的进程，帮助你运用此智能来应用允许列表规则。 此功能大大简化配置和维护应用程序允许列表的过程，让你可以：

- 阻止运行恶意应用程序的尝试（包括在其他情况下可能会被反恶意软件解决方案遗漏的尝试）或者向用户发出此方面的警报。
- 遵循组织要求只能使用许可软件的安全策略。
- 避免在环境中使用不需要的软件。
- 避免运行旧的不受支持的应用。
- 防止使用组织不允许的特定软件工具。
- 允许 IT 部门控制用户使用应用来访问敏感数据。

> [!NOTE]
> 自适应应用程序控制以受限公共预览版的形式提供给 Azure 安全中心标准版客户。 若要体验预览版，请向[我们](mailto:ASC_appcontrol@microsoft.com)发送包含订阅 ID 的电子邮件。

## <a name="how-to-enable-adaptive-application-controls"></a>如何启用自适应应用程序控制？
可以通过自适应应用程序控制来定义一组应用程序，允许这些应用程序在配置的资源组上运行。 此功能仅适用于 Windows 计算机（所有版本，不管是经典部署模型还是 Azure 资源管理器部署模型）。 请执行以下步骤，在安全中心配置应用程序允许列表功能：

1.  打开“安全中心”仪表板，然后单击“概览”。
2.  “高级云防御”下的“自适应应用程序控制”磁贴显示有多少 VM（相对于所有 VM 来说）目前已将控制设置到位。 它还显示在上一周发现的问题数： 

    ![自适应应用程序控制](./media/security-center-adaptive-application\security-center-adaptive-application-fig1.png)

3. 单击“自适应应用程序控制”磁贴可查看更多选项。

    ![controls](./media/security-center-adaptive-application/security-center-adaptive-application-fig2.png)

4. “资源组”部分包含三个选项卡：
    * 已建议：建议对其实施应用程序控制的资源组的列表。 安全中心使用机器学习，根据 VM 是否一致地运行相同应用程序来确定适用于应用程序控制的 VM。
    * 已配置：所含 VM 已配置了应用程序控制的资源组的列表。 
    * 无建议：所含 VM 没有任何应用程序控制建议的资源组的列表。 例如，其上的应用程序始终变化，尚未达到稳定状态的 VM。

以下部分会更详细地介绍每个选项及其用法。

### <a name="configure-a-new-application-control-policy"></a>配置新的应用程序控制策略
单击“已建议”选项卡会出现一个列表，其中包含的资源组带有应用程序控制建议：

![建议](./media/security-center-adaptive-application/security-center-adaptive-application-fig3.png)

此列表包括：
- 名称：订阅和资源组的名称
- VM 数：资源组中虚拟机的数目
- 状态：建议的状态，大多数情况下为“开放”
- 严重性：建议的严重性级别

选择一个资源组，打开“创建应用程序控制规则”选项：

![应用程序控制规则](./media/security-center-adaptive-application/security-center-adaptive-application-fig4.png)

在“选择 VM”中查看建议的 VM 的列表，取消选中不需向其应用应用程序控制的 VM。 在“选择适用于允许列表规则的进程”中查看建议的应用程序的列表，取消选中不需向其应用应用程序控制的应用程序。 此列表包括：

- 名称：完整的应用程序路径
- 进程数：每个路径中驻留的应用程序数
- 常用：true 表示这些进程已在此资源组的大多数 VM 上执行。
- 可利用：警告图标表示攻击者可能会利用应用程序来规避应用程序允许列表。 强烈建议在核准这些应用程序之前对其进行审查。 

选择完以后，请单击“创建”按钮。 安全中心始终默认在“审核”模式下启用应用程序控制。 在验证允许列表对工作负荷没有任何负面影响之后，即可更改为“强制”模式。

> [!NOTE]
> 安全中心始终会遵循安全方面的最佳做法，尝试为应该加入允许列表的应用程序创建发布者规则，只有在应用程序没有发布者信息（即未签名）的情况下，才会为特定 EXE 的完整路径创建路径规则。
>   

### <a name="editing-and-monitoring-a-group-configured-with-application-control"></a>编辑和监视配置了应用程序控制的组

若要编辑和监视配置了应用程序控制的组，请在“资源组”下单击“已配置”：

![资源组](./media/security-center-adaptive-application/security-center-adaptive-application-fig5.png)

此列表包括：

- 名称：订阅和资源组的名称
- VM 数：资源组中虚拟机的数目
- 模式：“审核”模式会记录运行未加入允许列表的应用程序的尝试；“阻止”模式会阻止未加入允许列表的应用程序运行
- 严重性：建议的严重性级别

选择一个资源组，在“编辑应用程序控制策略”页中进行更改。

![保护](./media/security-center-adaptive-application/security-center-adaptive-application-fig6.png)

在“保护模式”下，可以在以下选项之间进行选择：
- 审核：在此模式下，应用程序控制解决方案不会强制实施规则，只审核受保护 VM 上的活动。 如果在阻止应用在目标 VM 中运行之前需先观察总体行为，建议使用此模式。
- 强制：在此模式下，应用程序控制解决方案会强制实施规则，确保阻止不允许运行的应用程序。 

如前所述，默认情况下，新的应用程序控制策略会始终在“审核”模式下配置。 可以在“策略扩展”下添加自己的需要加入允许列表的应用程序路径。 添加这些路径以后，安全中心会在已有规则的基础上为这些应用程序创建适当的规则。 在“问题”部分会列出任何当前的冲突。

![问题](./media/security-center-adaptive-application/security-center-adaptive-application-fig7.png)

这些应用程序包括：

- 问题：已记录的任何冲突，可能包括：
    - ViolationsBlocked：当解决方案启用“强制”模式时，尝试执行未加入允许列表的应用程序所出现的冲突。
    - ViolationsAudited：当解决方案启用“审核”模式时，执行未加入允许列表的应用程序所出现的冲突。
    - RulesViolatedManually：当用户尝试在 VM 上手动配置规则（而不是通过 ASC 管理门户进行配置）时出现的冲突。
- **VM 和数：有此类问题的虚拟机数。

单击这其中的每一行都会重定向到 [Azure 活动日志](https://docs.microsoft.com/azure/monitoring-and-diagnostics/monitoring-overview-activity-logs)页，其中显示具有此类冲突的所有 VM 的相关信息。 如果单击每行末尾的三点符号，则可删除该特定条目。 “已配置虚拟机”部分列出了应用这些规则的 VM。 

![已配置虚拟机](./media/security-center-adaptive-application/security-center-adaptive-application-fig8.png)

“发布者允许列表规则”列出了根据证书信息为其创建发布者规则的应用程序，这些信息是为每个应用程序找到的。 有关详细信息，请参阅 [Understanding Publisher Rules in Applocker](https://docs.microsoft.com/windows/device-security/applocker/understanding-the-publisher-rule-condition-in-applocker)（了解 Applocker 中的发布者规则）。

![允许列表规则](./media/security-center-adaptive-application/security-center-adaptive-application-fig9.png)

如果单击每行末尾的三点符号，则可删除该特定规则。 “路径允许列表规则”列出了整个应用程序路径（包括可执行文件），该路径适用于未使用数字证书进行签名但目前仍在允许列表规则中的应用程序。 

> [!NOTE]
> 默认情况下，安全中心始终会遵循安全方面的最佳做法，尝试为应该加入允许列表的 EXE 创建发布者规则，只有在 EXE 没有发布者信息（即未签名）的情况下，才会为特定 EXE 的完整路径创建路径规则。

![路径允许列表规则](./media/security-center-adaptive-application/security-center-adaptive-application-fig10.png)

此列表包含：
- 名称：可执行文件的完整路径
- 可利用：true 表示攻击者可能会利用应用程序来规避应用程序允许列表。  

如果单击每行末尾的三点符号，则可删除该特定规则。 进行更改后，可单击“保存”按钮；如果决定不应用所做的更改，则可单击“放弃”。

### <a name="not-recommended-list"></a>“不建议”列表

安全中心只会为运行稳定的一组应用程序的虚拟机建议应用程序允许列表。 如果已关联 VM 上的应用程序始终变化不定，则不会创建建议。 

![建议](./media/security-center-adaptive-application/security-center-adaptive-application-fig11.png)

此列表包含：
- 名称：订阅和资源组的名称。
- VM 数：资源组中虚拟机的数目

## <a name="see-also"></a>另请参阅
本文档介绍了如何在 Azure 安全中心使用自适应应用程序控制将在 Azure VM 中运行的应用程序加入允许列表。 若要了解更多有关 Azure 安全中心的详细信息，请参阅以下内容：

* [Managing and responding to security alerts in Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-managing-and-responding-alerts)（管理和响应 Azure 安全中心的安全警报）。 了解如何管理警报并响应安全中心的安全事件。
* [Security health monitoring in Azure Security Center](security-center-monitoring.md)（在 Azure 安全中心进行安全运行状况监视）。 了解如何监视 Azure 资源的运行状况。
* [了解 Azure 安全中心中的安全警报](https://docs.microsoft.com/azure/security-center/security-center-alerts-type)。 了解不同类型的安全警报。
* [Azure 安全中心故障排除指南](https://docs.microsoft.com/azure/security-center/security-center-troubleshooting-guide)。 了解如何排查安全中心的常见问题。 
* [Azure Security Center FAQ](security-center-faq.md)（Azure 安全中心常见问题）。 查找有关如何使用服务的常见问题。
* [Azure 安全性博客](http://blogs.msdn.com/b/azuresecurity/)。 查找关于 Azure 安全性及合规性的博客文章。

