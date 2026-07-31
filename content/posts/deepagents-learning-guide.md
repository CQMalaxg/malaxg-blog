+++
date = '2026-07-25T21:30:00+08:00'
draft = false
title = 'Deep Agents 学习文档'
tags = ['AI Agent', 'LangChain', 'Deep Agents']
categories = ['学习资料']
+++

更新日期：2026-07-31  
适用对象：已经了解 Python、LLM API、基础 Agent/工具调用，想系统学习 LangChain `deepagents` 框架的人。

> 说明：`deepagents` 仍在快速演进（部分能力需要 `deepagents>=0.7`）。本文按当前官方文档与 GitHub 仓库整理，写项目时应以官方文档和 API reference 为准。本次更新补充了 **中间件机制、shell/沙箱、解释器、MCP 集成、上下文压缩内部机制、内置工具完整清单、生态与多语言版本**，并新增了三张可读的 SVG 图解。

<div class="deepagents-hero" id="deepagents-start">
  <div class="deepagents-hero__copy">
    <span class="deepagents-kicker">DEEP AGENTS / FIELD GUIDE</span>
    <p class="deepagents-hero__title">把复杂任务交给一个会规划、会协作、会记笔记的 Agent。</p>
    <p class="deepagents-hero__lede">这不是 API 目录，而是一条从“跑通第一个 agent”到“设计生产系统”的学习路径。先看 SVG 图，再读代码，最后用一个博客选题研究 Agent 把知识串起来。</p>
    <div class="deepagents-hero__actions">
      <a href="#deepagents-roadmap">查看学习路线 <span aria-hidden="true">&rarr;</span></a>
      <a class="deepagents-hero__action--quiet" href="#deepagents-architecture">先看架构图</a>
    </div>
  </div>
  <div class="deepagents-hero__signal" aria-label="Deep Agents 核心执行闭环">
    <span class="deepagents-kicker">THE CORE LOOP</span>
    <svg class="deepagents-hero__loop" viewBox="0 0 200 200" aria-hidden="true">
      <defs>
        <marker id="loopArrow" markerWidth="8" markerHeight="8" refX="4" refY="4" orient="auto"><path d="M0 0 L8 4 L0 8 z" fill="currentColor"/></marker>
      </defs>
      <circle cx="100" cy="100" r="72" fill="none" stroke="currentColor" stroke-width="1.4" stroke-dasharray="4 6" opacity="0.5"/>
      <path d="M100 34 A66 66 0 0 1 157 133" fill="none" stroke="currentColor" stroke-width="2.4" marker-end="url(#loopArrow)"/>
      <path d="M150 145 A66 66 0 0 1 43 145" fill="none" stroke="currentColor" stroke-width="2.4" marker-end="url(#loopArrow)"/>
      <path d="M43 133 A66 66 0 0 1 100 34" fill="none" stroke="currentColor" stroke-width="2.4" opacity="0.85"/>
      <g class="deepagents-hero__loop-dot"><circle cx="100" cy="28" r="7"/><circle cx="158" cy="128" r="7"/><circle cx="100" cy="168" r="7"/><circle cx="42" cy="128" r="7"/></g>
      <text x="100" y="16" text-anchor="middle">Plan</text>
      <text x="182" y="132" text-anchor="middle">Work</text>
      <text x="100" y="190" text-anchor="middle">Delegate</text>
      <text x="20" y="132" text-anchor="middle">Review</text>
    </svg>
    <strong>Plan &rarr; Work &rarr; Delegate &rarr; Review</strong>
    <div class="deepagents-hero__steps">
      <span><b>01</b>规划</span>
      <span><b>02</b>执行</span>
      <span><b>03</b>委派</span>
      <span><b>04</b>验收</span>
    </div>
  </div>
</div>

<div class="deepagents-roadmap" id="deepagents-roadmap">
  <div class="deepagents-roadmap__head">
    <div>
      <span class="deepagents-kicker">LEARNING MAP / 07 DAYS</span>
      <p class="deepagents-roadmap__title">从一个工具调用，走到一套可控的 Agent 系统</p>
    </div>
    <span class="deepagents-roadmap__status">建议顺序：上到下</span>
  </div>
  <ol class="deepagents-roadmap__list">
    <li><span>01</span><strong>基础</strong><small>安装、模型、最小 agent</small></li>
    <li><span>02</span><strong>工具</strong><small>让模型知道何时调用</small></li>
    <li><span>03</span><strong>上下文</strong><small>文件、backend、权限</small></li>
    <li><span>04</span><strong>协作</strong><small>subagents 与 task</small></li>
    <li><span>05</span><strong>长期化</strong><small>skills、memory、HITL</small></li>
    <li><span>06</span><strong>进阶</strong><small>middleware、sandbox、压缩</small></li>
    <li><span>07</span><strong>评测</strong><small>streaming、tracing、验收</small></li>
  </ol>
