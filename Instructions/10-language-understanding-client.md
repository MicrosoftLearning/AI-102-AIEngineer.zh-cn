---
lab:
    title: '创建语言理解客户端应用程序'
    module: '模块 5 - 创建语言理解解决方案'
---

# 创建语言理解客户端应用程序

语言理解服务使你能够定义封装了语言模型的应用，应用程序可使用该模型来解释用户的自然语言输入、预测用户的意图（他们想要实现的目标），以及识别应应用该*意图*的任何*实体*。可以直接通过 REST 接口或通过使用特定于语言的软件开发工具包 (SDK) 创建使用语言理解应用的客户端应用程序。

## 克隆本课程的存储库

如果已将 **AI-102-AIEngineer** 代码存储库克隆到了要完成本实验室的环境，请在 Visual Studio Code 中将其打开；否则，请按照以下步骤立即将其克隆。

1. 启动 Visual Studio Code。
2. 打开面板 (Shift+Ctrl+P) 并运行 **Git: Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**： 如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

## 创建语言理解资源

如果 Azure 订阅中已有语言理解创作和预测资源，可以在本练习中使用它们。如果没有此类资源，请按以下说明进行创建操作。

1. 打开 Azure 门户 (`https://portal.azure.com`)，使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择 **“&#65291;创建资源”** 按钮，搜索 *“语言理解”*，并使用以下设置创建一个**语言理解**资源：
    - **创建选项**： 两个
    - **订阅**： *你的 Azure 订阅*
    - **资源组**： *选择或创建一个资源组（如果你使用的是受限订阅，则可能无权创建新资源组，在此情况下，可使用一个已提供的资源组）*
    - **名称**： *输入唯一名称*
    - **创作位置**： *选择首选位置*
    - **创作定价层**： F0
    - **预测位置**： *选择与创作位置相同的位置*
    - **预测定价层**： F0（如果 F0 不可用，请选择 S0）

3. 等待资源创建完成，并注意是否已预配两个语言理解资源；一个用于创作，另一个用于预测。可以导航到你在其中创建了这些资源的资源组，然后查看这两个资源。

## 导入、训练并发布一个语言理解应用

如果你已在前面的练习中准备好**时钟**应用，可以在本练习中使用它。如果尚未准备好该应用，请按照以下说明进行创建。

1. 在新的浏览器标签页中，打开语言理解门户 (`https://www.luis.ai`)。
2. 使用与 Azure 订阅关联的 Microsoft 帐户登录。如果这是你首次登录语言理解门户，可能需要授予该应用一些权限，用于访问你的帐户详细信息。然后选择你的 Azure 订阅和刚刚创建的创作资源，完成*欢迎*步骤。
3. 打开 **“&#65291;新建应用”** 旁边的 **“对话应用”** 页面，查看下拉列表并选择 **“导入为 LU”**。
在包含用于本练习的实验室文件的项目文件夹中浏览至 **“10-luis-client”** 子文件夹，并选择 **“Clock.lu”**。然后为时钟应用指定唯一名称。
4. 如果显示一个包含有关如何创建有效语言理解应用的提示的面板，请关闭该面板。
5. 在“语言理解”门户顶部，选择 **“训练”** 以训练应用。
6. 选择“语言理解”门户右上角的 **“发布”**，并将应用发布至 **“生产槽位”**。
7. 发布完成后，选择语言理解门户顶部的 **“管理”**。
8. 记下 **“设置”** 页面中的 **“应用 ID”**。客户端应用程序在使用应用时需要用到该 ID。
9. 在 **“Azure 资源”** 页面上的 **“预测资源”** 下，如果其中没有列出任何预测资源，请添加 Azure 订阅中的预测资源。
10. 请记下预测资源的 **“主密钥”**、 **“辅助密钥”** 以及 **“终结点 URL”**。要连接到预测资源并完成身份验证，客户端应用程序需要使用终结点以及其中一个密钥。

## 准备使用语言理解 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用时钟语言理解应用预测用户输入中的意图并作出适当响应。

