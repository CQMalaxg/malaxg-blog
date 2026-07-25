+++
date = '2026-07-25T21:30:00+08:00'
draft = false
title = 'Deep Agents 学习文档'
tags = ['AI Agent', 'LangChain', 'Deep Agents']
categories = ['学习资料']
+++

更新日期：2026-07-25  
适用对象：已经了解 Python、LLM API、基础 Agent/工具调用，想系统学习 LangChain `deepagents` 框架的人。

> 说明：`deepagents` 仍在快速演进。本文按当前官方文档整理，写项目时应以官方文档和 API reference 为准。

## 1. 一句话理解

`deepagents` 是 LangChain 生态里的一个 Agent harness：它不是替代 LangChain/LangGraph 的底层框架，而是在 LangChain agent 与 LangGraph runtime 之上，把复杂任务常用能力预装好。

它默认帮你处理这些事：

- 规划：内置 `write_todos`，让 agent 把复杂任务拆成步骤。
- 文件系统：内置 `ls`、`read_file`、`write_file`、`edit_file`、`glob`、`grep`，让 agent 把大上下文写到文件里再检索。
- 子代理：内置 `task` 工具，可把复杂子任务交给隔离上下文的 subagent。
- 上下文工程：大输入/大输出可 offload 到文件系统，长对话可自动总结。
- 持久记忆：用文件形式保存长期 memory。
- Skills：把专门工作流、领域知识、模板、脚本按需加载。
- Human-in-the-loop：对危险工具调用暂停并等待人类批准。
- Streaming：基于 LangGraph streaming 查看主 agent 和 subagent 的执行过程。

更简单地说：LangChain 给你积木，LangGraph 给你运行时，Deep Agents 给你一套“复杂任务 agent 默认应该有的脚手架”。

## 2. 什么时候该用 Deep Agents

适合：

- 研究报告、竞品分析、资料搜集、写作辅助这类多步骤任务。
- 编码助手、测试修复、文档维护这类需要读写文件和运行命令的任务。
- 需要把任务分给多个角色的系统，比如 researcher、critic、data-analyzer。
- 需要长期记住用户偏好、项目约定、历史经验的 agent。
- 需要审计、审批、权限控制的生产 agent。

不一定适合：

- 单轮聊天。
- 一个模型加一两个工具的简单问答。
- 你需要完全自定义状态机、严格控制每个节点和边的流程。这时直接用 LangGraph 往往更清楚。

选择原则：

- 简单工具调用：优先 `langchain.create_agent`。
- 强流程编排：优先 LangGraph。
- 长任务、多文件、多子任务、需要 memory/skills/HITL：优先 Deep Agents。

## 3. 安装与环境

最小安装：

```bash
pip install -U deepagents
```

如果使用 OpenAI：

```bash
pip install -U deepagents langchain-openai
```

如果使用 Anthropic：

```bash
pip install -U deepagents langchain-anthropic
```

如果使用 Google Gemini：

```bash
pip install -U deepagents langchain-google-genai
```

Windows PowerShell 设置环境变量示例：

```powershell
$env:OPENAI_API_KEY="你的 API key"
$env:DEEPAGENTS_MODEL="openai:gpt-5"
```

macOS/Linux 示例：

```bash
export OPENAI_API_KEY="你的 API key"
export DEEPAGENTS_MODEL="openai:gpt-5"
```

注意：

- 模型必须支持 tool calling。
- `model` 可以传 `"provider:model-name"` 字符串，也可以传已经初始化好的 LangChain chat model。
- 官方默认模型和推荐模型会更新，项目里建议通过环境变量配置模型名。

## 4. 最小可运行 Agent

创建 `hello_deepagents.py`：

