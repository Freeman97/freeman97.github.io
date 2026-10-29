---
title: 从零开始搭建最小可用的 Coding Agent - LangGraph
tags:
---

# 是什么
* LangChain 的底层实现
* 提供状态图的定义和装配，提供 LLM 对接能力，通过 LLM 的输出驱动状态图的流转。
    * LLM 对接能力也要封装？不就是接口调用吗？一个很明显的场景：LLM 对话的历史思考是要显式传递的（HTTP 调用LLM本身是无状态的），框架会帮忙封装这部分内容。


# 核心概念
## Graph
* LangGraph 构建的是LLM驱动流转的状态图
* 图的节点是完成一个独立流程的函数，例如 LLM 调用、工具调用
* 通过指定判断条件定义节点的出边，如：
```python
def should_continue(state: MessagesState) -> Literal["tool_node", END]:
    """Decide if we should continue the loop or stop based upon whether the LLM made a tool call"""

    messages = state["messages"]
    last_message = messages[-1]

    # If the LLM makes a tool call, then perform an action
    if last_message.tool_calls:
        return "tool_node"

    # Otherwise, we stop (reply to the user)
    return END

# Build workflow
agent_builder = StateGraph(MessagesState)

# Add nodes
agent_builder.add_node("llm_call", llm_call)
agent_builder.add_node("tool_node", tool_node)

# Add edges to connect nodes
agent_builder.add_edge(START, "llm_call")
agent_builder.add_conditional_edges(
    "llm_call",
    should_continue,
    ["tool_node", END]
)
agent_builder.add_edge("tool_node", "llm_call")

# Compile the agent
agent = agent_builder.compile()
```

## State
* 多个不同节点中都可以访问的共享数据。

## 节点的错误处理
* 需要用户交互：使用 Interrupt 暂停后让用户补充更多信息，用户完成补充后回到相同节点
```python
from langgraph.types import Command


def lookup_customer_history(
    state: State
) -> Command[Literal["lookup_customer_history", "draft_response"]]:
    if not state.get('customer_id'):
        user_input = interrupt({
            "message": "Customer ID needed",
            "request": "Please provide the customer's account ID to look up their subscription history"
        })
        return Command(
            update={"customer_id": user_input['customer_id']},
            goto="lookup_customer_history"
        )
    # Now proceed with the lookup
    customer_data = fetch_customer_history(state['customer_id'])
    return Command(update={"customer_history": customer_data}, goto="draft_response")
```