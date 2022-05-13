---
lab:
  title: 创建对话语言理解客户端应用程序（预览版）
ms.openlocfilehash: 486a6716a189156739e9107824e561f7bb8fa95c
ms.sourcegitcommit: 2c92ea1491cac72a4765755cf2bc6d8b5f657a46
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/09/2022
ms.locfileid: "141580024"
---
# <a name="create-a-language-service-client-application"></a>创建语言服务客户端应用程序

适用于语言的 Azure 认知服务的对话语言理解功能使你能够定义一个对话语言模型，客户端应用可使用该模型来解释来自用户的自然语言输入，预测用户的意向（他们想要实现的目标），并识别应该应用该意向的任何实体 。 可以直接通过 REST 接口或使用特定于语言的软件开发工具包 (SDK) 创建使用对话语言理解模型的客户端应用程序。

## <a name="clone-the-repository-for-this-course"></a>克隆本课程的存储库

如果已将 AI-102-AIEngineer 代码存储库克隆到了要完成本实验室的环境，请在 Visual Studio Code 中将其打开；否则，请按照以下步骤立即将其克隆。

1. 启动 Visual Studio Code。

2. 打开面板 (SHIFT+CTRL+P) 并运行“Git: 克隆”命令，以将 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存储库克隆到本地文件夹（任意文件夹均可）。

3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。

4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择“以后再说”。

## <a name="create-language-service-resources"></a>创建语言服务资源

如果 Azure 订阅中已有语言服务资源，则可以在本练习中使用它， 否则请按照以下说明创建一个。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。

2. 选择“&#65291;创建资源”按钮，搜索“语言服务”，并使用以下设置创建“语言服务”资源：

    - **默认功能**：全部
    - **自定义功能**：无
    - **订阅**：Azure 订阅
    - 资源组：选择或创建一个资源组（如果使用受限制的订阅，你可能无权创建新的资源组 - 请使用提供的资源组）
    - **区域**：westus2
    - **名称**：输入唯一名称
    - **定价层**：*S*
    - **法律条款**：选中复选框以确认
    - **负责任的 AI 通知**：选中复选框以确认

3. 等待资源创建完成。 可以导航到你在其中创建了资源的资源组，然后查看该资源。

## <a name="import-train-and-publish-a-conversational-language-understanding-model"></a>导入、训练和发布对话语言理解模型

如果你已在前面的实验室或练习中准备好时钟项目，可以在本练习中使用它。 如果尚未准备好该应用，请按照以下说明进行创建。

1. 在新的浏览器标签页中，打开“语言工作室 - 预览版”门户 (`https://language.cognitive.azure.com`)。

2. 使用与 Azure 订阅关联的 Microsoft 帐户登录。 如果是首次登录“语言服务”门户，可能需要向应用授予一些用于访问帐户详细信息的权限。 然后选择你的 Azure 订阅和刚刚创建的创作资源，完成欢迎步骤。

3. 打开“对话语言理解”页面。

3. 在“&#65291;新建项目”旁边，查看下拉列表并选择“导入” 。 单击“选择文件”，然后浏览到项目文件夹中的 10b-clu-client-(preview) 子文件夹，其中包含本练习的实验室文件 。 选择“Clock.json”，单击“打开”，然后单击“完成”  。

4. 如果显示一个包含有关如何创建有效语言服务应用的提示的面板，请关闭该面板。

5. 在“语言工作室”门户的左侧，选择“训练模型”以训练应用。 单击“开始训练作业”，将模型命名为“时钟”，并确保启用了“训练时进行评估” 。 训练过程可能需要几分钟才能完成。

    > **注意**：由于模型名称“时钟”是在时钟客户端代码中进行硬编码的（稍后将在实验室中使用），因此请完全按照描述拼写名称并将其大写。 

6. 在“语言工作室”门户的左侧，选择“部署模型”，并使用“添加部署”为名为“生产”的时钟模型创建部署  。

    > **注意**：由于部署名称“生产”是在时钟客户端代码中进行硬编码的（稍后将在实验室中使用），因此请完全按照描述拼写名称并将其大写。 

7. 部署完成后，选择“生产”部署，然后单击“获取预测 URL” 。 客户端应用程序需要终结点 URL 才能使用已部署的模型。

8. 在“语言工作室”门户的左侧，选择“项目设置”并记下“主密钥” 。 客户端应用程序需要主密钥才能使用已部署的模型。

9. 客户端应用程序需要来自预测 URL 和语言服务密钥的信息才能连接到已部署的模型并进行身份验证。

