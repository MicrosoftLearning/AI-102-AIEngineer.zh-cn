---
lab:
    title: '使用 Bot Framework SDK 创建机器人'
    module: '模块 7 - 对话式 AI 和 Azure 机器人服务'
---

# 使用 Bot Framework SDK 创建机器人

*机器*人是一个软件代理，可参与人类用户的对话式对话。Microsoft Bot Framework 为构建可通过 Azure 机器人服务作为云服务交付的机器人提供了一个综合平台。

在本练习中，你将使用 Microsoft Bot Framework SDK 创建和部署机器人。

## 准备工作

首先，为机器人开发准备环境。

### 更新 Bot Framework Emulator

你将使用 Bot Framework SDK 来创建机器人，并使用 Bot Framework Emulator 对其进行测试。Bot Framework Emulator 会定期更新，因此请确保已安装最新版本。

> **备注**：更新可能包括更改用户界面，这可能会影响此练习中的说明。

1. 启动 **Bot Framework Emulator**，如果系统提示你安装更新，请为当前登录的用户完成该操作。如果系统没有自动发出提示，请使用 **“帮助”** 菜单上的 **“检查更新”** 选项来检查更新。
2. 安装任意可用更新后，关闭 Bot Framework Emulator，直到下次需要它时再打开。

### 克隆本课程的存储库

如果尚未将 **AI-102-AIEngineer** 代码存储库克隆到你要在此实验室中使用的环境，请按照以下步骤克隆它。否则，请在 Visual Studio Code 中打开克隆的文件夹。

1. 启动 Visual Studio Code。
2. 打开面板 (Shift+Ctrl+P) 并运行 **Git: Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**：如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

## 创建机器人

你可以使用 Bot Framework SDK 基于模板创建机器人，然后自定义代码以满足你的特定要求。

> **备注**：在此练习中，可以选择使用 **C#** 或 **Python**。在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的 **“资源管理器”** 窗格中，浏览到 **13-bot-framework** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击所选语言的文件夹，并打开集成终端。
3. 在终端中，运行以下命令以安装所需的机器人模板和包：

**C#**

```C#
dotnet new -i Microsoft.Bot.Framework.CSharp.EchoBot
dotnet new -i Microsoft.Bot.Framework.CSharp.CoreBot
dotnet new -i Microsoft.Bot.Framework.CSharp.EmptyBot
```

**Python**

```Python
pip install botbuilder-core
pip install asyncio
pip install aiohttp
pip install cookiecutter==1.7.0
```

4. 安装模板和包后，运行以下命令以基于 *EchoBot* 模板创建机器人：

**C#**

```C#
dotnet new echobot -n TimeBot
```

**Python**

```Python
cookiecutter https://github.com/microsoft/botbuilder-python/releases/download/Templates/echo.zip
```

如果使用的是 Python，则在 cookiecutter 提示时，输入以下详细信息：
- **bot_name**：TimeBot
- **bot_description**：时间机器人
    
5. 在终端窗格中，输入以下命令以将当前目录更改为 **TimeBot** 文件夹，其中列出了已为机器人生成的代码文件：

    ```Code
    cd TimeBot
    dir
    ```

## 在 Bot Framework Emulator 中测试机器人

你已经基于 *EchoBot* 模板创建了一个机器人。现在，可以在本地运行该机器人，并使用 Bot Framework Emulator（应将其安装在系统上）对其进行测试。

1. 在终端窗格中，确保当前目录是包含机器人代码文件的 **TimeBot** 文件夹，然后输入以下命令以启动在本地运行的机器人。

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```
    
机器人启动时，请注意将显示正在运行该机器人的终结点。该终结点类似于 **http://localhost:3978**

2. 启动 Bot Framework Emulator，然后通过指定附加了 **/api/messages** 路径的终结点来打开机器人，如下所示：

    `http://localhost:3978/api/messages`

