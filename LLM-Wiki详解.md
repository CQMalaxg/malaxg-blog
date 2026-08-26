# LLM Wiki 详解：从知识检索到知识编译

> 一份关于 LLM Wiki 知识编译范式的完整学习文档。涵盖它是什么、思想起源、三层架构、三大操作、工程化落地、以及适用边界。
>
> 本文融合了 Karpathy 原始范式的公开资料与一套企业级 LLM Wiki 系统的真实工程实现（已做通用化脱敏）。文末附参考资料与事实核查说明。

---

## 目录

- [一、一句话理解 LLM Wiki](#一一句话理解-llm-wiki)
- [二、核心洞察：编译知识，而非检索知识](#二核心洞察编译知识而非检索知识)
- [三、思想起源：一个 1500 万浏览的"旧点子"](#三思想起源一个-1500-万浏览的旧点子)
- [四、三层架构](#四三层架构)
- [五、六种页面类型（产物的类型系统）](#五六种页面类型产物的类型系统)
- [六、Frontmatter：产物的机器可读元数据](#六frontmatter产物的机器可读元数据)
- [七、三大操作：知识库如何"活"起来](#七三大操作知识库如何活起来)
- [八、输入 / 处理 / 输出 全景](#八输入--处理--输出-全景)
- [九、工程化落地：从个人玩具到生产系统](#九工程化落地从个人玩具到生产系统)
- [十、增量编译：两份账本的解耦设计](#十增量编译两份账本的解耦设计)
- [十一、状态机](#十一状态机)
- [十二、双提交工作流](#十二双提交工作流)
- [十三、发布：两遍推送破解"鸡生蛋"](#十三发布两遍推送破解鸡生蛋)
- [十四、端到端实例：一个源的完整生命周期](#十四端到端实例一个源的完整生命周期)
- [十五、从零复现的最小清单](#十五从零复现的最小清单)
- [十六、冷静一点：它不是银弹](#十六冷静一点它不是银弹)
- [参考资料与事实核查说明](#参考资料与事实核查说明)

---

## 一、一句话理解 LLM Wiki

**LLM Wiki 是一个「知识编译器」**：它把杂乱的原始资料（raw）当作**源代码**，用一个受严格提示词约束的 **LLM Agent** 作为**编译器**，产出结构化、相互链接、可持续增量更新的**知识库产物**（wiki/docs），最后可发布到团队平台供人阅读。

用 Karpathy 本人的类比：

> **"Obsidian 是 IDE，LLM 是程序员，Wiki 是代码库。"**
>
> **"LLM 是作者，人是主编。"**

人负责机器做不好的事——**策展**（挑选高质量来源）、**探索**（决定研究方向）、**提问**（提出好问题）；LLM 负责那些"无人愿做、成本极低"的繁重簿记：总结、建立交叉引用、归档、记录变更。

---

## 二、核心洞察：编译知识，而非检索知识

这是整套范式的灵魂。

如果你用 RAG 搭过知识库，多半有过这种别扭感：明明是同一批文档，用户每问一次，系统就要重新检索、重新塞进上下文、让模型重新"读懂"一遍。答案用完即弃，下一次从零再来。

Karpathy 用一个精准的类比点破了这件事的荒谬：

> 这就像**每次运行程序都重新解释一遍源代码，而不是先把它编译好**。
>
> **RAG 是解释型语言，LLM Wiki 是编译型语言。**

两种范式在"知识何时被处理"这件事上，存在根本分歧：

![编译知识 vs 检索知识](assets/01-compile-vs-retrieve.svg)

*图 1：RAG 在「查询时」反复现读、不积累；LLM Wiki 在「摄取时」编译一次、之后反复复用并自我生长。*

| 维度 | 传统 RAG（解释型） | LLM Wiki（编译型） |
|------|-------------------|-------------------|
| 知识处理时机 | 查询时——每次提问都重来 | 摄取时——每个源只处理一次 |
| 是否积累 | 不积累，每次从零"重新发现" | 随每个源和每次查询复利增长 |
| 算力/Token | 每次都加载大量原始文档 | 读索引即可导航，Token 大幅下降 |
| 基础设施 | 向量数据库、embedding 管道 | 零基础设施，纯 Markdown + Git |
| 可读性 | chunk 碎片，人难以直接阅读 | 人类可读的 wiki，可直接编辑、可 diff |
| 矛盾与缺口 | 难以显式追踪 | 元数据里显式记录矛盾与待解问题 |
| 维护者 | 系统（黑盒） | LLM（透明、有据可查） |

> ⚠️ **预防针**：社区里流传"Token 减少约 95%、比 RAG 高效约 70 倍"之类的说法，这些数字来自个别实践者的估算，**并非严格基准测试**，请当作"方向性直觉"而非"性能承诺"。

---

## 三、思想起源：一个 1500 万浏览的"旧点子"

- **提出者**：Andrej Karpathy（OpenAI 联合创始人、前特斯拉 AI 高级总监、"vibe coding"一词提出者）
- **时间**：2026 年 4 月初，一条推文 + 一个 GitHub gist（"LLM Wiki" idea file）
- **热度**：约 1500 万浏览、数万转发收藏，引爆 AI 社区
- **原话**：*"using LLMs to build personal knowledge bases for various topics of research interest"*（用 LLM 为各研究主题构建个人知识库）

**这不是新问题，而是老问题终于等来了答案。** 把知识组织成相互链接的网络，这个梦想至少可以追溯到：

- **1945 年 Vannevar Bush** 在《As We May Think》中构想的 **Memex**；
- 社会学家 **Niklas Luhmann** 用一生实践的 **Zettelkasten（卡片盒笔记法）**——"原子化笔记 + 密集互链"。

但这些方法有一个共同死穴：**谁来维护？** 手动建立和更新成千上万条交叉引用，是一项没人愿意长期坚持的苦役。而 LLM 恰好补上了这最后一块拼图——一个不知疲倦、成本近乎为零的知识管理员。

---

## 四、三层架构

LLM Wiki 的骨架是三层，各司其职、边界清晰。这套分层正是它区别于"让 LLM 随便记点笔记"的关键。

![LLM Wiki 三层架构](assets/02-three-layers.svg)

*图 2：三层架构。数据只能自下而上流动。Karpathy 特别强调，最容易被忽视却最关键的是第三层 Schema——它是让 LLM 保持纪律的「语言规范」。*

| 层 | 是什么 | 软件工程类比 | 可变性 | 谁拥有 |
|----|--------|-------------|--------|--------|
| **① Raw** | 原始来源（文档、代码、网页存档） | 源代码 `.c/.java` | **不可变**，事实基准 | 拉取脚本（LLM 只读） |
| **② Wiki** | LLM 生成的结构化 Markdown 页面 | 编译产物 `.o/a.out` | LLM 完全拥有，禁止人工乱改 | LLM Agent |
| **③ Schema** | 规则/约定文件（如 `CLAUDE.md`、系统提示词） | 语言规范 + 编译选项 | 定义结构与工作流 | 提示词本身 |

**为什么 Schema 层"最关键"？** 因为同样一个大模型，喂不喂这份规则文件，行为天差地别。没有它，LLM 只是个会瞎聊的助手，产出的笔记杂乱无章、格式漂移、引用混乱；有了它，LLM 才变成一个**守纪律的编译器**——知道该建哪几类页面、每类页面的元数据长什么样、如何交叉引用、遇到矛盾怎么标注。

> **Agent 的自我定位（系统提示词原文示例）**：
> "你是『raw→LLM Wiki 编译器』，一个有纪律的 wiki 维护者，不是普通聊天机器人。你完全拥有并维护 docs/ 知识库层；人类只负责选材、提问、把方向，繁琐的摘要、交叉引用、归档、记账全部由你完成。"

---

## 五、六种页面类型（产物的类型系统）

编译产物不是一堆自由格式的笔记，而是有严格"类型系统"的结构化页面：

| type | 目录 | 产出什么 | 命名规则 | 由哪种操作产出 |
|------|------|---------|---------|--------------|
| `source` | `sources/` | 每篇已摄入 raw 的摘要（与源 1:1 对应） | `summary-{slug}.md` | ingest |
| `entity` | `entities/` | 人/团队/平台/服务/接口/工具/库表/组件 | `{name}.md` | ingest |
| `concept` | `concepts/` | 业务知识/数据链路/方法/术语/机制 | `{name}.md` | ingest |
| `comparison` | `comparisons/` | 若干对象的横向对比 | `{slug}.md` | query 归档 |
| `synthesis` | `syntheses/` | 围绕一个主题的纵深综述 | `{slug}.md` | query 归档 |

此外还有**系统页面**（非内容页，随每次操作更新）：

- `index.md` —— 主目录，按主题分组（非扁平堆叠）
- `overview.md` —— 高层综述与主题导航
- `sources/index.md` —— 用户可见的源文档清单
- `log.md` —— append-only 活动日志（仅本地，通常不发布）

---

## 六、Frontmatter：产物的机器可读元数据

每个产物页面头部都有 YAML frontmatter，这是知识图谱的"符号表"。以 **concept 概念页**为例：

```yaml
---
type: concept
title: "概念名"
aliases: [别名, 缩写]           # 供 query 命中
sources: [sources/summary-x.md] # 溯源：这页知识来自哪些源摘要
related: [entities/entity1.md]  # 交叉引用（≥2 条）
created: 2026-07-15
updated: 2026-08-03
confidence: high | medium | low # 置信度
cluster: "{主题分组}"           # 与 index.md 分组一致
contradictions: []              # 与其它源的冲突（不自动选边）
open_questions: []              # 未解决的缺口
---
```

不同类型的关键差异字段：

- **source 页**：额外有 `source_id` / `source_type` / `source_version` / `key_claims`
- **entity 页**：额外有 `entity_type`（person/team/platform/service/api/tool/table/component）
- **comparison / synthesis 页**：有 `filed_from_query: true`，`related` 必须回链概念页

---

## 七、三大操作：知识库如何"活"起来

如果说三层架构是静态骨架，那么让知识库真正运转、生长的，是三个核心操作。它们恰好对应软件构建里的三个动作：

![三大操作与复利飞轮](assets/03-three-operations.svg)

*图 3：INGEST / QUERY / LINT 三大操作。QUERY 中产生的好答案会被归档回 Wiki，形成"越用越聪明"的复利飞轮（曲线为示意）。*

### INGEST（摄取）—— 最重要的操作

当一个新来源进入 `raw/`，LLM 会通读它，然后**提炼要点、抽取实体与概念、建立与已有页面的交叉引用**。

- 一次摄取通常波及 **10–15 个页面**：不只是新建一篇摘要，还要回头更新相关概念页、补上双向链接。
- 每个源提炼 **3–5 条**关键要点（key_claims）。
- 这正是"编译"的分量所在——它做的是碎片化 chunk 永远做不到的**知识整合**。

**ingest 流程（逐步）**：

1. 读取候选源对应的 raw 文件原文
2. 通读原文，提炼 3–5 条关键要点，不臆造
3. 产出/更新 `sources/summary-{slug}.md`（与源 1:1 对应）
4. 抽取实体与概念，创建或更新 `entities/*.md`、`concepts/*.md`
5. 建立交叉引用（新页面 ≥2 条入链），形成知识网络
6. 标注矛盾（若与已有页面冲突，显式并列，不覆盖）
7. 归类 cluster；出现新分组则登记到 index/overview
8. 更新系统页（index.md、sources/index.md、log.md）

### QUERY（查询）—— 会自我增值的问答

查询时，LLM 先读索引定位到相关页面，必要时再下钻到源摘要核对证据，然后带着引用作答。

**关键在于**：如果这次问答产生了有价值的新分析（比如一个横向对比），它可以被**归档回 wiki**，变成一篇新的对比页或综述页（`filed_from_query: true`）。于是知识库不是被动仓库，而是"越用越丰富"。

### LINT（健康检查）—— 给知识做体检

就像代码需要 linter，知识库也会"腐坏"。LINT 定期扫描：

| 检查项 | 标准 | 是否 autofix |
|--------|------|-------------|
| 断链 | 所有相对链接目标必须存在 | 仅报告 |
| 孤儿页 | 每页 ≥2 条入链（系统页除外） | 仅报告 |
| H1 缺失 | 正文首行必须是一级标题 | 仅报告 |
| cluster 缺失/不一致 | frontmatter 与 index 分组一致 | **自动修复**（只改 frontmatter） |
| 回链缺失 | 综述/对比引用的概念页需有回链 | 自动补回链 |
| 过时声明 | 已编译版本落后于源版本 → 标记 stale | 报告为待办 |
| 矛盾未标注 | 不同源冲突必须记录 | 不自动选边 |

有些实现还加入 **MERGE**（合并高度重叠的页面），进一步对抗熵增。

### 三条让人放心的纪律（硬约束）

- **① 忠实原文**——raw 里没有的不写，不确定就标低置信度
- **② 矛盾显式化**——遇到冲突不悄悄覆盖旧结论，而是并列标注、留待主编裁决
- **③ 永不删除**——过时页面标记为 `deprecated` 而非删掉，保留可追溯历史

---

## 八、输入 / 处理 / 输出 全景

![输入 / 处理 / 输出 全景](assets/04-io-overview.svg)

*图 4：四类输入喂给 LLM Agent 处理器，产出四类输出。处理是核心，输入决定"编译什么"，输出决定"产出什么形态"。*

**四类输入**：

| 输入 | 相当于编译器的 | 说明 |
|------|--------------|------|
| ① raw/ 原始资料 | 源代码 | 两类源：文档、代码。不可变，只读 |
| ② `_fetch_state.json` | 源文件时间戳 | 源的版本、内容哈希、拉取状态 |
| ③ 系统提示词 | 语言规范+编译选项 | Schema 层，最关键 |
| ④ 调用指令 | `make` 目标+选项 | mode + operation + input |

**调用指令**必须显式指定 mode，与 operation 正交组合：

```bash
mode=rebuild     operation=ingest                        # 全量重建
mode=incremental operation=ingest source_id=iwiki:xxxxx  # 增量摄入（日常默认）
mode=incremental operation=query "某字段怎么查？"          # 查询，可选 --archive
mode=incremental operation=lint                          # 健康检查，可选 --fix
```

|  | ingest 摄入 | query 查询 | lint 健康检查 |
|--|------------|-----------|--------------|
| **rebuild** 全量 | 枚举全部源逐个编译，末尾 Lint 收敛，重写 index/overview | — | rebuild 末尾一致性收敛 |
| **incremental** 增量（默认） | 只编译候选（stale）源，只动受影响页面 | 读库作答，有价值则归档 | 扫矛盾/孤儿页/断链，报告+可选修复 |

---

## 九、工程化落地：从个人玩具到生产系统

Karpathy 的原始版本是面向个人的：把文件拖进 `raw/`，在 Obsidian 里用 `[[双链]]` 浏览知识图谱。但当团队想把它变成**自动化生产系统**时，会遇到一系列现实问题，范式也在这些地方被工程化"加固"：

| 问题 | 个人版做法 | 工程化改造 |
|------|-----------|-----------|
| 源怎么进来 | 手动拖文件 | 写**自动拉取脚本**，定时从内部 wiki/代码仓库同步，记录每个源的版本 |
| 怎么知道哪些要重编 | 靠人/LLM 隐式判断 | 维护**两份状态账本**（已拉取 vs 已编译），精确比对"变脏"的源，只增量重编 |
| 交叉引用语法 | `[[wiki 双链]]` | 改用**标准 Markdown 链接** `[标题](path.md)`——因为要发布到不渲染双链的平台 |
| 怎么给别人看 | 本地 Obsidian | 写**发布脚本**推送到团队 wiki，解决"页面互链 ID 发布后才知道"的鸡生蛋问题（两遍推送） |
| 怎么调用 | 自然语言聊 | 结构化指令：**模式 × 操作**（rebuild/incremental × ingest/query/lint） |
| 矛盾追踪 | flagged for review | 引用块显式标注 + frontmatter `contradictions` 双记录 |

**一句话总结**：Karpathy 原始范式是面向**个人、手动、Obsidian 本地浏览**的轻量方案；工程化版本把它升级为面向**团队、自动拉取多源、结构化状态管理、自动发布**的生产系统。但核心思想（三层架构 / 编译而非检索 / ingest-query-lint / 复利积累）一脉相承。

---

## 十、增量编译：两份账本的解耦设计

最能体现"工程 vs 玩具"差异的，是**增量编译**——几乎是从 `make` 里原样搬来的智慧：只重新编译改动过的文件。

实现的关键，是让"拉取"和"编译"各记一本账，然后 join 对比：

![增量编译：两份账本 join 判定](assets/05-incremental-join.svg)

*图 5：拉取账本记录源的当前版本，编译账本记录已编译到的版本，两者 join 对比即可精确判定"哪些源变脏了"。*

```python
# 拉取账本记：这个源现在是什么版本
fetch_state[src] = { "version": 9, "hash": "sha256:04f3..." }

# 编译账本记：这个源已经编译到哪个版本
compile_state[src] = { "compiled_version": 7 }

# 判定：编译落后于拉取 → 这个源"脏"了，需要重编
if compile_state[src]["compiled_version"] < fetch_state[src]["version"]:
    mark_as_stale(src)   # 只重编它，其余页面纹丝不动
```

**增量候选判定的精确规则**（满足任一即为候选）：

- `_fetch_state.json` 里 `fetch.state ∈ {pending, changed, error}`
- `_compile_state.json` 里 `compile.state ∈ {never_compiled, stale, compile_error}`，或该 source_id 在编译账本中缺失

**stale 怎么算**：

- iWiki 类源用 `version` 整数比较（compiled_version < 当前 version → stale）
- 代码类源用 blob SHA **字符串等值**比较（不等即 stale）
- stale 是**运行时派生态，不单独存储**

> 🧱 **一个优雅的解耦**：拉取脚本完全不需要懂编译逻辑，编译器也完全不需要懂拉取逻辑，它们只通过两份账本"隔空对账"。"哪些源需要重编"这个状态甚至不用存储，每次运行时现算即可。这种单一职责的解耦，正是范式能从个人玩具长成生产系统的底气。

**源与产物的主键规范（source_id）**：

```
iwiki:{docid}                      # wiki 文档
gongfeng:{repo}:{ref}:{path}       # 代码单文件
gongfeng_dir:{repo}:{ref}:{path}   # 代码目录
```

---

## 十一、状态机

整个系统靠两份账本的状态流转驱动增量。

**fetch.state（输入侧，Agent 只读）**：

| 状态 | 含义 | 对编译的意义 |
|------|------|-------------|
| `pending` | 待拉取 | → 候选 |
| `fresh` | 已拉取、与上次一致 | 不一定候选，看 compile 侧 |
| `changed` | 源端已变更待重拉 | → 候选 |
| `fetching` | 拉取中 | 暂不处理 |
| `error` | 拉取失败 | → 候选（可用现有快照编译） |
| `removed` | 已移除 | 对应页面标 deprecated，不删 |

**compile.state（输出侧，Agent 独写）**：

| 状态 | 含义 | 对编译的意义 |
|------|------|-------------|
| `never_compiled` | 从未编译 | → 候选 |
| `stale` | 源版本已超过已编译版本 | → 候选 |
| `compiling` | 编译中 | 处理中 |
| `compiled` | 已编译到最新 | 跳过 |
| `compile_error` | 上次编译失败 | → 候选（重试） |
| `ignored` | 人为忽略 | 永不编译 |

---

## 十二、双提交工作流

**为什么必须两次提交**：编译账本需要记录"产出这批 wiki 的那次提交 hash"，但提交前无法知道自己的 hash。所以：**先提交产物拿到 hash（Commit A），再把 hash 回填进编译账本单独提交（Commit B）**。

```
1. 基础检查    ── 新增页有 H1；相对链接目标存在
2. Commit A    ── git add docs/ && commit  →  拿到 hash 如 abc123
3. Commit B    ── 把 abc123 写入 _compile_state.json 的 compiled_commit
                  git add raw/_compile_state.json && commit
4. 一次性 push ── A、B 都提交后再统一推送
```

> **不修改文件的操作不 commit**：lint 仅报告、query 仅回答、增量无候选时 → 直接说明"无需更新"并停止，不产生空提交。

这与下面发布环节的"两遍推送"同构——都是**"先落地拿标识，再回填引用"**。

---

## 十三、发布：两遍推送破解"鸡生蛋"

wiki 编译完成后，由发布脚本推送到团队平台。核心难点是**页面互链的 docid 解析**。

**鸡生蛋问题**：wiki 里 A 页链接到 B 页用的是相对路径 `../entities/b.md`，但推送到平台需要换成 `https://.../p/{B的docid}`。而 B 的 docid 只有推送后才知道。

**两遍推送**解决：

![两遍推送：破解页面互链的鸡生蛋](assets/06-two-pass-push.svg)

*图 6：第一遍先给所有页面建档拿到 docid、补全映射表；第二遍再把正文里的相对链接解析成平台 URL 后完整推送。*

- 映射表（如 `iwiki-mapping.json`）由发布脚本独写，记录 `docs 文件路径 → docid` 以及目录结构。
- frontmatter 通常会被平台自动剥离（正文干净），所以照常写不会污染发布内容。
- `log.md` 是"构建日志"，一般不发布，留在本地仓库。

---

## 十四、端到端实例：一个源的完整生命周期

跟随一个 wiki 源，从被添加到最终发布，把输入/处理/输出串起来：

1. **【输入】添加源**：运维执行 `fetch_raw.py --add 4028672179`。脚本拉取正文、加 frontmatter、算 content_hash，落盘为 `raw/xxx-4028672179.md`，在 `_fetch_state.json` 写入 `fetch.state=fresh, version=3`。
2. **【输入】触发编译**：调用 `mode=incremental operation=ingest source_id=iwiki:4028672179`。
3. **【处理】候选判定**：Agent join 两份账本，发现该 source_id 在编译账本中缺失（never_compiled）→ 判定为候选。
4. **【处理】编译**：通读原文 → 提炼 3–5 条要点 → 新建 `sources/summary-xxx.md`、若干 `concepts/*.md` 与 `entities/*.md` → 建立 ≥2 交叉引用 → 标注矛盾（若有）。
5. **【输出】写产物**：更新 index.md（注册新页+主题分组）、overview.md、sources/index.md、log.md（追加 ingest 记录）。
6. **【输出】Commit A**：`git add docs/ && git commit -m "docs: ingest xxx"`，得到 hash `abc123`。
7. **【输出】Commit B**：把 `abc123` 回填进 `_compile_state.json`（state=compiled, compiled_version=3, compiled_commit=abc123），单独提交后 push。
8. **【发布】sync**：CI 触发发布脚本，第一遍建档拿 docid、第二遍解析链接推送，平台上出现可搜索、可互跳的新文档。
9. **【再处理】下次源更新**：某天该文档被人改动，fetch 拉到 version=4。下次 ingest 时 join 账本发现 compiled_version(3) < version(4) → stale → 只重编这一个源的相关页面（增量），其余纹丝不动。

---

## 十五、从零复现的最小清单

复现这套"知识编译器"门槛低得惊人——不需要向量库，一个目录 + 一份提示词就能起步。

### 目录骨架

```
my-wiki/
├── raw/                 # 放原始来源（只读）
│   └── _fetch_state.json   # 拉取账本 {"schema_version":3,"docs":[]}
├── wiki/  (或 docs/)     # LLM 编译出的页面
│   ├── sources/            # 每个源一篇摘要
│   ├── entities/           # 实体页
│   ├── concepts/           # 概念页
│   ├── comparisons/        # 对比页
│   ├── syntheses/          # 综述页
│   ├── index.md            # 主目录
│   └── log.md              # 活动日志
├── _compile_state.json  # 编译账本 {"schema_version":3,"docs":[]}
└── CLAUDE.md            # ★ Schema 层：最关键的文件
```

### Schema 层（CLAUDE.md）核心规则节选

```markdown
- 你是"raw→wiki 编译器"，只读 raw/，完全拥有 wiki/
- 每类页面的 frontmatter 必须包含：type / sources / related / confidence
- 每个新页面至少交叉引用 2 个已有页面
- 只用标准 markdown 链接 [标题](path.md)，绝不用 [[双链]]
- 正文第一行必须是 H1 标题
- 遇到矛盾并列标注，不覆盖；过时页面标 deprecated，不删除
- 每次改 wiki 都更新 index.md 与 log.md
- 忠实原文，raw 没有的不写；不确定设 confidence: low
- 全文不使用 emoji；专有名词保留英文原名
```

### 复现步骤

1. **【输入】搭输入管道**：写拉取脚本，把外部源拉成"带 source_id 的标准化 raw 文件 + 版本账本"
2. **【输入】准备空产物骨架**：建 wiki/ 目录树 + 初始两份账本
3. **【处理】装配处理器（最关键）**：把系统提示词全文设为 Agent 的 system prompt
4. **【处理】首次全量编译**：调用 `mode=rebuild operation=ingest`
5. **【输出】验证产物**：检查双提交闭环、log.md 记录、lint 报告（断链/孤儿页为 0）
6. **【发布】接发布管道**：配置映射表，CI 里跑发布脚本（首次全量）
7. **【处理】建立增量循环**：定时 fetch 检测更新 → 有 stale 就增量 ingest → 定期 lint 保健康

### 复现避坑（机制层面）

| 陷阱 | 正确做法 |
|------|---------|
| 让 Agent 直接改 raw 或拉取账本 | 禁止。输入不可变，编译器无权改输入 |
| 不给系统提示词就让 LLM 编译 | 产物会杂乱无章。提示词是编译器的"语言规范" |
| 用 Obsidian 双链 `[[..]]` | 多数平台不渲染。只用标准 markdown 链接 |
| 页面无 H1 首行 | 平台标题会退化成文件名。首行必须 `# 标题` |
| 新页面不建交叉引用 | 会变孤儿页。每页 ≥2 入链 |
| 只提交 docs 不回填 compiled_commit | 下次增量判定会错乱。必须双提交 |
| 发布只跑一遍 | 页面互链全失效。必须两遍（建档 → 解析链接） |
| 矛盾时直接覆盖旧结论 | 用引用块 + contradictions 显式标注，不选边 |

---

## 十六、冷静一点：它不是银弹

LLM Wiki 很优雅，但把它当成"RAG 终结者"是危险的。它的甜蜜区其实相当明确：

| ✅ 适合 LLM Wiki | ⚠️ 仍应用 RAG / 混合架构 |
|-----------------|------------------------|
| 精选、稳定、高价值的知识（百量级源） | 海量、远超上下文窗口的语料 |
| 个人研究库、团队核心知识、Agent 长期记忆 | 高度动态的实时数据（行情、新闻流） |
| 希望零基础设施、可 Git 版本化 | 需要毫秒级检索的高并发场景 |

更诚实地说，它还有几个尚未被充分验证的问题：

- **编译本身要消耗不菲的 LLM 算力**——成本从"查询时"挪到了"摄取时"，并没凭空消失。
- **当源数量涨到成千上万，wiki 自身也会大到装不进上下文**，届时你反而需要——没错——给 wiki 再套一层检索。

所以更可能的终局不是"编译取代检索"，而是**二者分层协作**：用编译沉淀高价值的稳定知识，用检索兜底海量的长尾与实时信息。

> 与其争论"编译还是检索"，不如问"**这块知识值不值得被编译**"。

抛开工程细节，LLM Wiki 真正动人的地方，是它悄悄改变了我们与知识的关系。在 RAG 的世界里，知识是"用完即弃的一次性检索结果"；而在 wiki 的世界里，**每一次阅读、每一次提问，都在为一座会永久留存、还会自我生长的知识大厦添砖加瓦。**

Bush 在 1945 年畅想 Memex 时，缺的是那个不知疲倦的管理员。八十年后，这个管理员终于到岗了。剩下的问题只是：你想让它帮你编译哪座知识大厦？

---

## 参考资料与事实核查说明

**原始出处**

1. Andrej Karpathy 关于 LLM Wiki 的推文与 GitHub Gist（2026 年 4 月初发布，社区广泛转载）。Karpathy 身份：OpenAI 联合创始人、前特斯拉 AI 高级总监、"vibe coding"一词提出者。
   - 官方发布账号（原始推文 / Gist 请以其官方发布为准）：<https://x.com/karpathy>

2. 思想渊源
   - Vannevar Bush,《As We May Think》, The Atlantic, 1945（Memex 构想）：<https://www.theatlantic.com/magazine/archive/1945/07/as-we-may-think/303881/>
   - Niklas Luhmann 的 Zettelkasten 卡片盒笔记法（社区资料站）：<https://zettelkasten.de/>

**社区解析文章**（均 2026 年 4 月前后发布）

3. Karpathy's LLM Wiki: The Complete Guide to His Idea File（完整指南，本文主要参考）：<https://agentpedia.codes/zh/blog/karpathy-llm-wiki-idea-file>
4. 掘金《8 万人收藏：Karpathy 的 LLM Wiki 到底是什么？完整拆解》：<https://juejin.cn/post/7625301482213130291>
5. 深入解析《Karpathy 的 LLM Wiki：从"检索信息"到"编译知识"》：<https://xdlkc.github.io/2026/04/11/karpathy-llm-wiki-deep-dive/index.html>
6. abmedia《Karpathy 亲揭：用 LLM 打造个人知识库的完整方法》：<https://abmedia.io/karpathy-llm-knowledge-base-personal-wiki-obsidian-method-2026>

**事实核查说明**：

- Karpathy 的身份、原始类比（"Obsidian 是 IDE / LLM 是程序员 / Wiki 是代码库"、"LLM 是作者，人是主编"）、三层架构（Raw / Wiki / Schema）、三大操作（INGEST / QUERY / LINT）、思想渊源（Memex、Zettelkasten）均来自上述公开资料并交叉印证。
- 原始 Gist 的确切 URL（gist ID）各二手来源未统一暴露，故此处仅提供 Karpathy 官方账号入口，**请以其官方发布为准**，避免引用未经核实的镜像链接。
- 所有量化性能数字（如"Token 减少约 95%"、"比 RAG 高效约 70 倍"、"8.8 万收藏""1500 万浏览"）来自二手报道与社区估算，**非严格基准测试**，请谨慎引用。
- 第九至十四节的工程实现细节，来自一套真实的企业级 LLM Wiki 系统，已做**通用化脱敏处理**，不指向任何特定内部系统的敏感信息。
