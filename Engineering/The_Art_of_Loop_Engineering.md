# The Art of Loop Engineering — LangChain 四层循环架构

## Part 1: 总结

> 来源：LangChain 官方博客 *The Art of Loop Engineering* — Sydney Runkle, 2026-06-16

### 核心观点

Agent 的本质是一个模型在循环中调用工具直到任务完成。但要让 Agent 可靠地完成有价值的工作，仅靠一个好模型是不够的，还需要一个精心设计的 **harness（框架）**。核心思想是：**通过堆叠和扩展多层循环来构建更高效的 Agent**。

### 四层循环架构

文章提出了四层循环，层层叠加，让 Agent 从"能做事"进化到"能做好事、主动做事、自我优化"：

| 层级 | 名称 | 核心作用 | 解决的核心问题 |
| :---: | :--- | :--- | :--- |
| **Loop 1** | **Agent 循环** | 模型调用工具完成任务 | "能不能做"——从回答者变为执行者 |
| **Loop 2** | **验证循环** | 评分器检查质量，不合格打回重做 | "能不能做对"——保障一致性和正确性 |
| **Loop 3** | **事件驱动循环** | 通过 Webhook/定时任务自动触发 | "能不能主动做"——融入业务生态 |
| **Loop 4** | **爬山循环** | 利用生产追踪数据自动优化配置 | "能不能越做越好"——实现自我进化 |

### Loop 1：Agent 循环

从本质上讲，Agent 只是一个模型在循环中调用工具直到任务完成。这就是 LangChain 的 `create_agent` 所提供的功能。选择任意模型，接入工具，即可获得一个可运行的 Agent 循环。工具赋予 Agent 在现实世界中采取行动的能力。

> 以 LangChain 内部文档 Agent 为例：在第一层循环中，它收到一个文档改进请求，模型进行规划并草拟更改，然后使用工具克隆代码仓库、读取文件、编写文档、发起拉取请求等。

### Loop 2：验证循环

Agent 循环能够完成工作，但并非总能在第一次运行中就产生正确或一致的结果。当一致性至关重要时，需要将其封装在一个**验证循环**中，该循环会检查输出并在结果不理想时向模型发送反馈。

验证循环增加了一个**评分器（Grader）**：它会根据评估标准检查 Agent 的输出，如果不达标，就将结果连同反馈一起打回。评分器可以是确定性的，也可以是 Agentic（LLM 作为裁判是典型例子）。

LangChain 实现方式：`RubricMiddleware` 或 `create_agent` 上的 `after_agent` 钩子。

> 以文档编写器为例：评分器在每次尝试后运行测试，检查所有链接是否可访问、所有 CI 检查是否通过，以及代码变更范围是否与请求一致。无需人工审查即可捕获这些类型的错误。

**权衡**：增加验证会增加每次运行的延迟和成本。当质量比速度更重要时——这适用于大多数生产用例——这种取舍是值得的。

### Loop 3：事件驱动循环

Agent 开发中最重要的部分之一是**集成层**：将 Agent 连接到生态系统，使其能够在后台运行。事件驱动循环将 Agent 连接到生态系统——当一个事件触发时（新文档到达、定时任务触发、Webhook 到达），Agent 就会运行。

Agent 不是手动调用的；它是作为组件在更大的系统中持续运行。

LangSmith Deployment 支持触发基础设施，包括定时任务（cron）和 Webhook。一个典型的应用示例是 openclaw 中的"心跳"机制，它将 Agent 变成始终在线的主动型助手。LangChain 的文档 Agent 由 Fleet（无代码 Agent 构建器）驱动，Fleet 的频道和调度功能处理事件驱动和定时任务式触发——每当在 #docs-plz Slack 频道中发送消息时，就会触发文档 Agent。

### Loop 4：爬山循环

前三层循环实现了工作自动化，第四层循环（也是最重要的）实现了**改进自动化**。每次 Agent 运行都会产生一条 **trace**（追踪记录）：记录模型做了什么、调用了哪些工具、评分器的反馈等。这些 trace 包含了关于哪些有效、哪些无效的高价值信号。爬山循环运行一个**分析 Agent** 来审查这些 trace，并利用发现的问题来重写 harness 配置，包括提示词/工具调整或评分器调整。

