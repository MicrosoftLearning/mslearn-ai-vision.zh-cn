---
lab:
  title: 读取图像中的文本
  module: Module 11 - Reading Text in Images and Documents
---

# 读取图像中的文本

光学字符识别 (OCR) 是计算机视觉服务的一项功能，用于读取图像和文档中的文本。 **Azure AI 视觉**服务提供两个用于读取文本的 API，你将在此练习中对其进行探索。

## 克隆本课程的存储库

如果尚未克隆用于本课程的存储库，请克隆它：

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行**Git：克隆**命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-vision` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。 如果系统提示*检测到文件夹中有 Azure 函数项目*，则可以放心地关闭该消息。

## 预配 Azure AI 服务资源

如果订阅中还没有 **Azure AI 服务**资源，则需要预配认知服务资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 在顶部搜索栏中搜索 *Azure AI 服务*，选择 **Azure AI 服务**，然后使用以下设置创建 Azure AI 服务多服务帐户资源：
    - **订阅**：*Azure 订阅*
    - **资源组**：*选择或创建一个资源组（如果使用受限制的订阅，你可能无权创建新的资源组 - 请使用提供的资源组）*
    - **区域：***从美国东部、法国中部、韩国中部、北欧、东南亚、西欧、美国西部或东亚中选择\**
    - **名称**：输*入唯一名称*
    - **定价层**：标准 S0

    \*Azure AI Vision 4.0 功能目前仅在这些地区提供。

3. 选中所需的复选框并创建资源。
4. 等待部署完成，然后查看部署详细信息。
5. 部署资源后，转到该资源并查看其**密钥和终结点**页面。 你将在下一个过程中用到此页面中的终结点和其中一个密钥。

## 准备使用 Azure AI 视觉 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用 Azure AI 视觉 SDK 来读取文本。

> **注意**：可选择将该 SDK 用于 **C#** 或 **Python**。 在以后的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的**资源管理器**窗格中，浏览到 **Labfiles\05-ocr** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **read-text** 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装 Azure AI 视觉 SDK 包：

    **C#**
    
    ```csharp
    dotnet add package Azure.AI.Vision.ImageAnalysis --prerelease
    ```

    > **注意**：如果提示您安装开发工具包扩展，您可以安全关闭该消息。

    **Python**
    
    ```python
    pip install azure-ai-vision
    ```

3. 查看 **read-text** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#** ：appsettings.json
    - **Python**：.env

    打开配置文件，然后更新其中包含的配置值，以反映认知服务资源的**终结点**和身份验证**密钥**。 保存所做更改。


## 使用 Azure AI Vision SDK 从图像中读取文本

**Azure AI Vision SDK** 的功能之一是从图像中读取文本。 在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用 Azure AI 视觉 SDK 从图像中读取文本。

1. **读取文本**文件夹包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：read-text.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释**导入命名空间**。 然后在此注释下添加以下特定于语言的代码，以导入使用 Azure AI 视觉 SDK 所需的命名空间：

**C#**

```C#
// Import namespaces
using Azure.AI.Vision.Common;
using Azure.AI.Vision.ImageAnalysis;
```

**Python**

```Python
# Import namespaces
import azure.ai.vision as sdk
```

2. 在客户端应用程序的代码文件中，可在 **Main** 函数中看到已提供用于加载配置设置的代码。 然后查找注释**Azure AI Vision 客户端的注释** 然后在此注释下添加以下特定于语言的代码，以创建 Azure AI 视觉对象客户端对象并对其进行身份验证：

**C#**

```C#
// Authenticate Azure AI Vision client
var cvClient = new VisionServiceOptions(
    aiSvcEndpoint,
    new AzureKeyCredential(aiSvcKey));
```

**Python**

```Python
# Authenticate Azure AI Vision client
cv_client = sdk.VisionServiceOptions(ai_endpoint, ai_key)
```

3. 在 **Main** 函数中，可在你刚刚添加的代码下看到，代码指定了图像文件的路径，然后将图像路径传递给 **GetTextRead** 函数。 此函数尚未完全实现。

4. 让我们在 **GetTextRead** 函数的正文中添加一些代码。 找到注释**使用分析图像函数读取图像中的文本**。 然后，在该注释下添加以下特定语言代码：
 
**C#**

```C#
// Use Analyze image function to read text in image
Console.WriteLine($"\nReading text in {imageFile}\n");

