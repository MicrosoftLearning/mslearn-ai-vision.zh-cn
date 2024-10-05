---
lab:
  title: 使用 Azure AI 视觉分析图像
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# 使用 Azure AI 视觉分析图像

Azure AI 视觉是一种人工智能功能，可支持软件系统通过分析图像来解释视觉输入。 在 Microsoft Azure 中，**视觉** Azure AI 服务为常见的计算机视觉任务提供预建模型，包括分析图像以建议标题和标签、检测常见物体和人物。 您还可以使用 Azure AI 视觉服务移除背景或创建图像的前景垫层。

## 克隆本课程的存储库

如果还没有将 **Azure AI Vision** 代码库克隆到正在进行本实验的环境中，请按照以下步骤进行克隆。 否则，请在 Visual Studio Code 中打开克隆的文件夹。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“**Git：克隆**”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-vision` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择**以后再说**。 如果系统提示*检测到文件夹中有 Azure 函数项目*，则可以放心地关闭该消息。

## 预配 Azure AI 服务资源

如果订阅中还没有，则需要预配 **Azure AI 服务**资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 选择“创建资源”。
3. 在搜索栏中，搜索“Azure AI 服务”**，选择“Azure AI 服务”****，并使用以下设置创建 Azure AI 服务多服务帐户资源：
    - **订阅**：*Azure 订阅*
    - **资源组**：*选择或创建一个资源组（如果使用受限制的订阅，你可能无权创建新的资源组 - 请使用提供的资源组）*
    - **区域**：*从美国东部、美国西部、法国中部、韩国中部、北欧、东南亚、西欧或东亚中选择\**
    - **名称**：输*入唯一名称*
    - **定价层**：标准版 S0

    \*Azure AI 视觉 4.0 完整功能集目前仅在这些区域提供。

4. 选中所需的复选框并创建资源。
5. 等待部署完成，然后查看部署详细信息。
6. 部署资源后，转到该资源并查看其**密钥和终结点**页面。 你将在下一个过程中用到此页面中的终结点和其中一个密钥。

## 准备使用 Azure AI 视觉 SDK

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用计算机视觉 SDK 来分析图像。

> **注意**：可选择将该 SDK 用于 **C#** 或 **Python**。 在下面的步骤中，请执行适用于你的语言首选项的操作。

1. 在 Visual Studio Code 的**资源管理器**窗格中，浏览到 **Labfiles/01-analyze-images** 文件夹，并根据你的语言首选项展开 **C-Sharp** 文件夹或 **Python** 文件夹。
2. 右键单击 **image-analysis** 文件夹，并打开集成终端。 然后通过运行适用于你的语言首选项的命令，安装 Azure AI 视觉 SDK 包：

    **C#**
    
    ```
    dotnet add package Azure.AI.Vision.ImageAnalysis -v 1.0.0-beta.3
    ```

    > **注意**：如果提示您安装开发工具包扩展，您可以安全关闭该消息。

    **Python**
    
    ```
    pip install azure-ai-vision-imageanalysis==1.0.0b3
    ```

    > **提示**：如果在自己的计算机上执行此操作，则还需要安装 `matplotlib` 和 `pillow`。
    
3. 查看 **text-analysis** 文件夹的内容，并注意其中包含一个配置设置文件：
    - **C#** ：appsettings.json
    - **Python**：.env

    打开配置文件，然后更新其中包含的配置值，以反映 Azure AI 服务资源的**终结点**和身份验证**密钥**。 保存所做更改。
4. 请注意，**image-analysis** 文件夹中包含客户端应用程序的代码文件：

    - **C#** ：Program.cs
    - **Python**：image-analysis.py

    打开代码文件，并在顶部的现有命名空间引用下找到注释**导入命名空间**。 然后在此注释下添加以下特定于语言的代码，以导入使用计算机视觉 SDK 所需的命名空间：

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

在此练习中，你将使用 Azure AI 视觉服务来分析多个图像。

1. 在 Visual Studio Code 中，展开 **image-vision** 文件夹以及其中包含的 **images** 文件夹。
2. 依次选择每个图像文件，以在 Visual Studio Code 中查看它们。

## 分析图像以建议描述文字

现在，你已准备好使用 SDK 来调用视觉服务并分析图像。

1. 在客户端应用程序的代码文件（**Program.cs** 或** image-analysis.py**）中，可在 **Main** 函数中看到已提供用于加载配置设置的代码。 然后查找注释**Azure AI Vision 客户端的注释** 然后在此注释下添加以下特定于语言的代码，以创建计算机视觉对象客户端对象并对其进行身份验证：

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

2. 在 **Main** 函数中，可在你刚刚添加的代码下看到，代码指定了图像文件的路径，然后将图像路径传递给另外两个函数（**AnalyzeImage** 和 **BackgroundForeground**）。 这两个函数尚未完全实现。

3. 在 **AnalyzeImage** 函数中的注释 **通过指定要检索的功能来获取结果** 下，添加以下代码：

**C#**

```C#
// Get result with specified features to be retrieved
ImageAnalysisResult result = client.Analyze(
    BinaryData.FromStream(stream),
    VisualFeatures.Caption | 
    VisualFeatures.DenseCaptions |
    VisualFeatures.Objects |
    VisualFeatures.Tags |
    VisualFeatures.People);
