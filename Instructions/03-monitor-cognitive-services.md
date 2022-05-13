---
lab:
  title: 监视认知服务
  module: Module 2 - Developing AI Apps with Cognitive Services
ms.openlocfilehash: e0e0042421a4f7150fc3b95cef80887c81a78f3a
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2022
ms.locfileid: "137819518"
---
# <a name="monitor-cognitive-services"></a>监视认知服务

Azure 认知服务是整个应用程序基础结构的关键部分。 能够监视活动，并对可能需要注意的问题发出警报，这一点很重要。

## <a name="clone-the-repository-for-this-course"></a>克隆本课程的存储库

如果已将 AI-102-AIEngineer 代码存储库克隆到了要完成本实验室的环境，请在 Visual Studio Code 中将其打开；否则，请按照以下步骤立即将其克隆。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“Git: 克隆”命令，以将 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择“以后再说”。

## <a name="provision-a-cognitive-services-resource"></a>预配认知服务资源

如果订阅中还没有认知服务资源，则需要预配认知服务资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择“&#65291;创建资源”按钮，搜索“认知服务”，并使用以下设置创建一个认知服务资源：
    - **订阅**：Azure 订阅
    - 资源组：选择或创建一个资源组（如果使用受限制的订阅，你可能无权创建新的资源组 - 请使用提供的资源组）
    - **区域**：选择任何可用区域
    - **名称**：输入唯一名称
    - **定价层**：标准 S0
3. 选中任何所需复选框并创建资源。
4. 等待部署完成，然后查看部署详细信息。
5. 部署资源后，转到该资源并查看其“密钥和终结点”页面。 记下终结点 URI，稍后你会用到它。

## <a name="configure-an-alert"></a>配置警报

通过定义预警规则开始监视，以便你可以检测认知服务资源中的活动。

1. 在 Azure 门户中，前往认知服务资源并查看其“警报”页面（在“监视”部分中） 。
2. 选择“+ 新建警报规则”
3. 在“创建预警规则”页面的“范围”下，验证是否列出了认知服务资源 。
4. 在“条件”下，单击“添加条件”，然后查看右侧显示的“选择信号”窗格，你可以在其中选择要监视的信号类型  。
5. 在“信号类型”列表中，选择“活动日志”，然后在筛选的列表中，选择“列表密钥”  。
6. 查看过去 6 小时的活动。
7. 选择“操作”选项卡。请注意，可以指定一个操作组。 这样，你可以配置触发警报时的自动化操作，例如发送电子邮件通知。 在本练习中，我们不会执行此操作；但在生产环境中，执行此操作可能会很有用。
8. 在“详细信息”选项卡中，将“预警规则名称”设置为“密钥列表警报”  。
9. 选择“查看 + 创建”。 
10. 查看警报的配置。 选择“创建”，等待预警规则创建完成。
11. 在 Visual Studio Code 中，右键单击 03-monitor 文件夹，并打开集成终端。 然后输入以下命令，以使用 Azure CLI 登录到 Azure 订阅。

    ```
    az login
    ```

    Web 浏览器标签页随即打开，并提示你登录到 Azure。 按照提示操作，然后关闭浏览器标签页并返回到 Visual Studio Code。

    > **提示**：如果你有多个订阅，需要确保使用包含认知服务资源的订阅。  使用此命令来确定当前订阅。
    >
    > ```
    > az account show
    > ```
    >
    > 如需更改订阅，请运行以下命令，并将 &lt;subscriptionName&gt; 更改为正确的订阅名称。
    >
    > ```
    > az account set --subscription <subscriptionName>
    > ```

10. 现在，可以使用以下命令来获取认知服务密钥的列表，将 &lt;resourceName&gt; 替换为认知服务资源的名称，并将 &lt;resourceGroup&gt; 替换为在其中创建资源的资源组的名称 。

    ```
    az cognitiveservices account keys list --name <resourceName> --resource-group <resourceGroup>
    ```

该命令返回认知服务资源的密钥列表。

11. 切换回包含 Azure 门户的浏览器，并刷新“警报”页面。 表中应会出现一个 Sev 4 警报（如果没有，请等待五分钟，然后再次刷新）。
12. 选择警报以查看其详细信息。

## <a name="visualize-a-metric"></a>可视化指标

除了定义警报之外，还可以查看认知服务资源的指标以监视其利用率。

1. 在 Azure 门户的认知服务资源页面中，选择“指标”（在“监视”部分中） 。
2. 如果没有现有图表，请选择“+ 新建图表”。 然后在“指标”列表中，查看可以可视化的可能指标，然后选择“总调用次数” 。
3. 在“聚合”列表中，选择“计数”。  通过此设置，你可以监视对认知服务资源的总调用次数，这对于确定一段时间内使用的服务数量非常有用。
4. 若要生成对认知服务的一些请求，你将使用 curl - 用于处理 HTTP 请求的命令行工具。 在 Visual Studio Code 中的 03-monitor 文件夹中，打开 rest-test.cmd 并编辑其中包含的 curl 命令（如下所示），并将 &lt;yourEndpoint&gt; 和 &lt;yourKey&gt; 分别替换为终结点 URI 和 Key1 密钥，以在认知服务资源中使用文本分析 API   。

    ```
    curl -X POST "<yourEndpoint>/text/analytics/v3.1/languages?" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: <yourKey>" --data-ascii "{'documents':           [{'id':1,'text':'hello'}]}"
    ```

5. 保存所做的更改，然后在 03-monitor 文件夹的集成终端中，运行以下命令：

    ```
    rest-test
    ```

该命令返回一个 JSON 文档，其中包含有关在输入数据中检测到的语言（该语言应为英语）的信息。

6. 多次重新运行 rest-test 命令，以生成一些调用活动（可以使用 ^ 键循环显示先前的命令） 。
7. 返回到 Azure 门户中的“指标”页面，并刷新“总调用次数”计数图表 。 使用 curl 进行的调用可能需要几分钟的时间才能反映在图表中，请不断刷新图表，直到图表更新为包含这些调用为止。

## <a name="more-information"></a>详细信息

用于监视认知服务的选项之一是使用诊断日志记录。 启用后，诊断日志记录将捕获有关认知服务资源使用情况的丰富信息，并且是一个有用的监视和调试工具。 设置诊断日志后可能需要超过一个小时才会生成任何信息，这就是我们在本练习中没有对其进行探讨的原因，但你可以在[认知服务文档](https://docs.microsoft.com/azure/cognitive-services/diagnostic-logging)中了解关于它的更多信息。
