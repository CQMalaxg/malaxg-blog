+++
date = '2026-05-12T20:52:27+08:00'
draft = false
title = 'Harness Engineering：理解 AI Agent 的工程底座'
+++
## 你用过的是模型，你要搭建的是系统

如果你用过 ChatGPT 或 Claude，你大概知道它们能做什么——回答问题、写代码、分析文档。你输入一段话，它输出一段话。这种交互方式很自然，也很强大。

但当你开始构建一个 AI Agent 时，你会很快发现，事情不再是这样运转的。

用模型和搭 Agent，这两件事之间有一道明确的分水岭。**用模型时，你是那个"循环"**——你读取模型的输出，判断它对不对，决定下一步做什么，再把新的上下文喂给它。你的大脑在负责记忆、判断、决策、纠错。模型只负责在每一步生成文字。

**搭 Agent 时，这个"循环"需要由系统来承担**。没有人在中间手动审阅每一条输出，没有人帮模型记住上次做到哪里，没有人在模型犯错时及时叫停，没有人帮它判断应该调用哪个工具、有没有权限这么做。所有这些，都需要你设计的系统来处理。

这个系统，就是 Harness。

---

## 一、Harness 是什么

最简洁的定义：

> **Agent = Model + Harness**

模型负责一件事：给定一段输入文本，生成下一段输出文本。推理、写代码、分析数据——这些在模型眼里都是同一件事：文字进，文字出。模型本身不会"执行"任何操作，不会"记住"任何跨会话的信息，不会"决定"什么时候停下来。

Harness 是模型之外的一切工程层——工具调用体系、上下文管理机制、持久化记忆、权限控制、多 Agent 协调。简单说，**让模型能在一个真实任务里稳定工作所需要的全部基础设施**，都是 Harness。

有一个类比流传得很广：**模型是 CPU，Harness 是操作系统**。这个类比抓住了一个关键真相：CPU 再强，没有操作系统，你也只能对着裸硬件写机器码。操作系统管理内存、调度进程、提供文件系统、控制权限——它让 CPU 的能力真正对应用程序可用。Harness 对模型做的事情，在结构上是相似的。

不过这个类比有其局限：操作系统管理的是确定性程序，给定相同输入，输出永远相同。模型是概率性的——相同输入可能产生不同输出，有时候会犯错，有时候会"幻觉"。Harness 必须处理这种不确定性，这是 OS 从来不需要面对的挑战。

Harness 也常被和另外两个概念混淆，有必要区分清楚：

| 层级 | 解决什么 | 核心关注点 |
|------|---------|-----------|
| **Prompt Engineering** | 怎么写指令才能让模型理解意图 | 一次对话的质量 |
| **Context Engineering** | 模型在某个时刻应该看到什么信息 | 信息的选择与组织 |
| **Harness Engineering** | Agent 整个执行过程中系统怎么稳定运转 | 长链路任务的可靠性 |

Prompt Engineering 是关于如何"说话"。Context Engineering 是关于"给模型看什么"。Harness Engineering 是关于整个系统的运行环境——如何设计约束、如何管理状态、如何处理失败、如何跨会话保持连续性。三者不是并列竞争关系，而是嵌套包含的层次结构。

---

## 二、上下文管理：模型的工作记忆

### 上下文窗口是什么

模型在执行任何任务时，它"知道"的一切——你的指令、对话历史、工具调用的结果、它自己的推理过程——全部存在一个地方：**上下文窗口（Context Window）**。

上下文窗口是模型在当前时刻能处理的全部文字。不在窗口里的，对模型来说不存在。这就是为什么当你在 ChatGPT 里聊了太久，它会开始"忘记"对话早期的内容——那些内容已经被移出了窗口。

主流模型的上下文窗口在 128K 到 200K token 之间（大致相当于 10 万到 15 万汉字）。听起来很大，但在真实的 Agent 任务中，窗口会出人意料地快被填满：任务描述、代码文件内容、工具调用的输入和输出、对话历史……一个稍微复杂的任务跑上十几步，窗口就满了大半。

### 反直觉的真相：上下文越多，Agent 越容易出错

这是 Harness Engineering 里最重要也最反直觉的一条经验：**往上下文里塞更多信息，通常不会让 Agent 变得更聪明，反而会让它变蠢。**

