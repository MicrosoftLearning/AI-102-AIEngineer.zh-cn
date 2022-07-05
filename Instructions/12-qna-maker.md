---
lab:
  title: 创建问题解答解决方案
  module: Module 6 - Building a QnA Solution
ms.openlocfilehash: 42b3b6e6955188d3d442a4f587ead6afe0f389c6
ms.sourcegitcommit: a879eae298aaf2ab6415fc4fdb67f52f592d907a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/02/2022
ms.locfileid: "146902477"
---
# <a name="create-a-question-answering-solution"></a>创建问题解答解决方案

最常见的对话方案之一是通过常见问题解答 (FAQ) 知识库提供支持。 许多组织将 FAQ 发布为文档或网页，这适用于少量的问答对，但针对大型文档搜索可能很麻烦且十分耗时。

语言服务包含一项问题解答功能，通过该功能，你能够创建可使用自然语言输入来查询的问答对知识库，该功能通常用作一种资源，机器人可以用它来查找用户提交的问题的答案。

> **注意**：语言服务中的问题解答功能是 QnA Maker 服务的较新版本，该服务仍作为单独的服务提供。

## <a name="clone-the-repository-for-this-course"></a>克隆本课程的存储库

如果尚未将 AI-102-AIEngineer 代码存储库克隆到你要在此实验室中使用的环境，请按照以下步骤克隆它。 否则，请在 Visual Studio Code 中打开克隆的文件夹。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“Git: 克隆”命令，以将 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择“以后再说”。

## <a name="create-a-language-resource"></a>创建“语言”资源

若要创建和托管问题解答知识库，Azure 订阅中需要包含语言服务。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择“&#65291;创建资源”按钮，搜索“语言”，并创建“语言服务”资源。
3. 单击“自定义问答”块上的“选择” 。 然后单击“继续创建资源”。 你需要输入以下设置：
    
    - **订阅**：Azure 订阅
    - 资源组：选择或创建一个资源组（如果你使用的是受限订阅，则可能无权创建新资源组，在此情况下，可使用一个已提供的资源组）
    - 区域：选择任何可用位置
    - **名称**：输入唯一名称
    - **定价层**：标准版 S
    - **Azure 搜索位置**\*：在与语言资源相同的全球区域中选择一个位置。
    - **Azure 搜索定价层**：免费 (F)（如果该层级不可用，请选择基本 (B)）
    - **法律条款**：同意 
    - **负责任的 AI 声明**：同意
    
    \*自定义问题解答使用 Azure 搜索来对问答知识库编制索引和进行查询。

4. 等待部署完成，然后查看部署详细信息。

## <a name="create-a-question-answering-project"></a>创建问题解答项目

