# 全景

AI Agent 架构师必备知识体系全景图

## 一、核心认知：LLM → Agent 的范式跃迁
LLM 四大局限：只会说不会做、无状态记忆、知识截止、不会规划

Agent 定义：Agent = LLM + 工具 + 记忆 + 规划，在自主循环中完成目标

本质跃迁：LLM 告诉你怎么做 → Agent 直接帮你做完

## 二、架构决策：Agent vs Workflow
| 维度 | Agent | Workflow |
|------|-------|----------|
| 控制权 | LLM 自主决策 | 代码写死流程 |
| Token 消耗 | 4-8x | 1x |
| 灵活性 | 高 | 低 |
| 可预测性 | 低 | 高 |
生产黄金法则：混合架构——Workflow 提供稳定骨架，Agent 处理异常与复杂场景

## 三、四种工作模式
| 模式 | 核心逻辑 | 适用场景 |
|------|----------|----------|
| ReAct | 思考→行动→观察→循环 | 通用任务，追求透明度 |
| Plan-Execute | 先完整规划，再逐步执行 | 结构清晰的任务，Token 仅 ReAct 的 20% |
| Reflection | 生成→审查→迭代优化 | 代码生成、文书写作、幻觉校正 |
| Multi-Agent | 协调者+多个专业 Agent | 复杂协作任务（⚠️ 优先用单 Agent） |

## 四、集成生态：三层工具链
Skills（怎么想）→ MCP（用什么）→ Function Call（怎么调）

| 层级 | 作用 | 技术本质 |
|------|------|----------|
| Function Call | LLM 输出结构化工调用指令（JSON） | API 协议 |
| MCP 协议 | Agent↔工具 标准化集成，解决 N×M 爆炸 | 通信标准（stdio/HTTP+SSE） |
| Skills | 领域专家知识按需注入 | 模块化提示词扩展 |
| A2A 协议 | Agent↔Agent 横向连接 | 智能体间通信（互补 MCP） |

## 五、工程落地四大支柱

1. 记忆系统
短期：上下文窗口
长期：向量数据库（语义）+ 关系数据库（精确）+ KV（高速）
压缩策略：滑动窗口 / 摘要压缩 / 重要性过滤

2. 可靠性四设计
幂等性 → 回滚机制 → 超时控制 → 降级策略

3. 安全防御
四大威胁：Prompt Injection、权限越界、数据泄露、资源滥用
核心原则：最小权限 + Human-in-the-Loop
Prompt Injection 四法：数据指令分离 / 输入过滤 / 模板隔离 / 上下文标记

4. 可观测性
追踪每一轮 Thought→Action→Observation，防止死循环

## 六、RAG 与 Agent 的关系
RAG 是 Agent 工具箱里的「知识查询器」
Agentic RAG：RAG 做成 Agent，自主判断检索策略，动态调整

## 七、问题速查
| 领域 | 核心考点 |
|------|----------|
| 系统设计 | 工具层、记忆层、可靠性、可观测性、安全 |
| 成本优化 | 工具选择、Workflow替代、上下文压缩、模型路由、缓存 |
| 防死循环 | 最大步数（15步）、重复检测（3次退出）、超时控制 |
| 幻觉应对 | Reflection 自我验证 + 工具调用结果校验 |
| 生产踩坑 | 死循环、幻觉调用、上下文污染、Token爆炸、Prompt注入 |

---

## 相关笔记

#architecture #ai #agent #knowledge

- [[AI 友好架构设计：从决策到落地的完整指南]] — AI 架构决策框架
- [[Palantir 本体论 (Ontology)]] — 本体论与数据建模
- [[Loop Engineering]] — 循环工程设计模式
- [[Harness 工程化]] — 流程工程化实践
- [[AI Agent & Skill 测评方案]] — Agent 评测框架
- [[AI Coding Agent 成本优化]] — Token 成本优化策略
