---
lab:
  title: 使用 Azure AI 视觉自定义模型对图像进行分类
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# 使用 Azure AI 视觉自定义模型对图像进行分类

Azure AI Vision 可让您训练自定义模型，使用您指定的标签对物体进行分类和检测。 在本实验室中，我们将建立一个自定义图像分类模型，对水果图像进行分类。

## 预配计算机视觉资源

如果订阅中还没有计算机视觉资源，则需要预配计算机视觉资源****。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
1. 选择“创建资源”。****
1. 在搜索栏中，搜索“计算机视觉”，选择“计算机视觉”，然后使用以下设置创建资源：******
    - **订阅**：*Azure 订阅*
    - **资源组**：*选择或创建一个资源组（如果使用受限制的订阅，你可能无权创建新的资源组 - 请使用提供的资源组）*
    - **区域**：*从美国东部、美国西部、法国中部、韩国中部、北欧、东南亚、西欧或东亚中选择\**
    - **名称**：*输入唯一名称*
    - **定价层**：免费 F0

    \*Azure AI 视觉 4.0 完整功能集目前仅在这些区域提供。

1. 选中所需的复选框并创建资源。
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

我们还需要一个存储账户来存储训练图像。

1. 在 Azure 门户中，搜索并选择**存储账户**，然后使用以下设置创建一个新的存储账户：
    - **订阅**：*Azure 订阅*
    - **资源组**：选择在其中创建自定义视觉资源的同一资源组**
    - **存储账户名称**：customclassifySUFFIX 
        - 注意：用你的姓名缩写或其他值替换 `SUFFIX` 令牌，以确保资源名称的全局唯一性。**
    - **区域**：*选择与您的 Azure AI 服务资源相同的区域*
    - **主服务**：Azure Blob 存储或 Azure Data Lake Storage Gen 2
    - **主要工作负载**：其他
    - **性能**：标准
    - **冗余**：本地冗余存储 (LRS)

1. 部署资源后，选择“前往资源”****。
1. 启用存储账户的公共访问。 在左侧窗格中，导航至**设置**组中的**配置**，然后启用*允许 Blob 匿名访问*。 选择**保存**
1. 在左侧窗格的"数据存储"**** 中，选择"容器"**** 并新建名为 `fruit` 的容器，然后将"匿名访问级别"**** 设置为"容器（对容器和 Blob 的匿名读取访问）"**。

    > **注意**：如果**匿名访问级别**被禁用，请刷新浏览器页面。
   
## 克隆本课程的存储库

用于模型训练的映像文件已在 GitHub 存储库中提供。 你将从 Azure 门户使用 Cloud Shell 克隆存储库并将映像上传到存储帐户。 

> **提示**：如果最近克隆了 mslearn-ai-vision**** 存储库，则可以跳过该克隆任务。 否则，请按照以下步骤将存储库克隆到开发环境中。

