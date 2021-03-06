---
title: "配置 Elemental Live 编码器以发送单比特率实时流 | Microsoft Docs"
description: "本主题说明如何配置 Elemental Live 编码器，以便将单比特率流发送到用于实时编码的 AMS 频道。"
services: media-services
documentationcenter: 
author: cenkdin
manager: cfowler
editor: 
ms.assetid: 9c6bf6a9-6273-4fdd-9477-f0e565280b5b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: ne
ms.topic: article
ms.date: 01/05/2017
ms.author: cenkd;anilmur;juliako
ms.openlocfilehash: 668a3ab46a70c0ee25fa87031d27c0f4333ec89c
ms.sourcegitcommit: 18ad9bc049589c8e44ed277f8f43dcaa483f3339
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 08/29/2017
---
# <a name="use-the-elemental-live-encoder-to-send-a-single-bitrate-live-stream"></a>使用 Elemental Live 编码器发送单比特率实时流
> [!div class="op_single_selector"]
> * [Elemental Live](media-services-configure-elemental-live-encoder.md)
> * [Tricaster](media-services-configure-tricaster-live-encoder.md)
> * [Wirecast](media-services-configure-wirecast-live-encoder.md)
> * [FMLE](media-services-configure-fmle-live-encoder.md)
>
>