```python
import os
from deepagents import create_deep_agent


def get_weather(city: str) -> str:
    """Get mock weather for a city."""
    return f"It is sunny in {city}."


agent = create_deep_agent(
    model=os.getenv("DEEPAGENTS_MODEL", "openai:gpt-5"),
    tools=[get_weather],
    system_prompt="You are a helpful assistant. Use tools when they are useful.",
)

result = agent.invoke(
    {
        "messages": [
            {"role": "user", "content": "What is the weather in Chongqing?"}
        ]
    },
    config={"configurable": {"thread_id": "demo-hello"}},
)

print(result["messages"][-1].content)
```

运行：

```bash
python hello_deepagents.py
```

你要观察的不是天气结果，而是这几个点：

- 函数 `get_weather` 被当作工具暴露给模型。
- 函数参数类型和 docstring 会影响模型是否正确调用工具。
- `create_deep_agent(...)` 返回的是一个可 invoke/stream 的 LangGraph compiled graph。
- `thread_id` 用来把同一个会话的状态串起来。

## 5. 核心 API：`create_deep_agent`

常用参数：

- `model`：模型字符串或模型实例。
- `tools`：你提供给 agent 的业务工具。
- `system_prompt`：你的业务角色和任务要求。它会和 Deep Agents 的内置提示一起工作。
- `middleware`：扩展或拦截 agent 行为。
- `subagents`：定义自定义子代理。
- `backend`：文件系统后端，比如内存、磁盘、Store、sandbox。
- `skills`：skills 目录路径。
- `memory`：启动时加载到系统提示里的 memory 文件路径。
- `response_format`：结构化输出 schema。
- `interrupt_on`：为指定工具启用 human-in-the-loop。
- `checkpointer`：让线程状态可持久化；HITL 场景必需。

Deep Agents 默认会挂载多类 middleware，包括 todo、filesystem、subagent、summarization 等。你通常先用默认值起步，只有遇到明确需求时再扩展 middleware。

## 6. System Prompt 怎么写

Deep Agents 已经有一套内置系统提示，教模型如何规划、读写文件、调用子代理。你的 `system_prompt` 应该补充“业务身份”和“质量标准”，不要重复解释框架工具。

推荐结构：

```text
You are a research assistant for technical writing.

Goals:
- Produce accurate, sourced, concise reports.
- Separate facts, assumptions, and recommendations.

Workflow:
- First clarify the task if necessary.
- Use search tools for current or external facts.
- Write large notes to files before synthesizing.
- Use subagents for independent research branches.

Output:
- Start with the conclusion.
- Include source links.
- Mention uncertainty clearly.
```

常见错误：

- 把 prompt 写成很长的操作手册，挤占上下文。
- 没有说明输出标准，导致结果散。
- 没有说明何时使用工具，模型会乱查或完全不查。
- 把用户每次变化的信息写进静态 `system_prompt`；动态偏好更适合 memory 或 runtime context。

## 7. 工具设计

Deep Agents 可以接收普通 callable、LangChain `@tool`、BaseTool 或 dict 工具。

一个好工具应该：

- 名字明确，避免泛化成 `do_task`。
- 参数有类型标注。
- docstring 写清楚何时使用、输入约束、返回内容。
- 返回结构化、紧凑、可被模型消费的数据。
- 对错误给出可读信息，不要只抛异常。

示例：

```python
from typing import Literal


def search_articles(
    query: str,
    max_results: int = 5,
    topic: Literal["general", "news", "technical"] = "general",
) -> dict:
    """Search article references for a query.

    Use this when the user asks for external or current information.
    Returns a dict with title, url, snippet, and published_at when available.
    """
    return {
        "query": query,
        "results": [
            {
                "title": "Example result",
                "url": "https://example.com",
                "snippet": "Short summary...",
                "published_at": None,
            }
        ],
    }
```

工具设计的关键不是“能不能调用”，而是“模型能不能判断什么时候调用、怎么传参、如何解释返回”。

## 8. 内置文件系统能力

Deep Agents 的一个核心思路是：不要把所有东西塞进上下文窗口，把中间资料、搜索结果、草稿、结构化产物写进文件系统。

常见内置文件工具：

