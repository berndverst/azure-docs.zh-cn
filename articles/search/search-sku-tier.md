---
title: "选择 Azure 搜索的 SKU 或定价层 | Microsoft Docs"
description: "Azure 搜索可在以下 SKU 上进行预配：“免费”、“基本”和“标准”，其中“标准”在各种资源配置和容量级别中均可用。"
services: search
documentationcenter: 
author: HeidiSteen
manager: jhubbard
editor: 
tags: azure-portal
ms.assetid: 8d4b7bca-02a5-43ee-b3f8-03551dfb32fd
ms.service: search
ms.devlang: NA
ms.workload: search
ms.topic: article
ms.tgt_pltfrm: na
ms.date: 10/24/2016
ms.author: heidist
ms.openlocfilehash: f9f3a7b2369818791ffac1c8eeccef45216c2ff0
ms.sourcegitcommit: f537befafb079256fba0529ee554c034d73f36b0
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/11/2017
---
# <a name="choose-a-sku-or-pricing-tier-for-azure-search"></a>选择 Azure 搜索的 SKU 或定价层
在 Azure 搜索中，[服务已在特定的定价层或 SKU 上进行预配](search-create-service-portal.md)。 选项包括“免费”、“基本”或“标准”，其中“标准”在多个配置和容量中均可用。

本文旨在帮助你选择某一层。 如果层容量经证实过低，将需要在更高的层上预配新服务，然后重新加载索引。 相同的服务无法从一个 SKU 就地升级到另一个。

> [!NOTE]
> 选择层并[预配搜索服务](search-create-service-portal.md)后，可在该服务内增加副本和分区计数。 有关指南，请参阅[缩放资源级别以便查询工作负荷并编制索引](search-capacity-planning.md)。
>
>

## <a name="how-to-approach-a-pricing-tier-decision"></a>如何进行定价层决定
在 Azure 搜索中，层确定容量而非功能可用性。 通常情况下，每个层都会提供功能，包括预览功能。 一个例外是，不支持 S3 HD 中的索引器。

> [!TIP]
> 我们建议始终预配“免费”服务（每个订阅一个，不会过期）以便可随时用于轻量项目。 将“免费”服务用于测试和评估；在“基本”或“标准”层上创建第二个可计费服务以便用于生产或更大的测试工作负荷。
>
>

容量与运行服务的成本密切相关。 本文中的信息可帮助你决定哪一个 SKU 可提供适当的平衡，但如果要使任何一个都有用，将至少需要粗略估计以下内容：

* 计划创建的索引的数目和大小
* 要上传的文档的数目和大小
* 查询量的一些概念，即每秒查询次数 (QPS)

数目和大小非常重要，因为可以通过服务中索引或文档计数的严格限制，或服务所使用的资源（存储或副本）上索引或文档计数的严格限制，达到最大值限制。 服务的实际限制将是首先耗尽的项目：资源或对象。

在进行了评估后，以下步骤应该能简化过程：

* **步骤 1** 查看以下 SKU 描述，了解可用选项。
* **步骤 2** 回答以下问题，以便做出初步的决定。
* **步骤 3** 通过查看存储和定价的严格限制，最后落实决定。

## <a name="sku-descriptions"></a>SKU 描述
下表提供每个层的描述。

| 层 | 主要方案 |
| --- | --- |
| **免费** |共享的服务，不收费，用于评估、调查或少量工作负载。 由于它是与其他订阅者共享的，因此查询吞吐量和索引会因使用该服务的其他用户而异。 容量很小（50 MB 或各具有最多 10000 个文档的 3 个索引）。 |
| **基本** |专用硬件上的小量生产工作负荷。 高可用性。 容量最多可容纳 3 个副本和 1 个分区 (2 GB)。 |
| **S1** |标准 1 支持分区 (12) 和副本 (12) 的弹性组合，用于专用硬件上的中等生产工作负荷。 可以根据计费搜索单位最大数目 36 所支持的组合来分配分区和副本。 在此级别上，每个分区为 25 GB，而 QPS 是大约每秒 15 次查询。 |
| **S2** |标准 2 使用与 S1 相同的 36 个搜索单位来运行较大的生产工作负荷，不过使用较大的分区和副本。 在此级别上，每个分区为 100 GB，而 QPS 是大约每秒 60 次查询。 |
| **S3** |标准 3 在更高端的系统上按比例运行更大的生产工作负荷，具体配置如下：在 36 个搜索单位下最高可达 12 个分区或 12 个副本。 在此级别上，每个分区为 200 GB，而 QPS 是每秒超过 60 次查询。 |
| **S3 HD** |标准 3 高密度专用于大量的较小索引。 最多可具有 3 个分区，每个各 200 GB。 QPS 是每秒超过 60 次查询。 |

> [!NOTE]
> 副本和分区最大值会按搜索单位计费（每个服务最多 36 个单位），这会强制执行比表面上所指最大限制更低的有效限制。 例如，若要使用最多 12 个副本，最多可以有 3 个分区（12 * 3 = 36 个单位）。 同样地，要使用最大分区数，则将副本数减少为 3。 有关允许组合的图表，请参阅[在 Azure 搜索中缩放资源级别以便查询工作负荷并编制索引](search-capacity-planning.md)。
>
>

