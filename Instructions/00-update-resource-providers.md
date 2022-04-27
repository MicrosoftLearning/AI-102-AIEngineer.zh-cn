---
lab:
  title: 启用资源提供程序
  module: Setup
ms.openlocfilehash: 2cd48d0d9f67b9bab3e64452bf6199e1f438492a
ms.sourcegitcommit: e06fbeaa3bc15e4dbe99f72216dfeeb27f58babd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 02/15/2022
ms.locfileid: "138765490"
---
# <a name="enable-resource-providers"></a>启用资源提供程序

某些资源提供程序必须在 Azure 订阅中注册。 请按照以下步骤操作，确保其已注册。

1. 使用与订阅关联的 Microsoft 凭据在 `https://portal.azure.com` 登录 Azure 门户。
2. 在主页中，选择“订阅”（或展开 &#8801; 菜单，选择“所有服务”，然后在“一般”类别中，选择“订阅”）     。
3. 选择 Azure 订阅（如果有多个订阅，请选择通过兑换 Azure Pass 创建的订阅）。
4. 在订阅边栏选项卡左侧窗格中的“设置”部分，选择“资源提供程序” 。
5. 在资源提供者列表中，确保注册了以下提供程序（如果没有，选择它们并单击“注册”）：
    - Microsoft.BotService
    - Microsoft.Web
    - Microsoft.ManagedIdentity
    - Microsoft.Search
    - Microsoft.Storage
    - Microsoft.CognitiveServices
    - Microsoft.AlertsManagement
    - microsoft.insights
    - Microsoft.KeyVault
    - Microsoft.ContainerInstance