- `ls`：列目录。
- `read_file`：读文件。
- `write_file`：写新文件或覆盖文件。
- `edit_file`：局部编辑文件。
- `glob`：按 pattern 找文件。
- `grep`：全文搜索。

使用文件系统的价值：

- 大工具返回结果可以先落盘，再摘要。
- 多轮任务可以复用中间产物。
- subagent 可以把结果写成文件，主 agent 再读取。
- memory 和 skills 都可以建立在文件抽象之上。

实践建议：

- 要求 agent 把大型研究资料写到 `/notes/` 或 `/workspace/`。
- 要求最终产物写成 `/outputs/report.md`。
- 对代码项目，只让 agent 操作工作区内路径。
- 对生产服务，避免直接暴露真实磁盘。

## 9. Backends：文件系统后端

Deep Agents 的文件工具不是只能操作真实磁盘。它们通过 backend 抽象读写不同存储。

### 9.1 `StateBackend`

默认后端。文件存储在 LangGraph state 中，适合临时草稿和单线程会话。

```python
from deepagents import create_deep_agent
from deepagents.backends import StateBackend

agent = create_deep_agent(
    model="openai:gpt-5",
    backend=StateBackend(),
)
```

适合：

- demo。
- 临时 scratchpad。
- 不希望 agent 碰真实文件系统的场景。

### 9.2 `FilesystemBackend`

读写本地磁盘。适合本地开发和受控 CI，不建议直接放到 Web API 生产服务里。

```python
from deepagents import create_deep_agent
from deepagents.backends import FilesystemBackend

agent = create_deep_agent(
    model="openai:gpt-5",
    backend=FilesystemBackend(root_dir=".", virtual_mode=True),
)
```

重点：

- `root_dir` 限定根目录。
- `virtual_mode=True` 才能更好地把路径限制在 root 下。
- 本地磁盘读写有安全风险，尤其是 `.env`、密钥、私有代码。

### 9.3 `StoreBackend`

把文件存进 LangGraph store，适合跨 thread 的长期数据。

```python
from deepagents import create_deep_agent
from deepagents.backends import StoreBackend
from langgraph.store.memory import InMemoryStore

store = InMemoryStore()

agent = create_deep_agent(
    model="openai:gpt-5",
    backend=StoreBackend(namespace=lambda ctx: ("demo-user",)),
    store=store,
)
```

生产里要认真设计 namespace：

- 用户级：`(user_id,)`
- 租户级：`(tenant_id,)`
- agent 级：`(assistant_id,)`

不要让不同用户共享同一个 memory namespace，除非这就是你的产品设计。

### 9.4 `CompositeBackend`

把不同路径路由到不同后端。

```python
from deepagents import create_deep_agent
from deepagents.backends import CompositeBackend, StateBackend, StoreBackend
from langgraph.store.memory import InMemoryStore

store = InMemoryStore()

agent = create_deep_agent(
    model="openai:gpt-5",
    backend=CompositeBackend(
        default=StateBackend(),
        routes={
            "/memories/": StoreBackend(namespace=lambda ctx: ("demo-user",)),
        },
    ),
    store=store,
)
```

常见组合：

- `/workspace/`：临时 StateBackend。
- `/memories/`：持久 StoreBackend。
- `/docs/`：只读自定义 backend。
- `/outputs/`：本地磁盘或对象存储。

## 10. Permissions：路径权限

如果 agent 能读写文件，必须考虑权限。

基础示例：只允许访问 `/workspace/`，其他全部拒绝。

```python
from deepagents import FilesystemPermission, create_deep_agent

agent = create_deep_agent(
    model="openai:gpt-5",
    backend=backend,
    permissions=[
        FilesystemPermission(
            operations=["read", "write"],
            paths=["/workspace/**"],
            mode="allow",
        ),
        FilesystemPermission(
            operations=["read", "write"],
            paths=["/**"],
            mode="deny",
        ),
    ],
)
```

要点：

