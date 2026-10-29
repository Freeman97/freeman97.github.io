---
title: 从零开始搭建最小可用的 Coding Agent - LangChain
tags:
---

# 是什么
* LLM 只会做猜下一个字游戏。LLM + Harness 才能构成对外提供服务的 Agent
* LangChain 可以认为是一个最小程度、可配置的 harness

# 核心概念
## Agent
* 一个基于循环的模型调用工具。Agent 会不停调用 LLM，直至判断问题解决或达到最大循环次数。

### Agent State
* 当前的执行上下文信息。
* 执行上下文至少会包括所有的历史对话信息（`List[BaseMessage]`），Append Only。

### Agent 的调用
* 调用方（可能是 CLI、GUI 或其他 headless 模式）构造结构化消息 Message 发送给 Agent（调用 `agent.invoke`），实现对 Agent 的调用。
* 调用时配置`config`:
    * 调用方通过在 `config` 参数里面指定 `thread_id` 来判断多个消息是否在同一对话上下文中。
* 其他对话上下文信息或其他调用配置（per-run configuration）通过 `context` 传递，例如用户ID、模型 API Keys、可观测性工具等。
* 为什么 Agent 使用基于消息的通信机制？

### Agent 的流式调用（Streaming）
* 最朴素的调用模式：调用 Agent 后直接等待其返回最终结果。
* 常见场景：一次 Agent 调用过程中调用多个外部工具，部分工具的执行中需要用户提供额外信息或做处理（例如审批高危操作、提供鉴权凭据等）。
* 引入 Streaming 机制，调用方和 Agent 之间可以进行双向通信
```python
from langchain.messages import AIMessage, HumanMessage

# 返回消息迭代器
stream = agent.stream_events(
    {"messages": [{"role": "user", "content": "Search for AI news and summarize the findings"}]},
    version="v3",
)

# 一条 snapshot 可以认为是消息列表里 (AgentState) 产生的一条消息, 具体原理需要看 agent.stream_events 的实现方式。
for snapshot in stream.values:
    # Each snapshot contains the full state at that point
    latest_message = snapshot["messages"][-1]
    # 问题: 流式通信怎么样算结束？
    if latest_message.content:
        if isinstance(latest_message, HumanMessage):
            print(f"User: {latest_message.content}")
        elif isinstance(latest_message, AIMessage):
            print(f"Agent: {latest_message.content}")
    elif latest_message.tool_calls:
        print(f"Calling tools: {[tc['name'] for tc in latest_message.tool_calls]}")
```

## 对 harness 的配置
### Middleware
* Middleware 是 harness 配置的原语，可视为挂载在 Agent Loop 中的钩子方法。
    * 挂载点（`Node-Style`）：在 Agent Loop 的某个特定节点执行，例如 `before_agent` - 每次 Agent 调用前就调用一次 Middleware，`after_model` - 每次 LLM 响应之后就调用一次 Middleware
    * Wrap （`Wrap-Style`）：在每次调用行为前后（around）调用，例如每次 LLM 调用（`wrap_model_call`）或每次工具调用（`wrap_tool_call`）
    * 两类middleware都可以影响 Agent State: node style 直接返回 dict，内部包含消息或一些控制原语（`jump_to`）；wrap style 返回 command 直接对 graph state 进行控制。
    * 多个 middleware 的返回值会被 langchain 编排和合并（reducer）
#### Middleware 发生作用的典型场景：
* 执行环境（Execution Environment）：这也是 LangChain 推荐的工具注入最佳实践
    * 执行环境包括：工具列表、文件系统、可以跑代码的运行环境、隔离沙箱。
    * 例如 `FileSystemMiddleware`，提供了对文件系统工具的高层抽象：
        * Agent 视角来看，`FileSystemMiddleware` 是文件系统类 Tool 的直接提供者 -> 在 langchain `create_agent` 方法中会将 middleware 的tool字段声明的工具列表添加到本次 request 的工具列表中
        * `FileSystemMiddleware` 在声明时通过参数动态组合出不同的文件系统实现，支持不同的行为，例如内存文件操作、隔离沙箱中的文件系统操作或命令执行、实际工作区（非隔离环境）的文件系统操作。
        * `FileSystemMiddleware` 提供文件系统操作的统一拦截逻辑；起到了类似策略模式的作用（统一逻辑由middleware实现，有差异的部分根据构造时的参数委派给实际的 backend）。例如：
            * 文件系统权限检查：读、写，对应允许、拒绝、中断（要求人工介入）；检查命令是否能执行、范围限制、拦截高危操作（删除、读写敏感目录）。
            * 系统提示词注入：
            * 大结果 evit：部分工具的返回结果可能很大；中间件负责基于阈值判断是否要 evit，即不直接将响应内容写入工具返回，而是写入文件系统。

