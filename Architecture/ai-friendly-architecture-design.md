# AI 友好架构设计：从决策到落地的完整指南

> 本文整理自 [ai-friendly-architecture-design-skill](https://github.com/GanJiaKouN16/ai-friendly-architecture-design-skill) 项目，一个经过 227 个测试场景验证（98%+ 准确率）的 AI 架构决策引擎。

---

## 一、核心理念

**使用合适的架构解决问题——当传统方案足够时，不要过度使用 AI。**

三个范式转变：

| 转变 | 说明 |
|------|------|
| 确定性 → 概率性 | 接受 AI 输出的不确定性，设计容错机制 |
| 结构化 → 语义化 | 意图优于格式，理解用户想做什么而非严格匹配输入 |
| 静态 → 动态 | 规划优于规则，让 AI 能根据上下文调整行为 |

---

## 二、何时使用 AI

### 适用场景

- 用户询问"是否应该为 X 使用 AI/Agent"
- 设计供 AI 消费的 API
- 选择 Agent 类型（ReAct / Plan / Multi-Agent）
- 评估 AI vs 传统架构
- 选择 MCP vs Function Call
- 选择 Workflow vs Agent
- AI 组件选型（Prompt / Skills / MCP / A2A / CLI）

### 不适用场景

- 无 AI 需求的简单 CRUD
- 纯确定性、基于规则的逻辑
- 实现细节问题（如何编码 X）

---

## 三、Q1-Q5 决策框架

这是整个架构设计的核心——通过五个递进问题，逐步缩小选择范围，输出结构化的架构推荐。

### Q1：是否需要 AI？

```
任务是否确定性的？
├─ 纯确定性 → 传统架构（MVC/DDD）
├─ 混合任务 → 混合架构
│   ├─ 确定性子任务 → 规则/验证（快速路径）
│   └─ 概率性子任务 → AI 模型（慢速路径）
│   └─ → Q2: 检查 AI 子任务的约束
└─ 需要语言理解 → Q2
```

**需要语言理解（→ Q2）的条件（满足任一即可）：**

- 自然语言输入（用户查询、聊天、文档）
- 意图识别（理解用户想做什么）
- 多轮对话（跨对话上下文）
- 语义搜索（基于含义的检索）
- 内容生成（文本、摘要、翻译）
- 情感分析（理解语气/情绪）

**不需要语言理解（→ 规则/ML）的条件：**

- 结构化数据处理（CSV、JSON 转换）
- 模式匹配（正则、关键词匹配）
- 特征明确的分类（基于规则的欺诈检测）
- 图像/信号处理（CNN、传统 CV）

**混合任务示例：**
- 订单处理：格式验证（确定性）+ 欺诈检测（概率性）
- 内容审核：关键词过滤（确定性）+ 上下文分析（概率性）
- 客服系统：路由（确定性）+ 回复生成（概率性）

### Q2：什么约束？

```
检查约束（硬约束 > 软约束优先级）：
├─ 延迟 <100ms（硬约束）
│   └─ 混合：AI 离线 + 规则在线，缓存 → Q3
├─ 严格成本约束（软约束）
│   └─ 混合：AI + 规则，小模型，缓存 → Q3
├─ 数据隐私要求（硬约束）
│   └─ 本地模型，数据匿名化 → Q3
├─ 多约束冲突？
│   └─ 参见约束冲突优先级矩阵 → Q3
└─ 无硬约束 → Q3
```

**约束冲突优先级矩阵：**

| 冲突 | 解决方案 | 示例 |
|------|----------|------|
| 性能 + 成本 | 性能优先（硬 > 软） | 缓存 + 小模型，保持 <100ms SLA |
| 性能 + 隐私 | 均为硬约束，找折中 | 本地模型 + 缓存，<100ms |
| 成本 + 隐私 | 隐私优先（硬 > 软） | 本地模型，简单任务用规则 |
| 准确性 + 成本 | 定义最低准确性阈值，优化成本 | 95% 准确率要求 → 混合架构 |
| 速度 + 质量 | 异步处理 | FAQ 快速回复，复杂投诉用 AI |

**决策公式：**
1. 列出所有约束
2. 分类硬约束 vs 软约束
3. 硬约束冲突 → 找折中方案
4. 仅软约束 → 优化业务影响
5. 硬 + 软 → 硬约束优先

### Q3：什么工作模式？

```
需要动态工具选择吗？
├─ 不需要 → Q3.1: 单次调用还是工作流？
└─ 需要 → Q3.2: 工作模式选择
```

#### Q3.1：单次调用还是工作流？

```
任务是单次操作吗？
├─ 是 → 批量处理（100+ 项）？
│   ├─ 每项处理相同 → 循环 + 单次 LLM 调用
│   └─ 每项处理不同 → AI Workflow
└─ 否 → 多个 LLM 步骤顺序执行？
    ├─ 是 → AI Workflow（prompt chaining）
    └─ 否 → 单次 LLM 调用
```

| 场景 | 选择 | 原因 |
|------|------|------|
| 翻译一个产品描述 | 单次 LLM 调用 | 一次性，无依赖 |
| 批量翻译 1000 个描述 | 循环 + 单次 LLM 调用 | 每项处理相同 |
| 提取 + 分类 + 路由 PDF | AI Workflow | 不同步骤，需错误处理 |
| 先生成文案再翻译 | AI Workflow | 顺序步骤（prompt chaining） |

#### Q3.2：工作模式选择

```
需要多步推理吗？
├─ 否 → 单次 LLM 调用或 LLM + Function Call（不需要 Agent）
│   └─ → Q4: 选择组件
└─ 是 → 输出质量关键（代码/法律/学术）？
    ├─ 是 → Reflection 模式（生成 → 评审 → 迭代）
    └─ 否 → 有大量并行子任务？
        ├─ 是 → Multi-Agent 模式（编排器 + 专家）
        └─ 否 → Plan-and-Execute 模式（规划一次，逐步执行）
```

**工作模式对比：**

| 模式 | 最佳场景 | 权衡 |
|------|----------|------|
| **ReAct** | 通用任务、工具使用 | 透明；Token 成本高，可能死循环 |
| **Plan-and-Execute** | 多步任务、研究 | 节省 ~80% 规划 Token；需重规划检查点 |
| **Reflection** | 高质量输出（代码、法律、学术） | 质量更高；更多 Token 循环 |
| **Multi-Agent** | 复杂并行任务 | 可扩展；协调开销 |

#### Q3.3：Agent 数量与协调

```
任务跨越多个领域吗？
├─ 否 → 单 Agent
└─ 是 → 子任务需要并行执行？
    ├─ 是 → Multi-Agent（MOE 模式）
    │         └─ Agent 需要运行时发现？
    │             ├─ 是 → A2A 协议
    │             └─ 否 → 静态编排（代码定义路由）
    └─ 否 → 单 Agent + 多领域 Skills
```

**Agent 类型：**

| Agent 类型 | 能力 | 使用场景 |
|------------|------|----------|
| **BaseAgent** | 固定工作流，无动态规划 | 简单聊天机器人、AI Workflow |
| **ReActAgent** | 思考→行动→观察循环 | 需要工具使用的理性任务 |
| **PlanAgent** | 全局规划 + ReAct 执行 | 需要策略的复杂任务 |

### Q4：什么组件？

```
任务需要什么级别的 AI 能力？
├─ 仅需文本生成/理解（无工具调用）
│   └─ Q4.1: Prompt 还是 Skills？
├─ 需要外部工具/数据访问
│   └─ Q4.2: MCP 还是 Function Call？
├─ 需要多步编排
│   ├─ 步骤预定义 → Workflow
│   └─ 步骤未预定义 → Agent（→ Q3.2/Q3.3）
└─ 需要多 Agent 协作
    ├─ 需要运行时发现 → A2A 协议
    └─ 静态编排 → 代码定义路由
```

**关键原则：从最简单的组件开始。Prompt 优先于 Skills，Function Call 优先于 MCP，Workflow 优先于 Agent，单 Agent 优先于 Multi-Agent。**

#### AI 组件全景

| 层级 | 组件 | 用途 | 类比 |
|------|------|------|------|
| **基础层** | LLM | 文本生成/理解 | 大脑 |
| **基础层** | Prompt | 指令模板 | 说话方式 |
| **能力层** | Function Call | 单次工具调用 | 手（抓取一个东西） |
| **能力层** | MCP | 标准化多工具接入 | USB 接口（即插即用） |
| **能力层** | Skills | 可复用能力包 | 技能书（封装知识） |
| **编排层** | Workflow | 确定性多步流水线 | 流水线（固定流程） |
| **编排层** | Agent | 动态推理 + 工具使用 | 工人（自主判断） |
| **协作层** | A2A | Agent 间通信协议 | 电话系统（跨团队） |
| **交互层** | CLI | 命令行界面 | 控制台（直接操作） |

#### Q4.1：Prompt vs Skills vs CLI

| 组件 | 本质 | 最佳场景 |
|------|------|----------|
| **Prompt** | 单一指令模板 | 快速问答、原型设计、简单生成 |
| **Skills** | 可复用的 Prompt + 上下文包 | 领域知识封装、团队工作流 |
| **CLI** | 命令行界面 | 开发者工具、系统管理、自动化脚本 |

**Prompt → Skills 毕业标准：** 使用 ≥3 次 或 ≥3 人使用 → 升级为 Skills。需要版本控制 → 必须使用 Skills。

#### Q4.2：MCP vs Function Call

| 维度 | Function Call | MCP |
|------|--------------|-----|
| **工具数量** | 少量固定工具（1-5） | 动态发现的大量工具 |
| **可复用性** | 单一应用 | 跨应用、跨模型 |
| **搭建成本** | 低（纯代码） | 中（需要服务进程） |
| **最佳场景** | 内部简单工具调用 | 工具市场、多数据源访问 |

```
工具数量和复杂度如何？
├─ 少量固定工具（1-5），仅内部使用 → Function Call
├─ 多工具，需发现，或跨应用复用 → MCP
└─ 已有 MCP Server 生态 → MCP
```

### Q5：什么部署策略？

完成 Q1-Q4 后，参考部署和运维指导：

- **部署模式：** 云、本地、边缘、混合
- **安全与认证：** 按场景选择认证方式
- **可观测性：** AI 特定监控指标和告警阈值
- **成本优化：** 模型分层、缓存、Prompt 压缩、批处理
- **版本管理：** Prompt、模型、MCP Server、Skills 的版本控制
- **CI/CD：** 测试 Prompt、模型和 Skills 的 7 阶段流水线
- **合规与治理：** 审计、偏见检测、数据血缘、人工监督
- **多租户架构：** 数据、工具、Prompt 和模型隔离模式

---

## 四、35 种组件组合模式

LLM 始终是基础组件，按复杂度分为四个层级：

### Tier 1：最小组合（1-2 个组件）

| # | 模式 | 组件 | 示例 |
|---|------|------|------|
| 1 | 直接 API | LLM | 简单文本生成 API |
| 2 | Prompt 应用 | LLM + Prompt | 带系统提示的聊天机器人 |
| 3 | 工具增强 | LLM + Function Call | 能搜索网页的 AI |
| 4 | 平台工具 | LLM + MCP | 带可插拔工具市场的 AI |
| 5 | 领域知识 | LLM + Skills | 带团队规则的代码审查 |
| 6 | 流水线 | LLM + Workflow | 文档处理（提取→分类→路由） |
| 7 | 对话 | LLM + Session | 带上下文的多轮聊天 |

### Tier 2：标准组合（2-3 个组件）

| # | 模式 | 组件 | 示例 |
|---|------|------|------|
| 8 | Prompt 工具 | LLM + Prompt + Function Call | 能调用 API 的聊天机器人 |
| 18 | Agent 工具 | LLM + Agent + Function Call | 带固定工具的轻量 Agent |
| 19 | 领域 Agent | LLM + Agent + Skills | 带领域知识的 Agent |
| 22 | 开发者平台 | LLM + CLI + MCP | Claude Code、Cursor |

### Tier 3：高级组合（3-4 个组件）

| # | 模式 | 组件 | 示例 |
|---|------|------|------|
| 24 | 完整 Agent | LLM + Agent + MCP + Skills | 带领域专长的自主 Agent |
| 25 | 跨组织 Agent | LLM + Agent + A2A + Skills | 跨组织领域 Agent |
| 32 | 混合编排 | LLM + Workflow + Agent + MCP | 固定 + 动态步骤的流水线 |

### 模式选择指南

```
需要几个组件？
├─ 最小（1-2）→ 多数项目从这里开始
│   ├─ 不需要工具？ → LLM + Prompt
│   ├─ 需要工具？ → LLM + Function Call
│   ├─ 工具很多？ → LLM + MCP
│   └─ 团队知识？ → LLM + Skills
├─ 标准（2-3）→ 成长中的项目
│   ├─ 流水线？ → LLM + Workflow + [FC/MCP/Skills]
│   ├─ 聊天机器人？ → LLM + Function Call + Session
│   ├─ Agent？ → LLM + Agent + [FC/MCP]
│   └─ 开发工具？ → LLM + CLI + [FC/MCP]
├─ 高级（3-4）→ 成熟项目
│   ├─ 领域 Agent？ → LLM + Agent + MCP + Skills
│   └─ 跨组织？ → LLM + Agent + A2A + Skills
└─ 复杂（4+）→ 企业系统
    ├─ 混合编排？ → LLM + Workflow + Agent + MCP
    └─ 完整平台？ → LLM + Agent + MCP + Session + Skills
```

---

## 五、RAG 决策路径

当用户提到"知识库"、"文档问答"或"检索增强"时，进入 RAG 决策路径：

```
相关内容能否全部放入上下文窗口？
├─ 能（<100K tokens）
│   ├─ 静态内容 → 长上下文
│   └─ 动态内容 → RAG
└─ 不能（>100K tokens）→ RAG（必须检索相关子集）
    └─ → Q4.2: MCP vs Function Call（检索工具选择）
```

| RAG 变体 | 组件 | 使用场景 |
|----------|------|----------|
| 简单 RAG | LLM + Function Call（单次检索） | 文档一次性问答 |
| 高级 RAG | LLM + Function Call（多次检索）+ Workflow | 多步：查询→检索→重排→生成 |
| Agentic RAG | LLM + Agent + Function Call | 检索策略需要动态规划 |

### 向量数据库选择

| 数据库 | 类型 | 最佳场景 | 规模 |
|--------|------|----------|------|
| **pgvector** | PostgreSQL 扩展 | 中小规模，已有 PostgreSQL | <10M 向量 |
| **Milvus** | 专用向量数据库 | 大规模，高吞吐 | 100M+ 向量 |
| **Pinecone** | 托管向量数据库 | 快速启动，免运维 | 任意规模 |
| **Weaviate** | 向量搜索引擎 | 多模态搜索 | 中大规模 |
| **Chroma** | 嵌入式向量数据库 | 原型开发，本地 | <1M 向量 |

### RAG 质量指标

| 指标 | 衡量内容 | 目标值 | 改进方法 |
|------|----------|--------|----------|
| 检索精度 | 检索到的块中相关的比例 | >80% | 混合搜索、查询转换、重排序 |
| 检索召回率 | 相关块被检索到的比例 | >90% | 多查询扩展、块重叠、多样化检索 |
| 忠实度 | 声明有检索上下文支持的比例 | >95% | 引用提取、LLM-as-judge 验证 |
| 答案相关性 | 回复中回答查询的比例 | >90% | 查询改写、生成前重排序 |
| 端到端延迟 | 查询到响应的时间 | <2s | 缓存、索引优化、嵌入量化 |

---

## 六、快速决策卡片

### 卡片 1：是否需要 AI？

```
能用规则解决吗？
├─ 能 → 传统架构（不需要 AI）
└─ 不能 → 需要语言理解？
    ├─ 不需要 → 传统 ML（分类/预测）
    └─ 需要 → 进入卡片 2
```

### 卡片 2：选择哪个 AI 组件？

```
需要什么能力？
├─ 仅生成/理解文本 → LLM + Prompt
├─ 访问外部工具/数据 → Function Call 或 MCP
├─ 多步编排 → Workflow（固定）或 Agent（动态）
└─ 跨 Agent 协作 → A2A 协议
关键：从简单开始，仅在需要时增加复杂度。
```

### 卡片 3：是否需要 Agent？

```
需要动态规划吗？
├─ 不需要 → Workflow 或 LLM + Function Call（更便宜、更快）
└─ 需要 → 单 Agent 还是 Multi-Agent？
    ├─ 单领域 → 单 Agent
    └─ 多领域 → Multi-Agent（MOE 模式）
反模式：什么问题都用 Agent。
```

### 卡片 4：约束检查

```
有硬约束吗？
├─ 延迟 <100ms → 优先 Function Call，避免 Agent，积极缓存
├─ 高成本敏感 → 模型分层 + 语义缓存
├─ 数据隐私 → 本地模型，进程内工具
└─ 多个硬约束 → 约束冲突优先级矩阵
```

### 卡片 5：Skills 决策（10 秒判断）

```
Prompt 使用 ≥3 次或 ≥3 人使用？
├─ 是 → 升级为 Skills（封装 + 版本控制）
└─ 否 → 继续用 Prompt（更简单）
团队从第一天就需要共享领域知识？
├─ 是 → 直接用 Skills（跳过 Prompt 阶段）
└─ 否 → Prompt 就够了
```

---

## 七、代码模板

### API 设计：原子工具定义

```json
{
  "name": "get_product",
  "description": "Retrieve product basic information",
  "parameters": {
    "type": "object",
    "properties": {
      "product_id": {
        "type": "string",
        "description": "Unique product identifier"
      }
    },
    "required": ["product_id"]
  }
}
```

**错误响应格式：**

```json
{
  "error": {
    "code": "PRODUCT_NOT_FOUND",
    "message": "Product with given ID does not exist",
    "suggestion": "Verify product_id or use search_product tool"
  }
}
```

**API 重新设计示例：**

```json
// ❌ 错误：巨型 API
{
  "name": "getProductWithInventoryAndPricing",
  "parameters": {"id": "string"}
}

// ✅ 正确：原子工具
{
  "tools": [
    {"name": "get_product", "parameters": {"product_id": "string"}},
    {"name": "get_inventory", "parameters": {"product_id": "string"}},
    {"name": "get_pricing", "parameters": {"product_id": "string"}}
  ]
}
```

### Agent 架构：ReAct Agent

```python
class ReActAgent:
    def __init__(self, tools, max_steps=15, timeout=300):
        self.tools = tools
        self.max_steps = max_steps
        self.timeout = timeout

    def run(self, task):
        history = []
        for step in range(self.max_steps):
            thought = self.think(task, history)
            action = self.select_action(thought)
            if action.type == "FINISH":
                return self.format_output(history)
            observation = self.execute(action)
            history.append({
                "thought": thought,
                "action": action,
                "observation": observation
            })
        return self.summarize_partial(history)
```

### Agent 架构：Plan-and-Execute Agent

```python
class PlanAndExecuteAgent:
    def __init__(self, planner, executor, max_replans=3):
        self.planner = planner
        self.executor = executor
        self.max_replans = max_replans

    def run(self, task):
        plan = self.planner.create_plan(task)
        results = []
        for step in plan.steps:
            result = self.executor.execute(step)
            results.append(result)
            if self.needs_replan(result):
                plan = self.planner.replan(task, results)
        return self.synthesize(results)
```

### MCP 配置

```json
{
  "mcpServers": {
    "product-service": {
      "command": "npx",
      "args": ["-y", "@example/mcp-product-server"],
      "env": {"API_KEY": "${PRODUCT_API_KEY}"}
    },
    "inventory-service": {
      "command": "python",
      "args": ["-m", "mcp_inventory_server"],
      "env": {"DB_URL": "${INVENTORY_DB_URL}"}
    }
  }
}
```

### 死循环防护

```python
guards = {
    "max_steps": 15,
    "timeout_seconds": 300,
    "duplicate_detection": {
        "enabled": True,
        "threshold": 3,  # 相同工具+参数 3 次 → 退出
        "action": "summarize_and_exit"
    }
}
```

### 工具调用幂等性

当 Agent 重试工具调用（超时、网络错误）时，工具可能已经执行过了。没有幂等性，重试会导致重复的副作用（双重扣费、重复邮件等）。

| 工具类型 | 幂等？ | 策略 | 示例 |
|----------|--------|------|------|
| 读/查询 | ✅ 是 | 自由重试 | `get_user`、`search_products` |
| 创建（唯一约束） | ⚠️ 有条件 | 先检查是否存在 | `create_order` |
| 更新（设置状态） | ✅ 是 | 自由重试 | `set_status("completed")` |
| 更新（递增） | ❌ 否 | 幂等键 | `add_balance(100)` → 重复 = +200 |
| 删除 | ✅ 是 | 自由重试 | `delete_item`（已删除 = 无操作） |
| 发送（邮件/通知） | ❌ 否 | 幂等键 | `send_email` → 重复 = 2 封邮件 |
| 支付 | ❌ 否 | 幂等键（必须） | `charge(50)` → 重复 = $100 |

```python
def call_tool_with_idempotency(tool, params, idempotency_key):
    cached = idempotency_store.get(idempotency_key)
    if cached:
        return cached  # 返回之前的结果，不重新执行
    result = tool.execute(params)
    idempotency_store.set(idempotency_key, result, ttl=24h)
    return result
```

**关键原则：** 任何有副作用的工具（写入、发送、支付）在被可能重试的 Agent 调用时，**必须**使用幂等键。只读工具天然幂等。

---

## 八、常见错误与反模式

### 架构选型错误

| 错误 | 为什么错 | 正确做法 |
|------|----------|----------|
| 为简单 FAQ 使用 Multi-Agent | 过度工程，增加延迟/成本 | 单 Agent + 知识库 |
| 为 Agent 设计复杂嵌套 API | Agent 无法解析深层结构 | 原子工具 + 扁平参数 |
| 为确定性任务使用 ReAct | 不必要的推理开销 | BaseAgent 或 Workflow |
| MCP 用于 1-3 个简单内部工具 | 不必要的基础设施开销 | 直接用 Function Call |
| Agent 用于固定步骤流水线 | 动态推理增加成本/延迟但无收益 | 使用 Workflow |
| Workflow 用于开放式研究 | 无法适应意外发现 | 使用 Agent |
| 原始 Prompt 用于团队重复知识 | 无复用、无版本控制、无团队共享 | 升级为 Skills |
| A2A 用于同代码库静态路由 | 协议开销用于代码定义的路径 | 使用静态编排 |
| LLM 用于结构化数据预测 | 传统 ML 更快、更便宜、更准确 | 使用传统 ML（XGBoost 等） |
| GPT-4 用于简单分类 | 10 倍成本换取微小准确率提升 | 模型分层：先用小模型 |

### 辩护表格（常见借口 vs 现实）

| 借口 | 现实 |
|------|------|
| "Multi-Agent 总是更好" | 协调开销对简单任务不合理 |
| "ReAct 能处理一切" | 确定性任务不需要动态推理 |
| "AI 友好 API 太费力" | 原子工具更容易维护和测试 |
| "上下文工程可选" | 记忆和上下文比模型选择更重要 |
| "一切都需要 AI" | 传统架构更好地处理确定性逻辑 |
| "MCP 总是比 Function Call 好" | MCP 有基础设施开销；对简单内部工具过度 |
| "我们需要 GPT-4 做所有事" | 小模型以 10% 的成本处理 80% 的任务 |
| "LLM 能替代所有 ML" | 传统 ML 在结构化数据预测上更优 |
| "AI 不需要监控" | 没有可观测性，AI 系统会静默失败 |

### 危险信号（停下来重新考虑）

- 没有明确问题就添加 AI
- 能用 LLM 调用解决却用 Agent
- 为 Agent 设计复杂嵌套 API
- 跳过评估和可观测性
- 忽略"何时不使用"指南
- 没有缓存策略（重复查询浪费成本）
- 没有可观测性（静默失败）
- 云端部署但隐私要求本地
- 高风险决策无人工监督
- 视频流实时 LLM（应使用帧采样 + 批处理）
- 多租户系统无数据/工具/Prompt 隔离
- 无 Prompt 注入防御
- 修改 Prompt 不做 A/B 测试
- 消费级 AI 产品无用户记忆/个性化

---

## 九、安全与合规

### 认证方式

| 场景 | 认证方式 |
|------|----------|
| 内部工具 | 服务间认证（mTLS） |
| 外部 API | API Key / OAuth |
| 用户数据 | OAuth + 用户授权 |
| 敏感操作 | 人工确认 |

### 合规要素

- **输出审计：** 记录所有 AI 输入/输出
- **偏见检测：** 定期测试数据集
- **人工监督：** 高风险决策人工审批
- **可解释性：** 要求推理链（CoT）

### 多租户隔离

| 隔离维度 | 实现方式 |
|----------|----------|
| 数据隔离 | 租户 ID + 行级安全 |
| 工具隔离 | 租户级 MCP 配置 |
| Prompt 隔离 | 租户级模板 |
| 模型隔离 | 按租户层级路由 |

### 对抗性输入防御

- 输入过滤：检测攻击模式
- System Prompt 加固：防覆盖
- 输出验证：防数据泄露
- 沙箱执行：最小权限
- 限流 + 异常检测

---

## 十、渐进式采用路径

对于已有系统，不要推倒重来，而是渐进式集成 AI：

```
已有 REST API → Function Call 包装
已有工作流引擎 → 在现有流程中加 LLM 步骤
已有数据库 → Text-to-SQL + Function Call
已有微服务 → Agent + MCP 包装服务
```

**完整迁移路径：** Prompt → Function Call → MCP → Agent → Multi-Agent → A2A

---

## 十一、成本优化策略

| 策略 | 说明 |
|------|------|
| 模型分层 | 小模型处理 80% 任务，大模型处理 20% 复杂任务 |
| 语义缓存 | 相似请求复用结果（30-50% 命中率） |
| Prompt 压缩 | 减少 Token 使用 |
| 本地模型 | 高频简单任务使用本地模型 |
| 批处理 | 批量请求降低单次成本 |

---

## 十二、可观测性

| 维度 | 监控指标 |
|------|----------|
| 模型性能 | 准确率、延迟、错误率 |
| Token 用量 | 成本、缓存命中率 |
| Agent 行为 | 步数、工具调用、循环检测 |
| 用户体验 | 任务完成率、满意度 |

---

## 十三、输出模板

### 架构推荐

```markdown
**推荐：** [类型]
**理由：** [1-2 句，引用 Q1-Q4]
**工作量：** [低/中/高] — [时间线]；驱动因素：[基础设施/开发/Token]
**硬约束：** [...] **软约束：** [...]
**备选方案：** [方案 A] 因 [原因] 排除；[方案 B] 因 [原因] 排除
**下一步：** 1. [启动] → 2. [扩展] → 3. [迁移]
```

### Agent 类型选择

```markdown
**Agent 类型：** [BaseAgent/ReActAgent/PlanAgent]
**模式：** [ReAct/Plan-and-Execute/Reflection/Multi-Agent]
**防护：** max_steps=[n], timeout=[s], dedup=[yes/no]
**工作量：** [低/中/高] — [时间线]；复杂度：[驱动因素]
```

### 反模式告警

```markdown
**问题：** [什么问题]
**原因：** [违反了什么规则]
**替代方案：** [应该怎么做]
**成本：** [浪费的努力/Token/复杂度]
```

---

## 参考资料

- [ReAct 论文](https://arxiv.org/abs/2210.03629)（Yao et al., 2022）
- [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)
- [Google A2A Protocol](https://google.github.io/A2A/)
- [原始项目仓库](https://github.com/GanJiaKouN16/ai-friendly-architecture-design-skill)

---

## 相关笔记

#architecture #ai #agent #decision

- [[全景]] — AI Agent 架构师必备知识体系全景图
- [[Palantir 本体论 (Ontology)]] — 本体论与数据建模
- [[Loop Engineering]] — 循环工程设计模式
- [[Harness 工程化]] — 流程工程化实践
- [[AI Agent & Skill 测评方案]] — Agent 评测框架
- [[AI Coding Agent 成本优化]] — Token 成本优化策略