## <a name="review-limits-per-tier"></a>查看每个层的限制
下图是 [Azure 搜索的服务限制](search-limits-quotas-capacity.md)中的限制子集。 它列出最有可能影响 SKU 决策的因素。 查看以下问题时，可参阅此图表。

| 资源 | 免费 | 基本 | S1 | S2 | S3 | S3 HD |
| --- | --- | --- | --- | --- | --- | --- |
| 服务级别协议 (SLA) |否 <sup>1</sup> |是 |是 |是 |是 |是 |
| 索引限制 |3 |5 |50 |200 |200 |1000 <sup>2</sup> |
| 文档限制 |总计 10,000 |每个服务 100 万 |每个分区 1500 万个 |每个分区 6000 万 |每个分区 1.2 亿 |每个索引 100 万 |
| 分区数上限 |不适用 |1 |12 |12 |12 |3 <sup>2</sup> |
| 分区大小 |总计 50 MB |每个服务 2 GB |每个分区 25 GB |每个分区 100 GB（每个服务最多 1.2 TB） |每个分区 200 GB（每个服务最多 2.4 TB） |200 GB（每个服务最多 600 GB） |
| 副本数上限 |不适用 |3 |12 |12 |12 |12 |
| 每秒查询次数 |不适用 |每个副本大约 3 个 |每个副本大约 15 个 |每个副本大约 60 个 |每个副本大于 60 个 |每个副本大于 60 个 |

<sup>1</sup> 免费层和预览功能不带服务级别协议 (SLA)。 对于所有可计费的层，SLA 将在用户为服务提供足够冗余时生效。 查询（读取）SLA 需要两个或多个副本。 查询和索引（读-写）SLA 需要不少于三个副本。 分区数不属于 SLA 相关考虑因素。 

<sup>2</sup> S3 和 S3 HD 受相同高容量基础结构支持，但各采用不同的方式达到其最大限制。 S3 面向较小数目的非常大的索引。 在这种情况下，其最大限制是资源限制型（每个服务 2.4 TB）。 S3 HD 面向较大数目的非常小的索引。 在 1,000 个索引的情况下，S3 HD 达到其索引约束形式的限制。 如果 S3 HD 客户需要超过 1,000 个索引，请联系 Microsoft 支持，了解有关如何继续的信息。

## <a name="eliminate-skus-that-dont-meet-requirements"></a>消除不满足要求的 SKU
以下问题有助于根据工作负荷做出正确的 SKU 决策。

1. 是否有**服务级别协议 (SLA)** 要求？ 可以使用任何计费层（基本以上），但必须为冗余配置服务。 查询（读取）SLA 需要两个或多个副本。 查询和索引（读-写）SLA 需要不少于三个副本。 分区数不属于 SLA 相关考虑因素。
2. 需要**多少索引**？ 其中一个纳入 SKU 决策的最大变量是每个 SKU 支持的索引数。 索引支持在较低的定价层明显属于不同级别。 索引数方面的要求可能是 SKU 决策的主要决定因素。
3. **有多少个文档**将加载到每个索引？ 文档的数量和大小将确定索引的最终大小。 假设可估算预计的索引大小，则可以将该数字与每个 SKU 的分区大小进行比较，并根据存储该索引大小所需的分区数进行扩展。
4. **什么是预期的查询负载**？ 了解存储要求后，请考虑查询工作负荷。 S2 和两个 S3 SKU 提供几乎相等的吞吐量，但 SLA 要求会排除任何预览 SKU。
5. 如果正在考虑 S2 或 S3 层，请确定是否需要[索引器](search-indexer-overview.md)。 索引器在 S3 HD 层上还不能使用。 另一种方法是对索引更新使用推送模型，在该模型中编写应用程序代码以将数据集推送到索引。

大多数客户可以根据对四个问题的答案来纳入或排除特定的 SKU。 如果仍然不确定要使用哪一个 SKU，可以将问题发布到 MSDN 或 StackOverflow 论坛，也可以联系 Azure 支持以寻求进一步的指导。

## <a name="decision-validation-does-the-sku-offer-sufficient-storage-and-qps"></a>决策验证：SKU 是否提供足够的存储和 QPS？
最后一步是重新访问[定价页](https://azure.microsoft.com/pricing/details/search/)和[服务限制中的每个服务和每个索引部分](search-limits-quotas-capacity.md)，仔细检查对订阅帐户与服务限制的估计。

如果价格或存储要求超出界限，用户可能希望对多个较小服务之间的工作负荷进行重构（举例来说）。 在更细微的级别上，可以将索引重新设计为更小，或使用筛选器让查询更有效。

> [!NOTE]
> 如果文档包含无关数据，存储要求可能会过高。 在理想情况下，文档仅包含可搜索的数据或元数据。 二进制数据不可搜索，应该分开存储（或许存储在 Azure 表或 blob 存储中），并且在索引中要有一个字段用于保存外部数据的 URL 参考。 个别文档的最大大小是 16 MB（如果在一次请求中批量上传了多个文档，则小于 16 MB）。 有关详细信息，请参阅 [Azure 搜索中的服务限制](search-limits-quotas-capacity.md)。
>
>

## <a name="next-step"></a>后续步骤
在了解哪些 SKU 最适合后，请继续执行以下步骤：

* [在门户中创建搜索服务](search-create-service-portal.md)
* [更改分区和副本的分配以扩展服务](search-capacity-planning.md)
