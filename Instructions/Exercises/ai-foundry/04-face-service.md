---
lab:
  title: 检测和分析人脸
  module: Module 4 - Detecting and Analyze Faces
---

# 检测和分析人脸

检测和分析人脸的能力是一项核心 AI 功能。 在此练习中，你将探索两个可用于处理图像中的人脸的 Azure AI 服务：**Azure AI 视觉**服务和**人脸**服务。

> **重要说明**：无需请求对受限功能进行任何额外访问即可完成此实验室。

> **注意**：从 2022 年 6 月 21 日开始，返回个人身份信息的 Azure AI 服务的功能仅限于被授予[有限访问权限](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)的客户。 此外，推断情绪状态的功能不再可用。 有关 Microsoft 所做的更改的更多详细信息，以及原因 - 请参阅[负责任 AI 投资和面部识别防护措施](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/)。

## 克隆本课程的存储库

如果尚未克隆用于本课程的存储库，请克隆它：

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“**Git：Clone**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-vision` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。

## 预配 Azure AI 服务资源

如果订阅中还没有，则需要预配 **Azure AI 服务**资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 在顶部搜索栏中，搜索 *Azure AI 服务*，选择Azure AI 服务****，并使用以下设置创建 Azure AI 服务多服务帐户资源：
    - **订阅**：*Azure 订阅*
    - **资源组**：*选择或创建一个资源组（如果使用受限制的订阅，你可能无权创建新的资源组 - 请使用提供的资源组）*
    - **区域**：*选择任何可用区域*
    - **名称**：*输入唯一名称*
    - **定价层**：标准版 S0
3. 选中所需的复选框并创建资源。
4. 等待部署完成，然后查看部署详细信息。
5. 部署资源后，转到该资源并查看其**密钥和终结点**页面。 你将在下一个过程中用到此页面中的终结点和其中一个密钥。

## 准备使用 Azure AI 视觉 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用 Azure AI 视觉 SDK 来分析图像中的人脸。

> **注意**：可选择将该 SDK 用于 **C#** 或 **Python**。 在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的**资源管理器**窗格中，浏览到 **04-face** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **computer-vision** 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装 Azure AI 视觉 SDK 包：

    **C#**

    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.3
    ```

    **Python**

    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b3
    ```
    
3. 查看 **computer-vision** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#** ：appsettings.json
    - **Python**：.env

4. 打开配置文件，然后更新其中包含的配置值，以反映 Azure AI 服务资源的**终结点**和身份验证**密钥**。 保存所做更改。

5. 请注意，**computer-vision** 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：detect-people.py

6. 打开代码文件，并在顶部的现有命名空间引用下找到注释**导入命名空间**。 然后在此注释下添加以下特定于语言的代码，以导入使用计算机视觉 SDK 所需的命名空间：

    **C#**

    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    ```

    **Python**

    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```

## 查看要分析的图像

在此练习中，你将使用 Azure AI 视觉服务来分析人群图像。

1. 在 Visual Studio Code 中，展开 **computer-vision** 文件夹以及其中包含的 **images** 文件夹。
2. 选择并查看 **people.jpg** 图像。

## 在图像中检测人脸

现在，你可使用 SDK 来调用 Azure AI 视觉服务并检测图像中的人脸。

1. 在客户端应用程序的代码文件（**Program.cs** 或 **detect-people.py**）中，可在 **Main** 函数中看到已提供用于加载配置设置的代码。 然后查找注释对 Azure AI 视觉客户端进行身份验证****。 然后在此注释下添加以下特定于语言的代码，以创建计算机视觉对象客户端对象并对其进行身份验证：

    **C#**

    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient cvClient = new ImageAnalysisClient(
        new Uri(aiSvcEndpoint),
        new AzureKeyCredential(aiSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Azure AI Vision client
    cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key)
    )
    ```

2. 在 **Main** 函数中，可在你刚刚添加的代码下看到，代码指定了图像文件的路径，然后将图像路径传递给名为 **AnalyzeImage** 的函数。 此函数尚未完全实现。

3. 在 **AnalyzeImage** 函数中的注释**获取具有要检索的指定特征的结果（人物）** 下，添加以下代码：

    **C#**

    ```C#
    // Get result with specified features to be retrieved (PEOPLE)
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        VisualFeatures.People);
    ```

    **Python**

    ```Python
    # Get result with specified features to be retrieved (PEOPLE)
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[
            VisualFeatures.PEOPLE],
    )
    ```

