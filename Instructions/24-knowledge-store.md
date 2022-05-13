---
lab:
  title: 使用 Azure 认知搜索创建知识存储
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: 8d7008ee592d4a76f24ca26a4b192c7dd5cbe9ea
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819413"
---
# <a name="create-a-knowledge-store-with-azure-cognitive-search"></a>使用 Azure 认知搜索创建知识存储

Azure 认知搜索使用认知技能的扩充管道从文档中提取 AI 生成的字段并将这些字段包含在搜索索引中。 虽然索引可能会被认为是索引过程的主要输出，但其包含的扩充数据在其他方面也可能很有用。 例如：

- 由于索引本质上是 JSON 对象的集合，每个对象代表一个索引记录，因此对于使用 Azure 数据工厂等工具将对象导出为 JSON 文件以集成到数据业务流程过程，这可能会很有用。
- 你可能想要使用 Microsoft Power BI 等工具将索引记录规范化为表的关系架构，以供分析和报告。
- 在索引过程中从文档提取嵌入图像后，你可能想要将这些图像另存为文件。

在此练习中，你将为 Margie's Travel（一家虚构的旅行社）实现知识存储，以使用宣传册和酒店评价中的信息来帮助客户计划旅行。

## <a name="clone-the-repository-for-this-course"></a>克隆本课程的存储库

如果已将 AI-102-AIEngineer 代码存储库克隆到了要完成本实验室的环境，请在 Visual Studio Code 中将其打开；否则，请按照以下步骤立即将其克隆。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“Git: 克隆”命令，以将 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择“以后再说”。

## <a name="create-azure-resources"></a>创建 Azure 资源

> **注意**：如果你之前已完成了“[创建 Azure 认知搜索解决方案](22-azure-search.md)”练习，并且在订阅中仍有这些 Azure 资源，则可以跳过这部分，从“创建搜索解决方案”部分开始 。 否则，请按照以下步骤预配所需的 Azure 资源。

1. 在 Web 浏览器中，打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 查看订阅中的资源组。
3. 如果使用的受限订阅已提供了资源组，请选择该资源组以查看其属性。 否则，请使用首选名称创建一个新的资源组，并在创建后进入该资源组。
4. 记下资源组“概述”页上的“订阅 ID”和“位置”。 在后续步骤中，你将需要这些值以及资源组的名称。
5. 在 Visual Studio Code 中，展开 24-knowledge-store 文件夹，选择 setup.cmd。 你将使用此批处理脚本来运行创建所需 Azure 资源时需要的 Azure 命令行接口 (CLI) 命令。
6. 右键单击 24-knowledge-store 文件夹，选择“在集成终端中打开”。
7. 在终端窗格中输入以下命令，与 Azure 订阅建立经过身份验证的连接。

    ```
    az login --output none
    ```

8. 根据提示登录到 Azure 订阅。 然后，返回到 Visual Studio Code 并等待登录过程完成。
9. 运行以下命令以列出 Azure 位置。

    ```
    az account list-locations -o table
    ```

10. 在输出中，找到与资源组位置相对应的“名称”值（例如，对于美国东部，对应的名称是 eastus） 。
11. 在 setup.cmd 脚本中，使用订阅 ID、资源组名称和位置名称的适当值修改 subscription_id、resource_group 和 location 变量声明。 保存更改。
12. 在 24-knowledge-store 文件夹的终端中，输入以下命令运行脚本：

    ```
    setup
    ```
    > **注意**：“搜索 CLI”模块为预览版，可能会卡在“- 正在运行…” 映像。 如果这种情况超过 2 分钟，请按 CTRL+C 取消该长时间运行的操作，然后在系统询问是否要终止脚本时选择“N”。 然后就应该可以成功完成。
    >
    > 如果脚本失败，请确保已使用正确的变量名保存脚本，然后再试一次。

13. 脚本完成后，查看输出，并记录有关 Azure 资源的以下信息（稍后会用到这些值）：
    - 存储帐户名称
    - 存储连接字符串
    - 认知服务帐户
    - 认知服务密钥
    - 搜索服务终结点
    - 搜索服务管理密钥
    - 搜索服务查询密钥

14. 在 Azure 门户中，刷新资源组并验证它是否包含 Azure 存储帐户、Azure 认知服务资源和 Azure 认知搜索资源。