为什么？这需要理解模型处理长上下文时的几个机制性问题。

**注意力稀释**。Transformer 模型的自注意力机制会在窗口中的所有 token 之间分配权重，权重总和恒为 1。当窗口里有 1 万个 token 时，一条关键指令可能获得 0.5% 的注意力；当窗口扩展到 10 万个 token，同一条指令能获得的注意力权重被稀释到了 0.05%。信息还在，但模型对它的"关注程度"已经降低了一个数量级。

**中间丢失（Lost in the Middle）**。斯坦福大学的研究系统性地证明了这一现象：把关键信息放在长上下文的开头或结尾，模型的召回率最高；放在中间位置，准确率会下降 20% 到 40%。原因在于位置编码在极端位置（最开始和最末尾）有更强的区分信号，而中间段的位置编码相对"模糊"。

**两种偏差同时发生**。模型还存在明显的近因偏差（Recency Bias）——对窗口末尾的内容赋予更高权重，对早期内容关注度更低。这和"中间丢失"叠加的结果是：**只有上下文的头部和尾部是相对可靠的信息区域，大量塞进中间的内容往往处于"信息黑洞"里**。

实验数据印证了这个结论：以 168K token 的上下文为例，**当利用率超过大约 40% 时，Agent 的输出质量就会开始明显下降**——幻觉增多、忘记已完成的步骤、工具调用参数出错、在同一个问题上反复绕圈。

### Harness 怎么管理上下文

Harness 的上下文管理层，本质上是在回答一个持续的问题：**在任何时刻，模型的上下文窗口里应该放什么？**

这不是一次性的决定，而是需要随任务进展动态调整的工程。几种核心策略：

**选择性保留（Selective Retention）**。不是所有信息都等价。工具返回的原始结果（文件内容、代码执行输出、API 响应）是不可再生的——丢了就没了。而模型的中间推理过程（"我认为应该先检查 A，再处理 B，因为……"）大多可以从工具结果中重新推导。压缩上下文时，优先丢弃中间推理，保留工具结果和关键指令。

**锚点注入（Pinned Context）**。某些信息需要永远在上下文里，不能被滚出：任务的总目标、核心约束条件、已确认的关键事实、最近一次的失败记录。Harness 把这些信息"钉"在上下文头部，不管窗口怎么滚动，它们始终可见。

**滑动窗口压缩（Rolling Window）**。当对话历史过长时，不是简单截断，而是把超出范围的历史先经过摘要压缩，以更紧凑的形式保留其中的关键信息。类似操作系统的虚拟内存：不常用的数据被"换出"到更低密度的存储形式，需要时再以压缩摘要的形式"换回来"。

**主动重启（Context Reset）**。当任务足够长、上下文已经接近填满时，最彻底的做法是主动清空上下文窗口，启动一个全新的 Agent 会话，通过结构化的交接文档（后面会详细说）把关键状态传递给新的 Agent。类似重启一个内存耗尽的进程——代价是丢失一些中间状态，但换来的是全新的、干净的工作环境。

上下文管理是 Harness 里最基础的能力，也是最容易被忽视的一层。很多 Agent "越跑越蠢"的根本原因，不是模型能力不足，而是上下文管理缺失——模型在信息噪声中挣扎，把大量注意力花在了不重要的内容上。

---

## 三、工具体系：模型如何与世界交互

### 模型只能生成文字

这是一个经常被忽略的基本事实：**模型本身不能做任何事，除了生成文字**。

它不能真正"执行"代码——它只能生成代码文字。它不能"读取"文件——它只能根据上下文里已有的文件内容进行推理。它不能"搜索网络"——它只能根据训练数据里的知识回答问题。

那么，当我们说一个 Agent "运行了代码"或"读取了文件"，实际上发生了什么？

答案是：Harness 做了这些事，并把结果告诉了模型。

### 工具是什么

工具（Tool）是你暴露给模型的能力接口——一个模型可以"请求调用"的函数。模型不直接执行这个函数；它输出一个结构化的请求，Harness 拦截这个请求，实际执行对应的操作，然后把结果作为文字返回给模型。

一次完整的工具调用流程是这样的：

