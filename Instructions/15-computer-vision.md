---
lab:
    title: '使用计算机视觉分析图像'
    module: '模块 8 - 计算机视觉入门'
---

# 使用计算机视觉分析图像

计算机视觉是一种人工智能功能，可支持软件系统通过分析图像来解释视觉输入。在 Microsoft Azure 中，**计算机视觉**认知服务提供用于完成常见计算机视觉任务的预生成模型，包括分析图像以建议描述文字和标记，以及检测常见物体、地标、名人、品牌和是否存在成人内容。还可使用计算机视觉服务来分析图像颜色和格式，并生成“智能裁剪”的缩略图图像。

## 克隆本课程的存储库

如果尚未将 **AI-102-AIEngineer** 代码存储库克隆到你要在此实验室中使用的环境，请按照以下步骤克隆它。否则，请在 Visual Studio Code 中打开克隆的文件夹。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行 **Git: Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**： 如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

## 预配认知服务资源

如果你的订阅中还没有**认知服务**资源，需要预配一个。

1. 打开 Azure 门户 `https://portal.azure.com`，使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择 **“&#65291;创建资源”** 按钮，搜索 *“认知服务”*，并使用以下设置创建一个**认知服务** 资源：
    - **订阅**： *你的 Azure 订阅*
    - **资源组**： *选择或创建一个资源组（如果你使用的是受限订阅，则可能无权创建新资源组，在此情况下，可使用一个已提供的资源组）*
    - **区域**： *选择任何可用区域*
    - **名称**： *输入唯一名称*
    - **定价层**：标准 S0
3. 选中所需复选框并创建资源。
4. 等待部署完成，然后查看部署详细信息。
5. 部署资源后，转到该资源并查看其 **“密钥和终结点”** 页面。你将在下一个过程中用到此页面中的终结点和其中一个密钥。

## 准备使用计算机视觉 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用计算机视觉 SDK 来分析图像。

> **备注**： 可选择将该 SDK 用于 **C#** 或 **Python**。在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的 **“资源管理器”** 窗格中，浏览到 **15-computer-vision** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **image-analysis** 文件夹，并打开集成终端。然后通过运行适用于你的语言首选项的命令，安装计算机视觉 SDK 包：

**C#**

```
dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
```

**Python**

```
pip install azure-cognitiveservices-vision-computervision==0.7.0
```
    
3. 查看 **image-analysis** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#**： appsettings.json
    - **Python**： .env

    打开配置文件，然后更新其中包含的配置值，以反映认知服务资源的终**结点**和身份验证**密钥**。保存更改。
4. 请注意，**image-analysis** 文件夹中包含客户端应用程序的代码文件：

    - **C#**： Program.cs
    - **Python**： image-analysis.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释 **“导入命名空间”**。然后在此注释下添加以下特定于语言的代码，以导入使用计算机视觉 SDK 所需的命名空间：

**C#**

```C#
// import namespaces
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision;
using Microsoft.Azure.CognitiveServices.Vision.ComputerVision.Models;
```

**Python**

```Python
# import namespaces
from azure.cognitiveservices.vision.computervision import ComputerVisionClient
from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
from msrest.authentication import CognitiveServicesCredentials
```
    
## 查看要分析的图像

在此练习中，你将使用计算机视觉服务来分析多张图像。

1. 在 Visual Studio Code 中，展开 **image-vision** 文件夹以及其中包含的 **images** 文件夹。
2. 依次选择每个图像文件，以在 Visual Studio Code 中查看它们。

## 分析图像以建议描述文字

现在，你已准备好使用 SDK 来调用计算机视觉服务并分析图像。

1. 在客户端应用程序的代码文件（**Program.cs** 或 **image-analysis.py**）中，可在 **Main** 函数中看到已提供用于加载配置设置的代码。然后查找注释 **“对计算机视觉对象客户端进行身份验证”**。然后在此注释下添加以下特定于语言的代码，以创建计算机视觉对象客户端对象并对其进行身份验证：

**C#**

```C#
// Authenticate Computer Vision client
ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
cvClient = new ComputerVisionClient(credentials)
{
    Endpoint = cogSvcEndpoint
};
```

