---
title: "Azure Monitor 中的 Azure 存储指标 | Microsoft Docs"
description: "了解 Azure Monitor 提供的新指标。"
services: storage
documentationcenter: na
author: fhryo-msft
manager: cbrooks
editor: fhryo-msft
ms.assetid: 
ms.service: storage
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage
ms.date: 09/05/2017
ms.author: fryu
ms.openlocfilehash: d30a99044e335723e5d2c4bbd71fab7e4fd51145
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="azure-storage-metrics-in-azure-monitor-preview"></a>Azure Monitor 中的 Azure 存储指标（预览）

使用 Azure 存储的指标可以分析使用趋势、跟踪请求，以及诊断存储帐户的问题。

Azure Monitor 提供统一的用户界面用于监视不同的 Azure 服务。 有关详细信息，请参阅 [Azure Monitor](../../monitoring-and-diagnostics/monitoring-overview.md)。 Azure 存储通过向 Azure Monitor 平台发送指标数据来与 Azure Monitor 集成。

## <a name="access-metrics"></a>访问指标

Azure Monitor 提供多种访问指标的方法。 可从 [Azure 门户](https://portal.azure.com)、Azure Monitor API（REST 和 .Net）与分析解决方案（例如 Operation Management Suite 和事件中心）访问指标。 有关详细信息，请参阅 [Azure Monitor 指标](../../monitoring-and-diagnostics/monitoring-overview-metrics.md)。

默认已启用指标，可以访问最近 30 天的数据。 如需将数据保留更长一段时间，可将指标数据存档到 Azure 存储帐户。 可在 Azure Monitor 的“诊断设置”中完成这种配置。[](../../monitoring-and-diagnostics/monitoring-overview-of-diagnostic-logs.md#resource-diagnostic-settings)

### <a name="access-metrics-in-the-azure-portal"></a>在 Azure 门户中访问指标

可在 Azure 门户中访问不同时间的指标。 以下演示如何查看帐户级别的 **UsedCapacity**。

![在 Azure 门户中访问指标的屏幕截图](./media/storage-metrics-in-azure-monitor/access-metrics-in-portal.png)

对于支持维度的指标，必须使用所需的维度值进行筛选。 以下示例演示如何查看帐户级别的、“成功”响应类型的“事务”。

![在 Azure 门户中访问包含维度的指标的屏幕截图](./media/storage-metrics-in-azure-monitor/access-metrics-in-portal-with-dimension.png)

### <a name="access-metrics-with-the-rest-api"></a>使用 REST API 访问指标

Azure Monitor 提供 [REST API](/rest/api/monitor/) 用于读取指标定义和值。 本部分介绍如何读取存储指标。 所有 REST API 中使用了资源 ID。 有关详细信息，请阅读[了解存储中服务的资源 ID](#understanding-resource-id-for-services-in-storage)。

以下示例演示如何在命令行中使用 [ArmClient](https://github.com/projectkudu/ARMClient)，借助 REST API 来简化测试。

#### <a name="list-account-level-metric-definition-with-the-rest-api"></a>使用 REST API 列出帐户级指标定义

以下示例演示如何列出帐户级别的指标定义：

```
# Login to Azure and enter your credentials when prompted.
> armclient login

> armclient GET /subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/providers/microsoft.insights/metricdefinitions?api-version=2017-05-01-preview

```

如果想要列出 Blob、表、文件或队列的指标定义，必须使用 API 指定每个服务的不同资源 ID。

响应包含 JSON 格式的指标定义：

```Json
{
  "value": [
    {
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/providers/microsoft.insights/metricdefinitions/UsedCapacity",
      "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}",
      "category": "Capacity",
      "name": {
        "value": "UsedCapacity",
        "localizedValue": "Used capacity"
      },
      "isDimensionRequired": false,
      "unit": "Bytes",
      "primaryAggregationType": "Average",
      "metricAvailabilities": [
        {
          "timeGrain": "PT1M",
          "retention": "P30D"
        },
        {
          "timeGrain": "PT1H",
          "retention": "P30D"
        }
      ]
    },
    ... next metric definition
  ]
}

```

#### <a name="read-account-level-metric-values-with-the-rest-api"></a>使用 REST API 读取帐户级指标值

以下示例演示如何读取帐户级别的指标数据：

```
> armclient GET "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/providers/microsoft.insights/metrics?metric=Availability&api-version=2017-05-01-preview&aggregation=Average&interval=PT1H"

```

在上述示例中，如果想要读取 Blob、表、文件或队列的指标值，必须使用 API 指定每个服务的不同资源 ID。

以下响应包含 JSON 格式的指标值：

```Json
{
  "cost": 0,
  "timespan": "2017-09-07T17:27:41Z/2017-09-07T18:27:41Z",
  "interval": "PT1H",
  "value": [
    {
      "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/providers/Microsoft.Insights/metrics/Availability",
      "type": "Microsoft.Insights/metrics",
      "name": {
        "value": "Availability",
        "localizedValue": "Availability"
      },
      "unit": "Percent",
      "timeseries": [
        {
          "metadatavalues": [],
          "data": [
            {
              "timeStamp": "2017-09-07T17:27:00Z",
              "average": 100.0
            }
          ]
        }
      ]
    }
  ]
}

```

## <a name="billing-for-metrics"></a>指标计费

目前，在 Azure Monitor 中可以免费使用指标。 但是，如果使用引入指标数据的其他解决方案，这些解决方案可能收费。 例如，如果将指标数据存档到 Azure 存储帐户，则 Azure 存储会收费。 或者，如果将指标数据流式传输到 OMS 进行高级分析，则 Operation Management Suite (OMS) 会收费。

## <a name="understanding-resource-id-for-services-in-azure-storage"></a>了解 Azure 存储中服务的资源 ID

资源 ID 是资源在 Azure 中的唯一标识符。 使用 Azure Monitor REST API 读取指标定义或值时，必须使用要对其执行操作的资源的资源 ID。 资源 ID 模板遵循以下格式：

`
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/{resourceProviderNamespace}/{resourceType}/{resourceName}
`

存储通过 Azure Monitor 提供存储帐户级别和服务级别的指标。 例如，可以仅检索 Blob 存储的指标。 每个级别有其自身的资源 ID，用于仅检索该级别的指标。

### <a name="resource-id-for-a-storage-account"></a>存储帐户的资源 ID

下面显示了指定存储帐户的资源 ID 时所用的格式。

`
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}
`

### <a name="resource-id-for-the-storage-services"></a>存储服务的资源 ID

下面显示了指定每个存储服务的资源 ID 时所用的格式。

* Blob 服务资源 ID `
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/blobServices/default
`
* 表服务资源 ID `
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/tableServices/default
`
* 队列服务资源 ID `
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/queueServices/default
`
* 文件服务资源 ID `
/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/{storageAccountName}/fileServices/default
`

### <a name="resource-id-in-azure-monitor-rest-api"></a>Azure Monitor REST API 中的资源 ID

下面显示了调用 Azure Monitor REST API 时所用的模式。

`
GET {resourceId}/providers/microsoft.insights/metrics?{parameters}
`

## <a name="capacity-metrics"></a>容量度量值

容量指标每隔一小时发送到 Azure Monitor。 值每日刷新。 时间粒度定义呈现指标值的时间间隔。 所有容量指标的受支持时间粒度为一小时 (PT1H)。

Azure 存储在 Azure Monitor 中提供以下容量指标。

### <a name="account-level"></a>帐户级别

| 指标名称 | 说明 |
| ------------------- | ----------------- |
| UsedCapacity | 存储帐户使用的存储量。 对于标准存储帐户，该指标是 Blob、表、文件和队列使用的容量总和。 对于高级存储帐户和 Blob 存储帐户，这与 BlobCapacity 相同。 <br/><br/> 单元：字节 <br/> 聚合类型：平均 <br/> 值示例：1024 |

### <a name="blob-storage"></a>Blob 存储

| 指标名称 | 说明 |
| ------------------- | ----------------- |
| BlobCapacity | 存储帐户中使用的 Blob 存储总计。 <br/><br/> 单元：字节 <br/> 聚合类型：平均 <br/> 值示例：1024 <br/> 维度：BlobType（[定义](#metrics-dimensions)） |
| BlobCount    | 在存储帐户中存储的 Blob 对象数。 <br/><br/> 单位：计数 <br/> 聚合类型：平均 <br/> 值示例：1024 <br/> 维度：BlobType（[定义](#metrics-dimensions)） |
| ContainerCount    | 存储帐户中的容器数。 <br/><br/> 单位：计数 <br/> 聚合类型：平均 <br/> 值示例：1024 |

### <a name="table-storage"></a>表存储

| 指标名称 | 说明 |
| ------------------- | ----------------- |
| TableCapacity | 存储帐户使用的表存储量。 <br/><br/> 单元：字节 <br/> 聚合类型：平均 <br/> 值示例：1024 |
| TableCount   | 存储帐户中的表数目。 <br/><br/> 单位：计数 <br/> 聚合类型：平均 <br/> 值示例：1024 |
| TableEntityCount | 存储帐户中的表实体数目。 <br/><br/> 单位：计数 <br/> 聚合类型：平均 <br/> 值示例：1024 |

### <a name="queue-storage"></a>队列存储

| 指标名称 | 说明 |
| ------------------- | ----------------- |
| QueueCapacity | 存储帐户使用的队列存储量。 <br/><br/> 单元：字节 <br/> 聚合类型：平均 <br/> 值示例：1024 |
| QueueCount   | 存储帐户中的队列数目。 <br/><br/> 单位：计数 <br/> 聚合类型：平均 <br/> 值示例：1024 |
| QueueMessageCount | 存储帐户中未失效的队列消息数目。 <br/><br/>单位：计数 <br/> 聚合类型：平均 <br/> 值示例：1024 |

### <a name="file-storage"></a>文件存储

| 指标名称 | 说明 |
| ------------------- | ----------------- |
| FileCapacity | 存储帐户使用的文件存储量。 <br/><br/> 单元：字节 <br/> 聚合类型：平均 <br/> 值示例：1024 |
| FileCount   | 存储帐户中的文件数目。 <br/><br/> 单位：计数 <br/> 聚合类型：平均 <br/> 值示例：1024 |
| FileShareCount | 存储帐户中的文件共享数目。 <br/><br/> 单位：计数 <br/> 聚合类型：平均 <br/> 值示例：1024 |

## <a name="transaction-metrics"></a>事务指标

事务指标每隔一分钟从 Azure 存储发送到 Azure Monitor。 所有事务指标在帐户和服务级别（Blob 存储、表存储、Azure 文件和队列存储）提供。 时间粒度定义呈现指标值的时间间隔。 所有事务指标的受支持时间粒度为 PT1H 和 PT1M。

Azure 存储在 Azure Monitor 中提供以下事务指标。

| 指标名称 | 说明 |
| ------------------- | ----------------- |
| 事务 | 向存储服务或指定的 API 操作发出的请求数。 此数字包括成功和失败的请求数，以及引发错误的请求数。 <br/><br/> 单位：计数 <br/> 聚合类型：总计 <br/> 适用的维度：ResponseType、GeoType、ApiName（[定义](#metrics-dimensions)）<br/> 值示例：1024 |
| 流入量 | 流入数据量。 此数字包括从外部客户端到 Azure 存储流入的数据量，以及流入 Azure 中的数据量。 <br/><br/> 单元：字节 <br/> 聚合类型：总计 <br/> 适用的维度：GeoType、ApiName（[定义](#metrics-dimensions)） <br/> 值示例：1024 |
| 流出量 | 流出数据量。 此数字包括从外部客户端到 Azure 存储流出的数据量，以及流出 Azure 中的数据量。 因此，此数字不反映计费的流出量。 <br/><br/> 单元：字节 <br/> 聚合类型：总计 <br/> 适用的维度：GeoType、ApiName（[定义](#metrics-dimensions)） <br/> 值示例：1024 |
| SuccessServerLatency | Azure 存储处理成功请求所用的平均时间。 此值不包括 SuccessE2ELatency 中指定的网络延迟。 <br/><br/> 单位：毫秒 <br/> 聚合类型：平均 <br/> 适用的维度：GeoType、ApiName（[定义](#metrics-dimensions)） <br/> 值示例：1024 |
| SuccessE2ELatency | 向存储服务或指定的 API 操作发出的成功请求的平均端到端延迟。 此值包括在 Azure 存储中读取请求、发送响应和接收响应确认所需的处理时间。 <br/><br/> 单位：毫秒 <br/> 聚合类型：平均 <br/> 适用的维度：GeoType、ApiName（[定义](#metrics-dimensions)） <br/> 值示例：1024 |
| 可用性 | 存储服务或指定的 API 操作的可用性百分比。 可用性通过由“计费请求总数”值除以适用的请求数（包括引发意外错误的请求）计算得出。 所有意外错误都会导致存储服务或指定的 API 操作的可用性下降。 <br/><br/> 单位：百分比 <br/> 聚合类型：平均 <br/> 适用的维度：GeoType、ApiName（[定义](#metrics-dimensions)） <br/> 值示例：99.99 |

## <a name="metrics-dimensions"></a>指标维度

Azure 存储支持对 Azure Monitor 中的指标使用以下维度。

| 维度名称 | 说明 |
| ------------------- | ----------------- |
| /BlobType | 仅限 Blob 指标的 Blob 类型。 支持的值为 **BlockBlob** 和 **PageBlob**。 BlockBlob 中包含追加 Blob。 |
| ResponseType | 事务响应类型。 可用的值包括： <br/><br/> <li>ServerOtherError：除描述的错误以外的其他所有服务器端错误 </li> <li> ServerBusyError：返回了 HTTP 503 状态代码的已经过身份验证的请求。 （尚不支持） </li> <li> ServerTimeoutError：返回了 HTTP 500 状态代码的已超时且经过身份验证的请求。 由于服务器错误而发生超时。 </li> <li> ThrottlingError：客户端和服务器端限制错误的总和（支持 ServerBusyError 和 ClientThrottlingError 后，会删除该维度） </li> <li> AuthorizationError：由于未经授权访问数据或者授权失败，经过身份验证的请求失败。 </li> <li> NetworkError：由于网络错误，经过身份验证的请求失败。 往往发生于客户端在超时失效之前提前关闭了连接时。 </li> <li>  ClientThrottlingError：客户端限制错误（尚不支持） </li> <li> ClientTimeoutError：返回了 HTTP 500 状态代码的已超时且经过身份验证的请求。 如果将客户端的网络超时或请求超时设置为比存储服务预期值更小的值，则预期会发生此超时。 否则，会报告为 ServerTimeoutError。 </li> <li> ClientOtherError：除描述的错误以外的其他所有客户端错误。 </li> <li> Success：成功请求|
| GeoType | 来自主要或辅助群集的事务。 可用值包括 Primary 和 Secondary。 从辅助租户读取对象时，该维度会应用到读取访问异地冗余存储 (RA-GRS)。 |
| ApiName | 操作的名称。 例如： <br/> <li>CreateContainer</li> <li>DeleteBlob</li> <li>GetBlob</li> 有关所有操作名称，请参阅[文档](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages#logged-operations.md)。 |

对于指标支持的维度，需要指定维度值才能查看相应的指标值。 例如，如果查看成功响应的 **Transactions** 值，需要使用 **Success** 筛选 **ResponseType** 维度。 或者，如果查看块 Blob 的 **BlobCount** 值，需要使用 **BlockBlob** 筛选 **BlobType** 维度。

## <a name="service-continuity-of-legacy-metrics"></a>旧指标的服务连续性

旧指标可与 Azure Monitor 托管的指标一同使用。 在 Azure 存储终止旧指标的服务之前，支持范围保持不变。 正式发布 Azure Monitor 托管的指标后，我们会公布终止计划。

## <a name="see-also"></a>另请参阅

* [Azure Monitor](../../monitoring-and-diagnostics/monitoring-overview.md)