```
1. 你定义工具（名称、描述、参数定义）
2. 工具定义被放入模型的上下文
3. 模型在推理时决定需要调用某个工具
4. 模型输出一个结构化的 tool_use 块（包含工具名和参数）
5. Harness 解析这个请求，做参数校验和权限检查
6. Harness 实际执行对应的函数（读文件、运行代码、调用 API……）
7. 执行结果以 tool_result 消息的形式返回给模型
8. 模型继续推理，把这个结果纳入考量
```

在模型眼里，整个过程是：我想知道某个文件的内容 → 我请求调用 `read_file` 工具 → 我得到了文件内容。但实际上，"读文件"这个操作是 Harness 做的，模型只是发出了请求并接收了结果。

### 工具设计为什么至关重要

模型如何决定调用哪个工具、用什么参数？答案是：完全依赖你对工具的描述。

当模型拿到工具定义时，它看到的是：工具的名字、描述这个工具做什么的文字、每个参数的名字和描述。没有别的。它没有任何其他方式去理解这个工具的行为。这意味着：**工具的描述质量，直接决定模型使用工具的准确率**。

来看一个对比：

```
# 差的工具定义
{
  "name": "run",
  "description": "运行命令",
  "parameters": {
    "cmd": { "type": "string" }
  }
}

# 好的工具定义
{
  "name": "run_shell_command",
  "description": "在受限沙箱中执行 shell 命令。仅支持读操作（ls、cat、grep、find）。
                  禁止写操作（rm、mv、cp）。命令超时为 10 秒。工作目录为 /workspace。",
  "parameters": {
    "command": {
      "type": "string",
      "description": "完整的 shell 命令，例如 'ls -la /workspace' 或 'grep -r error ./logs'"
    }
  }
}
```

好的定义告诉模型：这个工具能做什么（读操作）、不能做什么（写操作）、有什么限制（10 秒超时）、参数应该长什么样（给了具体例子）。模型能基于这些信息做出正确的调用决策。差的定义让模型在猜——猜错了，调用失败，任务出错。

同样重要的是**错误信息的设计**。当工具调用失败时，Harness 会把错误信息返回给模型，让它决定如何恢复。如果返回的是 `{"error": "failed"}`，模型完全无法判断发生了什么，只能盲目重试或放弃。如果返回的是 `{"error": "文件 /workspace/data.csv 不存在，当前 /workspace 目录下有：config.json、input.txt"}`, 模型就能直接定位问题并修正参数。

工具体系还需要处理并发问题。现代模型可以在一次响应中请求调用多个工具，Harness 会并行执行它们以提高效率。但如果两个并发工具同时写入同一个文件，后写入的会覆盖前者，而两个工具都报告"成功"——这种竞态问题对模型完全透明，它根本不知道发生了什么。Harness 必须在执行层面处理这类并发安全问题，这是模型完全感知不到、也无能为力的领域。

---

## 四、记忆与状态：跨越会话的连续性

### 每次会话都从零开始

这是初次构建 Agent 的开发者最容易被惊到的地方：**会话结束后，模型的上下文完全清空**。下次启动新会话时，它对上一次会话的一切一无所知。

对于简单的单次任务，这不是问题——任务在一个会话里完成，不需要跨会话记忆。但对于复杂任务，这是一个根本性的挑战：

- 一个需要几小时才能完成的代码重构任务，中间必然要多次开启新会话
- 一个要持续学习你的代码库偏好的 Agent，需要跨会话积累认知
- 一个工作流 Agent，需要记住上次执行到了哪一步、哪些子任务已经完成

没有记忆机制的 Agent，就像一个每天上班都失忆的员工——工作能力没问题，但每次都得重新了解项目、重新犯同样的错误、重新走同样的弯路。

### 三种不同类型的记忆

理解 Agent 记忆系统，需要区分三种本质不同的记忆类型：

**当次任务记忆（Working Memory）**

这就是上下文窗口本身。在一个会话内，模型对当前任务的所有了解都在这里——你的指令、工具返回的结果、已经完成的步骤、当前正在处理的内容。分辨率最高，更新最即时，但是临时的——会话结束就消失。

**任务历史记忆（Episodic Memory）**

记录跨会话的经历：这个任务之前做到了哪里、尝试过哪些方案、哪些失败了、原因是什么、已经确认了哪些事实。这些信息存储在上下文窗口之外（通常是文件系统或数据库），在新会话开始时按需加载回来。