</div>

<div class="deepagents-visual" id="deepagents-architecture" aria-label="Deep Agents 分层架构图">
  <div class="deepagents-visual__head">
    <span class="deepagents-kicker">FIGURE 01 / ARCHITECTURE</span>
    <span class="deepagents-visual__hint">先记住：Deep Agents 是上层 harness</span>
  </div>
  <div class="deepagents-stack">
    <div class="deepagents-stack__layer deepagents-stack__layer--product"><span>你的业务层</span><strong>Tools &middot; Skills &middot; Memory</strong><small>把领域知识、权限和业务动作接入 agent</small></div>
    <div class="deepagents-stack__connector" aria-hidden="true">&darr;</div>
    <div class="deepagents-stack__layer deepagents-stack__layer--deep"><span>Deep Agents</span><strong>Planning &middot; Filesystem &middot; Subagents</strong><small>复杂任务默认需要的工作台和护栏</small></div>
    <div class="deepagents-stack__connector" aria-hidden="true">&darr;</div>
    <div class="deepagents-stack__layer deepagents-stack__layer--runtime"><span>LangChain + LangGraph</span><strong>Agent loop &middot; State &middot; Runtime</strong><small>模型、工具调用、状态和执行图</small></div>
    <div class="deepagents-stack__connector" aria-hidden="true">&darr;</div>
    <div class="deepagents-stack__layer deepagents-stack__layer--model"><span>模型与基础设施</span><strong>LLM &middot; Checkpointer &middot; Store</strong><small>推理能力、会话恢复和持久化</small></div>
  </div>
  <p class="deepagents-visual__caption">把它理解成“积木 + 运行时 + 工程脚手架”：你仍然拥有 LangChain/LangGraph 的控制力，但不用每次从零拼出复杂 agent 的基础能力。</p>
</div>

<div class="deepagents-visual deepagents-visual--flow" aria-label="Deep Agents 任务执行流程图">
  <div class="deepagents-visual__head">
    <span class="deepagents-kicker">FIGURE 02 / EXECUTION LOOP</span>
    <span class="deepagents-visual__hint">大任务的中间产物要离开对话窗口</span>
  </div>
  <div class="deepagents-flow">
    <div class="deepagents-flow__node deepagents-flow__node--input"><span>01</span><strong>目标</strong><small>用户提出一个复杂问题</small></div>
    <span class="deepagents-flow__arrow" aria-hidden="true">&rarr;</span>
    <div class="deepagents-flow__node deepagents-flow__node--plan"><span>02</span><strong>计划</strong><small><code>write_todos</code> 拆解步骤</small></div>
    <span class="deepagents-flow__arrow" aria-hidden="true">&rarr;</span>
    <div class="deepagents-flow__node deepagents-flow__node--work"><span>03</span><strong>工作区</strong><small>笔记与结果写入文件</small></div>
    <span class="deepagents-flow__arrow" aria-hidden="true">&rarr;</span>
    <div class="deepagents-flow__node deepagents-flow__node--delegate"><span>04</span><strong>委派</strong><small><code>task</code> 调用 subagent</small></div>
    <span class="deepagents-flow__arrow" aria-hidden="true">&rarr;</span>
    <div class="deepagents-flow__node deepagents-flow__node--review"><span>05</span><strong>交付</strong><small>主 agent 汇总并验收</small></div>
  </div>
</div>

<div class="deepagents-capability-map" aria-label="Deep Agents 能力地图">
  <div class="deepagents-capability-map__intro">
    <span class="deepagents-kicker">CAPABILITY MAP</span>
    <p>学习时可以把每个概念放回这八个问题里。</p>
  </div>
  <div class="deepagents-capability-map__grid">
    <div><span>PLAN</span><strong>怎么拆任务？</strong><small><code>write_todos</code></small></div>
    <div><span>CONTEXT</span><strong>信息放在哪里？</strong><small>filesystem / backend</small></div>
    <div><span>DELEGATE</span><strong>谁来做子任务？</strong><small>subagents / <code>task</code></small></div>
    <div><span>EXECUTE</span><strong>怎么跑命令？</strong><small>sandbox / <code>execute</code></small></div>
    <div><span>EXTEND</span><strong>怎么接外部系统？</strong><small>tools / MCP</small></div>
    <div><span>REUSE</span><strong>什么值得长期记住？</strong><small>skills / memory</small></div>
    <div><span>CONTROL</span><strong>危险动作谁批准？</strong><small>permissions / HITL</small></div>
    <div><span>ASSEMBLE</span><strong>默认行为怎么改？</strong><small>middleware</small></div>
  </div>
