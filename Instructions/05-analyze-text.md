---
lab:
    title: '分析文本'
    module: '模块 3 - 自然语言处理入门指南'
---

# 分析文本

**文本分析 API** 是一种支持文本分析的认知服务，包括语言检测、情绪分析、关键短语提取和实体识别。

例如，假设一家旅行社想要处理已提交到公司网站的酒店评论。通过使用文本分析 API，他们可以确定每条评论所采用的语言、评论所包含的情绪（积极、中立或消极）、可能指示评论中所讨论主题的关键短语，以及命名实体，例如评论中提及的地点、地标或人。

## 克隆本课程的存储库

如果尚未将 **AI-102-AIEngineer** 代码存储库克隆到你要在此实验室中使用的环境，请按照以下步骤克隆它。 否则，请在 Visual Studio Code 中打开克隆的文件夹。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行 **Git: Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**： 如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

## 预配认知服务资源

如果你的订阅中还没有**认知服务**资源，需要预配一个。

1. 打开 Azure 门户 `https://portal.azure.com`，使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择 **“&#65291;创建资源”** 按钮，搜索 *“认知服务”*，并使用以下设置创建一个**认知服务**资源：
    - **订阅**： *你的 Azure 订阅*
    - **资源组**： *选择或创建一个资源组（如果你使用的是受限订阅，则可能无权创建新资源组，在此情况下，可使用一个已提供的资源组）*
    - **区域**：*选择任何可用区域*
    - **名称**： *输入唯一名称*
    - **定价层**： 标准 S0
3. 选中所需复选框并创建资源。
4. 等待部署完成，然后查看部署详细信息。
5. 部署资源后，转到该资源并查看其 **“密钥和终结点”** 页面。你将在下一个过程中用到此页面中的终结点和其中一个密钥。

## 准备使用文本分析 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用文本分析 SDK 来分析酒店评论。

> **备注**： 可选择将该 SDK 用于 **C#** 或 **Python**。在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的 **“资源管理器”** 窗格中，浏览到 **05-analyze-text** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **text-analysis** 文件夹，并打开集成终端。然后通过运行适用于你的语言首选项的命令，安装文本分析 SDK 包：
    
    **C#**
    
    ```
    dotnet add package Azure.AI.TextAnalytics --version 5.0.0
    ```
    
    **Python**
    
    ```
    pip install azure-ai-textanalytics==5.0.0
    ```
    
3. 查看 **text-analysis** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#**： appsettings.json
    - **Python**： .env

    打开配置文件，然后更新其中包含的配置值，以反映认知服务资源的终**结点**和身份验证**密钥**。保存更改。

4. 请注意，**text-analysis** 文件夹中包含客户端应用程序的代码文件：

    - **C#**： Program.cs
    - **Python**： text-analysis.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释 **“导入命名空间”**。然后在此注释下添加以下特定于语言的代码，以导入使用文本分析 SDK 所需的命名空间：

    **C#**
    
    ```C#
    // 导入命名空间
    using Azure;
    using Azure.AI.TextAnalytics;
    ```

    **Python**

    ```Python
    # 导入命名空间
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.textanalytics import TextAnalyticsClient
    ```

5. 请注意，**Main** 函数中已经提供从配置文件加载认知服务终结点和密钥的代码。然后，找到注释 **“使用终结点和密钥创建客户端”**，并添加以下代码为文本分析 API 创建客户端：

    **C#**

    ```C#
    // 使用终结点和密钥创建客户端
    AzureKeyCredential credentials = new AzureKeyCredential(cogSvcKey);
    Uri endpoint = new Uri(cogSvcEndpoint);
    TextAnalyticsClient CogClient = new TextAnalyticsClient(endpoint, credentials);
    ```

    **Python**

    ```Python
    # 使用终结点和密钥创建客户端
    credential = AzureKeyCredential(cog_key)
    cog_client = TextAnalyticsClient(endpoint=cog_endpoint, credential=credential)
    ```

6. 保存你的更改并返回到 **text-analysis** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

6. 观察输出，因为代码应正常运行且不出现错误，并在 **“reviews”** 文件夹中显示每个评论文本文件的内容。应用程序为文本分析 API 成功创建客户端，但未使用该客户端。我们将在下一过程中修复此问题。

## 检测语言

为文本分析 API 创建客户端后，现在，让我们使用该客户端来检测每条评论所用的语言。

1. 在程序的 **Main** 函数中，找到注释 **“获取语言”**。然后，在此注释下，添加检测每个评论文档中的语言所需的代码：

    **C#**
    
    ```C
    // 获取语言
    DetectedLanguage detectedLanguage = CogClient.DetectLanguage(text);
    Console.WriteLine($"\nLanguage: {detectedLanguage.Name}");
    ```

    **Python**
    
    ```Python
    # 获取语言
    detectedLanguage = cog_client.detect_language(documents=[text])[0]
    print('\nLanguage: {}'.format(detectedLanguage.primary_language.name))
    ```

    > **备注**： *此示例对每条评论单独进行了分析，从而导致对每个文件分别调用服务。另一种方法是创建文档集合，并在单次调用中将它们传递给服务。在这两种方法中，服务的响应都包含文档集合；这就是为什么在上面的 Python 代码中，指定了响应 ([0]) 中第一个（也是唯一一个）文档的索引。*