## <a name="prepare-to-use-the-language-service-sdk"></a>准备使用语言服务 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用时钟模型（已发布的对话语言理解模型）来预测用户输入中的意向并作出适当响应。

> **注意**：可选择将该 SDK 用于 .NET 或 Python 。 在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 10b-clu-client-(preview) 文件夹，并根据你的语言首选项展开 C-Sharp 文件夹或 Python 文件夹   。

2. 右键单击 clock-client 文件夹，然后选择“打开集成终端” 。 然后通过运行适用于你的语言首选项的命令，安装对话语言服务 SDK 包：

    **C#**

    ```
    dotnet add package Azure.AI.Language.Conversations --version 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-language-conversations --pre
    python -m pip install python-dotenv
    python -m pip install python-dateutil
    ```

3. 查看 clock-client 文件夹的内容，并注意其中包含一个配置设置文件：

    - **C#** ：appsettings.json
    - **Python**：.env

    打开配置文件并更新其中包含的配置值，以包括语言资源的“终结点 URL”和“主密钥” 。 可在 Azure 门户或语言工作室中找到所需的值，如下所示：

    - Azure 门户：打开语言资源。 在“资源管理”下，选择“密钥和终结点” 。 将“密钥 1”和“终结点”值复制到配置设置文件中 。
    - 语言工作室：打开“时钟”项目。 可在“部署模型”页的“获取预测 URL”下找到语言服务终结点，并且可以在“项目设置”页中找到“主密钥”   。 预测 URL 的语言服务终结点部分以 .cognitiveservices.azure.com/ 结尾。 例如：`https://ai102-langserv.cognitiveservices.azure.com/`。

