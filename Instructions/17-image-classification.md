---
lab:
  title: 使用自定义视觉对图像进行分类
  module: Module 9 - Developing Custom Vision Solutions
ms.openlocfilehash: a8d60adee3e43ce0c613b71c7cba72067fe2b799
ms.sourcegitcommit: acbffd6019fe2f1a6ea70870cf7411025c156ef8
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 01/12/2022
ms.locfileid: "137819557"
---
# <a name="classify-images-with-custom-vision"></a>使用自定义视觉对图像进行分类

使用“自定义视觉”服务可创建计算机视觉模型并基于自己的图像进行训练。 可以使用该服务来训练“图像分类”和“对象检测”模型；然后可以在应用程序中发布并使用这两个模型 。

在此练习中，你将使用自定义视觉服务来训练图像分类模型，该模型可识别三种水果（苹果、香蕉和橙子）。

## <a name="clone-the-repository-for-this-course"></a>克隆本课程的存储库

如果尚未将 AI-102-AIEngineer 代码存储库克隆到你要在此实验室中使用的环境，请按照以下步骤克隆它。 否则，请在 Visual Studio Code 中打开克隆的文件夹。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“Git: 克隆”命令，以将 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择“以后再说”。

## <a name="create-custom-vision-resources"></a>创建自定义视觉资源

需要先获取可用于训练和预测的 Azure 资源才能训练模型 。 可以为这些任务中的每一个创建“自定义视觉”资源，也可以创建一个“认知服务”资源并在任一（或两个）任务中使用 。

在此练习中，你将创建用于训练和预测的“自定义视觉”资源，以便分别管理这些工作负载的访问权限和成本。

1. 在新的浏览器标签页中，打开 Azure 门户 (`https://portal.azure.com`)，然后使用与 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择“&#65291;创建资源”按钮，搜索“自定义视觉”，并使用以下设置创建一个自定义视觉资源：
    - **创建选项**：共同点
    - **订阅**：Azure 订阅
    - 资源组：选择或创建一个资源组（如果你使用的是受限订阅，则可能无权创建新资源组，在此情况下，可使用一个已提供的资源组）
    - **区域**：选择任何可用区域
    - **名称**：输入唯一名称
    - **训练定价层**：F0
    - **预测定价层**：F0

    > **注意**：如果在你的订阅中已有 F0 自定义视觉服务，此处请选择“S0”。

3. 等待资源创建完成，然后查看部署详细信息，并注意两个自定义视觉资源是否已预配。一个资源用于训练，另一个用于预测（通过 -Prediction 后缀可判断出来）。 可以导航到你在其中创建了这些资源的资源组，然后查看这些资源。

> **重要说明**：每个资源都有自己的终结点和密钥，用于管理来自代码的访问 。 若要训练图像分类模型，代码必须使用训练资源（及其终结点和密钥）；若要使用经过训练的模型来预测图像类别，代码必须使用预测资源（及其终结点和密钥） 。

## <a name="create-a-custom-vision-project"></a>创建自定义视觉项目

若要训练图像分类模型，需根据训练资源创建自定义视觉项目。 为此，你将使用自定义视觉门户。

1. 在 Visual Studio Code 中，查看 17-image-classification/training-images 文件夹（你在其中克隆了存储库）中的训练图像。 此文件夹包含含苹果、香蕉和橙子图像的子文件夹。
2. 在新的浏览器标签页中，打开自定义视觉门户 (`https://customvision.ai`)。 如果出现提示，请使用与你的 Azure 订阅关联的 Microsoft 帐户进行登录，并同意服务条款。
3. 在自定义视觉门户中，创建一个具有以下设置的新项目：
    - **名称**：对水果进行分类
    - **说明**：水果的图像分类
    - 资源：先前创建的自定义视觉资源
    - **项目类型**：分类
    - 分类类型：多类（每个图像一个标记）
    - 域：食品
4. 在新项目中，单击“\[+\] 添加图像”，然后选择先前查看的 training-images/apple 文件夹中的所有文件 。 然后上传图像文件，指定标记“苹果”，如下所示：

![上传带有“苹果”标记的苹果](./images/upload_apples.jpg)
   
5. 重复上一步操作，上传“香蕉”文件夹中带有标记“香蕉”的图像，以及“橘子”文件夹中带有标记“橘子”的图像。
6. 浏览已在自定义视觉项目中上传的图像 - 每个类应有 15 个图像，如下所示：

![已标记的水果图像 - 15 个苹果、15 个香蕉和 15 个橘子](./images/fruit.jpg)
    
7. 在自定义视觉项目的图像上方，单击“训练”以使用已标记的图像训练分类模型。 选择“快速训练”选项，然后等待训练迭代完成（这可能需要一分钟左右）。
8. 模型迭代已训练后，请查看“精度”、“召回率”和“AP”性能指标 - 这些指标度量分类模型的预测准确性，并且应该都很高。

> **注意**：性能指标基于 50% 这一概率阈值来实现每一次预测（也就是说，如果模型计算得出图像为某特定类别的概率为 50% 或更高，则预测其为该类别）。 可以在页面左上角调整该阈值。

## <a name="test-the-model"></a>测试模型

现在，你已经训练了模型，接下来可对其进行测试。

1. 在性能指标上方，单击“快速测试”。
2. 在“图像 URL”框中，键入 `https://aka.ms/apple-image` 并单击“&#10132;”
3. 查看模型返回的预测 -“苹果”的概率分数应为最高，如下所示：

![带有“苹果”的类预测的图像](./images/test-apple.jpg)

4. 关闭“快速测试”窗口。

