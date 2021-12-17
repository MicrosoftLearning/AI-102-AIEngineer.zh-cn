---
lab:
    title: '读取图像中的文本'
    module: '模块 11 - 读取图像和文档中的文本'
---

# 读取图像中的文本

光学字符识别 (OCR) 是计算机视觉服务的一项功能，用于读取图像和文档中的文本。**计算机视觉**服务提供两个用于读取文本的 API，你将在此练习中对其进行探索。

## 克隆本课程的存储库

如果尚未克隆用于本课程的存储库，请克隆它：

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行 **Git: Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**：如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

## 预配认知服务资源

如果你的订阅中还没有**认知服务**资源，需要预配一个。

1. 打开 Azure 门户 `https://portal.azure.com`，使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择 **“&#65291;创建资源”** 按钮，搜索 *“认知服务”*，并使用以下设置创建一个**认知服务**资源：
    - **订阅**： *你的 Azure 订阅*
    - **资源组**： *选择或创建一个资源组（如果你使用的是受限订阅，则可能无权创建新资源组，在此情况下，可使用一个已提供的资源组）*
    - **区域**： *选择任何可用区域*
    - **名称**： *输入唯一名称*
    - **定价层**： 标准 S0
3. 选中所需复选框并创建资源。
4. 等待部署完成，然后查看部署详细信息。
5. 部署资源后，转到该资源并查看其 **“密钥和终结点”** 页面。你将在下一个过程中用到此页面中的终结点和其中一个密钥。

## 准备使用计算机视觉 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用计算机视觉 SDK 来读取文本。

> **备注**： 可选择将该 SDK 用于 **C#** 或 **Python**。在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的 **“资源管理器”** 窗格中，浏览到 **20-ocr** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **read-text** 文件夹，并打开集成终端。然后通过运行适用于你的语言首选项的命令，安装计算机视觉 SDK 包：

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```

3. 查看 **read-text** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#**： appsettings.json
    - **Python**： .env

    打开配置文件，然后更新其中包含的配置值，以反映认知服务资源的终**结点**和身份验证**密钥**。保存更改。
4. 请注意，**read-text** 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**： read-text.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释 **“导入命名空间”**。然后在此注释下添加以下特定于语言的代码，以导入使用计算机视觉 SDK 所需的命名空间：

**C#**

```C#
// 导入命名空间
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```

**Python**

```Python
# 导入命名空间
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import OperationStatusCodes
from msrest.authentication import CognitiveServicesCredentials
```

5. 在客户端应用程序的代码文件中，可在 **Main** 函数中看到已提供用于加载配置设置的代码。然后查找注释 **“对计算机视觉对象客户端进行身份验证”**。然后在此注释下添加以下特定于语言的代码，以创建计算机视觉对象客户端对象并对其进行身份验证：

**C#**

```C#
// 对计算机视觉对象客户端进行身份验证
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# 对计算机视觉对象客户端进行身份验证
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```
    
## 使用 OCR API

**OCR** API 是一种光学字符识别 API，并针对读取 *jpg*、*png*、*gif* 和 *bmp* 格式图像中的少到中量印刷体文本进行了优化。它支持多种语言，并且除了读取图像中的文本外，还可确定每个文本区域的方向，并返回有关文本相对于图像的旋转角度的信息

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **1** 时运行的代码。此代码会调用 **GetTextOcr** 函数并传递图像文件的路径。
2. 在 **read-text/images** 文件夹中，打开 **Lincoln.jpg** 以查看代码将处理的图像。
3. 返回代码文件，查找 **GetTextOcr** 函数，并在用于在控制台中显示消息的现有代码下添加以下代码：

**C#**

```C#
// 使用 OCR API 读取图像中的文本
using (var imageData = File.OpenRead(imageFile))
{    
    var ocrResults = await cvClient.RecognizePrintedTextInStreamAsync(detectOrientation:false, image:imageData);

    // 准备要绘制的图像
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Magenta, 3);

    foreach(var region in ocrResults.Regions)
    {
        foreach(var line in region.Lines)
        {
            // 显示文本行的位置
            int[] dims = line.BoundingBox.Split(",").Select(int.Parse).ToArray();
            Rectangle rect = new Rectangle(dims[0], dims[1], dims[2], dims[3]);
            graphics.DrawRectangle(pen, rect);

            // 读取文本行中的文字
            string lineText = "";
            foreach(var word in line.Words)
            {
                lineText += word.Text + " ";
            }
            Console.WriteLine(lineText.Trim());
        }
    }

    // 保存突出显示文本位置的图像
    String output_file = "ocr_results.jpg";
    image.Save(output_file);
    Console.WriteLine("Results saved in " + output_file);
}
```

**Python**

```Python
# 使用 OCR API 读取图像中的文本
with open(image_file, mode="rb") as image_data:
    ocr_results = cv_client.recognize_printed_text_in_stream(image_data)

# 准备要绘制的图像
fig = plt.figure(figsize=(7, 7))
img = Image.open(image_file)
draw = ImageDraw.Draw(img)