这类记忆的核心价值是**防止重复犯错**。一个 Agent 在第一个会话里花了 30 分钟排查后发现某个方案行不通，如果没有记录，下一个会话里它可能又走同一条弯路。把失败案例明确记录下来，是让 Agent 在跨会话任务中能真正"进步"的前提。

**知识库记忆（Semantic Memory）**

Agent 执行任务所需的背景知识：你的代码库内容、项目文档、业务规则、API 参考手册。这些信息量通常远超上下文窗口容量，不可能全部常驻窗口。

解法是 RAG（检索增强生成）：把知识库存储在向量数据库中，Agent 需要某类知识时，Harness 根据当前任务的语义相关性检索最相关的片段，注入上下文。而不是一次性全部塞进去。就像图书馆——你不需要把所有书都放在桌上，需要哪本书就去取哪本。

### 交接文档：跨会话的状态传递

当一个 Agent 会话因为任何原因结束（完成阶段性任务、上下文窗口满了、程序崩溃），Harness 需要保存足够的状态信息，让下一个会话能从断点继续，而不是从零开始。

这个状态信息就是**交接文档（Handoff Document）**。它不是给人读的自然语言摘要，而是结构化的状态快照，供下一个 Agent 实例解析和使用。

一份设计良好的交接文档至少需要包含五个维度：

```json
{
  "objective": "将认证模块从 JWT v1 迁移到 v2，保持接口向后兼容",
  "completed_steps": [
    {"step": "分析现有实现", "result": "发现 3 处硬编码密钥"},
    {"step": "更新依赖版本", "result": "jsonwebtoken 8.5.1 → 9.0.0，已测试通过"}
  ],
  "current_state": "正在修改 middleware/auth.js 的 verify 函数",
  "failures": [
    {
      "attempt": "直接替换 verify 回调函数",
      "error": "TypeError: callback is not a function",
      "conclusion": "v9 API 已改为 Promise 风格，不支持回调，需要重写"
    }
  ],
  "next_steps": ["将 verify 重写为 async/await", "更新对应的单元测试"]
}
```

`failures` 字段是最容易被忽略、也最关键的部分——它记录了已经走过的死路，防止新会话里的 Agent 重复同样的错误。

为什么用 JSON 而不是自然语言？因为 JSON 的 Schema 可以被程序校验。如果一个字段缺失，系统会立刻报错，而不是等到执行过程中才发现信息缺失。自然语言写的交接文档可能说"基本完成了备份"——这个"基本"意味着什么？备份在哪里？大小多少？这种模糊性在任务关键路径上是危险的。结构化格式强制完整性，而不是依赖写作者的细心。

---

## 五、权限与安全边界：限制不是枷锁

### 为什么不能给 Agent 所有权限

初次接触 Agent 系统的工程师，常常会做一个看似合理的决定：给 Agent 最宽泛的权限，免得它因为权限不足而失败。这个思路有一个根本性的问题。

**Agent 会犯错。**

不是因为模型不够好，而是因为任何足够复杂的系统都会犯错——概率系统更是如此。模型可能误解你的指令，可能推理出错误的结论，可能在一个场景里应用了本该用在另一个场景的逻辑。这些错误在简单任务里只是小麻烦；在有完整权限的 Agent 里，它们可能是灾难。

一个真实案例：某团队的代码 Agent 有完整的文件系统写权限。有一次，模型误判了任务意图，认为"清理残留文件"是必要步骤，结果把整个 `/src` 目录下的文件全部替换成了空文件。没有沙箱，没有权限限制，人工恢复花了两个多小时。

这不是模型的问题——这是 Harness 的问题。Harness 没有把 Agent 能造成的损害限制在可接受的范围内。

### 最小权限原则

最小权限原则（Principle of Least Privilege）在软件工程里不是新概念，但在 Agent 系统里有更强的适用理由：**权限越小，错误的代价越低，系统越容易调试**。

不只是安全意义上的"防止 Agent 越界"，还有工程意义上的"让系统行为可预测"。一个只有只读权限的 Agent，哪怕逻辑完全出错，也无法对文件系统造成任何破坏；一个有完整写权限的 Agent，任何一个逻辑错误都可能产生不可逆的副作用。

权限分级在实践中大致分为几个层次：