> **备注**： 可选择将该 SDK 用于 **C#** 或 **Python**。在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的“ **资源管理器”** 窗格中，浏览到 **10-luis-client** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **clock-client** 文件夹，并打开集成终端。然后通过运行适用于你的语言首选项的命令，安装语言理解 SDK 包：

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime --version 3.0.0
```

*除了**运行**时（预测）包以外，还有一个**创作**包，能编写用于创建和管理语言理解模型的代码。*

**Python**

```
pip install azure-cognitiveservices-language-luis==0.7.0
```
    
*Python SDK 包包括用于**预测**和**创作**的类。*

3. 查看 **clock-client** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#**： appsettings.json
    - **Python**： .env

    打开配置文件并更新其中包含的配置值，以添加语言理解应用**的应用 ID**、 **终结点 URL** 及预测资源的**密钥**之一（通过 “语言理解” 门户中应用的 **“管理”** 页面）。

4. 请注意， **clock-client** 文件夹中包含客户端应用程序的代码文件：

    - **C#**： Program.cs
    - **Python**： clock-client.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释 **“导入命名空间”**。然后在此注释下添加以下特定于语言的代码，以导入使用语言理解预测 SDK 所需的命名空间：

**C#**

```C#
// 导入命名空间
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime;
using Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.Models;
```

**Python**

```Python
# 导入命名空间
from azure.cognitiveservices.language.luis.runtime import LUISRuntimeClient
from msrest.authentication import CognitiveServicesCredentials
```

## 通过语言理解应用获取预测结果

现在你已准备好实现使用 SDK 从语言理解应用获取预测结果的代码。

1. 请注意，**Main** 函数中已经提供从配置文件加载应用 ID、预测终结点和密钥的代码。下一步是找到注释 **“为 LU 应用创建客户端”**，并添加以下代码来为语言理解应用创建预测客户端：

**C#**

```C#
// 为 LU 应用创建客户端
var credentials = new Microsoft.Azure.CognitiveServices.Language.LUIS.Runtime.ApiKeyServiceClientCredentials(predictionKey);
var luClient = new LUISRuntimeClient(credentials) { Endpoint = predictionEndpoint };
```

**Python**

```Python
# 为 LU 应用创建客户端
credentials = CognitiveServicesCredentials(lu_prediction_key)
lu_client = LUISRuntimeClient(lu_prediction_endpoint, credentials)
```

2. 请注意，**Main** 函数中的代码会提示用户进行输入，直到用户输入 “quit”。在此循环中找到注释 **“调用 LU 应用以获取意图和实体”** 并添加以下代码：

**C#**

```C#
// 调用 LU 应用以获取意图和实体
var slot = "Production";
var request = new PredictionRequest { Query = userText };
PredictionResponse predictionResponse = await luClient.Prediction.GetSlotPredictionAsync(luAppId, slot, request);
Console.WriteLine(JsonConvert.SerializeObject(predictionResponse, Formatting.Indented));
Console.WriteLine("--------------------\n");
Console.WriteLine(predictionResponse.Query);
var topIntent = predictionResponse.Prediction.TopIntent;
var entities = predictionResponse.Prediction.Entities;
```

**Python**

```Python
# 调用 LU 应用以获取意图和实体
request = { "query" : userText }
slot = 'Production'
prediction_response = lu_client.prediction.get_slot_prediction(lu_app_id, slot, request)
top_intent = prediction_response.prediction.top_intent
entities = prediction_response.prediction.entities
print('Top Intent: {}'.format(top_intent))
print('Entities: {}'.format (entities))
print('-----------------\n{}'.format(prediction_response.query))
```

对语言理解应用的调用会返回一个预测，其中包括排名第一的（最有可能的）意图以及在输入言语中检测到的所有实体。你的客户端应用程序现在必须使用该预测来确定并执行适当的操作。

3. 找到注释 **“应用适当的操作”**，并添加以下代码，该代码检查是否有应用程序支持的意图（**GetTime**、 **GetDate** 和 **GetDay**）并确定是否检测到相关实体，然后调用一个现有函数来生成适当的响应。

**C#**

```C#
// 应用适当的操作
switch (topIntent)
{
    case "GetTime":
        var location = "local";
        // 检查是否存在实体
        if (entities.Count > 0)
        {
            // 检查是否存在位置实体
            if (entities.ContainsKey("Location"))
            {
                //获取实体的 JSON
                var entityJson = JArray.Parse(entities["Location"].ToString());
                // ML 实体是字符串，获取第一个
                location = entityJson[0].ToString();
            }
        }

        // 获取指定位置的时间
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;

    case "GetDay":
        var date = DateTime.Today.ToShortDateString();
        // 检查是否存在实体
        if (entities.Count > 0)
        {
            // 检查是否存在日期实体
            if (entities.ContainsKey("Date"))
            {
                //获取实体的 JSON
                var entityJson = JArray.Parse(entities["Date"].ToString());
                // 正则表达式实体是字符串，获取第一个
                date = entityJson[0].ToString();
            }
        }
        // 获取指定日期的星期
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;

    case "GetDate":
        var day = DateTime.Today.DayOfWeek.ToString();
        // 检查是否存在实体
        if (entities.Count > 0)
        {
            // 检查是否存在工作日实体
            if (entities.ContainsKey("Weekday"))
            {
                //获取实体的 JSON
                var entityJson = JArray.Parse(entities["Weekday"].ToString());
                // 列表实体是列表
                day = entityJson[0][0].ToString();
            }
        }
        // 获取指定日的日期
        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;

    default:
        // 预测到一些其他意图（例如，“None”）
        Console.WriteLine("Try asking me for the time, the day, or the date.");
        break;
}
```

**Python**

```Python
# 应用适当的操作
if top_intent == 'GetTime':
    location = 'local'
    # 检查是否存在实体
    if len(entities) > 0:
        # 检查是否存在位置实体
        if 'Location' in entities:
            # ML 实体是字符串，获取第一个
            location = entities['Location'][0]
    # 获取指定位置的时间
    print(GetTime(location))

