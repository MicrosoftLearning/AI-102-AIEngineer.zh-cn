---
lab:
    title: '翻译语音'
    module: '模块 4 - 构建支持语音的应用程序'
---

# 翻译语音

**语音**服务包括一个可用于翻译口语的**语音翻译** API。例如，假设你想要开发一种翻译工具应用程序，当人们在不会讲当地语言的地方旅行时便可以使用该应用程序。他们能够以自己的语言说出诸如“车站在哪里？”或“我需要寻找药店”之类的惯用语，并让该应用程序将其翻译为当地语言。

**备注**： 此练习需要使用配有扬声器/耳机的计算机。为了获得最佳体验，还需要麦克风。某些托管的虚拟环境可能能够通过本地麦克风捕获音频，但如果这不起作用（或者根本没有麦克风），则可以使用提供的音频文件进行语音输入。请仔细按照说明进行操作，因为需要根据使用的是麦克风还是音频文件来选择不同的选项。

## 克隆本课程的存储库

如果已将 **AI-102-AIEngineer** 代码存储库克隆到了要完成本实验室的环境，请在 Visual Studio Code 中将其打开；否则，请按照以下步骤立即将其克隆。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行 **Git: Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**： 如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

## 预配认知服务资源

如果你的订阅中还没有**认知服务**资源，则需要预配一个该资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择 **“&#65291;创建资源”** 按钮，搜索 *“认知服务”*，并使用以下设置创建一个**认知服务**资源：
    - **订阅**： *你的 Azure 订阅*
    - **资源组**： *选择或创建一个资源组（如果你使用的是受限订阅，则可能无权创建新资源组，在此情况下，可使用一个已提供的资源组）*
    - **区域**： *选择任何可用区域*
    - **名称**： *输入唯一名称*
    - **定价层**： 标准 S0
3. 选中所需复选框并创建资源。
4. 等待部署完成，然后查看部署详细信息。
5. 部署资源后，转到该资源并查看其 **“密钥和终结点”** 页面。在下一过程中，你将需要其中一个密钥以及从此页面预配服务的位置。

## 准备使用语音翻译服务

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用语音 SDK 来识别、翻译和合成语音。

> **备注**： 可选择将该 SDK 用于 **C#** 或 **Python**。在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的 **“资源管理器”** 窗格中，浏览到 **08-speech-translation** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **translator** 文件夹，并打开集成终端。然后通过运行适用于你的语言首选项的命令，安装语音 SDK 包：

    **C#**

    ```
    dotnet add package Microsoft.CognitiveServices.Speech --version 1.14.0
    ```
    
    **Python**
    
    ```
    pip install azure-cognitiveservices-speech==1.14.0
    ```

3. 查看 **translator** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#**： appsettings.json
    - **Python**： .env

    打开配置文件并更新其包含的配置值，以包括认知服务资源的身份验证**密钥**以及部署该资源的**位置**。保存更改。
4. 请注意，**translator** 文件夹中包含客户端应用程序的代码文件：

    - **C#**： Program.cs
    - **Python**： translator.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释 **“导入命名空间”**。然后在此注释下添加以下语言特定的代码，以导入使用语音 SDK 所需的命名空间：

    **C#**
    
    ```C#
    // Import namespaces
    using Microsoft.CognitiveServices.Speech;
    using Microsoft.CognitiveServices.Speech.Audio;
    using Microsoft.CognitiveServices.Speech.Translation;
    ```
    
    **Python**
    
    ```Python
    # Import namespaces
    import azure.cognitiveservices.speech as speech_sdk
    ```

