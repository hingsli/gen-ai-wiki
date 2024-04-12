---
layout: post
title: "如何在本地使用大语言模型"
date: 2024-04-10 15:02:34 -0700
categories: "llm"
tags: "lm-studio"
---


# 如何在本地使用大语言模型

要在本地使用大型语言模型，可以考虑使用 OLLaMa 或 LM Studio 这两种工具。

1. **使用 LM Studio**：
    - LM Studio 是一个有UI界面的桌面应用程序，设计用于在本地运行和试验不同的大型语言模型，模型通常从 Hugging Face 下载的。
    - 它提供了一个与 OpenAI 兼容的本地服务器和一个聊天界面，使你可以直接与下载的模型进行交互。
    - LM Studio 支持跨平台使用，包括 Windows、Mac 和 Linux，使其成为一个多功能的本地模型运行解决方案。
2. **使用 OLLaMa**：
    - OLLaMa 支持 Hugging Face 的开源模型，并允许通过命令行直接上传新模型。
    - 你可以通过其命令行界面进行交互，例如使用 **`ollama list`** 来查看可用模型，或者用 **`ollama run <model_name>`** 来运行特定模型。
    - OLLaMa 也支持 GPU 加速，可以通过添加 **`-gpu`** 标志来启用，从而提高模型推理的速度。

更多关于 OLLaMa 的使用方法和下载选项，可以访问 [OLLaMa GitHub 页面](https://github.com/jmorganca/ollama)。关于 LM Studio 的详细信息和下载，可以查看 [LM Studio 官方网站](https://lmstudio.ai/)。

## LM Studio

1. **安装 LM Studio**：首先，需要安装 LM Studio。可以从其官方网站 [LM Studio](https://lmstudio.ai/) 下载安装程序。
    
    ![Screenshot 2024-04-12 at 20.04.03.png](assets/how-to-use-llm-locally/Screenshot_2024-04-12_at_20.04.03.png)
    
2. **下载 LLM**：安装完成后，打开 LM Studio，使用应用内浏览器搜索并下载选择的大型语言模型（LLM），例如从 Hugging Face。下载的模型大小大约为 4GB。
3. **设置本地服务器**：在 LM Studio 中，转到“Local Server”标签页。在这里，你可以加载已下载的任何 LLM。
    
    ![Screenshot 2024-04-12 at 20.04.30.png](assets/how-to-use-llm-locally/Screenshot_2024-04-12_at_20.04.30.png)
    
4. **配置和运行**：选择你下载的 LLM，并设置为在本地服务器上运行。你可以在这一步配置额外的设置，如 GPU 加速等。
5. **使用 API**：配置完本地服务器后，LM Studio 提供一个与 OpenAI API 兼容的本地服务器。你可以像使用 OpenAI 的在线服务一样，通过设置的本地端点调用模型。在终端里面测试API端点：
    
    ![Untitled](assets/how-to-use-llm-locally/Untitled.png)