```

**Python**

```Python
# Get result with specified features to be retrieved
result = cv_client.analyze(
    image_data=image_data,
    visual_features=[
        VisualFeatures.CAPTION,
        VisualFeatures.DENSE_CAPTIONS,
        VisualFeatures.TAGS,
        VisualFeatures.OBJECTS,
        VisualFeatures.PEOPLE],
)
```
    
4. 在 **AnalyzeImage** 函数中的注释 **显示分析结果** 下，添加以下代码（包括指示稍后将添加更多代码的注释。）：

**C#**

```C#
// Display analysis results
// Get image captions
if (result.Caption.Text != null)
{
    Console.WriteLine(" Caption:");
    Console.WriteLine($"   \"{result.Caption.Text}\", Confidence {result.Caption.Confidence:0.00}\n");
}

// Get image dense captions
Console.WriteLine(" Dense Captions:");
foreach (DenseCaption denseCaption in result.DenseCaptions.Values)
{
    Console.WriteLine($"   Caption: '{denseCaption.Text}', Confidence: {denseCaption.Confidence:0.00}");
}

// Get image tags


// Get objects in the image


// Get people in the image
```

**Python**

```Python
# Display analysis results
# Get image captions
if result.caption is not None:
    print("\nCaption:")
    print(" Caption: '{}' (confidence: {:.2f}%)".format(result.caption.text, result.caption.confidence * 100))

# Get image dense captions
if result.dense_captions is not None:
    print("\nDense Captions:")
    for caption in result.dense_captions.list:
        print(" Caption: '{}' (confidence: {:.2f}%)".format(caption.text, caption.confidence * 100))

# Get image tags


# Get objects in the image


# Get people in the image

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

这有时可用于标识相关*标记*，这些标记提供了与图像内容有关的线索。

1. 在 **AnalyzeImage** 函数的注释**获取图像标记**下，添加以下代码：

**C#**

```C#
// Get image tags
if (result.Tags.Values.Count > 0)
{
    Console.WriteLine($"\n Tags:");
    foreach (DetectedTag tag in result.Tags.Values)
    {
        Console.WriteLine($"   '{tag.Name}', Confidence: {tag.Confidence:F2}");
    }
}
```

**Python**

```Python
# Get image tags
if result.tags is not None:
    print("\nTags:")
    for tag in result.tags.list:
        print(" Tag: '{}' (confidence: {:.2f}%)".format(tag.name, tag.confidence * 100))
```

2. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，注意到除了图像描述文字外，还会显示建议的标记列表。

## 检测图像中的物体并指示其位置

*物体检测*是计算机视觉服务的一种特定形式，可识别图像中的各个物体，并使用边界框指示其位置。

1. 在 **AnalyzeImage** 函数的注释**获取图像中的物体**下，添加以下代码：

**C#**

