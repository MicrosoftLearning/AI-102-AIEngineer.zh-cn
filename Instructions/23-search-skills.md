---
lab:
    title: '创建 Azure 认知搜索的自定义技能'
    module: '模块 12 - 创建知识挖掘解决方案'
---

# 创建 Azure 认知搜索的自定义技能

Azure 认知搜索使用认知技能的扩充管道从文档中提取 AI 生成的字段并将这些字段包含在搜索索引中。该服务提供了一套综合性的内置技能供你使用，但如果你有这些技能无法满足的特定要求，你可以创建自定义技能。

在本次练习中，你将创建一个自定义技能，该技能可以将文档中各单词出现的频率制表以生成前五个最常用单词的列表，然后将其添加到 Margie's Travel（一家虚构的旅行社）的搜索解决方案中。

## 克隆本课程的存储库

如果已将 **AI-102-AIEngineer** 代码存储库克隆到了要完成本实验室的环境，请在 Visual Studio Code 中将其打开；否则，请按照以下步骤立即将其克隆。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行 **Git: Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**： 如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

## 创建 Azure 资源

> **备注**： 如果你之前已完成了 **[创建 Azure 认知搜索解决方案](22-azure-search.md)** 练习，并且在订阅中仍有这些 Azure 资源，则可以跳过这部分，从 **“创建搜索解决方案”** 部分开始。否则，请按照以下步骤预配所需的 Azure 资源。

1. 在 Web 浏览器中，打开 Azure 门户 `https://portal.azure.com`，并使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 查看订阅中的**资源组**。
3. 如果使用的受限订阅已提供了资源组，请选择该资源组以查看其属性。否则，请使用首选名称创建一个新的资源组，并在创建后进入该资源组。
4. 记下资源组 **“概述”** 页上的 **“订阅 ID”** 和 **“位置”**。在后续步骤中，你将需要这些值以及资源组的名称。
5. 在 Visual Studio Code 中，展开 **23-custom-search-skill** 文件夹，选择 **setup.cmd**。你将使用此批处理脚本来运行创建所需 Azure 资源时需要的 Azure 命令行接口 (CLI) 命令。
6. 右键单击 **23-custom-search-skill**文件夹，选择 **“在集成终端中打开”**。
7. 在终端窗格中，输入以下命令以与 Azure 订阅建立经认证的连接。

    ```
    az login --output none
    ```

8. 根据提示登录到 Azure 订阅。然后返回 Visual Studio Code，等待登录过程完成。
9. 运行以下命令以列出 Azure 位置。

    ```
    az account list-locations -o table
    ```

10. 在输出中，找到与资源组位置相对应的 **“名称”** 值（例如，对于*美国东部*，对应的名称是 eastus）。
11. 在 **setup.cmd** 脚本中，使用订阅 ID、资源组名称和位置名称的适当值修改 **subscription_id**、 **resource_group** 和 **location** 变量声明。然后保存更改。
12. 在 **23-custom-search-skill** 文件夹的终端中，输入以下命令运行脚本：

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
- **技能组** - 定义用于从文档中提取 AI 生成的字段的技能扩充管道。
- **索引** - 定义一组可搜索的文档记录。
- **索引器** - 从数据源中提取文档、应用技能组并填充索引。

在本次练习中，你将使用 Azure 认知搜索 REST 接口通过提交 JSON 请求来创建这些组件。

1. 在 Visual Studio Code 的 **23-custom-search-skill** 文件夹中，展开 **create-search** 文件夹，选择 **data_source.json**。该文件包含数据源 **“margies-custom-data”** 的 JSON 定义。
2. 将 **YOUR_CONNECTION_STRING** 占位符替换为 Azure 存储帐户的连接字符串，该字符串应如下所示：

    ```
    DefaultEndpointsProtocol=https;AccountName=ai102str123;AccountKey=12345abcdefg...==;EndpointSuffix=core.windows.net
    ```

    *可以在 Azure 门户中存储帐户的 **“访问密钥”** 页面上找到该连接字符串。*

