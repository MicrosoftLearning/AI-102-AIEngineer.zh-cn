---
lab:
  title: 实验室环境设置
  module: Setup
ms.openlocfilehash: 57363ce9fec94cb3a1a6ef7622a10bca48a6f055
ms.sourcegitcommit: d6da3bcb25d1cff0edacd759e75b7608a4694f03
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/16/2021
ms.locfileid: "137819556"
---
# <a name="lab-environment-setup"></a>实验室环境设置

这些练习被设计为在托管实验室环境中完成。 如果你想要在自己的计算机上完成，可以安装以下软件来实现这一目标。 使用自己的环境时，可能会遇到意外的对话和行为。 由于可能的本地配置的范围很广，因此课程团队可能无法解决你在自己的环境中遇到的问题。

> **注意**：以下说明适用于 Windows 10 计算机。 还可以使用 Linux 或 MacOS。 你可能需要根据所选的 OS 调整实验室说明。

### <a name="base-operating-system-windows-10"></a>基本操作系统 (Windows 10)

#### <a name="windows-10"></a>Windows 10

安装 Windows 10 并应用所有更新。

#### <a name="edge"></a>Microsoft Edge

安装 [Microsoft Edge (Chromium)](https://microsoft.com/edge)

### <a name="net-core-sdk"></a>.NET Core SDK

1. 从 https://dotnet.microsoft.com/download 下载并安装（下载 .NET Core SDK - 不仅是运行时）

### <a name="c-redistributable"></a>C++ Redistributable

1. 从 https://aka.ms/vs/16/release/vc_redist.x64.exe 下载并安装 Visual C++ 可再发行程序包 (x64)。

### <a name="nodejs"></a>Node.JS

1. 从 https://nodejs.org/en/download/ 下载最新的 LTS 版本 
2. 使用默认选项进行安装

### <a name="python-and-required-packages"></a>Python（和必需的包）

1. 从 https://docs.conda.io/en/latest/miniconda.html 下载版本 3.8 
2. 运行安装程序进行安装 - 注意事项：选择“将 Miniconda 添加到 PATH 变量”和“将 Miniconda 注册为默认 Python 环境”选项。
3. 安装后，打开 Anaconda 提示符并输入以下命令来安装包： 

```
pip install flask requests python-dotenv pylint matplotlib pillow
pip install --upgrade numpy
```

### <a name="azure-cli"></a>Azure CLI

1. 从 https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest 下载 
2. 使用默认选项进行安装

### <a name="git"></a>Git

1. 使用默认选项从 https://git-scm.com/download.html 下载并安装


### <a name="visual-studio-code-and-extensions"></a>Visual Studio Code（和扩展）

1. 从 https://code.visualstudio.com/Download 下载 
2. 使用默认选项进行安装 
3. 安装后，启动 Visual Studio Code，然后在“扩展”选项卡 (Ctrl+Shift+X) 上，搜索并安装以下 Microsoft 扩展：
    - Python
    - C#
    - Azure Functions
    - PowerShell


### <a name="bot-framework-emulator"></a>Bot Framework Emulator

按照 https://github.com/Microsoft/BotFramework-Emulator/blob/master/README.md 中的说明为操作系统下载并安装最新稳定版 Bot Framework Emulator。

### <a name="bot-framework-composer"></a>Bot Framework Composer

从 https://docs.microsoft.com/en-us/composer/install-composer 安装。