</div>

<div class="deepagents-metrics" aria-label="Deep Agents 关键默认参数速览">
  <div class="deepagents-metrics__head">
    <span class="deepagents-kicker">KEY DEFAULTS / 记住这几个阈值</span>
    <p>这些数字直接决定 agent 在长任务里的行为，调优前先记住默认值。</p>
  </div>
  <div class="deepagents-metrics__grid">
    <div class="deepagents-metric">
      <svg viewBox="0 0 24 24" aria-hidden="true" width="26" height="26"><path d="M4 4h16v4H4zM4 10h16v4H4zM4 16h10v4H4z" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linejoin="round"/></svg>
      <strong>20K</strong>
      <span>token</span>
      <small>单次工具输入/输出超过约 2 万 token 就卸载到文件。</small>
    </div>
    <div class="deepagents-metric">
      <svg viewBox="0 0 36 36" aria-hidden="true" width="26" height="26"><circle cx="18" cy="18" r="15" fill="none" stroke="currentColor" stroke-width="3" opacity="0.25"/><circle cx="18" cy="18" r="15" fill="none" stroke="currentColor" stroke-width="3" stroke-dasharray="80 94" stroke-linecap="round" transform="rotate(-90 18 18)"/></svg>
      <strong>85%</strong>
      <span>上下文</span>
      <small>越过模型窗口约 85% 且无可卸载内容时触发摘要。</small>
    </div>
    <div class="deepagents-metric">
      <svg viewBox="0 0 24 24" aria-hidden="true" width="26" height="26"><path d="M4 6h16M4 12h10M4 18h13" fill="none" stroke="currentColor" stroke-width="1.7" stroke-linecap="round"/></svg>
      <strong>10</strong>
      <span>行预览</span>
      <small>卸载后对话里只留路径指针 + 文件前 10 行预览。</small>
    </div>
    <div class="deepagents-metric">
      <svg viewBox="0 0 24 24" aria-hidden="true" width="26" height="26"><path d="M12 3l3 3-3 3-3-3zM12 15l3 3-3 3-3-3zM3 12l3-3 3 3-3 3zM15 12l3-3 3 3-3 3z" fill="none" stroke="currentColor" stroke-width="1.6" stroke-linejoin="round"/></svg>
      <strong>0.7+</strong>
      <span>版本</span>
      <small>替换中间件、<code>delete</code>、文件工具白名单等需要此版本。</small>
    </div>
  </div>
</div>

## SVG 图解：把 Agent 过程变成可检查的图

SVG 不只是网页上的一张配图，它是一种可以被阅读、修改和版本控制的文本格式。对 Deep Agents 来说，SVG 最适合表达三件事：**谁在做事、信息流向哪里、哪个动作需要人确认**。

<div class="deepagents-svg-lesson" id="deepagents-svg">
  <div class="deepagents-svg-lesson__intro">
    <span class="deepagents-kicker">SVG STUDY / READ THE SYSTEM</span>
    <p class="deepagents-svg-lesson__title">先读拓扑，再读 API，最后才写代码。</p>
    <p>下面这张图把一个“研究并交付报告”的任务拆成主 agent、文件工作区、subagents、HITL 和最终输出。你可以沿着箭头复述整个执行过程，而不是只记住一串参数名。</p>
  </div>
  <figure class="deepagents-svg-figure">
    <img src="/deepagents-learning/deepagents-execution-map.svg" alt="Deep Agents 执行拓扑图：用户目标经过主 Agent、文件工作区、子代理和人工批准后生成交付物" loading="lazy">
    <figcaption>图 03：一个可控的 Deep Agents 任务，把对话变成有边界、有中间产物、有验收点的工作流。</figcaption>
  </figure>
</div>

### 为什么用 SVG，而不是截图

- **可解释**：节点、连线、标签都能对应到代码和运行时事件。
- **可维护**：SVG 是纯文本，可以和文章、代码一起进入 Git diff。
- **可复用**：同一张图可以嵌入博客、课程笔记、README 或演示页面。
- **可访问**：`title`、`desc`、`text` 和 `alt` 可以帮助读屏和搜索理解图的语义。