3. 保存并关闭更新后的 JSON 文件。
4. 在 **create-search** 文件夹中，打开 **skillset.json**。该文件包含技能组 **“margies-custom-skillset”** 的 JSON 定义。
5. 在技能组定义顶部的 **cognitiveServices** 元素中，将 **YOUR_COGNITIVE_SERVICES_KEY** 占位符替换为认知服务资源的任何一个密钥。

    *可以在 Azure 门户中认知服务资源的 **“密钥和终结点”** 页面中找到这些密钥。*

6. 保存并关闭更新后的 JSON 文件。
7. 在 **create-search** 文件夹中，打开 **index.json**。该文件包含索引 **“margies-custom-index”** 的 JSON 定义。
8. 查看该索引的 JSON，然后关闭文件，不做任何修改。
9. 在 **create-search** 文件夹中，打开 **indexer.json**。该文件包含索引器 **“margies-custom-indexer”** 的 JSON 定义。
10. 查看该索引器的 JSON，然后关闭文件，不做任何修改。
11. 在 **create-search** 文件夹中，打开 **create-search.cmd**。此批处理脚本使用 cURL 实用程序将 JSON 定义提交到 Azure 认知搜索资源的 REST 接口。
12. 将 **YOUR_SEARCH_URL** 和 **YOUR_ADMIN_KEY** 变量占位符替换为 Azure 认知搜索资源的 **Url** 和其中一个管**理员密钥**。

    *可以在 Azure 门户中 Azure 认知搜索资源的 **“概述”** 和 **“密钥”** 页面上找到这些值。*

13. 保存更新后的批处理文件。
14. 右键单击 **create-search** 文件夹，选择 **“在集成终端中打开”**。
15. 在 **create-search** 文件夹的终端窗格中，输入以下命令运行批处理脚本。

    ```
    create-search
    ```

16. 脚本完成后，在 Azure 门户中 Azure 认知搜索资源的页面上，选择 **“索引器”** 页面，然后等待索引过程完成。

    *你可以选择 **“刷新”** 来跟踪索引操作的进度。可能需要一分钟左右的时间才能完成。*

## 搜索索引

现在你已拥有索引，可以搜索它。

1. 在 Azure 认知搜索资源边栏选项卡的顶部，选择 **“搜索资源管理器”**。
2. 在搜索资源管理器的 **“查询字符串”** 框中，输入以下查询字符串，然后选择 **“搜索”**。

    ```
    search=London&$select=url,sentiment,keyphrases&$filter=metadata_author eq 'Reviewer' and sentiment gt 0.5
    ```

    该查询可检索评论家 *Reviewer* 撰写的所有提及伦敦 *London* 时**情绪**分数大于 *0.5*（换句话说，提及伦敦的正面评论）的文档的 **url**、**情绪**和**关键词**

## 为自定义技能创建 Azure 函数

搜索解决方案包括一些内置的认知技能，这些技能可以使用文档中的信息（例如在之前的任务中看到的情绪分数和关键短语列表）来扩充索引。

你可以通过创建自定义技能来进一步增强索引。例如，识别每个文档中使用频率最高的单词可能很有用，但无内置技能可提供此功能。

要以自定义技能的形式实现字数统计功能，你需要使用首选语言创建一个 Azure 函数。

