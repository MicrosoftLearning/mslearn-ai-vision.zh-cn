---
lab:
  title: 检测和分析人脸
  module: Module 4 - Detecting and Analyze Faces
---

# 检测和分析人脸

检测和分析人脸的能力是一项核心 AI 功能。 在此练习中，你将探索两个可用于处理图像中的人脸的 Azure AI 服务：**Azure AI 视觉**服务和**人脸**服务。

> **重要说明**：无需请求对受限功能进行任何额外访问即可完成此实验室。

> **注意**：从 2022 年 6 月 21 日开始，返回个人身份信息的 Azure AI 服务的功能仅限于被授予[有限访问权限](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-limited-access)的客户。 此外，推断情绪状态的功能不再可用。 有关 Microsoft 所做的更改的更多详细信息，以及原因 - 请参阅[负责任 AI 投资和面部识别防护措施](https://azure.microsoft.com/blog/responsible-ai-investments-and-safeguards-for-facial-recognition/)。

## 预配计算机视觉资源

如果订阅中还没有计算机视觉资源，则需要预配计算机视觉资源****。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
1. 选择“创建资源”。****
1. 在搜索栏中，搜索“计算机视觉”**，选择“计算机视觉”****，然后使用以下设置创建资源：
    - **订阅**：*Azure 订阅*
    - **资源组**：*选择或创建一个资源组（如果使用受限制的订阅，你可能无权创建新的资源组 - 请使用提供的资源组）*
    - **区域**：*从美国东部、美国西部、法国中部、韩国中部、北欧、东南亚、西欧或东亚中选择\**
    - **名称**：*输入唯一名称*
    - **定价层**：免费 F0

    \*Azure AI 视觉 4.0 完整功能集目前仅在这些区域提供。

1. 选中所需的复选框并创建资源。
1. 等待部署完成，然后查看部署详细信息。
1. 部署资源后，转到该资源并查看其**密钥和终结点**页面。 你将在下一个过程中用到此页面中的终结点和其中一个密钥。

## 克隆本课程的存储库

你将在 Azure 门户中使用 Cloud Shell 开发代码。 你的应用的代码文件已在 GitHub 存储库中提供。

> **提示**：如果最近克隆了 mslearn-ai-vision**** 存储库，则可以跳过此任务。 否则，请按照以下步骤将其克隆到开发环境中。

1. 在 Azure 门户中，使用页面顶部搜索栏右侧的 **[\>_]** 按钮，在 Azure 门户中创建新的 Cloud Shell，选择 ***PowerShell*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    > **提示**：将命令粘贴到 cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 在 PowerShell 窗格中，输入以下命令以克隆包含此练习的 GitHub 存储库：

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. 克隆存储库后，导航到包含应用程序代码文件的文件夹：  

    ```
   cd mslearn-ai-vision/Labfiles/04-face
    ```

## 准备使用 Azure AI 视觉 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用计算机视觉 SDK 来分析图像。

> **注意**：可选择将该 SDK 用于 **C#** 或 **Python**。 在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 导航到包含以你的首选语言显示的应用程序代码文件的文件夹：  

    **C#**

    ```
   cd C-Sharp/computer-vision
    ```
    
    **Python**

    ```
   cd Python/computer-vision
    ```

1. 以你首选的语言运行相应的命令，安装 Azure AI 视觉 SDK 包和必需的依赖项：

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0
    dotnet add package SkiaSharp --version 3.116.1
    dotnet add package SkiaSharp.NativeAssets.Linux --version 3.116.1
    ``` 

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0
    pip install dotenv
    pip install matplotlib
    ```

1. 使用 `ls` 命令可以查看 computer-vision **** 文件夹的内容。 请注意，其中包含用于配置设置的文件：

    - **C#** ：appsettings.json
    - **Python**：.env

1. 输入以下命令以编辑已提供的配置文件：

    **C#**

    ```
   code appsettings.json
    ```

    **Python**

    ```
   code .env
    ```

    该文件已在代码编辑器中打开。

1. 在代码文件中，更新其中包含的配置值，以反映计算机视觉资源的“终结点”**** 和身份验证“密钥”****。
1. 替换占位符后，使用 **Ctrl+S** 命令保存更改，然后使用 **Ctrl+Q** 命令关闭代码编辑器，同时使 Cloud Shell 命令行保持打开状态。

## 查看要分析的图像

在此练习中，你将使用 Azure AI 视觉服务来分析人群图像。

1. 在 Cloud Shell 工具栏中，依次选择“上传/下载文件”、“下载”。******** 在“新建”对话框中，键入下列文件路径，然后选择“下载”：****

    ```
    mslearn-ai-vision/Labfiles/04-face/C-Sharp/computer-vision/images/people.jpg
    ```

1. 打开并查看 people.jpg**** 图像。

## 在图像中检测人脸

现在，你可使用 SDK 来调用 Azure AI 视觉服务并检测图像中的人脸。

1. 请注意，**computer-vision** 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：detect-people.py

1. 输入以下命令，以打开客户端应用程序的代码文件：

    **C#**

    ```
   code Program.cs
    ```

    **Python**

    ```
   code image-analysis.py
    ```

1. 然后在注释“导入命名空间”下添加以下特定于语言的代码，以导入使用 Azure AI 视觉 SDK 所需的命名空间：****

    **C#**
    
    ```C#
    // Import namespaces
    using Azure.AI.Vision.ImageAnalysis;
    using SkiaSharp;
    ```
    
    **Python**
    
    ```Python
    # import namespaces
    from azure.ai.vision.imageanalysis import ImageAnalysisClient
    from azure.ai.vision.imageanalysis.models import VisualFeatures
    from azure.core.credentials import AzureKeyCredential
    ```
    
1. 请注意，**Main** 函数中已提供用于加载配置设置的代码。 然后查找注释对 Azure AI 视觉客户端进行身份验证****。 然后在此注释下添加以下特定于语言的代码，以创建计算机视觉对象客户端对象并对其进行身份验证：

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

1. 在 **Main** 函数中，可在你刚刚添加的代码下看到，代码指定了图像文件的路径，然后将图像路径传递给名为 **AnalyzeImage** 的函数。 此函数尚未完全实现。

1. 在 **AnalyzeImage** 函数中的注释**获取具有要检索的指定特征的结果（人物）** 下，添加以下代码：

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

1. 在 **AnalyzeImage** 函数中的注释**在检测到的人物周围绘制边界框**下，添加以下代码：

    **C#**

    ```C
    // Draw bounding box around detected people
    foreach (DetectedPerson person in result.People.Values)
    {
        if (person.Confidence > 0.5) 
        {
            // Draw object bounding box
            var r = person.BoundingBox;
            SKRect rect = new SKRect(r.X, r.Y, r.X + r.Width, r.Y + r.Height);
            canvas.DrawRect(rect, paint);
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

1. 保存更改，关闭代码编辑器，输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    **Python**

    ```
    python detect-people.py
    ```

6. 查看输出，它应该会指出检测到的人脸数。
7. 下载在代码文件所在的同一文件夹中生成的 people.jpg 文件，以查看带有批注的人脸。**** 本例的代码使用人脸特征来标记框左上角的位置，并使用边界框坐标来绘制框住每张人脸的矩形。

如果希望看到该服务检测到的所有人员的置信度分数，可以在注释 `Return the confidence of the person detected` 下取消注释代码行并重新运行代码。

## 准备使用人脸 SDK

虽然 **Azure AI 视觉**服务提供了基本的人脸检测功能（以及许多其他图像分析功能），但**人脸**服务提供更全面的面部分析和识别功能。

1. 运行命令 `cd ../face-api` 以导航到 face-api**** 文件夹。 然后通过运行适用于你的语言首选项的命令，安装人脸 SDK 包：

    **C#**

    ```
    dotnet add package Azure.AI.Vision.Face -v 1.0.0-beta.2
    ```

    **Python**

    ```
    pip install azure-ai-vision-face==1.0.0b2
    ```
    
1. 查看 **face-api** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#** ：appsettings.json
    - **Python**：.env

1. 打开配置文件，然后更新其中包含的配置值，以反映 Azure AI 服务资源的**终结点**和身份验证**密钥**。 保存所做更改。

1. 请注意，**face-api** 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：analyze-faces.py

1. 打开代码文件，并在顶部的现有命名空间引用下找到注释**导入命名空间**。 然后在此注释下添加以下特定于语言的代码，以导入使用视觉 SDK 所需的命名空间：

    **C#**

    ```C#
    // Import namespaces
    using Azure;
    using Azure.AI.Vision.Face;
    using SkiaSharp;
    ```

    **Python**

    ```Python
    # Import namespaces
    from azure.ai.vision.face import FaceClient
    from azure.ai.vision.face.models import FaceDetectionModel, FaceRecognitionModel, FaceAttributeTypeDetection03
    from azure.core.credentials import AzureKeyCredential
    ```

1. 请注意，**Main** 函数中已提供用于加载配置设置的代码。 然后查找注释**对人脸客户端进行身份验证**。 然后在此注释下添加以下特定于语言的代码，以创建 **FaceClient** 对象并对其进行身份验证：

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

1. 在 **Main** 函数中，可在你刚刚添加的代码下看到，代码显示了一个菜单，可通过该菜单调用代码中的函数以探索人脸服务的功能。 你将在此练习的剩余部分中实现这些函数。

## 检测和分析人脸

人脸服务最基本的功能之一是检测图像中的人脸并确定其特征（例如头部姿势、模糊、是否佩戴口罩等）。

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **1** 时运行的代码。 此代码会调用 **DetectFaces** 函数并传递图像文件的路径。
1. 在代码文件中查找 **DetectFaces** 函数，并在注释**指定要检索的面部特征**下添加以下代码：

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

1. 在 **DetectFaces** 函数中刚刚添加的代码下，查找注释**获取人脸**并添加以下代码：

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

            // Load the image using SkiaSharp
            using SKBitmap bitmap = SKBitmap.Decode(imageFile);
            using SKCanvas canvas = new SKCanvas(bitmap);

            // Set up paint styles for drawing
            SKPaint rectPaint = new SKPaint
            {
                Color = SKColors.LightGreen,
                StrokeWidth = 3,
                Style = SKPaintStyle.Stroke,
                IsAntialias = true
            };

            SKPaint textPaint = new SKPaint
            {
                Color = SKColors.White,
                TextSize = 16,
                IsAntialias = true
            };

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

                // Create an SKRect from the face rectangle data
                SKRect rect = new SKRect(r.Left, r.Top, r.Left + r.Width, r.Top + r.Height);
                canvas.DrawRect(rect, rectPaint);

                string annotation = $"Face number {faceCount}";
                canvas.DrawText(annotation, r.Left, r.Top, textPaint);
            }

            // Save annotated image
            using (SKFileWStream output = new SKFileWStream("detected_faces.jpg"))
            {
                bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
            }

            Console.WriteLine(" Results saved in detected_faces.jpg");   
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

1. 检查添加到 **DetectFaces** 函数的代码。 它分析图像文件，并检测其中包含的任何人脸及其特征（头部姿势、模糊以及是否佩戴口罩）。 该代码还会显示每张人脸的详细信息（包括为每张人脸分配的唯一人脸标识符）；并使用边界框在图像上标出了人脸所在的位置。
1. 保存更改并关闭代码编辑器，然后输入以下命令以运行程序：

    **C#**

    ```
    dotnet run
    ```

    *C# 输出可能显示有关异步函数未使用 **await** 运算符的警告。可以忽略这些警告。*

    **Python**

    ```
    python analyze-faces.py
    ```

1. 在出现提示时输入 **1** 并观察输出，输出中应包含检测到的每张人脸的 ID 和特征。
1. 下载在代码文件所在的同一文件夹中生成的 detected_faces.jpg 文件，以查看带有批注的人脸。****

## 清理资源

如果不将本实验室创建的 Azure 资源用于其他培训模块，则可以删除这些资源以避免产生更多费用。

1. 打开 Azure 门户网站 `https://portal.azure.com`，在顶部搜索栏中搜索在本实验室中创建的资源。

1. 在资源页面上，选择**删除**，然后按照说明删除资源。 或者，也可以删除整个资源组，同时清理所有资源。

## 详细信息

**人脸**服务中提供了几个附加功能，但根据[负责任 AI 标准](https://aka.ms/aah91ff)，这些功能受限于有限访问策略。 这些功能包括识别、验证和创建面部识别模型。 若要了解详细信息并申请访问权限，请参阅[Azure AI 服务的有限访问](https://docs.microsoft.com/en-us/azure/cognitive-services/cognitive-services-limited-access)。

有关使用 **Azure AI 视觉**服务进行面部检测的详细信息，请参阅[Azure AI 视觉文档](https://docs.microsoft.com/azure/cognitive-services/computer-vision/concept-detecting-faces)。

要详细了解**人脸**服务，请参阅[人脸文档](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-identity)。