<figure class="deepagents-svg-figure deepagents-svg-figure--anatomy">
  <img src="/deepagents-learning/svg-anatomy.svg" alt="SVG 三个基本元素：rect 表示节点，path 表示关系，text 表示解释" loading="lazy">
  <figcaption>图 04：一张学习图通常可以先拆成 `rect`、`path`、`text` 三类 SVG 基元。</figcaption>
</figure>

| SVG 部件 | 图上表达什么 | Deep Agents 对应什么 |
| --- | --- | --- |
| `<rect>` | 一个稳定的对象或边界 | 主 agent、文件区、subagent、审批节点 |
| `<path>` | 有方向的关系或数据流 | `write_todos`、`task`、工具调用、结果回传 |
| `<text>` | 让读者知道节点的语义 | 工具名、路径、角色、交付物类型 |
| `marker` | 箭头的方向 | 谁调用谁、谁等待谁、结果回到哪里 |

### 从一行 SVG 回到一个 Agent 概念

```xml
<path class="edge" d="M 270 236 H 355" />
<rect class="process" x="355" y="160" width="330" height="168" rx="10" />
<text x="381" y="230">主 Agent</text>
```

把它翻译成人话：**一条 path 表示“目标进入主 agent”的关系，rect 表示主 agent 这个执行节点，text 给节点补上可读的解释**。如果你把 `path` 画反了，图就会误导读者；如果没有 `text`，图就只有形状，没有知识。

<div class="deepagents-svg-practice">
  <span class="deepagents-kicker">SVG PRACTICE / 15 MIN</span>
  <strong>把一个真实任务画成 5 个节点</strong>
  <ol>
    <li>写下用户目标，标成 input。</li>
    <li>画出主 agent，并在旁边写出它会调用的工具。</li>
    <li>把大段研究资料放进 `/notes/`，把最终结果放进 `/outputs/`。</li>
    <li>用一个 subagent 节点表示独立研究或审查。</li>
    <li>给写入、发送、删除等副作用动作加一个 HITL 节点。</li>
  </ol>
</div>

## 1. 一句话理解

`deepagents` 是 LangChain 生态里的一个 Agent harness：它不是替代 LangChain/LangGraph 的底层框架，而是在 LangChain agent 与 LangGraph runtime 之上，把复杂任务常用能力预装好。

它默认帮你处理这些事：

- 规划：内置 `write_todos`，让 agent 把复杂任务拆成步骤。
- 文件系统：内置 `ls`、`read_file`、`write_file`、`edit_file`、`glob`、`grep`、`delete`，让 agent 把大上下文写到文件里再检索。
- 子代理：内置 `task` 工具，可把复杂子任务交给隔离上下文的 subagent，也支持异步子代理。
- 上下文工程：大输入/大输出可 offload 到文件系统，长对话可自动总结（summarization）。
- 持久记忆：用文件形式保存长期 memory。
- Skills：把专门工作流、领域知识、模板、脚本按需加载。
- Shell 访问：配合 sandbox 后端提供 `execute`，在隔离环境里跑命令；也可用 QuickJS 解释器跑受限 JS。
- 工具生态：接入任意自定义函数或任意 MCP server。
- Human-in-the-loop：对危险工具调用暂停并等待人类批准。
- Streaming：基于 LangGraph streaming 查看主 agent 和 subagent 的执行过程。

它由一整套 LangChain **中间件（middleware）** 组装而成，这是理解 Deep Agents 的关键：几乎每个内置能力都对应栈里的一个中间件，你可以替换、扩展或裁剪它们（见第 11 节）。

更简单地说：LangChain 给你积木，LangGraph 给你运行时，Deep Agents 给你一套“复杂任务 agent 默认应该有的脚手架”。它的设计取向是 opinionated（有主张）、extensible（可扩展不用 fork）、model-agnostic（任意支持 tool calling 的模型）、production-ready（基于 LangGraph 的流式、持久化、checkpoint，配合 LangSmith 追踪与评测）。

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

- `ls`：列目录，带大小和修改时间等元数据。
- `read_file`：读文件，带行号，支持 `offset`/`limit` 处理大文件；对图片、视频、音频、PDF/PPT 等可返回多模态内容块。
- `write_file`：写新文件或覆盖文件。
- `edit_file`：局部精确字符串替换，支持全局替换。
- `delete`：删除文件，或递归删除目录（需要 `deepagents>=0.7`；不支持删除的后端会自动隐藏它）。
- `glob`：按 pattern 找文件，比如 `**/*.py`。
- `grep`：全文搜索，支持“仅文件名 / 带上下文 / 计数”等输出模式。
- `execute`：在环境里运行 shell 命令，**仅 sandbox 后端可用**（见第 10 节）。

