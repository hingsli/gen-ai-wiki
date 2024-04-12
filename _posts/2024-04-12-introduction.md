---
layout: post
title: "AutoGen基本介绍"
date: 2024-04-12 15:01:34 -0700
categories: autogen
---

# 1. 简介

## 1. 安装

在本地安装 AutoGen 时，推荐使用虚拟环境进行安装，以确保 AutoGen 的依赖项不会与系统中的其他部分冲突。

可以选择使用 venv、Conda 或 Poetry 创建并激活虚拟环境。例如，如果你选择 venv，可以通过以下命令创建并激活虚拟环境：

```bash
python3 -m venv pyautogen
source pyautogen/bin/activate
```

当不再需要虚拟环境时，可以通过运行 **`deactivate`** 命令来停用它。

接下来，安装 AutoGen。AutoGen 要求 Python 版本在 3.8 到 3.13 之间。可以通过 pip 命令来安装 AutoGen：

```bash
pip install pyautogen
```

需要注意的是，如果你安装的是 pyautogen 版本小于 0.2，则需要 openai 版本小于 1。从 pyautogen 版本 0.2 开始，要求 openai 版本必须大于等于 1。

## 2. 代理（Agent）

代理（Agent）是一种能够在其环境中与其他代理发送和接收消息的实体。代理可以由模型（如 GPT-4 这样的大型语言模型）、代码执行器（如 IPython 内核）、人类，或这些以及其他可插拔和可自定义组件的组合来驱动。

其中一个代理的例子是内置的 `**ConversableAgent**`，它支持以下组件：

- 一系列的 LLMs
- 一个代码执行器
- 一个函数和工具执行器
- 一个保持人在循环（human-in-the-loop）中的组件

```python
import os
from autogen import ConversableAgent

agent = ConversableAgent(
    "chatbot",
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ.get("OPENAI_API_KEY")}]},
    code_execution_config=False,  # 关闭代码执行，默认情况下为关闭。
    function_map=None,  # 没有注册的函数，默认为 None。
    human_input_mode="NEVER",  # 从不要求人工输入。
)
```

这段代码展示了如何使用 AutoGen 框架创建一个名为 **`chatbot`** 的 **`ConversableAgent`**。该代理配置了使用 GPT-4 模型，其中 API 密钥从环境变量中获取。代码执行功能被禁用，没有注册任何函数，并且设置为不请求人类输入（**`human_input_mode`** 设置为 "NEVER"）。这样配置的代理完全自动化，不涉及人类干预，适用于需要完全自动回复的场景。

接下来可以通过调用 **`ConversableAgent`** 的 **`generate_reply`** 方法生成对消息的回复。在此例中，传入的消息内容是用户（"role": "user"）请求：“Tell me a joke.”（告诉我一个笑话）。然后，使用 **`print`** 函数打印出代理的回应。这允许检验代理如何利用其配置（如LLM模型）来生成自然语言回应。

```python
reply = agent.generate_reply(messages=[{"content": "Tell me a joke.", "role": "user"}])
print(reply)
```

```
Sure, here's a light-hearted joke for you:

Why don't scientists trust atoms?

Because they make up everything!
```

## 3. 角色和对话

"Roles" 和 "Conversations" 是用于定义代理如何交互和通信的关键概念。"Roles" 指的是分配给每个代理的特定职责或角色，这些角色定义了代理在对话中的行为方式和它们如何响应其他代理的消息。例如，你可以设置一个代理作为提问者，而另一个代理则作为回答者。

"Conversations" 则是代理之间交换消息的序列，用于完成特定任务或达成共同目标。在 AutoGen 中，你可以通过设定系统消息来指定不同代理的角色，并使用这些角色来引导对话的流程和结构。通过这种方式，开发者可以设计复杂的对话流程，使代理能够有效地协同工作，解决问题或执行任务。