5. 请注意，**Main** 函数中已经提供了从配置文件加载认知服务密钥和区域的代码。必须使用这些变量为认知服务资源创建将用于翻译语音输入的 **SpeechTranslationConfig**。在注释 **“配置翻译”** 下添加以下代码：

    **C#**
    
    ```C#
    // Configure translation
    translationConfig = SpeechTranslationConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    translationConfig.SpeechRecognitionLanguage = "en-US";
    translationConfig.AddTargetLanguage("fr");
    translationConfig.AddTargetLanguage("es");
    translationConfig.AddTargetLanguage("hi");
    Console.WriteLine("Ready to translate from " + translationConfig.SpeechRecognitionLanguage);
    ```
    
    **Python**
    
    ```Python
    # Configure translation
    translation_config = speech_sdk.translation.SpeechTranslationConfig(cog_key, cog_region)
    translation_config.speech_recognition_language = 'en-US'
    translation_config.add_target_language('fr')
    translation_config.add_target_language('es')
    translation_config.add_target_language('hi')
    print('Ready to translate from',translation_config.speech_recognition_language)
    ```

6. 将使用 **SpeechTranslationConfig** 将语音转换为文本，但还将使用 **SpeechConfig** 将译文合成为语音。在注释 **“配置语音”** 下添加以下代码：

    **C#**
    
    ```C#
    // Configure speech
    speechConfig = SpeechConfig.FromSubscription(cogSvcKey, cogSvcRegion);
    ```
    
    **Python**
    
    ```Python
    # Configure speech
    speech_config = speech_sdk.SpeechConfig(cog_key, cog_region)
    ```

7. 保存你的更改并返回到 **translator** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

8. 如果你使用 C#，则可以忽略关于在异步方法中使用 **await** 运算符的任何警告 - 我们之后将修复此问题。代码应显示一条消息，指示已准备好从 en-US 进行翻译。按 Enter 结束程序。

## 实现语音翻译

现在，你已经为认知服务资源中的语音服务添加了 **SpeechTranslationConfig**，接下来，可以使用**语音翻译** API 来识别和翻译语音。

### 如果麦克风正常工作