本主题说明如何配置 [Elemental Live](http://www.elementaltechnologies.com/products/elemental-live) 编码器，以便将单比特率流发送到用于实时编码的 AMS 频道。  有关详细信息，请参阅 [使用能够通过 Azure 媒体服务执行实时编码的频道](media-services-manage-live-encoder-enabled-channels.md)。

本教程演示了如何通过 Azure 媒体服务浏览器 (AMSE) 工具管理 Azure 媒体服务 (AMS)。 此工具仅在 Windows 电脑上运行。 如果使用的是 Mac 或 Linux，则可使用 Azure 门户创建[频道](media-services-portal-creating-live-encoder-enabled-channel.md#create-a-channel)和[节目](media-services-portal-creating-live-encoder-enabled-channel.md)。

## <a name="prerequisites"></a>先决条件
* 必须具有实践知识，了解如何使用 Elemental Live Web 界面来创建实时事件。
* [创建 Azure 媒体服务帐户](media-services-portal-create-account.md)
* 确保流式处理终结点正在运行。 有关详细信息，请参阅[在媒体服务帐户中管理流式处理终结点](media-services-portal-manage-streaming-endpoints.md)。
* 安装最新版本的 [AMSE](https://github.com/Azure/Azure-Media-Services-Explorer) 工具。
* 启动该工具并连接到 AMS 帐户。

## <a name="tips"></a>提示
* 尽可能使用硬编码的 Internet 连接。
* 在确定带宽要求时，可以认为它就是将流式处理比特率翻倍。 虽然此要求不是强制性要求，但它可以减轻网络拥塞的影响。
* 使用基于软件的编码器时，请关闭任何不需要的程序。

## <a name="elemental-live-with-rtp-ingest"></a>带 RTP 引入的 Elemental Live
本部分演示如何配置 Elemental Live 编码器，以便通过 RTP 发送单比特率实时流。  有关详细信息，请参阅[基于 RTP 的 MPEG-TS 流](media-services-manage-live-encoder-enabled-channels.md#channel)。

### <a name="create-a-channel"></a>创建频道

1. 在 AMSE 工具中，导航到“实时”选项卡，并右键单击频道区域。 从菜单中选择“创建频道…” 从菜单中。

    ![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental1.png)

2. 指定频道名称，说明字段为可选字段。 在“频道设置”下针对“实时编码”选项选择“标准”，将“输入协议”设置为“RTP (MPEG-TS)”。 所有其他设置可保留原样。

    确保选中“立即启动新频道”。

3. 单击“创建频道”。

   ![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental12.png)

> [!NOTE]
> 启动频道可能需要长达 20 分钟的时间。
>
>

启动频道时，可以[配置编码器](media-services-configure-elemental-live-encoder.md#configure_elemental_rtp)。

> [!IMPORTANT]
> 请注意，只要频道进入就绪状态，就会开始计费。 有关详细信息，请参阅[频道的状态](media-services-manage-live-encoder-enabled-channels.md#states)。
>
>

### <a id=configure_elemental_rtp></a>配置 Elemental Live 编码器
本教程中使用以下输出设置。 本部分的其余内容介绍更详细的配置步骤。

**视频**：

* 编解码器：H.264
* 配置文件：高（等级 4.0）
* 比特率：5000 kbps
* 关键帧：2 秒（60 秒）
* 帧速率：30

**音频**：

* 编码解码器：AAC (LC)
* 比特率：192 kbps
* 采样速率：44.1 kHz

#### <a name="configuration-steps"></a>配置步骤
1. 导航到 **Elemental Live** Web 界面，针对 **UDP/TS** 流式处理设置编码器。
2. 一旦创建新的事件，即可向下滚动到输出组并添加 **UDP/TS** 输出组。
3. 创建新的输出时，可选择“新建流”，然后单击“添加输出”。  

    ![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental13.png)

   > [!NOTE]
   > 建议将 Elemental 事件的时间代码设置为“系统时钟”，方便编码器在出现流故障时重新进行连接。
   >
   >
4. 由于已创建输出，因此此时可单击“添加流”。 现在可以配置输出设置。
5. 向下滚动到刚创建的“流 1”，单击左侧的“视频”选项卡，展开“高级”设置部分。

    ![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental4.png)

    虽然 Elemental Live 的可用自定义设置有很多，但在一开始向 AMS 进行流式传输时，建议使用以下设置。

   * 分辨率：1280 x 720
   * 帧速率：30
   * GOP 大小：60 帧
   * 隔行扫描模式：渐进式
   * 比特率：5000000 位/秒（可根据网络限制进行调整）

    ![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental5.png)

1. 获取频道的输入 URL。

    导航回 AMSE 工具，查看频道完成状态。 一旦状态从“正在启动”变为“正在运行”，即可获取输入 URL。

    频道正在运行时，右键单击频道名称，向下导航，将鼠标悬停在“将输入 URL 复制到剪贴板”上方，然后选择“主要输入 URL”。  

    ![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental6.png)
2. 将此信息粘贴到 Elemental 的“主目标”字段中。 所有其他设置可以保留默认值。

    ![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental14.png)

    为了实现额外的冗余，可对“辅助输入 URL”重复这些步骤，为 UDP/TS 流式处理创建一个单独的“输出”选项卡。
3. 单击“创建”（如果已创建新事件）或“更新”（如果要编辑现有事件），并开始启动编码器。

> [!IMPORTANT]
> 在 Elemental Live Web 界面上单击“启动”之前，**必须**确保频道已就绪。
> 另外，请确保不要让频道在没有一个事件的情况下处于就绪状态的时间超出 15 分钟。
>
>

在流运行 30 秒以后，导航回 AMSE 工具并测试播放情况。  

### <a name="test-playback"></a>测试播放

导航回 AMSE 工具，并右键单击要测试的频道。 在菜单中，将鼠标悬停在“播放预览”上方，然后选择“使用 Azure 媒体播放器”。  

    ![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental8.png)

如果流出现在播放器中，则编码器已正确配置，可以连接到 AMS。

如果收到错误，则需重置频道并调整编码器设置。 请参阅[故障排除](media-services-troubleshooting-live-streaming.md)主题以获取相关指导。   

### <a name="create-a-program"></a>创建节目
1. 一旦确认频道可以播放，则可创建节目。 在 AMSE 工具的“实时”选项卡下，右键单击节目区域，并选择“创建新节目”。  

    ![Elemental](./media/media-services-elemental-live-encoder/media-services-elemental9.png)
2. 为节目命名，并根据需要调整“存档时段长度”（默认为 4 小时）。 还可以指定存储位置，也可以将其保留为默认值。  
3. 选中“立即启动节目”框。
4. 单击“创建节目”。  

    >[!NOTE]
    > 创建节目需要的时间比创建频道需要的时间少。   
      
5. 可以运行节目以后，可通过下述方式来确认其是否能够播放：右键单击该节目，导航到“播放节目”，并选择“使用 Azure Media Player”。  
6. 确认以后，再次右键单击该节目，然后选择“将输出 URL 复制到剪贴板”（或通过菜单从“节目信息和设置”选项检索此信息）。

现在可以将流嵌入到播放器中，也可将其分发给受众进行实时观看。  

## <a name="troubleshooting"></a>故障排除
请参阅[故障排除](media-services-troubleshooting-live-streaming.md)主题以获取相关指导。

## <a name="media-services-learning-paths"></a>媒体服务学习路径
[!INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供反馈
[!INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]
