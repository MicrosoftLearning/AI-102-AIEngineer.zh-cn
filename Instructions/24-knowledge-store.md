---
lab:
    title: '使用 Azure 认知搜索创建知识存储'
    module: '模块 12 - 创建知识挖掘解决方案'
---

# 使用 Azure 认知搜索创建知识存储

Azure 认知搜索使用认知技能的扩充管道从文档中提取 AI 生成的字段并将这些字段包含在搜索索引中。尽管可以将索引视为索引过程的主要输出，但其包含的扩充数据也可以在其他方面发挥作用。例如：

- 由于索引本质上是一组 JSON 对象，且每个对象表示一个索引记录，因此使用 Azure 数据工厂等工具将对象导出为 JSON 文件以集成到数据协调流程可能会很有用。
- 建议将索引记录规范化为表的关系架构，以便使用 Microsoft Power BI 等工具进行分析和报告。
- 在索引过程中从文档中提取了嵌入图像后，建议将这些图像保存为文件。

在此练习中，你将为 *Margie's Travel* （一家虚构的旅行社）实现知识存储，以使用宣传册和酒店评价中的信息来帮助客户计划旅行。

## 克隆本课程的存储库

如果已将 **AI-102-AIEngineer** 代码存储库克隆到了要完成本实验室的环境，请在 Visual Studio Code 中将其打开；否则，请按照以下步骤立即将其克隆。

1. 启动 Visual Studio Code。
2. 打开面板 (Shift+Ctrl+P) 并运行 **Git: Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**： 如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

## 创建 Azure 资源

> **备注**： 如果你之前已完成了 **[创建 Azure 认知搜索解决方案](22-azure-search.md)** 练习，并且在订阅中仍有这些 Azure 资源，则可以跳过这部分，从 **“创建搜索解决方案”** 部分开始。否则，请按照以下步骤预配所需的 Azure 资源。

1. 在 Web 浏览器中，打开 Azure 门户 (`https://portal.azure.com`)，并使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 查看订阅中的**资源组**。
3. 如果使用的受限订阅已提供了资源组，请选择该资源组以查看其属性。否则，请使用首选名称创建一个新的资源组，并在创建后进入该资源组。
4. 记下资源组 **“概述”** 页上的 **“订阅 ID”** 和 **“位置”**。在后续步骤中，你将需要这些值以及资源组的名称。
5. 在 Visual Studio Code 中，展开 **24-knowledge-store** 文件夹，选择 **setup.cmd**。你将使用此批处理脚本来运行创建所需 Azure 资源时需要的 Azure 命令行接口 (CLI) 命令。
6. 右键单击 **24-knowledge-store** 文件夹，选择 **“在集成终端中打开”**。
7. 在终端窗格中，输入以下命令以与 Azure 订阅建立经认证的连接。

    ```
    az login --output none
    ```

8. 根据提示登录到 Azure 订阅。然后返回 Visual Studio Code，等待登录过程完成。
9. 运行以下命令以列出 Azure 位置。

    ```
    az account list-locations -o table
    ```

10. 在输出中，找到与资源组位置相对应的 **“名称”** 值（例如，对于*美国东部*，对应的名称是 *eastus*）。
11. 在 **setup.cmd** 脚本中，使用订阅 ID、资源组名称和位置名称的适当值修改 **subscription_id**、 **resource_group** 和 **location** 变量声明。然后保存更改。
12. 在 **24-knowledge-store** 文件夹的终端中，输入以下命令运行脚本：

    ```
    setup
    ```
    > **备注**： “搜索 CLI”模块为预览版，可能会卡在 *“- 正在运行…”* 过程中。如果这种情况超过 2 分钟，请按 CTRL+C 取消该长时间运行的操作，然后在系统询问是否要终止脚本时选择 **“N”**。然后就应该可以成功完成。
    >
    > 如果脚本失败，请确保已使用正确的变量名保存脚本，然后再试一次。

13. 脚本完成后，请查看它显示的输出，并记下以下有关 Azure 资源的信息（稍后将需要这些值）：
    - 存储帐户名称
    - 存储连接字符串
    - 认知服务帐户
    - 认知服务密钥
    - 搜索服务终结点
    - 搜索服务管理员密钥
    - 搜索服务查询密钥

14. 在 Azure 门户中，刷新资源组并验证它是否包含 Azure 存储帐户、Azure 认知服务资源和 Azure 认知搜索资源。

## 创建搜索解决方案

在获得必要的 Azure 资源后，即可创建由以下组件组成的搜索解决方案：