**Python**

```Python
# Authenticate Computer Vision client
credential = CognitiveServicesCredentials(cog_key) 
cv_client = ComputerVisionClient(cog_endpoint, credential)
```

2. 在 **Main** 函数中，可在你刚刚添加的代码下看到，代码指定了图像文件的路径，然后将图像路径传递给另外两个函数（**AnalyzeImage** 和 **GetThumbnail**）。这两个函数尚未完全实现。

3. 在 **AnalyzeImage** 函数的注释 **“指定要检索的特征”** 下，添加以下代码：

**C#**

```C#
// Specify features to be retrieved
List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
{
    VisualFeatureTypes.Description,
    VisualFeatureTypes.Tags,
    VisualFeatureTypes.Categories,
    VisualFeatureTypes.Brands,
    VisualFeatureTypes.Objects,
    VisualFeatureTypes.Adult
};
```

**Python**

```Python
# Specify features to be retrieved
features = [VisualFeatureTypes.description,
            VisualFeatureTypes.tags,
            VisualFeatureTypes.categories,
            VisualFeatureTypes.brands,
            VisualFeatureTypes.objects,
            VisualFeatureTypes.adult]
```
    
4. 在 **AnalyzeImage** 函数的注释 **“获取图像分析”** 下，添加以下代码（包括指示你稍后将在何处添加更多代码的注释。）：

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // get image captions
    foreach (var caption in analysis.Description.Captions)
    {
        Console.WriteLine($"Description: {caption.Text} (confidence: {caption.Confidence.ToString("P")})");
    }

    // Get image tags


    // Get image categories


    // Get brands in the image


    // Get objects in the image


    // Get moderation ratings
    

}            
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