`read_file` 支持的多模态扩展名大致包括：

- 图片：`.png` `.jpg` `.jpeg` `.gif` `.webp` `.heic` `.heif`
- 视频：`.mp4` `.mov` `.webm` `.avi` 等
- 音频：`.wav` `.mp3` `.flac` `.aac` `.ogg` 等
- 文档：`.pdf` `.ppt` `.pptx`

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

<figure class="deepagents-svg-figure">
  <img src="/deepagents-learning/backends-map.svg" alt="Deep Agents 后端选型地图：同一套文件工具通过 CompositeBackend 路由到 StateBackend、FilesystemBackend、StoreBackend 或 Sandbox 后端" loading="lazy">
  <figcaption>图 05：一套文件工具，四种存储位置。先按“数据活多久、给谁看、能不能执行命令”选后端，再用 CompositeBackend 按路径组合。</figcaption>
</figure>


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

## 11. Middleware：Deep Agents 的装配层

Deep Agents 并不是一个黑盒。`create_deep_agent` 实际上是把一整套 LangChain **中间件**按固定顺序组装成一条流水线。理解这条流水线，你就能精确地替换、扩展或裁剪它的默认行为。

<figure class="deepagents-svg-figure">
  <img src="/deepagents-learning/middleware-stack.svg" alt="Deep Agents 默认中间件栈：请求依次穿过 Skills、Filesystem、SubAgent、Summarization、PatchToolCalls、PromptCaching、Memory、HumanInTheLoop 等中间件" loading="lazy">
  <figcaption>图 06：一次请求穿过默认中间件栈。蓝色始终启用，黄色按参数启用，绿色是核心支架。</figcaption>
</figure>

主 agent 的默认栈大致顺序：

1. `SkillsMiddleware`：传入 `skills` 时启用，注入在文件系统中间件**之前**，先拿到 skill 元数据。
2. `FilesystemMiddleware`：处理读写与目录操作，传入 `permissions` 时权限校验在此执行。它是必需支架，不能被移除。
3. `SubAgentMiddleware`：生成并协调子代理，只有主 agent 暴露 `task` 工具。
4. `SummarizationMiddleware`：对话过长时压缩历史。
5. `PatchToolCallsMiddleware`：修复中断恢复后遗留的悬空工具调用。
6. `AsyncSubAgentMiddleware`：配置了异步子代理时启用。
7. **你传入的中间件**：在 Patch 之后合并。
8. Harness profile 附带的供应商专用中间件、排除工具过滤。
9. Prompt caching（Anthropic / Bedrock）：始终注册，对不支持的模型 no-op。
10. `MemoryMiddleware`：传入 `memory` 时启用，放在缓存之后减少缓存前缀失效。
11. `HumanInTheLoopMiddleware`：传入 `interrupt_on` 时启用。

### 11.1 替换默认中间件

传入的中间件按 `.name` 与默认栈匹配：**名字相同则原地替换，名字不同则插入到 `PatchToolCallsMiddleware` 之后**。替换需要 `deepagents>=0.7`。

注意：替换是**完全覆盖，不是合并**，替换实例必须自己配置好所有参数。尤其覆盖 `FilesystemMiddleware` 时，必须直接把 `backend`（和 `permissions`）传给它，它不会从 `create_deep_agent()` 继承。

```python
from deepagents import create_deep_agent
from deepagents.backends import StateBackend
from deepagents.middleware import SummarizationMiddleware

backend = StateBackend()
model = "openai:gpt-5.5"

custom_summarization = SummarizationMiddleware(
    model=model,
    backend=backend,
    trigger=("tokens", 100000),  # 超过 10 万 token 才压缩
    keep=("messages", 20),       # 保留最近 20 条消息
    summary_prompt="Your custom summary prompt here.",
)

agent = create_deep_agent(
    model=model,
    middleware=[custom_summarization],  # 按 name 替换默认 SummarizationMiddleware
)
```

### 11.2 添加自定义中间件

想在每次工具调用前后做日志、埋点、限流，可以用 `wrap_tool_call`：

```python
from langchain.agents.middleware import wrap_tool_call
from deepagents import create_deep_agent


@wrap_tool_call
def log_tool_calls(request, handler):
    """拦截并记录每次工具调用。"""
    result = handler(request)
    return result


agent = create_deep_agent(
    model="openai:gpt-5.5",
    tools=[get_weather],
    middleware=[log_tool_calls],
)
```

### 11.3 只读 agent：裁剪文件工具

