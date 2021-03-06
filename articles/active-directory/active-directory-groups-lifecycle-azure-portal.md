---
title: "Azure Active Directory 中的 Office 365 组过期预览 | Microsoft Docs"
description: "如何在 Azure Active Directory 中为 Office 365 组设置过期（预览版）"
services: active-directory
documentationcenter: 
author: curtand
manager: femila
editor: 
ms.assetid: 
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/09/2017
ms.author: curtand
ms.reviewer: kairaz.contractor
ms.custom: it-pro
ms.openlocfilehash: 8a43df84fd050d7b4bd8d937b8c55e744cb805d3
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="configure-office-365-groups-expiration-preview"></a>配置 Office 365 组过期（预览版）

现在可以通过为所选的任何 Office 365 组设置过期来管理 Office 365 组的生命周期。 设置过期后，会要求组的所有者续订这些组（如果仍需要）。 将删除所有未续订的 Office 365 组。 组所有者或管理员可在 30 日内还原任何被删除的 Office 365 组。  


> [!NOTE]
> 可以仅为 Office 365 组设置过期。
>
> 要为 O365 组设置过期，需要先将 Azure AD Premium 许可证分配给
>   - 为租户配置过期设置的管理员
>   - 为此设置选择的组的所有成员

## <a name="set-office-365-groups-expiration"></a>设置 Office 365 组过期

1. 使用 Azure AD 租户中的全局管理员帐户打开 [Azure AD 管理中心](https://aad.portal.azure.com)。

2. 打开 Azure AD，选择“用户和组”。

3. 选择“组设置”，然后选择“过期”以打开过期设置。
  
  ![过期边栏选项卡](./media/active-directory-groups-lifecycle-azure-portal/expiration-settings.png)

4. 在“过期”边栏选项卡中，可以：

  * 设置组的生存期（天）。 可以从预设值中任选其一，或自定义一个值（应为 31 天或以上）。 
  * 指定当组没有所有者时续订和过期通知应发送到的电子邮件地址。 
  * 选择会过期的 Office 365 组。 可以为所有 Office 365 组启用过期，从各 Office 365 组中进行选择，或选择“无”为所有组禁用过期。
  * 设置完成后，选择“保存”来保存设置。

有关如何通过 PowerShell 下载和安装 Microsoft PowerShell 模块以配置 Office 365 组过期的指南，请参阅 [Azure Active Directory V2 PowerShell Module - Public Preview Release 2.0.0.137](https://www.powershellgallery.com/packages/AzureADPreview/2.0.0.137)（Azure Active Directory V2 PowerShell 模块 - 公共预览版 2.0.0.137）。

如下电子邮件通知将在组过期前的 30 天、15 天和 1 天发送到 Office 365 组所有者。

![过期电子邮件通知](./media/active-directory-groups-lifecycle-azure-portal/expiration-notification.png)

组所有者可以选择“续订组”以续订 Office 365 组，或选择“转到组”查看关于组的成员和其他详细信息。

当某个组过期时，它将在到期日期的后一天被删除。 如下电子邮件通知将发送到 Office 365 组所有者，通知其关于 Office 365 组过期及后续删除的信息。

![组删除电子邮件通知](./media/active-directory-groups-lifecycle-azure-portal/deletion-notification.png)

可通过选择“还原组”或使用 PowerShell cmdlet 来还原组，如 [在 Azure Active Directory 中还原已删除的 Office 365 组] (https://docs.microsoft.com/azure/active-directory/active-directory-groups-restore-azure-portal) 中所述。
    
如果所还原的组包含文档、SharePoint 站点或其他持久对象，可能需要最多 24 小时才能完全还原该组及其内容。

> [!NOTE]
> * 在部署过期设置时，可能存在某些比过期时段更早的组。 这些组不会立即删除，而会被设置为在 30 天后过期。 首个续订通知电子邮件将在一天内发出。 例如，组 A 创建于 400 天前，而过期间隔设置为 180 天。 当应用过期设置后，组 A 在被删除前还有 30 天（除非所有者续订它）。
> * 当删除并还原动态组时，会将视其为一个新组并根据规则重新填充。 此过程可能最多需要 24 小时。

## <a name="next-steps"></a>后续步骤
以下文章提供有关 Azure AD 组的更多信息。

* [查看现有组](active-directory-groups-view-azure-portal.md)
* [管理组的设置](active-directory-groups-settings-azure-portal.md)
* [管理组的成员](active-directory-groups-members-azure-portal.md)
* [管理组的成员身份](active-directory-groups-membership-azure-portal.md)
* [管理组中用户的动态规则](active-directory-groups-dynamic-membership-azure-portal.md)