在 LangSmith 中，可以使用 **Engine**（trace 分析 Agent）来实现第四层循环。以文档 Agent 为例：Engine 审查文档 Agent 的 trace 来检测问题，当多条 trace 指示潜在问题时，会自动提交 issue 请求修改相关的提示词或工具。

关键在于：**返回箭头不只是循环回到顶部，而是深入内部直接更新 Agent 循环**。外层循环的每次迭代都会让内层循环变得更有效。

**展望**：提示词和工具配置是最简单的改进目标，但不是唯一选项。对于运行开源权重模型的团队，爬山循环可以接入 **RL 微调**，使用 trace 或评估结果作为训练信号来改进模型本身。辅助上下文（如记忆和检索到的技能）也可以用同样的方式改进。**循环是模式，优化什么由你决定。**

### 人工监督与专业知识

自动化并不意味着将人类从循环中移除。在每一层中，都有人工监督能增加价值的自然节点。自动化评分器可以检查链接是否可访问；但只有人类才能发现内容的定位对目标受众不合适。这种基于语境、经验和品味的判断力，正是人工审查发挥作用的地方。

有些专业知识应固化在提示词和工具本身中，但对于敏感操作，实时人工审查至关重要（如金融交易、数据库操作等）。LangChain 使在每一层循环中添加这些接触点变得简单：

- **Agent 循环中**：在敏感操作/工具调用前要求人工输入
- **验证循环中**：人工可以作为敏感工作流的评分器
- **应用循环中**：人工可以在输出返回给最终用户前进行审批
- **爬山循环中**：harness 改进可以经过人工审查后再部署

LangChain 的所有开源框架都将"human in the loop"（人在回路中）作为一等公民原语。

### 四层循环汇总

| 循环 | 功能 | 影响 | LangChain 原语 |
| :--- | :--- | :--- | :--- |
| **1: Agent 循环**（模型 + 工具） | 模型反复调用工具直到任务完成 | 工作自动化 | `create_agent`、任意 LangChain 支持的模型 |
| **2: 验证循环**（Agent + 评分器） | Agent 运行，输出按评分标准打分，失败则带反馈重试 | 保障质量 | `RubricMiddleware` |
| **3: 事件循环**（验证 + 系统） | 事件触发 Agent 运行并更新真实系统 | 规模化运作 | LangSmith Deployment / Fleet 频道 |
| **4: 爬山循环**（系统 + Engine） | 生产 trace 喂入分析 Agent 改进 harness 配置 | 持续改进 | LangSmith Engine |

这就是循环工程（或如 swyx 所说的 loopcraft）在实践中的样子。AI 领袖如 Steipete、Boris 和 Andrej 都得出了相同的结论：**Agent 的潜力在于你围绕它们构建的循环**。

我们一直在思考循环 1 和 2。但重心应转向循环 3 和 4——通过将 Agent 嵌入生态系统并持续根据你的标准进行改进，价值会不断复合。Satya 指出了组织层面的关键：**那些尽早构建学习循环——让人类判断力和 token 资本共同复合——的公司将建立难以复制的优势。**

### 结语

LangSmith 是 LangChain 的 Agent 工程平台，帮助开发者调试每一个 Agent 决策、评估变更，并一键部署。

### 与 Osmani 框架的对应关系

| Osmani 六大部件 | LangChain 四层循环 |
| :--- | :--- |
| 自动化（心跳） | Loop 3 事件驱动循环 |
| sub-agent（写查分离） | Loop 2 验证循环 |
| connector（外部集成） | Loop 3 事件驱动循环 |
| skill（项目规范） | 贯穿所有层级 |
| worktree（隔离工作目录） | 贯穿所有层级 |
| 记忆（磁盘状态） | Loop 4 爬山循环的数据基础 |

> **总结**：Osmani 的框架更侧重工程实践（怎么搭），LangChain 的框架更侧重能力演进（怎么进化）。两者互补——用 Osmani 的部件搭骨架，用 LangChain 的层级规划成长路径。

---

**标签**：`engineering` · `loop` · `langchain` · `agent`

- Loop Engineering — Osmani 的 Loop Engineering 六大部件框架
- Harness 工程化 — 流程工程化实践
- AI Agent & Skill 测评方案 — Agent 评测框架

---

## Part 2: 中英对照