3. 在 **“实时聊天”** 窗格中打开对话后，等待出现消息 *“欢迎使用！”*。
4. 输入 *“你好”* 等消息，并查看机器人的响应，该响应应回显你输入的消息。
5. 关闭 Bot Framework Emulator，并返回到 Visual Studio Code，然后在终端窗口中，按 **Ctrl+C** 停止机器人。

## 修改机器人代码

你已经创建了一个机器人，该机器人将用户的输入回显给他们。这并不是特别有用，但可以用来说明对话式对话的基本流程。与机器人的对话包括一系列活动，其中文本、图形或用户界面卡用于交换信息。机器人以问候语开始对话，这是用户初始化与机器人的聊天会话时触发的对话更新活动的结果。然后，对话包含一系列进一步的活动，在对话中，用户和机器人进行多回合对话以发送消息。

1. 在 Visual Studio Code 中，为机器人打开以下代码文件：
    - **C#**：TimeBot/Bots/EchoBot.cs
    - **Python**：TimeBot/bot.py

    请注意，此文件中的代码包含活动处理程序函数，一个用于 **“已添加成员”** 对话更新活动（当某人加入聊天会话时），另一个用于 *“消息”* 活动（当收到消息时）。对话基于 *“回合”* 的概念，其中每个回合代表机器人在对话中接收、处理和响应活动的交互。*“回合上下文”* 用于跟踪有关当前回合中正在处理的活动的信息。

2. 在代码文件的顶部，添加以下命名空间导入语句：

**C#**

```C#
using System;
```

**Python**

```Python
from datetime import datetime
```

3. 修改消息活动的活动处理程序函数以匹配以下代码：

**C#**

```C#
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    string inputMessage = turnContext.Activity.Text;
    string responseMessage = "Ask me what the time is.";
    if (inputMessage.ToLower().StartsWith("what") && inputMessage.ToLower().Contains("time"))
    {
        var now = DateTime.Now;
        responseMessage = "The time is " + now.Hour.ToString() + ":" + now.Minute.ToString("D2");
    }
    await turnContext.SendActivityAsync(MessageFactory.Text(responseMessage, responseMessage), cancellationToken);
}
```

**Python**

```Python
async def on_message_activity(self, turn_context: TurnContext):
    input_message = turn_context.activity.text
    response_message = 'Ask me what the time is.'
    if (input_message.lower().startswith('what') and 'time' in input_message.lower()):
        now = datetime.now()
        response_message = 'The time is {}:{:02d}.'.format(now.hour,now.minute)
    await turn_context.send_activity(response_message)
```
    
4. 保存所做的更改，然后在终端窗格中，确保当前目录是包含机器人代码文件的 **TimeBot** 文件夹，然后输入以下命令以启动在本地运行的机器人。

**C#**

```C#
dotnet run
```

**Python**

```Python
python app.py
```

同样，当机器人启动时，请注意将显示正在运行该机器人的终结点。

5. 启动 Bot Framework Emulator，然后通过指定附加了 **/api/messages** 路径的终结点来打开机器人，如下所示：

    `http://localhost:3978/api/messages`

6. 在 **“实时聊天”** 窗格中打开对话后，等待出现消息 *“欢迎使用！”*。
7. 输入 *“你好”* 等消息，并查看机器人的响应，该响应应为 *“问我几点了”*。
8. 输入 *“几点了？”* 并查看响应。

    机器人现在通过显示机器人运行的本地时间来响应查询“几点了？”。对于任何其他查询，机器人会提示用户询问它几点了。这是一个非常有限的机器人，可以通过与语言理解服务和其他自定义代码集成来进行改进，但是它可以作为一个工作示例，说明如何通过扩展从模板创建的机器人来使用 Bot Framework SDK 构建解决方案。

9. 关闭 Bot Framework Emulator，并返回到 Visual Studio Code，然后在终端窗口中，按 **Ctrl+C** 停止机器人。

## 更多信息

要了解有关 Bot Framework 的详细信息，请参阅 [Bot Framework 文档](https://docs.microsoft.com/azure/bot-service/index-bf-sdk)。
