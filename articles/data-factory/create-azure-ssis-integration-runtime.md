---
title: "在 Azure 数据工厂中创建自承载的集成运行时 | Microsoft Docs"
description: "了解如何使用 SQL Server 存储过程活动从数据工厂管道调用 Azure SQL 数据库或 Azure SQL 数据仓库中的存储过程。"
services: data-factory
documentationcenter: 
author: spelluru
manager: jhubbard
editor: monicar
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 08/10/2017
ms.author: spelluru
ms.translationtype: HT
ms.sourcegitcommit: a6bba6b3b924564fe7ae16fa1265dd4d93bd6b94
ms.openlocfilehash: b9ab20b6edbc0ab913690a5fcac181cd2d66e3b3
ms.contentlocale: zh-cn
ms.lasthandoff: 09/28/2017

---

# <a name="create-an-azure-ssis-integration-runtime-in-azure-data-factory"></a>在 Azure 数据工厂中创建 Azure-SSIS 集成运行时
本文提供在 Azure 数据工厂中配置 Azure-SSIS 集成运行时所要执行的步骤。 然后，可以使用 SQL Server Data Tools (SSDT) 或 SQL Server Management Studio (SSMS) 将 SQL Server Integration Services (SSIS) 包部署到 Azure 上的此运行时。

教程[教程：将 SQL Server Integration Services 包 (SSIS) 部署到 Azure](tutorial-deploy-ssis-packages-azure.md) 介绍了如何使用 Azure SQL 数据库作为 SSIS 目录的存储来创建 Azure-SSIS 集成运行时 (IR)。 本文对该教程的内容做了延伸，介绍了如何执行以下操作： 

- 使用 Azure SQL 托管实例（个人预览版）来承载 SSIS 目录（SSISDB 数据库）。
- 将 Azure-SSIS IR 加入 Azure 虚拟网络 (VNet)。 

有关将 Azure-SSIS IR 加入 VNet 以及在 Azure 门户中配置 VNet 的概念性信息，请参阅[将 Azure-SSIS IR 加入 VNet](join-azure-ssis-integration-runtime-virtual-network.md)。 

## <a name="prerequisites"></a>先决条件

