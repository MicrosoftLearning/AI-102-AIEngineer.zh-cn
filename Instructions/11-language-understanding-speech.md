---
lab:
    title: '使用语音和语言理解服务'
    module: '模块 5 - 创建语言理解解决方案'
---

# 使用语音和语言理解服务

可将语音服务与语言理解服务集成，以创建可以智能地根据语音输入确定用户意图的应用程序。

> **备注**： 本练习最适合使用麦克风的情况。某些托管的虚拟环境可能能够通过本地麦克风捕获音频，但如果这不起作用（或者根本没有麦克风），则可以使用提供的音频文件进行语音输入。请仔细按照说明进行操作，因为需要根据使用的是麦克风还是音频文件来选择不同的选项。

## 克隆本课程的存储库

如果已将 **AI-102-AIEngineer** 代码存储库克隆到了要完成本实验室的环境，请在 Visual Studio Code 中将其打开；否则，请按照以下步骤立即将其克隆。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行 **Git: Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**： 如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

## 创建语言理解资源

如果 Azure 订阅中已有语言理解创作和预测资源，可以在本练习中使用它们。如果没有此类资源，请按以下说明进行创建操作。

1. 打开 Azure 门户 (`https://portal.azure.com`)，使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择 **“&#65291;创建资源”** 按钮，搜索 *“语言理解”*，并使用以下设置创建一个**语言理解**资源：
    - **创建选项**：两个
    - **订阅**： *你的 Azure 订阅*
    - **资源组**： *选择或创建一个资源组（如果你使用的是受限订阅，则可能无权创建新资源组，在此情况下，可使用一个已提供的资源组）*
    - **名称**： *输入唯一名称*
    - **创作位置**： *选择首选位置*
    - **创作定价层**： F0
    - **预测位置**： *选择与创作位置相同的位置*
    - **预测定价层**： F0（如果 F0 不可用，请选择 S0）

3. 等待资源创建完成，并注意是否已预配两个语言理解资源；一个用于创作，另一个用于预测。可以导航到你在其中创建了这些资源的资源组，然后查看这两个资源。

## 准备语言理解应用