# 逐行处理文本
for region in ocr_results.regions:
    for line in region.lines:

        # 显示文本行的位置
        l,t,w,h = list(map(int, line.bounding_box.split(',')))
        draw.rectangle(((l,t), (l+w, t+h)), outline='magenta', width=5)

        # 读取文本行中的文字
        line_text = ''
        for word in line.words:
            line_text += word.text + ' '
        print(line_text.rstrip())

# 保存突出显示文本位置的图像
plt.axis('off')
plt.imshow(img)
outputfile = 'ocr_results.jpg'
fig.savefig(outputfile)
print('Results saved in', outputfile)
```

4. 检查添加到 **GetTextOcr** 函数的代码。该代码会在图像文件中检测印刷体文本所在的区域，针对每个区域提取文本行，并在图像上突出显示这些位置。然后它会提取每行的文字并显示。
5. 保存你的更改并返回到 **read-text** 文件夹的集成终端，然后输入以下命令以运行程序：

**C#**

```
dotnet run
```

*C# 输出可能显示有关异步函数在使用 **await** 运算符的警告。可以忽略该警告。*

**Python**

```
python read-text.py
```

6. 在出现提示时输入 **1** 并观察输出，该输出是从图像中提取的文本。
7. 查看在代码文件所在的同一文件夹中生成的 **ocr_results.jpg** 文件，以查看图像中带有批注的文本行。

## 使用读取 API

**读取** API 使用比 OCR API 更新的文本识别模型，并且对于包含大量文本的大型图像效果更佳。它还支持从 **pdf** 文件中提取文本，并且可识别印刷体文本（多种语言）和手写文本（英语）。

**读取** API 使用异步操作模型，在该模型中，在提交开始文本识别的请求后，即可使用请求返回的操作 ID 来检查进度和检索结果。

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **2** 时运行的代码。此代码会调用 **GetTextRead** 函数并传递 PDF 文档文件的路径。
2. 在 **read-text/images** 文件夹中，右键单击 **Rome.pdf** 并选择 **“在文件资源管理器中显示”**。然后在文件资源管理器中打开并查看 PDF 文件。
3. 返回 Visual Studio Code 中的代码文件，查找 **GetTextRead** 函数，并在用于在控制台中显示消息的现有代码下添加以下代码：

**C#**

```C#
// 使用读取 API 读取图像中的文本
using (var imageData = File.OpenRead(imageFile))
{    
    var readOp = await cvClient.ReadInStreamAsync(imageData);

    // 获取异步操作 ID 以便检查结果
    string operationLocation = readOp.OperationLocation;
    string operationId = operationLocation.Substring(operationLocation.Length - 36);

    // 等待异步操作完成
    ReadOperationResult results;
    do
    {
        Thread.Sleep(1000);
        results = await cvClient.GetReadResultAsync(Guid.Parse(operationId));
    }
    while ((results.Status == OperationStatusCodes.Running ||
            results.Status == OperationStatusCodes.NotStarted));

    // 如果操作成功，则逐行处理文本
    if (results.Status == OperationStatusCodes.Succeeded)
    {
        var textUrlFileResults = results.AnalyzeResult.ReadResults;
        foreach (ReadResult page in textUrlFileResults)
        {
            foreach (Line line in page.Lines)
            {
                Console.WriteLine(line.Text);
            }
        }
    }
}  
```

**Python**

```Python
# 使用读取 API 读取图像中的文本
with open(image_file, mode="rb") as image_data:
    read_op = cv_client.read_in_stream(image_data, raw=True)

    # 获取异步操作 ID 以便检查结果
    operation_location = read_op.headers["Operation-Location"]
    operation_id = operation_location.split("/")[-1]

    # 等待异步操作完成
    while True:
        read_results = cv_client.get_read_result(operation_id)
        if read_results.status not in [OperationStatusCodes.running, OperationStatusCodes.not_started]:
            break
        time.sleep(1)

    # 如果操作成功，则逐行处理文本
    if read_results.status == OperationStatusCodes.succeeded:
        for page in read_results.analyze_result.read_results:
            for line in page.lines:
                print(line.text)
```
    
4. 检查添加到 **GetTextRead** 函数的代码。该代码会提交读取操作请求，然后重复检查状态，直到操作完成。如果操作成功，则代码会通过先循环访问每一页，然后循环访问每一行来处理结果。
5. 保存你的更改并返回到 **read-text** 文件夹的集成终端，然后输入以下命令以运行程序：

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

6. 在出现提示时输入 **2** 并观察输出，该输出是从文档中提取的文本。

## 读取手写文本

除了印刷体文本外，**读取** API 还可提取英语手写文本。

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **3** 时运行的代码。此代码会调用 **GetTextRead** 函数并传递图像文件的路径。
2. 在 **read-text/images** 文件夹中，打开 **Note.jpg** 以查看代码将处理的图像。
3. 在 **read-text** 文件夹的集成终端中，输入以下命令以运行程序：

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

4. 在出现提示时输入 **3** 并观察输出，该输出是从文档中提取的文本。

## 更多信息

有关使用**计算机视觉**服务读取文本的详细信息，请参阅[计算机视觉文档](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-recognizing-text)。
