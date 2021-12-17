---
lab:
    title: '检测、分析和识别人脸'
    module: '模块 10 - 检测、分析和识别人脸'
---

# 检测、分析和识别人脸

检测、分析和识别人脸的能力是一项核心 AI 功能。在此练习中，你将探索两个可用于处理图像中的人脸的 Azure 认知服务： **计算机视觉**服务和**人脸**服务。

## 克隆本课程的存储库

如果尚未克隆用于本课程的存储库，请克隆它：

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行 **Git:Clone** 命令，将 `https://github.com/MicrosoftLearning/AI-102ZH-Designing-and-Implementing-a-Microsoft-Azure-AI-Solution` 存储库克隆到本地文件夹（具体克隆到哪个文件夹无关紧要）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **备注**： 如果系统提示你添加生成和调试所需的资产，请选择 **“以后再说”**。

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

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用计算机视觉 SDK 来分析图像中的人脸。

> **备注**： 可选择将该 SDK 用于 **C#** 或 **Python**。在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的 **“资源管理器”** 窗格中，浏览到 **19-face** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **computer-vision** 文件夹，并打开集成终端。然后通过运行适用于你的语言首选项的命令，安装计算机视觉 SDK 包：

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.ComputerVision --version 6.0.0
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-computervision==0.7.0
    ```
    
3. 查看 **computer-vision** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#**： appsettings.json
    - **Python**： .env

4. 打开配置文件，然后更新其中包含的配置值，以反映认知服务资源的终**结点**和身份验证**密钥**。保存更改。

5. 请注意，**computer-vision** 文件夹中包含客户端应用程序的代码文件：

    - **C#**： Program.cs
    - **Python**： detect-faces.py

6. 打开代码文件，并在顶部的现有命名空间引用下找到注释 **“导入命名空间”**。然后在此注释下添加以下特定于语言的代码，以导入使用计算机视觉 SDK 所需的命名空间：

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
    from azure.cognitiveservices.vision.computervision.models import VisualFeatureTypes
    from msrest.authentication import CognitiveServicesCredentials
    ```

## 查看要分析的图像

在此练习中，你将使用计算机视觉服务来分析人群图像。

1. 在 Visual Studio Code 中，展开 **computer-vision** 文件夹以及其中包含的 **images** 文件夹。
2. 选择并查看 **people.jpg** 图像。

## 检测图像中的人脸

现在，你可使用 SDK 来调用计算机视觉服务并检测图像中的人脸。

1. 在客户端应用程序的代码文件（**Program.cs** 或 **detect-faces.py**）中，可在 **Main** 函数中看到已提供用于加载配置设置的代码。然后查找注释 **“对计算机视觉对象客户端进行身份验证”**。然后在此注释下添加以下特定于语言的代码，以创建计算机视觉对象客户端对象并对其进行身份验证：

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

2. 在 **Main** 函数中，可在你刚刚添加的代码下看到，代码指定了图像文件的路径，然后将图像路径传递给名为 **AnalyzeFaces** 的函数。此函数尚未完全实现。

3. 在 **AnalyzeFaces** 函数的注释 **“指定要检索的特征（人脸）”** 下，添加以下代码：

    **C#**

    ```C#
    // 指定要检索的特征（人脸）
    List<VisualFeatureTypes?> features = new List<VisualFeatureTypes?>()
    {
        VisualFeatureTypes.Faces
    };
    ```

    **Python**

    ```Python
    # 指定要检索的特征（人脸）
    features = [VisualFeatureTypes.faces]
    ```

4. 在 **AnalyzeFaces** 函数的注释 **“获取图像分析”** 下，添加以下代码：

**C#**

```C
// 获取图像分析
using (var imageData = File.OpenRead(imageFile))
{    
    var analysis = await cvClient.AnalyzeImageInStreamAsync(imageData, features);

    // 获取人脸
    if (analysis.Faces.Count > 0)
    {
        Console.WriteLine($"{analysis.Faces.Count} faces detected.");

        // 准备要绘制的图像
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 3);
        SolidBrush brush = new SolidBrush(Color.LightGreen);

        // 绘制每张人脸并为其添加批注
        foreach (var face in analysis.Faces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Person aged approximately {face.Age}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // 保存已批注的图像
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}        
```

