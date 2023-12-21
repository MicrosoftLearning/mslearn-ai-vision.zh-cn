---
lab:
  title: 使用 Azure AI 视觉自定义模型对图像进行分类
  module: Module 2 - Develop computer vision solutions with Azure AI Vision
---

# 使用 Azure AI 视觉自定义模型对图像进行分类

使用 Azure AI 视觉可以训练自定义模型，以使用指定的标签对对象进行分类和检测。 在此实验室中，我们将构建一个自定义图像分类模型来对水果图像进行分类。

## 克隆本课程的存储库

如果尚未将 Azure AI 视觉**** 代码存储库克隆到此实验室的工作环境，请按照以下步骤执行此操作。 否则，请在 Visual Studio Code 中打开克隆的文件夹。

1. 启动 Visual Studio Code。
2. 打开面板 (SHIFT+CTRL+P) 并运行“Git: 克隆”命令，以将 `https://github.com/MicrosoftLearning/mslearn-ai-vision` 存储库克隆到本地文件夹（任意文件夹均可）。
3. 克隆存储库后，在 Visual Studio Code 中打开文件夹。
4. 等待其他文件安装完毕，以支持存储库中的 C# 代码项目。

    > **注意**：如果系统提示你添加生成和调试所需的资产，请选择“以后再说”。 如果系统提示此消息：*“在文件夹中检测到 Azure 函数项目”*，则可以安全地关闭该消息。

## 预配 Azure 资源

如果订阅中尚无资源，则需要预配 Azure AI 服务**** 资源。

1. 打开 Azure 门户 (`https://portal.azure.com`)，然后使用与你的 Azure 订阅关联的 Microsoft 帐户登录。
2. 在顶部搜索栏中，搜索 Azure AI 服务**，选择 Azure AI 服务****，并使用以下设置创建 Azure AI 服务多服务帐户资源：
    - **订阅**：Azure 订阅
    - 资源组：选择或创建一个资源组（如果使用受限制的订阅，你可能无权创建新的资源组 - 请使用提供的资源组）
    - **区域**：从美国东部、西欧、美国西部、美国西部 2 \*中进行选择**
    - **名称**：输入唯一名称
    - **定价层**：标准 S0

    \*Azure AI Vision 4.0 自定义模型标记目前仅在这些区域可用。

3. 选中所需的复选框并创建资源。
<!--4. When the resource has been deployed, go to it and view its **Keys and Endpoint** page. You will need the endpoint and one of the keys from this page in a future step. Save them off or leave this browser tab open.-->

我们还需要一个存储帐户来存储训练图像。

1. 返回 Azure 门户，搜索并选择“存储帐户”****，并使用以下设置创建新的存储帐户：
    - **订阅**：Azure 订阅
    - **资源组**：选择在其中创建了 Azure AI 服务的同一资源组**
    - **存储帐户名称**：customclassifySUFFIX 
        - *注意：将 `SUFFIX` 令牌替换为你的姓名首字母缩写或其他值，以确保资源名称是全局唯一的。*
    - **区域**：选择用于 Azure AI 服务资源的同一区域**
    - **性能**：标准
    - 冗余：本地冗余存储 (LRS)
1. 创建存储帐户时，转到 Visual Studio Code，然后展开 02-image-classification**** 文件夹。
1. 在该文件夹中，选择“replace.ps1”**** 并查看代码。 你将看到，它替换了我们将在后续步骤中使用的 JSON 文件（COCO 文件）中占位符的存储帐户名称。 将文件第一行中的占位符替换为存储帐户的名称。 保存文件。
1. 右键单击并打开 02-image-classification**** 上的集成终端，然后运行以下命令。

    ```powershell
    ./replace.ps1
    ```

