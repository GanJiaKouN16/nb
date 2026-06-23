# CLI 工具使用指南

## 1. RTK (Rust Token Killer)

**作用**: CLI 代理工具，优化 Claude Code 的 token 使用（节省 60-90%）

**常用命令**:
```bash
rtk <command>          # 代理执行命令，自动过滤输出
rtk git status         # Git 命令的优化版本
rtk read file.txt      # 读取文件并压缩输出
rtk gain               # 查看 token 节省统计
rtk proxy <command>    # 原始执行命令（不经过滤）
```

**示例**:
- `rtk git log --oneline` - 压缩的 git 日志
- `rtk ls -la` - 简化的目录列表

---

## 2. nb

**作用**: CLI 笔记和书签管理工具

**常用命令**:
```bash
nb add "笔记内容"      # 添加笔记
nb list                # 列出所有笔记
nb search "关键词"     # 搜索笔记
nb show <id>           # 查看笔记详情
nb edit <id>           # 编辑笔记
nb delete <id>         # 删除笔记
```

**笔记本管理**:
```bash
nb notebooks           # 列出所有笔记本
nb notebooks add <name> # 创建新笔记本
nb use <notebook>      # 切换笔记本
```

---

## 3. CodeGraph

**作用**: 代码智能和知识图谱工具，用于代码库分析

**常用命令**:
```bash
codegraph init         # 初始化代码图谱
codegraph index        # 索引项目文件
codegraph sync         # 同步代码变更
codegraph status       # 查看索引状态
codegraph query "搜索" # 搜索符号
codegraph explore "问题" # 探索代码区域
```

**MCP 工具** (在 Claude Code 中):
- `codegraph_explore` - 探索代码符号和调用路径
- `codegraph_node` - 查看单个符号的源码和调用者

---

## 4. Superpowers

**作用**: Claude Code 增强插件

**状态**: 作为 Claude 插件自动加载，非独立 CLI 工具

**功能**: 提供额外的 Claude Code 功能和集成

---

## 5. Playwright

**作用**: 浏览器自动化和端到端测试框架

**常用命令**:
```bash
playwright test        # 运行测试
playwright codegen     # 录制浏览器操作
playwright install     # 安装浏览器
playwright show-report # 查看测试报告
```

**注意**: 如果遇到 RTK 拦截问题，使用：
- 完整路径: `/home/qinlei/.npm-global/bin/playwright`
- RTK 代理: `rtk proxy playwright`

---

## 快速参考

| 工具 | 核心用途 | 常用命令 |
|------|----------|----------|
| RTK | Token 优化 | `rtk <command>` |
| nb | 笔记管理 | `nb add/list/search` |
| CodeGraph | 代码分析 | `codegraph init/query/explore` |
| Superpowers | Claude 增强 | 插件自动加载 |
| Playwright | 浏览器测试 | `playwright test/codegen` |

---

*创建时间: 2026-06-19*

---

## 相关笔记

#tool #cli #rtk #nb #codegraph

- [[NB CLI 使用笔记]] — nb 工具详细使用
- [[WSL2 远程调试 Windows Chrome]] — Chrome 调试配置
- [[AI Coding Agent 成本优化]] — RTK 等工具的 token 优化原理