elif top_intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # 检查是否存在实体
    if len(entities) > 0:
        # 检查是否存在日期实体
        if 'Date' in entities:
            # 正则表达式实体是字符串，获取第一个
            date_string = entities['Date'][0]
    # 获取指定日期的星期
    print(GetDay(date_string))

elif top_intent == 'GetDate':
    day = 'today'
    # 检查是否存在实体
    if len(entities) > 0:
        # 检查是否存在工作日实体
        if 'Weekday' in entities:
            # 列表实体是列表
            day = entities['Weekday'][0][0]
    # 获取指定日的日期
    print(GetDate(day))

else:
    # 预测到一些其他意图（例如，“None”）
    print('Try asking me for the time, the day, or the date.')
```
    
4. 保存你的更改并返回到 **clock-client** 文件夹的集成终端，然后输入以下命令以运行程序：

**C#**

```
dotnet run
```

**Python**

```
python clock-client.py
```

5. 系统提示时，输入言语来测试应用程序。例如尝试输入以下言语：

    *你好*
    
    *现在几点了？*

    *伦敦现在几点了？*

    *今天是几月几号？*

    *星期天是几号？*

    *今天星期几？*

    *2025 年 1 月 1 日是星期几？*

> **备注**： 该应用程序中的逻辑设计得很简单，并且有许多限制。例如在获取时间时支持的城市是有限的，并且没有考虑夏令时。我们的目标是通过一个示例了解使用语言理解的典型模式，在这个模式中，你的应用程序必须：
>
>   1. 连接到预测终结点。
>   2. 通过提交一个言语来获取预测结果。
>   3. 实现逻辑，针对预测的意图和实体作出适当的响应。

6. 完成测试后，请输入 *“quit”*。

## 更多信息

要详细了解如何创建语言理解客户端，请参阅[开发人员文档](https://docs.microsoft.com/azure/cognitive-services/luis/developer-reference-resource)