1. 可以查看 COCO 文件，以确保存储帐户名称存在。 选择“training-images/training_labels.json”**** 并查看前几个条目。 在“absolute_url”** 字段中，应该会显示类似 `"https://myStorage.blob.core.windows.net/fruit/...` 的内容。
1. 关闭 JSON 和 PowerShell 文件，然后返回到浏览器窗口。
1. 你的存储帐户已完成。 转到存储帐户。
1. 在存储帐户上启用公共访问。 在左侧窗格中，导航到”设置“**** 组中的”配置“****，并启用“允许 Blob 匿名访问”**。 选择 **“保存”**
1. 在左侧窗格中，选择”容器“**** 并新建名为 `fruit` 的容器，并将“匿名访问级别”**** 设置为“容器（容器和 Blob 的匿名读取访问权限）”**。

    > **备注**：如果禁用”匿名访问级别“****，请刷新浏览器页面。

1. 导航到 `fruit`，然后将 02-image-classificationtraining-images**** 中的图像（和一个 JSON 文件）上传到该容器。

## 创建自定义模型训练项目

接下来，你将在 Vision Studio 中为自定义图像分类新建训练项目。

1. 在 Web 浏览器中，导航到 `https://portal.vision.cognitive.azure.com/`，然后使用在其中创建 Azure AI 资源的 Microsoft 帐户登录。
1. 选择“使用图像自定义模型”**** 磁贴（如果默认视图中未显示，则可在“图像分析”**** 标签页中找到），如果出现提示，请选择创建的 Azure AI 资源。
1. 在项目中，选择顶部的“添加新数据集”****。 使用以下设置配置 ：
    - **数据集名称**：training_images
    - **模型类型**：图像分类
    - **选择“Azure Blob 存储容器”**：选择“选择容器”****
        - **订阅**：Azure 订阅
        - **存储帐户**：创建的存储帐户**
        - **Blob 容器**：水果
    - 选中“允许 Vision Studio 读取和写入 Blob 存储”框
1. 选择 training_images**** 数据集。

在项目创建的当前阶段，通常会选择“创建 Azure ML 数据标签项目”**** 并标记图像，这将生成 COCO 文件。 如果你有时间，建议你尝试这样做，但出于本实验室的目的，我们已经为你标记了图像，并提供了生成的 COCO 文件。

1. 选择“添加 COCO 文件”****。
1. 在下拉列表中，选择“从 Blob 容器导入 COCO 文件”****
1. 由于已经连接名为 `fruit` 的容器，因此 Vision Studio 会在其中搜索 COCO 文件。 从下拉列表中选择 training_labels.json****，然后添加 COCO 文件。
1. 导航到左侧的“自定义模型”****，然后选择“训练新模型”****。 使用以下设置：
    - **模型名称**：classifyfruit
    - **模型类型**：图像分类
    - **选择训练数据集**：training_images
    - 将其余设置保留默认值，然后选择“训练模型”****

训练可能需要一些时间 - 默认预算最多为一个小时，但对于这个小型数据集，它通常比这快得多。 每隔几分钟选择“刷新”**** 按钮，直到作业状态为“已成功”**。 选择模型。

可在此处查看训练作业的性能。 查看已训练模型的精度和准确性。

## 测试自定义模型

模型已经过训练，并已准备好进行测试。

1. 在自定义模型的页面顶部，选择“试用”****。
1. 从指定要使用的模型的下拉列表中选择 classifyfruit**** 模型，然后浏览到 02-image-classificationtest-images**** 文件夹。
1. 选择每个图像并查看结果。 在结果框中选择“JSON”**** 标签页以检查完整的 JSON 响应。

<!-- Option coding example to run-->
## 清理资源

如果未将本实验室中创建的 Azure 资源用于其他训练模块，则可以将其删除，以避免产生进一步的费用。

1. 在 `https://portal.azure.com` 处打开 Azure 门户，然后在顶部搜索栏中搜索在此实验室中创建的资源。

2. 在“资源”页上，选择“删除”**** 并按照说明删除资源。 或者，可以删除整个资源组以同时清理所有资源。