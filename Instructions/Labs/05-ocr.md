---
lab:
  title: 读取图像中的文本
  module: Module 11 - Reading Text in Images and Documents
---

# 读取图像中的文本

光学字符识别 (OCR) 是计算机视觉服务的一项功能，用于读取图像和文档中的文本。 **Azure AI 视觉**服务提供两个用于读取文本的 API，你将在此练习中对其进行探索。

## 预配计算机视觉资源

如果订阅中还没有计算机视觉资源，则需要进行预配****。

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
   cd mslearn-ai-vision/Labfiles/05-ocr
    ```

## 准备使用 Azure AI 视觉 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用 Azure AI 视觉 SDK 来读取文本。

> **注意**：可选择将该 SDK 用于 **C#** 或 **Python**。 在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 导航到包含以你的首选语言显示的应用程序代码文件的文件夹：  

    **C#**

    ```
   cd C-Sharp/read-text
    ```
    
    **Python**

    ```
   cd Python/read-text
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
1. 替换占位符后，在代码编辑器中使用 **CTRL+S** 命令或 ** 右键单击 > 保存** 保存更改，然后使用 **CTRL+Q** 命令或 ** 右键单击 > 退出** 关闭代码编辑器，同时保持 Cloud Shell 命令行打开。

## 使用 Azure AI Vision SDK 从图像中读取文本

**Azure AI 视觉 SDK** 的其中一项功能是从图像中读取文本。 在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用 Azure AI 视觉 SDK 从图像中读取文本。

1. **读取文本**文件夹包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：read-text.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释**导入命名空间**。 然后在此注释下添加以下特定于语言的代码，以导入使用 Azure AI 视觉 SDK 所需的命名空间：

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

1. 在客户端应用程序的代码文件中，可在 **Main** 函数中看到已提供用于加载配置设置的代码。 然后查找注释**Azure AI Vision 客户端的注释** 然后在此注释下添加以下特定于语言的代码，以创建 Azure AI 视觉对象客户端对象并对其进行身份验证：

    **C#**
    
    ```C#
    // Authenticate Azure AI Vision client
    ImageAnalysisClient client = new ImageAnalysisClient(
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

1. 在 **Main** 函数中，可在你刚刚添加的代码下看到，代码指定了图像文件的路径，然后将图像路径传递给 **GetTextRead** 函数。 此函数尚未完全实现。

1. 让我们在 **GetTextRead** 函数的正文中添加一些代码。 找到注释**使用分析图像函数读取图像中的文本**。 然后，在此注释下，添加以下特定于语言的代码，指出在调用 `Analyze` 函数时指定视觉特征：

    **C#**

    ```C#
    // Use Analyze image function to read text in image
    ImageAnalysisResult result = client.Analyze(
        BinaryData.FromStream(stream),
        // Specify the features to be retrieved
        VisualFeatures.Read);
    
    stream.Close();
    
    // Display analysis results
    if (result.Read != null)
    {
        Console.WriteLine($"Text:");
    
        // Load the image using SkiaSharp
        using SKBitmap bitmap = SKBitmap.Decode(imageFile);
        // Create canvas to draw on the bitmap
        using SKCanvas canvas = new SKCanvas(bitmap);

        // Create paint for drawing polygons (bounding boxes)
        SKPaint paint = new SKPaint
        {
            Color = SKColors.Cyan,
            StrokeWidth = 3,
            Style = SKPaintStyle.Stroke,
            IsAntialias = true
        };

        foreach (var line in result.Read.Blocks.SelectMany(block => block.Lines))
        {
            // Return the text detected in the image
    
    
        }
            
        // Save the annotated image using SkiaSharp
        using (SKFileWStream output = new SKFileWStream("text.jpg"))
        {
            // Encode the bitmap into JPEG format with full quality (100)
            bitmap.Encode(output, SKEncodedImageFormat.Jpeg, 100);
        }

        Console.WriteLine("\nResults saved in text.jpg\n");
    }
    ```
    
    **Python**
    
    ```Python
    # Use Analyze image function to read text in image
    result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ]
    )

    # Display the image and overlay it with the extracted text
    if result.read is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.read.blocks[0].lines:
            # Return the text detected in the image

            
        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
    ```

1. 在刚刚添加到 **GetTextRead** 函数的代码中，在**返回图像中检测到的文本**注释下，添加以下代码（此代码会将图像文本输出到控制台并生成 **text.jpg**，它会突出显示图像的文本）：

    **C#**
    
    ```C#
    // Return the text detected in the image
    Console.WriteLine($"   '{line.Text}'");
    
    // Draw bounding box around line
    bool drawLinePolygon = true;
    
    // Return the position bounding box around each line
    
    
    
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    
    // Draw line bounding polygon
    if (drawLinePolygon)
    {
        var r = line.BoundingPolygon;
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

    DrawPolygon(canvas, polygonPoints, paint);
    }
    ```
    
    **Python**
    
    ```Python
    # Return the text detected in the image
    print(f"  {line.text}")    
    
    drawLinePolygon = True
    
    r = line.bounding_polygon
    bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
    
    # Return the position bounding box around each line
    
    
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    
    
    # Draw line bounding polygon
    if drawLinePolygon:
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

1. 仅对于 C# 程序文件，仍需要帮助程序函数来绘制多边形。 在“帮助程序方法在给定 SKPoints 数组的情况下绘制多边形”注释下，添加以下代码：****

    **C#**
   
    ```C#
    // Helper method to draw a polygon given an array of SKPoints
    static void DrawPolygon(SKCanvas canvas, SKPoint[] points, SKPaint paint)
    {
        if (points == null || points.Length == 0)
            return;

        using (var path = new SKPath())
        {
            path.MoveTo(points[0]);
            for (int i = 1; i < points.Length; i++)
            {
                path.LineTo(points[i]);
            }
            path.Close();
            canvas.DrawPath(path, paint);
        }
    }
    ```