4. 在 **AnalyzeImage** 函数中的注释**在检测到的人物周围绘制边界框**下，添加以下代码：

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
        }

        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }
    ```

    **Python**
    
    ```Python
    # Draw bounding box around detected people
    for detected_people in result.people.list:
        if(detected_people.confidence > 0.5):
            # Draw object bounding box
            r = detected_people.bounding_box
            bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
            draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
    ```

5. 保存你的更改并返回到 **computer-vision** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. 查看输出，它应该会指出检测到的人脸数。
7. 查看在代码文件所在的同一文件夹中生成的 **people.jpg** 文件，以查看带有批注的人脸。 本例的代码使用人脸特征来标记框左上角的位置，并使用边界框坐标来绘制框住每张人脸的矩形。

如果希望看到该服务检测到的所有人员的置信度分数，可以在注释 `Return the confidence of the person detected` 下取消注释代码行并重新运行代码。

## 准备使用人脸 SDK

虽然 **Azure AI 视觉**服务提供了基本的人脸检测功能（以及许多其他图像分析功能），但**人脸**服务提供更全面的面部分析和识别功能。

1. 在 Visual Studio Code 的**资源管理器**窗格中，浏览到 **04-face** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **face-api** 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装人脸 SDK 包：

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
3. 查看 **face-api** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#** ：appsettings.json
    - **Python**：.env

4. 打开配置文件，然后更新其中包含的配置值，以反映 Azure AI 服务资源的**终结点**和身份验证**密钥**。 保存所做更改。

5. 请注意，**face-api** 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：analyze-faces.py

6. 打开代码文件，并在顶部的现有命名空间引用下找到注释**导入命名空间**。 然后在此注释下添加以下特定于语言的代码，以导入使用视觉 SDK 所需的命名空间：

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

7. 请注意，**Main** 函数中已提供用于加载配置设置的代码。 然后查找注释**对人脸客户端进行身份验证**。 然后在此注释下添加以下特定于语言的代码，以创建 **FaceClient** 对象并对其进行身份验证：

    **C#**

    ```C#
    // Authenticate Face client
    faceClient = new FaceClient(
        new Uri(cogSvcEndpoint),
        new AzureKeyCredential(cogSvcKey));
    ```

    **Python**

    ```Python
    # Authenticate Face client
    face_client = FaceClient(
        endpoint=cog_endpoint,
        credential=AzureKeyCredential(cog_key)
    )
    ```

8. 在 **Main** 函数中，可在你刚刚添加的代码下看到，代码显示了一个菜单，可通过该菜单调用代码中的函数以探索人脸服务的功能。 你将在此练习的剩余部分中实现这些函数。

## 检测和分析人脸

人脸服务最基本的功能之一是检测图像中的人脸并确定其特征（例如头部姿势、模糊、是否佩戴口罩等）。

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **1** 时运行的代码。 此代码会调用 **DetectFaces** 函数并传递图像文件的路径。
2. 在代码文件中查找 **DetectFaces** 函数，并在注释**指定要检索的面部特征**下添加以下代码：

    **C#**

    ```C#
    // Specify facial features to be retrieved
    FaceAttributeType[] features = new FaceAttributeType[]
    {
        FaceAttributeType.Detection03.HeadPose,
        FaceAttributeType.Detection03.Blur,
        FaceAttributeType.Detection03.Mask
    };
    ```

    **Python**

    ```Python
    # Specify facial features to be retrieved
    features = [FaceAttributeTypeDetection03.HEAD_POSE,
                FaceAttributeTypeDetection03.BLUR,
                FaceAttributeTypeDetection03.MASK]
    ```

3. 在 **DetectFaces** 函数中刚刚添加的代码下，查找注释**获取人脸**并添加以下代码：

**C#**

```C#
// Get faces
using (var imageData = File.OpenRead(imageFile))
{    
    var response = await faceClient.DetectAsync(
        BinaryData.FromStream(imageData),
        FaceDetectionModel.Detection03,
        FaceRecognitionModel.Recognition04,
        returnFaceId: false,
        returnFaceAttributes: features);
    IReadOnlyList<FaceDetectionResult> detected_faces = response.Value;

    if (detected_faces.Count() > 0)
    {
        Console.WriteLine($"{detected_faces.Count()} faces detected.");

        // Prepare image for drawing
        Image image = Image.FromFile(imageFile);
        Graphics graphics = Graphics.FromImage(image);
        Pen pen = new Pen(Color.LightGreen, 3);
        Font font = new Font("Arial", 4);
        SolidBrush brush = new SolidBrush(Color.White);
        int faceCount=0;

        // Draw and annotate each face
        foreach (var face in detected_faces)
        {
            faceCount++;
            Console.WriteLine($"\nFace number {faceCount}");
            
            // Get face properties
            Console.WriteLine($" - Head Pose (Yaw): {face.FaceAttributes.HeadPose.Yaw}");
            Console.WriteLine($" - Head Pose (Pitch): {face.FaceAttributes.HeadPose.Pitch}");
            Console.WriteLine($" - Head Pose (Roll): {face.FaceAttributes.HeadPose.Roll}");
            Console.WriteLine($" - Blur: {face.FaceAttributes.Blur.BlurLevel}");
            Console.WriteLine($" - Mask: {face.FaceAttributes.Mask.Type}");

            // Draw and annotate face
            var r = face.FaceRectangle;
            Rectangle rect = new Rectangle(r.Left, r.Top, r.Width, r.Height);
            graphics.DrawRectangle(pen, rect);
            string annotation = $"Face number {faceCount}";
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
    detected_faces = face_client.detect(
        image_content=image_data.read(),
        detection_model=FaceDetectionModel.DETECTION03,
        recognition_model=FaceRecognitionModel.RECOGNITION04,
        return_face_id=False,
        return_face_attributes=features,
    )

    if len(detected_faces) > 0:
        print(len(detected_faces), 'faces detected.')

        # Prepare image for drawing
        fig = plt.figure(figsize=(8, 6))
        plt.axis('off')
        image = Image.open(image_file)
        draw = ImageDraw.Draw(image)
        color = 'lightgreen'
        face_count = 0

        # Draw and annotate each face
        for face in detected_faces:

            # Get face properties
            face_count += 1
            print('\nFace number {}'.format(face_count))

            print(' - Head Pose (Yaw): {}'.format(face.face_attributes.head_pose.yaw))
            print(' - Head Pose (Pitch): {}'.format(face.face_attributes.head_pose.pitch))
            print(' - Head Pose (Roll): {}'.format(face.face_attributes.head_pose.roll))
            print(' - Blur: {}'.format(face.face_attributes.blur.blur_level))
            print(' - Mask: {}'.format(face.face_attributes.mask.type))

            # Draw and annotate face
            r = face.face_rectangle
            bounding_box = ((r.left, r.top), (r.left + r.width, r.top + r.height))
            draw = ImageDraw.Draw(image)
            draw.rectangle(bounding_box, outline=color, width=5)
            annotation = 'Face number {}'.format(face_count)
            plt.annotate(annotation,(r.left, r.top), backgroundcolor=color)

        # Save annotated image
        plt.imshow(image)
        outputfile = 'detected_faces.jpg'
        fig.savefig(outputfile)

        print('\nResults saved in', outputfile)
```

4. 检查添加到 **DetectFaces** 函数的代码。 它分析图像文件，并检测其中包含的任何人脸及其特征（头部姿势、模糊以及是否佩戴口罩）。 该代码还会显示每张人脸的详细信息（包括为每张人脸分配的唯一人脸标识符）；并使用边界框在图像上标出了人脸所在的位置。
5. 保存你的更改并返回到 **face-api** 文件夹的集成终端，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    *C# 输出可能显示有关异步函数未使用 **await** 运算符的警告。可以忽略这些警告。*

    **Python**

    ```
    python analyze-faces.py
    ```

6. 在出现提示时输入 **1** 并观察输出，输出中应包含检测到的每张人脸的 ID 和特征。
7. 查看在代码文件所在的同一文件夹中生成的 **detected_faces.jpg** 文件，以查看带有批注的人脸。

## 详细信息

**人脸**服务中提供了几个附加功能，但根据[负责任 AI 标准](https://aka.ms/aah91ff)，这些功能受限于有限访问策略。 这些功能包括识别、验证和创建面部识别模型。 若要了解详细信息并申请访问权限，请参阅[Azure AI 服务的有限访问](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access)。

有关使用 **Azure AI 视觉**服务进行面部检测的详细信息，请参阅[Azure AI 视觉文档](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)。

要详细了解**人脸**服务，请参阅[人脸文档](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity)。