## <a name="create-a-search-solution"></a>创建搜索解决方案

在获得必要的 Azure 资源后，即可创建由以下组件组成的搜索解决方案：

- 数据源 - 引用 Azure 存储容器中的文档。
- 技能组 - 定义用于从文档中提取 AI 生成的字段的技能扩充管道。 该技能组还定义将在知识存储中生成的投影 。
- 索引 - 定义一组可搜索的文档记录。
- 索引器 - 从数据源中提取文档、应用技能组并填充索引。 索引编制过程还会在知识存储中保留技能组中定义的投影。

在本次练习中，你将使用 Azure 认知搜索 REST 接口通过提交 JSON 请求来创建这些组件。

### <a name="prepare-json-for-rest-operations"></a>为 REST 操作准备 JSON

使用 REST 接口提交 Azure 认知搜索组件的 JSON 定义。

1. 在 Visual Studio Code 的 24-knowledge-store 文件夹中，展开 create-search 文件夹，选择 data_source.json。 该文件包含数据源“margies-knowledge-data”的 JSON 定义。
2. 将 YOUR_CONNECTION_STRING 占位符替换为 Azure 存储帐户的连接字符串，该字符串应如下所示：

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    可以在 Azure 门户中存储帐户的“访问密钥”页上找到该连接字符串。

3. 保存并关闭更新后的 JSON 文件。
4. 在 create-search 文件夹中，打开 skillset.json。 该文件包含技能组“margies-knowledge-skillset”的 JSON 定义。
5. 在技能组定义顶部的 cognitiveServices 元素中，将 YOUR_COGNITIVE_SERVICES_KEY 占位符替换为认知服务资源的任何一个密钥。

    可以在 Azure 门户中认知服务资源的“密钥和终结点”页上找到这些密钥。

6. 在技能组中技能集合的末尾，找到名为 define-projection 的 Microsoft.Skills.Util.ShaperSkill 技能 。 该技能为将用于投影的扩充数据定义了一个 JSON 结构，管道将在知识存储中为索引器处理的每个文档保留该结构。
7. 在技能组文件的底部，注意该技能组还包括一个 knowledgeStore 定义，其中包括要创建知识存储的 Azure 存储帐户的连接字符串，以及一个投影集合 。 此技能组包括 3 个投影组：
    - 一组包含基于技能组中形状绘制技能的 knowledge_projection 输出的对象投影。
    - 一组包含基于从文档提取的图像数据的 normalized_images 集合的文件投影。
    - 一组包含以下表投影：
        - **KeyPhrases**：包含自动生成的键列和 keyPhrase 列，这些列映射到形状绘制技能的 knowledge_projection/key_phrases/ 集合输出中。
        - **Locations**：包含自动生成的键列和位置列，这些列映射到形状绘制技能的 knowledge_projection/key_phrases/ 集合输出中。
        - **ImageTags**：包含自动生成的键列和标记列，这些列映射到形状绘制技能的 knowledge_projection/image_tags/ 集合输出中。
        - **Docs**：包含自动生成的键列以及所有来自尚未分配给表的形状绘制技能的 knowledge_projection 输出值。
8. 使用存储帐户的连接字符串替换 storageConnectionString 值的 YOUR_CONNECTION_STRING 占位符 。
9. 保存并关闭更新后的 JSON 文件。
10. 在 create-search 文件夹中，打开 index.json。 该文件包含索引“margies-knowledge-index”的 JSON 定义。
11. 查看该索引的 JSON，然后关闭文件，不做任何修改。
12. 在 create-search 文件夹中，打开 indexer.json。 该文件包含索引器“margies-knowledge-indexer”的 JSON 定义。
13. 查看该索引器的 JSON，然后关闭文件，不做任何修改。

### <a name="submit-rest-requests"></a>提交 REST 请求

准备好用于定义搜索解决方案组件的 JSON 对象后，你可以将 JSON 文档提交到 REST 接口以创建这些组件。

1. 在 create-search 文件夹中，打开 create-search.cmd。 此批处理脚本使用 cURL 实用程序将 JSON 定义提交到 Azure 认知搜索资源的 REST 接口。
2. 将 YOUR_SEARCH_URL 和 YOUR_ADMIN_KEY 变量占位符替换为 Azure 认知搜索资源的 Url 和其中一个管理员密钥。

    可以在 Azure 门户中 Azure 认知搜索资源的“概述”和“密钥”页上找到这些值 **。