**Python**

```Python
# 获取图像分析
with open(image_file, mode="rb") as image_data:
    analysis = cv_client.analyze_image_in_stream(image_data , features)

    # 获取人脸
    if analysis.faces:
        print(len(analysis.faces), 'faces detected.')

        # 准备要绘制的图像
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # 绘制每张人脸并为其添加批注
        for face in analysis.faces:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Person aged approximately {}'.format(face.age)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # 保存已批注的图像
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('Results saved in', outputfile)
```

5. 保存你的更改并返回到 **computer-vision** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-faces.py
    ```

6. 查看输出，它应该会指出检测到的人脸数。
7. 查看在代码文件所在的同一文件夹中生成的 **detected_faces.jpg** 文件，以查看带有批注的人脸。本例的代码使用人脸特征来估计图像中每个人的年龄，并使用边界框坐标来绘制框住每张人脸的矩形。

## 准备使用人脸 SDK

虽然**计算机视觉**服务提供了基本的人脸检测功能（以及许多其他图像分析功能），但**人脸**服务提供更全面的面部分析和识别功能。

1. 在 Visual Studio Code 的 **“资源管理器”** 窗格中，浏览到 **19-face** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **face-api** 文件夹，并打开集成终端。然后通过运行适用于你的语言首选项的命令，安装人脸 SDK 包：

    **C#**

    ```
    dotnet add package Microsoft.Azure.CognitiveServices.Vision.Face --version 2.6.0-preview.1
    ```

    **Python**

    ```
    pip install azure-cognitiveservices-vision-face==0.4.1
    ```
    
3. 查看 **face-api** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#**： appsettings.json
    - **Python**： .env

4. 打开配置文件，然后更新其中包含的配置值，以反映认知服务资源的终**结点**和身份验证**密钥**。保存更改。

5. 请注意，**face-api** 文件夹中包含客户端应用程序的代码文件：

    - **C#**： Program.cs
    - **Python**： analyze-faces.py

6. 打开代码文件，并在顶部的现有命名空间引用下找到注释 **“导入命名空间”**。然后在此注释下添加以下特定于语言的代码，以导入使用计算机视觉 SDK 所需的命名空间：

    **C#**

    ```C#
    // 导入命名空间
    using Microsoft.Azure.CognitiveServices.Vision.Face;
    using Microsoft.Azure.CognitiveServices.Vision.Face.Models;
    ```

    **Python**

    ```Python
    # 导入命名空间
    from azure.cognitiveservices.vision.face import FaceClient
    from azure.cognitiveservices.vision.face.models import FaceAttributeType
    from msrest.authentication import CognitiveServicesCredentials
    ```

7. 请注意，**Main** 函数中已提供用于加载配置设置的代码。然后查找注释 **“对人脸客户端进行身份验证”**。然后在此注释下添加以下特定于语言的代码，以创建 **FaceClient** 对象并对其进行身份验证：

    **C#**

    ```C#
    // 对人脸客户端进行身份验证
    ApiKeyServiceClientCredentials credentials = new ApiKeyServiceClientCredentials(cogSvcKey);
    faceClient = new FaceClient(credentials)
    {
        Endpoint = cogSvcEndpoint
    };
    ```

    **Python**

    ```Python
    # 对人脸客户端进行身份验证
    credentials = CognitiveServicesCredentials(cog_key)
    face_client = FaceClient(cog_endpoint, credentials)
    ```

8. 在 **Main** 函数中，可在你刚刚添加的代码下看到，代码显示了一个菜单，可通过该菜单调用代码中的函数以探索人脸服务的功能。你将在此练习的剩余部分中实现这些函数。

## 检测和分析人脸

人脸服务最基本的功能之一是检测图像中的人脸并确定其特征（例如年龄、情绪表达、头发颜色以及是否佩戴眼镜等等）。

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **1** 时运行的代码。此代码会调用 **DetectFaces** 函数并传递图像文件的路径。
2. 在代码文件中查找 **DetectFaces** 函数，并在注释 **“指定要检索的面部特征”** 下添加以下代码：

    **C#**

    ```C#
    // 指定要检索的面部特征
    List<FaceAttributeType?> features = new List<FaceAttributeType?>
    {
        FaceAttributeType.Age,
        FaceAttributeType.Emotion,
        FaceAttributeType.Glasses
    };
    ```

    **Python**

    ```Python
    # 指定要检索的面部特征
    features = [FaceAttributeType.age,
                FaceAttributeType.emotion,
                FaceAttributeType.glasses]
    ```

3. 在 **DetectFaces** 函数中刚刚添加的代码下，查找注释 **“获取人脸”** 并添加以下代码：

**C#**

```C
// 获取人脸
using (var imageData = File.OpenRead(imageFile))
{    
    var detected_faces = await faceClient.Face.DetectWithStreamAsync(imageData, returnFaceAttributes: features);

    if (detected_faces.Count > 0)
    {
        Console.WriteLine($"{detected_faces.Count} faces detected.");

        // 准备要绘制的图像
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Black);

        // 绘制每张人脸并为其添加批注
        foreach (var face in detected_faces)
        {
            // 获取人脸特征
            Console.WriteLine($"\nFace ID: {face.FaceId}");
            Console.WriteLine($" - Age: {face.FaceAttributes.Age}");
            Console.WriteLine($" - Emotions:");
            foreach (var emotion in face.FaceAttributes.Emotion.ToRankedList())
            {
                Console.WriteLine($"   - {emotion}");
            }

            Console.WriteLine($" - Glasses: {face.FaceAttributes.Glasses}");

            // 绘制人脸并为其添加批注
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face ID: {face.FaceId}";
            graphics.DrawString(annotation,font,brush,r.Left, r.Top);
        }

        // 保存已批注的图像
        String output_file = "detected_faces.jpg";
        image.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# 获取人脸
with open(image_file, mode="rb") as image_data:
    detected_faces = face_client.face.detect_with_stream(image=image_data,
                                                            return_face_attributes=features)

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # 准备要绘制的图像
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'

        # 绘制每张人脸并为其添加批注
        for face in detected_faces:

            # 获取人脸特征
            print('\nFace ID: {}'.format(face.face_id))
            detected_attributes = face.face_attributes.as_dict()
            age = 'age unknown' if 'age' not in detected_attributes.keys() else int(detected_attributes['age'])
            print(' - Age: {}'.format(age))

            if 'emotion' in detected_attributes:
                print(' - Emotions:')
                for emotion_name in detected_attributes['emotion']:
                    print('   - {}: {}'.format(emotion_name, detected_attributes['emotion'][emotion_name]))
            
            if 'glasses' in detected_attributes:
                print(' - Glasses:{}'.format(detected_attributes['glasses']))

            # 绘制人脸并为其添加批注
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face ID: {}'.format(face.face_id)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # 保存已批注的图像
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. 检查添加到 **DetectFaces** 函数的代码。该代码分析了图像文件，并检测了其中包含的任何人脸及其特征（年龄、情绪以及是否佩戴眼镜）。该代码还会显示每张人脸的详细信息（包括为每张人脸分配的唯一人脸标识符）；并使用边界框在图像上标出了人脸所在的位置。
5. 保存你的更改并返回到 **face-api** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    *C# 输出可能显示有关异步函数在使用 **await** 运算符的警告。可以忽略该警告。*

    **Python**

    ```
    python analyze-faces.py
    ```

6. 在出现提示时输入 **1** 并观察输出，输出中应包含检测到的每张人脸的 ID 和特征。
7. 查看在代码文件所在的同一文件夹中生成的 **detected_faces.jpg** 文件，以查看带有批注的人脸。

## 比较人脸

一个常见的任务是比较人脸，并查找相似的人脸。在此例中，你不需要*确定*图像中每个人的身份，而只需确定是否有多张图像显示了*同一*人（或者至少是外表相似的人）。 

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **2** 时运行的代码。此代码会调用 **CompareFaces** 函数并传递两个图像文件（**person1.jpg** 和 **people.jpg**）的路径。
2. 在代码文件中查找 **CompareFaces** 函数，并在用于在控制台中显示消息的现有代码下添加以下代码：

**C#**

```C
// 确定图像 1 中的人脸是否也出现在图像 2 中
DetectedFace image_i_face;
using (var image1Data = File.OpenRead(image1))
{    
    // 获取图像 1 中的第一张人脸
    var image1_faces = await faceClient.Face.DetectWithStreamAsync(image1Data);
    if (image1_faces.Count > 0)
    {
        image_i_face = image1_faces[0];
        Image img1 = Image.FromFile(image1);
        Graphics graphics = Graphics.FromImage(img1);
        Pen pen = new Pen(Color.LightGreen, 3);
        var r = image_i_face.FaceRectangle;
        Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        String output_file = "face_to_match.jpg";
        img1.Save(output_file);
        Console.WriteLine(" Results saved in " + output_file); 

        // 获取图像 2 中的所有人脸
        using (var image2Data = File.OpenRead(image2))
        {    
            var image2Faces = await faceClient.Face.DetectWithStreamAsync(image2Data);

            // 获取人脸
            if (image2Faces.Count > 0)
            {

                var image2FaceIds = image2Faces.Select(f => f.FaceId).ToList<Guid?>();
                var similarFaces = await faceClient.Face.FindSimilarAsync((Guid)image_i_face.FaceId,faceIds:image2FaceIds);
                var similarFaceIds = similarFaces.Select(f => f.FaceId).ToList<Guid?>();

                // 准备要绘制的图像
                Image img2 = Image.FromFile(image2);
                Graphics graphics2 = Graphics.FromImage(img2);
                Pen pen2 = new Pen(Color.LightGreen, 3);
                Font font2 = new Font("Arial", 4);
                SolidBrush brush2 = new SolidBrush(Color.Black);

                // 绘制每张人脸并为其添加批注
                foreach (var face in image2Faces)
                {
                    if (similarFaceIds.Contains(face.FaceId))
                    {
                        // 绘制人脸并为其添加批注
                        var r2 = face.FaceRectangle;
                        Rectangle rect2 = new Rectangle(r2.Left, r2.Top, r2.Width, r2.Height);
                        graphics2.DrawRectangle(pen2, rect2);
                        string annotation = "Match!";
                        graphics2.DrawString(annotation,font2,brush2,r2.Left, r2.Top);
                    }
                }

                // 保存已批注的图像
                String output_file2 = "matched_faces.jpg";
                img2.Save(output_file2);
                Console.WriteLine(" Results saved in " + output_file2);   
            }
        }

    }
}
```

**Python**

```Python
# 确定图像 1 中的人脸是否也出现在图像 2 中
with open(image_1, mode="rb") as image_data:
    # 获取图像 1 中的第一张人脸
    image_1_faces = face_client.face.detect_with_stream(image=image_data)
    image_1_face = image_1_faces[0]

    # 突出显示图像中的人脸
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_1)
    draw = ImageDraw.Draw(image)
    color = 'lightgreen'
    r = image_1_face.face_rectangle
    bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
    draw = ImageDraw.Draw(image)
    draw.rectangle(bounding_box, outline=color, width=5)
    plt.imshow(image)
    outputfile = 'face_to_match.jpg'
    fig.savefig(outputfile)

# 获取图像 2 中的所有人脸
with open(image_2, mode="rb") as image_data:
    image_2_faces = face_client.face.detect_with_stream(image=image_data)
    image_2_face_ids = list(map(lambda face: face.face_id, image_2_faces))

    # 在图像 2 中查找与图像 1 中的人脸相似的人脸
    similar_faces = face_client.face.find_similar(face_id=image_1_face.face_id, face_ids=image_2_face_ids)
    similar_face_ids = list(map(lambda face: face.face_id, similar_faces))

    # 准备要绘制的图像
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_2)
    draw = ImageDraw.Draw(image)

    # 绘制匹配的人脸并为其添加批注
    for face in image_2_faces:
        if face.face_id in similar_face_ids:
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline='lightgreen', width=10)
            plt.annotate('Match!',(r.left, r.top + r.height + 15), backgroundcolor='white')

    # 保存已批注的图像
    plt.imshow(image)
    outputfile = 'matched_faces.jpg'
    fig.savefig(outputfile)
```

3. 检查添加到 **CompareFaces** 函数的代码。该代码可查找图像 1 中的第一张人脸，并在名为 **face_to_match.jpg** 的新图像文件中对其进行批注。然后该代码会查找图像 2 中的所有人脸，并使用其人脸 ID 查找与图像 1 中相似的人脸。相似的人脸会经过批注，并保存在名为 **matched_faces.jpg** 的新图像中。
4. 保存你的更改并返回到 **face-api** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    *C# 输出可能显示有关异步函数在使用 **await** 运算符的警告。可以忽略该警告。*

    **Python**

    ```
    python analyze-faces.py
    ```
    
5. 在出现提示时输入 **2** 并观察输出。然后查看在代码文件所在的同一文件夹中生成的 **face_to_match.jpg** 和 **matched_faces.jpg** 文件，以查看匹配的人脸。
6. 编辑 **Main** 函数中菜单选项 **2** 对应的代码，以将 **person2.jpg** 与 **people.jpg** 进行比较，然后重新运行程序并查看结果。

## 训练面部识别模型

在某些情况下，你需要维护特定人员的模型，并可通过 AI 应用程序识别这些人员的人脸。例如，帮助使用面部识别技术的生物识别安全系统验证安全建筑中员工的身份。

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **3** 时运行的代码。此代码会调用 **TrainModel** 函数，并传递将在认知服务资源中注册的新 **PersonGroup** 的名称和 ID，以及员工姓名列表。
2. 查看 **face-api/images** 文件夹中是否包含与员工同名的文件夹。每个文件夹中都包含多个指定员工的图像。
3. 在代码文件中查找 **TrainModel** 函数，并在用于在控制台中显示消息的现有代码下添加以下代码：

**C#**

```C
// 删除已存在的组
var groups = await faceClient.PersonGroup.ListAsync();
foreach(var group in groups)
{
    if (group.PersonGroupId == groupId)
    {
        await faceClient.PersonGroup.DeleteAsync(groupId);
    }
}

// 创建组
await faceClient.PersonGroup.CreateAsync(groupId, groupName);
Console.WriteLine("Group created!");

// 将每个人添加到组中
Console.Write("Adding people to the group...");
foreach(var personName in imageFolders)
{
    // 添加人员
    var person = await faceClient.PersonGroupPerson.CreateAsync(groupId, personName);

    // 添加某人的多张照片
    string[] images = Directory.GetFiles("images/" + personName);
    foreach(var image in images)
    {
        using (var imageData = File.OpenRead(image))
        { 
            await faceClient.PersonGroupPerson.AddFaceFromStreamAsync(groupId, person.PersonId, imageData);
        }
    }

}

    // 训练模型
Console.WriteLine("Training model...");
await faceClient.PersonGroup.TrainAsync(groupId);

// 获取组中的人员列表
Console.WriteLine("Facial recognition model trained with the following people:");
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    Console.WriteLine($"-{person.Name}");
}
```

**Python**

```Python
# 删除已存在的组
groups = face_client.person_group.list()
for group in groups:
    if group.person_group_id == group_id:
        face_client.person_group.delete(group_id)

# 创建组
face_client.person_group.create(group_id, group_name)
print ('Group created!')

# 将每个人添加到组中
print('Adding people to the group...')
for person_name in image_folders:
    # 添加人员
    person = face_client.person_group_person.create(group_id, person_name)

    # 添加某人的多张照片
    folder = os.path.join('images', person_name)
    person_pics = os.listdir(folder)
    for pic in person_pics:
        img_path = os.path.join(folder, pic)
        img_stream = open(img_path, "rb")
        face_client.person_group_person.add_face_from_stream(group_id, person.person_id, img_stream)

# 训练模型
print('Training model...')
face_client.person_group.train(group_id)

# 获取组中的人员列表
print('Facial recognition model trained with the following people:')
people = face_client.person_group_person.list(group_id)
for person in people:
    print('-', person.name)
```

4. 检查添加到 **TrainModel** 函数的代码。该代码会执行以下任务：
    - 获取服务中注册的 **PersonGroup** 的列表，并删除指定的组（如果存在）。
    - 使用指定的 ID 和姓名创建一个组。
    - 针对指定的每个姓名向组中添加一位人员，并添加每个人的多张图像。
    - 根据组中的指定人员及其人脸图像训练面部识别模型。
    - 在组中检索指定人员列表并显示他们。
5. 保存你的更改并返回到 **face-api** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    *C# 输出可能显示有关异步函数在使用 **await** 运算符的警告。可以忽略该警告。*

    **Python**

    ```
    python analyze-faces.py
    ```

6. 在出现提示时输入 **3** 并观察输出，注意到所创建的 **PersonGroup** 中包含两人。

## 识别人脸

至此，已定义 **PeopleGroup** 并训练了一个面部识别模型，现在可使用该模型在图像中识别指定人员。

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **4** 时运行的代码。此代码会调用 **RecognizeFaces** 函数，并传递图像文件 (**people.jpg**) 的路径以及要用于人脸识别的 **PeopleGroup** 的 ID。
2. 在代码文件中查找 **RecognizeFaces** 函数，并在用于在控制台中显示消息的现有代码下添加以下代码：

**C#**

```C
// 检测图像中的人脸
using (var imageData = File.OpenRead(imageFile))
{    
    var detectedFaces = await faceClient.Face.DetectWithStreamAsync(imageData);

    // 获取人脸
    if (detectedFaces.Count > 0)
    {
        
        // 获取人脸 ID 列表
        var faceIds = detectedFaces.Select(f => f.FaceId).ToList<Guid?>();

        // 识别人群组中的人脸
        var recognizedFaces = await faceClient.Face.IdentifyAsync(faceIds, groupId);

        // 获取已识别的人脸对应的姓名
        var faceNames = new Dictionary<Guid?, string>();
        if (recognizedFaces.Count> 0)
        {
            foreach(var face in recognizedFaces)
            {
                var person = await faceClient.PersonGroupPerson.GetAsync(groupId, face.Candidates[0].PersonId);
                Console.WriteLine($"-{person.Name}");
                faceNames.Add(face.FaceId, person.Name);

            }
        }

        
        // 在图像中批注人脸
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen penYes = new Pen(Color.LightGreen, 3);
        Pen penNo = new Pen(Color.Magenta, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.Cyan);
        foreach (var face in detectedFaces)
        {
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            if (faceNames.ContainsKey(face.FaceId))
            {
                // 如果已识别人脸，则为其批注绿色的姓名
                graphics.DrawRectangle(penYes, rect);
                string personName = faceNames[face.FaceId];
                graphics.DrawString(personName,font,brush,r.Left, r.Top);
            }
            else
            {
                // 否则，仅使用品红色批注未识别的人脸
                graphics.DrawRectangle(penNo, rect);
            }
        }

        // 保存已批注的图像
        String output_file = "recognized_faces.jpg";
        image.Save(output_file);
        Console.WriteLine("Results saved in " + output_file);   
    }
}
```

**Python**

```Python
# 检测图像中的人脸
with open(image_file, mode="rb") as image_data:

    # 获取人脸
    detected_faces = face_client.face.detect_with_stream(image=image_data)

    # 获取人脸 ID 列表
    face_ids = list(map(lambda face: face.face_id, detected_faces))

    # 识别人群组中的人脸
    recognized_faces = face_client.face.identify(face_ids, group_id)

    # 获取已识别的人脸对应的姓名
    face_names = {}
    if len(recognized_faces) > 0:
        print(len(recognized_faces), 'faces recognized.')
        for face in recognized_faces:
            person_name = face_client.person_group_person.get(group_id, face.candidates[0].person_id).name
            print('-', person_name)
            face_names[face.face_id] = person_name

    # 在图像中批注人脸
    fig = plt.figure(figsize=(8, 6))
    plt.axis('off')
    image = Image.open(image_file)
    draw = ImageDraw.Draw(image)
    for face in detected_faces:
        r = face.face_rectangle
        bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
        draw = ImageDraw.Draw(image)
        if face.face_id in face_names:
            # 如果已识别人脸，则为其批注绿色的姓名
            draw.rectangle(bounding_box, outline='lightgreen', width=3)
            plt.annotate(face_names[face.face_id],
                        (r.left, r.top + r.height + 15), backgroundcolor='white')
        else:
            # 否则，仅使用品红色批注未识别的人脸
            draw.rectangle(bounding_box, outline='magenta', width=3)

    # 保存已批注的图像
    plt.imshow(image)
    outputfile = 'recognized_faces.jpg'
    fig.savefig(outputfile)

    print('\nResults saved in', outputfile)
```
    
3. 检查添加到 **RecognizeFaces** 函数的代码。该代码会在图像中查找人脸并创建其 ID 列表。然后会使用人群组来尝试识别人脸 ID 列表中各人脸的身份。已识别的人脸会批注有已识别人员的姓名，结果保存在 **recognized_faces.jpg** 中。
4. 保存你的更改并返回到 **face-api** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    *C# 输出可能显示有关异步函数在使用 **await** 运算符的警告。可以忽略该警告。*

    **Python**

    ```
    python analyze-faces.py
    ```

5. 在出现提示时输入 **4** 并观察输出。然后查看在代码文件所在的同一文件夹中生成的 **recognized_faces.jpg** 文件，以查看已识别的人员。
6. 编辑 **Main** 函数中菜单选项 **4** 对应的代码，以识别 **people2.jpg** 中的人脸，然后重新运行程序并查看结果。应该已识别相同的人员

## 验证人脸

面部识别经常用于身份验证。使用人脸服务，可通过将图像中的人脸与另一张人脸或者 **PersonGroup** 中注册的人员进行比较来验证该人脸。

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **5** 时运行的代码。此代码会调用 **VerifyFace** 函数，并传递图像文件 (**person1.jpg**) 的路径、姓名以及要用于人脸连识别的 **PeopleGroup** 的 ID。
2. 在代码文件中查找 **VerifyFace**  函数，并在注释 **“获取人员组中的人员的 ID”** （用于显示结果的代码上方）下添加以下代码：

**C#**

```C
// 获取人员组中的人员的 ID
var people = await faceClient.PersonGroupPerson.ListAsync(groupId);
foreach(var person in people)
{
    if (person.Name == personName)
    {
        Guid personId = person.PersonId;

        // 获取图像中的第一张人脸
        using (var imageData = File.OpenRead(personImage))
        {    
            var faces = await faceClient.Face.DetectWithStreamAsync(imageData);
            if (faces.Count > 0)
            {
                Guid faceId = (Guid)faces[0].FaceId;

                // 我们获得了一张人脸和一个 ID。它们是否匹配？
                var verification = await faceClient.Face.VerifyFaceToPersonAsync(faceId, personId, groupId);
                if (verification.IsIdentical)
                {
                    result = "Verified";
                }
            }
        }
    }
}
```

**Python**

```Python
# 获取人员组中的人员的 ID
people = face_client.person_group_person.list(group_id)
for person in people:
    if person.name == person_name:
        person_id = person.person_id

        获取图像中的第一张人脸
        with open(person_image, mode="rb") as image_data:
            faces = face_client.face.detect_with_stream(image=image_data)
            if len(faces) > 0:
                face_id = faces[0].face_id

                我们获得了一张人脸和一个 ID。它们是否匹配？
                verification = face_client.face.verify_face_to_person(face_id, person_id, group_id)
                if verification.is_identical:
                    result = 'Verified'
```

3. 检查添加到 **VerifyFace** 函数的代码。它会在组中查找具有指定姓名的人员的 ID。如果该人员存在，则代码会获取图像中第一张人脸的人脸 ID。最后，如果图像中包含人脸，则代码会根据指定人员的 ID 对其进行验证。
4. 保存你的更改并返回到 **face-api** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python analyze-faces.py
    ```

5. 在出现提示时输入 **5** 并观察结果。
6. 编辑 **Main** 函数中菜单选项 **5** 对应的代码，以尝试各姓名以及图像 **person1.jpg** 和 **person2.jpg** 的不同组合。

## 更多信息

有关使用**计算机视觉**服务进行面部检测的详细信息，请参阅[计算机视觉文档](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)。

要详细了解**人脸**服务，请参阅[人脸文档](https://docs.microsoft.com/azure/cognitive-services/face/)。
