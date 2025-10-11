---
lab:
  title: 读取图像中的文本
  description: 使用 Azure AI 视觉图像分析服务中的光学字符识别 (OCR) 查找和提取图像中的文本。
---

# 读取图像中的文本

光学字符识别 (OCR) 是计算机视觉服务的一项功能，用于读取图像和文档中的文本。 Azure AI 视觉**** 图像分析服务提供了用于读取文本的 API，你将在本练习中探索该 API。

> **注意**：本练习基于预发布版 SDK 软件，未来可能会有所变动。 必要时，我们使用了特定版本的包；这可能没有反映最新的可用版本。 可能会遇到一些意想不到的行为、警告或错误。

尽管本练习基于 Azure 视觉分析 Python SDK，但你也可以使用多种语言特定的 SDK 开发视觉应用程序，包括：

* [适用于 JavaScript 的 Azure AI 视觉分析](https://www.npmjs.com/package/@azure-rest/ai-vision-image-analysis)
* [适用于 Microsoft .NET 的 Azure AI 视觉分析](https://www.nuget.org/packages/Azure.AI.Vision.ImageAnalysis)
* [适用于 Java 的 Azure AI 视觉分析](https://mvnrepository.com/artifact/com.azure/azure-ai-vision-imageanalysis)

此练习大约需要 **30** 分钟。

## 预配 Azure AI 视觉资源

如果订阅中还没有 Azure AI 视觉资源，则需要进行预配。

> **注意**：在本练习中，你将使用独立的“计算机视觉”**** 资源。 还可以直接使用 Azure AI 服务多服务资源中的 Azure AI 视觉服务，或在 Azure AI Foundry 项目中使用。****

1. 打开 [Azure 门户](https://portal.azure.com) (网址为 `https://portal.azure.com`)，然后使用你的 Azure 凭据登录。 关闭显示的任何欢迎消息或提示。
1. 选择“创建资源”。****
1. 在搜索栏中，搜索 `Computer Vision`，选择“计算机视觉”，然后使用以下设置创建资源：****
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - 区域****：从以下区域进行选择：美国东部、美国西部、法国中部、韩国中部、北欧、东南亚、西欧或东亚*********************************\**
    - **名称**：计算机视觉资源的有效名称**
    - **定价层**：免费 F0

    \*Azure AI 视觉 4.0 完整功能集目前仅在这些区域提供。

1. 选中所需的复选框并创建资源。
1. 等待部署完成，然后查看部署详细信息。
1. 部署资源后，转到该资源，然后在导航窗格中的“资源管理”节点下查看其“密钥和终结点”页面。******** 你将在下一个过程中用到此页面中的终结点和其中一个密钥。

## 使用 Azure AI 视觉 SDK 开发文本提取应用

在此练习中，你将完成一个已部分实现的客户端应用程序，该应用程序使用 Azure AI 视觉 SDK 从图像中提取文本。

### 准备应用程序配置

1. 在 Azure 门户中，使用页面顶部搜索栏右侧的“[\>_]”按钮在 Azure 门户中创建新的 Cloud Shell，选择订阅中不含存储的“PowerShell”环境。**********

    在 Azure 门户底部的窗格中，Cloud Shell 提供命令行接口。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

    > **注意**：如果门户要求你选择存储来保存文件，请选择“不需要存储帐户”，选择正在使用的订阅，然后按“应用”。********

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 重设 Cloud Shell 窗格的大小，以便你仍然可以看到计算机视觉资源的“密钥和终结点”页面。****

    > 提示****：可以通过拖动上边框来调整窗格的大小。 还可以使用最小化和最大化按钮在 Cloud Shell 和主门户界面之间切换。

1. 在 Cloud Shell 窗格中，输入以下命令以克隆包含此练习代码文件的 GitHub 存储库（键入命令，或将其复制到剪贴板后，在命令行中右键单击并粘贴为纯文本）：

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **提示**：将命令粘贴到 Cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 克隆存储库后，使用以下命令导航到应用程序代码文件：

    ```
   cd mslearn-ai-vision/Labfiles/ocr/python/read-text
   ls -a -l
    ```

    该文件夹包含应用的应用程序配置和代码文件。 它还包含 /images**** 子文件夹，其中包含应用将分析的某些图像文件。

1. 运行以下命令安装 Azure AI 视觉 SDK 包和其他所需包：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-ai-vision-imageanalysis==1.0.0
    ```

1. 输入以下命令来编辑应用的配置文件：

    ```
   code .env
    ```

    该文件已在代码编辑器中打开。

1. 在代码文件中，更新它所包含的配置值以反映计算机视觉资源的终结点和身份验证密钥（从Azure 门户的“密钥和终结点”页面复制）。************
1. 替换占位符后，使用 **Ctrl+S** 命令保存更改，然后使用 **Ctrl+Q** 命令关闭代码编辑器，同时使 Cloud Shell 命令行保持打开状态。

### 添加代码以读取图像中的文本

1. 在 Cloud Shell 命令行中，输入以下命令以打开客户端应用程序的代码文件：

    ```
   code read-text.py
    ```

    > **提示**：你可能希望最大化 Cloud Shell 窗格并在命令行控制台和代码编辑器之间移动拆分栏，以便更轻松地查看代码。

1. 在代码文件中，找到注释“导入命名空间”****，并添加以下代码以导入使用 Azure AI 视觉 SDK 所需的命名空间：

    ```python
   # import namespaces
   from azure.ai.vision.imageanalysis import ImageAnalysisClient
   from azure.ai.vision.imageanalysis.models import VisualFeatures
   from azure.core.credentials import AzureKeyCredential
    ```

1. 在 Main**** 函数中，已提供用于加载配置设置和确定要分析的文件的代码。 然后找到注释“对 Azure AI 视觉客户端进行身份验证”，并添加以下特定于语言的代码以创建和验证 Azure AI 视觉图像分析客户端物体：****

    ```python
   # Authenticate Azure AI Vision client
   cv_client = ImageAnalysisClient(
        endpoint=ai_endpoint,
        credential=AzureKeyCredential(ai_key))
    ```

1. 在 Main**** 函数中刚刚添加的代码下，找到注释“读取图像中的文本”****，并添加以下代码以使用图像分析客户端读取图像中的文本：

    ```python
   # Read text in image
   with open(image_file, "rb") as f:
        image_data = f.read()
   print (f"\nReading text in {image_file}")

   result = cv_client.analyze(
        image_data=image_data,
        visual_features=[VisualFeatures.READ])
    ```

1. 找到注释“输出文本”****，并添加以下代码（包括最终注释）以输出找到的文本行，并调用函数以在图像中为其添加批注（使用为每行文本返回的 bounding_polygon****）：

    ```python
   # Print the text
   if result.read is not None:
        print("\nText:")
    
        for line in result.read.blocks[0].lines:
            print(f" {line.text}")        
        # Annotate the text in the image
        annotate_lines(image_file, result.read)

        # Find individual words in each line
        
    ```

1. 保存更改 (Ctrl+S**)，但如果需要修复任何拼写错误，请确保代码编辑器处于打开状态。

1. 调整窗格的大小，以便可以看到控制台的更多内容，然后输入以下命令来运行程序：

    ```
   python read-text.py images/Lincoln.jpg
    ```

1. 程序将读取指定图像文件中的文本 (images/Lincoln.jpg**)，如下所示：

    ![亚伯拉罕·林肯雕像的照片。](../media/Lincoln.jpg)

1. 在 read-text 文件夹中****，已创建 lines.jpg 图像。**** 使用（特定于 Azure Cloud Shell）“下载”**** 命令进行下载：

    ```
   download lines.jpg
    ```

    下载命令会在浏览器右下角创建弹出链接，可以选择此链接下载并打开文件。 图像应如下所示：

    ![突出显示文本的图像。](../media/text.jpg)

1. 再次运行程序，这次指定参数 images/Business-card.jpg**，以从下图中提取文本：

    ![扫描的名片的图像。](../media/Business-card.jpg)

    ```
   python read-text.py images/Business-card.jpg
    ```

1. 下载并查看生成的 lines.jpg**** 文件：

    ```
   download lines.jpg
    ```

1. 再次运行程序，这次指定参数 images/Note.jpg**，以从该图像中提取文本：

    ![手写购物清单的照片。](../media/Note.jpg)

    ```
   python read-text.py images/Note.jpg
    ```

1. 下载并查看生成的 lines.jpg**** 文件：

    ```
   download lines.jpg
    ```

### 添加代码以返回单个单词的位置

1. 调整窗格的大小，以便看到代码文件的更多内容。 找到注释“在每行查找单个单词”并添加以下代码（注意保持正确的缩进级别）：****

    ```python
   # Find individual words in each line
   print ("\nIndividual words:")
   for line in result.read.blocks[0].lines:
        for word in line.words:
            print(f"  {word.text} (Confidence: {word.confidence:.2f}%)")
   # Annotate the words in the image
   annotate_words(image_file, result.read)
    ```

1. 保存所做的更改 (Ctrl+S)。** 然后，在命令行窗格中，再次运行程序以从 images/Lincoln.jpg** 中提取文本。
1. 观察输出，该输出应包括图像中的每个单词以及与其预测关联的置信度。
1. 在 read-text 文件夹中****，已创建 words.jpg 图像。**** 使用（特定于 Azure Cloud Shell）“下载”**** 命令下载并查看：

    ```
   download words.jpg
    ```

1. 针对 images/Business-card.jpg** 和 images/Note.jpg** 再次运行程序；查看为每个图像生成的 words.jpg**** 文件。

## 清理资源

如果已完成 Azure AI 视觉的探索，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本：

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。

1. 在顶部搜索栏中，搜索“计算机视觉”**，然后选择在本实验室中创建的计算机视觉资源。

1. 在资源页面上，选择**删除**，然后按照说明删除资源。
