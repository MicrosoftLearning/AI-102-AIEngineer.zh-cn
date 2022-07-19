---
lab:
  title: 检测和分析人脸
  module: Module 10 - Detecting, Analyzing, and Recognizing Faces
ms.openlocfilehash: ac73112414f9e0e4f944029dbfb1f5df0f204f28
ms.sourcegitcommit: f19509304dff38600d6431f4873e0d580fb51425
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/14/2022
ms.locfileid: "147124829"
---
# <a name="detect-and-analyze-faces"></a>检测和分析人脸

检测和分析人脸的能力是一项核心 AI 功能。 在此练习中，你将探索两个可用于处理图像中的人脸的 Azure 认知服务：计算机视觉服务和人脸服务。

> **注意**：从 2022 年 6 月 21 日开始，返回个人身份信息的认知服务的功能仅限于被授予 [有限访问权限](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)的客户。 此外，推断情绪状态的功能不再可用。 这些限制可能会影响本实验室练习。 我们正在努力解决此问题，但与此同时，执行以下步骤时可能会遇到一些错误；对此我们深表歉意。 有关 Microsoft 所做的更改的更多详细信息，以及原因 - 请参阅[负责任 AI 投资和面部识别防护措施](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/)。

## <a name="clone-the-repository-for-this-course"></a>克隆本课程的存储库

如果尚未克隆用于本课程的存储库，请克隆它：

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“Git: 克隆”命令，以将 `https://github.com/MicrosoftLearning/AI-102-AIEngineer` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择“以后再说”。

## <a name="provision-a-cognitive-services-resource"></a>预配认知服务资源

如果订阅中还没有认知服务资源，则需要预配认知服务资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择“&#65291;创建资源”按钮，搜索“认知服务”，并使用以下设置创建一个认知服务资源：
    - **订阅**：Azure 订阅
    - 资源组：选择或创建一个资源组（如果你使用的是受限订阅，则可能无权创建新资源组，在此情况下，可使用一个已提供的资源组）
    - **区域**：选择任何可用区域
    - **名称**：输入唯一名称
    - 定价层：标准版 S0
3. 选中所需的复选框并创建资源。
4. 等待部署完成，然后查看部署详细信息。
5. 部署资源后，转到该资源并查看其“密钥和终结点”页面。 你将在下一个过程中用到此页面中的终结点和其中一个密钥。

## <a name="prepare-to-use-the-computer-vision-sdk"></a>准备使用计算机视觉 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用计算机视觉 SDK 来分析图像中的人脸。

> **注意**：可选择将该 SDK 用于 C# 或 Python 。 在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 19-face 文件夹，并根据你的语言首选项展开 C-Sharp 文件夹或 Python 文件夹。
2. 右键单击 computer-vision 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装计算机视觉 SDK 包：

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-computervision==0.7.0
    ```
    
3. 查看 computer-vision 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#** ：appsettings.json
    - **Python**：.env

4. 打开配置文件，然后更新其中包含的配置值，以反映认知服务资源的终结点和身份验证密钥。 保存更改。

5. 请注意，computer-vision 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：detect-faces.py

6. 打开代码文件，并在顶部的现有命名空间引用下找到注释“导入命名空间”。 然后在此注释下添加以下特定于语言的代码，以导入使用计算机视觉 SDK 所需的命名空间：

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

## <a name="view-the-image-you-will-analyze"></a>查看要分析的图像

在此练习中，你将使用计算机视觉服务来分析人群图像。

1. 在 Visual Studio Code 中，展开 computer-vision 文件夹以及其中包含的 images 文件夹。
2. 选择并查看 people.jpg 图像。

## <a name="detect-faces-in-an-image"></a>在图像中检测人脸

现在，你可使用 SDK 来调用计算机视觉服务并检测图像中的人脸。

1. 在客户端应用程序的代码文件（Program.cs 或 detect-faces.py）中，可在 Main 函数中看到已提供用于加载配置设置的代码  。 然后查找注释“对计算机视觉对象客户端进行身份验证”。 然后在此注释下添加以下特定于语言的代码，以创建计算机视觉对象客户端对象并对其进行身份验证：

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

2. 在 Main 函数中，可在你刚刚添加的代码下看到，代码指定了图像文件的路径，然后将图像路径传递给名为 AnalyzeFaces 的函数。 此函数尚未完全实现。

3. 在 AnalyzeFaces 函数的注释“指定要检索的特征（人脸）”下，添加以下代码：

    **C#**

    ```C#
    // Specify features to be retrieved (faces)
    List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
    {
        VisualFeatureTypes.Faces
    };
    ```

    **Python**

    ```Python
    # Specify features to be retrieved (faces)
    features = [VisualFeatureTypes.faces]
    ```

4. 在 AnalyzeFaces 函数的注释“获取图像分析”下，添加以下代码：

**C#**

```C
// Get image analysis
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // Get faces
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // Draw and annotate each face
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person at approximately {r.Left}, {r.Top}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
# Get image analysis
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

    # Get faces
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person at approximately {}, {}'.format(r.left, r.top)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```