- **只读访问**：可以查看文件、读取数据，但不能修改任何内容。适合分析类、审查类任务
- **受限读写**：在指定目录内可以读写（如 `/workspace/output`），其他区域只读。适合代码生成类任务
- **命令执行**：可以执行特定的命令，但命令集合经过白名单约束。适合需要运行测试、编译代码的任务
- **网络访问**：可以访问外部网络，但只允许白名单内的目标地址。防止意外数据外泄或产生不预期的外部副作用

### 沙箱：隔离爆炸半径

沙箱（Sandbox）是权限控制的底层实现机制：把 Agent 的执行环境隔离在一个受限的容器里，里面发生的一切不会影响到容器外的真实环境，除非通过明确定义的接口。

操作系统级别的沙箱通常涉及几个维度：**文件系统命名空间**（Agent 看到的文件系统是一个隔离的视图，看不到宿主机的真实文件系统）、**网络隔离**（默认不能访问外部网络，只有白名单域名例外）、**进程能力控制**（通过 Linux capabilities 限制能做的系统调用类型）、**资源配额**（CPU、内存、磁盘用量都有上限，防止 Agent 把宿主机资源耗尽）。

沙箱的价值不仅是安全，也是**可恢复性**。一个在沙箱里运行出错的 Agent，它造成的所有损坏都局限在沙箱内——清空沙箱重来就行。而一个在真实环境里运行出错的 Agent，损坏可能扩散到生产数据、代码库、外部服务，恢复成本远非比较。

### 人在回路：不可逆操作的守门人

不是所有操作都适合完全自动化。对于某些高风险、不可逆的操作，Harness 的正确行为是**暂停、请求人工确认，而不是自主执行**。

典型场景包括：删除数据库记录、向生产环境推送变更、发送外部通知（邮件、消息）、访问超出预定权限范围的资源。

这不是不信任模型，而是认可一个现实：某些错误的代价高到不值得承担。与其事后处理一个 Agent 误删了生产数据的事故，不如在执行前加一个人工确认的关卡。

实现上，Harness 在这类操作前将"待执行操作"序列化写入一个等待队列，Agent 进入暂停状态；人工审核批准后，Harness 通知 Agent 继续执行。这个机制让人类在系统中保持了对关键节点的控制权，而不是把一切都交给自动化。

---

## 六、编排：从一个 Agent 到一支团队

### 单 Agent 的天花板

对于简单任务，一个 Agent 够了。但随着任务变得更复杂，单 Agent 会遭遇一个结构性瓶颈：**它只有一个上下文窗口**。

一个上下文窗口意味着：所有的任务状态、所有的工具调用历史、所有的推理过程，都必须挤在一个有限空间里。任务越复杂，上下文窗口越快填满，Agent 越快进入"Dumb Zone"。

更根本的问题是：**一个通才 Agent 在任何时候的上下文里都携带着大量无关信息**。如果你让一个 Agent 同时负责写代码、写文档、做代码审查，它的上下文里就会混杂着三件事的工具定义、指令和中间结果——大量信息对当前子任务来说是噪声。

专才 Agent 解决这个问题：每个 Agent 只做一件事，上下文只包含和这件事相关的信息，噪声最小，注意力最集中，出错率最低。**专业化本身就是上下文管理策略**，不只是组织架构的考量。

### 三种编排拓扑

多个 Agent 协作时，如何组织它们之间的关系？有三种基本拓扑：

**中央调度（Orchestrator-Subagent）**

一个 Orchestrator Agent 理解整体目标，把具体子任务分配给专业化的 Subagent，收集它们的结果，决定下一步。

```
          ┌─────────────────┐
          │   Orchestrator  │
          └────────┬────────┘
        ┌──────────┼──────────┐
        ▼          ▼          ▼
   代码 Agent  文档 Agent  测试 Agent
```

适合目标明确但执行路径复杂的任务。Orchestrator 需要保持全局视图，因此上下文管理压力较大——所有 Subagent 的进展报告都需要汇报给它，随着任务推进，Orchestrator 的上下文会持续增长。好的设计是让 Subagent 上报摘要而非原文，减少 Orchestrator 的上下文压力。

**并行扇出（Parallel Fan-out）**

任务被切分成 N 个独立的同构子任务，N 个 Agent 并行处理，最后由一个汇总步骤合并结果。

