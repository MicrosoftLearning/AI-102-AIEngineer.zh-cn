---
lab:
    title: '翻译文本'
    module: '模块 3 - 自然语言处理入门指南'
---

# 翻译文本

**翻译**服务是一种认知服务，使你能够在不同语言之间翻译文本。

例如，假设一家旅行社想要检查已提交到公司网站的酒店评论，将英语标准化为用于分析的语言。通过使用翻译服务，他们可以确定每条评论所采用的语言，如果采用的并非英语，则将其从任何源语言翻译为英语。

## 克隆本课程的存储库

如果已将 **AI-102-AIEngineer** 代码存储库克隆到了要完成本实验室的环境，请在 Visual Studio Code 中将其打开；否则，请按照以下步骤立即将其克隆。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行 **Git: Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**： 如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

## 预配认知服务资源

如果你的订阅中还没有**认知服务**资源，需要预配一个。

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

## 准备使用翻译服务

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用翻译 REST API 来翻译酒店评论。

> **备注**： 可以选择在 **C#** 或 **Python** 中使用该 API。在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的 **“资源管理器”** 窗格中，浏览到 **06-translate-text** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 查看 **text-translation** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#**： appsettings.json
    - **Python**： .env

    打开配置文件并更新其包含的配置值，以添加认知服务资源的身份验证**密钥**以及部署该资源的**位置**（<u>不是</u>终结点）- 你应从认知服务资源的 **“密钥和终结点”** 页面复制这两项。保存更改。
3. 请注意，**text-translation** 文件夹中包含客户端应用程序的代码文件：

    - **C#**： Program.cs
    - **Python**： text-translation.py

    打开代码文件并检查其中包含的代码。

4. 请注意，**Main** 函数中已经提供了从配置文件加载认知服务密钥和区域的代码。代码中还指定了翻译服务的终结点。
5. 右键单击 **text-translation** 文件夹，打开集成终端，并输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

6. 观察输出，因为代码应正常运行且不出现错误，并在 **“reviews”** 文件夹中显示每个评论文本文件的内容。应用程序当前未使用翻译服务。我们将在下一过程中修复此问题。

## 检测语言

翻译服务可以自动检测要翻译的文本的源语言，但也支持显式检测撰写文本所用的语言。

1. 在代码文件中，找到 **GetLanguage** 函数，该函数当前针对所有文本值返回 “en”。
2. 在 **GetLanguage** 函数的注释 **“使用 Translator detect 函数”** 下，添加以下代码以使用翻译的 REST API 来检测指定文本的语言，注意不要替换返回语言的函数末尾的代码：

**C#**

```C#
// 使用 Translator detect 函数
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // 生成请求
        string path = "/detect?api-version=3.0";
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // 发送请求并获取响应
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // 以字符串形式读取响应
        string responseContent = await response.Content.ReadAsStringAsync();

        // 分析 JSON 数组并获取语言
        JArray jsonResponse = JArray.Parse(responseContent);
        language = (string)jsonResponse[0]["language"]; 
    }
}
```

**Python**

```Python
# 使用 Translator detect 函数
path = '/detect'
url = translator_endpoint + path

# 生成请求
params = {
    'api-version': '3.0'
}

headers = {
'Ocp-Apim-Subscription-Key': cog_key,
'Ocp-Apim-Subscription-Region': cog_region,
'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# 发送请求并获取响应
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# 分析 JSON 数组并获取语言
language = response[0]["language"]
```

3. 保存你的更改并返回到 **text-translation** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. 观察输出，注意此次已确定每条评论的语言。

## 翻译文本

在应用程序可以确定评论所采用的语言之后，现在，你可以使用翻译服务将任何非英语评论翻译为英语。

1. 在代码文件中，找到 **Translate** 函数，该函数当前针对所有文本值返回空字符串。
2. 在 **Translate** 函数的注释 **“使用 Translator translate 函数”** 下，添加以下代码以使用翻译的 REST API 将指定文本从其源语言翻译为英语，注意不要替换返回译文的函数末尾的代码：

**C#**

```C#
// 使用 Translator translate 函数
object[] body = new object[] { new { Text = text } };
var requestBody = JsonConvert.SerializeObject(body);
using (var client = new HttpClient())
{
    using (var request = new HttpRequestMessage())
    {
        // 生成请求
        string path = "/translate?api-version=3.0&from=" + sourceLanguage + "&to=en" ;
        request.Method = HttpMethod.Post;
        request.RequestUri = new Uri(translatorEndpoint + path);
        request.Content = new StringContent(requestBody, Encoding.UTF8, "application/json");
        request.Headers.Add("Ocp-Apim-Subscription-Key", cogSvcKey);
        request.Headers.Add("Ocp-Apim-Subscription-Region", cogSvcRegion);

        // 发送请求并获取响应
        HttpResponseMessage response = await client.SendAsync(request).ConfigureAwait(false);
        // 以字符串形式读取响应
        string responseContent = await response.Content.ReadAsStringAsync();

        // 分析 JSON 数组并获取翻译
        JArray jsonResponse = JArray.Parse(responseContent);
        translation = (string)jsonResponse[0]["translations"][0]["text"];  
    }
}
```

**Python**

```Python
# 使用 Translator translate 函数
path = '/translate'
url = translator_endpoint + path

# 生成请求
params = {
    'api-version': '3.0',
    'from': source_language,
    'to': ['en']
}

headers = {
    'Ocp-Apim-Subscription-Key': cog_key,
    'Ocp-Apim-Subscription-Region': cog_region,
    'Content-type': 'application/json'
}

body = [{
    'text': text
}]

# 发送请求并获取响应
request = requests.post(url, params=params, headers=headers, json=body)
response = request.json()

# 分析 JSON 数组并获取翻译
translation = response[0]["translations"][0]["text"]
```

3. 保存你的更改并返回到 **text-translation** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python text-translation.py
    ```

4. 观察输出，注意非英语评论已翻译为英语。

## 更多信息

如需详细了解如何使用**翻译**服务，请参阅[翻译文档](https://docs.microsoft.com/azure/cognitive-services/translator/)。