Agents are useful because they help us automate work by taking actions in the real world. But getting agents to do valuable work reliably takes more than just a good model: it requires a carefully designed harness that's fit to a set of tasks.

Agent 的价值在于帮助我们通过在现实世界中执行操作来自动化工作。但要让 Agent 可靠地完成有价值的 work，仅仅依靠一个好的模型是不够的：它需要一个经过精心设计的、适合特定任务集的 harness（运行框架）。

The core agent algorithm is simple: give the LLM context and let it call tools in a loop until it's done. This is the most fundamental loop. But it's far from the only loop that powers agents. Swyx recently wrote a great piece on "loopcraft: the art of stacking loops", the idea that you can stack and extend loops to build more effective agents. Here's how we think about that stack, and how to instrument each level with LangChain primitives.

核心的 Agent 算法很简单：给 LLM 提供上下文，让它在循环中调用工具直到完成任务。这是最基础的循环，但远非驱动 Agent 的唯一循环。Swyx 最近写了一篇很棒的文章《loopcraft: the art of stacking loops》（循环工艺：堆叠循环的艺术），提出了可以通过堆叠和扩展循环来构建更有效 Agent 的理念。以下是我们如何看待这个堆叠，以及如何用 LangChain 原语来实现每一层。

### Loop 1: The Agent

At its core, an agent is just a model calling tools in a loop until a task is complete. This is what LangChain's `create_agent` gives you. Pick any model, plug in tools, and you have a working agent loop.

从本质上讲，Agent 只是一个在循环中调用工具直到任务完成的模型。这就是 LangChain 的 `create_agent` 提供的功能。选择任意模型，接入工具，你就有了一个可工作的 Agent 循环。