- `operations=["read"]` 覆盖 `ls`、`read_file`、`glob`、`grep`。
- `operations=["write"]` 覆盖 `write_file`、`edit_file`。
- 规则按顺序匹配，first match wins。
- 如果没有任何规则匹配，默认允许。
- permissions 只管内置文件系统工具，不管自定义工具、MCP 工具，也不管 sandbox 里的任意 shell 命令。

生产建议：

- 先 deny，再精确 allow，或者显式 allow 工作区后 deny 全部。
- secrets、配置、凭据目录默认不可读。
- 写操作配合 HITL。
- 对自定义工具单独做权限校验。

## 11. Subagents：子代理

Subagent 的核心价值是上下文隔离。主 agent 把一个子任务交出去，subagent 自己搜索、读写、推理，最后只把摘要结果交还主 agent。

基础示例：

```python
from deepagents import create_deep_agent


def web_search(query: str) -> dict:
    """Search the web for a query."""
    return {"query": query, "results": []}


research_subagent = {
    "name": "researcher",
    "description": "Use for deep research on a specific technical topic.",
    "system_prompt": "You are a careful technical researcher. Cite sources.",
    "tools": [web_search],
    "model": "openai:gpt-5",
}

agent = create_deep_agent(
    model="openai:gpt-5",
    subagents=[research_subagent],
    system_prompt="You coordinate research and produce final answers.",
)
```

何时使用 subagent：

- 并行研究多个方向。
- 让 critic/reviewer 独立检查结果。
- 数据分析、代码审查、事实核查等角色明显不同。
- 某个子任务会产生大量中间上下文，不想污染主对话。

不要滥用：

- 每个 subagent 都会增加成本、延迟和调试复杂度。
- 子任务边界不清时，subagent 输出会空泛。
- 主 agent 仍要负责整合、判断和验收。

内置 `general-purpose` subagent：

- 每个 deep agent 默认都有一个 general-purpose subagent。
- 它默认继承主 agent 的模型、工具和 system prompt。
- 配置了 skills 时，general-purpose subagent 会继承主 agent skills。
- 自定义 subagent 默认不继承主 agent skills，需要单独传 `skills`。

## 12. Structured Output

如果你希望 agent 返回可解析 JSON，而不是自由文本，可以用 `response_format`。

```python
from pydantic import BaseModel, Field
from deepagents import create_deep_agent


class ResearchReport(BaseModel):
    title: str = Field(description="Report title")
    summary: str = Field(description="Short executive summary")
    key_points: list[str] = Field(description="Important findings")
    sources: list[str] = Field(description="Source URLs")


agent = create_deep_agent(
    model="openai:gpt-5",
    tools=[web_search],
    system_prompt="Produce accurate research reports.",
    response_format=ResearchReport,
)

result = agent.invoke(
    {"messages": [{"role": "user", "content": "Research Deep Agents."}]}
)

print(result["structured_response"])
```

适合：

- API 返回给前端。
- 后续程序要消费 agent 输出。
- 评测和自动化验收。

## 13. Memory：长期记忆

Deep Agents 的 memory 是文件形式的长期上下文。创建 agent 时传入 memory 文件路径，agent 启动会读取这些文件并注入系统提示；会话过程中也可以通过文件编辑更新 memory。

用户级 memory 示例：

```python
from deepagents import create_deep_agent
from deepagents.backends import CompositeBackend, StateBackend, StoreBackend

agent = create_deep_agent(
    model="openai:gpt-5",
    memory=["/memories/preferences.md"],
    backend=CompositeBackend(
        default=StateBackend(),
        routes={
            "/memories/": StoreBackend(
                namespace=lambda ctx: (ctx.runtime.context.user_id,),
            ),
        },
    ),
)
```

设计 memory 时先分三类：

- 用户偏好：语言、格式、禁忌、长期目标。
- 项目知识：架构、约定、目录、测试方式。
- 经验总结：过去成功/失败的工作方法。