```C#
// Get objects in the image
if (result.Objects.Values.Count > 0)
{
    Console.WriteLine(" Objects:");

    // Prepare image for drawing
    stream.Close();
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedObject detectedObject in result.Objects.Values)
    {
        Console.WriteLine($"   \"{detectedObject.Tags[0].Name}\"");

        // Draw object bounding box
        var r = detectedObject.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        graphics.DrawString(detectedObject.Tags[0].Name,font,brush,(float)r.X, (float)r.Y);
    }

    // Save annotated image
    String output_file = "objects.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get objects in the image
if result.objects is not None:
    print("\nObjects in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_object in result.objects.list:
        # Print object name
        print(" {} (confidence: {:.2f}%)".format(detected_object.tags[0].name, detected_object.tags[0].confidence * 100))
        
        # Draw object bounding box
        r = detected_object.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height)) 
        draw.rectangle(bounding_box, outline=color, width=3)
        plt.annotate(detected_object.tags[0].name,(r.x, r.y), backgroundcolor=color)

    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'objects.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，注意检测到的任何物体。 在每次运行后，查看在代码文件所在的同一文件夹中生成的 **objects.jpg** 文件，以查看带有批注的物体。

## 检测图像中的人物并指示其位置

*人物检测*是计算机视觉服务的一种特定形式，可识别图像中的各个人物，并使用边界框指示其位置。

1. 在 **AnalyzeImage** 函数的注释**获取图像中的人物**下，添加以下代码：

**C#**

```C#
// Get people in the image
if (result.People.Values.Count > 0)
{
    Console.WriteLine($" People:");

    // Prepare image for drawing
    System.Drawing.Image image = System.Drawing.Image.FromFile(imageFile);
    Graphics graphics = Graphics.FromImage(image);
    Pen pen = new Pen(Color.Cyan, 3);
    Font font = new Font("Arial", 16);
    SolidBrush brush = new SolidBrush(Color.WhiteSmoke);

    foreach (DetectedPerson person in result.People.Values)
    {
        // Draw object bounding box
        var r = person.BoundingBox;
        Rectangle rect = new Rectangle(r.X, r.Y, r.Width, r.Height);
        graphics.DrawRectangle(pen, rect);
        
        // Return the confidence of the person detected
        //Console.WriteLine($"   Bounding box {person.BoundingBox.ToString()}, Confidence: {person.Confidence:F2}");
    }

    // Save annotated image
    String output_file = "persons.jpg";
    image.Save(output_file);
    Console.WriteLine("  Results saved in " + output_file + "\n");
}
```

**Python**

```Python
# Get people in the image
if result.people is not None:
    print("\nPeople in image:")

    # Prepare image for drawing
    image = Image.open(image_filename)
    fig = plt.figure(figsize=(image.width/100, image.height/100))
    plt.axis('off')
    draw = ImageDraw.Draw(image)
    color = 'cyan'

    for detected_people in result.people.list:
        # Draw object bounding box
        r = detected_people.bounding_box
        bounding_box = ((r.x, r.y), (r.x + r.width, r.y + r.height))
        draw.rectangle(bounding_box, outline=color, width=3)

        # Return the confidence of the person detected
        #print(" {} (confidence: {:.2f}%)".format(detected_people.bounding_box, detected_people.confidence * 100))
        
    # Save annotated image
    plt.imshow(image)
    plt.tight_layout(pad=0)
    outputfile = 'people.jpg'
    fig.savefig(outputfile)
    print('  Results saved in', outputfile)