# Get image description
for caption in analysis.description.captions:
    print("Description: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get image categories 


# Get brands in the image


# Get objects in the image


# Get moderation ratings

```
    
5. 保存你的更改并返回到 **image-analysis** 文件夹的集成终端，然后输入以下命令以使用参数 **images/street.jpg** 运行程序：

**C#**

```
dotnet run images/street.jpg
```

**Python**

```
python image-analysis.py images/street.jpg
```
    
6. 观察输出，其中应包括图像 **street.jpg** 的建议描述文字。
7. 再次运行程序，但此次使用参数 **images/building.jpg**，以查看为图像 **building.jpg** 生成的描述文字。
8. 重复前面的步骤，为文件 **images/person.jpg** 生成描述文字。

## 获取图像的建议标记

这有时可用于标识相关标记，这*些标*记提供了与图像内容有关的线索。

1. 在 **AnalyzeImage** 函数的注释 **“获取图像标记”** 下，添加以下代码：

**C#**

```C
// Get image tags
if (analysis.Tags.Count > 0)
{
    Console.WriteLine("Tags:");
    foreach (var tag in analysis.Tags)
    {
        Console.WriteLine($" -{tag.Name} (confidence: {tag.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get image tags
if (len(analysis.tags) > 0):
    print("Tags: ")
    for tag in analysis.tags:
        print(" -'{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，注意到除了图像描述文字外，还会显示建议的标记列表。

## 获取图像类别

计算机视觉服务可建议图像的*类别*，并且可标识每个类别中的知名地标或名人。

1. 在 **AnalyzeImage** 函数的注释 **“获取图像类别（包括名人和地标）”** 下，添加以下代码：

**C#**

```C
// Get image categories (including celebrities and landmarks)
List<LandmarksModel> landmarks = new List<LandmarksModel> {};
List<CelebritiesModel> celebrities = new List<CelebritiesModel> {};
Console.WriteLine("Categories:");
foreach (var category in analysis.Categories)
{
    // Print the category
    Console.WriteLine($" -{category.Name} (confidence: {category.Score.ToString("P")})");

    // Get landmarks in this category
    if (category.Detail?.Landmarks != null)
    {
        foreach (LandmarksModel landmark in category.Detail.Landmarks)
        {
            if (!landmarks.Any(item => item.Name == landmark.Name))
            {
                landmarks.Add(landmark);
            }
        }
    }

    // Get celebrities in this category
    if (category.Detail?.Celebrities != null)
    {
        foreach (CelebritiesModel celebrity in category.Detail.Celebrities)
        {
            if (!celebrities.Any(item => item.Name == celebrity.Name))
            {
                celebrities.Add(celebrity);
            }
        }
    }
}

// If there were landmarks, list them
if (landmarks.Count > 0)
{
    Console.WriteLine("Landmarks:");
    foreach(LandmarksModel landmark in landmarks)
    {
        Console.WriteLine($" -{landmark.Name} (confidence: {landmark.Confidence.ToString("P")})");
    }
}

// If there were celebrities, list them
if (celebrities.Count > 0)
{
    Console.WriteLine("Celebrities:");
    foreach(CelebritiesModel celebrity in celebrities)
    {
        Console.WriteLine($" -{celebrity.Name} (confidence: {celebrity.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get image categories (including celebrities and landmarks)
if (len(analysis.categories) > 0):
    print("Categories:")
    landmarks = []
    celebrities = []
    for category in analysis.categories:
        # Print the category
        print(" -'{}' (confidence: {:.2f}%)".format(category.name, category.score * 100))
        if category.detail:
            # Get landmarks in this category
            if category.detail.landmarks:
                for landmark in category.detail.landmarks:
                    if landmark not in landmarks:
                        landmarks.append(landmark)

            # Get celebrities in this category
            if category.detail.celebrities:
                for celebrity in category.detail.celebrities:
                    if celebrity not in celebrities:
                        celebrities.append(celebrity)

    # If there were landmarks, list them
    if len(landmarks) > 0:
        print("Landmarks:")
        for landmark in landmarks:
            print(" -'{}' (confidence: {:.2f}%)".format(landmark.name, landmark.confidence * 100))

    # If there were celebrities, list them
    if len(celebrities) > 0:
        print("Celebrities:")
        for celebrity in celebrities:
            print(" -'{}' (confidence: {:.2f}%)".format(celebrity.name, celebrity.confidence * 100))

```
    
2. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，注意到除了图像描述文字和标记外，还会显示建议的类别以及任何已识别的地标或名人（尤其是图像 **building.jpg** 和 **person.jpg** 中的地标和名人）。

## 获取图像中的品牌

可通过可视方式从徽标中识别某些品牌，即使徽标中未显示品牌名称也是如此。计算机视觉服务经过训练，可识别数千个知名品牌。

1. 在 **AnalyzeImage** 函数的注释 **“获取图像中的品牌”** 下，添加以下代码：

**C#**

```C
// Get brands in the image
if (analysis.Brands.Count > 0)
{
    Console.WriteLine("Brands:");
    foreach (var brand in analysis.Brands)
    {
        Console.WriteLine($" -{brand.Name} (confidence: {brand.Confidence.ToString("P")})");
    }
}
```

**Python**

```Python
# Get brands in the image
if (len(analysis.brands) > 0):
    print("Brands: ")
    for brand in analysis.brands:
        print(" -'{}' (confidence: {:.2f}%)".format(brand.name, brand.confidence * 100))
```
    
2. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，注意识别的任何品牌（尤其是图像 **person.jpg** 中的品牌）。

## 检测图像中的物体并指示其位置

*物体检测*是计算机视觉服务的一种特定形式，可识别图像中的各个物体，并使用边界框指示其位置。

1. 在 **AnalyzeImage** 函数的注释 **“获取图像中的物体”** 下，添加以下代码：

**C#**

```C
// Get objects in the image
if (analysis.Objects.Count > 0)
{
    Console.WriteLine("Objects in image:");

    // Prepare image for drawing
    Image image = Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.Black);

    foreach (var detectedObject in analysis.Objects)
    {
        // Print object name
        Console.WriteLine($" -{detectedObject.ObjectProperty} (confidence: {detectedObject.Confidence.ToString("P")})");

        // Draw object bounding box
        var r = detectedObject.Rectangle;
        Rectangle rect = new Rectangle(r.X, r.Y, r.W, r.H);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.ObjectProperty,font,brush,r.X, r.Y);

    }
    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file);   
}
```

**Python**

```Python
# Get objects in the image
if len(analysis.objects) > 0:
    print("Objects in image:")

    # Prepare image for drawing
    fig = plt.figure(figsize=(8, 8))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    color = 'cyan'
    for detected_object in analysis.objects:
        # Print object name
        print(" -{} (confidence: {:.2f}%)".format(detected_object.object_property, detected_object.confidence * 100))
        
        # Draw object bounding box
        r = detected_object.rectangle
        bounding_box = ((r.x, r.y), (r.x + r.w, r.y + r.h))
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.object_property,(r.x, r.y), backgroundcolor=color)
    # Save annotated image
    plt.imshow(image)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```
    
2. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，注意检测到的任何物体。在每次运行后，查看在代码文件所在的同一文件夹中生成的 **objects.jpg** 文件，以查看带有批注的物体。

## 获取图像的审核评级

一些图像可能并非适合所有受众，你可能需要进行一些审核，以识别成人或暴力性质的图像。

1. 在 **AnalyzeImage** 函数的注释 **“获取审核评级”** 下，添加以下代码：

**C#**

```C
// Get moderation ratings
string ratings = $"Ratings:\n -Adult: {analysis.Adult.IsAdultContent}\n -Racy: {analysis.Adult.IsRacyContent}\n -Gore: {analysis.Adult.IsGoryContent}";
Console.WriteLine(ratings);
```

**Python**

```Python
# Get moderation ratings
ratings = 'Ratings:\n -Adult: {}\n -Racy: {}\n -Gore: {}'.format(analysis.adult.is_adult_content,
                                                                    analysis.adult.is_racy_content,
                                                                    analysis.adult.is_gory_content)
print(ratings)
```
    
2. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，注意每张图像的评级。

> **备注**： 在前面的任务中，你使用了单种方法来分析图像，然后逐步添加代码来分析和显示结果。SDK 还提供了用于建议描述文字、识别标记和检测物体等的单独方法，这意味着你可使用最适合的方法来仅返回所需信息，减少需要返回的数据有效负载的大小。有关更多详细信息，请参阅 [.NET SDK 文档](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/computervision?view=azure-dotnet) 或 [Python SDK 文档](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/computervision?view=azure-python)。

## 生成缩略图图像

在某些情况下，你可能需要创建较小的图像（即*缩略图*），将其裁剪为在新的图像尺寸中包含主要视觉主体。

1. 在代码文件中查找 **GetThumbnail** 函数，在注释 **“生成缩略图”** 下添加以下代码：

**C#**

```C
// Generate a thumbnail
using (var imageData = File.OpenRead(imageFile))
{
    // Get thumbnail data
    var thumbnailStream = await cvClient.GenerateThumbnailInStreamAsync(100, 100,imageData, true);

    // Save thumbnail image
    string thumbnailFileName = "thumbnail.png";
    using (Stream thumbnailFile = File.Create(thumbnailFileName))
    {
        thumbnailStream.CopyTo(thumbnailFile);
    }

    Console.WriteLine($"Thumbnail saved in {thumbnailFileName}");
}
```

**Python**

```Python
# Generate a thumbnail
with open(image_file, mode="rb") as image_data:
    # Get thumbnail data
    thumbnail_stream = cv_client.generate_thumbnail_in_stream(100, 100, image_data, True)

# Save thumbnail image
thumbnail_file_name = 'thumbnail.png'
with open(thumbnail_file_name, "wb") as thumbnail_file:
    for chunk in thumbnail_stream:
        thumbnail_file.write(chunk)

print('Thumbnail saved in.', thumbnail_file_name)
```
    
2. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，打开在每张图像的代码文件所在的同一文件夹中生成的 **thumbnail.jpg** 文件。

## 更多信息

在此练习中，你探索了计算机视觉服务的一些图像分析和操作功能。该服务器还包括用于读取文本、检测人脸和完成其他计算机视觉任务的功能。

有关使用**计算机视觉**服务的详细信息，请参阅[计算机视觉文档](https://docs.microsoft.com/azure/cognitive-services/computer-vision/)。