如果你已在前面的练习中准备好**时钟**应用，请在语言理解门户 (https://www.luis.ai) 中打开它。如果尚未准备好该应用，请按照以下说明进行创建。

1. 在新的浏览器标签页中，打开语言理解门户 (`https://www.luis.ai`)。
2. 使用与 Azure 订阅关联的 Microsoft 帐户登录。如果这是你首次登录语言理解门户，可能需要授予该应用一些权限，用于访问你的帐户详细信息。然后选择你的 Azure 订阅和刚刚创建的创作资源，完成**欢迎**步骤。
3. 打开 **“&#65291;新建应用”** 旁边的 **“对话应用”** 页面，查看下拉列表并选择 **“导入为 LU”**。
在包含用于本练习的实验室文件的项目文件夹中浏览至 **“11-luis-speech”** 子文件夹，并选择 **“Clock.lu”**。然后为时钟应用指定唯一名称。
4. 如果显示一个包含有关如何创建有效语言理解应用的提示的面板，请关闭该面板。

## 使用*语音启动*训练并发布该应用

1. 如果尚未训练应用，请在“语言理解”门户顶部，选择 **“训练”** 以训练应用。
2. 在语言理解门户的右上角选择 **“发布”**。然后选择 **“生产槽位”** 并更改设置以启用 **“语音启动”** （这将改善语音识别的性能）。
3. 发布完成后，选择语言理解门户顶部的 **“管理”**。
4. 记下 **“设置”** 页面中的 **“应用 ID”**。客户端应用程序在使用应用时需要用到该 ID。
5. 在 **“Azure 资源”** 页面上的 **“预测资源”** 下，如果其中没有列出任何预测资源，请添加 Azure 订阅中的预测资源。
6. 请记下预测资源的 **“主密钥”**、 **“辅助密钥”** 以及 **“位置”** （不是终结点！）。要连接到预测资源并完成身份验证，语音 SDK 客户端应用程序需要使用位置以及其中一个密钥。

## 配置语言理解客户端应用程序

在本练习中，你将创建一个客户端应用程序，该应用程序接受语音输入并使用语言理解应用来预测用户的意图。

> **备注**： 在本练习中，可选择将该 SDK 用于 **C#** 或 **Python**。在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的 **“资源管理器”** 窗格中，浏览到 **11-luis-speech** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 查看 **speaking-clock-client** 文件夹的内容，并注意其中包含用于配置设置的文件：
    - **C#**： appsettings.json
    - **Python**： .env

    打开配置文件并更新其中包含的配置值，以添加语言理解应用**的应用 ID**、预测资源的**位置**（不是完整的终结点，例如 *“eastus”*）以及预测资源的**密钥**之一（通过语言理解门户中应用的 **“管理”** 页面）。

## 安装 SDK 包

若要结合使用语音 SDK 与语言理解服务，需要为编程语言安装语音 SDK 包。

1. 在 Visual Studio 中，右键单击 **speaking-clock-client** 文件夹，并打开集成终端。然后通过运行适用于你的语言首选项的命令，安装语言理解 SDK 包：

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

2. 此外，如果系统<u>没有</u>正常工作的麦克风，则需要使用音频文件为应用程序提供语音输入。在这种情况下，请使用以下命令安装其他包，确保程序可以播放音频文件（如果要使用麦克风，可以跳过此步骤）：

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

3. 请注意， **speaking-clock-client** 文件夹中包含客户端应用程序的代码文件：

    - **C#**： Program.cs
    - **Python**： speaking-clock-client.py

4. 打开代码文件，并在顶部的现有命名空间引用下找到注释 **“导入命名空间”**。然后在此注释下添加以下语言特定的代码，以导入使用语音 SDK 所需的命名空间：

    **C#**

    ```C#
    // 导入命名空间
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Intent;
    ```

    **Python**

    ```Python
    # 导入命名空间
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. 此外，如果系统<u>没有</u>正常工作的麦克风，在现有命名空间导入下，请添加以下代码以导入将用于播放音频文件的库：

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

## 创建 *IntentRecognizer*

**IntentRecognizer** 类提供了一个客户端对象，你可以使用该对象从语音输入中获取语言理解预测。

1. 请注意，**Main** 函数中已经提供从配置文件加载应用 ID、预测区域和密钥的代码。然后，找到注释 **“配置语音服务并获取意图识别器”**，并根据是使用麦克风还是使用音频文件进行语音输入来添加以下代码：

    ### **如果麦克风正常工作：**

    **C#**

    ```C#
    // 配置语音服务并获取意图识别器
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # 配置语音服务并获取意图识别器
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

    ### **如果需要使用音频文件：**

    **C#**

    ```C#
    // 配置语音服务并获取意图识别器
    string audioFile = "time-in-london.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    System.Threading.Thread.Sleep(2000);
    SpeechConfig speechConfig = SpeechConfig.FromSubscription(predictionKey, predictionRegion);
    AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    IntentRecognizer recognizer = new IntentRecognizer(speechConfig, audioConfig);
    ```

    **Python**

    ```Python
    # 配置语音服务并获取意图识别器
    audioFile = 'time-in-london.wav'
    playsound(audioFile)
    speech_config = speech_sdk.SpeechConfig(subscription=lu_prediction_key, region=lu_prediction_region)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    recognizer = speech_sdk.intent.IntentRecognizer(speech_config, audio_config)
    ```

## 通过语音输入预测意图

现在你已准备好实现使用语音 SDK 从语音输入中预测意图的代码。

1. 在 **Main** 函数中，在刚刚添加的代码下找到注释 **“通过应用 ID 获取模型并添加要使用的意图”**，然后添加以下代码来获取语言理解模型（根据其应用 ID）并指定我们需要识别器识别的意图。

    **C#**

    ```C#
    // 通过应用 ID 获取模型并添加要使用的意图
    var model = LanguageUnderstandingModel.FromAppId(luAppId);
    recognizer.AddIntent(model, "GetTime", "time");
    recognizer.AddIntent(model, "GetDate", "date");
    recognizer.AddIntent(model, "GetDay", "day");
    recognizer.AddIntent(model, "None", "none");
    ```

    *请注意，可以为每个意图指定一个基于字符串的 ID*

    **Python**

    ```Python
    # 通过应用 ID 获取模型并添加要使用的意图
    model = speech_sdk.intent.LanguageUnderstandingModel(app_id=lu_app_id)
    intents = [
        (model, "GetTime"),
        (model, "GetDate"),
        (model, "GetDay"),
        (model, "None")
    ]
    recognizer.add_intents(intents)
    ```

2. 在注释 **“处理语音输入”** 下，添加以下代码，该代码使用识别器通过语音输入异步调用语言理解服务，并检索响应。如果响应中包括预测的意图，则会显示语音查询、预测的意图和完整的 JSON 响应。否则，代码将根据返回的原因处理响应。

**C#**

```C
// 处理语音输入
string intent = "";
var result = await recognizer.RecognizeOnceAsync().ConfigureAwait(false);
if (result.Reason == ResultReason.RecognizedIntent)
{
    // 识别到意图
    intent = result.IntentId;
    Console.WriteLine($"Query: {result.Text}");
    Console.WriteLine($"Intent Id: {intent}.");
    string jsonResponse = result.Properties.GetProperty(PropertyId.LanguageUnderstandingServiceResponse_JsonResult);
    Console.WriteLine($"JSON Response:\n{jsonResponse}\n");
    
    // 获取第一个实体（若有）

    // 应用适当的操作
    
}
else if (result.Reason == ResultReason.RecognizedSpeech)
{
    // 识别到语音，但未识别到意图。
    intent = result.Text;
    Console.Write($"I don't know what {intent} means.");
}
else if (result.Reason == ResultReason.NoMatch)
{
    // 未识别到语音
    Console.WriteLine($"Sorry. I didn't understand that.");
}
else if (result.Reason == ResultReason.Canceled)
{
    // 出错
    var cancellation = CancellationDetails.FromResult(result);
    Console.WriteLine($"CANCELED: Reason={cancellation.Reason}");
    if (cancellation.Reason == CancellationReason.Error)
    {
        Console.WriteLine($"CANCELED: ErrorCode={cancellation.ErrorCode}");
        Console.WriteLine($"CANCELED: ErrorDetails={cancellation.ErrorDetails}");
    }
}
```

**Python**

```Python
# 处理语音输入
intent = ''
result = recognizer.recognize_once_async().get()
if result.reason == speech_sdk.ResultReason.RecognizedIntent:
    intent = result.intent_id
    print("Query: {}".format(result.text))
    print("Intent: {}".format(intent))
    json_response = json.loads(result.intent_json)
    print("JSON Response:\n{}\n".format(json.dumps(json_response, indent=2)))
    
    # 获取第一个实体（若有）
    
    # 应用适当的操作
    
elif result.reason == speech_sdk.ResultReason.RecognizedSpeech:
    # 识别到语音，但未识别到意图。
    intent = result.text
    print("I don't know what {} means.".format(intent))
elif result.reason == speech_sdk.ResultReason.NoMatch:
    # 未识别到语音
    print("Sorry. I didn't understand that.")
elif result.reason == speech_sdk.ResultReason.Canceled:
    # 出错
    print("Intent recognition canceled: {}".format(result.cancellation_details.reason))
    if result.cancellation_details.reason == speech_sdk.CancellationReason.Error:
        print("Error details: {}".format(result.cancellation_details.error_details))
```

你目前已添加的代码会识别*意图*，但是一些意图可以引用*实体*，所以必须添加代码以从服务返回的 JSON 中提取实体信息。

3. 在刚刚添加的代码中找到注释 **“获取第一个实体（若有）”**，并在该注释下添加以下代码：

**C#**

```C
// 获取第一个实体（若有）
JObject jsonResults = JObject.Parse(jsonResponse);
string entityType = "";
string entityValue = "";
if (jsonResults["entities"].HasValues)
{
    JArray entities = new JArray(jsonResults["entities"][0]);
    entityType = entities[0]["type"].ToString();
    entityValue = entities[0]["entity"].ToString();
    Console.WriteLine(entityType + ": " + entityValue);
}
```

**Python**

```Python
# 获取第一个实体（若有）
entity_type = ''
entity_value = ''
if len(json_response["entities"]) > 0:
    entity_type = json_response["entities"][0]["type"]
    entity_value = json_response["entities"][0]["entity"]
    print(entity_type + ': ' + entity_value)
```
    
你的代码现在使用语言理解应用来预测意图以及在输入的言语中检测到的任何实体。你的客户端应用程序现在必须使用该预测来确定并执行适当的操作。

4. 在刚刚添加的代码下找到注释 **“应用适当的操作”**，并添加以下代码，该代码检查是否有应用程序支持的意图（**GetTime**、**GetDate** 和 **GetDay**）并确定是否检测到相关实体，然后调用一个现有函数来生成适当的响应。

**C#**

```C#
// 应用适当的操作
switch (intent)
{
    case "time":
        var location = "local";
        // 检查是否存在实体
        if (entityType == "Location")
        {
            location = entityValue;
        }
        // 获取指定位置的时间
        var getTimeTask = Task.Run(() => GetTime(location));
        string timeResponse = await getTimeTask;
        Console.WriteLine(timeResponse);
        break;
    case "day":
        var date = DateTime.Today.ToShortDateString();
        // 检查是否存在实体
        if (entityType == "Date")
        {
            date = entityValue;
        }
        // 获取指定日期的星期
        var getDayTask = Task.Run(() => GetDay(date));
        string dayResponse = await getDayTask;
        Console.WriteLine(dayResponse);
        break;
    case "date":
        var day = DateTime.Today.DayOfWeek.ToString();
        // 检查是否存在实体
        if (entityType == "Weekday")
        {
            day = entityValue;
        }

        var getDateTask = Task.Run(() => GetDate(day));
        string dateResponse = await getDateTask;
        Console.WriteLine(dateResponse);
        break;
    default:
        // 预测到一些其他意图（例如，“None”）
        Console.WriteLine("You said " + result.Text.ToLower());
        if (result.Text.ToLower().Replace(".", "") == "stop")
        {
            intent = result.Text;
        }
        else
        {
            Console.WriteLine("Try asking me for the time, the day, or the date.");
        }
        break;
}
```

**Python**

```Python
# 应用适当的操作
if intent == 'GetTime':
    location = 'local'
    # 检查是否存在实体
    if entity_type == 'Location':
        location = entity_value
    # 获取指定位置的时间
    print(GetTime(location))

elif intent == 'GetDay':
    date_string = date.today().strftime("%m/%d/%Y")
    # 检查是否存在实体
    if entity_type == 'Date':
        date_string = entity_value
    # 获取指定日期的星期
    print(GetDay(date_string))

elif intent == 'GetDate':
    day = 'today'
    # 检查是否存在实体
    if entity_type == 'Weekday':
        # 列表实体是列表
        day = entity_value
    # 获取指定日的日期
    print(GetDate(day))

else:
    # 预测到一些其他意图（例如，“None”）
    print('You said {}'.format(result.text))
    if result.text.lower().replace('.', '') == 'stop':
        intent = result.text
    else:
        print('Try asking me for the time, the day, or the date.')
```

## 运行客户端应用程序

1. 保存你的更改并返回到 **speaking-clock-client** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python speaking-clock-client.py
    ```

2. 如果使用麦克风，请通过大声说话来测试应用程序。例如，尝试说出以下语句（每次都应重新运行程序）：

    *What is the time?（现在几点了？）*
    
    *现在几点了？*

    *今天星期几？*

    *What is the time in London?（伦敦现在几点了？）*

    *今天是几月几号？*

    *星期天是几号？*

> **备注**： 该应用程序中的逻辑设计得很简单，并且有许多限制，但是应该能够测试语言理解模型是否有使用语音 SDK 预测语音输入意图的能力。你可能很难通过具体的日期实体来识别 **GetDay** 意图，因为用言语表述 *MM/DD/YYYY* 格式的日期是很困难的！

## 更多信息

要了解关于语音和语言理解集成的详细信息，请参阅[语音文档](https://docs.microsoft.com/azure/cognitive-services/speech-service/quickstarts/intent-recognition)。