建议：

- memory 文件保持短而可维护。
- 让 agent 写 memory 前尽量经过确认。
- 不要把短期任务细节都写进长期 memory。
- 不要保存敏感信息，除非你已经设计好加密、隔离和删除机制。

## 14. Skills：按需加载的能力包

Skill 是一个目录，里面必须有 `SKILL.md`，可以附带脚本、模板、参考资料、资产文件。

典型目录：

```text
skills/
  langgraph-docs/
    SKILL.md
    references/
      api-notes.md
  code-review/
    SKILL.md
    review_checklist.md
```

`SKILL.md` 示例：

```markdown
---
name: code-review
description: Use this skill when reviewing code changes for bugs, regressions, security issues, and missing tests.
---

# Code Review

## Instructions

1. Inspect the diff and surrounding code.
2. Prioritize correctness, security, and user-visible regressions.
3. Report findings with file and line references.
4. Keep summaries brief.

## References

- Read `review_checklist.md` when the review touches authentication, billing, or permissions.
```

在 SDK 中使用：

```python
from deepagents import create_deep_agent
from deepagents.backends import FilesystemBackend

agent = create_deep_agent(
    model="openai:gpt-5",
    backend=FilesystemBackend(root_dir=".", virtual_mode=True),
    skills=["/skills/"],
)
```

Skill 的心智模型：

- tools 是“低层能力”：搜索、读文件、发请求、查数据库。
- skills 是“工作方法”：怎么研究、怎么审查、怎么写报告、怎么用某套内部规范。
- memory 是“长期背景”：这个用户/项目一直相关的事实。

Progressive disclosure：

- 启动时 agent 只读 skill 的 frontmatter。
- 当任务匹配 skill description 时，才读取完整 `SKILL.md`。
- `SKILL.md` 再指向额外参考资料或脚本。

写 skill 的关键是 description。模型主要靠 description 判断是否使用这个 skill。

## 15. Human-in-the-loop

对删除、发邮件、写数据库、执行 shell、修改文件这类敏感动作，应加人工审批。

```python
from langchain.tools import tool
from langgraph.checkpoint.memory import MemorySaver
from deepagents import create_deep_agent


@tool
def send_email(to: str, subject: str, body: str) -> str:
    """Send an email."""
    return f"Sent email to {to}"


checkpointer = MemorySaver()

agent = create_deep_agent(
    model="openai:gpt-5",
    tools=[send_email],
    interrupt_on={
        "send_email": {"allowed_decisions": ["approve", "reject"]},
    },
    checkpointer=checkpointer,
)
```

常见 decision：

- `approve`：按原参数执行。
- `edit`：修改参数后执行。
- `reject`：拒绝执行，并可给 agent 反馈。

要点：

- HITL 需要 checkpointer，否则暂停后无法可靠恢复。
- 审批策略应按工具风险分级。
- 读操作不一定要审批，写操作、外部副作用、不可逆操作优先审批。

## 16. Streaming 与调试

Streaming 让你实时看到 agent 进展、工具调用、subagent 执行。

```python
for chunk in agent.stream(
    {"messages": [{"role": "user", "content": "Research LangGraph and summarize."}]},
    stream_mode="updates",
    subgraphs=True,
    version="v2",
):
    print(chunk["type"], chunk["ns"], chunk["data"])
```

你可以观察：

- 主 agent 当前在哪一步。
- 是否调用了 `write_todos`。
- 是否把资料写入文件。
- 是否创建 subagent。
- subagent 是否在并行执行。
- 工具参数是否合理。

调试建议：

- 打开 LangSmith tracing。
- 先用假工具或小数据跑通流程。
- 对工具调用记录输入输出。
- 对最终输出做自动评测，比如是否包含 source、是否是合法 JSON、是否覆盖所有问题。

## 17. Context Engineering

Deep Agents 的上下文工程默认围绕四件事：

