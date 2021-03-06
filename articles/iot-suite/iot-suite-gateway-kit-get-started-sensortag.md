---
title: "使用 Intel NUC 将网关连接到 Azure IoT 套件 | Microsoft Docs"
description: "使用 Microsoft IoT 商业网关工具包和远程监视预配置解决方案。 使用 Azure IoT Edge 网关将 SensorTag 设备连接到远程监视解决方案，将遥测数据发送到云，并响应从解决方案仪表板调用的方法。"
services: 
suite: iot-suite
documentationcenter: 
author: dominicbetts
manager: timlt
editor: 
ms.service: iot-suite
ms.devlang: c
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 07/24/2017
ms.author: dobett
ms.openlocfilehash: bda16be1094276fcecef1e708f9d7db307d94a89
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="connect-your-azure-iot-edge-gateway-to-the-remote-monitoring-preconfigured-solution-and-send-telemetry-from-a-sensortag"></a>将 Azure IoT Edge 网关连接到远程监视预配置解决方案，并从 SensorTag 发送遥测

[!INCLUDE [iot-suite-gateway-kit-selector](../../includes/iot-suite-gateway-kit-selector.md)]

本教程介绍如何使用 Azure IoT Edge 将温度和湿度数据从 SensorTag 设备发送到远程监视预配置解决方案。 SensorTag 使用蓝牙连接到 Intel NUC 网关。 本教程使用：

- 可实现示例网关的 Azure IoT Edge。
- 使用 IoT 套件远程监视预配置解决方案作为基于云的后端。

## <a name="overview"></a>概述

在本教程中，将完成以下步骤：

- 将远程监视预配置解决方案的实例部署到 Azure 订阅。 此步骤会自动部署并配置多个 Azure 服务。
- 将 Intel NUC 网关设备设置为与计算机和远程监视解决方案通信。
- 设置 Intel NUC 网关，以便从 SensorTag 设备接收遥测，然后将其发送到远程监视仪表板。

[!INCLUDE [iot-suite-gateway-kit-prerequisites](../../includes/iot-suite-gateway-kit-prerequisites.md)]

[Texas Instruments BLE SensorTag][lnk-sensortag]。 本教程通过蓝牙从 SensorTag 设备检索遥测数据。

[!INCLUDE [iot-suite-provision-remote-monitoring](../../includes/iot-suite-provision-remote-monitoring.md)]

> [!WARNING]
> 远程监视解决方案在 Azure 订阅中预配一组 Azure 服务。 部署反映实际企业体系结构。 要避免产生不必要的 Azure 使用费用，请在使用完预配置解决方案的实例后，在 azureiotsuite.com 上将其删除。 如果再次需要预配置解决方案，可以轻松地重新创建它。 若要详细了解如何在远程监视解决方案运行时减少消耗，请参阅[出于演示目的配置 Azure IoT 套件预配置解决方案][lnk-demo-config]。

[!INCLUDE [iot-suite-gateway-kit-view-solution](../../includes/iot-suite-gateway-kit-view-solution.md)]

[!INCLUDE [iot-suite-gateway-kit-prepare-nuc-connectivity](../../includes/iot-suite-gateway-kit-prepare-nuc-connectivity.md)]

## <a name="configure-bluetooth-connectivity"></a>配置蓝牙连接

在 Intel NUC 上配置蓝牙，以便 SensorTag 设备连接和发送遥测。

### <a name="find-the-mac-address-of-the-sensortag"></a>找到 SensorTag 的 MAC 地址

1. 在 Intel NUC 的 shell 中运行以下命令，取消阻止蓝牙服务：

    ```bash
    sudo rfkill unblock bluetooth
    ```

1. 运行以下命令，在 Intel NUC 上启动蓝牙服务，并进入蓝牙 shell：

    ```bash
    sudo systemctl start bluetooth
    bluetoothctl
    ```

1. 运行以下命令，启动蓝牙控制器：

    ```bash
    power on
    ```

    控制器打开后，会显示消息“更改启动成功”。

1. 运行以下命令，扫描附近的蓝牙设备：

    ```bash
    scan on
    ```

1. 按 SensorTag 上的启动按钮，使其变为可发现。 绿色 LED 闪烁。

1. 在 shell 中出现一条消息，指出控制器已发现 SensorTag 时，请记下设备的 MAC 地址。 MAC 地址类似于 **A0:E6:F8:B5:F6:00**。 在本教程后面配置网关时，需要该 MAC 地址。

1. 运行以下命令，关闭蓝牙扫描：

    ```bash
    scan off
    ```

1. 运行以下命令，验证能否连接到 SensorTag 设备：

    ```bash
    connect <SensorTag MAC address>
    ```

    如果连接成功，shell 会显示消息“连接成功”，并输出有关 SensorTag 设备的信息。 如果无法连接，请检查 SensorTag 是否仍处于开启状态。

1. 现在可以通过运行以下命令，从 SensorTag 断开连接，并退出蓝牙 shell：

    ```bash
    disconnect
    exit
    ```

[!INCLUDE [iot-suite-gateway-kit-prepare-nuc-software](../../includes/iot-suite-gateway-kit-prepare-nuc-software.md)]

## <a name="build-the-custom-iot-edge-module"></a>生成自定义 IoT Edge 模块

