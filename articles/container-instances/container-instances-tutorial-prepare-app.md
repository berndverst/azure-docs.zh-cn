---
title: "Azure 容器实例教程 - 准备应用 | Azure Docs"
description: "准备部署到 Azure 容器实例的应用"
services: container-instances
documentationcenter: 
author: seanmck
manager: timlt
editor: 
tags: 
keywords: 
ms.assetid: 
ms.service: container-instances
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/01/2017
ms.author: seanmck
ms.custom: mvc
ms.openlocfilehash: ca4cd00b3e9e58fd1137b896e7aac96549bf6d05
ms.sourcegitcommit: 6699c77dcbd5f8a1a2f21fba3d0a0005ac9ed6b7
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 10/11/2017
---
# <a name="create-container-for-deployment-to-azure-container-instances"></a>创建部署到 Azure 容器实例的容器

使用 Azure 容器实例可以将 Docker 容器部署到 Azure 基础结构中，不需预配任何虚拟机，也不需采用任何更高级别的服务。 本教程将使用 Node.js 生成简单的 Web 应用程序并将其打包到容器中，然后即可使用 Azure 容器实例运行该容器。 内容涵盖：

> [!div class="checklist"]
> * 克隆 GitHub 中的应用程序源  
> * 根据应用程序源创建容器映像
> * 在本地 Docker 环境中测试映像

在后续教程中，需要将映像上传到 Azure 容器注册表，然后将其部署到 Azure 容器实例。

## <a name="before-you-begin"></a>开始之前

本教程假定基本了解核心 Docker 的概念，如容器、容器映像和基本的 Docker 命令。 如需要，请参阅 [Docker 入门]( https://docs.docker.com/get-started/)，了解容器基本知识。 

若要完成本教程，需要 Docker 开发环境。 Docker 提供的包可在任何 [Mac](https://docs.docker.com/docker-for-mac/)、[Windows](https://docs.docker.com/docker-for-windows/) 或 [Linux](https://docs.docker.com/engine/installation/#supported-platforms) 系统上轻松配置 Docker。

Azure Cloud Shell 不包含完成本教程每个步骤所需的 Docker 组件。 因此，我们建议使用完整的 Docker 开发环境。

## <a name="get-application-code"></a>获取应用程序代码

本教程中的示例包括使用 [Node.js](http://nodejs.org) 生成的简单 Web 应用程序。 应用提供静态的 HTML 页面，如下所示：

![显示在浏览器中的教程应用][aci-tutorial-app]

使用 git 下载示例：

```bash
git clone https://github.com/Azure-Samples/aci-helloworld.git
```

## <a name="build-the-container-image"></a>生成容器映像

在示例存储库中提供的 Dockerfile 演示如何生成容器。 它从基于 [Alpine Linux](https://alpinelinux.org/) 的[正式 Node.js 映像][dockerhub-nodeimage]开始，该映像是适用于容器的小型分发。 然后，它会将应用程序文件复制到容器中，使用 Node 包管理器安装依赖项，最后启动应用程序。

```
FROM node:8.2.0-alpine
RUN mkdir -p /usr/src/app
COPY ./app/* /usr/src/app/
WORKDIR /usr/src/app
RUN npm install
CMD node /usr/src/app/index.js
```

使用 `docker build` 命令创建容器映像，将其标记为“aci-tutorial-app”：

```bash
docker build ./aci-helloworld -t aci-tutorial-app
```

使用 `docker images` 查看生成的映像：

```bash
docker images
```

输出：

```bash
REPOSITORY                   TAG                 IMAGE ID            CREATED              SIZE
aci-tutorial-app             latest              5c745774dfa9        39 seconds ago       68.1 MB
```

## <a name="run-the-container-locally"></a>在本地运行容器

尝试将容器部署到 Azure 容器实例之前，请先在本地运行，确认其功能正常。 `-d` 开关可以让容器在后台运行，而 `-p` 则可以将计算机上的任意端口映射到容器中的端口 80。

```bash
docker run -d -p 8080:80 aci-tutorial-app
```

打开浏览器并访问 http://localhost:8080 ，确认容器正在运行。

![在浏览器中以本地方式运行应用][aci-tutorial-app-local]

## <a name="next-steps"></a>后续步骤

在本教程中，你创建了一个可以部署到 Azure 容器实例的容器映像。 已完成以下步骤：

> [!div class="checklist"]
> * 克隆 GitHub 中的应用程序源  
> * 根据应用程序源创建容器映像
> * 在本地测试容器

请转到下一教程，了解如何在 Azure 容器注册表中存储容器映像。

> [!div class="nextstepaction"]
> [向 Azure 容器注册表推送映像](./container-instances-tutorial-prepare-acr.md)

<!-- LINKS -->
[dockerhub-nodeimage]: https://hub.docker.com/r/library/node/tags/8.2.0-alpine/

<!--- IMAGES --->
[aci-tutorial-app]:./media/container-instances-quickstart/aci-app-browser.png
[aci-tutorial-app-local]: ./media/container-instances-tutorial-prepare-app/aci-app-browser-local.png