6. 保存你的更改并返回到 **text-analysis** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

7. 观察输出，注意此次已确定每条评论的语言。

## 评估情绪

*情绪分析*是一种常用技术，可将文本分类为积*极情绪*或*消极*情绪（也可能为*中立或*混合情绪）。它通常用于分析社交媒体文章、产品评论和其他项目，其中，文本的情绪可能提供有用的见解。

1. 在程序的 **Main** 函数中，找到注释 **“获取情绪”**。然后，在此注释下，添加检测每个评论文档的情绪所需的代码：

    **C#**
    
    ```C
    // 获取情绪
    DocumentSentiment sentimentAnalysis = CogClient.AnalyzeSentiment(text);
    Console.WriteLine($"\nSentiment: {sentimentAnalysis.Sentiment}");
    ```

    **Python**
    
    ```Python
    # 获取情绪
    sentimentAnalysis = cog_client.analyze_sentiment(documents=[text])[0]
    print("\nSentiment: {}".format(sentimentAnalysis.sentiment))
    ```

2. 保存你的更改并返回到 **text-analysis** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```

    **Python**

    ```
    python text-analysis.py
    ```

3. 观察输出，注意已检测到评论的情绪。

## 标识关键短语

识别正文中的关键短语可帮助确定文本所讨论的主题。

1. 在程序的 **Main** 函数中，找到注释 **“获取关键短语”**。然后，在此注释下，添加检测每个评论文档中的关键短语所需的代码：

    **C#**

    ```C
    // 获取关键短语
    KeyPhraseCollection phrases = CogClient.ExtractKeyPhrases(text);
    if (phrases.Count > 0)
    {
        Console.WriteLine("\nKey Phrases:");
        foreach(string phrase in phrases)
        {
            Console.WriteLine($"\t{phrase}");
        }
    }
    ```
    
    **Python**
    
    ```Python
    # 获取关键短语
    phrases = cog_client.extract_key_phrases(documents=[text])[0].key_phrases
    if len(phrases) > 0:
        print("\nKey Phrases:")
        for phrase in phrases:
            print('\t{}'.format(phrase))
    ```

2. 保存你的更改并返回到 **text-analysis** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. 观察输出，注意每个文档都包含一些关键短语，你可通过这些短语了解评论的内容。

## 提取实体

通常，文档或其他文本正文会提及人物、地点、时期或其他实体。文本分析 API 可以检测文本中多个类别（和子类别）的实体。

1. 在程序的 **Main** 函数中，找到注释 **“获取实体”**。然后，在此注释下，添加识别每条评论中所提及的实体所需的代码：

    **C#**
    
    ```C
    // 获取实体
    CategorizedEntityCollection entities = CogClient.RecognizeEntities(text);
    if (entities.Count > 0)
    {
        Console.WriteLine("\nEntities:");
        foreach(CategorizedEntity entity in entities)
        {
            Console.WriteLine($"\t{entity.Text} ({entity.Category})");
        }
    }
    ```

    **Python**
    
    ```Python
    # 获取实体
    entities = cog_client.recognize_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nEntities")
        for entity in entities:
            print('\t{} ({})'.format(entity.text, entity.category))
    ```

2. 保存你的更改并返回到 **text-analysis** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. 观察输出，注意文本中已检测到的实体。

## 提取链接实体

除了分类实体外，文本分析 API 还可以检测与数据源（如 Wikipedia）存在已知链接的实体。

1. 在程序的 **Main** 函数中，找到注释 **“获取链接实体”**。然后，在此注释下，添加识别每条评论中所提及的链接实体所需的代码：

    **C#**
    
    ```C
    // 获取链接实体
    LinkedEntityCollection linkedEntities = CogClient.RecognizeLinkedEntities(text);
    if (linkedEntities.Count > 0)
    {
        Console.WriteLine("\nLinks:");
        foreach(LinkedEntity linkedEntity in linkedEntities)
        {
            Console.WriteLine($"\t{linkedEntity.Name} ({linkedEntity.Url})");
        }
    }
    ```

    **Python**
    
    ```Python
    # 获取链接实体
    entities = cog_client.recognize_linked_entities(documents=[text])[0].entities
    if len(entities) > 0:
        print("\nLinks")
        for linked_entity in entities:
            print('\t{} ({})'.format(linked_entity.name, linked_entity.url))
    ```

2. 保存你的更改并返回到 **text-analysis** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-analysis.py
    ```

3. 观察输出，注意已识别的链接实体。

## 更多信息

如需详细了解如何使用**文本分析**服务，请参阅[文本分析文档](https://docs.microsoft.com/azure/cognitive-services/text-analytics/)。