现在可以生成自定义 IoT Edge 模块，以便网关将消息发送到远程监视解决方案。 若要详细了解如何配置网关和 IoT Edge 模块，请参阅 [Azure IoT Edge 概念][lnk-gateway-concepts]。

使用以下命令，从 GitHub 下载自定义 IoT Edge 模块的源代码：

```bash
cd ~
git clone https://github.com/Azure-Samples/iot-remote-monitoring-c-intel-nuc-gateway-getting-started.git
```

使用以下命令，生成自定义 IoT Edge 模块：

```bash
cd ~/iot-remote-monitoring-c-intel-nuc-gateway-getting-started/basic
chmod u+x build.sh
sed -i -e 's/\r$//' build.sh
./build.sh
```

生成脚本将 libsensor2remotemonitoring.so 自定义 IoT Edge 模块置于生成文件夹中。

## <a name="configure-and-run-the-iot-edge-gateway"></a>配置和运行 IoT Edge 网关

现在可以配置 IoT Edge 网关，将遥测从 SensorTag 设备发送到远程监视仪表板。 若要详细了解如何配置网关和 IoT Edge 模块，请参阅 [Azure IoT Edge 概念][lnk-gateway-concepts]。

> [!TIP]
> 在本教程中，在 Intel NUC 上使用标准的 `vi` 文本编辑器。 如果以前没有用过 `vi`，则应完成入门教程（例如 [Unix - vi 编辑器教程][lnk-vi-tutorial]），让自己熟悉该编辑器。 也可使用命令 `smart install nano -y` 安装用户友好性更强的 [nano](https://www.nano-editor.org/) 编辑器。

使用以下命令在 **vi** 编辑器中打开示例配置文件：

```bash
vi ~/iot-remote-monitoring-c-intel-nuc-gateway-getting-started/basic/remote_monitoring.json
```

在 IoTHub 模块的配置中找到以下行：

```json
"args": {
  "IoTHubName": "<<Azure IoT Hub Name>>",
  "IoTHubSuffix": "<<Azure IoT Hub Suffix>>",
  "Transport": "http"
}
```

将占位符值替换为在本教程开始时创建并保存的 IoT 中心信息。 IoTHubName 的值类似于 **yourrmsolution37e08**，IoTSuffix 的值通常为 **azure-devices.net**。

在映射模块的配置中找到以下行：

```json
args": [
  {
    "macAddress": "<<AA:BB:CC:DD:EE:FF>>",
    "deviceId": "<<Azure IoT Hub Device ID>>",
    "deviceKey": "<<Azure IoT Hub Device Key>>"
  }
]
```

将 **macAddress** 占位符替换为此前记下的 SensorTag 的 MAC 地址。 将 **deviceID** 和 **deviceKey** 占位符替换为此前在远程监视解决方案中创建的两个设备的 ID 和密钥。

在 SensorTag 模块的配置中找到以下行：

```json
"args": {
  "controller_index": 0,
  "device_mac_address": "<<AA:BB:CC:DD:EE:FF>>",
  ...
}
```

将 **device\_mac\_address** 占位符替换为此前记下的 SensorTag 的 MAC 地址。

保存所做更改。

现在可使用以下命令运行网关：

```bash
cd ~/iot-remote-monitoring-c-intel-nuc-gateway-getting-started/basic
/usr/share/azureiotgatewaysdk/samples/ble_gateway/ble_gateway remote_monitoring.json
```

IoT Edge 网关在 Intel NUC 上启动，并将遥测从 SensorTag 发送到远程监视解决方案：

![IoT Edge 网关从 SensorTag 发送遥测][img-telemetry]

随时都可按 **Ctrl-C** 退出程序。

## <a name="view-the-telemetry"></a>查看遥测

网关现在正将遥测从 SensorTag 设备发送到远程监视解决方案。 可以在解决方案仪表板上查看遥测。 也可在解决方案仪表板中通过网关将命令发送到 SensorTag 设备。

- 导航到解决方案仪表板。
- 在“要查看的设备”下拉列表中，选择在网关中配置的代表 SensorTag 的设备。
- 来自 SensorTag 设备的遥测显示在仪表板上。

![显示来自 SensorTag 设备的遥测][img-telemetry-display]

> [!WARNING]
> 如果让远程监视解决方案在 Azure 帐户中保持运行状态，系统会按其运行时间计费。 若要详细了解如何在远程监视解决方案运行时减少消耗，请参阅[出于演示目的配置 Azure IoT 套件预配置解决方案][lnk-demo-config]。 请在用完后从 Azure 帐户中删除预配置的解决方案。


## <a name="next-steps"></a>后续步骤

有关 Azure IoT 的更多示例和文档，请访问 [Azure IoT 开发人员中心](https://azure.microsoft.com/develop/iot/)。

[img-telemetry]: ./media/iot-suite-gateway-kit-get-started-sensortag/appoutput.png


[img-telemetry-display]: ./media/iot-suite-gateway-kit-get-started-sensortag/telemetry.png

[lnk-demo-config]: https://github.com/Azure/azure-iot-remote-monitoring/blob/master/Docs/configure-preconfigured-demo.md
[lnk-vi-tutorial]: http://www.tutorialspoint.com/unix/unix-vi-editor.htm
[lnk-sensortag]: http://www.ti.com/ww/en/wireless_connectivity/sensortag/
[lnk-gateway-concepts]: https://docs.microsoft.com/azure/iot-hub/iot-hub-linux-iot-edge-get-started