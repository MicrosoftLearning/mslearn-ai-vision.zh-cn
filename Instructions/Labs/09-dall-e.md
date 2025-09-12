---
lab:
  title: 使用 AI 生成图像
  description: 使用 Azure AI Foundry 中的 OpenAI DALL-E 模型生成图像。
---

# 使用 AI 生成图像

在本练习中，你将使用 OpenAI DALL-E 生成式 AI 模型生成图像。 还可以使用 OpenAI Python SDK 创建一个简单的应用，来根据提示生成图像。

> **注意**：本练习基于预发布版 SDK 软件，未来可能会有所变动。 必要时，我们使用了特定版本的包；这可能没有反映最新的可用版本。 可能会遇到一些意想不到的行为、警告或错误。

尽管本练习基于 OpenAI Python SDK，但你也可以使用多种特定语言的 SDK 开发 AI 聊天应用程序，包括：

* [适用于 Microsoft .NET 的 OpenAI 项目](https://www.nuget.org/packages/OpenAI)
* [适用于 JavaScript 的 OpenAI 项目](https://www.npmjs.com/package/openai)

此练习大约需要 **30** 分钟。

## 打开 Azure AI Foundry 门户

首先登录到 Azure AI Foundry 门户。

1. 在 Web 浏览器中打开 [Azure AI Foundry 门户](https://ai.azure.com)，网址为：`https://ai.azure.com`，然后使用 Azure 凭据登录。 关闭首次登录时打开的任何使用技巧或快速入门窗格，如有必要，使用左上角的 **Azure AI Foundry** 徽标导航到主页，类似下图所示（若已打开**帮助**面板，请关闭）：

    ![Azure AI Foundry 门户的屏幕截图。](./media/ai-foundry-home.png)

1. 查看主页上的信息。

## 选择要启动项目的模型

Azure AI *项目*为 AI 开发提供协作工作区。 首先，选择要使用的模型并创建要在其中使用模型的项目。

> **备注**：AI Foundry 项目可以基于 *Azure AI Foundry* 资源，该资源提供对 AI 模型（包括 Azure OpenAI）、Azure AI 服务和其他资源的访问权限，用于开发 AI 代理和聊天解决方案。 或者，项目可以基于 *AI 中心*资源；其中包含与 Azure 资源的连接，用于安全存储、计算和专用工具。 基于 Azure AI Foundry 的项目非常适合想要管理 AI 代理或聊天应用开发的资源的开发人员。 基于 AI 中心的项目更适用于处理复杂 AI 解决方案的企业开发团队。

1. 在主页的“**浏览模型和功能**”部分中，搜索 `dall-e-3` 模型；我们将在项目中使用它。

1. 在搜索结果中，选择 dall-e-3 模型以查看其详细信息，然后在模型的页面顶部，选择“使用此模型”。********

1. 当提示创建项目时，输入项目的有效名称并展开“**高级选项**”。

1. 选择“**自定义**”并为中心指定以下设置：
    - **Azure AI Foundry 资源**：*Azure AI Foundry 资源的有效名称*
    - **订阅**：Azure 订阅
    - **资源组**：*创建或选择资源组*
    - **区域**：选择推荐的任何 AI Foundry******\*

    > \* 某些 Azure AI 资源受区域模型配额约束。 如果稍后在练习中达到配额限制，你可能需要在不同的区域中创建另一个资源。

1. 选择“创建”并等待项目（包括所选的 dall-e-3 模型部署）创建。****

    > 注意:根据选择的模型，可能会在项目创建过程中收到其他提示。 同意条款并完成部署。

1. 创建项目后，模型将显示在“模型 + 终结点”页面中****。

## 在操场中测试模型

在创建客户端应用程序之前，让我们在操场中测试 DALL-E 模型。

1. 选择“操场”，然后选择“图像操场”。********

1. 确保已选择 DALL-E 模型部署。 然后，在页面底部附近的框中，输入提示，例如 `Create an image of an robot eating spaghetti`，然后选择“生成”。****

1. 查看操场中生成的图像：

    ![图像操场的屏幕截图，其中包含生成的图像。](../media/images-playground.png)

1. 输入跟进提示，例如 `Show the robot in a restaurant` 并查看生成的图像。

1. 继续测试新的提示以优化图像，直到对图像感到满意。 

1. 选择“\</\> 查看代码”按钮，确保位于“Entra ID 身份验证”选项卡上。******** 然后记录以下信息以便稍后在练习中使用。 请注意，这些值为示例值，请务必记录部署中的信息。

    * OpenAI 终结点：https://dall-e-aus-resource.cognitiveservices.azure.com/**
    * OpenAI API 版本：2024-04-01-preview**
    * 部署名称（模型名称）：dall-e-3**

## 创建客户端应用程序

模型似乎在操场上工作。 现在，你可以使用 OpenAI SDK 以在客户端应用程序中使用它了。

### 准备应用程序配置

1. 打开新的浏览器选项卡（使 Azure AI Foundry 门户在现有选项卡中保持打开状态）。 然后在新选项卡中，浏览到 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`；如果出现提示，请使用 Azure 凭据登录。

1. 使用页面顶部搜索栏右侧的 **[\>_]** 按钮在 Azure 门户中创建新的 Cloud Shell，选择 ***PowerShell*** 环境。 Cloud Shell 在 Azure 门户底部的窗格中提供命令行接口。

    > **备注**：如果以前创建了使用 *Bash* 环境的 Cloud Shell，请将其切换到 ***PowerShell***。

    > **注意**：如果门户要求你选择存储来保存文件，请选择“不需要存储帐户”，选择正在使用的订阅，然后按“应用”。********

1. 在 Cloud Shell 工具栏的“**设置**”菜单中，选择“**转到经典版本**”（这是使用代码编辑器所必需的）。

    **<font color="red">在继续作之前，请确保已切换到 Cloud Shell 的经典版本。</font>**

1. 在 Cloud Shell 窗格中，输入以下命令以克隆包含此练习代码文件的 GitHub 存储库（键入命令，或将其复制到剪贴板后，在命令行中右键单击并粘贴为纯文本）：

    ```
    rm -r mslearn-ai-vision -f
    git clone https://github.com/MicrosoftLearning/mslearn-ai-vision
    ```

    > **提示**：将命令粘贴到 cloudshell 中时，输出可能会占用大量屏幕缓冲区。 可以通过输入 `cls` 命令来清除屏幕，以便更轻松地专注于每项任务。

1. 克隆存储库后，导航到包含应用程序代码文件的文件夹：  

    ```
   cd mslearn-ai-vision/Labfiles/dalle-client/python
    ```

1. 在 Cloud Shell 命令行窗格中，输入以下命令安装将使用的库：

    ```
   python -m venv labenv
   ./labenv/bin/Activate.ps1
   pip install -r requirements.txt azure-identity openai requests
    ```

1. 输入以下命令以编辑已提供的配置文件：

    ```
   code .env
    ```

    该文件已在代码编辑器中打开。

1. 将 your_endpoint、your_model_deployment 和 your_api_version 占位符替换为在“图像操场”中记录的值。****************

1. 替换占位符后，使用 **Ctrl+S** 命令保存更改，然后使用 **Ctrl+Q** 命令关闭代码编辑器，同时使 Cloud Shell 命令行保持打开状态。

### 写入代码以连接到项目并与模型聊天

> **提示**：添加代码时，请务必保持正确的缩进。

1. 输入以下命令以编辑已提供的代码文件：

    ```
   code dalle-client.py
    ```

1. 在代码文件中，请注意在文件顶部添加的现有语句，以导入必要的 SDK 命名空间。 然后，在注释“**添加引用**”下，添加以下代码以引用之前安装的库中的命名空间：

    ```python
   # Add references
   from dotenv import load_dotenv
   from azure.identity import DefaultAzureCredential, get_bearer_token_provider
   from openai import AzureOpenAI
   import requests
    ```

1. 在 main 函数的注释“获取配置设置”下，请注意，代码将加载配置文件中定义的终结点、API 版本和模型部署名称值。********

1. 在注释“初始化客户端”下，添加以下代码，以使用当前登录所使用的 Azure 凭据连接到模型：****

    ```python
   # Initialize the client
   token_provider = get_bearer_token_provider(
       DefaultAzureCredential(exclude_environment_credential=True,
           exclude_managed_identity_credential=True), 
       "https://cognitiveservices.azure.com/.default"
   )
    
   client = AzureOpenAI(
       api_version=api_version,
       azure_endpoint=endpoint,
       azure_ad_token_provider=token_provider
   )
    ```

1. 请注意，代码包含一个循环，允许用户输入提示，直到输入“退出”。 然后，在循环部分的注释“**获取图像**”下，添加以下代码，以提交提示，并从模型中获取生成图像的 URL：

    **Python**

    ```python
   # Generate an image
   result = client.images.generate(
        model=model_deployment,
        prompt=input_text,
        n=1
    )

   json_response = json.loads(result.model_dump_json())
   image_url = json_response["data"][0]["url"] 
    ```

1. 请注意，**main** 函数其余部分的代码将图像 URL 和文件名传递给提供的函数，该函数下载生成的图像并将其保存为.png文件。

1. 使用 **Ctrl+S** 命令保存对代码文件的更改，然后使用 **Ctrl+Q** 命令关闭代码编辑器，同时保持 Cloud Shell 命令行处于打开状态。

### 运行客户端应用程序

1. 在 Cloud Shell 命令行窗格中，输入以下命令以登录到 Azure。

    ```
   az login
    ```

    **<font color="red">必须登录到 Azure - 即使 Cloud Shell 会话已经过身份验证。</font>**

    > **备注**：在大多数情况下，仅使用 *az login* 就足够了。 但是，如果在多个租户中有订阅，则可能需要使用 *--tenant* 参数指定租户。 有关详细信息，请参阅[使用 Azure CLI 以交互方式登录到 Azure](https://learn.microsoft.com/cli/azure/authenticate-azure-cli-interactively)。

1. 出现提示时，请按照说明在新选项卡中打开登录页，并输入提供的验证码和 Azure 凭据。 然后在命令行中完成登录过程，并在出现提示时选择包含 Azure AI Foundry 中心的订阅。

1. 在 Cloud Shell 命令行窗格中，输入以下命令以运行应用：

    ```
   python dalle-client.py
    ```

1. 出现提示时，输入图像请求，例如 `Create an image of a robot eating pizza`。 一两分钟后，应用应确认图像已保存。

1. 再尝试一些提示。 完成后，输入`quit`退出程序。

    > **备注**：在此简单应用中，我们尚未实现用于保留对话历史记录的逻辑；因此模型会将每个提示视为一个新请求，且没有上一提示的上下文。

1. 要下载和查看应用生成的图像，请使用 Cloud Shell **下载**命令 - 指定生成的 .png 文件：

    ```
   download ./images/image_1.png
    ```

    下载命令会在浏览器右下角创建一个弹出链接，可以选择此链接下载并打开该文件。

## 总结

在本练习中，你使用 Azure AI Foundry 和 Azure OpenAI SDK 创建客户端应用程序，使用 DALL-E 模型生成图像。

## 清理

如果已完成对 DALL-E 的探索，则应删除在本练习中创建的资源，以避免产生不必要的 Azure 成本。

1. 返回到包含 Azure 门户的浏览器选项卡（或在新的浏览器选项卡中重新打开 [Azure 门户](https://portal.azure.com)，网址为：`https://portal.azure.com`），查看已在其中部署本练习中使用的资源的资源组内容。
1. 在工具栏中，选择“删除资源组”****。
1. 输入资源组名称，并确认要删除该资源组。