1. 在 Visual Studio Code 中，查看 Azure 扩展选项卡 (**&boxplus;**)，并验证 **Azure Functions** 扩展是否已安装。利用此扩展，你可以在 Visual Studio Code 中创建和部署 Azure Functions。
2. 在 Azure 选项卡 (**&Delta;**) 上的 **Azure Functions** 窗格中，创建一个新项目 (&#128194;)，并根据首选语言进行以下设置：

    ### **C#**

    - **文件夹**： 浏览至 **23-custom-search-skill/C-Sharp/wordcount**
    - **语言**： C#
    - **模板**： HTTP 触发器
    - **函数名称**： wordcount
    - **命名空间**： margies.search
    - **授权级别**： 函数

    ### **Python**

    - **文件夹**： 浏览至 **23-custom-search-skill/Python/wordcount**
    - **语言**： Python
    - **虚拟环境**： 跳过虚拟环境
    - **模板**： HTTP 触发器
    - **函数名称**： wordcount
    - **授权级别**： 函数

    *如果系统提示覆盖 **launch.json**，请执行此操作！*

3. 切换回资源**管理器** (**&#128461;**) 选项卡，验证 **wordcount** 文件夹现在是否包含 Azure 函数的代码文件。

    *如果选择 Python，代码文件可能会在子文件夹中，名称也为 **wordcount***

4. 函数的主代码文件应该已自动打开。如果没有，请为所选语言打开相应文件：
    - **C#**： wordcount.cs
    - **Python**： \_\_init\_\_&#46;py

5. 将文件的全部内容替换为以下所选语言的代码：

### **C#**

```C#
using System.IO;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.AspNetCore.Http;
using Newtonsoft.Json;
using System.Collections.Generic;
using Microsoft.Extensions.Logging;
using System.Text.RegularExpressions;
using System.Linq;

namespace margies.search
{
    public static class wordcount
    {

        //定义响应的类
        private class WebApiResponseError
        {
            public string message { get; set; }
        }

        private class WebApiResponseWarning
        {
            public string message { get; set; }
        }

        private class WebApiResponseRecord
        {
            public string recordId { get; set; }
            public Dictionary<string, object> data { get; set; }
            public List<WebApiResponseError> errors { get; set; }
            public List<WebApiResponseWarning> warnings { get; set; }
        }

        private class WebApiEnricherResponse
        {
            public List<WebApiResponseRecord> values { get; set; }
        }

        //自定义技能的函数
        [FunctionName("wordcount")]
        public static IActionResult Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)]HttpRequest req, ILogger log)
        {
            log.LogInformation("Function initiated.");

            string recordId = null;
            string originalText = null;

            string requestBody = new StreamReader(req.Body).ReadToEnd();
            dynamic data = JsonConvert.DeserializeObject(requestBody);

            // 验证
            if (data?.values == null)
            {
                return new BadRequestObjectResult(" Could not find values array");
            }
            if (data?.values.HasValues == false || data?.values.First.HasValues == false)
            {
                return new BadRequestObjectResult("Could not find valid records in values array");
            }

            WebApiEnricherResponse response = new WebApiEnricherResponse();
            response.values = new List<WebApiResponseRecord>();
            foreach (var record in data?.values)
            {
                recordId = record.recordId?.Value as string;
                originalText = record.data?.text?.Value as string;

                if (recordId == null)
                {
                    return new BadRequestObjectResult("recordId cannot be null");
                }

                // 汇集响应。
                WebApiResponseRecord responseRecord = new WebApiResponseRecord();
                responseRecord.data = new Dictionary<string, object>();
                responseRecord.recordId = recordId;
                responseRecord.data.Add("text", Count(originalText));

                response.values.Add(responseRecord);
            }

            return (ActionResult)new OkObjectResult(response); 
        }


            public static string RemoveHtmlTags(string html)
        {
            string htmlRemoved = Regex.Replace(html, @"<script[^>]*>[\s\S]*?</script>|<[^>]+>| ", " ").Trim();
            string normalised = Regex.Replace(htmlRemoved, @"\s{2,}", " ");
            return normalised;
        }

        public static List<string> Count(string text)
        {
            
            //删除 html 元素
            text=text.ToLowerInvariant();
            string html = RemoveHtmlTags(text);
            
            //拆分为单词列表
            List<string> list = html.Split(" ").ToList();
            
            //删除任何非子母字符
            var onlyAlphabetRegEx = new Regex(@"^[A-z]+$");
            list = list.Where(f => onlyAlphabetRegEx.IsMatch(f)).ToList();

            //删除非索引字
            string[] stopwords = { "", "i", "me", "my", "myself", "we", "our", "ours", "ourselves", "you", 
                    "you're", "you've", "you'll", "you'd", "your", "yours", "yourself", 
                    "yourselves", "he", "him", "his", "himself", "she", "she's", "her", 
                    "hers", "herself", "it", "it's", "its", "itself", "they", "them", 
                    "their", "theirs", "themselves", "what", "which", "who", "whom", 
                    "this", "that", "that'll", "these", "those", "am", "is", "are", "was",
                    "were", "be", "been", "being", "have", "has", "had", "having", "do", 
                    "does", "did", "doing", "a", "an", "the", "and", "but", "if", "or", 
                    "because", "as", "until", "while", "of", "at", "by", "for", "with", 
                    "about", "against", "between", "into", "through", "during", "before", 
                    "after", "above", "below", "to", "from", "up", "down", "in", "out", 
                    "on", "off", "over", "under", "again", "further", "then", "once", "here", 
                    "there", "when", "where", "why", "how", "all", "any", "both", "each", 
                    "few", "more", "most", "other", "some", "such", "no", "nor", "not", 
                    "only", "own", "same", "so", "than", "too", "very", "s", "t", "can", 
                    "will", "just", "don", "don't", "should", "should've", "now", "d", "ll",
                    "m", "o", "re", "ve", "y", "ain", "aren", "aren't", "couldn", "couldn't", 
                    "didn", "didn't", "doesn", "doesn't", "hadn", "hadn't", "hasn", "hasn't", 
                    "haven", "haven't", "isn", "isn't", "ma", "mightn", "mightn't", "mustn", 
                    "mustn't", "needn", "needn't", "shan", "shan't", "shouldn", "shouldn't", "wasn", 
                    "wasn't", "weren", "weren't", "won", "won't", "wouldn", "wouldn't"}; 
            list = list.Where(x => x.Length > 2).Where(x => !stopwords.Contains(x)).ToList();
            
            //按键和计数获取不同单词，然后按计数排序
            var keywords = list.GroupBy(x => x).OrderByDescending(x => x.Count());
            var klist = keywords.ToList();

            // 返回前 10 个单词
            var numofWords = 10;
            if(klist.Count<10)
                numofWords=klist.Count;
            List<string> resList = new List<string>();
            for (int i = 0; i < numofWords; i++)
            {
                resList.Add(klist[i].Key);
            }
            return resList;
        }
    }
}
```

## **Python**

```Python
import logging
import os
import sys
import json
from string import punctuation
from collections import Counter
import azure.functions as func


def main(req: func.HttpRequest) -> func.HttpResponse:
    logging.info('Wordcount function initiated.')

    # 结果将为“values”口袋
    result = {
        "values": []
    }
    statuscode = 200

    # 我们将在单词计数中排除此列表中的单词
    stopwords = ['', 'i', 'me', 'my', 'myself', 'we', 'our', 'ours', 'ourselves', 'you', 
                "you're", "you've", "you'll", "you'd", 'your', 'yours', 'yourself', 
                'yourselves', 'he', 'him', 'his', 'himself', 'she', "she's", 'her', 
                'hers', 'herself', 'it', "it's", 'its', 'itself', 'they', 'them', 
                'their', 'theirs', 'themselves', 'what', 'which', 'who', 'whom', 
                'this', 'that', "that'll", 'these', 'those', 'am', 'is', 'are', 'was',
                'were', 'be', 'been', 'being', 'have', 'has', 'had', 'having', 'do', 
                'does', 'did', 'doing', 'a', 'an', 'the', 'and', 'but', 'if', 'or', 
                'because', 'as', 'until', 'while', 'of', 'at', 'by', 'for', 'with', 
                'about', 'against', 'between', 'into', 'through', 'during', 'before', 
                'after', 'above', 'below', 'to', 'from', 'up', 'down', 'in', 'out', 
                'on', 'off', 'over', 'under', 'again', 'further', 'then', 'once', 'here', 
                'there', 'when', 'where', 'why', 'how', 'all', 'any', 'both', 'each', 
                'few', 'more', 'most', 'other', 'some', 'such', 'no', 'nor', 'not', 
                'only', 'own', 'same', 'so', 'than', 'too', 'very', 's', 't', 'can', 
                'will', 'just', 'don', "don't", 'should', "should've", 'now', 'd', 'll',
                'm', 'o', 're', 've', 'y', 'ain', 'aren', "aren't", 'couldn', "couldn't", 
                'didn', "didn't", 'doesn', "doesn't", 'hadn', "hadn't", 'hasn', "hasn't", 
                'haven', "haven't", 'isn', "isn't", 'ma', 'mightn', "mightn't", 'mustn', 
                "mustn't", 'needn', "needn't", 'shan', "shan't", 'shouldn', "shouldn't", 'wasn', 
                "wasn't", 'weren', "weren't", 'won', "won't", 'wouldn', "wouldn't"]

    try:
        values = req.get_json().get('values')
        logging.info(values)

        for rec in values:
            # 构造此记录的基本 JSON 响应
            val = {
                    "recordId": rec['recordId'],
                    "data": {
                        "text":None
                    },
                    "errors": None,
                    "warnings": None
                }
            try:
                # 从输入记录中获取要处理的文本
                txt = rec['data']['text']
                # 删除数字
                txt = ''.join(c for c in txt if not c.isdigit())
                # 删除标点并改为小写
                txt = ''.join(c for c in txt if c not in punctuation).lower()
                # 删除非索引字
                txt = ' '.join(w for w in txt.split() if w not in stopwords)
                # 计算单词数量并获取最常见的 10 个单词
                wordcount = Counter(txt.split()).most_common(10)
                words = [w[0] for w in wordcount]
                # 将最常见的 10 个单词添加到此文本记录的输出中
                val["data"]["text"] = words
            except:
                # 这条文本记录发生了错误，因此请添加错误和警告列表
                val["errors"] =[{"message": "处理文本时发生了错误。"}]
                val["warnings"] = [{"message": "一个或多个输入无法处理。"}]
            finally:
                # 将此记录的值添加到响应
                result["values"].append(val)
    except Exception as ex:
        statuscode = 500
        # 发生了全局错误，因此返回错误响应
        val = {
                "recordId": None,
                "data": {
                    "text":None
                },
                "errors": [{"message": ex.args}],
                "warnings": [{"message": "The request failed to process."}]
            }
        result["values"].append(val)
    finally:
        # 返回响应
        return func.HttpResponse(body=json.dumps(result), mimetype="application/json", status_code=statuscode)
```
    
6. 保存更新的文件。
7. 右键单击包含代码文件的 **wordcount** 文件夹，选择 **“部署到函数应用”**。然后使用以下特定于语言的设置部署该函数（如有提示，请登录 Azure）：

    ### **C#**

    - **订阅** （如有提示）：选择 Azure 订阅。
    - **函数**： 在 Azure 中新建函数应用（高级）
    - **函数应用名称**： 输入全局唯一名称。
    - **运行时**： .NET Core 3.1
    - **操作系统**： Linux
    - **托管计划**： 使用
    - **资源组**： 包含 Azure 认知搜索资源的资源组。
        - 备注：如果此资源组已经包含一个基于 Windows 的 Web 应用，你将无法在其中部署一个基于 Linux 的函数。要么删除现有 Web 应用，要么将该函数部署到其他资源组。
    - **存储帐户**： 存储 Margie's Travel 文档的存储帐户。
    - **Application Insights**： 暂时跳过

    *Visual Studio Code 将根据创建函数项目时保存的 **.vscode** 文件夹中的配置设置，（在 **bin** 子文件夹中）部署函数的编译版本。*

    ### **Python**

    - **订阅** （如有提示）：选择 Azure 订阅。
    - **函数**： 在 Azure 中新建函数应用（高级）
    - **函数应用名称**： 输入全局唯一名称。
    - **运行时**： Python 3.8
    - **托管计划**： 使用
    - **资源组**： 包含 Azure 认知搜索资源的资源组。
        - 备注：如果此资源组已经包含一个基于 Windows 的 Web 应用，你将无法在其中部署一个基于 Linux 的函数。要么删除现有 Web 应用，要么将该函数部署到其他资源组。
    - **存储帐户**： 存储 Margie's Travel 文档的存储帐户。
    - **Application Insights**： 暂时跳过

8. 等待 Visual Studio Code 部署函数。部署完成后，将显示一条通知。

## 测试函数

将函数部署到 Azure 后，即可在 Azure 门户中进行测试。

1. 打开 [Azure 门户](https://portal.azure.com)，浏览到在其中创建函数应用的资源组。然后打开函数应用的应用服务。
2. 在应用服务的边栏选项卡的 **“函数”** 页面上，打开 **wordcount** 函数。
3. 在 **wordcount** 函数边栏选项卡上，查看 **“代码 + 测试”** 页面并打开 **“测试/运行”** 窗格。
4. 在 **“测试/运行”** 窗格中，将现有的正文替换为以下 **Body**，该 JSON 反映了 Azure 认知搜索技能期望的模式，即提交包含一个或多个文档数据的记录进行处理：

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
    
5. 单击 **“运行”** 并查看函数返回的 HTTP 响应内容。这反映了 Azure 认知搜索在使用技能时期望的模式，即返回每个文档的响应。在本例中，响应最多包含各文档中的 10 个术语，并按其出现频率降序排列：

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

6. 关闭 **“测试/运行”** 窗格，在 **wordcount** 函数边栏选项卡中，单击 **“获取函数 URL”**。然后将默认键的 URL 复制到剪贴板。下一过程将需要此 URL。

## 将自定义技能添加到搜索解决方案

现在需要将函数作为自定义技能包含在搜索解决方案技能组中，并将其生成的结果映射到索引中的字段。 

1. 在 Visual Studio Code 的 **23-custom-search-skill/update-search** 文件夹中，打开 **update-skillset.json** 文件。这包含技能组的 JSON 定义。
2. 查看技能组定义。它包括与之前相同的技能，以及一个名为 **get-top-words** 的新 **WebApiSkill** 技能。
3. 编辑 **get-top-words** 技能定义，将 **uri** 值设置为 Azure 函数的 URL（在之前的过程中已复制到剪贴板），从而替换 **YOUR-FUNCTION-APP-URL**。
4. 在技能组定义顶部的 **cognitiveServices** 元素中，将 **YOUR_COGNITIVE_SERVICES_KEY**占位符替换为认知服务资源的任何一个密钥。

    *可以在 Azure 门户中认知服务资源的 **“密钥和终结点”** 页面中找到这些密钥。*

5. 保存并关闭更新后的 JSON 文件。
6. 在 **update-search** 文件夹中，打开 **update-index.json**。此文件包含 **margies-custom-index** 索引的 JSON 定义，并在索引定义的底部有一个名为 **top_words** 的附加字段。
7. 查看该索引的 JSON，然后关闭文件，不做任何修改。
8. 在 **update-search** 文件夹中，打开 **update-indexer.json**。此文件包含 **margies-custom-indexer** 的 JSON 定义，并具有 **top_words** 字段的附加映射。
9. 查看该索引器的 JSON，然后关闭文件，不做任何修改。
10. 在 **update-search** 文件夹中，打开 **update-search.cmd**。此批处理脚本使用 cURL 实用程序将更新后的 JSON 定义提交到 Azure 认知搜索资源的 REST 接口。
11. 将 **YOUR_SEARCH_URL** 和 **YOUR_ADMIN_KEY** 变量占位符替换为 Azure 认知搜索资源的 **Url** 和其中一个管**理员密钥**。

    *可以在 Azure 门户中 Azure 认知搜索资源的 **“概述”** 和 **“密钥”** 页面上找到这些值。*

12. 保存更新后的批处理文件。
13. 右键单击 **update-search** 文件夹，选择 **“在集成终端中打开”**。
14. 在 **update-search**文件夹的终端窗格中，输入以下命令运行批处理脚本。

    ```
    update-search
    ```

15. 脚本完成后，在 Azure 门户中 Azure 认知搜索资源的页面上，选择 **“索引器”** 页面，然后等待索引过程完成。

    *你可以选择 **“刷新”** 来跟踪索引操作的进度。可能需要一分钟左右的时间才能完成。*

## 搜索索引

现在你已拥有索引，可以搜索它。

1. 在 Azure 认知搜索资源边栏选项卡的顶部，选择 **“搜索资源管理器”**。
2. 在搜索资源管理器的 **“查询字符串”** 框中，输入以下查询字符串，然后选择 **“搜索”**。

    ```
    search=Las Vegas&$select=url,top_words
    ```

    此查询检索提到拉斯维加斯 **Las Vegas** 的所有文档的 **url** 和 **top_words** 字段。

## 更多信息

要了解有关为 Azure 认知搜索创建自定义技能的更多信息，请参阅 [Azure 认知搜索文档](https://docs.microsoft.com/azure/search/cognitive-search-custom-skill-interface)。
