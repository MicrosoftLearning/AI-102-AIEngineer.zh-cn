---
lab:
  title: 创建 Azure 认知搜索的自定义技能
  module: Module 12 - Creating a Knowledge Mining Solution
ms.openlocfilehash: 1321115b5c06fb6791f24f9f89311524be14accc
ms.sourcegitcommit: 883a607fbe52f48a9425c38a997f776ecd0130af
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/01/2022
ms.locfileid: "141347626"
---
# <a name="create-a-custom-skill-for-azure-cognitive-search"></a>创建 Azure 认知搜索的自定义技能

Azure 认知搜索使用认知技能的扩充管道从文档中提取 AI 生成的字段并将这些字段包含在搜索索引中。 该服务提供了一套综合性的内置技能供你使用，但如果你有这些技能无法满足的特定要求，你可以创建自定义技能。

在本次练习中，你将创建一个自定义技能，该技能可以将文档中各单词出现的频率制表以生成前五个最常用单词的列表，然后将其添加到 Margie's Travel（一家虚构的旅行社）的搜索解决方案中。

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
5. 在 Visual Studio Code 中，展开 23-custom-search-skill 文件夹，选择 setup.cmd。 你将使用此批处理脚本来运行创建所需 Azure 资源时需要的 Azure 命令行接口 (CLI) 命令。
6. 右键单击 23-custom-search-skill 文件夹，选择“在集成终端中打开”。
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
12. 在 23-custom-search-skill 文件夹的终端中，输入以下命令运行脚本：

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
- 技能组 - 定义用于从文档中提取 AI 生成的字段的技能扩充管道。
- 索引 - 定义一组可搜索的文档记录。
- 索引器 - 从数据源中提取文档、应用技能组并填充索引。

在本次练习中，你将使用 Azure 认知搜索 REST 接口通过提交 JSON 请求来创建这些组件。

1. 在 Visual Studio Code 的 23-custom-search-skill 文件夹中，展开 create-search 文件夹，选择 data_source.json。 该文件包含数据源“margies-custom-data”的 JSON 定义。
2. 将 YOUR_CONNECTION_STRING 占位符替换为 Azure 存储帐户的连接字符串，该字符串应如下所示：

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    可以在 Azure 门户中存储帐户的“访问密钥”页上找到该连接字符串。

3. 保存并关闭更新后的 JSON 文件。
4. 在 create-search 文件夹中，打开 skillset.json。 该文件包含技能组“margies-custom-skillset”的 JSON 定义。
5. 在技能组定义顶部的 cognitiveServices 元素中，将 YOUR_COGNITIVE_SERVICES_KEY 占位符替换为认知服务资源的任何一个密钥。

    可以在 Azure 门户中认知服务资源的“密钥和终结点”页上找到这些密钥。

6. 保存并关闭更新后的 JSON 文件。
7. 在 create-search 文件夹中，打开 index.json。 该文件包含索引“margies-custom-index”的 JSON 定义。
8. 查看该索引的 JSON，然后关闭文件，不做任何修改。
9. 在 create-search 文件夹中，打开 indexer.json。 该文件包含索引器“margies-custom-indexer”的 JSON 定义。
10. 查看该索引器的 JSON，然后关闭文件，不做任何修改。
11. 在 create-search 文件夹中，打开 create-search.cmd。 此批处理脚本使用 cURL 实用程序将 JSON 定义提交到 Azure 认知搜索资源的 REST 接口。
12. 将 YOUR_SEARCH_URL 和 YOUR_ADMIN_KEY 变量占位符替换为 Azure 认知搜索资源的 Url 和其中一个管理员密钥。

    可以在 Azure 门户中 Azure 认知搜索资源的“概述”和“密钥”页上找到这些值 **。

13. 保存更新后的批处理文件。
14. 右键单击 create-search 文件夹，选择“在集成终端中打开”。
15. 在 create-search 文件夹的终端窗格中，输入以下命令运行批处理脚本。

    ```
    create-search
    ```

16. 脚本完成后，在 Azure 门户中 Azure 认知搜索资源的页面上，选择“索引器”页面，然后等待索引过程完成。

    你可以选择“刷新”来跟踪索引操作的进度。可能需要一分钟左右的时间才能完成。

## <a name="search-the-index"></a>搜索索引

现在你已拥有索引，可以搜索它。