- **Azure 订阅**。 如果没有订阅，可以创建一个[免费试用](http://azure.microsoft.com/pricing/free-trial/)帐户。
- **Azure SQL 数据库服务器**或 **SQL Server 托管实例（专用预览）（扩展专用预览）**。 如果还没有数据库服务器，请在启动之前在 Azure 门户中创建一个。 此服务器承载 SSIS 目录数据库 (SSISDB)。 建议在集成运行时所在的同一 Azure 区域中创建数据库服务器。 此配置允许集成运行时将执行日志写入 SSISDB 而无需跨 Azure 区域。 
- **经典虚拟网络 (VNet)（可选）**。 如果至少满足以下条件之一，则必须具有 Azure 虚拟网络 (VNet)：
    - 作为 VNet 一部分的 SQL Server 托管实例（专用预览）上承载着 SSIS 目录数据库。
    - 希望从在 Azure-SSIS 集成运行时上运行的 SSIS 包连接到本地数据存储。
- **Azure PowerShell**。 遵循[如何安装和配置 Azure PowerShell](/powershell/azure/install-azurerm-ps) 中的说明。 使用 PowerShell 运行脚本来预配在云中运行 SSIS 包的 Azure-SSIS 集成运行时。 

## <a name="create-variables"></a>创建变量
定义在本教程的脚本中使用的变量：

```powershell
# Azure Data Factory version 2 information
$SubscriptionName = "[your Azure subscription name]"
$ResourceGroupName = "[your Azure resource group name]"
$DataFactoryName = "[your data factory name]"
$DataFactoryLocation = "EastUS" # In public preview, only EastUS|EastUS2 are supported.

# Azure-SSIS integration runtime information - This is the Data Factory compute resource for running SSIS packages
$AzureSSISName = "[your Azure-SSIS integration runtime name]"
$AzureSSISDescription = "This is my Azure-SSIS integration runtime"
$AzureSSISLocation = "EastUS" # In public preview, only EastUS|NorthEurope are supported.
$AzureSSISNodeSize = "Standard_A4_v2" # In public preview, only Standard_A4_v2|Standard_A8_v2|Standard_D1_v2|Standard_D2_v2|Standard_D3_v2|Standard_D4_v2 are supported.
$AzureSSISNodeNumber = 2 # In public preview, only 1-10 nodes are supported.
$AzureSSISMaxParallelExecutionsPerNode = 2 # In public preview, only 1-8 parallel executions per node are supported.

# SSISDB info
$SSISDBServerEndpoint = "[your Azure SQL Database server name.database.windows.net or your Azure SQL Managed Instance (private preview) server endpoint]"
$SSISDBServerAdminUserName = "[your server admin username]"
$SSISDBServerAdminPassword = "[your server admin password]"

# Remove the SSISDBPricingTier variable if you are using Azure SQL Managed Instance (private preview)
# This parameter applies only to Azure SQL Database. 
$SSISDBPricingTier = "[your Azure SQL Database pricing tier, e.g. S3. Remove the variable for Azure SQL Managed Instance (private preview)]"

## Remove these two variables if you are using Azure SQL Database. 
## These two parameters apply if you are using VNet and Azure SQL Managed Instance (private preview). 
$VnetId = "[your VNet resource ID or leave it empty]" # OPTIONAL: In public preview, only classic virtual network (VNet) is supported.
$SubnetName = "[your subnet name or leave it empty]" # OPTIONAL: In public preview, only classic VNet is supported.

```

## <a name="validate-the-connection-to-database"></a>验证与数据库的连接
添加以下脚本用于验证 Azure SQL 数据库服务器 server.database.windows.net 或 Azure SQL 托管实例（个人预览版）服务器终结点。 

```powershell
$SSISDBConnectionString = "Data Source=" + $SSISDBServerEndpoint + ";User ID="+ $SSISDBServerAdminUserName +";Password="+ $SSISDBServerAdminPassword
$sqlConnection = New-Object System.Data.SqlClient.SqlConnection $SSISDBConnectionString;
Try
{
    $sqlConnection.Open();
}
Catch [System.Data.SqlClient.SqlException]
{
    Write-Warning "Cannot connect to your Azure SQL DB logical server/Azure SQL MI server, exception: $_"  ;
    Write-Warning "Please make sure the server you specified has already been created. Do you want to proceed? [Y/N]"
    $yn = Read-Host
    if(!($yn -ieq "Y"))
    {
        Return;
    } 
}
```

## <a name="log-in-and-select-subscription"></a>登录并选择订阅
在脚本中添加以下代码，用于登录并选择 Azure 订阅： 

```powershell
Login-AzureRmAccount
Select-AzureRmSubscription -SubscriptionName $SubscriptionName
```

## <a name="configure-virtual-network"></a>配置虚拟网络
添加以下脚本以自动配置加入 Azure-SSIS 集成运行时的虚拟网络权限/设置。

```powershell
# Register to Azure Batch resource provider
if(![string]::IsNullOrEmpty($VnetId) -and ![string]::IsNullOrEmpty($SubnetName))
{
    $BatchObjectId = (Get-AzureRmADServicePrincipal -ServicePrincipalName "MicrosoftAzureBatch").Id
    Register-AzureRmResourceProvider -ProviderNamespace Microsoft.Batch
    while(!(Get-AzureRmResourceProvider -ProviderNamespace "Microsoft.Batch").RegistrationState.Contains("Registered"))
    {
    Start-Sleep -s 10
    }
    # Assign VM contributor role to Microsoft.Batch
    New-AzureRmRoleAssignment -ObjectId $BatchObjectId -RoleDefinitionName "Classic Virtual Machine Contributor" -Scope $VnetId
}
```

## <a name="create-a-resource-group"></a>创建资源组
使用 [New-AzureRmResourceGroup](/powershell/module/azurerm.resources/new-azurermresourcegroup) 命令创建 [Azure 资源组](../azure-resource-manager/resource-group-overview.md)。 资源组是在其中以组的形式部署和管理 Azure 资源的逻辑容器。 以下示例在 `westeurope` 位置创建名为 `myResourceGroup` 的资源组。

```powershell
New-AzureRmResourceGroup -Location $DataFactoryLocation -Name $ResourceGroupName
```

## <a name="create-a-data-factory"></a>创建数据工厂
运行以下命令以创建数据工厂：

```powershell
Set-AzureRmDataFactoryV2 -ResourceGroupName $ResourceGroupName `
                         -Location $DataFactoryLocation `
                         -Name $DataFactoryName
```

## <a name="create-an-integration-runtime"></a>创建集成运行时
运行以下命令以创建在 Azure 中运行 SSIS 包的 Azure SSIS 集成运行时： 

如果使用 **Azure SQL 数据库**承载 SSISDB 数据库（SSIS 目录）： 

```powershell
$secpasswd = ConvertTo-SecureString $SSISDBServerAdminPassword -AsPlainText -Force
$serverCreds = New-Object System.Management.Automation.PSCredential($SSISDBServerAdminUserName, $secpasswd)
Set-AzureRmDataFactoryV2IntegrationRuntime  -ResourceGroupName $ResourceGroupName `
                                            -DataFactoryName $DataFactoryName `
                                            -Name $AzureSSISName `
                                            -Type Managed `
                                            -CatalogServerEndpoint $SSISDBServerEndpoint `
                                            -CatalogAdminCredential $serverCreds `
                                            -CatalogPricingTier $SSISDBPricingTier `
                                            -Description $AzureSSISDescription `
                                            -Location $AzureSSISLocation `
                                            -NodeSize $AzureSSISNodeSize `
                                            -NodeCount $AzureSSISNodeNumber `
                                            -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode
```

不需要传递 VNetId 和 Subnet 的值，除非需要访问本地数据（即，SSIS 包中包含本地数据源/目标）。 必须传递 CatalogPricingTier 参数的值。 

如果使用 **Azure SQL 托管实例（个人预览版）**承载 SSISDB 数据库：

```powershell
$secpasswd = ConvertTo-SecureString $SSISDBServerAdminPassword -AsPlainText -Force
$serverCreds = New-Object System.Management.Automation.PSCredential($SSISDBServerAdminUserName, $secpasswd)
Set-AzureRmDataFactoryV2IntegrationRuntime  -ResourceGroupName $ResourceGroupName `
                                            -DataFactoryName $DataFactoryName `
                                            -Name $AzureSSISName `
                                            -Type Managed `
                                            -CatalogServerEndpoint $SSISDBServerEndpoint `
                                            -CatalogAdminCredential $serverCreds `
                                            -Description $AzureSSISDescription `
                                            -Location $AzureSSISLocation `
                                            -NodeSize $AzureSSISNodeSize `
                                            -NodeCount $AzureSSISNodeNumber `
                                            -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
                                            -VnetId $VnetId `
                                            -Subnet $SubnetName
```

需要在加入 VNet 的 Azure SQL 托管实例（个人预览版）中传递 VnetId 和 Subnet 参数的值。 CatalogPricingTier 参数不适用于 Azure SQL 托管实例。 

## <a name="start-integration-runtime"></a>启动集成运行时
运行以下命令以启动 Azure-SSIS 集成运行时： 

```powershell
write-host("##### Starting #####")
Start-AzureRmDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                             -DataFactoryName $DataFactoryName `
                                             -Name $AzureSSISName `
                                             -Force

write-host("##### Completed #####")
write-host("If any cmdlet is unsuccessful, please consider using -Debug option for diagnostics.")                                  
```
此命令将花费 **20 到 30 分钟**才能完成。 

## <a name="deploy-ssis-packages"></a>部署 SSIS 包
现在，使用 SQL Server Data Tools (SSDT) 或 SQL Server Management Studio (SSMS) 来将 SSIS 包部署到 Azure。 连接到承载 SSIS 目录 (SSISDB) 的 Azure SQL 服务器。 Azure SQL 服务器的名称采用以下格式：&lt;servername&gt;.database.windows.net（适用于 Azure SQL 数据库）。 有关说明，请参阅文章[部署包](/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages#deploy-packages-to-integration-services-server)。 

## <a name="next-steps"></a>后续步骤
请参阅本文档中的其他 Azure-SSIS IR 主题：

- [Azure-SSIS 集成运行时](concepts-integration-runtime.md#azure-ssis-integration-runtime)。 此文提供有关集成运行时（包括 Azure-SSIS IR）的一般概念性信息。 
- [教程：将 SSIS 包部署到 Azure](tutorial-deploy-ssis-packages-azure.md)。 此文提供有关创建 Azure-SSIS IR，并使用 Azure SQL 数据库来承载 SSIS 目录的分步说明。 
- [监视 Azure-SSIS IR](monitor-integration-runtime.md#azure-ssis-integration-runtime)。 此文介绍如何检索有关 Azure-SSIS IR 的信息，以及返回的信息中的状态说明。 
- [管理 Azure-SSIS IR](manage-azure-ssis-integration-runtime.md)。 此文介绍如何停止、启动或删除 Azure-SSIS IR。 此外，介绍如何通过在 Azure-SSIS IR 中添加更多节点来扩展 IR。 
- [将 Azure-SSIS IR 加入 VNet](join-azure-ssis-integration-runtime-virtual-network.md)。 此文提供有关将 Azure-SSIS IR 加入 Azure 虚拟网络 (VNet) 的概念性信息。 此外，介绍可以执行哪些步骤来使用 Azure 门户配置 VNet，以便 Azure-SSIS IR 能够加入 VNet。 