3. 保存更新后的批处理文件。
4. 右键单击 create-search 文件夹，选择“在集成终端中打开”。
5. 在 create-search 文件夹的终端窗格中，输入以下命令运行批处理脚本。

    ```
    create-search
    ```

6. 脚本完成后，在 Azure 门户中 Azure 认知搜索资源的页面上，选择“索引器”页面，然后等待索引过程完成。

    你可以选择“刷新”来跟踪索引操作的进度。可能需要一分钟左右的时间才能完成。

    > **提示**：如果脚本失败，请检查你在 data_source.json、skillset.json 和 create-search.cmd 文件中添加的占位符  。 在纠正任何错误后，可能需要使用 Azure 门户用户界面删除搜索资源中创建的任何组件，然后重新运行脚本。

## <a name="view-the-knowledge-store"></a>查看知识存储

运行使用技能组创建知识存储的索引器后，由索引过程提取的扩充数据将持久保存在知识存储投影中。

### <a name="view-object-projections"></a>查看对象投影

对于每个索引文档，Margie's Travel 技能组中定义的对象投影由 JSON 文件组成。 这些文件存储在技能组定义中指定的 Azure 存储帐户中的 Blob 容器中。

1. 在 Azure 门户中，查看之前创建的 Azure 存储帐户。
2. （在左侧窗格中）选择“存储资源管理器”选项卡，在 Azure 门户的存储资源管理器界面中查看存储帐户。
2. 展开 BLOB CONTAINERS，查看存储帐户中的容器。 除了存储源数据的 margies 容器，还应有两个新的容器：margies-images 和 margies-knowledge。 这些容器都是由索引过程创建的。
3. 选择 margies-knowledge 容器。 对于每个索引文档，它都应包含一个文件夹。
4. 打开任意一个文件夹，然后下载并打开其中包含的 knowledge-projection.json 文件。 每个 JSON 文件都包含一个索引文档的表示形式，包括由技能组提取的扩充数据，如下所示。

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

借助可创建这类对象投影的功能，你能够生成可合并到企业数据分析解决方案的扩充数据对象；例如，通过将 JSON 文件引入 Azure 数据工厂管道中，来进一步处理或加载到数据仓库。

### <a name="view-file-projections"></a>查看文件投影

在索引过程中，技能组中定义的文件投影会为从文档中提取的每个图像都创建 JPEG 文件。

1. 在 Azure 门户的存储资源管理器界面中，选择 margies-images blob 容器。 该容器会为每个包含图像的文档包含一个文件夹。
2. 打开任意文件夹并查看其内容，每个文件夹至少包含一个 \*.jpg 文件。
3. 打开任意图像文件，确认其包含从文档中提取的图像。

凭借可生成这类文件投影的功能，索引成为了从大量文档中提取嵌入图像的有效方法。

### <a name="view-table-projections"></a>查看表投影

技能组中定义的表投影构成了扩充数据的关系架构。

1. 在 Azure 门户的存储资源管理器界面中，展开“TABLES”。
2. 选择“Docs”表以查看其列。 这些列包括一些标准的 Azure 存储表列；要隐藏它们，请修改“列选项”以仅选择以下列：
    - document_id（由索引过程自动生成的键列）
    - file_id（编码的文件 URL）
    - file_name（从文档元数据中提取的文件名）
    - 语言（编写文档所用的语言）。
    - 情绪（为文档计算的情绪分数）。
    - URL（Azure 存储中文档 Blob 的 URL）。
3. 查看索引过程创建的其他表：
    - ImageTags（包含每个单独的图像标记对应的行，并包含显示了该标记的文档的 document_id）。
    - KeyPhrases（包含每个单独的关键短语对应的行，并包含在显示了该短语的文档的 document_id）。
    - Locations（包含每个单独位置对应的行，并包含显示了该位置的文档的 document_id）。

能够创建表投影，就能够构建（例如，使用 Microsoft Power BI）查询关系模式的分析和报告解决方案。 自动生成的键列可用于联接查询中的表，例如返回特定文档中提到的所有位置。

## <a name="more-information"></a>详细信息

要了解有关使用 Azure 认知搜索创建知识存储的更多信息，请参阅 [Azure 认知搜索文档](https://docs.microsoft.com/azure/search/knowledge-store-concept-intro)。