1. 在程序的 **Main** 函数中，请注意代码使用 **Translate** 函数来翻译语音输入。
2. 在 **Translate** 函数的注释 **“翻译语音”** 下，添加以下代码以创建 **TranslationRecognizer** 客户端，该客户端可用于识别和翻译使用默认系统麦克风输入的语音。

    **C#**
    
    ```C#
    // Translate speech
    using AudioConfig audioConfig = AudioConfig.FromDefaultMicrophoneInput();
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Speak now...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audio_config = speech_sdk.AudioConfig(use_default_microphone=True)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Speak now...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **备注**： 应用程序中的代码在一次调用中将输入翻译为三种语言。仅显示特定语言的翻译，但你可以通过在结果的翻译集合中指定目标语言代码来检索任何**翻译**。

3. 现在，进入下面的 **“运行程序”** 部分。

### 或者，使用来自文件的音频输入

1. 在终端窗口中，输入以下命令以安装可用于播放音频文件的库：

    **C#**

    ```
    dotnet add package System.Windows.Extensions --version 4.6.0 
    ```

    **Python**

    ```
    pip install playsound==1.2.2
    ```

2. 在程序的代码文件中的现有命名空间导入下，添加以下代码以导入刚刚安装的库：

    **C#**

    ```C#
    using System.Media;
    ```

    **Python**

    ```Python
    from playsound import playsound
    ```

3. 在程序的 **Main** 函数中，请注意代码使用 **Translate** 函数来翻译语音输入。然后，在 **Translate** 函数的注释 **“翻译语音”** 下，添加以下代码以创建 **TranslationRecognizer** 客户端，该客户端可用于识别和翻译文件中的语音。

    **C#**
    
    ```C#
    // Translate speech
    string audioFile = "station.wav";
    SoundPlayer wavPlayer = new SoundPlayer(audioFile);
    wavPlayer.Play();
    using AudioConfig audioConfig = AudioConfig.FromWavFileInput(audioFile);
    using TranslationRecognizer translator = new TranslationRecognizer(translationConfig, audioConfig);
    Console.WriteLine("Getting speech from file...");
    TranslationRecognitionResult result = await translator.RecognizeOnceAsync();
    Console.WriteLine($"Translating '{result.Text}'");
    translation = result.Translations[targetLanguage];
    Console.OutputEncoding = Encoding.UTF8;
    Console.WriteLine(translation);
    ```
    
    **Python**
    
    ```Python
    # Translate speech
    audioFile = 'station.wav'
    playsound(audioFile)
    audio_config = speech_sdk.AudioConfig(filename=audioFile)
    translator = speech_sdk.translation.TranslationRecognizer(translation_config, audio_config)
    print("Getting speech from file...")
    result = translator.recognize_once_async().get()
    print('Translating "{}"'.format(result.text))
    translation = result.translations[targetLanguage]
    print(translation)
    ```

    > **备注**： 应用程序中的代码在一次调用中将输入翻译为三种语言。仅显示特定语言的翻译，但你可以通过在结果的翻译集合中指定目标语言代码来检索任何**翻译**。

### 运行程序

1. 保存你的更改并返回到 **translator** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

2. 出现提示时，输入有效的语言代码（*fr*、 *es* 或 *hi*），然后，如果使用的是麦克风，请对着麦克风清晰地说出“车站在哪里？”或出国旅行时可能使用的一些其他惯用语。程序应转录你的语音输入并将其翻译为你指定的语言（法语、西班牙语或印地语）。重复此过程，尝试使用应用程序支持的各种语言。完成后，按 Enter 结束程序。

    > **备注**： TranslationRecognizer 会为你提供大约 5 秒钟的说话时间。如果它未检测到任何语音输入，则会生成“无匹配”结果。
    >
    > 由于字符编码问题，控制台窗口中可能无法始终正确显示印地语译文。

## 将翻译合成为语音

到目前为止，应用程序已将语音输入转换为文本；如果你在旅行时需要向其他人寻求帮助，这可能已经足够。不过，最好是能够通过适当的语音将译文大声朗读出来。

1. 在 **Translate** 函数的注释 **“合成翻译”** 下，添加以下代码以使用 **SpeechSynthesizer** 客户端通过默认扬声器将翻译合成为语音：

    **C#**
    
    ```C#
    // Synthesize translation
    var voices = new Dictionary<string, string>
                    {
                        ["fr"] = "fr-FR-HenriNeural",
                        ["es"] = "es-ES-ElviraNeural",
                        ["hi"] = "hi-IN-MadhurNeural"
                    };
    speechConfig.SpeechSynthesisVoiceName = voices[targetLanguage];
    using SpeechSynthesizer speechSynthesizer = new SpeechSynthesizer(speechConfig);
    SpeechSynthesisResult speak = await speechSynthesizer.SpeakTextAsync(translation);
    if (speak.Reason != ResultReason.SynthesizingAudioCompleted)
    {
        Console.WriteLine(speak.Reason);
    }
    ```
    
    **Python**
    
    ```Python
    # Synthesize translation
    voices = {
            "fr": "fr-FR-HenriNeural",
            "es": "es-ES-ElviraNeural",
            "hi": "hi-IN-MadhurNeural"
    }
    speech_config.speech_synthesis_voice_name = voices.get(targetLanguage)
    speech_synthesizer = speech_sdk.SpeechSynthesizer(speech_config)
    speak = speech_synthesizer.speak_text_async(translation).get()
    if speak.reason != speech_sdk.ResultReason.SynthesizingAudioCompleted:
        print(speak.reason)
    ```

2. 保存你的更改并返回到 **translator** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python translator.py
    ```

3. 出现提示时，输入有效的语言代码（*fr*、 *es* 或 *hi*），然后对着麦克风清晰地说出出国旅行时可能使用的惯用语。 程序应转录你的语音输入并通过语音翻译作出响应。重复此过程，尝试使用应用程序支持的各种语言。完成后，按 Enter 结束程序。

    > **备注**： 在此示例中，你使用 **SpeechTranslationConfig** 将语音转换为了文本，然后使用 **SpeechConfig** 将翻译合成为了语音。*事实上，你可以使用 **SpeechTranslationConfig** 直接合成翻译，但这仅适用于翻译为一种语言的情况，并且会生成通常是保存为文件而不是直接发送给扬声器的音频流。*

## 更多信息

有关如何使用**语音翻译** API 的详细信息，请参阅[语音翻译文档](https://docs.microsoft.com/azure/cognitive-services/speech-service/index-speech-translation)。
