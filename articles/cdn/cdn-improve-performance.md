---
title: "通过在 Azure CDN 中压缩文件来提高性能 | Microsoft Docs"
description: "了解如何通过在 Azure CDN 中压缩文件来提高文件传输速度和页面加载性能。"
services: cdn
documentationcenter: 
author: zhangmanling
manager: erikre
editor: 
ms.assetid: af1cddff-78d8-476b-a9d0-8c2164e4de5d
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
ms.openlocfilehash: 86f1f8f0aaa9a699d2f816781cef65cda868ee55
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="improve-performance-by-compressing-files-in-azure-cdn"></a>通过在 Azure CDN 中压缩文件来提高性能
压缩是提高文件传输速度和增加页面负载性能的一种简单有效的方法，可通过在从服务器发送文件之前减少其大小来实现。 这样可以减少带宽成本，并为用户提供更快的响应体验。

有两种方法可启用压缩：

* 在源服务器上启用压缩。 在本例中，CDN 传递压缩文件，将其交付给发出请求的客户端。
* 直接在 CDN 边缘服务器上启用压缩。 这种情况下 CDN 会压缩文件并将其提供给最终用户，即使源服务器未压缩文件也是如此。

> [!IMPORTANT]
> CDN 配置变更可能需要一些时间才能传播到整个网络。  对于<b>来自 Akamai 的 Azure CDN</b> 配置文件，传播通常在一分钟内完成。  对于<b>来自 Verizon 的 Azure CDN</b> 配置文件，通常会在 90 分钟内应用更改。  如果首次为 CDN 终结点设置压缩，请考虑等待 1-2 个小时，确保将压缩设置传播到 POP 之后再排查问题。
> 
> 

## <a name="enabling-compression"></a>启用压缩
> [!NOTE]
> 标准和高级版 CDN 层级提供相同的压缩功能，但用户界面不同。  有关标准和高级版 CDN 层级之间的差异的详细信息，请参阅 [Azure CDN 概述](cdn-overview.md)。
> 
> 

### <a name="standard-tier"></a>标准层
> [!NOTE]
> 本部分适用于**来自 Verizon 的标准 Azure CDN** 和**来自 Akamai 的标准 Azure CDN** 配置文件。
> 
> 

1. 从 CDN 配置文件页，单击想要管理的 CDN 终结点。
   
    ![CDN 配置文件终结点](./media/cdn-file-compression/cdn-endpoints.png)
   
    CDN 终结点页面会打开。
2. 单击“**配置**”按钮。
   
    ![CDN 配置文件管理按钮](./media/cdn-file-compression/cdn-config-btn.png)
   
    “CDN 配置”页面会打开。
3. 开启**压缩**。
   
    ![CDN 压缩选项](./media/cdn-file-compression/cdn-compress-standard.png)
4. 请使用默认类型，或通过删除或添加文件类型来修改列表。
   
   > [!TIP]
   > 建议不要对压缩格式应用压缩（虽然可以执行此操作）。 压缩格式示例有 ZIP、MP3、MP4 或 JPG。
   > 
 
5. 完成更改后，单击“**保存**”按钮。

### <a name="premium-tier"></a>高级层
> [!NOTE]
> 本部分适用于**来自 Verizon 的高级 Azure CDN** 配置文件。
> 
> 

1. 从 CDN 配置文件页面，单击“管理”按钮。
   
    ![CDN 配置文件管理按钮](./media/cdn-file-compression/cdn-manage-btn.png)
   
    CDN 管理门户打开。
2. 将鼠标悬停在“**HTTP 大**”选项卡上，然后悬停在“**缓存设置**”浮出控件。  单击“**压缩**”。

    ![文件压缩选择](./media/cdn-file-compression/cdn-compress-select.png)
   
    压缩选项将随即显示。
   
    ![文件压缩选项](./media/cdn-file-compression/cdn-compress-files.png)
3. 通过单击“**启用压缩**”单选按钮启用压缩。  在“**文件类型**”文本框中，输入你想要以逗号分隔的列表（无空格）压缩的 MIME 类型。
   
   > [!TIP]
   > 建议不要对压缩格式应用压缩（虽然可以执行此操作）。 压缩格式示例有 ZIP、MP3、MP4 或 JPG。
   > 
4. 完成更改后，单击“**更新**”按钮。

## <a name="compression-rules"></a>压缩规则
这些表描述在每种方案中 Azure CDN 压缩行为。

> [!IMPORTANT]
> 对于**来自 Verizon 的 Azure CDN** 配置文件（标准和高级），只有符合条件的文件才会被压缩。  要符合压缩条件，文件必须：
> 
> * 大于 128 个字节。
> * 小于 1 MB。
> 
> 这些配置文件支持 gzip (GNU zip)、deflate、bzip2 或 br (Brotli) 编码。 对于 Brotli 编码，压缩仅在边缘完成。 客户端/浏览器必须发送 Brotli 编码请求，压缩资产必须首先已在原始端压缩。 

> [!IMPORTANT]
> 对于**来自 Akamai 的 Azure CDN** 配置文件，所有文件都适合压缩。 但文件必须为已[配置为压缩](#enabling-compression)的 MIME 类型。
> 
>这些配置文件仅支持 gzip 编码。 配置文件终结点请求 gzip 编码的文件时，始终是从源请求，而不考虑客户端请求。 

### <a name="compression-disabled-or-file-is-ineligible-for-compression"></a>压缩被禁用或文件不适合压缩
| 客户端请求的格式（通过 Accept-Encoding 标头） | 缓存的文件格式 | CDN 对客户端的响应 | 说明 |
| --- | --- | --- | --- |
| 压缩 |压缩 |压缩 | |
| 压缩 |未压缩 |未压缩 | |
| 压缩 |未缓存 |压缩或未压缩 |取决于源响应 |
| 未压缩 |压缩 |未压缩 | |
| 未压缩 |未压缩 |未压缩 | |
| 未压缩 |未缓存 |未压缩 | |

### <a name="compression-enabled-and-file-is-eligible-for-compression"></a>已启用压缩且文件适合压缩
| 客户端请求的格式（通过 Accept-Encoding 标头） | 缓存的文件格式 | CDN 对客户端的响应 | 说明 |
| --- | --- | --- | --- |
| 压缩 |压缩 |压缩 |支持的格式之间的 CDN 转码 |
| 压缩 |未压缩 |压缩 |CDN 执行压缩 |
| 压缩 |未缓存 |压缩 |如果源返回未压缩文件，CDN 将执行压缩。  来自 Verizon 的 Azure CDN 将传递第一个请求上的未压缩文件，然后压缩并缓存文件以供后续请求使用。  永远不会压缩带 `Cache-Control: no-cache` 标头的文件。 |
| 未压缩 |压缩 |未压缩 |CDN 执行解压缩 |
| 未压缩 |未压缩 |未压缩 | |
| 未压缩 |未缓存 |未压缩 | |

## <a name="media-services-cdn-compression"></a>媒体服务 CDN 压缩
对于启用了媒体服务 CDN 的流式处理终结点，默认为以下内容类型启用压缩： 
- application/vnd.ms-sstr+xml 
- application/dash+xml
- application/vnd.apple.mpegurl
- application/f4m+xml. 

不能使用 Azure 门户对上述类型启用或禁用压缩。  

## <a name="see-also"></a>另请参阅
* [排查 CDN 文件压缩问题](cdn-troubleshoot-compression.md)    