Tools are what give the agent the power to take action in the real world. Take our internal docs agent as an example (which we'll use as a motivating example for the rest of this blog). At the first loop level, it receives a request for a documentation improvement, the model plans and draft changes, and it uses tools to clone repos, read files, write docs, open a pull request, etc.

工具赋予了 Agent 在现实世界中采取行动的能力。以我们的内部文档 Agent 为例（在本文后续部分我们将以此作为示例）。在第一层循环中，它接收文档改进请求，模型进行规划和起草更改，并使用工具克隆仓库、读取文件、编写文档、提交 Pull Request 等。

### Level 2: Verification Loop

The agent loop gets work done, but it doesn't always produce correct or consistent work on the first pass. When consistency matters, it's often useful to wrap it in a verification loop that checks the output and sends feedback back to the model when it falls short.

Agent 循环可以完成工作，但并不总是在第一次就产出正确或一致的结果。当一致性很重要时，将其包裹在一个验证循环中通常很有用——该循环检查输出，并在不达标时将反馈发送回模型。

The verification loop adds a grader: something that checks the agent's output against a rubric and, if it fails, sends the result back with feedback. Graders can either be deterministic or agentic (LLM as a judge is a classic example, here). `RubricMiddleware` handles this pattern, or you can wire it up with an `after_agent` hook on `create_agent`.

验证循环添加了一个 grader（评分器）：它根据评分标准检查 Agent 的输出，如果失败，则将结果连同反馈一起发回。评分器可以是确定性的，也可以是基于 Agent 的（LLM as a judge 是一个经典例子）。`RubricMiddleware` 处理这种模式，或者你可以在 `create_agent` 上使用 `after_agent` 钩子来实现。

For our docs writer example, the grader runs tests after each attempt, checking that all links resolve, all CI checks pass, and the diff is scoped to what was actually requested. No manual review needed to catch those classes of error.

以我们的文档编写器为例，评分器在每次尝试后运行测试，检查所有链接是否有效、所有 CI 检查是否通过、以及 diff 是否限定在实际请求的范围内。无需人工审查即可捕获这类错误。

One tradeoff: adding verification increases latency and cost per run. It's worth it when quality matters more than speed, which is most production use cases.

一个权衡是：添加验证会增加每次运行的延迟和成本。当质量比速度更重要时（大多数生产场景都是如此），这是值得的。

### Level 3: Event Driven Loop

One of the most important parts of agent development is the integrations layer: connecting your agent to your ecosystem so that it can run in the background. The event-driven loop connects your agent to your ecosystem. An event fires — a new document lands, a schedule triggers, a webhook arrives — and the agent runs. The agent isn't something you invoke manually; it's a component running continuously inside a larger system.

Agent 开发最重要的部分之一是集成层：将你的 Agent 连接到生态系统，使其能够在后台运行。事件驱动循环将你的 Agent 与生态系统连接起来。当事件触发——新文档落地、定时任务触发、Webhook 到达——Agent 就会运行。Agent 不是你手动调用的东西；它是一个在更大系统中持续运行的组件。

LangSmith Deployment supports the trigger infrastructure, including support for cron schedules and webhooks. One popular example of crons in action is "heartbeats" in openclaw, which turn your agent into an always-on, proactive assistant.

LangSmith Deployment 支持触发基础设施，包括 cron 定时任务和 webhook。cron 的一个流行应用示例是 openclaw 中的"心跳"机制，它将你的 Agent 变成一个始终在线的主动型助手。

Our docs agent is powered by Fleet, our no-code agent builder. Fleet's channels and schedules handle event-driven and cron-style triggers. We use a channel to fire off the docs agent whenever a message is sent in our `#docs-plz` Slack channel.

我们的文档 Agent 由 Fleet（我们的无代码 Agent 构建器）驱动。Fleet 的频道和调度功能处理事件驱动和 cron 风格的触发器。我们使用一个频道，每当在 `#docs-plz` Slack 频道发送消息时，就会触发文档 Agent。

### Level 4: Hill Climbing Loop

The first three loops automate work. The fourth (and arguably most important) automates improvement! Every agent run produces a trace: a record of what the model did, the tools it called, grader feedback, etc. Those traces contain high-value signal regarding what's working and what isn't. The hill climbing loop runs an analysis agent over those traces and uses the findings to rewrite the harness with improved configuration. That can include prompt/tool tweaks or grader tweaks.

前三层循环自动化工作。第四层（可以说最重要的）自动化改进！每次 Agent 运行都会产生一个 trace（追踪记录）：记录模型做了什么、调用了哪些工具、评分器反馈等。这些 trace 包含关于哪些有效、哪些无效的高价值信号。迭代优化循环在这些 trace 上运行分析 Agent，并使用发现来重写 harness 的配置。这可能包括 prompt/工具调整或评分器调整。

In LangSmith, you can use Engine, our trace analysis agent, to instrument this fourth loop. Wrapping up the docs agent analogy, we run Engine over the docs agent traces to detect any issues. When multiple traces signal a potential problem, an issue is filed requesting changes to the offending prompt or tool.

在 LangSmith 中，你可以使用 Engine（我们的 trace 分析 Agent）来实现这第四层循环。总结文档 Agent 的类比，我们在文档 Agent 的 trace 上运行 Engine 来检测任何问题。当多个 trace 表明存在潜在问题时，会提交一个 issue 请求更改有问题的 prompt 或工具。

The key move here is that the return arrow doesn't just loop back to the top — it reaches inside and updates the agent loop directly. Each cycle of the outer loop makes the inner loops more effective.

这里的关键点是，返回箭头不只是循环回到顶部——它深入内部直接更新 Agent 循环。外层循环的每次迭代都使内层循环更加有效。

Looking forward: prompt and tool configuration are the most simple things to improve, but they're not the only options. For teams running open-weight models, the hill climbing loop can feed into RL fine-tuning, using trace or eval outcomes as training signal to improve the model itself. Auxiliary context like memory and retrieved skills can be improved the same way. The loop is the pattern; what it optimizes is up to you.

展望未来：prompt 和工具配置是最简单的改进对象，但并非唯一选择。对于运行开源权重模型的团队，迭代优化循环可以接入 RL 微调，使用 trace 或评估结果作为训练信号来改进模型本身。辅助上下文（如记忆和检索到的技能）也可以用同样的方式改进。循环是模式；它优化什么取决于你。

### Human Oversight and Expertise

Automation doesn't mean removing humans from the loop. At every level, there are natural points where human oversight adds value. An automated grader can check whether links resolve; it takes a human to notice the framing is wrong for the audience. That kind of judgment, earned from context, experience, and taste, is exactly where human review earns its place.

自动化并不意味着将人类从循环中移除。在每一层，都有人类监督能增加价值的自然节点。自动化的评分器可以检查链接是否有效；但只有人类才能注意到内容的框架不适合目标受众。这种来自上下文、经验和品味的判断，正是人类审查发挥作用的地方。

Some expertise should be codified in the prompt/tools themselves, but for sensitive actions, live human review is essential (think financial transactions, DB operations, etc). LangChain makes it straightforward to instrument these touch points in every loop:

某些专业知识应该被编码到 prompt/工具中，但对于敏感操作，实时的人类审查是必不可少的（例如金融交易、数据库操作等）。LangChain 使在每一层循环中实现这些接触点变得简单：

- **In the agent loop**, require human input before sensitive actions/tool calls — **在 Agent 循环中**，在敏感操作/工具调用前要求人类输入
- **In the verification loop**, a human can act as the grader for sensitive workflows — **在验证循环中**，人类可以作为敏感工作流的评分器
- **In the application loop**, a human can approve outputs before they're returned to the end user — **在应用循环中**，人类可以在输出返回给最终用户之前进行审批
- **In the hill climbing loop**, harness improvements can flow through human review before deployment — **在迭代优化循环中**，harness 的改进可以在部署前经过人类审查

All of LangChain's open source frameworks make adding a "human in the loop" a first class primitive.

LangChain 的所有开源框架都将"human in the loop"（人在回路中）作为一等公民原语。

### Putting It All Together

| Loop | What it does | Impact | LangChain primitive |
|------|--------------|--------|---------------------|
| **1: Agent loop** (model + tools) | Model calls tools repeatedly until a task is complete | Automate work | `create_agent`, any LangChain-supported model |
| **2: Verification loop** (agent + grader) | Agent runs, output is scored against a rubric, retried with feedback if it fails | Ensure quality | `RubricMiddleware` |
| **3: Event loop** (verification + system) | Events trigger agent runs that update a real system | Work at scale | LangSmith Deployment / Fleet channels |
| **4: Hill climbing loop** (system + engine) | Production traces feed an analysis agent that improves the harness config | Continuous improvement | LangSmith Engine |

| 循环层 | 功能 | 影响 | LangChain 原语 |
|--------|------|------|----------------|
| **1: Agent 循环**（模型 + 工具） | 模型反复调用工具直到任务完成 | 自动化工作 | `create_agent`，任意 LangChain 支持的模型 |
| **2: 验证循环**（Agent + 评分器） | Agent 运行，输出根据评分标准打分，失败时带反馈重试 | 确保质量 | `RubricMiddleware` |
| **3: 事件循环**（验证 + 系统） | 事件触发 Agent 运行，更新真实系统 | 规模化工作 | LangSmith Deployment / Fleet 频道 |
| **4: 迭代优化循环**（系统 + 引擎） | 生产 trace 喂给分析 Agent，改进 harness 配置 | 持续改进 | LangSmith Engine |

This is what loop engineering — or loopcraft, as swyx puts it — actually looks like in practice. AI leaders like Steipete, Boris, and Andrej have all arrived at the same conclusion: the potential in agents is in the loops you build around them.

这就是循环工程——或者如 swyx 所说的 loopcraft（循环工艺）——在实践中的样子。像 Steipete、Boris 和 Andrej 这样的 AI 领袖都得出了相同的结论：Agent 的潜力在于你围绕它们构建的循环。

We've been thinking about loops 1 and 2 for a while. But focus should pivot to loops 3 and 4 where value compounds by embedding agents into your ecosystem that continuously improve in response to your criteria. Satya frames the organizational stakes: companies that build learning loops early, where human judgment and token capital compound together, will build an advantage that's hard to replicate.

我们已经在思考第 1 和第 2 层循环有一段时间了。但重点应该转向第 3 和第 4 层循环——通过将 Agent 嵌入你的生态系统，使其根据你的标准持续改进，从而实现价值的复利增长。Satya 阐述了组织层面的利害关系：那些尽早构建学习循环的公司——人类判断力和 token 资本共同复利增长——将建立起难以复制的优势。

### Acknowledgements

*See what your agent is really doing.* LangSmith, our agent engineering platform, helps developers debug every agent decision, eval changes, and deploy in one click.

*看看你的 Agent 到底在做什么。* LangSmith——我们的 Agent 工程平台——帮助开发者调试每一个 Agent 决策、评估更改，并一键部署。