* 上下文（context）管理：
    * Context Engineering：制定策略管理模型在对话过程中能读到的信息使其输出更可靠。典型的上下文包括用户输入、历史对话记录、长期记忆（例如 system prompt，持久化的 agents.md 文件，tool prompt，skills）。  
    * 典型的利用 Middleware 管理上下文的实践：上下文压缩（历史对话压缩）、记忆管理、Skill 加载
    * `SummarizationMiddleware`：达到 token 窗口的时候压缩上下文（`@before_model`）。
        * 怎么压缩？提供一个配置 N 或比例 M%，Agent State 的最近 N 条消息或 最近的 M% 消息保留，剩余的消息作为待压缩的消息。使用一个额外配置的 LLM (可以与 Agent 的主 LLM 不同) 生成一段摘要，将摘要放回消息列表的头部。
    * `MemoryMiddleware`: 读取并加载记忆
        * 记忆的来源可以是多个持久化的文件，例如 `AGENTS.md`。
        * LangChain 原生的 `MemoryMiddleware` 作用比较有限：从文件中提取出记忆后简单格式化追加到 System Prompt。
        * 问题：生产 Agent 的记忆系统怎么构建？
        * 问题2：记忆规模大了怎么筛选，怎么评估召回效果？
    * Skill 支持：LangChain 的官方示例是将 Skill 加载逻辑放进了 Middleware，将 Skill 列表作为工具列表装载到 Skill 加载 Middleware 中。然后将 Skill 的描述装载到 System Prompt。
        * 同样属于比较简陋的 Skill 系统，至少需要解决以下问题：
            * 我装了 114514 个 Skill, 总不能全塞进 System Prompt 里面吧，会有什么过滤机制吗？
            * 我发起对话后才引入的 Skill，怎么在下一轮对话就让模型感知到新 Skill？

* Plan And Execute（Subagent架构）
    * `LangChain` 内部封装的 SubAgent 实现工具是一个 middleware：`SubAgent Middleware`
    * 为什么这么设计？大概是因为一种流行的做法是将 主Agent 与 SubAgent 之间的交互定义为工具调用（对SubAgent的调用封装为一种工具），然后采用了 LangChain 定义的 工具系统最佳实践。

* 容错设计
    * 在工具调用、模型调用通过中间件机制注入异常重试逻辑。

* 护栏机制
    * 是什么？：对 LLM 的输入和输出进行过滤。
    * 输入：例如防范可能的提示词注入攻击，或过滤用户的敏感信息防止被传输到服务端。
    * 输出：幻觉防范、道德审查、版权审查


* 方向盘机制：高位操作前请求人类二次确认，`HumanInTheLoopMiddleware`。
    * 基本上是纯规则驱动的逻辑。如果要有复杂逻辑，例如引入 LLM 参与判断，则需要更强的手段，例如在定义图的时候就显式定义某个节点间流转必须人工。


## Model
* 目前除了原生的文本生成能力，LangChain 的模型通过消息类型定义了模型的以下能力：
    * Tool Calling - 调用工具：
        * 一个最简单的 Agent 循环：将用户消息和候选工具列表投递给 Model，Model 通过结构化的消息（`ToolMessage`）输出所有待调用的工具。Tool calling 处理完所有的工具调用后将输出的结果再送回给 Model，输出最终结果。
        ```python
            # Bind (potentially multiple) tools to the model
            tools = [get_weather]
            tools_by_name = {tool.name: tool for tool in tools}
            model_with_tools = model.bind_tools(tools)

            # Step 1: Model generates tool calls
            messages = [{"role": "user", "content": "What's the weather in Boston?"}]
            ai_msg = model_with_tools.invoke(messages)
            messages.append(ai_msg)

            # Step 2: Execute tools and collect results
            for tool_call in ai_msg.tool_calls:
                # Execute the tool with the generated arguments
                tool = tools_by_name[tool_call["name"]]
                tool_result = tool.invoke(tool_call)
                messages.append(tool_result)

            # Step 3: Pass results back to model for final response
            final_response = model_with_tools.invoke(messages)
            print(final_response.text)
            # "The current weather in Boston is 72°F and sunny."
        ```
    * Structured Output
        * Pydantic: 格式强校验 - 模型输出类型和格式不对可以直接抛出异常；提供字段级 `description` 可以作为 prompt 的一部分引导 LLM；支持嵌套模型、枚举约束等能力。
        * TypedDict: Python 标准库自带，但没有原生的运行时校验
        * JSON Schema: 跨语言
    * Multimodality - 多模态
        * `"type": "image"`
    * Reasoning - 推理
        * `"type": "reasoning"`