## <a name="view-the-project-settings"></a>查看项目设置

你创建的项目已分配有唯一标识符，你需要在与其交互的任意代码中指定该标识符。

1. 单击“性能”页面右上方的“设置”(&#9881;) 图标，查看项目设置。
2. 在“常规”（左侧）下，注意唯一标识该项目的项目 ID 。
3. 在右侧的“资源”下，可看到显示了密钥和终结点。 这些是训练资源的详细信息（还可通过在 Azure 门户中查看资源来获取这些信息）。

## <a name="use-the-training-api"></a>使用训练 API

自定义视觉门户提供了方便的用户界面，可用于上传和标记图像，以及训练模型。 但在某些情况下，你可能想要使用自定义视觉训练 API 自动训练模型。

> **注意**：在此练习中，可以选择在 C# 或 Python SDK 中使用 API 。 在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 17-image_classification 文件夹，并根据你的语言首选项展开 C-Sharp 文件夹或 Python 文件夹   。
2. 右键单击 train-classifier 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装自定义视觉训练包：

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Training --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.0
```

3. 查看 train-classifier 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#** ：appsettings.json
    - **Python**：.env

    打开配置文件并更新其包含的配置值，以反映自定义视觉训练资源的终结点和密钥，以及先前创建的分类项目的项目 ID。 保存更改。
4. 请注意，train-classifier 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：train-classifier.py

    打开代码文件并查看其中包含的代码，并注意以下详细信息：
    - 已导入你安装的包中的命名空间
    - Main 函数检索配置设置，并使用密钥和终结点来创建经过身份验证的 CustomVisionTrainingClient，然后将其与项目 ID 结合使用以创建对项目的项目引用  。
    - Upload_Images 函数检索自定义视觉项目中定义的标记，然后将图像文件从相应名称的文件夹上传到项目，并分配适当的标记 ID。
    - Train_Model 函数为项目创建新的训练迭代，然后等待训练完成。
5. 返回 train-classifier 文件夹的集成终端，然后输入以下命令以运行程序：

**C#**

```
dotnet run
```

**Python**

```
python train-classifier.py
```
    
6. 等待程序结束。 然后返回到浏览器，并在自定义视觉门户中查看项目的“训练图像”页面（如有必要，请刷新浏览器）。
7. 验证某些带标记的新图像是否已添加到项目中。 然后，查看“性能”页面，并验证是否已创建新迭代。

## <a name="publish-the-image-classification-model"></a>发布图像分类模型

现在即可发布经过训练的模型，以便在客户端应用程序中使用。

1. 在“自定义视觉”门户的“性能”页面上，单击“&#128504; 发布”以使用以下设置发布经过训练的模型 ：
    - **模型名称**：fruit-classifier
    - **预测资源**：先前创建的以“-Prediction”结尾的预测资源（<u>不是</u>训练资源）。
2. 在“项目设置”页面的左上角，单击“项目库”(&#128065;) 图标以返回到自定义视觉门户主页，此时其中列出了你的项目。
3. 在自定义视觉门户主页的右上角，单击“设置”(&#9881;) 图标以查看自定义视觉服务的设置。 然后，在“资源”下查找以“-Prediction”结尾的预测资源（<u>不是</u>训练资源），以确定其密钥和终结点值（也可以通过在 Azure 门户中查看资源来获取这些信息） 。

## <a name="use-the-image-classifier-from-a-client-application"></a>使用客户端应用程序中的图像分类器

现在，你已经发布了图像分类模型，接下来，可在客户端应用程序中使用。 同样，可以选择使用 C# 或 Python 。

1. 在 Visual Studio Code 的 17-image-classification 文件夹的首选语言（C-Sharp 或 Python）的子文件夹中，右键单击 test-classifier 文件夹并打开集成终端   ·。 然后，输入以下 SDK 特定的命令，安装自定义视觉预测包：

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.CustomVision.Prediction --version 2.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-customvision==3.1.0
```

> **注意**：Python SDK 包包括训练和预测包，并且可能已安装完毕。

2. 展开 test-classifier 文件夹以查看其包含的文件，这些文件用于为图像分类模型实现测试客户端应用程序。
3. 打开客户端应用程序的配置文件（对于 C#，是 appsettings.json；对于 Python，则是 env），并更新其包含的配置值，以反映自定义视觉预测资源的终结点和密钥、分类项目的项目 ID，以及发布的模型的名称（应为 fruit-classifier）   。 保存更改。
4. 打开客户端应用程序的代码文件（对于 C#，是 Program.cs；对于 Python，则是 test-classification.py），然后查看其包含的代码，并注意以下详细信息 ：
    - 已导入你安装的包中的命名空间
    - Main 函数检索配置设置，并使用密钥和终结点创建经过身份验证的 CustomVisionPredictionClient 。
    - 预测客户端对象用于预测 test-images 文件夹中每个图像的类别，并为每个请求指定项目 ID 和模型名称。 每个预测均包含每个可能类别的概率，但仅显示概率超过 50% 的预测标记。
5. 返回 test-classifier 文件夹的集成终端，然后输入以下 SDK 特定的命令以运行程序：

**C#**

```
dotnet run
```

**Python**

```
python test-classifier.py
```

6. 查看每个预测的标签（标记）和概率分数。 可以查看 test-images 文件夹中的图像，以验证模型是否已将其正确分类。

## <a name="more-information"></a>详细信息

若要详细了解自定义视觉服务中的图像分类，请参阅[自定义视觉文档](https://docs.microsoft.com/azure/cognitive-services/custom-vision-service/)。
