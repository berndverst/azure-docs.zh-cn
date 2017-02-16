---
title: "了解 Azure 外部服务收费 | Microsoft Docs"
description: "了解 Azure 中外部服务（以前称为应用商店）的计费。"
services: 
documentationcenter: 
author: adpick
manager: ruchic
editor: 
tags: billing
ms.assetid: 5e0e2a3c-d111-4054-8508-0c111c1b749b
ms.service: billing
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/12/2016
ms.author: adpick
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 45e0d553179b63de2d0664314271472f1151d73f


---
# <a name="understand-your-azure-external-service-charges"></a>了解 Azure 外部服务收费
本文介绍 Azure 中外部服务的计费。 外部服务过去称为应用商店订单。 外部服务由独立服务供应商提供，但已完全集成到 Azure 生态系统中。 了解如何：

* 确定外部服务
* 了解此计费方式与其他 Azure 资源的计费方式有何不同
* 查看和跟踪使用外部服务时产生的任何费用
* 管理外部服务订单及其付款方式

## <a name="what-are-azure-external-services"></a>什么是 Azure 外部服务？
外部服务过去称为 Azure 应用商店。 通常情况下，外部服务是第三方发布的适用于 Azure 的服务。 例如，ClearDB 和 SendGrid 是可以在 Azure 中购买的外部服务，但不是 Microsoft 发布的。

### <a name="identify-external-services"></a>确定外部服务
预配新的外部服务或资源时，会显示警告：

![应用商店购买警告](./media/billing-understand-your-azure-marketplace-charges/marketplace-warning.PNG)

> [!NOTE]
> 外部服务是 Microsoft 以外的公司发布的，但有时候，Microsoft 产品也会被归类为外部服务。
> 
> 

### <a name="external-services-are-billed-separately"></a>外部服务单独计费
外部服务属于 Azure 订阅中的单个订单。 购买每个服务时，会设置该服务的计费周期。 请勿将其与购买的服务所属的订阅的计费周期混淆。 用户会收到单独的帐单，信用卡也会单独收费。

### <a name="each-external-service-has-a-different-billing-model"></a>每个外部服务都有不同的计费模式
某些服务按即用即付方式计费，某些服务则使用按月付款模式。 需要使用信用卡购买 Azure 外部服务，不能使用发票付款方式购买外部服务。

### <a name="you-cant-use-monthly-free-credits-for-external-services"></a>购买外部服务时不能使用每月的免费信用额度
如果所使用的 Azure 订阅包含[免费信用额度](https://azure.microsoft.com/pricing/spending-limits/)，则不能将其用于外部服务帐单。 使用信用卡购买外部服务。

## <a name="view-external-service-spending-and-history"></a>查看外部服务支出和历史记录
可以在 [Azure 门户](https://portal.azure.com/)中查看每个订阅的外部服务的列表： 

1. 以帐户管理员身份登录到 [Azure 门户](https://portal.azure.com/)。
2. 在“中心”菜单中，选择“订阅”。
   
    ![在“中心”菜单中，选择“订阅”](./media/billing-understand-your-azure-marketplace-charges/sub-button.png) 
3. 在“订阅”边栏选项卡中，选择要查看的订阅，然后选择“外部服务”。
   
    ![在计费边栏选项卡中选择订阅](./media/billing-understand-your-azure-marketplace-charges/select-sub-external-services.png)
4. 此时会看到每个外部服务订单、发布者名称、购买的服务层、提供给资源的名称，以及当前订单状态。 选择一项外部服务即可查看过去的帐单。
   
    ![选择外部服务](./media/billing-understand-your-azure-marketplace-charges/external-service-blade2.png)
5. 在此处可查看过去的帐单金额，包括税款明细。
   
    ![查看外部服务帐单](./media/billing-understand-your-azure-marketplace-charges/billing-overview-blade.png)

## <a name="manage-payment-methods-for-external-service-orders"></a>管理外部服务订单的付款方式
在[帐户中心](https://account.windowsazure.com/)更新外部服务订单的付款方式。

> [!NOTE]
> 如果是使用工作帐户或学校帐户购买的订阅，则应[联系支持人员](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)，对付款方式进行更改。
> 
> 

1. 登录到[帐户中心](https://account.windowsazure.com/)，[导航到**应用商店**边栏选项卡](https://account.windowsazure.com/Store)
   
    ![在帐户中心选择“应用商店”](./media/billing-understand-your-azure-marketplace-charges/select-marketplace.png)
2. 选择要管理的外部服务
   
    ![选择要管理的外部服务](./media/billing-understand-your-azure-marketplace-charges/select-ext-service.png)
3. 在页面右侧，单击“更改付款方式”。 此链接指向另一管理付款方式的门户。
   
    ![订单摘要](./media/billing-understand-your-azure-marketplace-charges/change-payment.PNG)
4. 单击“编辑信息”，按说明更新付款信息。
   
    ![选择“编辑信息”](./media/billing-understand-your-azure-marketplace-charges/edit-info.png)

## <a name="cancel-an-external-service-order"></a>取消外部服务订单
若要取消外部服务订单，需在 [Azure 门户](https://portal.azure.com)中删除资源。

![删除资源](./media/billing-understand-your-azure-marketplace-charges/deleteMarketplaceOrder.PNG)

## <a name="need-help-contact-support"></a>需要帮助？ 联系支持人员。
如果仍有疑问，请[联系支持人员](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade)以快速解决问题。




<!--HONumber=Nov16_HO3-->