* Prompt Caching(?)




## Message
* LangChain 中 context 的最小单元。封装了用户发出的信息、模型调用的结果、Agent 内部的控制流信息。LangChain 通过消息列表表征当前 Agent 的状态。
    * `Role`: 消息类型，如 system, user
    * `Content`: 消息的实际内容，可能有文本、图片、文档等
    * `Metadata`: 消息的元信息，为消息补充额外的描述字段

* 消息被划分为如下的类型：
    * System message - Tells the model how to behave and provide context for interactions
    * Human message - Represents user input and interactions with the model
    * AI message - Responses generated by the model, including text content, tool calls, and metadata
    * Tool message - Represents the outputs of tool calls


## Tool
* 工具能访问什么：
    * 设计者定义的任意参数
    * 运行时信息：（`ToolRuntime`）
        * State
        * Context：单次调用不可变的信息，例如用户标识、对话信息...
        * Store：长期记忆，不一定是文件，可能是继承 `BaseStore` 的任意 K-V 存储。例如内存、Redis、Mongo...
        * Stream Writer: ？
        * Execution Info: 可观测性字段
        * Server Info: ？

    * 通过 `Command` 直接更新 Agent State（产生一个消息、编辑自定义字段）。对于自定义字段，最好自定义 Reducer 处理多条并发产生的消息。

* 工具能返回什么：
    * String
        * 会给模型读到，但不会直接影响 Agent State, 除非 LLM 决定
    * object
        * 会被序列化后作为模型输出
        * 也不会直接影响 State
    * Command
        * 需要直接影响 Graph State: 增加一条消息、直接更新其他自定义字段
    * 将工具结果直接返回，而不将工具结果送回model（`@tool(return_direct=True)`），此时 Agent 会立即停止循环，将工具结果直接返回给调用者。
        * 如果有多个并行的工具调用，只有每个 tool 都是直接返回型，才会立即停止循环。（只有一部分有的话怎么办？）

* 动态工具选择（工具的动态过滤和动态注册）
    * 为什么设计这个机制：工具候选项太多的时候会为模型注入太多上下文导致注意力涣散，难以选出正确的工具调用
    * 实现方式1：运行时将预先注册的工具排除掉。定义 middleware 将request中的工具列表 override 成过滤后的工具列表。适用于在每次 invoke 开始的时候能列出一个比较具体的工具范围。
    * 实现方式2：运行时（agent 被invoke后）注册工具。
        * 工具列表可能运行时发现（例如 MCP Server）。
        * 运行时动态生成的工具（“写脚本帮用户批量处理某些事”属于这个范畴吗？）
 
 * Headless tool
    * 是什么：注册进工具列表的仅仅是一个客户端，负责发起工具调用。
    * 典型场景：浏览器 API、位置信息、设备信息、GUI操作。（为什么不包含网络操作？）
    * 怎么实现的：？- 前面的世界以后再来探索吧（https://docs.langchain.com/oss/python/langchain/frontend/headless-tools）

## 短期记忆（short-term memory）
* 特指一次会话（包含多次 agent 调用）中的记忆，让部分信息在多次 Invoke 之间传递。
* 用法：创建agent 的时候 定义 `checkpointer`。`checkpointer` 可以是纯内存存储（`InMemorySaver`），也可以是数据库存储。
* 问题：如果一次对话要创建新 agent 怎么办。。
* 问题2：存的到底是什么？整个 State（包含历史消息和用户自定义字段）吗？
* 问题3：进行上下文压缩的时候，会讲压缩后的上下文刷写进 `checkpointer` 定义的存储中吗？如果会，怎么做？

### 记忆的访问
* 读取：`ToolRuntime` 中 `state` 字段。直接用 K-V 方式访问。
* 写入：返回 `Command`
* 读取的一种场景：Dynamic Prompt，即通过记忆动态拼装或改写 system prompt

## Event Streaming
TBD

## 