```
[任务] → [分片] → [Agent1][Agent2][Agent3]...[AgentN] → [汇总] → [结果]
```

适合"处理列表中的每一项"类型的任务：分析代码库里的每个模块、并行翻译多个文档、同时执行多组测试。关键前提是子任务必须真正独立——如果 Agent2 的结果依赖 Agent1 的输出，就不能用并行扇出，需要流水线。

**流水线（Pipeline）**

每个 Agent 是一个处理阶段，上游的输出是下游的输入，形成一条处理链：

```
需求分析 Agent → 架构设计 Agent → 代码生成 Agent → 测试 Agent → 审查 Agent
```

适合有明确阶段顺序的多步骤任务。每个阶段的接口设计非常关键——上下游之间传递的应该是结构化的 Artifact（一份 JSON 格式的设计文档、一个明确的函数签名列表），而不是自然语言描述。模糊的"你觉得合适就行"传递到下一个阶段会迅速放大成混乱。

### Agent 之间如何通信

多 Agent 系统里，一个直觉上看似自然的做法是让 Agent 直接互相"聊天"——你把一个 Agent 的输出直接发给另一个 Agent，它们来回交流。这个做法在生产环境里几乎总是行不通。

原因：**每轮对话都会把对方的上下文带入自己的窗口**。三轮之后，双方的窗口里都充斥着对方的推理过程，有效信息密度急剧下降，很快双双进入 Dumb Zone。

正确做法是通过**结构化 Artifact** 传递信息：Agent A 把输出写成一个 JSON 文件，Agent B 直接读取这个文件的内容。没有实时的来回对话，只有明确定义的数据接口。

Git 提交是这个模式在代码协作场景里的自然实现：Agent A 完成一个功能，写一个 commit，commit message 描述做了什么；Agent B 检出这个 commit，继续在它的基础上工作。Agents 之间从不"交谈"，只通过版本控制系统里的结构化记录交流。Git 本身的冲突检测机制也天然解决了多个 Agent 并发修改同一文件的问题。

### 多 Agent 系统的常见失败模式

**雪崩效应**：流水线中，上游 Agent 输出了低质量的结果，下游 Agent 不加验证地接受并继续处理，错误被逐层放大。到最后一个阶段，问题已经深度嵌入整个处理链，几乎无法追溯源头。解法是在每个阶段出口加入验证步骤——不满足质量条件的 Artifact 不允许流入下一阶段，而是原地报错、提前中断。

**循环依赖**：Agent A 在等待 Agent B 的输出，Agent B 在等待 Agent A 的输出，系统无限期挂起。解法是在任务设计时强制构建无环的依赖图（DAG），任何形成环路的依赖关系在规划阶段就应该被检测和拒绝。

**责任真空**：一个跨越多个 Agent 职责边界的问题，在 A 的视角里属于 B 的职责，在 B 的视角里属于 A 的职责，结果没有任何 Agent 处理它。解法是每个子任务有且只有一个明确的责任 Agent，完成时需要声明"哪些问题我没有处理、由谁接手"。

---

## 七、Harness 的完整图景

把前面五个部分拼在一起，Harness 的全貌是这样的：