`FilesystemMiddleware` 支持用 `tools` 白名单限制暴露哪些文件工具（需 `deepagents>=0.7`）。`read_file` 必须包含，否则报 `ValueError`；后端不支持时 `execute`、`delete` 会自动移除。

```python
from deepagents import create_deep_agent
from deepagents.middleware import FilesystemMiddleware

# 只读 agent：write_file、edit_file、delete、execute 永远不出现
agent = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    backend=backend,
    middleware=[
        FilesystemMiddleware(backend=backend, tools=["read_file", "ls", "glob", "grep"]),
    ],
)
```

### 11.4 中间件状态管理的坑

**不要在初始化后就地修改实例属性**来跨调用追踪值——那会在并发下引发竞态。需要计数器一类的跨 hook 状态时，写进图状态（按线程隔离，并发安全）。

```python
from langchain.agents.middleware import AgentMiddleware


class CountMiddleware(AgentMiddleware):
    def before_agent(self, state, runtime):
        return {"x": state.get("x", 0) + 1}  # 更新图状态，正确
```

子代理的中间件继承规则也要注意：

- general-purpose 子代理会继承主 agent 对默认中间件的**覆盖**，但不携带主 agent 专属中间件；它的 skills 运行在 `PatchToolCallsMiddleware` 之后。
- 通过 `subagents=` 声明的子代理**不继承**主 agent 的中间件自定义，需要在它自己的 `middleware` 字段里单独传。

## 12. Shell、沙箱与解释器

普通后端（State/Filesystem/Store）只暴露文件操作。当你需要 agent 真正**运行命令**——装依赖、跑测试、调 CLI、做系统级文件操作——就需要 sandbox 后端或解释器。

### 12.1 Sandbox 后端与 `execute`

沙箱本身就是一种 backend。配置后 agent 会额外获得 `execute` 工具，可在隔离环境里运行任意 shell 命令。框架每次模型调用会检查后端是否实现沙箱协议，没实现就把 `execute` 过滤掉，agent 根本看不到它。

可用提供商（各自独立安装包）：

| 提供商 | 安装包 | 后端类 |
| --- | --- | --- |
| LangSmith | `langsmith[sandbox]` | `LangSmithSandbox` |
| Daytona | `langchain-daytona` | `DaytonaSandbox` |
| E2B | `langchain-e2b` | `E2BSandbox` |
| Modal | `langchain-modal` | `ModalSandbox` |
| Runloop | `langchain-runloop` | `RunloopSandbox` |
| Vercel | `langchain-vercel-sandbox` | `VercelSandbox` |
| AgentCore | `langchain-agentcore-codeinterpreter` | `AgentCoreSandbox` |

沙箱后端唯一必须实现的方法是 `execute()`，其他文件操作都由基类基于它构建。`execute()` 返回合并的 stdout/stderr、退出码，输出过大时自动落盘并提示用 `read_file` 增量读取。

```python
from deepagents.backends.langsmith import LangSmithSandbox
from langsmith.sandbox import SandboxClient

client = SandboxClient()
backend = LangSmithSandbox(sandbox=client.create_sandbox())

result = backend.execute("python --version")
print(result.output)
```

两种集成模式：

- **Sandbox-as-tool（推荐，也是默认）**：agent 跑在你的机器上，需要时调用沙箱。密钥留在沙箱外、代码可即时更新，代价是每次调用有网络延迟。
- **Agent-in-sandbox**：agent 整个跑在沙箱里，更贴近本地开发，但密钥必须放进沙箱、更新要重建镜像。

生命周期：默认线程级（每个会话一个沙箱，建议配 `idle_ttl_seconds` 自动清理），也可助手级共享。沙箱持续计费，用完记得关。

> **安全铁律**：沙箱能隔离宿主，但挡不住上下文注入和网络泄露。**绝不要把 API key、令牌、数据库凭证放进沙箱**——被注入的 agent 会读走并外泄。正确做法是把密钥留在沙箱外的工具里，agent 只按名字调用工具、永远看不到凭证；必要时阻断沙箱网络、对所有工具调用开 HITL。把沙箱产出的一切当作不可信输入处理。

### 12.2 解释器（Interpreters）

如果你只需要让 agent 跑一点确定性的数据变换、循环、批处理，而不想给它整个 shell，可以用解释器：它添加一个 `eval` 工具，在受限的 QuickJS 运行时里跑 JavaScript。它**不提供** shell、包安装、文件系统或网络访问，因此比沙箱安全得多，适合“程序化地调用工具 / 转换结构化数据”。

选择原则：

