---
title: "Azure 存储可伸缩性和性能目标 | Microsoft Docs"
description: "了解有关 Azure 存储帐户的可伸缩性和性能目标的信息，包括标准和高级存储账户的容量、请求速率以及入站和出站带宽。 了解每个 Azure 存储服务中各分区的性能目标。"
services: storage
documentationcenter: na
author: tamram
manager: timlt
editor: tysonn
ms.assetid: be721bd3-159f-40a1-88c1-96418537fe75
ms.service: storage
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage
ms.date: 07/12/2017
ms.author: tamram
ms.openlocfilehash: 1ed933493da1842201bb9293f514ea4d0e7a75ce
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="azure-storage-scalability-and-performance-targets"></a>Azure 存储可伸缩性和性能目标
## <a name="overview"></a>概述
本主题介绍 Microsoft Azure 存储的可伸缩性和性能主题。 有关其他 Azure 限制的摘要，请参阅 [Azure 订阅和服务限制、配额与约束](../../azure-subscription-service-limits.md)。

> [!NOTE]
> 所有存储帐户都在新的扁平网络拓扑上运行，无论它们创建于何时，都支持下文概述的可伸缩性和性能目标。 有关 Azure 存储的扁平网络体系结构和可伸缩性的详细信息，请参阅 [Microsoft Azure Storage: A Highly Available Cloud Storage Service with Strong Consistency](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)（Microsoft Azure 存储：具有高度一致性的高可用云存储服务）。
> 
> [!IMPORTANT]
> 以下所列的可伸缩性和性能目标为高端目标，但却是能够实现的。 在任何情况下，存储帐户实现的请求速率和带宽取决于存储对象大小、使用的访问模式、应用程序执行的工作负荷类型。 请务必测试服务，以确定其性能是否达到要求。 如果可能，应避免流量速率突发峰值，并确保流量在各个分区上均匀分布。
> 
> 当应用程序达到分区能够处理的工作负荷极限时，Azure 存储将开始返回错误代码 503（服务器忙）或错误代码 500（操作超时）响应。 发生这种情况时，应用程序应使用指数退让策略进行重试。 使用指数退让策略，可以减少分区上的负载，缓解该分区的流量高峰。
> 
> 

如果应用程序的需求超过单个存储帐户的伸缩性目标，则在构建时让应用程序使用多个存储帐户，并将数据对象分布到这些存储帐户中。 有关批量定价的信息，请参阅 [Azure 存储定价](https://azure.microsoft.com/pricing/details/storage/) 。

## <a name="scalability-targets-for-blobs-queues-tables-and-files"></a>Blob、队列、表和文件的可伸缩性目标
[!INCLUDE [azure-storage-limits](../../../includes/azure-storage-limits.md)]

<!-- conceptual info about disk limits -- applies to unmanaged and managed -->
## <a name="scalability-targets-for-virtual-machine-disks"></a>虚拟机磁盘的可伸缩性目标
[!INCLUDE [azure-storage-limits-vm-disks](../../../includes/azure-storage-limits-vm-disks.md)]

请参阅 [Windows VM 大小](../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)或 [Linux VM 大小](../../virtual-machines/windows/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)了解其他详细信息。

## <a name="managed-virtual-machine-disks"></a>托管虚拟机磁盘

[!INCLUDE [azure-storage-limits-vm-disks-managed](../../../includes/azure-storage-limits-vm-disks-managed.md)]

## <a name="unmanaged-virtual-machine-disks"></a>非托管虚拟机磁盘
[!INCLUDE [azure-storage-limits-vm-disks-standard](../../../includes/azure-storage-limits-vm-disks-standard.md)]

[!INCLUDE [azure-storage-limits-vm-disks-premium](../../../includes/azure-storage-limits-vm-disks-premium.md)]

## <a name="scalability-targets-for-azure-resource-manager"></a>Azure Resource Manager 的可伸缩性目标
[!INCLUDE [azure-storage-limits-azure-resource-manager](../../../includes/azure-storage-limits-azure-resource-manager.md)]

## <a name="partitions-in-azure-storage"></a>Azure 存储中的分区
可容纳存储在 Azure 存储中的数据的每个对象（Blob、消息、实体和文件）都属于某个分区，可用分区键进行标识。 分区决定了 Azure 存储如何在多个服务器之间实现 Blob、消息、实体和文件的负载均衡，以满足这些对象的流量需求。 分区键是唯一的，用于查找 Blob、消息或实体。

上面[标准存储帐户的可伸缩性目标](#standard-storage-accounts)中所示的表列出了每项服务在单个分区的性能目标。

分区会对每个存储服务的负载均衡和可伸缩性产生以下影响：

* **Blob**：Blob 的分区键是帐户名称 + 容器名称 + Blob 名称。 这意味着如果 Blob 上的负载需要，每个 Blob 都可以具有其自己的分区。 尽管可以在众多服务器间分布 Blob 以便扩大对其的访问权限，但只能由单个服务器为单个 Blob 提供服务。 虽然 Blob 可在 Blob 容器中进行逻辑分组，但这种分组不会对分区产生影响。
* **文件**：文件的分区键是帐户名称 + 文件共享名称。 这意味着一个文件共享中的所有文件也都位于单个分区。
* **消息**：消息的分区键是帐户名称 + 队列名称，因此一个队列中的所有消息都分组到单个分区中，由单个服务器提供服务。 不同队列可以由不同服务器处理，无论存储帐户有多少队列，都可以平衡负载。
* **实体**：实体的分区键是帐户名称 + 表名称 + 分区键，其中，分区键是实体所需的用户定义的 **PartitionKey** 属性。 具有相同分区键值的所有实体都分组到同一分区中，并由同一个分区服务器提供服务。 在设计应用程序的过程中，了解这一点非常重要。 将实体分布在多个分区中能够实现伸缩性优势，而将实体分组到单个分区中则能够提供数据访问优势，需要平衡应用程序的这两大优势。  

将表中的一组实体分组到单个分区中的一大关键优势是能够对位于同一分区中的实体执行原子操作，因为一个分区存在于单个服务器上。 因此，如果想在一组实体上执行批处理操作，请考虑对具有相同分区键的实体进行分组。 

另一方面，位于同一表中但具有不同分区键的实体可在不同服务器之间实现负载均衡，因而可能具有更好的伸缩性。

关于表的设计分区策略的详细建议可在[此处](https://msdn.microsoft.com/library/azure/hh508997.aspx)找到。

## <a name="see-also"></a>另请参阅
* [存储定价详细信息](https://azure.microsoft.com/pricing/details/storage/)
* [Azure 订阅和服务限制、配额和约束](../../azure-subscription-service-limits.md)
* [高级存储：适用于 Azure 虚拟机工作负荷的高性能存储](../storage-premium-storage.md)
* [Azure 存储复制](../storage-redundancy.md)
* [Microsoft Azure 存储性能和可伸缩性清单](../storage-performance-checklist.md)
* [Microsoft Azure 存储：具有高度一致性的高可用云存储服务](http://blogs.msdn.com/b/windowsazurestorage/archive/2011/11/20/windows-azure-storage-a-highly-available-cloud-storage-service-with-strong-consistency.aspx)