using (var imageData = File.OpenRead(imageFile))
{    
    var analysisOptions = new ImageAnalysisOptions()
    {
        // Specify features to be retrieved


    };

    using var imageSource = VisionSource.FromFile(imageFile);

    using var analyzer = new ImageAnalyzer(serviceOptions, imageSource, analysisOptions);

    var result = analyzer.Analyze();

    if (result.Reason == ImageAnalysisResultReason.Analyzed)
    {
        // get image captions
        if (result.Text != null)
        {
            Console.WriteLine($"Text:");

            // Prepare image for drawing
            System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
            Graphics graphics = Graphics.FromImage(image);
            Pen pen = new Pen(Color.Cyan, 3);

            foreach (var line in result.Text.Lines)
            {
                // Return the text detected in the image



            }

            // Save image
            String output_file = "text.jpg";
            image.Save(output_file);
            Console.WriteLine("\nResults saved in " + output_file + "\n");   
        }
    }

}  
```

**Python**

```Python
# Use Analyze image function to read text in image
print('Reading text in {}\n'.format(image_file))

analysis_options = sdk.ImageAnalysisOptions()

features = analysis_options.features = (
    # Specify the features to be retrieved


)

# Get image analysis
image = sdk.VisionSource(image_file)

image_analyzer = sdk.ImageAnalyzer(cv_client, image, analysis_options)

result = image_analyzer.analyze()

if result.reason == sdk.ImageAnalysisResultReason.ANALYZED:

    # Get image captions
    if result.text is not None:
        print("\nText:")

        # Prepare image for drawing
        image = Image.open(image_file)
        fig = plt.figure(figsize=(image.width/100, image.height/100))
        plt.axis('off')
        draw = ImageDraw.Draw(image)
        color = 'cyan'

        for line in result.text.lines:
            # Return the text detected in the image



        # Save image
        plt.imshow(image)
        plt.tight_layout(pad=0)
        outputfile = 'text.jpg'
        fig.savefig(outputfile)
        print('\n  Results saved in', outputfile)
```

5. 现在已经添加了 **GetTextRead** 函数的主体，在注释**指定要检索的特征**下，添加以下代码以指定要检索文本：

**C#**

```C#
// Specify features to be retrieved
Features =
    ImageAnalysisFeature.Text
```

**Python**

```Python
# Specify features to be retrieved
sdk.ImageAnalysisFeature.TEXT
```

7. 在 Visual Studio Code 的代码文件中，找到 **GetTextRead** 函数，在**返回图像中检测到的文本**注释下，添加以下代码（此代码会将图像文本打印到控制台，并生成突出显示图像文本的图像 **text.jpg**）：

**C#**

```C#
// Return the text detected in the image
Console.WriteLine(line.Content);

var drawLinePolygon = true;

// Return each line detected in the image and the position bounding box around each line



// Return each word detected in the image and the position bounding box around each word with the confidence level of each word



// Draw line bounding polygon
if (drawLinePolygon)
{
    var r = line.BoundingPolygon;

    Point[] polygonPoints = {
        new Point(r[0].X, r[0].Y),
        new Point(r[1].X, r[1].Y),
        new Point(r[2].X, r[2].Y),
        new Point(r[3].X, r[3].Y)
    };

    graphics.DrawPolygon(pen, polygonPoints);
}
```

**Python**

```Python
# Return the text detected in the image
print(line.content)    

drawLinePolygon = True

r = line.bounding_polygon
bounding_polygon = ((r[0], r[1]),(r[2], r[3]),(r[4], r[5]),(r[6], r[7]))

# Return each line detected in the image and the position bounding box around each line



# Return each word detected in the image and the position bounding box around each word with the confidence level of each word



# Draw line bounding polygon
if drawLinePolygon:
    draw.polygon(bounding_polygon, outline=color, width=3)