```

2. (可选）取消返回检测到人员的置信度部分下的 **Console.Writeline** 命令，以**查看返回的置信度**，即在图像的特定位置检测到人员。

3. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，注意检测到的任何物体。 在每次运行后，查看在代码文件所在的同一文件夹中生成的 **objects.jpg** 文件，以查看带有批注的物体。

> **注意**：在前面的任务中，你使用了单种方法来分析图像，然后逐步添加代码来分析和显示结果。 SDK 还提供了用于建议描述文字、识别标记和检测物体等的单独方法，这意味着你可使用最适合的方法来仅返回所需信息，减少需要返回的数据有效负载的大小。 有关更多详细信息，请参阅 [.NET SDK 文档](https://learn.microsoft.com/dotnet/api/overview/azure/cognitiveservices/computervision?view=azure-dotnet) 或 [Python SDK 文档](https://learn.microsoft.com/python/api/azure-cognitiveservices-vision-computervision/azure.cognitiveservices.vision.computervision)。

## 去除背景或生成图像的前景哑光

在某些情况下，您可能需要移除图像的背景或创建图像的前景哑光。 让我们从移除背景开始。

1. 在代码文件中找到 **BackgroundForeground** 函数，在注释**移除图像背景或生成前景哑光**下添加以下代码：

**C#**

```C#
// Remove the background from the image or generate a foreground matte
Console.WriteLine($" Background removal:");
// Define the API version and mode
string apiVersion = "2023-02-01-preview";
string mode = "backgroundRemoval"; // Can be "foregroundMatting" or "backgroundRemoval"

string url = $"computervision/imageanalysis:segment?api-version={apiVersion}&mode={mode}";

// Make the REST call
using (var client = new HttpClient())
{
    var contentType = new MediaTypeWithQualityHeaderValue("application/json");
    client.BaseAddress = new Uri(endpoint);
    client.DefaultRequestHeaders.Accept.Add(contentType);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", key);

    var data = new
    {
        url = $"https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{imageFile}?raw=true"
    };

    var jsonData = JsonSerializer.Serialize(data);
    var contentData = new StringContent(jsonData, Encoding.UTF8, contentType);
    var response = await client.PostAsync(url, contentData);

    if (response.IsSuccessStatusCode) {
        File.WriteAllBytes("background.png", response.Content.ReadAsByteArrayAsync().Result);
        Console.WriteLine("  Results saved in background.png\n");
    }
    else
    {
        Console.WriteLine($"API error: {response.ReasonPhrase} - Check your body url, key, and endpoint.");
    }
}
```

**Python**

```Python
# Remove the background from the image or generate a foreground matte
print('\nRemoving background from image...')
    
url = "{}computervision/imageanalysis:segment?api-version={}&mode={}".format(endpoint, api_version, mode)

headers= {
    "Ocp-Apim-Subscription-Key": key, 
    "Content-Type": "application/json" 
}

image_url="https://github.com/MicrosoftLearning/mslearn-ai-vision/blob/main/Labfiles/01-analyze-images/Python/image-analysis/{}?raw=true".format(image_file)  

body = {
    "url": image_url,
}
    
response = requests.post(url, headers=headers, json=body)

image=response.content
with open("background.png", "wb") as file:
    file.write(image)
print('  Results saved in background.png \n')
```
    
2. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，以打开在每张图像的代码文件所在同一文件夹中生成的 **background.jpg** 文件。  请注意，每张图片的背景都已去除。

现在让我们为图像生成前景哑光。

3. 在代码文件中，找到 **BackgroundForeground** 函数；然后，在注释 **定义 API 版本和模式** 下，将模式变量更改为 `foregroundMatting`。

4. 保存更改，并针对 **images** 文件夹中的每个图像文件运行一次程序，以打开在每张图像的代码文件所在同一文件夹中生成的 **background.jpg** 文件。  请注意，您的图像已经生成了前景效果。

## 清理资源

如果不将本实验室创建的 Azure 资源用于其他培训模块，可以将其删除，以免产生更多费用。 操作步骤如下：

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。

2. 在顶部搜索栏中，搜索 *Azure AI 服务多服务帐户*，然后选择在本实验室中创建的 Azure AI 服务多服务帐户资源。

3. 在资源页面上，选择**删除**，然后按照说明删除资源。

## 详细信息

在此练习中，你探索了 Azure AI 视觉服务的一些图像分析和操作功能。 该服务还包括检测物体和人员以及其他计算机视觉任务的功能。

有关使用 **Azure AI 视觉** 服务的详细信息，请参阅 [Azure AI Vision 文档](https://learn.microsoft.com/azure/ai-services/computer-vision/)。