4. 请注意，clock-client 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：clock-client.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释“导入命名空间”。 然后在此注释下添加以下语言特定的代码，以导入使用语言服务 SDK 所需的命名空间：

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Language.Conversations;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.core.credentials import AzureKeyCredential
    from azure.ai.language.conversations import ConversationAnalysisClient
    from azure.ai.language.conversations.models import ConversationAnalysisOptions
    ```

## <a name="get-a-prediction-from-the-conversational-language-model"></a>从对话语言模型获取预测结果

现在你已准备好实现使用 SDK 从对话语言模型获取预测结果的代码。

1. 请注意，Main 函数中已经提供从配置文件加载预测终结点和密钥的代码。 下一步是找到注释“为语言服务模型创建客户端”，并添加以下代码来为语言服务应用创建预测客户端：

    **C#**

    ```C#
    // Create a client for the Language service model
    Uri lsEndpoint = new Uri(predictionEndpoint);
    AzureKeyCredential lsCredential = new AzureKeyCredential(predictionKey);

    ConversationAnalysisClient lsClient = new ConversationAnalysisClient(lsEndpoint, lsCredential);
    ```

    **Python**

    ```Python
    # Create a client for the Language service model
    client = ConversationAnalysisClient(
        ls_prediction_endpoint, AzureKeyCredential(ls_prediction_key))
    ```

2. 请注意，Main 函数中的代码会提示用户进行输入，直到用户输入“quit”。 在此循环中找到注释“调用语言服务模型以获取意向和实体”并添加以下代码：

    **C#**

    ```C#
    // Call the Language service model to get intent and entities
    var lsProject = "Clock";
    var lsSlot = "production";

    ConversationsProject conversationsProject = new ConversationsProject(lsProject, lsSlot);
    Response<AnalyzeConversationResult> response = await lsClient.AnalyzeConversationAsync(userText, conversationsProject);

    ConversationPrediction conversationPrediction = response.Value.Prediction as ConversationPrediction;

    Console.WriteLine(JsonConvert.SerializeObject(conversationPrediction, Formatting.Indented));
    Console.WriteLine("--------------------\n");
    Console.WriteLine(userText);
    var topIntent = "";
    if (conversationPrediction.Intents[0].Confidence > 0.5)
    {
        topIntent = conversationPrediction.TopIntent;
    }

    var entities = conversationPrediction.Entities;
    ```

    **Python**

    ```Python
    # Call the Language service model to get intent and entities
    convInput = ConversationAnalysisOptions(
        query = userText
        )
    
    cls_project = 'Clock'
    deployment_slot = 'production'

    with client:
        result = client.analyze_conversations(
            convInput, 
            project_name=cls_project, 
            deployment_name=deployment_slot
            )

    # list the prediction results
    top_intent = result.prediction.top_intent
    entities = result.prediction.entities

    print("view top intent:")
    print("\ttop intent: {}".format(result.prediction.top_intent))
    print("\tcategory: {}".format(result.prediction.intents[0].category))
    print("\tconfidence score: {}\n".format(result.prediction.intents[0].confidence_score))

    print("view entities:")
    for entity in entities:
        print("\tcategory: {}".format(entity.category))
        print("\ttext: {}".format(entity.text))
        print("\tconfidence score: {}".format(entity.confidence_score))

    print("query: {}".format(result.query))
    ```

    对语言服务模型的调用会返回一个预测/结果，其中包括排名第一的（最有可能的）意向以及在输入语句中检测到的所有实体。 你的客户端应用程序现在必须使用该预测来确定并执行适当的操作。

3. 找到注释“应用适当的操作”，并添加以下代码，该代码检查是否有应用程序支持的意向（GetTime、GetDate 和 GetDay）并确定是否检测到相关实体，然后调用一个现有函数来生成适当的响应   。

    **C#**

    ```C#
    // Apply the appropriate action
    switch (topIntent)
    {
        case "GetTime":
            var location = "local";
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a location entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Location")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        location = entity.Text;
                    }
                    
                }

            }

            // Get the time for the specified location
            var getTimeTask = Task.Run(() => GetTime(location));
            string timeResponse = await getTimeTask;
            Console.WriteLine(timeResponse);
            break;

        case "GetDay":
            var date = DateTime.Today.ToShortDateString();
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a Date entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Date")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        date = entity.Text;
                    }
                }
            }
            // Get the day for the specified date
            var getDayTask = Task.Run(() => GetDay(date));
            string dayResponse = await getDayTask;
            Console.WriteLine(dayResponse);
            break;

        case "GetDate":
            var day = DateTime.Today.DayOfWeek.ToString();
            // Check for entities
            if (entities.Count > 0)
            {
                // Check for a Weekday entity
                foreach (ConversationEntity entity in conversationPrediction.Entities)
                {
                    if (entity.Category == "Weekday")
                    {
                        //Console.WriteLine($"Location Confidence: {entity.Confidence}");
                        day = entity.Text;
                    }
                }
                
            }
            // Get the date for the specified day
            var getDateTask = Task.Run(() => GetDate(day));
            string dateResponse = await getDateTask;
            Console.WriteLine(dateResponse);
            break;

        default:
            // Some other intent (for example, "None") was predicted
            Console.WriteLine("Try asking me for the time, the day, or the date.");
            break;
    }
    ```

    **Python**

    ```Python
    # Apply the appropriate action
    if top_intent == 'GetTime':
        location = 'local'
        # Check for entities
        if len(entities) > 0:
            # Check for a location entity
            for entity in entities:
                if 'Location' == entity.category:
                    # ML entities are strings, get the first one
                    location = entity.text
        # Get the time for the specified location
        print(GetTime(location))

    elif top_intent == 'GetDay':
        date_string = date.today().strftime("%m/%d/%Y")
        # Check for entities
        if len(entities) > 0:
            # Check for a Date entity
            for entity in entities:
                if 'Date' == entity.category:
                    # Regex entities are strings, get the first one
                    date_string = entity.text
        # Get the day for the specified date
        print(GetDay(date_string))

    elif top_intent == 'GetDate':
        day = 'today'
        # Check for entities
        if len(entities) > 0:
            # Check for a Weekday entity
            for entity in entities:
                if 'Weekday' == entity.category:
                # List entities are lists
                    day = entity.text
        # Get the date for the specified day
        print(GetDate(day))

    else:
        # Some other intent (for example, "None") was predicted
        print('Try asking me for the time, the day, or the date.')
    ```

4. 保存你的更改并返回到 clock-client 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python clock-client.py
    ```

5. 系统提示时，输入言语来测试应用程序。 例如，尝试以下操作：

    *Hello*
    
    现在几点了？

    伦敦现在几点了？

    今天是几月几号？

    星期天是几号？

    今天星期几？

    2025 年 1 月 1 日是星期几？

> **注意**：该应用程序中的逻辑设计得很简单，并且有许多限制。 例如在获取时间时支持的城市是有限的，并且没有考虑夏令时。 我们的目标是通过一个示例了解使用语言服务的典型模式，在该模式中，你的应用程序必须：
>
>   1. 连接到预测终结点。
>   2. 通过提交一个言语来获取预测结果。
>   3. 实现逻辑，针对预测的意图和实体作出适当的响应。

6. 完成测试后，请输入“quit”。

## <a name="more-information"></a>详细信息

若要详细了解如何创建语言服务客户端，请参阅[开发人员文档](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)