若要在语言资中创建问题解答知识库，可以使用 Language Studio 门户创建问题解答项目。 在本例中，你将创建包含有关 [Microsoft Learn](https://docs.microsoft.com/learn) 的问答知识库。

1. 在新的浏览器标签页中，转到 Language Studio 门户 (`https://language.azure.com`)，然后使用与 Azure 订阅关联的 Microsoft 帐户登录。
2. 如果系统提示选择语言资源，请选择以下设置：
    - **Azure Directory**：包含订阅的 Azure 目录。
    - **订阅**：Azure 订阅。
    - **语言资源**：先前创建的语言资源。
3. 如果系统未提示你选择语言资源，原因可能是订阅中有多个语言资源；在这种情况下，你应该：
    1. 在页面顶部的栏中，单击“设置(&#9881;)”按钮。
    2. 在“设置”页上，查看“资源”选项卡。
    3. 选择刚刚创建的语言资源，然后单击“切换资源”。
    4. 在页面顶部，单击“Language Studio”以返回到 Language Studio 主页。
3. 在门户顶部的“新建”菜单中，选择“自定义问题解答” 。
4. 在*“创建项目”向导中，在“选择语言设置”页上选择用于设置此资源中所有项目的语言的选项，然后选择“英语”作为语言  。 然后单击“下一步”。
5. 在“输入基本信息”页上，输入以下详细信息，然后单击“下一步” ：
    - **名称** LearnFAQ
    - **说明**：Microsoft Learn 常见问题解答
    - **未返回任何答案时的默认答案**：抱歉，我不明白这个问题
6. 在“检查并完成”页上，单击“创建项目”。

## <a name="add-a-sources-to-the-knowledge-base"></a>将源添加到知识库

可以从头开始创建知识库，但通常先从现有的 FAQ 页面或文档中导入问答。 在本例中，你将从 Microsoft Learn 的现有常见问题解答网页导入数据，并且还将导入一些预定义的“聊天”问题和答案，以支持常见的对话交流。

1. 在问题解答项目的“管理源”页上，在"&#9547; 添加源"列表中，选择“URL”  。 然后在“添加 URL”对话框中，单击“&#9547; 添加 url”并添加以下 URL，然后单击“全部添加”以将其添加到知识库  ：
    - **名称**：`Learn FAQ Page`
    - **URL**：`https://docs.microsoft.com/en-us/learn/support/faq`
2. 在问题解答项目的“管理源”页上，在“&#9547; 添加源”列表中，选择“聊天”  。 在“添加聊天”对话框中，选择“友好”，然后单击“添加聊天”  。

## <a name="edit-the-knowledge-base"></a>编辑知识库

你的知识库已填充了 Microsoft Learn FAQ 中的问答对，并补充了一组对话式聊天问答对。 可以通过添加更多问答对来扩展知识库。

1. 在 Language Studio 中的 LearnFAQ 项目中，选择“编辑知识库”页以查看现有问答对（如果显示了一些提示，请阅读提示，然后单击“知道了”将其消除，或单击“跳过所有提示”   ）
2. 在知识库中，选择“&#65291; 添加问题集”。
3. 在“问题”框中，键入 `What is Microsoft certification?` 并按 Enter******。
4. 选择“&#65291; 添加替代表述”，键入 `How can I demonstrate my Microsoft technology skills?` 并按 Enter 。
5. 在“答案”框中，键入 `The Microsoft Certified Professional program enables you to validate and prove your skills with Microsoft technologies.`，然后按“提交”以将问题（包括替代表述）和答案提交至知识库 。

    在某些情况下，让用户能够跟进答案以创建多回合对话是有意义的，这让用户能够以迭代方式细化问题，从而得到他们所需的答案。

6. 在你输入的关于认证问题的答案下，选择“&#65291; 添加跟进提示”。
7. 在“跟进提示”对话框中，输入以下设置，然后单击“添加提示” ：
    - 在提示中向用户显示的文本：`Learn more about certification`。
    - 选择“创建指向新对的链接”，然后输入以下文本：`You can learn more about certification on the [Microsoft certification page](https://docs.microsoft.com/learn/certifications/).`
    - 仅在上下文流中显示：选定。 此选项可确保仅在原始认证问题的跟进问题的上下文中返回答案。

## <a name="train-and-test-the-knowledge-base"></a>训练和测试知识库

现在你已有了知识库，接下来可以在 Language Studio 中对它进行测试。

1. 在页面右上角单击“保存更改”。
2. 保存更改后，单击“测试”以打开测试窗格。
3. 在测试窗格的顶部，取消选择用于显示简短答案的选项。 然后在底部输入消息 `Hello`。 应会返回相应的响应。
4. 在测试窗格的底部，输入消息 `What is Microsoft Learn?`。 应会返回来自常见问题解答的适当答复。
5. 输入消息 `Thanks!` 应返回相应的聊天响应。
6. 输入消息 `Tell me about certification`。 创建的答案应与跟进提示链接一起返回。
7. 选择“详细了解认证”跟进提示链接。 应会返回带有指向认证页面的链接的跟进答案。
8. 完成知识库测试后，关闭测试窗格。

## <a name="deploy-and-test-the-knowledge-base"></a>部署和测试知识库

知识库提供了客户端应用程序可以用来回答问题的后端服务。 现在，可以准备发布知识库并从客户端访问其 REST 接口了。

1. 在 Language Studio 的 LearnFAQ 项目中，选择“部署知识库”页 。
2. 在页面顶部，单击“部署”。 然后单击“部署”以确认要部署知识库。
3. 部署完成后，单击“获取预测 URL”以查看知识库的 REST 终结点，并将其复制到剪贴板（但不要关闭对话框）。
4. 在 Visual Studio Code 中，在 12-qna 文件夹中打开 ask-question.cmd 。 此脚本使用 Curl 调用问题解答终结点的 REST 接口。
5. 在脚本中，将 YOUR_PREDICTION_ENDPOINT 替换为复制的预测终结点（确保它是用括号括起来的）。
6. 返回到浏览器，在“获取预测 URL”对话框中，请注意，示例请求包含 Ocp-Apim-Subscription-Key 参数的值，该值与 ab12c345de678fg9hijk01lmno2pqrs34 类似 。 这是资源的授权密钥。 将它复制到剪贴板，然后单击“知道了”以关闭对话框。
7. 返回到 Visual Studio Code，在 ask-question.cmd 脚本中，将 YOUR_KEY 替换为复制的密钥（确保它是用括号括起来的）。
8. 请注意，脚本中的 Curl 命令会提交一个问题参数，其值为“What is a Learning Path?” 。
9. 验证整个脚本是否类似于以下代码，然后保存文件。

    ```
    @echo off
    SETLOCAL ENABLEDELAYEDEXPANSION

    rem Set variables
    set prediction_url="https://some-name.cognitiveservices.azure.com/language/......."
    set key="ab12c345de678fg9hijk01lmno2pqrs34"

    curl -X POST !prediction_url! -H "Ocp-Apim-Subscription-Key: !key!" -H "Content-Type: application/json" -d "{'question': 'What is a learning Path?' }"
    ```

10. 在 Visual Studio Code 中，在“Explorer”窗格中右键单击“ask-question.cmd”脚本，并选择“在集成终端中打开” 。
11. 在终端窗格中，输入命令 `ask-question.cmd` 以运行脚本并查看服务返回的 JSON 响应，其中应包含针对问题“What is a learning path?”的相应答案。

## <a name="create-a-bot-for-the-knowledge-base"></a>为知识库创建机器人

用于从知识库中检索答案的客户端应用程序通常都是机器人。

1. 在浏览器中返回到 Language Studio，在“部署知识库”页中，选择“创建机器人” 。 这将在新的浏览器标签页中打开 Azure 门户，以便你可以在 Azure 订阅中创建 Web 应用机器人（出现提示时，请登录）。
2. 在 Azure 门户中，使用以下设置创建一个 Web 应用机器人（其中大部分设置将预先填充）：

    如果缺少某些值，请刷新浏览器。  

  - **机器人句柄**：机器人的唯一名称
  - **订阅**：Azure 订阅
  - 资源组：包含语言资源的资源组
  - 位置：与文本分析服务相同的位置。
  - **定价层**：F0
  - **应用名称**：*与具有唯一 ID并自动追加“.azurewebsites.net”的“机器人句柄”相同
  - **SDK 语言**：选择 C# 或 Node.js
  - **QnA 身份验证密钥**：此项应自动设置为 QnA 知识库的身份验证密钥
  - **应用服务计划/位置**：如果存在，可以自动将其设置为合适的计划和位置。如果不存在，请创建一个新计划
  - **Application Insights**：关
  - **Microsoft 应用 ID 和密码**：自动创建应用 ID 和密码。
3. 等待机器人创建完成。 然后单击“转到资源”（或者，在主页上单击“资源组”，打开在其中创建了 Web 应用机器人的资源组并单击它。 ）
4. 在机器人的边栏选项卡中，查看“在 Web 聊天中测试”页，然后一直等待到机器人显示消息“Hello and welcome!” （初始化过程可能需要几秒钟时间）。
5. 使用测试聊天界面确保机器人按照预期回答知识库中的问题。 例如，尝试提交 `What is Microsoft certification?`。

## <a name="more-information"></a>详细信息

若要详细了解语言服务中的问题解答，请参阅[语言服务文档](https://docs.microsoft.com/en-us/azure/cognitive-services/language-service/question-answering/overview)。