5. 保存你的更改并返回到 computer-vision 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-faces.py
    ```

6. 查看输出，它应该会指出检测到的人脸数。
7. 查看在代码文件所在的同一文件夹中生成的 detected_faces.jpg 文件，以查看带有批注的人脸。 本例的代码使用人脸特征来标记框左上角的位置，并使用边界框坐标来绘制框住每张人脸的矩形。

## <a name="prepare-to-use-the-face-sdk"></a>准备使用人脸 SDK

虽然计算机视觉服务提供了基本的人脸检测功能（以及许多其他图像分析功能），但人脸服务提供更全面的面部分析和识别功能。

1. 在 Visual Studio Code 的“资源管理器”窗格中，浏览到 19-face 文件夹，并根据你的语言首选项展开 C-Sharp 文件夹或 Python 文件夹。
2. 右键单击 face-api 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装人脸 SDK 包：

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.4.1
    ```
    
3. 查看 face-api 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#** ：appsettings.json
    - **Python**：.env

4. 打开配置文件，然后更新其中包含的配置值，以反映认知服务资源的终结点和身份验证密钥。 保存更改。

5. 请注意，face-api 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：analyze-faces.py

6. 打开代码文件，并在顶部的现有命名空间引用下找到注释“导入命名空间”。 然后在此注释下添加以下特定于语言的代码，以导入使用计算机视觉 SDK 所需的命名空间：

    **C#**

    ```C#
    // Import namespaces
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. 请注意，Main 函数中已提供用于加载配置设置的代码。 然后查找注释“对人脸客户端进行身份验证”。 然后在此注释下添加以下特定于语言的代码，以创建 FaceClient 对象并对其进行身份验证：

    **C#**

    ```C#
    // Authenticate Face client
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # Authenticate Face client
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. 在 Main 函数中，可在你刚刚添加的代码下看到，代码显示了一个菜单，可通过该菜单调用代码中的函数以探索人脸服务的功能。 你将在此练习的剩余部分中实现这些函数。

## <a name="detect-and-analyze-faces"></a>检测和分析人脸

人脸服务最基本的功能之一是检测图像中的人脸并确定其特征（例如头部姿态、模糊、是否佩戴眼镜等等）。

1. 在应用程序的代码文件中，在 Main 函数中检查用户选择菜单选项 1 时运行的代码。 此代码会调用 DetectFaces 函数并传递图像文件的路径。
2. 在代码文件中查找 DetectFaces 函数，并在注释“指定要检索的面部特征”下添加以下代码：

    **C#**

    ```C#
    // Specify facial features to be retrieved
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Occlusion,
        FaceAttributeType.Blur,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeType.occlusion,
                FaceAttributeType.blur,
                FaceAttributeType.glasses]
    ```

3. 在 DetectFaces 函数中刚刚添加的代码下，查找注释“获取人脸”并添加以下代码：

**C#**

```C
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            // Get face properties
            Console.WriteLine($"\nFace ID: {face.FaceId}");
            Console.WriteLine($" - Mouth Occluded: {face.FaceAttributes.Occlusion.MouthOccluded}");
            Console.WriteLine($" - Eye Occluded: {face.FaceAttributes.Occlusion.EyeOccluded}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face ID: {face.FaceId}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // Save annotated image
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# Get faces
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            print('\nFace ID: {}'.format(face.face_id))
            detected_attributes = face.face_attributes.as_dict()
            if 'blur' in detected_attributes:
                print(' - Blur:')
                for blur_name in detected_attributes['blur']:
                    print('   - {}: {}'.format(blur_name, detected_attributes['blur'][blur_name]))
                    
            if 'occlusion' in detected_attributes:
                print(' - Occlusion:')
                for occlusion_name in detected_attributes['occlusion']:
                    print('   - {}: {}'.format(occlusion_name, detected_attributes['occlusion'][occlusion_name]))

            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face ID: {}'.format(face.face_id)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. 检查添加到 DetectFaces 函数的代码。 该代码分析了图像文件，并检测了其中包含的任何人脸及其特征（年龄、情绪以及是否佩戴眼镜）。 该代码还会显示每张人脸的详细信息（包括为每张人脸分配的唯一人脸标识符）；并使用边界框在图像上标出了人脸所在的位置。
5. 保存你的更改并返回到 face-api 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    C# 输出可能显示有关异步函数在使用 await 运算符的警告。可以忽略这些警告。

    **Python**

    ```
    python analyze-faces.py
    ```

6. 在出现提示时输入 1 并观察输出，输出中应包含检测到的每张人脸的 ID 和特征。
7. 查看在代码文件所在的同一文件夹中生成的 detected_faces.jpg 文件，以查看带有批注的人脸。

## <a name="more-information"></a>详细信息

人脸服务中提供了几个附加功能，但根据[负责任 AI 标准](https://aka.ms/aah91ff)，这些功能受限于有限访问策略。 这些功能包括识别、验证和创建面部识别模型。 若要了解详细信息并申请访问权限，请参阅[认知服务的有限访问](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access)。

有关使用计算机视觉服务进行面部检测的详细信息，请参阅[计算机视觉文档](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)。

要详细了解人脸服务，请参阅[人脸文档](https://docs.microsoft.com/azure/cognitive-services/face/)。