- **数据源** - 引用 Azure 存储容器中的文档。
- **技能组** - 定义用于从文档中提取 AI 生成的字段的技能扩充管道。该技能组还定义将在知识存储中生成**的投影**。
- **索引** - 定义一组可搜索的文档记录。
- **索引器** - 从数据源中提取文档、应用技能组并填充索引。索引编制过程还会在知识存储中保留技能组中定义的投影。

在本次练习中，你将使用 Azure 认知搜索 REST 接口通过提交 JSON 请求来创建这些组件。

### 为 REST 操作准备 JSON

使用 REST 接口提交 Azure 认知搜索组件的 JSON 定义。

1. 在 Visual Studio Code 的 **24-knowledge-store** 文件夹中，展开 **create-search** 文件夹，选择 **data_source.json**。该文件包含数据源 **“margies-knowledge-data”** 的 JSON 定义。
2. 将 **YOUR_CONNECTION_STRING** 占位符替换为 Azure 存储帐户的连接字符串，该字符串应如下所示：

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *可以在 Azure 门户中存储帐户的 **“访问密钥”** 页面上找到该连接字符串。*

3. 保存并关闭更新后的 JSON 文件。
4. 在 **create-search** 文件夹中，打开 **skillset.json**。该文件包含技能组 **“margies-knowledge-skillset”** 的 JSON 定义。
5. 在技能组定义顶部的 **cognitiveServices** 元素中，将 **YOUR_COGNITIVE_SERVICES_KEY** 占位符替换为认知服务资源的任何一个密钥。

    *可以在 Azure 门户中认知服务资源的 **“密钥和终结点”** 页面中找到这些密钥。*