1. 在 Azure 门户中，使用页面顶部搜索栏右侧的 **[\>_]** 按钮，在 Azure 门户中创建新的 Cloud Shell，选择 ***PowerShell*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    > **提示**：将命令粘贴到 cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 在 PowerShell 窗格中，输入以下命令以克隆包含此练习的 GitHub 存储库：

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/microsoftlearning/mslearn-ai-vision mslearn-ai-vision
    ```

1. 克隆存储库后，导航到包含练习文件的文件夹：  

    ```
   cd mslearn-ai-vision/Labfiles/02-image-classification
    ```

1. 运行命令 `code replace.ps1` 并查看代码。 你会看到它将存储账户的名称替换为我们在后面步骤中使用的 JSON 文件（COCO 文件）中的占位符。
1. 用存储账户名称替换*文件第一行*的占位符。
1. 替换占位符后，在代码编辑器中，使用 Ctrl+S 命令或通过右键单击 >“保存”来保存更改，然后使用 Ctrl+Q 命令或通过右键单击 >“退出”来关闭代码编辑器，同时将 Cloud Shell 命令行保持打开状态。****************
1. 使用以下命令运行该脚本：

    ```powershell
    ./replace.ps1
    ```

1. 您可以查看 COCO 文件，以确保您的存储账户名称在其中。 运行 `code training-images/training_labels.json` 并查看前几个条目。 在 *absolute_url* 字段中，您应该看到类似于...*https://myStorage.blob.core.windows.net/fruit/..* 的内容。 如果没有看到预期的更改，请确保只更新了 PowerShell 脚本中的第一个占位符。
1. 关闭代码编辑器。
1. 运行以下命令（将 `<your-storage-account>` 替换为存储帐户的名称），将 training-images**** 文件夹的内容上传到之前创建的 `fruit` 容器。

    ```powershell
    az storage blob upload-batch --account-name <your-storage-account> -d fruit -s ./training-images/
    ```

1. 打开 `fruit` 容器并验证文件是否已正确上传。

## 创建自定义模型训练项目

接下来，您将在 Vision Studio 中为自定义图像分类创建一个新的训练项目。

1. 在 Web 浏览器中，导航到`https://portal.vision.cognitive.azure.com/`创建 Azure AI 资源的 Microsoft 帐户并登录。
1. 选择“使用图像自定义模型”**** 磁贴（如果默认视图中未显示，可在“图像分析”**** 选项卡中找到）。
1. 选择创建的 Azure AI 服务帐户。
1. 在项目中，选择顶部的**添加新数据集**。 使用以下设置配置 ：
    - **数据集名称**：training_images
    - **模型类型**：图像分类
    - **选择 Azure blob 存储容器**：选择**选择容器**
        - **订阅**：*Azure 订阅*
        - **存储账户**：*您创建的存储账户*
        - **Blob 容器**：水果
    - 选择允许 Vision Studio 读写您的 Blob 存储复选框
1. 选择 **training_images** 数据集。

在项目创建的这一阶段，您通常会选择**创建 Azure ML 数据标注项目**并对图像进行标注，这将生成一个 COCO 文件。 如果您有时间，我们鼓励您尝试这样做，但为了本实验室的目的，我们已经为您标注了图像，并提供了生成的 COCO 文件。

1. 选择**添加 COCO 文件**
1. 在下拉菜单中，选择**从 Blob 容器导入 COCO 文件**
1. 由于您已经连接了名为 `fruit`，的容器，Vision Studio 会在其中搜索 COCO 文件。 从下拉菜单中选择 **training_labels.json**，然后添加 COCO 文件。
1. 导航至左侧的**自定义模型**，然后选择**训练一个新模型**。 使用以下设置：
    - **模型名称**：classifyfruit
    - **模型类型**：图像分类
    - **选择训练数据集**：training_images
    - 其他设置保持默认，然后选择**训练模型**

训练可能需要一些时间--默认预算为一小时，但对于这个小数据集来说，通常比这要快得多。 每隔几分钟选择**刷新**按钮，直到任务状态为*成功*。 选择该模型。

在此，您可以查看训练作业的性能。 查看训练模型的精度和准确度。

## 测试自定义模型

您的模型已经训练完成，可以进行测试了。

1. 在自定义模型页面的顶部，选择**试用**。
1. 从指定要使用的模型的下拉菜单中选择 **classifyfruit** 模型，并浏览到 **02-image-classification\test-images** 文件夹。
1. 选择每张图片并查看结果。 在结果框中选择 **JSON** 选项卡，检查完整的 JSON 响应。

<!-- Option coding example to run-->
## 清理资源

如果不将本实验室创建的 Azure 资源用于其他培训模块，则可以删除这些资源以避免产生更多费用。

1. 打开 Azure 门户网站 `https://portal.azure.com`，在顶部搜索栏中搜索在本实验室中创建的资源。

2. 在资源页面上，选择**删除**，然后按照说明删除资源。 或者，也可以删除整个资源组，同时清理所有资源。
