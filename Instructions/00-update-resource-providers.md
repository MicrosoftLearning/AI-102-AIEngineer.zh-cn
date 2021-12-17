---
lab:
    title: '启用资源提供程序'
    module: '设置'
---

# 启用资源提供程序

某些资源提供程序必须在 Azure 订阅中注册。请按照以下步骤操作，确保其已注册。

1. 使用与你的 Azure 订阅关联的 Microsoft 凭据登录到 Azure 门户 (`https://portal.azure.com`)。
2. 在主页中，选择 **“订阅”** （或展开 **&#8801;** 菜单，选择 **“所有服务”**，然后在 **“所有”** 类别中，选择 **“订阅”**）。
3. 选择 Azure 订阅（如果有多个订阅，请选择通过兑换 Azure Pass 创建的订阅）。
4. 在订阅边栏选项卡左侧窗格中的 **“设置”** 部分，选择 **“资源提供程序”**。
5. 在资源提供程序列表中，确保注册了以下提供程序（如果未注册，请将其选中并单击 **“注册”**）：
    - Microsoft.BotService
    - Microsoft.Web
    - Microsoft.ManagedIdentity
    - Microsoft.Search
    - Microsoft.Storage
    - Microsoft.CognitiveServices
    - Microsoft.AlertsManagement
    - Microsoft.Insights
    - Microsoft.KeyVault
    - Microsoft.ContainerInstance