- System prompt：角色、目标、规则、输出标准。
- Tool prompts：工具描述和工具使用说明。
- Filesystem offloading：把大内容写入文件，再按需读取。
- Summarization：上下文接近限制时压缩旧消息。

实践模式：

1. 让 agent 先写 todo。
2. 搜索或读取资料后写入 `/notes/`。
3. 中途产物写入 `/drafts/`。
4. 最终产物写入 `/outputs/`。
5. 对重要结论要求附证据来源。

一个高质量任务提示：

```text
Research Deep Agents and write a technical learning note.

Requirements:
- Create a short plan first.
- Save raw notes to /notes/deepagents.md.
- Compare Deep Agents with LangChain create_agent and LangGraph.
- Produce final output as /outputs/deepagents-learning-note.md.
- Include links to official docs.
```

这个提示比“介绍一下 Deep Agents”更容易激活框架的优势。

## 18. 推荐学习路线

### 第 1 天：跑通最小 agent

目标：

- 安装 `deepagents`。
- 配置一个支持 tool calling 的模型。
- 跑通 `create_deep_agent`。
- 理解 `invoke` 输入输出格式。

练习：

- 写一个 mock weather tool。
- 问 agent 三个问题：不需要工具、需要工具、需要多步推理。

### 第 2 天：掌握工具设计

目标：

- 学会给工具写类型和 docstring。
- 观察模型如何选择工具。
- 处理工具错误和空结果。

练习：

- 写 `search_articles`、`read_document_summary` 两个工具。
- 故意传模糊问题，看 agent 是否会问清楚或调用搜索。

### 第 3 天：文件系统与 backend

目标：

- 理解 StateBackend、FilesystemBackend、StoreBackend。
- 学会让 agent 把中间结果写文件。
- 学会权限限制。

练习：

- 要求 agent 把研究笔记写到 `/notes/`。
- 设置只允许 `/workspace/**` 读写。
- 测试 agent 访问被禁止路径时的行为。

### 第 4 天：subagents

目标：

- 理解上下文隔离。
- 定义 researcher、critic 两个子代理。
- 观察主 agent 如何合并子代理结果。

练习：

- researcher 查资料。
- critic 检查事实和遗漏。
- 主 agent 输出最终报告。

### 第 5 天：skills

目标：

- 写一个自己的 `SKILL.md`。
- 理解 progressive disclosure。
- 区分 tools、skills、memory。

练习：

- 写 `technical-blog-writing` skill。
- 让 agent 按你的博客风格生成文章大纲。

### 第 6 天：memory

目标：

- 配置用户级 memory。
- 设计 memory 文件结构。
- 避免污染长期记忆。

练习：

- 写 `/memories/preferences.md`。
- 让 agent 记住你的输出偏好。
- 换一个 `thread_id` 验证是否能读取 memory。

### 第 7 天：HITL、streaming、评测

目标：

- 对敏感工具加审批。
- 用 streaming 观察执行。
- 建立简单验收脚本或人工 checklist。

练习：

- 给 `write_file`、`send_email` 或真实副作用工具加 interrupt。
- 记录一次完整 trace。
- 对输出做结构化校验。

## 19. 小项目：博客选题研究 Agent

这个项目适合放在你的博客工作流里练习。

目标：

用户输入一个方向，比如“agent context engineering”，agent 自动完成：

1. 拆解研究计划。
2. 搜索官方资料和高质量文章。
3. 把原始笔记写到 `/notes/topic.md`。
4. 让 critic subagent 检查遗漏、过时信息和不可靠来源。
5. 输出博客大纲、核心论点、参考链接。
6. 可选：把最终 Markdown 写到 `/outputs/draft.md`。

建议角色：

- main agent：项目经理和最终写作者。
- researcher subagent：搜资料、摘录、整理事实。
- critic subagent：挑错、检查引用、标出不确定性。
- style skill：你的博客风格、标题习惯、段落节奏。

建议文件结构：