这种多代理对话框架不仅支持简单的一对一对话，还能扩展到更复杂的多代理互动，这在需要复杂决策和多步骤处理的应用中尤为重要。例如，可以设置一个群聊，其中多个代理根据特定的转换规则相互回应，共同完成一个复杂任务 ([Microsoft on GitHub](https://microsoft.github.io/autogen/docs/tutorial/introduction/)) ([Microsoft on GitHub](https://microsoft.github.io/autogen/))。

```python
cathy = ConversableAgent(
    "cathy",
    system_message="Your name is Cathy and you are a part of a duo of comedians.",  # 你的名字是Cathy，你是喜剧双人组的一部分。
    llm_config={"config_list": [{"model": "gpt-4", "temperature": 0.9, "api_key": os.environ.get("OPENAI_API_KEY")}]},  # 配置GPT-4模型，设定温度为0.9。
    human_input_mode="NEVER",  # 从不请求人类输入。
)

joe = ConversableAgent(
    "joe",
    system_message="Your name is Joe and you are a part of a duo of comedians.",  # 你的名字是Joe，你是喜剧双人组的一部分。
    llm_config={"config_list": [{"model": "gpt-4", "temperature": 0.7, "api_key": os.environ.get("OPENAI_API_KEY")}]},  # 配置GPT-4模型，设定温度为0.7。
    human_input_mode="NEVER",  # 从不请求人类输入。
)
```

这段代码主要用于创建两个具有不同特性的聊天机器人代理（Cathy 和 Joe）。每个代理都被赋予了一个角色，即他们是一组喜剧演员的一部分，这可能影响他们生成的回复内容的风格和类型。通过调整 **`llm_config`** 中的 **`temperature`** 参数，可以控制每个代理回复的随机性和创造性，温度越高，回复越新颖和不可预测。这两个代理都设置为不需要人类输入，意味着他们将完全自动地生成回答。这样的设置使得这两个代理适用于自动化的娱乐对话场景，例如自动进行喜剧表演或对话。

我们可以使用 AutoGen 框架中的 **`ConversableAgent`** 类型的代理进行对话初始化。在这个示例中，代理 Joe 向代理 Cathy 发起对话请求，内容是“Cathy, tell me a joke.”（Cathy, 告诉我一个笑话）。**`max_turns=2`** 参数指定了对话的最大回合数为2，这意味着对话将在两个回合后自动结束，无论是否达到了自然的结尾。

```python
result = joe.initiate_chat(cathy, message="Cathy, tell me a joke.", max_turns=2)
```

```
joe (to cathy):

Cathy, tell me a joke.

--------------------------------------------------------------------------------
cathy (to joe):

Sure, here's one for you:

Why don't scientists trust atoms?

Because they make up everything!

--------------------------------------------------------------------------------
joe (to cathy):

Haha, that's a good one, Cathy! Okay, my turn. 

Why don't we ever tell secrets on a farm?

Because the potatoes have eyes, the corn has ears, and the beans stalk.

--------------------------------------------------------------------------------
cathy (to joe):

Haha, that's a great one! A farm is definitely not the place for secrets. Okay, my turn again. 

Why couldn't the bicycle stand up by itself?

Because it was two-tired!

--------------------------------------------------------------------------------
```

# 2. 如何终止 AutoGen 代理之间的对话

终止代理间的对话主要有两种方法：

1. **通过初始化聊天时设置参数**：在启动对话时（使用 **`initiate_chat`** 方法），可以设定特定的参数来定义何时结束对话。例如，通过设置 **`max_turns`** 参数来限制对话的回合数，这样对话在达到指定回合后自动终止。
2. **通过配置代理触发终止**：在定义每个代理时，可以指定让代理根据特定条件自动终止对话。这包括设置如 **`max_consecutive_auto_reply`**（连续自动回复的最大次数）和 **`is_termination_msg`**（是否接收到特定的终止消息）等参数。

## 1. 设置 `initiate_chat` 中的参数

以下这段代码创建两个名为 Cathy 和 Joe 的 **`ConversableAgent`** 对象，这两个代理被配置为喜剧双人组的一部分。每个代理都配置了 GPT-4 模型，并且设置了不请求人类输入的模式（**`human_input_mode="NEVER"`**）。

```python
import os  # 导入os模块

from autogen import ConversableAgent  # 从autogen模块导入ConversableAgent类

# 创建代理Cathy
cathy = ConversableAgent(
    "cathy",
    system_message="Your name is Cathy and you are a part of a duo of comedians.",  # 系统消息，设定Cathy的角色和背景
    llm_config={"config_list": [{"model": "gpt-4", "temperature": 0.9, "api_key": os.environ.get("OPENAI_API_KEY")}]},  # 配置使用GPT-4模型，温度设为0.9
    human_input_mode="NEVER",  # 设置为不请求人类输入
)

# 创建代理Joe
joe = ConversableAgent(
    "joe",
    system_message="Your name is Joe and you are a part of a duo of comedians.",  # 系统消息，设定Joe的角色和背景
    llm_config={"config_list": [{"model": "gpt-4", "temperature": 0.7, "api_key": os.environ.get("OPENAI_API_KEY")}]},  # 配置使用GPT-4模型，温度设为0.7
    human_input_mode="NEVER",  # 设置为不请求人类输入
)

# Joe 向 Cathy 发起对话，请求Cathy讲一个笑话，对话最多进行两轮
result = joe.initiate_chat(cathy, message="Cathy, tell me a joke.", max_turns=2)

```

```
joe (to cathy):

Cathy, tell me a joke.

--------------------------------------------------------------------------------
cathy (to joe):

Sure, here's one for you:

Why don't scientists trust atoms?

Because they make up everything!

--------------------------------------------------------------------------------
joe (to cathy):

Haha, that's a good one, Cathy! Okay, my turn.

Why don't we ever tell secrets on a farm?

Because the potatoes have eyes, the corn has ears, and the beans stalk.

--------------------------------------------------------------------------------
cathy (to joe):

Haha, that's a great one! A farm is definitely not the place for secrets. Okay, my turn again.

Why couldn't the bicycle stand up by itself?

Because it was two-tired!

--------------------------------------------------------------------------------

```

通过 **`max_turns=2`** 参数限制对话最多进行两个回合。这种设置用于简短的互动，如快速的娱乐对话。

```python
result = joe.initiate_chat(
    cathy, message="Cathy, tell me a joke.", max_turns=3  # 向Cathy发起对话，要求她讲一个笑话，设置对话最大轮数为3轮
) 
```

```
joe (to cathy):

Cathy, tell me a joke.

--------------------------------------------------------------------------------
cathy (to joe):

Sure, here's one for you:

Why don't scientists trust atoms?

Because they make up everything!

--------------------------------------------------------------------------------
joe (to cathy):

Haha, that's a good one, Cathy! Okay, my turn. 

Why don't we ever tell secrets on a farm?

Because the potatoes have eyes, the corn has ears, and the beans stalk.

--------------------------------------------------------------------------------
cathy (to joe):

Haha, that's a great one! A farm is definitely not the place for secrets. Okay, my turn again. 

Why couldn't the bicycle stand up by itself?

Because it was two-tired!

--------------------------------------------------------------------------------
joe (to cathy):

Haha, that's a wheely good one, Cathy!

Why did the golfer bring two pairs of pants?

In case he got a hole in one!

--------------------------------------------------------------------------------
cathy (to joe):

Haha, that's a perfect swing of a joke!

Why did the scarecrow win an award?

Because he was outstanding in his field!

--------------------------------------------------------------------------------
```

此处设置对话的最大轮数为3轮，即 **`max_turns=3`**。

## 2. 配置**`max_consecutive_auto_reply`**

我们创建一个名为 **`joe`** 的 **`ConversableAgent`** 代理，配置了其使用的大型语言模型（LLM）GPT-4，温度参数设置为0.7以调整生成文本的随机性。此外，**`joe`** 被配置为一个喜剧表演者的角色，并设置为不接受人类输入，同时限制了在没有人类干预的情况下连续自动回复的最大次数为1次。

然后，**`joe`** 使用 **`initiate_chat`** 方法向 **`cathy`** 发起对话请求，内容是请求对方讲一个笑话。这种设置适用于进行简短而有控制的自动对话，尤其是在娱乐或表演场景中。

```python
joe = ConversableAgent(
    "joe",
    system_message="Your name is Joe and you are a part of a duo of comedians.",  # 设置系统消息，定义Joe的角色为喜剧双人组的一部分
    llm_config={"config_list": [{"model": "gpt-4", "temperature": 0.7, "api_key": os.environ.get("OPENAI_API_KEY")}]},  # 配置使用GPT-4模型，温度设置为0.7
    human_input_mode="NEVER",  # 设置为不请求人类输入
    max_consecutive_auto_reply=1,  # 限制连续自动回复的最大次数为1
)

result = joe.initiate_chat(cathy, message="Cathy, tell me a joke.")  # Joe向Cathy发起对话，内容是请求讲一个笑话

```

```
joe (to cathy):

Cathy, tell me a joke.

--------------------------------------------------------------------------------
cathy (to joe):

Sure, here's one for you:

Why don't scientists trust atoms?

Because they make up everything!

--------------------------------------------------------------------------------
joe (to cathy):

Haha, that's a good one, Cathy! Okay, my turn. 

Why don't we ever tell secrets on a farm?

Because the potatoes have eyes, the corn has ears, and the beans stalk.

--------------------------------------------------------------------------------
cathy (to joe):

Haha, that's a great one! A farm is definitely not the place for secrets. Okay, my turn again. 

Why couldn't the bicycle stand up by itself?

Because it was two-tired!

--------------------------------------------------------------------------------
```

## 3. 使用**`is_termination_msg`**

我们可以为 **`joe`** 这个 **`ConversableAgent`** 设置一些参数，包括一个特定的终止对话的条件。这个条件是如果收到的消息中包含 "good bye"，则会终止对话。

```python
joe = ConversableAgent(
    "joe",
    system_message="Your name is Joe and you are a part of a duo of comedians.",  # 设置系统消息，定义Joe为喜剧双人组一员
    llm_config={"config_list": [{"model": "gpt-4", "temperature": 0.7, "api_key": os.environ.get("OPENAI_API_KEY")}]},  # 配置使用GPT-4模型，温度设为0.7
    human_input_mode="NEVER",  # 设置不请求人类输入
    is_termination_msg=lambda msg: "good bye" in msg["content"].lower(),  # 如果消息内容包含"good bye"则触发对话终止
)

result = joe.initiate_chat(cathy, message="Cathy, tell me a joke and then say the words GOOD BYE.")  # Joe请求Cathy讲个笑话然后说"GOOD BYE"

```

```
joe (to cathy):

Cathy, tell me a joke and then say the words GOOD BYE.

--------------------------------------------------------------------------------
cathy (to joe):

Why don't scientists trust atoms?

Because they make up everything!

GOOD BYE!

--------------------------------------------------------------------------------
```

# 3. 如何在agent间对话中引入人类的干预

引入人类干预的主要方式是通过设置 **`ConversableAgent`** 的 **`human_input_mode`** 参数。这个参数支持三种模式：**`NEVER`**（从不请求人类输入）、**`TERMINATE`**（仅在满足终止条件时请求人类输入）、和 **`ALWAYS`**（总是请求人类输入）。这允许代理在需要时包含人类反馈，例如，通过人类输入来引导代理或校正其行为。详细信息可以在 AutoGen 的官方文档中查阅：[Allowing Human Feedback in Agents](https://microsoft.github.io/autogen/docs/tutorial/human-in-the-loop/)。

![Untitled](assets/introduction/Untitled.png)

## 1. 人类输入模式 = `NEVER`

我们创建了两个 **`ConversableAgent`** 代理，用于执行一个猜数字的游戏。其中一个代理 “agent_with_number” 充当游戏主持人，拥有一个数字（53），并指导另一个代理 “agent_guess_number” 来猜测这个数字。如果猜测的数字过高或过低，主持人会相应地反馈。游戏继续进行直到猜中数字，触发对话终止条件。这种设置演示了如何使用 AutoGen 框架来创建交互式的对话游戏。

```python
import os  # 导入os模块

from autogen import ConversableAgent  # 从autogen模块导入ConversableAgent类

# 创建一个代理，代表一个猜数字游戏的主持人
agent_with_number = ConversableAgent(
    "agent_with_number",
    system_message="You are playing a game of guess-my-number. You have the "
    "number 53 in your mind, and I will try to guess it. "
    "If I guess too high, say 'too high', if I guess too low, say 'too low'. ",  # 游戏规则说明
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 使用GPT-4模型
    is_termination_msg=lambda msg: "53" in msg["content"],  # 如果对方猜中53，则终止对话
    human_input_mode="NEVER",  # 不请求人类输入
)

# 创建另一个代理，用来猜数字
agent_guess_number = ConversableAgent(
    "agent_guess_number",
    system_message="I have a number in my mind, and you will try to guess it. "
    "If I say 'too high', you should guess a lower number. If I say 'too low', "
    "you should guess a higher number. ",  # 游戏规则说明
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 使用GPT-4模型
    human_input_mode="NEVER",
)

# 主持人代理发起与猜数字代理的对话
result = agent_with_number.initiate_chat(
    agent_guess_number,
    message="I have a number between 1 and 100. Guess it!",  # 发起对话的消息
)

```

```
agent_with_number (to agent_guess_number):

I have a number between 1 and 100. Guess it!

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 50?

--------------------------------------------------------------------------------
agent_with_number (to agent_guess_number):

Too low.

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 75?

--------------------------------------------------------------------------------
agent_with_number (to agent_guess_number):

Too high.

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 63?

--------------------------------------------------------------------------------
agent_with_number (to agent_guess_number):

Too high.

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 57?

--------------------------------------------------------------------------------
agent_with_number (to agent_guess_number):

Too high.

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 54?

--------------------------------------------------------------------------------
agent_with_number (to agent_guess_number):

Too high.

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 52?

--------------------------------------------------------------------------------
agent_with_number (to agent_guess_number):

Too low.

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 53?

--------------------------------------------------------------------------------
```

## 2. 人类输入模式 = `ALWAYS`

我们创建一个名为 **`human_proxy`** 的 **`ConversableAgent`**，这个代理被设置为始终请求人类输入，不使用任何语言模型（LLM），主要用于模拟真人参与对话。通过这个代理，我们发起了一次对话，初始猜测数字为10，与之前设置的有数字的代理 **`agent_with_number`** 进行互动。

```python
human_proxy = ConversableAgent(
    "human_proxy",
    llm_config=False,  # 不使用LLM配置，此代理用于人类代理
    human_input_mode="ALWAYS",  # 始终请求人类输入
)

# 使用人类代理启动与猜数字代理的聊天，并进行初始猜测
result = human_proxy.initiate_chat(
    agent_with_number,  # 这是之前的拥有数字的代理
    message="10",  # 初始猜测消息
)

```

```
human_proxy (to agent_with_number):

10

--------------------------------------------------------------------------------
agent_with_number (to human_proxy):

Too low.

--------------------------------------------------------------------------------
human_proxy (to agent_with_number):

79

--------------------------------------------------------------------------------
agent_with_number (to human_proxy):

Too high.

--------------------------------------------------------------------------------
human_proxy (to agent_with_number):

76

--------------------------------------------------------------------------------
agent_with_number (to human_proxy):

Too high.

--------------------------------------------------------------------------------
human_proxy (to agent_with_number):

I give up

--------------------------------------------------------------------------------
agent_with_number (to human_proxy):

That's okay! The number I was thinking of was 53.

--------------------------------------------------------------------------------
```

## 3. 人类输入模式 = `TERMINATE`

我们配置了两个 **`ConversableAgent`** 以模拟猜数字游戏。**`agent_with_number`** 有一个数字（53）并设置了游戏规则。如果另一个代理 **`agent_guess_number`** 猜测过高或过低，它会相应反馈，并在猜中正确数字时终止对话。**`human_input_mode="TERMINATE"`** 确保在满足特定条件前，人类可以介入游戏。这个设置允许测试代理在有限交互下的表现。

```python
agent_with_number = ConversableAgent(
    "agent_with_number",
    system_message="You are playing a game of guess-my-number. "
    "In the first game, you have the "
    "number 53 in your mind, and I will try to guess it. "
    "If I guess too high, say 'too high', if I guess too low, say 'too low'. ",  # 游戏规则说明
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 配置使用GPT-4模型
    max_consecutive_auto_reply=1,  # 在请求人类输入之前的最大连续自动回复数
    is_termination_msg=lambda msg: "53" in msg["content"],  # 如果猜中数字53，则终止对话
    human_input_mode="TERMINATE",  # 在游戏终止前始终请求人类输入
)

agent_guess_number = ConversableAgent(
    "agent_guess_number",
    system_message="I have a number in my mind, and you will try to guess it. "
    "If I say 'too high', you should guess a lower number. If I say 'too low', "
    "you should guess a higher number. ",  # 猜测规则说明
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 配置使用GPT-4模型
    human_input_mode="NEVER",  # 不请求人类输入
)

# 启动代理之间的猜数字游戏
result = agent_with_number.initiate_chat(
    agent_guess_number,
    message="I have a number between 1 and 100. Guess it!",  # 发起猜数字的消息
)

```

```
agent_with_number (to agent_guess_number):

I have a number between 1 and 100. Guess it!

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 50?

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
agent_with_number (to agent_guess_number):

Too low.

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 75?

--------------------------------------------------------------------------------
agent_with_number (to agent_guess_number):

It is too high my friend. 

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 60?

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
agent_with_number (to agent_guess_number):

Too high.

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 55?

--------------------------------------------------------------------------------
agent_with_number (to agent_guess_number):

still too high, but you are very close.

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 52?

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
agent_with_number (to agent_guess_number):

Too low.

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 54?

--------------------------------------------------------------------------------
agent_with_number (to agent_guess_number):

Almost there! 

--------------------------------------------------------------------------------
agent_guess_number (to agent_with_number):

Is it 53?

--------------------------------------------------------------------------------
```

# 4. 代码执行器

"code executor"（代码执行器）是一个特定类型的代理，负责执行其他代理或用户生成的代码。代码执行器可以与代码编写代理进行交互，类似于程序员与代码解释器之间的互动。这种设置使得可以在对话中编写并执行代码，适用于数据分析、机器学习、数学建模等需要动态代码执行的任务。

AutoGen 支持多种类型的代码执行器，包括命令行代码执行器和 Jupyter 代码执行器。命令行代码执行器通过在 Docker 容器中执行每个代码块来提高安全性，而 Jupyter 代码执行器则通过 Jupyter 服务器状态地执行代码，允许跨多个执行步骤访问和操作变量。

代码执行器可以集成到 AutoGen 的多代理对话中，允许创建复杂的编程和执行工作流。例如，一个代理可以负责编写代码，另一个代理则负责执行该代码并返回结果，这种模式增强了 AutoGen 在处理需要代码交互的复杂任务中的应用灵活性，更多内容参考官方文档[Code Executors](https://microsoft.github.io/autogen/docs/tutorial/code-executors)。

| Code Executor (autogen.coding) | Environment | Platform |
| --- | --- | --- |
| https://microsoft.github.io/autogen/docs/reference/coding/local_commandline_code_executor#localcommandlinecodeexecutor | Shell | Local |
| https://microsoft.github.io/autogen/docs/reference/coding/docker_commandline_code_executor#dockercommandlinecodeexecutor | Shell | Docker |
| https://microsoft.github.io/autogen/docs/reference/coding/jupyter/jupyter_code_executor#jupytercodeexecutor | Jupyter Kernel (e.g., python3) | Local/Docker |

## 1. **`LocalCommandLineCodeExecutor`**

![Untitled](assets/introduction/Untitled%201.png)

验证本节例子前，先安装：

```bash
! pip install -qqq matplotlib numpy
```

接下里我们看看如何在 AutoGen 框架中配置和使用一个本地命令行代码执行器。首先，使用 **`tempfile`** 创建一个临时目录来存放执行过程中生成的代码文件。然后，实例化 **`LocalCommandLineCodeExecutor`** 对象，指定代码执行的超时时间和工作目录。接着，创建一个 **`ConversableAgent`**，将 LLM 功能关闭，配置它使用前面创建的代码执行器，并设置该代理始终需要人类输入，这是为了确保在执行代码时的安全性。这种配置适用于需要执行外部代码或脚本的应用场景，例如自动化测试、数据处理等。

```bash
import tempfile  # 导入tempfile模块，用于创建临时文件目录

from autogen import ConversableAgent  # 从autogen模块导入ConversableAgent类
from autogen.coding import LocalCommandLineCodeExecutor  # 导入LocalCommandLineCodeExecutor类

# 创建一个临时目录来存储代码文件
temp_dir = tempfile.TemporaryDirectory()

# 创建一个本地命令行代码执行器
executor = LocalCommandLineCodeExecutor(
    timeout=10,  # 每次代码执行的超时时间，单位为秒
    work_dir=temp_dir.name,  # 使用临时目录来存储代码文件
)

# 创建一个配置了代码执行器的代理
code_executor_agent = ConversableAgent(
    "code_executor_agent",
    llm_config=False,  # 关闭此代理的LLM功能
    code_execution_config={"executor": executor},  # 使用本地命令行代码执行器
    human_input_mode="ALWAYS",  # 此代理始终需要人类输入，以确保安全
)

```

然后我们来看看如何使用一个配置了代码执行功能的 `ConversableAgent` 来处理并回复包含 Python 代码块的消息。代码块尝试创建一个散点图并将其保存为图片文件。首先，定义一个包含 Python 代码块的消息字符串，然后将这个消息作为输入传递给 `code_executor_agent` 的 `generate_reply` 方法。该方法处理输入消息，执行其中的代码，并生成一个回复，这个回复可能是代码执行的结果或相关输出。最后，使用 `print` 函数打印出生成的回复。这种方法适用于需要自动处理和执行代码的场景，如自动数据可视化、计算任务等。

```
message_with_code_block = """This is a message with code block.
The code block is below:
```python
import numpy as np
import matplotlib.pyplot as plt
x = np.random.randint(0, 100, 100)
y = np.random.randint(0, 100, 100)
plt.scatter(x, y)
plt.savefig('scatter.png')
print('Scatter plot saved to scatter.png')
```
This is the end of the message.
"""

# 为给定的代码生成回复
reply = code_executor_agent.generate_reply(messages=[{"role": "user", "content": message_with_code_block}])
print(reply)
```

```

>>>>>>>> NO HUMAN INPUT RECEIVED.

>>>>>>>> USING AUTO REPLY...

>>>>>>>> EXECUTING CODE BLOCK (inferred language is python)...
exitcode: 0 (execution succeeded)
Code output: 
Scatter plot saved to scatter.png
```

此时可以查看在前面代码执行过程中在临时目录 **`temp_dir`** 下生成的文件。通过调用 **`os.listdir`** 函数，代码列出了指定目录（这里是存储代码文件的临时目录）中的所有文件和目录。这样可以验证代码执行器是否成功生成了预期的文件（如 **`scatter.png`** 散点图图片文件），以及是否有其他相关的代码文件被创建。此操作常用于调试和验证代码执行结果，确保生成的输出符合预期。

```python
import os  # 导入os模块

print(os.listdir(temp_dir.name))  # 打印临时目录中的文件列表
# 我们可以看到输出文件scatter.png和代理生成的代码文件。
```

```
['scatter.png', '6507ea07b63b45aabb027ade4e213de6.py']
```

最后清除临时文件夹内容：

```python
temp_dir.cleanup()
```

## 2. 在Docker中执行

![Untitled](assets/introduction/Untitled%202.png)

在本小节中，我们来看看如何在 AutoGen 框架中配置和使用 Docker 命令行代码执行器。

首先，通过 **`tempfile.TemporaryDirectory()`** 创建一个临时目录用于存放代码执行过程中生成的文件。然后，实例化 **`DockerCommandLineCodeExecutor`**，指定 Docker 镜像、执行超时时间以及工作目录。接下来，创建一个名为 **`code_executor_agent_using_docker`** 的 **`ConversableAgent`**，配置为使用前面创建的 Docker 代码执行器，且设置为始终需要人类输入以确保执行过程中的安全性。

这种配置允许代理在受控的 Docker 环境中执行代码，适用于需要隔离执行环境以提高安全性的场景。例如，执行未知或潜在危险的代码时，使用 Docker 可以防止对主机系统造成影响。

```python
from autogen.coding import DockerCommandLineCodeExecutor  # 从autogen.coding导入DockerCommandLineCodeExecutor

# 创建一个临时目录来存储代码文件
temp_dir = tempfile.TemporaryDirectory()

# 创建一个Docker命令行代码执行器
executor = DockerCommandLineCodeExecutor(
    image="python:3.12-slim",  # 使用指定的Docker镜像名执行代码
    timeout=10,  # 每次代码执行的超时时间，单位为秒
    work_dir=temp_dir.name,  # 使用临时目录来存储代码文件
)

# 创建一个配置了Docker代码执行器的代理
code_executor_agent_using_docker = ConversableAgent(
    "code_executor_agent_docker",
    llm_config=False,  # 关闭此代理的LLM功能
    code_execution_config={"executor": executor},  # 使用Docker命令行代码执行器
    human_input_mode="ALWAYS",  # 此代理始终需要人类输入，以确保安全
)

# 当代码执行器不再使用时，停止它以释放资源
# executor.stop()

```

## 3. 在对话中使用代码执行器

![Untitled](assets/introduction/Untitled%203.png)

我们定义一个名为 **`code_writer_agent`** 的 **`ConversableAgent`**，它被设定为一个帮助解决任务的智能助理。这个代理专门用于编写并建议执行代码（Python或Shell脚本），以解决用户面对的问题。系统消息详细说明了代理如何指导任务的解决，包括信息收集、任务执行、错误处理和结果验证等步骤。此外，代理被配置为不执行代码，侧重于生成和建议代码。这样的设置使得代理可以安全地用于环境中，引导用户或其他代理执行代码，同时确保代码的正确性和完整性。

```python
# 代码编写代理的系统消息是指导LLM如何使用代码执行代理中的代码执行器
code_writer_system_message = """You are a helpful AI assistant.
Solve tasks using your coding and language skills.
In the following cases, suggest python code (in a python coding block) or shell script (in a sh coding block) for the user to execute.
1. When you need to collect info, use the code to output the info you need, for example, browse or search the web, download/read a file, print the content of a webpage or a file, get the current date/time, check the operating system. After sufficient info is printed and the task is ready to be solved based on your language skill, you can solve the task by yourself.
2. When you need to perform some task with code, use the code to perform the task and output the result. Finish the task smartly.
Solve the task step by step if you need to. If a plan is not provided, explain your plan first. Be clear which step uses code, and which step uses your language skill.
When using code, you must indicate the script type in the code block. The user cannot provide any other feedback or perform any other action beyond executing the code you suggest. The user can't modify your code. So do not suggest incomplete code which requires users to modify. Don't use a code block if it's not intended to be executed by the user.
If you want the user to save the code in a file before executing it, put # filename: <filename> inside the code block as the first line. Don't include multiple code blocks in one response. Do not ask users to copy and paste the result. Instead, use 'print' function for the output when relevant. Check the execution result returned by the user.
If the result indicates there is an error, fix the error and output the code again. Suggest the full code instead of partial code or code changes. If the error can't be fixed or if the task is not solved even after the code is executed successfully, analyze the problem, revisit your assumption, collect additional info you need, and think of a different approach to try.
When you find an answer, verify the answer carefully. Include verifiable evidence in your response if possible.
Reply 'TERMINATE' in the end when everything is done.
"""

# 创建一个名为code_writer_agent的代码编写代理，配置系统消息并关闭代码执行功能
code_writer_agent = ConversableAgent(
    "code_writer_agent",
    system_message=code_writer_system_message,
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 使用GPT-4模型
    code_execution_config=False,  # 关闭此代理的代码执行功能
)

```

**`code_executor_agent`** 代理向 **`code_writer_agent`** 代理发送请求，这个过程模拟了在实际开发环境中，一个团队成员请求另一个团队成员解决特定编程问题的场景。

```python
chat_result = code_executor_agent.initiate_chat(
    code_writer_agent,
    message="Write Python code to calculate the 14th Fibonacci number.",
)
# 使用代码执行代理向代码编写代理发起聊天，请求编写计算第14个斐波那契数的Python代码

```

```
code_executor_agent (to code_writer_agent):

Write Python code to calculate the 14th Fibonacci number.

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
code_writer_agent (to code_executor_agent):

Sure, here is a Python code snippet to calculate the 14th Fibonacci number. The Fibonacci series is a sequence of numbers in which each number is the sum of the two preceding ones, usually starting with 0 and 1.

```python
def fibonacci(n):
    if(n <= 0):
        return "Input should be a positive integer."
    elif(n == 1):
        return 0
    elif(n == 2):
        return 1
    else:
        fib = [0, 1]
        for i in range(2, n):
            fib.append(fib[i-1] + fib[i-2])
        return fib[n-1]

print(fibonacci(14))
```

This Python code defines a function `fibonacci(n)` which computes the n-th Fibonacci number. The function uses a list `fib` to store the Fibonacci numbers as they are computed, and then returns the (n-1)-th element as the n-th Fibonacci number due to zero-indexing in Python lists.

--------------------------------------------------------------------------------

>>>>>>>> NO HUMAN INPUT RECEIVED.

>>>>>>>> USING AUTO REPLY...

>>>>>>>> EXECUTING CODE BLOCK (inferred language is python)...
code_executor_agent (to code_writer_agent):

exitcode: 0 (execution succeeded)
Code output: 
233

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
code_writer_agent (to code_executor_agent):

Great, the execution was successful and the 14th Fibonacci number is 233. The sequence goes as follows: 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233... and so on, where each number is the sum of the previous two. Therefore, the 14th number in the Fibonacci series is 233. 

I hope this meets your expectations. If you have any other concerns or need further computations, feel free to ask.

TERMINATE

--------------------------------------------------------------------------------
```

我们可以使用  **`code_executor_agent`** 向 **`code_writer_agent`** 发起请求，生成绘制特斯拉和Meta年度股价增长的Python代码。

```python
import datetime  # 导入datetime模块用于处理日期和时间

today = datetime.datetime.now().strftime("%Y-%m-%d")  # 获取当前日期并格式化为"YYYY-MM-DD"
chat_result = code_executor_agent.initiate_chat(
    code_writer_agent,
    message=f"Today is {today}. Write Python code to plot TSLA's and META's "
    "stock price gains YTD, and save the plot to a file named 'stock_gains.png'.",
)
# 使用代码执行代理向代码编写代理发起聊天，请求编写Python代码来绘制特斯拉(TSLA)和Meta(META)的年初至今的股价增长，并将结果保存为名为'stock_gains.png'的文件。

```

```
code_executor_agent (to code_writer_agent):

Today is 2024-02-28. Write Python code to plot TSLA's and META's stock price gains YTD, and save the plot to a file named 'stock_gains.png'.

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
code_writer_agent (to code_executor_agent):

This task requires retrieving the historical data of the stocks from a reliable data source and calculating the Year-To-Date (YTD) gain values, and then plotting them. pandas_datareader library will be used for data retrieval, pandas will be used for data manipulation, and matplotlib for plotting. 

Below is the Python code to achieve this. To start, please install the required libraries by running to the following command:
```sh
pip install yfinance pandas matplotlib
```
Then run the python code:

```python
# filename: stock_gains.py

import yfinance as yf
import pandas as pd
import matplotlib.pyplot as plt
from datetime import datetime

# define the tickers
tickers = ['TSLA', 'META'] 

# define the start and end dates
start_date = datetime(2024, 1, 1)
end_date = datetime(2024, 2, 28)

# dictionary to hold dataframes
dfs = {}

for ticker in tickers:
    # get the data for the stocks
    df = yf.download(ticker, start_date, end_date)

    # get the close price and calculate the cumulative percentage gain
    df['Gain'] = df['Close'].pct_change().cumsum()

    # add to dictionary
    dfs[ticker] = df    

# plot
plt.figure(figsize=(10, 5))
for ticker, df in dfs.items():
    plt.plot(df.index, df['Gain'], label=ticker)

plt.title('YTD Stock Price Gain')
plt.xlabel('Date')
plt.ylabel('Percentage Gain')
plt.legend()

plt.grid(True)
plt.savefig('stock_gains.png')
plt.close()

print("The 'stock_gains.png' file has been successfully saved")
```
This script will download the historical data for TSLA and META from the start of the year to the specified date and calculates the YTD gains. It then generates the plot showing these gains and saves it to 'stock_gains.png'.

Please save the script to a file named 'stock_gains.py' and run it using Python. Remember to have the correct start and end dates for the YTD value when running the script. If your Python version is below 3.8, you should update it to execute this code perfectly.

--------------------------------------------------------------------------------

>>>>>>>> NO HUMAN INPUT RECEIVED.

>>>>>>>> USING AUTO REPLY...

>>>>>>>> EXECUTING CODE BLOCK 0 (inferred language is sh)...

>>>>>>>> EXECUTING CODE BLOCK 1 (inferred language is python)...
code_executor_agent (to code_writer_agent):

exitcode: 0 (execution succeeded)
Code output: 
Requirement already satisfied: yfinance in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (0.2.36)
Requirement already satisfied: pandas in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (2.1.4)
Requirement already satisfied: matplotlib in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (3.8.2)
Requirement already satisfied: numpy>=1.16.5 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from yfinance) (1.26.2)
Requirement already satisfied: requests>=2.31 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from yfinance) (2.31.0)
Requirement already satisfied: multitasking>=0.0.7 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from yfinance) (0.0.11)
Requirement already satisfied: lxml>=4.9.1 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from yfinance) (5.0.1)
Requirement already satisfied: appdirs>=1.4.4 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from yfinance) (1.4.4)
Requirement already satisfied: pytz>=2022.5 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from yfinance) (2023.3.post1)
Requirement already satisfied: frozendict>=2.3.4 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from yfinance) (2.4.0)
Requirement already satisfied: peewee>=3.16.2 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from yfinance) (3.17.0)
Requirement already satisfied: beautifulsoup4>=4.11.1 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from yfinance) (4.12.2)
Requirement already satisfied: html5lib>=1.1 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from yfinance) (1.1)
Requirement already satisfied: python-dateutil>=2.8.2 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from pandas) (2.8.2)
Requirement already satisfied: tzdata>=2022.1 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from pandas) (2023.4)
Requirement already satisfied: contourpy>=1.0.1 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from matplotlib) (1.2.0)
Requirement already satisfied: cycler>=0.10 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from matplotlib) (0.12.1)
Requirement already satisfied: fonttools>=4.22.0 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from matplotlib) (4.47.2)
Requirement already satisfied: kiwisolver>=1.3.1 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from matplotlib) (1.4.5)
Requirement already satisfied: packaging>=20.0 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from matplotlib) (23.2)
Requirement already satisfied: pillow>=8 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from matplotlib) (10.2.0)
Requirement already satisfied: pyparsing>=2.3.1 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from matplotlib) (3.1.1)
Requirement already satisfied: soupsieve>1.2 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from beautifulsoup4>=4.11.1->yfinance) (2.5)
Requirement already satisfied: six>=1.9 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from html5lib>=1.1->yfinance) (1.16.0)
Requirement already satisfied: webencodings in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from html5lib>=1.1->yfinance) (0.5.1)
Requirement already satisfied: charset-normalizer<4,>=2 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from requests>=2.31->yfinance) (3.3.2)
Requirement already satisfied: idna<4,>=2.5 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from requests>=2.31->yfinance) (3.6)
Requirement already satisfied: urllib3<3,>=1.21.1 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from requests>=2.31->yfinance) (2.1.0)
Requirement already satisfied: certifi>=2017.4.17 in /Users/ekzhu/miniconda3/envs/autogen/lib/python3.11/site-packages (from requests>=2.31->yfinance) (2024.2.2)

The 'stock_gains.png' file has been successfully saved

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
code_writer_agent (to code_executor_agent):

Great! The code executed successfully and the 'stock_gains.png' file has been saved successfully. This file contains the plot of TSLA's and META's stock price gains from the start of the year until February 28, 2024. You should now be able to view this image file in the same directory that you ran the script from. 

Please make sure to verify this image file. It should contain two plotted lines, each representing the percentage gain over the time for each stock (TSLA and META). The x-axis represents the date, and the y-axis represents the percentage gain. If everything looks correct, this would be the end of the task.

TERMINATE

--------------------------------------------------------------------------------
```

显示这张图像：

```python
from IPython.display import Image  # 从IPython.display模块导入Image类

Image(os.path.join(temp_dir, "stock_gains.png"))  # 使用Image类显示指定路径的图片

```

![Untitled](assets/introduction/Untitled%204.png)

清除临时文件夹：

```python
temp_dir.cleanup()
```

停止docker命令行代码执行器：

```python
executor.stop()  
```

## 4. 命令行代码执行器 VS Jupyter 代码执行器

选择使用命令行代码执行器还是 Jupyter 代码执行器取决于你的代码块的特性和需求：

1. **命令行代码执行器**：这种执行器适用于每个代码块作为独立脚本运行的场景，不需要前后代码块之间保留状态或共享变量。命令行执行器将每个代码块写入一个单独的文件，并在新的进程中执行这些文件，这意味着执行环境在每次运行后都会重置 ([Microsoft on GitHub](https://microsoft.github.io/autogen/docs/topics/code-execution/cli-code-executor/))。
2. **Jupyter 代码执行器**：如果你的代码块涉及复杂计算，如机器学习模型训练或需要加载大量数据，并且希望在执行间保持状态以避免重复计算，Jupyter 执行器会更合适。Jupyter 代码执行器在同一个 Jupyter 内核中运行所有代码块，可以保持变量和状态，适用于需要状态持久化的场景 ([Microsoft on GitHub](https://microsoft.github.io/autogen/docs/topics/code-execution/cli-code-executor/))。

所以，你的选择应基于代码执行的具体需求：是否需要状态持久化，以及是否每个代码块都可以独立执行而不依赖之前的状态。如果你的应用场景需要快速执行并频繁重启或不需状态保持，命令行执行器可能更适合。如果需要长时间运行且依赖之前的计算结果，Jupyter 执行器将是更好的选择。

# 5. 工具（Tool Use）

"Tool Use" 指的是在多代理对话中，代理能够使用各种工具来执行特定的任务。这些工具可以是函数、代码执行器，或者其他可以通过 AutoGen 注册并调用的资源。代理可以通过这些工具来增强它们的功能性，比如进行信息检索、执行计算任务或者其他需要特定能力的操作。

代理使用工具的过程通常涉及以下几个步骤：

1. **工具的注册**：在 AutoGen 中，工具可以通过 **`register_for_llm`** 或 **`register_for_execution`** 方法注册，使得特定的代理可以调用这些工具。这些方法允许代理通过使用工具的签名或函数对象来调用或执行工具的功能。
2. **工具的调用**：一旦注册，代理可以在对话中根据需要调用这些工具来执行任务。例如，代理可以使用代码执行工具来运行代码块，或者使用函数工具来处理数据。
3. **多代理协作**：在复杂的工作流中，不同的代理可能需要使用不同的工具来协作完成任务。AutoGen 支持通过工具使用来促进这种类型的多代理协作，例如在对话中集成人类输入或其他代理的反馈。

## 1. 工具的注册

下面的例子涉及创建和配置两个 **`ConversableAgent`**，一个作为助手，另一个作为用户代理。助手代理被设定为帮助执行简单计算，并在完成任务后返回“TERMINATE”。用户代理则用于与助手代理交互并执行建议的工具调用。

**`assistant`** 代理通过 **`register_for_llm`** 方法注册了一个名为 "calculator" 的工具，这意味着它可以建议使用这个工具进行计算。相对地，**`user_proxy`** 代理通过 **`register_for_execution`** 方法注册同样的工具，使其能够实际执行这些计算。

这种设置允许两个代理协作处理计算任务，其中一个代理负责提供计算逻辑的建议，另一个代理执行这些计算。

```python
import os  # 导入os模块

from autogen import ConversableAgent  # 从autogen模块导入ConversableAgent类

# 定义一个助手代理，提供工具调用建议
assistant = ConversableAgent(
    name="Assistant",  # 代理名称
    system_message="You are a helpful AI assistant. "
    "You can help with simple calculations. "
    "Return 'TERMINATE' when the task is done.",  # 系统消息，描述代理的功能和任务终止条件
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 配置使用GPT-4模型
)

# 用户代理用于与助手代理交互并执行工具调用
user_proxy = ConversableAgent(
    name="User",  # 代理名称
    llm_config=False,  # 关闭LLM配置
    is_termination_msg=lambda msg: msg.get("content") is not None and "TERMINATE" in msg["content"],  # 设置终止对话的条件
    human_input_mode="NEVER",  # 不需要人类输入
)

# 在助手代理上注册工具签名
assistant.register_for_llm(name="calculator", description="A simple calculator")(calculator)

# 在用户代理上注册工具函数
user_proxy.register_for_execution(name="calculator")(calculator)

```

另一个方法：可以使用 AutoGen 框架的 **`register_function`** 方法将一个计算器功能注册到两个不同的代理。这里的 **`calculator`** 是一个函数，代表一个简单的计算器功能。通过注册，助手代理（**`assistant`**）被授权可以建议调用此计算器功能，而用户代理（**`user_proxy`**）被授权执行这些调用。

```python
from autogen import register_function  # 从autogen模块导入register_function函数

# 向两个代理注册计算器功能
register_function(
    calculator,  # calculator函数是要注册的具体功能
    caller=assistant,  # 助手代理可以建议调用计算器
    executor=user_proxy,  # 用户代理可以执行计算器调用
    name="calculator",  # 默认情况下，使用函数名作为工具名称
    description="A simple calculator",  # 工具的描述
)

```

## 2. 工具的调用

注册好工具后，就可以使用工具：

```python
chat_result = user_proxy.initiate_chat(assistant, message="What is (44232 + 13312 / (232 - 32)) * 5?")
```

```
User (to Assistant):

What is (44232 + 13312 / (232 - 32)) * 5?

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
Assistant (to User):

***** Suggested tool Call (call_bACquf0OreI0VHh7rWiP6ZE7): calculator *****
Arguments: 
{
  "a": 13312,
  "b": 232 - 32,
  "operator": "/"
}
***************************************************************************

--------------------------------------------------------------------------------
User (to Assistant):

User (to Assistant):

***** Response from calling tool "call_bACquf0OreI0VHh7rWiP6ZE7" *****
Error: Expecting ',' delimiter: line 1 column 26 (char 25)
 You argument should follow json format.
**********************************************************************

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
Assistant (to User):

***** Suggested tool Call (call_2c0H5gzX9SWsJ05x7nEOVbav): calculator *****
Arguments: 
{
  "a": 13312,
  "b": 200,
  "operator": "/"
}
***************************************************************************

--------------------------------------------------------------------------------

>>>>>>>> EXECUTING FUNCTION calculator...
User (to Assistant):

User (to Assistant):

***** Response from calling tool "call_2c0H5gzX9SWsJ05x7nEOVbav" *****
66
**********************************************************************

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
Assistant (to User):

***** Suggested tool Call (call_ioceLhuKMpfU131E7TSQ8wCD): calculator *****
Arguments: 
{
  "a": 44232,
  "b": 66,
  "operator": "+"
}
***************************************************************************

--------------------------------------------------------------------------------

>>>>>>>> EXECUTING FUNCTION calculator...
User (to Assistant):

User (to Assistant):

***** Response from calling tool "call_ioceLhuKMpfU131E7TSQ8wCD" *****
44298
**********************************************************************

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
Assistant (to User):

***** Suggested tool Call (call_0rhx9vrbigcbqLssKLh4sS7j): calculator *****
Arguments: 
{
  "a": 44298,
  "b": 5,
  "operator": "*"
}
***************************************************************************

--------------------------------------------------------------------------------

>>>>>>>> EXECUTING FUNCTION calculator...
User (to Assistant):

User (to Assistant):

***** Response from calling tool "call_0rhx9vrbigcbqLssKLh4sS7j" *****
221490
**********************************************************************

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
Assistant (to User):

The result of the calculation (44232 + 13312 / (232 - 32)) * 5 is 221490. 

TERMINATE

--------------------------------------------------------------------------------
```

# 6. 对话模式（Conversation Patterns）

"Conversation Patterns" 是指定义代理间如何交互的模式。这些模式可以包括简单的一对一对话、顺序对话，以及更复杂的群组对话。群组对话允许多个代理共享同一对话线程并贡献信息，非常适合需要多代理协作的任务。管理群组对话的机制包括轮流选择发言代理，可以是随机的、轮流的或手动选择的。这种模式增强了代理间的协作与任务处理能力 ([Microsoft on GitHub](https://microsoft.github.io/autogen/docs/tutorial/conversation-patterns/))。

## 1. **Two-Agent Chat**

"Two-Agent Chat" 指的是两个代理之间的对话系统，这种设置通常包含一个助理代理和一个执行者代理。助理代理负责提出和优化代码或解决方案，而执行者代理则执行生成的代码或动作，并将结果反馈回来。这种模式使得可以将任务分解成独立的提议和执行部分，提高了处理的灵活性和效率。

Two-Agent Chat 是一种基本的对话模式，为复杂的多代理对话提供了基础。在这种模式下，两个代理协作解决问题，例如代码生成或问题回答，这通常遵循在 AutoGen 中定义的明确的对话和交互规则。

![Untitled](assets/introduction/Untitled%205.png)

设置并使用两个 **`ConversableAgent`**：一个模拟学生，另一个模拟数学老师。学生代理被设定为有学习意愿的角色，老师代理则被设定为数学教师。通过这种配置，学生代理能够向老师代理提出问题（例如询问三角不等式的定义），并通过对话获取信息。

```python
import os  # 导入os模块

from autogen import ConversableAgent  # 从autogen模块导入ConversableAgent类

# 创建一个代表学生的代理
student_agent = ConversableAgent(
    name="Student_Agent",  # 代理的名称
    system_message="You are a student willing to learn.",  # 系统消息，定义代理的角色和态度
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 配置使用GPT-4模型
)

# 创建一个代表老师的代理
teacher_agent = ConversableAgent(
    name="Teacher_Agent",  # 代理的名称
    system_message="You are a math teacher.",  # 系统消息，定义代理的角色
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 配置使用GPT-4模型
)

# 学生代理向老师代理发起对话，询问关于三角不等式的问题
chat_result = student_agent.initiate_chat(
    teacher_agent,
    message="What is triangle inequality?",  # 对话的初始消息
    summary_method="reflection_with_llm",  # 使用的总结方法
    max_turns=2,  # 对话的最大轮数限制
)

```

```
Student_Agent (to Teacher_Agent):

What is triangle inequality?

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
Teacher_Agent (to Student_Agent):

Triangle inequality theorem is a fundamental principle in geometry that states that the sum of the lengths of any two sides of a triangle must always be greater than the length of the third side. In a triangle with sides of lengths a, b, and c, the theorem can be written as:

a + b > c
a + c > b
b + c > a

Each of these represents the condition for one specific side (a, b, or c). All must be true for a triangle to exist.

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
Student_Agent (to Teacher_Agent):

Thank you for the explanation. This theorem helps in understanding the basic properties of a triangle. It can also be useful when solving geometric problems or proving other mathematical theorems. Can you give me an example of how we can use the triangle inequality theorem?

--------------------------------------------------------------------------------

>>>>>>>> USING AUTO REPLY...
Teacher_Agent (to Student_Agent):

Absolutely! Here's an example:

Suppose you're given three line segments with lengths 10, 7, and 3 units. The question is: "Can these three line segments form a triangle?"

To answer this, you would use the triangle inequality theorem. Adding any two side lengths together should be greater than the third:

- For sides 10 and 7: 10 + 7 = 17, which is larger than 3.
- For sides 10 and 3: 10 + 3 = 13, which is larger than 7.
- For sides 7 and 3: 7 + 3 = 10, which is equal to the length of the third side (10), but not greater.

So, these three lines cannot form a triangle, because not all pairs of sides satisfy the triangle inequality theorem.

--------------------------------------------------------------------------------
```

我们可以查看总结的内容是怎样的：

```python
print(chat_result.summary)
```

```
The triangle inequality theorem states that in a triangle, the sum of the lengths of any two sides must always be greater than the length of the third side. This principle is significant in geometry and is used in solving problems or proving theorems. For instance, if given three line segments, you can determine if they can form a triangle using this theorem.
```

在刚才的例子中，总结方法设置为 **`reflection_with_llm`** ，此时系统会根据对话内容生成一个摘要，首先尝试使用接收者的语言模型（LLM），如果不可用，则使用发送者的模型。在本例子中，接收者是“Teacher_Agent”而发送者是“Student_Agent”。通过执行 **`print(ConversableAgent.DEFAULT_summary_prompt)`**，可以查看在没有特定指定时，用于引导语言模型生成对话摘要的默认提示是什么。

```python
print(ConversableAgent.DEFAULT_summary_prompt)
```

```
Summarize the takeaway from the conversation. Do not add any introductory phrases.
```

我们可以查看**`chat_result`** 对象中保存的聊天历史：

```python
import pprint  # 导入pprint模块，用于美化打印输出

pprint.pprint(chat_result.chat_history)  # 美化打印chat_result对象中的聊天历史

```

```
[{'content': 'What is triangle inequality?', 'role': 'assistant'},
 {'content': 'Triangle inequality theorem is a fundamental principle in '
             'geometry that states that the sum of the lengths of any two '
             'sides of a triangle must always be greater than the length of '
             'the third side. In a triangle with sides of lengths a, b, and c, '
             'the theorem can be written as:\n'
             '\n'
             'a + b > c\n'
             'a + c > b\n'
             'b + c > a\n'
             '\n'
             'Each of these represents the condition for one specific side (a, '
             'b, or c). All must be true for a triangle to exist.',
  'role': 'user'},
 {'content': 'Thank you for the explanation. This theorem helps in '
             'understanding the basic properties of a triangle. It can also be '
             'useful when solving geometric problems or proving other '
             'mathematical theorems. Can you give me an example of how we can '
             'use the triangle inequality theorem?',
  'role': 'assistant'},
 {'content': "Absolutely! Here's an example:\n"
             '\n'
             "Suppose you're given three line segments with lengths 10, 7, and "
             '3 units. The question is: "Can these three line segments form a '
             'triangle?"\n'
             '\n'
             'To answer this, you would use the triangle inequality theorem. '
             'Adding any two side lengths together should be greater than the '
             'third:\n'
             '\n'
             '- For sides 10 and 7: 10 + 7 = 17, which is larger than 3.\n'
             '- For sides 10 and 3: 10 + 3 = 13, which is larger than 7.\n'
             '- For sides 7 and 3: 7 + 3 = 10, which is equal to the length of '
             'the third side (10), but not greater.\n'
             '\n'
             'So, these three lines cannot form a triangle, because not all '
             'pairs of sides satisfy the triangle inequality theorem.',
  'role': 'user'}]
```

打印 **`chat_result`** 对象中记录的聊天成本：

```python
import pprint  # 导入pprint模块，用于美化打印输出

pprint.pprint(chat_result.cost)  # 美化打印chat_result对象中的聊天成本

```

```
({'gpt-4-0613': {'completion_tokens': 399,
                 'cost': 0.04521,
                 'prompt_tokens': 709,
                 'total_tokens': 1108},
  'total_cost': 0.04521},
 {'total_cost': 0})
```

## 2. **Sequential Chat**

"Sequential Chats" 指的是一系列预定的对话，这些对话按顺序进行，每个对话处理不同的任务或问题。这种对话模式允许多个代理交替进行，每个代理在对话中承担特定的角色，完成特定的任务。这样的设置特别适合于需要多步骤处理或多个问题解决的场景。

"Sequential Chats" 可以用于解决复杂的问题，其中每个对话环节都是之前环节的延续，每个任务的输出可以作为下一个任务的输入。这种模式增加了处理任务的灵活性，允许更细致和深入的问题探讨，并使得任务处理更为有序和系统化。

更多关于 Sequential Chats 的信息：[AutoGen Documentation on Sequential Chats](https://microsoft.github.io/autogen/docs/tags/sequential-chats/)。

![Untitled](assets/introduction/Untitled%206.png)

我们创建了五个不同的 **`ConversableAgent`** 实例，每个代理都专注于一种简单的数学运算。这些代理分别是：返回给定数字的“Number_Agent”、将数字加1的“Adder_Agent”、将数字乘以2的“Multiplier_Agent”、将数字减1的“Subtracter_Agent”和将数字除以2的“Divider_Agent”。每个代理都配置了 GPT-4 模型以进行计算，但设置为不接受人类输入，完全自动化处理数学运算任务。这种设计可以在模拟或教学环境中用于展示基础数学运算的自动化处理。

```python
import os  # 导入os模块

from autogen import ConversableAgent  # 从autogen模块导入ConversableAgent类

# 定义一个总是返回相同数字的代理
number_agent = ConversableAgent(
    name="Number_Agent",
    system_message="You return me the numbers I give you, one number each line.",  # 系统消息，说明代理的功能
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 配置使用GPT-4模型
    human_input_mode="NEVER",  # 设置为不需要人类输入
)

# 定义一个增加器代理，对每个接收到的数字加1
adder_agent = ConversableAgent(
    name="Adder_Agent",
    system_message="You add 1 to each number I give you and return me the new numbers, one number each line.",  # 系统消息，说明代理的功能
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},
    human_input_mode="NEVER",
)

# 定义一个乘法器代理，对每个接收到的数字乘以2
multiplier_agent = ConversableAgent(
    name="Multiplier_Agent",
    system_message="You multiply each number I give you by 2 and return me the new numbers, one number each line.",  # 系统消息，说明代理的功能
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},
    human_input_mode="NEVER",
)

# 定义一个减法器代理，对每个接收到的数字减1
subtracter_agent = ConversableAgent(
    name="Subtracter_Agent",
    system_message="You subtract 1 from each number I give you and return me the new numbers, one number each line.",  # 系统消息，说明代理的功能
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},
    human_input_mode="NEVER",
)

# 定义一个除法器代理，对每个接收到的数字除以2
divider_agent = ConversableAgent(
    name="Divider_Agent",
    system_message="You divide each number I give you by 2 and return me the new numbers, one number each line.",  # 系统消息，说明代理的功能
    llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},
    human_input_mode="NEVER",
)

```

接下来使用 **`number_agent`** 代理发起一系列与其他计算代理的对话：

```python
# 启动一系列的两代理聊天
# 列表中的每个元素是一个字典，指定了initiate_chat方法的参数
chat_results = number_agent.initiate_chats(
    [
        {
            "recipient": adder_agent,  # 接收者为加法代理
            "message": "14",  # 发送的消息
            "max_turns": 2,  # 最大对话轮数
            "summary_method": "last_msg",  # 摘要方法
        },
        {
            "recipient": multiplier_agent,  # 接收者为乘法代理
            "message": "These are my numbers",  # 发送的消息
            "max_turns": 2,  # 最大对话轮数
            "summary_method": "last_msg",  # 摘要方法
        },
        {
            "recipient": subtracter_agent,  # 接收者为减法代理
            "message": "These are my numbers",  # 发送的消息
            "max_turns": 2,  # 最大对话轮数
            "summary_method": "last_msg",  # 摘要方法
        },
        {
            "recipient": divider_agent,  # 接收者为除法代理
            "message": "These are my numbers",  # 发送的消息
            "max_turns": 2,  # 最大对话轮数
            "summary_method": "last_msg",  # 摘要方法
        },
    ]
)

```

```

********************************************************************************
Start a new chat with the following message: 
14

With the following carryover: 

********************************************************************************
Number_Agent (to Adder_Agent):

14

--------------------------------------------------------------------------------
Adder_Agent (to Number_Agent):

15

--------------------------------------------------------------------------------
Number_Agent (to Adder_Agent):

15

--------------------------------------------------------------------------------
Adder_Agent (to Number_Agent):

16

--------------------------------------------------------------------------------

********************************************************************************
Start a new chat with the following message: 
These are my numbers

With the following carryover: 
16

********************************************************************************
Number_Agent (to Multiplier_Agent):

These are my numbers
Context: 
16

--------------------------------------------------------------------------------
Multiplier_Agent (to Number_Agent):

32

--------------------------------------------------------------------------------
Number_Agent (to Multiplier_Agent):

32

--------------------------------------------------------------------------------
Multiplier_Agent (to Number_Agent):

64

--------------------------------------------------------------------------------

********************************************************************************
Start a new chat with the following message: 
These are my numbers

With the following carryover: 
16
64

********************************************************************************
Number_Agent (to Subtracter_Agent):

These are my numbers
Context: 
16
64

--------------------------------------------------------------------------------
Subtracter_Agent (to Number_Agent):

15
63

--------------------------------------------------------------------------------
Number_Agent (to Subtracter_Agent):

15
63

--------------------------------------------------------------------------------
Subtracter_Agent (to Number_Agent):

14
62

--------------------------------------------------------------------------------

********************************************************************************
Start a new chat with the following message: 
These are my numbers

With the following carryover: 
16
64
14
62

********************************************************************************
Number_Agent (to Divider_Agent):

These are my numbers
Context: 
16
64
14
62

--------------------------------------------------------------------------------
Divider_Agent (to Number_Agent):

8
32
7
31

--------------------------------------------------------------------------------
Number_Agent (to Divider_Agent):

8
32
7
31

--------------------------------------------------------------------------------
Divider_Agent (to Number_Agent):

4
16
3.5
15.5

--------------------------------------------------------------------------------
```

打印各个阶段对话的结果：

```python
print("First Chat Summary: ", chat_results[0].summary)
print("Second Chat Summary: ", chat_results[1].summary)
print("Third Chat Summary: ", chat_results[2].summary)
print("Fourth Chat Summary: ", chat_results[3].summary)
```

```
First Chat Summary:  16
Second Chat Summary:  64
Third Chat Summary:  14
62
Fourth Chat Summary:  4
16
3.5
15.5
```

## 3. 群组对话（Group Chat）

"Group Chat" 指的是由多个代理参与的群组对话系统，这些代理可以是基于大型语言模型（LLM）、工具或人类的智能代理。群组对话允许这些代理通过自动化的聊天来集体执行任务，提高了任务处理的协同性和效率。

主要特点包括：

- **多代理协作**：群组对话使得多个代理可以在同一聊天环境中互动，共享信息和资源，共同解决问题。
- **灵活的对话管理**：群组聊天可以通过特定的管理设置来控制，例如限制或指定发言代理的顺序，以优化对话流程和效率。
- **任务分配**：在群组对话中，不同的代理可以承担不同的角色或任务，根据他们的专长和功能来分工协作。
- **集成工具和人类输入**：群组聊天支持将工具使用和人类参与融入对话中，这使得代理能够执行更复杂的任务，并且能够更灵活地适应各种情境。

![Untitled](assets/introduction/Untitled%207.png)

群组聊天由一个特殊的代理类型“群组聊天管理器”（GroupChatManager）来协调。在群组聊天的第一步，群组聊天管理器选择一个代理进行发言。被选中的代理发言后，消息会被发送回群组聊天管理器，然后由其广播给群组中的所有其他代理。这个过程会重复进行，直到对话结束。

群组聊天管理器可以使用几种策略来选择下一个发言的代理：

- **round_robin**：按照代理提供的顺序，轮流选择代理。
- **random**：随机选择代理。
- **manual**：通过请求人类输入来选择代理。
- **auto**：默认策略，使用群组聊天管理器的大型语言模型（LLM）来选择代理。

接下来我们举一个简单的例子，让各个算术运算代理在群组聊天中协作，目标是通过一系列算术操作将一个数字转换为特定的目标数字。

1. 先给几个不同的数学操作代理设置了 **`description`** 属性，描述了每个代理的主要功能。通过这些描述，可以快速理解每个代理的作用：
    - 加法代理（**`adder_agent`**）：对输入的每个数字加1。
    - 乘法代理（**`multiplier_agent`**）：对输入的每个数字乘以2。
    - 减法代理（**`subtracter_agent`**）：从输入的每个数字减去1。
    - 除法代理（**`divider_agent`**）：将输入的每个数字除以2。
    - 数字代理（**`number_agent`**）：简单地返回它接收到的数字。
    
    ```python
    # `description` 属性是一个描述代理功能的字符串
    # 它也可以在 `ConversableAgent` 构造函数中设置
    adder_agent.description = "Add 1 to each input number."  # 为加法代理设置描述，加1处理每个输入的数字
    multiplier_agent.description = "Multiply each input number by 2."  # 为乘法代理设置描述，将每个输入的数字乘以2
    subtracter_agent.description = "Subtract 1 from each input number."  # 为减法代理设置描述，从每个输入的数字减去1
    divider_agent.description = "Divide each input number by 2."  # 为除法代理设置描述，将每个输入的数字除以2
    number_agent.description = "Return the numbers given."  # 为数字代理设置描述，返回给定的数字
    
    ```
    
2. 创建叫 **`group_chat`** 的 **`GroupChat`** 实例，用于管理一个群组聊天。在这个群组聊天中，包含五个代理：加法代理、乘法代理、减法代理、除法代理和数字代理。这些代理将在群组聊天中互相协作或响应，进行数学运算。**`messages`** 属性被初始化为空列表，表示聊天开始时没有任何消息。**`max_round`** 设置为6，意味着聊天最多进行六轮，之后自动终止。
    
    ```python
    from autogen import GroupChat  # 从autogen模块导入GroupChat类
    
    # 创建一个群组聊天
    group_chat = GroupChat(
        agents=[adder_agent, multiplier_agent, subtracter_agent, divider_agent, number_agent],  # 将各种数学操作代理加入群组聊天
        messages=[],  # 初始消息列表为空
        max_round=6,  # 设置最大轮数为6轮
    )
    
    ```
    
3. 建一个叫 **`group_chat_manager`** 的 **`GroupChatManager`** 实例，它用于管理和协调前面定义的 **`group_chat`** 群组聊天。
    
    ```python
    from autogen import GroupChatManager  # 从autogen模块导入GroupChatManager类
    
    # 创建一个群组聊天管理器
    group_chat_manager = GroupChatManager(
        groupchat=group_chat,  # 关联到前面创建的群组聊天实例
        llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 配置使用GPT-4模型
    )
    
    ```
    
4. 使用 **`number_agent`**（数字代理）发起一次聊天，与 **`group_chat_manager`**（群组聊天管理器）交互。
    
    ```python
    # 使用数字代理发起与群组聊天管理器的聊天
    chat_result = number_agent.initiate_chat(
        group_chat_manager,  # 聊天的接收者是群组聊天管理器
        message="My number is 3, I want to turn it into 13.",  # 发起的消息，说明要将数字3转换为13
        summary_method="reflection_with_llm",  # 指定摘要方法为利用大型语言模型的反射
    )
    
    ```
    
    ```
    Number_Agent (to chat_manager):
    
    My number is 3, I want to turn it into 13.
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    6
    
    --------------------------------------------------------------------------------
    Adder_Agent (to chat_manager):
    
    7
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    14
    
    --------------------------------------------------------------------------------
    Subtracter_Agent (to chat_manager):
    
    13
    
    --------------------------------------------------------------------------------
    Number_Agent (to chat_manager):
    
    13
    
    --------------------------------------------------------------------------------
    ```
    
    ```python
    print(chat_result.summary)
    ```
    
    ```
    The agents cooperatively manipulated the initial number (3) through multipliying, adding, and subtracting operations to reach the target number (13).
    ```
    
    在群组聊天中，虽然我们通过设置代理的描述来帮助群组聊天管理器选择下一个代理，但这种方式仅对管理器有帮助，其他参与的代理并不了解彼此。有时让每个代理向群组中的其他代理自我介绍会更有用。这可以通过设置 **`send_introductions=True`** 实现。
    
    ```python
    group_chat_with_introductions = GroupChat(
        agents=[adder_agent, multiplier_agent, subtracter_agent, divider_agent, number_agent],
        messages=[],
        max_round=6,
        send_introductions=True,
    )
    ```
    
    在上面的示例中，我们创建了一个名为 **`group_chat_with_introductions`** 的 **`GroupChat`** 实例，并启用了代理间的自我介绍功能。
    
    在这背后，群组聊天管理器会在群组聊天开始之前向所有代理发送一条消息，这条消息包含了所有代理的名称和描述。这种介绍机制有助于建立更好的代理协作环境，因为每个代理在开始任务前都能了解彼此的功能和角色，从而提高整个群组聊天的效率和协同工作的效果。
    
    <aside>
    💡 **群组聊天可以作为序列聊天中的一部分**
    
    以下代码演示了如何使用群组聊天管理器作为两代理聊天序列中的一方代理进行对话。在这种设置中，数字代理 **`number_agent`** 向群组聊天管理器 **`group_chat_manager_with_intros`** 发起两次聊天请求。首先，数字代理请求将数字3转换成13，随后请求将一个数字转换成32。这里的群组聊天管理器作为一个普通代理参与到聊天中，并且管理来自不同代理的操作，以实现目标数字的转换。
    
    ```python
    from autogen import GroupChatManager  # 从autogen模块导入GroupChatManager类
    
    # 使用上面创建的带有介绍消息的群组聊天创建一个群组聊天管理器
    group_chat_manager_with_intros = GroupChatManager(
        groupchat=group_chat_with_introductions,  # 关联到带有自我介绍的群组聊天
        llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 配置使用GPT-4模型
    )
    
    # 在数字代理和群组聊天管理器之间启动一系列两代理聊天
    chat_result = number_agent.initiate_chats(
        [
            {
                "recipient": group_chat_manager_with_intros,  # 聊天的接收者是带有自我介绍的群组聊天管理器
                "message": "My number is 3, I want to turn it into 13.",  # 发送的第一条消息
            },
            {
                "recipient": group_chat_manager_with_intros,  # 聊天的接收者仍然是群组聊天管理器
                "message": "Turn this number to 32.",  # 发送的第二条消息
            },
        ]
    )
    
    ```
    
    ```
    
    ********************************************************************************
    Start a new chat with the following message: 
    My number is 3, I want to turn it into 13.
    
    With the following carryover: 
    
    ********************************************************************************
    Number_Agent (to chat_manager):
    
    My number is 3, I want to turn it into 13.
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    6
    
    --------------------------------------------------------------------------------
    Adder_Agent (to chat_manager):
    
    7
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    14
    
    --------------------------------------------------------------------------------
    Subtracter_Agent (to chat_manager):
    
    13
    
    --------------------------------------------------------------------------------
    Number_Agent (to chat_manager):
    
    Your number is 13.
    
    --------------------------------------------------------------------------------
    
    ********************************************************************************
    Start a new chat with the following message: 
    Turn this number to 32.
    
    With the following carryover: 
    Your number is 13.
    
    ********************************************************************************
    Number_Agent (to chat_manager):
    
    Turn this number to 32.
    Context: 
    Your number is 13.
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    26
    
    --------------------------------------------------------------------------------
    Adder_Agent (to chat_manager):
    
    14
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    28
    
    --------------------------------------------------------------------------------
    Adder_Agent (to chat_manager):
    
    15
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    30
    
    --------------------------------------------------------------------------------
    ```
    
    </aside>
    
    群组聊天是一种强大的对话模式，但当参与的代理数量较大时，控制对话可能会变得复杂。为了解决这一问题，AutoGen 提供了一种方法来限制下一个发言者的选择，即通过使用 **`GroupChat`** 类中的 **`allowed_or_disallowed_speaker_transitions`** 参数。
    
    **`allowed_or_disallowed_speaker_transitions`** 参数是一个字典，它将给定的代理映射到一个可以（或不可以）被选择为下一个发言者的代理列表。此外，**`speaker_transitions_type`** 参数用于指定这些转换是被允许的还是被禁止的。
    
    通过这种设置，可以更精确地控制代理之间的交流顺序，确保群组聊天的流畅和有序，特别是在涉及多个代理和复杂对话逻辑的场景中。
    
    比如：
    
    ```python
    # 定义允许的代理发言转换
    allowed_transitions = {
        number_agent: [adder_agent, number_agent],  # 数字代理后可由加法代理或数字代理接着发言
        adder_agent: [multiplier_agent, number_agent],  # 加法代理后可由乘法代理或数字代理接着发言
        subtracter_agent: [divider_agent, number_agent],  # 减法代理后可由除法代理或数字代理接着发言
        multiplier_agent: [subtracter_agent, number_agent],  # 乘法代理后可由减法代理或数字代理接着发言
        divider_agent: [adder_agent, number_agent],  # 除法代理后可由加法代理或数字代理接着发言
    }
    ```
    
    群组聊天中包括五个代理，通过定义的允许的代理发言转换规则，来控制代理间的对话顺序。每个代理在对话中只能按照定义好的顺序发言，这有助于维护聊天的有序性和目标的聚焦。
    
    ```python
    # 创建一个受限制的群组聊天
    constrained_graph_chat = GroupChat(
        agents=[adder_agent, multiplier_agent, subtracter_agent, divider_agent, number_agent],  # 参与的代理
        allowed_or_disallowed_speaker_transitions=allowed_transitions,  # 设置允许的发言转换
        speaker_transitions_type="allowed",  # 指定转换类型为“允许”
        messages=[],  # 聊天消息初始化为空
        max_round=12,  # 设置最大轮数为12轮
        send_introductions=True,  # 开启发送介绍信息
    )
    
    # 创建一个群组聊天管理器来管理受限的群组聊天
    constrained_group_chat_manager = GroupChatManager(
        groupchat=constrained_graph_chat,  # 关联受限群组聊天
        llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 配置使用GPT-4模型
    )
    
    # 数字代理发起对群组聊天管理器的聊天
    chat_result = number_agent.initiate_chat(
        constrained_group_chat_manager,
        message="My number is 3, I want to turn it into 10. Once I get to 10, keep it there.",  # 发起的消息
        summary_method="reflection_with_llm",  # 摘要方法
    )
    ```
    
    ```
    Number_Agent (to chat_manager):
    
    My number is 3, I want to turn it into 10. Once I get to 10, keep it there.
    
    --------------------------------------------------------------------------------
    Adder_Agent (to chat_manager):
    
    4
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    8
    
    --------------------------------------------------------------------------------
    Subtracter_Agent (to chat_manager):
    
    7
    
    --------------------------------------------------------------------------------
    Divider_Agent (to chat_manager):
    
    3.5
    
    --------------------------------------------------------------------------------
    Adder_Agent (to chat_manager):
    
    4.5
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    9
    
    --------------------------------------------------------------------------------
    Subtracter_Agent (to chat_manager):
    
    8
    
    --------------------------------------------------------------------------------
    Divider_Agent (to chat_manager):
    
    4
    
    --------------------------------------------------------------------------------
    Adder_Agent (to chat_manager):
    
    5
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    10
    
    --------------------------------------------------------------------------------
    Number_Agent (to chat_manager):
    
    10
    
    --------------------------------------------------------------------------------
    ```
    
    ## 4. 嵌套聊天（Nested Chat）
    
    "Nested Chats" 指的是在一个主对话中嵌套一个或多个子对话的结构。这允许代理在响应发起代理的消息之前，先与其他代理进行一系列的交流。嵌套聊天使得代理可以利用其他代理作为它们的内部独白，以增强决策过程或解决复杂问题。
    
    嵌套聊天由“嵌套聊天处理器”（nested chats handler）控制，这是一个可以插拔的组件，属于 **`ConversableAgent`**。当一个消息被接收时，嵌套聊天处理器会触发一系列的嵌套聊天。这些嵌套聊天会在接收代理回复发送代理的消息之前完成，从而允许在不暴露内部对话的情况下增强代理的响应。
    
    ![Untitled](assets/introduction/Untitled%208.png)
    
    我们创建三个不同功能的 **`ConversableAgent`**。
    
    **`arithmetic_agent`** 被设计为一个算术代理，总是需要人类输入来确保执行的代码安全。这种设置特别适合于需要执行敏感或危险操作的应用场景。**`code_writer_agent`** 是一个代码编写代理，专门用来编写Python脚本，输出格式为Markdown中的代码块，适用于需要自动生成代码文档或示例的情况。**`poetry_agent`**则被设置为一个AI诗人，能够创作诗歌，适用于娱乐或创意写作领域。
    
    ```python
    import tempfile  # 导入tempfile模块，用于处理临时文件目录
    
    temp_dir = tempfile.gettempdir()  # 获取系统的临时文件目录
    
    # 创建一个总是需要人类输入来确保代码安全执行的算术代理
    arithmetic_agent = ConversableAgent(
        name="Arithmetic_Agent",  # 代理的名称
        llm_config=False,  # 关闭LLM配置
        human_input_mode="ALWAYS",  # 总是需要人类输入
        code_execution_config={"use_docker": False, "work_dir": temp_dir},  # 设置代码执行配置，不使用Docker，工作目录为系统临时文件夹
    )
    
    # 创建一个代码编写代理，专门写Python脚本，并使用Markdown代码块格式
    code_writer_agent = ConversableAgent(
        name="Code_Writer_Agent",  # 代理的名称
        system_message="You are a code writer. You write Python script in Markdown code blocks.",  # 系统消息，定义代理角色和任务
        llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 使用GPT-4模型配置
        human_input_mode="NEVER",  # 不需要人类输入
    )
    
    # 创建一个诗歌代理，作为AI诗人
    poetry_agent = ConversableAgent(
        name="Poetry_Agent",  # 代理的名称
        system_message="You are an AI poet.",  # 系统消息，定义代理为AI诗人
        llm_config={"config_list": [{"model": "gpt-4", "api_key": os.environ["OPENAI_API_KEY"]}]},  # 使用GPT-4模型配置
        human_input_mode="NEVER",  # 不需要人类输入
    )
    
    ```
    
    接下来我们定义一个嵌套聊天列表 **`nested_chats`**，涉及三个不同的代理任务。第一个任务交给群组聊天管理器，请求总结通过一系列操作将一个数字转换为另一个数字的过程。第二个任务交给代码编写代理，要求编写一个Python脚本来验证这些算术操作是否正确。最后，诗歌代理被要求基于这些操作写一首诗，且对话仅限一轮。
    
    ```python
    nested_chats = [
        {
            "recipient": group_chat_manager_with_intros,  # 接收者为带有介绍的群组聊天管理器
            "summary_method": "reflection_with_llm",  
            "summary_prompt": "Summarize the sequence of operations used to turn the source number into target number.",  # 摘要提示，请求总结将源数字转换为目标数字的操作序列
        },
        {
            "recipient": code_writer_agent,  # 接收者为代码编写代理
            "message": "Write a Python script to verify the arithmetic operations is correct.",  # 消息内容，请求编写Python脚本来验证算术操作是否正确
            "summary_method": "reflection_with_llm",  
        },
        {
            "recipient": poetry_agent,  # 接收者为诗歌代理
            "message": "Write a poem about it.",  # 消息内容，请求编写一首关于此事的诗歌
            "max_turns": 1,  # 最大对话轮数限制为1
            "summary_method": "last_msg",  
        },
    ]
    
    ```
    
    下面我们为算术代理注册一系列嵌套聊天。
    
    **`register_nested_chats`** 方法允许算术代理根据特定条件触发嵌套聊天。这里定义的 **`trigger`** 函数是一个lambda表达式，其作用是检查发送者代理是否不在已定义的接收者列表中（包括群组聊天管理器、代码编写代理和诗歌代理）。这样设置的目的是防止算术代理在接收到来自这些特定代理的消息时触发嵌套聊天，从而避免可能的递归调用。
    
    ```python
    # 为算术代理注册嵌套聊天
    arithmetic_agent.register_nested_chats(
        nested_chats,  # 嵌套聊天的列表
        # 触发函数用于确定是否应该启动嵌套聊天，基于发送者代理
        # 在这个例子中，如果发送者是嵌套聊天的接收者之一，则算术代理不会启动嵌套聊天，以避免递归调用
        trigger=lambda sender: sender not in [group_chat_manager_with_intros, code_writer_agent, poetry_agent],
    )
    
    ```
    
    获得结果：
    
    ```python
    reply = arithmetic_agent.generate_reply(
        messages=[{"role": "user", "content": "I have a number 3 and I want to turn it into 7."}]
    )
    ```
    
    ```
    
    >>>>>>>> NO HUMAN INPUT RECEIVED.
    
    >>>>>>>> USING AUTO REPLY...
    
    ********************************************************************************
    Start a new chat with the following message: 
    I have a number 3 and I want to turn it into 7.
    
    With the following carryover: 
    
    ********************************************************************************
    Arithmetic_Agent (to chat_manager):
    
    I have a number 3 and I want to turn it into 7.
    
    --------------------------------------------------------------------------------
    Adder_Agent (to chat_manager):
    
    To give you the result, I'll add 1 to the number you gave me. So your new number is 4.
    
    --------------------------------------------------------------------------------
    Multiplier_Agent (to chat_manager):
    
    8
    
    --------------------------------------------------------------------------------
    Subtracter_Agent (to chat_manager):
    
    7
    
    --------------------------------------------------------------------------------
    Number_Agent (to chat_manager):
    
    7
    
    --------------------------------------------------------------------------------
    Number_Agent (to chat_manager):
    
    7
    
    --------------------------------------------------------------------------------
    
    ********************************************************************************
    Start a new chat with the following message: 
    Write a Python script to verify the arithmetic operations is correct.
    
    With the following carryover: 
    First, 1 was added to the initial number 3 to make it 4. Then it was multiplied by 2 which resulted in 8. Finally, 1 was subtracted from 8 to reach the target number 7.
    
    ********************************************************************************
    Arithmetic_Agent (to Code_Writer_Agent):
    
    Write a Python script to verify the arithmetic operations is correct.
    Context: 
    First, 1 was added to the initial number 3 to make it 4. Then it was multiplied by 2 which resulted in 8. Finally, 1 was subtracted from 8 to reach the target number 7.
    
    --------------------------------------------------------------------------------
    Code_Writer_Agent (to Arithmetic_Agent):
    
    Here is a Python script to verify the aforementioned arithmetic operations:
    
    ```python
    # defining the initial value
    initial_number = 3
    
    # Adding 1 to initial number
    initial_number += 1
    assert initial_number == 4, "The first operation failed!"
    
    # Multiplying the result by 2
    initial_number *= 2
    assert initial_number == 8, "The second operation failed!"
    
    # Subtracting 1 from the result
    initial_number -= 1
    assert initial_number == 7, "The final operation failed!"
    
    print("All operations were carried out successfully!")
    ```
    In the script, the entire process is broken down into steps. The `assert` function is used to verify the result at every step. If any of the operations doesn't yield the expected result, an `AssertionError` exception will be raised. If all operations pass, the message "All operations were carried out successfully!" will be printed.
    
    --------------------------------------------------------------------------------
    
    >>>>>>>> NO HUMAN INPUT RECEIVED.
    
    >>>>>>>> USING AUTO REPLY...
    
    >>>>>>>> EXECUTING CODE BLOCK 0 (inferred language is python)...
    Arithmetic_Agent (to Code_Writer_Agent):
    
    exitcode: 0 (execution succeeded)
    Code output: 
    All operations were carried out successfully!
    
    --------------------------------------------------------------------------------
    Code_Writer_Agent (to Arithmetic_Agent):
    
    Certainly, that means the python script was successful and every arithmetic operation performed correctly given the initial input and the steps performed.
    
    --------------------------------------------------------------------------------
    
    ********************************************************************************
    Start a new chat with the following message: 
    Write a poem about it.
    
    With the following carryover: 
    First, 1 was added to the initial number 3 to make it 4. Then it was multiplied by 2 which resulted in 8. Finally, 1 was subtracted from 8 to reach the target number 7.
    The Python script successfully performed and verified the arithmetic operations on the initial number provided. The steps included adding 1 to the initial number, multiplying the result by 2, and finally subtracting 1. The assert function was used to check the result at each step, and confirmed that all operations were carried out correctly.
    
    ********************************************************************************
    Arithmetic_Agent (to Poetry_Agent):
    
    Write a poem about it.
    Context: 
    First, 1 was added to the initial number 3 to make it 4. Then it was multiplied by 2 which resulted in 8. Finally, 1 was subtracted from 8 to reach the target number 7.
    The Python script successfully performed and verified the arithmetic operations on the initial number provided. The steps included adding 1 to the initial number, multiplying the result by 2, and finally subtracting 1. The assert function was used to check the result at each step, and confirmed that all operations were carried out correctly.
    
    --------------------------------------------------------------------------------
    Poetry_Agent (to Arithmetic_Agent):
    
    From numbers, logic, pure mathematical creation,
    Ponder this tale of numeric manipulation.
    In the universe of Python where operations exist,
    A story of integers and functions persist.
    
    Three was the number from where we began,
    Oblivious to the journey and its grandiosely plan.
    Added with 1, the sum it adorned,
    A sweet quadruple in the dawn was formed.
    
    The saga continued with a twist of the tale,
    The four was multiplied, while the winds wail.
    The duo of four unfolded its wings,
    An octet presence in our midst it brings.
    
    Then enters subtraction, sly and clever,
    Removing one to alter the endeavor.
    From eight, subtracted one in delight,
    To finally bask in the glow of seven's light.
    
    Each operation, together they conspired,
    In this tale of integers, creatively inspired.
    Through life's equation, the script ran so free,
    Amidst the language of Python, a symphony, you see.
    
    Tested with assert, cross-checked the chain,
    Confirming accuracy in program's domain.
    Each move calculated, each step so right,
    In the maze of coding, found was the light. 
    
    Such is the tale, of numbers and operations, 
    A dance among digits, logical iterations,
    Just another day, in this AI poet's life,
    Cutting through ambiguity, like a razor-sharp knife.
    
    --------------------------------------------------------------------------------
    ```