<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 760 570" style="width:100%;max-width:760px;display:block;margin:1.5em auto;">
  <defs>
    <clipPath id="hcp1"><rect x="40" y="90" width="330" height="120" rx="8"/></clipPath>
    <clipPath id="hcp2"><rect x="390" y="90" width="330" height="120" rx="8"/></clipPath>
    <clipPath id="hcp3"><rect x="40" y="228" width="330" height="120" rx="8"/></clipPath>
    <clipPath id="hcp4"><rect x="390" y="228" width="330" height="120" rx="8"/></clipPath>
    <clipPath id="hcp5"><rect x="40" y="366" width="680" height="78" rx="8"/></clipPath>
    <linearGradient id="hmg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" stop-color="#4F46E5"/>
      <stop offset="100%" stop-color="#7C3AED"/>
    </linearGradient>
    <filter id="hcs" x="-5%" y="-5%" width="110%" height="120%">
      <feDropShadow dx="0" dy="1" stdDeviation="2.5" flood-color="#00000018"/>
    </filter>
    <filter id="hms" x="-10%" y="-10%" width="120%" height="135%">
      <feDropShadow dx="0" dy="4" stdDeviation="10" flood-color="#4F46E532"/>
    </filter>
  </defs>

  <!-- Agent 外框 -->
  <rect x="8" y="8" width="744" height="554" rx="16" fill="#F8FAFC" stroke="#CBD5E1" stroke-width="1.5"/>
  <rect x="8" y="8" width="744" height="44" rx="16" fill="#F1F5F9"/>
  <rect x="8" y="36" width="744" height="16" fill="#F1F5F9"/>
  <text x="36" y="34" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="11" font-weight="700" fill="#94A3B8" letter-spacing="2">AGENT  SYSTEM</text>

  <!-- Harness 区域 -->
  <rect x="24" y="56" width="712" height="400" rx="12" fill="#EEF2FF" stroke="#C7D2FE" stroke-width="1.5"/>
  <text x="44" y="76" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="11" font-weight="700" fill="#6366F1" letter-spacing="2">HARNESS</text>

  <!-- 卡片 1：上下文管理 -->
  <rect x="40" y="90" width="330" height="120" rx="8" fill="white" stroke="#E8EAED" stroke-width="1" filter="url(#hcs)"/>
  <rect x="40" y="90" width="330" height="36" fill="#6366F1" clip-path="url(#hcp1)"/>
  <text x="58" y="113" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="12" font-weight="700" fill="white">上下文管理</text>
  <text x="163" y="113" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="9" fill="#A5B4FC">Context Management</text>
  <text x="56" y="142" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 窗口利用率监控与 Smart Zone 管理</text>
  <text x="56" y="159" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 压缩、摘要与锚点注入策略</text>
  <text x="56" y="176" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 选择性保留与主动上下文重置</text>

  <!-- 卡片 2：工具体系 -->
  <rect x="390" y="90" width="330" height="120" rx="8" fill="white" stroke="#E8EAED" stroke-width="1" filter="url(#hcs)"/>
  <rect x="390" y="90" width="330" height="36" fill="#10B981" clip-path="url(#hcp2)"/>
  <text x="408" y="113" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="12" font-weight="700" fill="white">工具体系</text>
  <text x="482" y="113" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="9" fill="#6EE7B7">Tool Interface</text>
  <text x="406" y="142" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 工具 Schema 定义与调用描述</text>
  <text x="406" y="159" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 调用解析、校验与执行分发</text>
  <text x="406" y="176" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 错误处理、重试与并发保护</text>

  <!-- 卡片 3：记忆与状态 -->
  <rect x="40" y="228" width="330" height="120" rx="8" fill="white" stroke="#E8EAED" stroke-width="1" filter="url(#hcs)"/>
  <rect x="40" y="228" width="330" height="36" fill="#F59E0B" clip-path="url(#hcp3)"/>
  <text x="58" y="251" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="12" font-weight="700" fill="white">记忆与状态</text>
  <text x="156" y="251" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="9" fill="#FDE68A">Memory &amp; State</text>
  <text x="56" y="280" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 当次任务记忆（上下文窗口内）</text>
  <text x="56" y="297" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 跨会话持久化与结构化交接文档</text>
  <text x="56" y="314" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 知识库检索（RAG / 向量存储）</text>

  <!-- 卡片 4：权限与安全边界 -->
  <rect x="390" y="228" width="330" height="120" rx="8" fill="white" stroke="#E8EAED" stroke-width="1" filter="url(#hcs)"/>
  <rect x="390" y="228" width="330" height="36" fill="#F43F5E" clip-path="url(#hcp4)"/>
  <text x="408" y="251" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="12" font-weight="700" fill="white">权限与安全边界</text>
  <text x="533" y="251" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="9" fill="#FECDD3">Permissions &amp; Safety</text>
  <text x="406" y="280" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 沙箱隔离（文件系统 / 网络 / 进程）</text>
  <text x="406" y="297" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 最小权限原则与分级授权控制</text>
  <text x="406" y="314" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 人在回路（不可逆操作守门）</text>

  <!-- 卡片 5：编排层（全宽） -->
  <rect x="40" y="366" width="680" height="78" rx="8" fill="white" stroke="#E8EAED" stroke-width="1" filter="url(#hcs)"/>
  <rect x="40" y="366" width="680" height="36" fill="#8B5CF6" clip-path="url(#hcp5)"/>
  <text x="58" y="389" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="12" font-weight="700" fill="white">编排层</text>
  <text x="112" y="389" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="9" fill="#DDD6FE">Orchestration</text>
  <text x="56" y="416" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· Agent 专业化分工与协作</text>
  <text x="264" y="416" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 拓扑：中央调度 / 并行扇出 / 流水线</text>
  <text x="56" y="432" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· Artifact 通信协议（非直接对话）</text>
  <text x="264" y="432" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#475569">· 失败模式检测与系统恢复</text>

  <!-- 连接箭头 -->
  <circle cx="380" cy="456" r="3" fill="#C7D2FE"/>
  <line x1="380" y1="456" x2="380" y2="497" stroke="#94A3B8" stroke-width="1.5" stroke-dasharray="5,3"/>
  <polygon points="373,495 380,508 387,495" fill="#94A3B8"/>

  <!-- Model 框 -->
  <rect x="195" y="508" width="370" height="52" rx="12" fill="url(#hmg)" filter="url(#hms)"/>
  <text x="380" y="531" text-anchor="middle" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="14" font-weight="700" fill="white">Model</text>
  <text x="380" y="549" text-anchor="middle" font-family="-apple-system,BlinkMacSystemFont,'Segoe UI',Helvetica,sans-serif" font-size="10.5" fill="#C7D2FE">推理 · 生成 · 语言理解</text>