1. 在 Main 函数中检查用户选择菜单选项 1 时运行的代码********。 此代码会调用 **GetTextRead** 函数并传递*Lincoln.jpg*图像文件的路径。

1. 保存更改并关闭代码编辑器。

1. 在 Cloud Shell 工具栏中，依次选择“上传/下载文件”、“下载”。******** 在“新建”对话框中，输入下列文件路径，然后选择“下载”：****

    **C#**
   
    ```
    mslearn-ai-vision/Labfiles/05-ocr/C-Sharp/read-text/images/Lincoln.jpg
    ```

    **Python**

    ```
    mslearn-ai-vision/Labfiles/05-ocr/Python/read-text/images/Lincoln.jpg
    ```
       
1. 打开并查看 Lincoln.jpg**** 图像。

1. 查看代码处理过的图像后，输入以下命令来运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 在出现提示时输入 **1** 并观察输出，该输出是从图像中提取的文本。

1. 在 read-text 文件夹中****，已创建 text.jpg 图像。**** 可以使用文件路径 `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/text.jpg` 下载它，请注意每行文本周围有一个多边形。**

1. 重新打开代码文件并找到注释“返回每行周围的位置边框”****。 然后，在该注释下添加以下代码：

    **C#**
    
    ```C#
    // Return the position bounding box around each line
    Console.WriteLine($"   Bounding Polygon: [{string.Join(" ", line.BoundingPolygon)}]");
    ```
    
    **Python**
    
    ```Python
    # Return the position bounding box around each line
    print("   Bounding Polygon: {}".format(bounding_polygon))
    ```

1. 保存更改并关闭代码编辑器，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 出现提示时，输入 **1** 并观察输出，输出应该是图像中的每一行文本及其在图像中的相应位置。

1. 再次打开代码文件，并找到注释“返回图像中检测到的每个字，以及每个字周围的位置边框和每个字的可信度”。**** 然后，在该注释下添加以下代码：

    **C#**
    
    ```C#
    // Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    foreach (DetectedTextWord word in line.Words)
    {
        Console.WriteLine($"     Word: '{word.Text}', Confidence {word.Confidence:F4}, Bounding Polygon: [{string.Join(" ", word.BoundingPolygon)}]");
        
        // Draw word bounding polygon
        drawLinePolygon = false;
        var r = word.BoundingPolygon;
    
        // Convert the bounding polygon into an array of SKPoints    
        SKPoint[] polygonPoints = new SKPoint[]
        {
            new SKPoint(r[0].X, r[0].Y),
            new SKPoint(r[1].X, r[1].Y),
            new SKPoint(r[2].X, r[2].Y),
            new SKPoint(r[3].X, r[3].Y)
        };

        // Draw the word polygon on the canvas
        DrawPolygon(canvas, polygonPoints, paint);
    }
    ```
    
    **Python**
    
    ```Python
    # Return each word detected in the image and the position bounding box around each word with the confidence level of each word
    for word in line.words:
        r = word.bounding_polygon
        bounding_polygon = ((r[0].x, r[0].y),(r[1].x, r[1].y),(r[2].x, r[2].y),(r[3].x, r[3].y))
        print(f"    Word: '{word.text}', Bounding Polygon: {bounding_polygon}, Confidence: {word.confidence:.4f}")
    
        # Draw word bounding polygon
        drawLinePolygon = False
        draw.polygon(bounding_polygon, outline=color, width=3)
    ```

1. 保存更改并关闭代码编辑器，然后输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 出现提示时，输入 **1** 并观察输出结果，即图像中的每个文本单词及其在图像中的相应位置。 注意每个单词的置信度也会返回。

1. 再次下载 text.jpg 图像，请注意每个字周围有一个多边形。******

## 使用 Azure AI Vision SDK 从图像中读取手写文本

在前面的练习中，您从图像中读取了定义明确的文本，但有时您可能还想从手写笔记或纸张中读取文本。 好消息是，**Azure AI Vision SDK** 也可以读取手写文本，其代码与读取定义明确的文本时所用的代码完全相同。 我们将使用之前练习中的相同代码，但这次我们将使用不同的图像。

1. 使用文件路径 `mslearn-ai-vision/Labfiles/05-ocr/***C-Sharp or Python***/read-text/images/Note.jpg` 下载 Note.jpg 以查看代码处理的下一个图像。****

1. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **2** 时运行的代码。 此代码会调用 **GetTextRead** 函数并传递 *Note.jpg* 图像文件的路径。

1. 在终端中，输入以下命令以运行程序：

    **C#**
    
    ```
    dotnet run
    ```
    
    **Python**
    
    ```
    python read-text.py
    ```

1. 出现提示时，输入 **2** 并观察输出结果，即从注释图像中提取的文本。

1. 再次下载 text.jpg 图像，请注意备注的每个字周围有一个多边形。******

## 清理资源

如果不将本实验室创建的 Azure 资源用于其他培训模块，可以将其删除，以免产生更多费用。 操作步骤如下：

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。

1. 在顶部搜索栏中，搜索“计算机视觉”**，然后选择在本实验室中创建的计算机视觉资源。

1. 在资源页面上，选择**删除**，然后按照说明删除资源。

## 详细信息

有关使用 **Azure AI 视觉** 服务读取文本的详细信息，请参阅 [Azure AI Vision 文档](https://learn.microsoft.com/azure/ai-services/computer-vision/concept-ocr)。