```

8. 在 **read-text/images** 文件夹中，选择 **Lincoln.jpg** 以查看代码将处理的文件。

9. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **1** 时运行的代码。 此代码会调用 **GetTextRead** 函数并传递*Lincoln.jpg*图像文件的路径。

10. 保存你的更改并返回到 **read-text** 文件夹的集成终端，然后输入以下命令以运行程序：

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

11. 在出现提示时输入 **1** 并观察输出，该输出是从图像中提取的文本。

12. 在 **read-text** 文件夹中，选择 **text.jpg** 图像，注意每*行*文字周围都有一个多边形。

13. 返回 Visual Studio Code 中的代码文件，找到注释**返回图像中检测到的每一行以及每一行周围的位置边界框**。 然后，在该注释下添加以下代码：

**C#**

```C#
// Return each line detected in the image and the position bounding box around each line
string pointsToString = "{" + string.Join(',', line.BoundingPolygon.Select(pointsToString => pointsToString.ToString())) + "}";
Console.WriteLine($"   Line: '{line.Content}', Bounding Polygon {pointsToString}");
```

**Python**

```Python
# Return each line detected in the image and the position bounding box around each line
print(" Line: '{}', Bounding Polygon: {}".format(line.content, bounding_polygon))
```

14. 保存你的更改并返回到 **read-text** 文件夹的集成终端，然后输入以下命令以运行程序：

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

15. 出现提示时，输入 **1** 并观察输出，输出应该是图像中的每一行文本及其在图像中的相应位置。


16. 返回 Visual Studio Code 中的代码文件，找到注释**返回图像中检测到的每个单词，以及每个单词周围的位置边框和每个单词的置信度**。 然后，在该注释下添加以下代码：

**C#**

```C#
// Return each word detected in the image and the position bounding box around each word with the confidence level of each word
foreach (var word in line.Words)
{
    pointsToString = "{" + string.Join(',', word.BoundingPolygon.Select(pointsToString => pointsToString.ToString())) + "}";
    Console.WriteLine($"     Word: '{word.Content}', Bounding polygon {pointsToString}, Confidence {word.Confidence:0.0000}");

    // Draw word bounding polygon
    drawLinePolygon = false;
    var r = word.BoundingPolygon;

    Point[] polygonPoints = {
        new Point(r[0].X, r[0].Y),
        new Point(r[1].X, r[1].Y),
        new Point(r[2].X, r[2].Y),
        new Point(r[3].X, r[3].Y)
    };

    graphics.DrawPolygon(pen, polygonPoints);
}
```

**Python**

```Python
# Return each word detected in the image and the position bounding box around each word with the confidence level of each word
for word in line.words:
    r = word.bounding_polygon
    bounding_polygon = ((r[0], r[1]),(r[2], r[3]),(r[4], r[5]),(r[6], r[7]))
    print("  Word: '{}', Bounding Polygon: {}, Confidence: {}".format(word.content, bounding_polygon,word.confidence))

    # Draw word bounding polygon
    drawLinePolygon = False
    draw.polygon(bounding_polygon, outline=color, width=3)
```

17. 保存你的更改并返回到 **read-text** 文件夹的集成终端，然后输入以下命令以运行程序：

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

18. 出现提示时，输入 **1** 并观察输出结果，即图像中的每个文本单词及其在图像中的相应位置。 注意每个单词的置信度也会返回。

19. 在 **read-text** 文件夹中，选择 **text.jpg** 图像，注意每个*单词*周围都有一个多边形。

## 使用 Azure AI Vision SDK 从图像中读取手写文本

在前面的练习中，您从图像中读取了定义明确的文本，但有时您可能还想从手写笔记或纸张中读取文本。 好消息是，**Azure AI Vision SDK** 也可以读取手写文本，其代码与读取定义明确的文本时所用的代码完全相同。 我们将使用之前练习中的相同代码，但这次我们将使用不同的图像。

1. 在 **read-text/images** 文件夹中，单击 **Note.jpg** 以查看代码将处理的文件。

2. 在应用程序的代码文件中，在 **Main** 函数中检查用户选择菜单选项 **2** 时运行的代码。 此代码会调用 **GetTextRead** 函数并传递 *Note.jpg* 图像文件的路径。

3. 在 **read-text **文件夹的集成终端中，输入以下命令以运行程序：

**C#**

```
dotnet run
```

**Python**

```
python read-text.py
```

4. 出现提示时，输入 **2** 并观察输出结果，即从注释图像中提取的文本。

5. 在 **read-text** 文件夹中，选择 **text.jpg** 图像，注意在纸条的每个*单词*周围都有一个多边形。

## 清理资源

如果不将本实验室创建的 Azure 资源用于其他培训模块，可以将其删除，以免产生更多费用。 操作步骤如下：

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。

2. 在顶部搜索栏中，搜索 *Azure AI 服务多服务帐户*，然后选择在本实验室中创建的 Azure AI 服务多服务帐户资源

3. 在资源页面上，选择**删除**，然后按照说明删除资源。

## 更多信息

有关使用 **Azure AI 视觉** 服务读取文本的详细信息，请参阅 [Azure AI Vision 文档](https://learn.microsoft.com/azure/ai-services/computer-vision/overview-ocr)。