6. 在技能组中技能集合的末尾，找到名为 **define-projection** 的 **Microsoft.Skills.Util.ShaperSkill** 技能。该技能为将用于投影的扩充数据定义了一个 JSON 结构，管道将在知识存储中为索引器处理的每个文档保留该结构。
7. 在技能组文件的底部，注意该技能组还包括一个 **knowledgeStore** 定义，其中包括要创建知识存储的 Azure 存储帐户的连接字符串，以及一个**投影**集合。此技能组包括以下三个*投影组*：
    - 包含*对象*投影的组，该组基于技能组中整形技能的 **knowledge_projection** 输出。
    - 包含*文件*投影的组，该组基于从文档中提取的图像数据的 **normalized_images** 集合。
    - 包含以下*表投*影的组：
        - **KeyPhrases**： 包含一个自动生成的键列和一个映射到整形技能的 **knowledge_projection/key_phrases/** 集合输出的 **keyPhrase** 列。
        - **Locations**： 包含一个自动生成的键列和一个映射到整形技能的 **knowledge_projection/key_phrases/** 集合输出的 **location** 列。
        - **ImageTags**： 包含一个自动生成的键列和一个映射到整形技能的 **knowledge_projection/image_tags/** 集合输出的 **tag** 列。
        - **Docs**： 包含一个自动生成的键列和整形技能中尚未分配到表的所有 **knowledge_projection** 输出值。
8. 使用存储帐户的连接字符串替换 **storageConnectionString** 值的 **YOUR_CONNECTION_STRING** 占位符。
9. 保存并关闭更新后的 JSON 文件。
10. 在 **create-search** 文件夹中，打开 **index.json**。该文件包含索引 **“margies-knowledge-index”** 的 JSON 定义。
11. 查看该索引的 JSON，然后关闭文件，不做任何修改。
12. 在 **create-search** 文件夹中，打开 **indexer.json**。该文件包含索引器 **“margies-knowledge-indexer”** 的 JSON 定义。
13. 查看该索引器的 JSON，然后关闭文件，不做任何修改。

### 提交 REST 请求

准备好用于定义搜索解决方案组件的 JSON 对象后，你可以将 JSON 文档提交到 REST 接口以创建这些组件。

1. 在 **create-search**  文件夹中，打开 **create-search.cmd**。此批处理脚本使用 cURL 实用程序将 JSON 定义提交到 Azure 认知搜索资源的 REST 接口。
2. 将 **YOUR_SEARCH_URL** 和 **YOUR_ADMIN_KEY** 变量占位符替换为 Azure 认知搜索资源的 **Url** 和其中一个**管理员密钥**。

    *可以在 Azure 门户中 Azure 认知搜索资源的 **“概述”** 和 **“密钥”** 页面上找到这些值。*

3. 保存更新后的批处理文件。
4. 右键单击 **create-search** 文件夹，选择 **“在集成终端中打开”**。
5. 在 **create-search** 文件夹的终端窗格中，输入以下命令运行批处理脚本。

    ```
    create-search
    ```

6. 脚本完成后，在 Azure 门户中 Azure 认知搜索资源的页面上，选择 **“索引器”** 页面，然后等待索引过程完成。

    *你可以选择 **“刷新”** 来跟踪索引操作的进度。可能需要一分钟左右的时间才能完成。*

    > **提示**： 如果脚本失败，请检查你在 **data_source.json**、 **skillset.json** 和 **create-search.cmd** 文件中添加的占位符。在纠正任何错误后，可能需要使用 Azure 门户用户界面删除搜索资源中创建的任何组件，然后重新运行脚本。

## 查看知识存储

在运行了使用技能组创建知识存储的索引器后，索引编制过程中提取的扩充数据会保留在知识存储投影中。

### 查看对象投影

Margie's Travel 技能组中定义的对象投影由每个索引文档的 JSON 文件组成。这些文件存储在技能集定义中指定的 Azure 存储帐户的 blob 容器中。

1. 在 Azure 门户中，查看之前创建的 Azure 存储帐户。
2. 选择 **“存储资源管理器”** 选项卡（在左侧窗格中），在 Azure 门户的存储资源管理器界面中查看存储帐户。
2. 展开 **“BLOB 容器”**，查看存储帐户中的容器。除了存储源数据的 **margies** 容器外，还应该有两个新的容器： **margies-images** 和 **margies-knowledge**。这些都是由索引编制过程产生的。
3. 选择 **margies-knowledge** 容器。它应该包含每个索引文档的文件夹。
4. 打开任意一个文件夹，然后下载并打开其中包含的 **knowledge-projection.json** 文件。每个 JSON 文件都包含索引文档的表示法，其中包括技能组提取的扩充数据，如下所示。

```
{
    "file_id":"abcd1234....",
    "file_name":"Margies Travel Company Info.pdf",
    "url":"https://store....blob.core.windows.net/margies/...pdf",
    "language":"en",
    "sentiment":0.83164644241333008,
    "key_phrases":[
        "Margie’s Travel",
        "Margie's Travel",
        "best travel experts",
        "world-leading travel agency",
        "international reach"
        ],
    "locations":[
        "Dubai",
        "Las Vegas",
        "London",
        "New York",
        "San Francisco"
        ],
    "image_tags":[
        "outdoor",
        "tree",
        "plant",
        "palm"
        ]
}
```

能够创建像这样的*对象*投影，就可以生成可纳入企业数据分析解决方案的扩充数据对象 - 例如，通过将 JSON 文件引入 Azure 数据工厂管道来进行进一步处理或加载到数据仓库。

### 查看文件投影

技能组中定义的文件投影将为在索引编制过程中从文档中提取的每个图像创建 JPEG *文件*。

1. 在 Azure 门户的存储资源管理器界面中，选择 **margies-images** blob 容器。此容器为每个包含图像的文档设置了一个文件夹。
2. 打开任何一个文件夹，查看其中的内容 - 每个文件夹至少包含一个 \*.jpg 文件。
3. 打开任何一个图像文件，验证它们是否包含从文件中提取的图像。

能够像这样生成*文件*投影，就可以高效编制索引，从大量文档中提取嵌入图像。

### 查看表投影

技能组中定义的*表投*影形成了扩充数据的关系模式。

1. 在 Azure 门户的存储资源管理器界面中，展开 **TABLES**。
2. 选择 **Docs** 表以查看其列。列包括一些标准的 Azure 存储表列 - 要隐藏这些列，请修改 **“列选项”**，仅选择以下列：
    - **document_id** （索引编制过程中自动生成的键列）
    - **file_id** （编码后的文件 URL）
    - **file_name** （从文件元数据中提取的文件名）
    - **language** （编写文档所采用的语言）
    - **sentiment** （计算出的该文档的情绪分数。）
    - **url** （Azure 存储中文档 blob 的 URL。）
3. 查看由索引编制过程创建的其他表：
    - **ImageTags** （为每个单独的图像标记设置一行，其中包含出现该标记的文档的 **document_id**）。
    - **KeyPhrases** （为每个单独的关键短语设置一行，其中包含出现该短语的文档的 **document_id**）。
    - **Locations** （为每个单独的位置设置一行，其中包含出现该短语的文档的 **document_id**）。

能够创建*表投*影，就能够构建（例如，使用 Microsoft Power BI）查询关系模式的分析和报告解决方案。自动生成的键列可用于在查询中连接各表，以返回特定文档中提到的所有位置等等。

## 更多信息

要了解有关使用 Azure 认知搜索创建知识存储的更多信息，请参阅 [Azure 认知搜索文档](https://docs.microsoft.com/azure/search/knowledge-store-concept-intro)。