</svg>

这五个部分不是孤立的功能模块，而是互相依赖的一个整体：

- 工具调用的结果要进入上下文，上下文管理决定这个结果保留多久、怎么压缩
- 记忆系统依赖工具体系（读写文件是工具调用）和上下文管理（决定什么历史信息加载回来）
- 权限系统横跨工具调用和沙箱执行，它是工具体系的安全约束层
- 编排层在更高维度上调度多个 Agent，每个 Agent 都有自己独立的上下文管理、工具体系、记忆和权限配置

### 类比的边界

回到操作系统的类比——它帮助我们快速建立直觉：内存管理对应上下文管理，文件系统对应持久化记忆，设备驱动对应工具接口，权限系统对应沙箱，进程调度对应编排。

但 Harness 有一些 OS 不需要处理的问题：

**不确定性管理**。操作系统管理的是确定性程序——相同输入总是产生相同输出。模型是概率性的——同一个任务可能走不同的路径，相同的错误信息可能触发不同的恢复策略。Harness 需要处理这种内在的不确定性，OS 从来不需要考虑这个问题。

**质量评估**。操作系统不需要判断一个程序"做得好不好"——只要程序正常退出就算成功。Harness 需要对模型的输出质量有一定的判断能力：这个工具调用的参数合理吗？这个中间结果符合预期吗？任务真的完成了，还是 Agent 以为完成了？这需要 Harness 层的评估机制（Evals），是 OS 完全没有对应物的能力。

**反馈回路**。OS 不会把程序的错误记录下来，让下次运行同类程序时"吸取教训"。Harness 需要：把 Agent 的失败案例写入 AGENTS.md 或类似的记忆文档，让后续的 Agent 会话能看到前人踩过的坑，从而避免重蹈覆辙。这是一个持续学习的机制，在 OS 的概念框架里没有直接的对应物。

### 工程师角色的转变

有一个实验数据值得反复思考：同一个模型，没有做任何调整，只是把工具调用接口的格式设计得更好，基准测试分数从 6.7% 跳到了 68.3%。模型没有变聪明——变的是它运行的环境。

这说明了什么？**在今天，Harness 工程的质量对 Agent 效果的影响，并不亚于模型本身的能力。**

这对工程师意味着一个角色上的转变。以前，工程师的工作是"写代码"——直接实现业务逻辑。在 Agent 系统里，工程师的核心工作变成了**设计让 AI 能稳定工作的环境**——设计约束（权限和沙箱）、设计信息流（上下文管理）、设计记忆（状态持久化）、设计工具（接口和错误处理）、设计协作方式（编排）。

这不是更简单的工作，而是不同维度的工作。模型的不确定性、上下文的有限性、工具调用的可靠性、跨会话状态的连续性——这些是新的工程挑战，需要新的工程思维。

Harness Engineering 的核心，是理解**你的工作不是教会模型做某件事，而是搭建一个让模型能可靠完成某件事的系统**。这两件事之间的距离，就是 Harness 存在的意义。