```text
agent_workspace/
  skills/
    blog-style/
      SKILL.md
  notes/
  drafts/
  outputs/
  memories/
    preferences.md
```

验收标准：

- 是否有明确选题角度。
- 是否引用官方或一手来源。
- 是否区分事实和观点。
- 是否有可写成文章的结构。
- 是否避免把搜索片段直接拼贴成正文。

## 20. 常见坑

- 模型不支持工具调用：agent 看起来会“思考”，但不会可靠调用工具。
- 工具 docstring 太弱：模型不知道何时调用，也不知道参数怎么填。
- 返回大段原始内容：上下文迅速膨胀，应写文件再摘要。
- 滥用 subagent：任务变慢、成本变高、结果更难调试。
- `FilesystemBackend` 直接暴露真实项目目录：容易误读 secrets 或改坏文件。
- 忘记 `virtual_mode=True`：`root_dir` 本身不等于完整安全边界。
- permissions 以为能管住所有东西：它只管内置文件工具，不管自定义工具和 sandbox shell。
- HITL 没有 checkpointer：暂停和恢复会出问题。
- memory 写太多：长期记忆变成垃圾堆，反而降低质量。
- skills description 不清楚：agent 根本不会触发 skill。

## 21. 生产化 checklist

上线前至少确认：

- 模型：支持 tool calling，延迟和成本可接受。
- 工具：参数 schema 清晰，错误可恢复，有超时。
- 权限：文件读写路径受限，secrets 不可读。
- 副作用：写文件、发请求、发邮件、执行命令等有 HITL 或策略保护。
- 状态：thread_id、checkpointer、store namespace 设计清楚。
- Memory：用户隔离，敏感信息策略明确。
- Skills：版本化，description 清楚，引用文件完整。
- Observability：LangSmith tracing 或等价日志可用。
- Evaluation：有样例任务、预期输出、回归检查。
- 成本控制：subagent 数量、搜索次数、工具返回大小有限制。

## 22. 官方资料入口

- Deep Agents overview: https://docs.langchain.com/oss/python/deepagents/overview
- Quickstart: https://docs.langchain.com/oss/python/deepagents/quickstart
- Customization: https://docs.langchain.com/oss/python/deepagents/customization
- Context engineering: https://docs.langchain.com/oss/python/deepagents/context-engineering
- Backends: https://docs.langchain.com/oss/python/deepagents/backends
- Permissions: https://docs.langchain.com/oss/python/deepagents/permissions
- Subagents: https://docs.langchain.com/oss/python/deepagents/subagents
- Memory: https://docs.langchain.com/oss/python/deepagents/memory
- Skills: https://docs.langchain.com/oss/python/deepagents/skills
- Streaming: https://docs.langchain.com/oss/python/deepagents/streaming
- Human-in-the-loop: https://docs.langchain.com/oss/python/deepagents/human-in-the-loop
- Deep Agents GitHub: https://github.com/langchain-ai/deepagents

## 23. 最短复习版

记住这张表：

| 你要解决的问题 | Deep Agents 里的东西 |
| --- | --- |
| 复杂任务怎么拆 | `write_todos` |
| 大上下文怎么放 | filesystem tools + offloading |
| 多角色怎么做 | `subagents` / `task` |
| 中间产物放哪 | backend |
| 长期偏好怎么记 | `memory` |
| 专门工作流怎么注入 | `skills` |
| 危险操作怎么拦 | `interrupt_on` + checkpointer |
| 执行过程怎么看 | `stream(...)` + LangSmith |
| 输出怎么给程序消费 | `response_format` |

学习顺序：

1. 先跑通最小 agent。
2. 再写两个好工具。
3. 接着学文件系统和 backend。
4. 然后加 subagent。
5. 最后补 memory、skills、HITL、streaming、评测。

Deep Agents 的真正难点不是 API，而是产品和工程判断：哪些上下文该进 prompt，哪些该进文件，哪些该变成 skill，哪些该变成 memory，哪些操作必须让人批准。