1. 在 Azure 认知搜索资源边栏选项卡的顶部，选择“搜索资源管理器”。
2. 在搜索资源管理器的“查询字符串”框中，输入以下查询字符串，然后选择“搜索” 。

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment eq 'positive'
    ```

    该查询可检索评论家撰写的所有提及伦敦时具有积极情绪标签（换句话说，提及伦敦的正面评论）的文档的 url、情绪和关键词   

## <a name="create-an-azure-function-for-a-custom-skill"></a>为自定义技能创建 Azure 函数

搜索解决方案包括一些内置的认知技能，这些技能可以使用文档中的信息（例如在之前的任务中看到的情绪分数和关键短语列表）来扩充索引。

你可以通过创建自定义技能来进一步增强索引。 例如，识别每个文档中使用频率最高的单词可能很有用，但无内置技能可提供此功能。

要以自定义技能的形式实现字数统计功能，你需要使用首选语言创建一个 Azure 函数。

> **注意**：在此练习中，你将使用 Azure 门户中的代码编辑功能来创建一个简单的 Node.JS 函数。 在生产解决方案中，通常使用 Visual Studio Code 等开发环境并采用首选语言（例如 C#、Python、Node.JS 或 Java）创建函数应用，并将其作为 DevOps 过程的一部分发布到 Azure。

1. 在 Azure 门户的“主页”上，创建一个新的函数应用资源，设置如下 ：
    - 订阅：*你的订阅*
    - **资源组**：Azure 认知搜索资源所在的同一资源组
    - **函数应用名称**：唯一的名称
    - **发布**：代码
    - **运行时堆栈**：Node.js
    - **版本**：14 LTS
    - 区域：Azure 认知搜索资源所在的同一区域

2. 等待部署完成，然后转到部署的函数应用资源。
3. 在函数应用的边栏选项卡的左侧窗格中，选择“函数”选项卡。然后创建一个新的函数，设置如下：
    - **设置开发环境**
        - **开发环境**：在门户中进行开发
    - **选择模板**
        - **模板**：HTTP 触发器
    - **模板详细信息**：
        - **新建函数**：wordcount
        - **授权级别**：函数
4. 等待 wordcount 函数创建完成。 然后，在其页面中，选择“代码 + 测试”选项卡。
5. 将默认函数代码替换为以下代码：

```javascript
module.exports = async function (context, req) {
    context.log('JavaScript HTTP trigger function processed a request.');

    if (req.body && req.body.values) {

        vals = req.body.values;

        // Array of stop words to be ignored
        var stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
        "youre", "youve", "youll", "youd", 'your', 'yours', 'yourself', 
        'yourselves', 'he', 'him', 'his', 'himself', 'she', "shes", 'her', 
        'hers', 'herself', 'it', "its", 'itself', 'they', 'them', 
        'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
        'this', 'that', "thatll", 'these', 'those', 'am', 'is', 'are', 'was',
        'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
        'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
        'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
        'about', 'against', 'between', 'into', 'through', 'during', 'before', 
        'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
        'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
        'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
        'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
        'only', 'own', 'same', 'so', 'than', 'too', 'very', 'can', 'will',
        'just', "dont", 'should', "shouldve", 'now', "arent", "couldnt", 
        "didnt", "doesnt", "hadnt", "hasnt", "havent", "isnt", "mightnt", "mustnt",
        "neednt", "shant", "shouldnt", "wasnt", "werent", "wont", "wouldnt"];

        res = {"values":[]};

        for (rec in vals)
        {
            // Get the record ID and text for this input
            resVal = {recordId:vals[rec].recordId, data:{}};
            txt = vals[rec].data.text;

            // remove punctuation and numerals
            txt = txt.replace(/[^ A-Za-z_]/g,"").toLowerCase();

            // Get an array of words
            words = txt.split(" ")

            // count instances of non-stopwords
            wordCounts = {}
            for(var i = 0; i < words.length; ++i) {
                word = words[i];
                if (stopwords.includes(word) == false )
                {
                    if (wordCounts[word])
                    {
                        wordCounts[word] ++;
                    }
                    else
                    {
                        wordCounts[word] = 1;
                    }
                }
            }

            // Convert wordcounts to an array
            var topWords = [];
            for (var word in wordCounts) {
                topWords.push([word, wordCounts[word]]);
            }

            // Sort in descending order of count
            topWords.sort(function(a, b) {
                return b[1] - a[1];
            });

            // Get the first ten words from the first array dimension
            resVal.data.text = topWords.slice(0,9)
              .map(function(value,index) { return value[0]; });

            res.values[rec] = resVal;
        };

        context.res = {
            body: JSON.stringify(res),
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
    else {
        context.res = {
            status: 400,
            body: {"errors":[{"message": "Invalid input"}]},
            headers: {
            'Content-Type': 'application/json'
        }

        };
    }
};
```

6. 保存函数，然后打开“测试/运行”窗格。
7. 在“测试/运行”窗格中，将现有的主体替换为以下 JSON，以反映 Azure 认知搜索技能期望的架构，在该架构中，提交包含一个或多个文档数据的记录以进行处理：

    ```
    {
        "values": [
            {
                "recordId": "a1",
                "data":
                {
                "text":  "Tiger, tiger burning bright in the darkness of the night.",
                "language": "en"
                }
            },
            {
                "recordId": "a2",
                "data":
                {
                "text":  "The rain in spain stays mainly in the plains! That's where you'll find the rain!",
                "language": "en"
                }
            }
        ]
    }
    ```
    
8. 单击“运行”，并查看函数返回的 HTTP 响应内容。 这反映了在使用一个技能时 Azure 认知搜索所需的架构，在该架构中返回每个文档的响应。 在本例中，响应包含每个文档中出现次数最多的 10 个词，并以出现频率降序排列：

    ```
    {
    "values": [
        {
        "recordId": "a1",
        "data": {
            "text": [
            "tiger",
            "burning",
            "bright",
            "darkness",
            "night"
            ]
        },
        "errors": null,
        "warnings": null
        },
        {
        "recordId": "a2",
        "data": {
            "text": [
            "rain",
            "spain",
            "stays",
            "mainly",
            "plains",
            "thats",
            "youll",
            "find"
            ]
        },
        "errors": null,
        "warnings": null
        }
    ]
    }
    ```

9. 关闭“测试/运行”窗格，然后在 wordcount 函数的边栏选项卡中，单击“获取函数 URL”。 然后将默认密钥的 URL 复制到剪贴板。 下一过程将需要此 URL。

## <a name="add-the-custom-skill-to-the-search-solution"></a>将自定义技能添加到搜索解决方案

现在需要将函数作为自定义技能包含在搜索解决方案技能组中，并将其生成的结果映射到索引中的字段。 

1. 在 Visual Studio Code 的 23-custom-search-skill/update-search 文件夹中，打开 update-skillset.json 文件 。 该文件中包含技能集的 JSON 定义。
2. 查看技能集定义。 其中包含与之前相同的技能，以及一个名为 get-top-words 的新 WebApiSkill 技能。
3. 编辑 get-top-words 技能定义，将 uri 值设置为 Azure 函数的 URL（在之前的过程中已复制到剪贴板），从而替换 YOUR-FUNCTION-APP-URL  。
4. 在技能组定义顶部的 cognitiveServices 元素中，将 YOUR_COGNITIVE_SERVICES_KEY 占位符替换为认知服务资源的任何一个密钥。

    可以在 Azure 门户中认知服务资源的“密钥和终结点”页上找到这些密钥。

5. 保存并关闭更新后的 JSON 文件。
6. 在 update-search 文件夹中，打开 update-index.json。 此文件包含 margies-custom-index 索引的 JSON 定义，并在索引定义的底部有一个名为 top_words 的附加字段 。
7. 查看该索引的 JSON，然后关闭文件，不做任何修改。
8. 在 update-search 文件夹中，打开 update-indexer.json。 此文件包含 margies-custom-indexer 的 JSON 定义，并具有 top_words 字段的附加映射 。
9. 查看该索引器的 JSON，然后关闭文件，不做任何修改。
10. 在 update-search 文件夹中，打开 update-search.cmd。 此批处理脚本使用 cURL 实用程序将更新后的 JSON 定义提交到 Azure 认知搜索资源的 REST 接口。
11. 将 YOUR_SEARCH_URL 和 YOUR_ADMIN_KEY 变量占位符替换为 Azure 认知搜索资源的 Url 和其中一个管理员密钥。

    可以在 Azure 门户中 Azure 认知搜索资源的“概述”和“密钥”页上找到这些值 **。

12. 保存更新后的批处理文件。
13. 右键单击 update-search 文件夹，选择“在集成终端中打开”。
14. 在 update-search 文件夹的终端窗格中，输入以下命令运行批处理脚本。

    ```
    update-search
    ```

15. 脚本完成后，在 Azure 门户中 Azure 认知搜索资源的页面上，选择“索引器”页面，然后等待索引过程完成。

    你可以选择“刷新”来跟踪索引操作的进度。可能需要一分钟左右的时间才能完成。

## <a name="search-the-index"></a>搜索索引

现在你已拥有索引，可以搜索它。

1. 在 Azure 认知搜索资源边栏选项卡的顶部，选择“搜索资源管理器”。
2. 在搜索资源管理器的“查询字符串”框中，输入以下查询字符串，然后选择“搜索” 。

    ```
    search=Las Vegas&$select=url,top_words
    ```

    此查询检索提到拉斯维加斯 (Las Vegas) 的所有文档的 url 和 top_words 字段 。

## <a name="more-information"></a>详细信息

要了解有关为 Azure 认知搜索创建自定义技能的更多信息，请参阅 [Azure 认知搜索文档](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface)。