- 只做数据变换、聚合、循环 → 解释器。
- 需要装依赖、跑测试、调系统命令 → 沙箱。
- 都不需要 → 普通 State/Filesystem/Store 后端就够。

## 13. MCP 与自定义工具生态

Deep Agents 的 `tools=` 参数同时接受三类东西：普通 Python 函数、LangChain 工具，以及**任意 MCP server 暴露的工具**。这让它可以直接接入数据库、外部 API、公司内部系统。

```python
from deepagents import create_deep_agent

# search / fetch_page / run_query 可以来自本地函数，
# 也可以来自 MCP server（通过 LangChain MCP 适配器加载后传进来）
agent = create_deep_agent(
    model="anthropic:claude-sonnet-4-6",
    tools=[search, fetch_page, run_query],
)
```

要点：

- 通过 `tools=` 加进来的自定义/MCP 工具**不受**文件系统白名单限制，也不受 `permissions` 约束——它们的安全边界要你自己在工具内部把控。
- MCP 工具的加载走 LangChain 的 MCP 集成，把 MCP server 的工具转成 LangChain 工具后再传入。
- 想彻底隐藏某些内置文件工具，可以注册 harness profile 用 `excluded_tools`；但 `FilesystemMiddleware` 本身是必需支架，不能被 `excluded_middleware` 移除。

## 14. Subagents：子代理

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

## 15. Structured Output

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

## 16. Memory：长期记忆

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

## 17. Skills：按需加载的能力包

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

## 18. Human-in-the-loop

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

## 19. Streaming 与调试

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

## 20. Context Engineering

Deep Agents 的上下文工程默认围绕四件事：

- System prompt：角色、目标、规则、输出标准。
- Tool prompts：工具描述和工具使用说明（filesystem、subagents、可选 planning 等中间件会自动追加各自的工具说明）。
- Filesystem offloading：把大内容写入文件，再按需读取。
- Summarization：上下文接近限制时压缩旧消息。

### 20.1 压缩生命周期：先卸载，再摘要

这套机制不是玄学，它有明确的阈值和顺序：

<figure class="deepagents-svg-figure">
  <img src="/deepagents-learning/context-lifecycle.svg" alt="Deep Agents 上下文压缩生命周期：对话增长后先把超过 2 万 token 的工具输入输出卸载到文件，越过 85% 时触发摘要" loading="lazy">
  <figcaption>图 07：上下文快满时的两级处理。先卸载大工具输入/输出到文件，再在 85% 阈值触发摘要。</figcaption>
</figure>

- **Offloading（卸载）**：单次工具的输入或输出超过约 **20,000 token** 时，Deep Agents 把内容写到 backend，对话里只留下**文件路径指针 + 前 10 行预览**，agent 后续可用 `read_file`/`grep` 按需拉回。当会话超过模型窗口约 **85%** 时，旧的冗余工具调用也会被截断成指针。
- **Summarization（摘要）**：越过 85% 且没有更多可卸载内容时，`SummarizationMiddleware` 触发。它做两件事——生成一份**结构化的上下文内摘要**（会话意图、已产出、下一步），替代工作内存里的完整历史；同时把**原始对话消息写入文件系统**作为规范记录。若模型 profile 不可用，回退为 17 万 token 触发 / 保留 6 条消息。
- **主动压缩**：`create_summarization_tool_middleware` 会给 agent 一个 `compact_conversation` 工具，让它在任务之间**主动**触发压缩，而不必等到 85%。这不会关掉自动摘要，二者共享同一引擎。

流式时，可以按 `metadata.lc_source == "summarization"` 把摘要产生的 token 过滤掉，避免污染前端输出。

### 20.2 实践模式

1. 让 agent 先写 todo。
2. 搜索或读取资料后写入 `/notes/`。
3. 中途产物写入 `/drafts/`。
4. 最终产物写入 `/outputs/`。
5. 对重要结论要求附证据来源。
6. 内存保持精简，只放始终相关的约定；任务专用能力交给聚焦的 skills。
7. 把繁重、输出量大的多步任务交给子代理，保持主 agent 上下文干净。

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

## 21. 推荐学习路线

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

## 22. 小项目：博客选题研究 Agent

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

## 23. 常见坑

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
- 替换默认中间件时以为是“合并”：其实是完全覆盖，覆盖 `FilesystemMiddleware` 忘了重新传 `backend`/`permissions` 会出错。
- 在中间件里就地改实例属性来计数：并发下会竞态，应写图状态。
- 把密钥放进 sandbox：被上下文注入的 agent 能读走并外泄，密钥要留在沙箱外的工具里。
- 该用解释器却上了 sandbox：只做数据变换却开放整个 shell，放大了攻击面。

## 24. 生产化 checklist

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

## 25. 官方资料入口

- Deep Agents overview: https://docs.langchain.com/oss/python/deepagents/overview
- Quickstart: https://docs.langchain.com/oss/python/deepagents/quickstart
- Customization（中间件）: https://docs.langchain.com/oss/python/deepagents/customization
- Context engineering: https://docs.langchain.com/oss/python/deepagents/context-engineering
- Backends: https://docs.langchain.com/oss/python/deepagents/backends
- Sandboxes: https://docs.langchain.com/oss/python/deepagents/sandboxes
- Interpreters: https://docs.langchain.com/oss/python/deepagents/interpreters
- Tools（含 MCP）: https://docs.langchain.com/oss/python/deepagents/tools
- Permissions: https://docs.langchain.com/oss/python/deepagents/permissions
- Subagents: https://docs.langchain.com/oss/python/deepagents/subagents
- Memory: https://docs.langchain.com/oss/python/deepagents/memory
- Skills: https://docs.langchain.com/oss/python/deepagents/skills
- Streaming: https://docs.langchain.com/oss/python/deepagents/streaming
- Human-in-the-loop: https://docs.langchain.com/oss/python/deepagents/human-in-the-loop
- API Reference: https://reference.langchain.com/python/deepagents/
- Deep Agents GitHub: https://github.com/langchain-ai/deepagents
- 官方示例: https://github.com/langchain-ai/deepagents/tree/main/examples

## 26. 生态与多语言版本

Deep Agents 不只有 Python SDK，围绕它还有一套生态，写生产系统前值得知道：

- **JavaScript / TypeScript 版本**：`deepagentsjs`（https://github.com/langchain-ai/deepagentsjs），API 与 Python 版对齐，前端 / Node 全栈项目可直接用。
- **Deep Agents Code**：官方预构建的终端编码 agent，定位类似 Claude Code / Cursor，可由任意 LLM 驱动。可用 `curl -LsSf https://langch.in/dcode | bash` 安装——注意这是从远程 URL 下载脚本直接执行，建议先下载审查脚本内容再决定是否运行。
- **分层可组合**：任何 LangGraph `CompiledStateGraph` 都能当作子代理传给 Deep Agent，你可以把已有的 LangGraph 工作流原样嵌进来。
- **模型无关**：支持前沿 API（OpenAI、Anthropic、Google、OpenRouter）、托管开源模型（Baseten、Fireworks）以及自托管（Ollama、vLLM、llama.cpp），只要支持 tool calling 即可。
- **配套工程**：基于 LangGraph 的流式、持久化、checkpoint，加上 LangSmith 的追踪、评估与部署，构成一条从原型到生产的链路。

> 安全提醒：Deep Agents 采用“信任 LLM”模型——agent 能执行其工具允许的任何操作。真正的边界应该在**工具 / 沙箱层面**强制实施，而不是指望模型自我约束。

## 27. 最短复习版

记住这张表：

| 你要解决的问题 | Deep Agents 里的东西 |
| --- | --- |
| 复杂任务怎么拆 | `write_todos` |
| 大上下文怎么放 | filesystem tools + offloading |
| 上下文快满了怎么办 | 卸载到文件 + `SummarizationMiddleware` |
| 多角色怎么做 | `subagents` / `task` |
| 中间产物放哪 | backend |
| 要跑命令 / 装依赖 | sandbox 后端 + `execute` |
| 只做数据变换 | interpreter + `eval` |
| 长期偏好怎么记 | `memory` |
| 专门工作流怎么注入 | `skills` |
| 接外部系统 | 自定义工具 / MCP |
| 危险操作怎么拦 | `interrupt_on` + checkpointer |
| 想改默认行为 | `middleware`（按 `.name` 替换） |
| 执行过程怎么看 | `stream(...)` + LangSmith |
| 输出怎么给程序消费 | `response_format` |

学习顺序：

1. 先跑通最小 agent。
2. 再写两个好工具。
3. 接着学文件系统和 backend。
4. 然后加 subagent。
5. 补 memory、skills、HITL、streaming、评测。
6. 最后深入 middleware、sandbox 与上下文压缩，把它当作可拆装的系统而非黑盒。

Deep Agents 的真正难点不是 API，而是产品和工程判断：哪些上下文该进 prompt，哪些该进文件，哪些该变成 skill，哪些该变成 memory，哪些操作必须让人批准，哪些能力该由中间件裁剪。
