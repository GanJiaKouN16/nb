# nb 知识图谱

## 核心概念

```
nb
├── 📓 笔记本 (Notebooks)
│   ├── home (主笔记本)
│   ├── 自定义笔记本
│   └── 归档笔记本
├── 📁 文件夹 (Folders)
│   ├── Architecture
│   ├── Engineering
│   ├── Project
│   ├── Tool
│   └── Other
├── 📝 笔记 (Notes)
│   ├── 普通笔记
│   ├── 书签 (Bookmarks)
│   └── 待办 (Todos)
└── 🔗 关联 (Links)
    ├── [[wiki-style links]]
    ├── #tags
    └── --related URLs
```

---

## 功能分类图

### 1. 📝 内容创建与管理

```
创建内容
├── nb add              # 添加笔记
│   ├── --filename      # 指定文件名
│   ├── --title         # 指定标题
│   ├── --tags          # 添加标签
│   └── --encrypt       # 加密笔记
├── nb add bookmark     # 添加书签
├── nb add folder       # 创建文件夹
└── nb add todo         # 添加待办事项
```

### 2. 📖 内容查看

```
查看内容
├── nb show             # 显示笔记详情
│   ├── --print         # 打印到标准输出
│   ├── --render        # 渲染 Markdown
│   └── --browse        # 在浏览器中打开
├── nb peek             # 在终端预览
├── nb open             # 打开笔记/书签
├── nb list (ls)        # 列出笔记
│   ├── --excerpt       # 显示摘要
│   ├── --tags          # 显示标签
│   └── --folders-first # 文件夹优先
└── nb count            # 统计笔记数量
```

### 3. ✏️ 内容编辑

```
编辑内容
├── nb edit             # 编辑笔记
│   ├── --editor        # 指定编辑器
│   ├── --last          # 编辑最后一条
│   └── --prepend       # 在开头添加
├── nb move             # 移动/重命名笔记
├── nb copy             # 复制笔记
└── nb delete           # 删除笔记
```

### 4. 🔍 搜索与过滤

```
搜索功能
├── nb search           # 搜索笔记
│   ├── --and           # 与条件
│   ├── --or            # 或条件
│   ├── --not           # 非条件
│   └── --tags          # 按标签搜索
├── nb list --tags      # 按标签过滤
└── nb list --type      # 按类型过滤
```

### 5. 🔗 关联与链接

```
关联功能
├── [[wiki-style links]]  # Wiki 风格链接
│   ├── [[笔记ID]]
│   ├── [[笔记标题]]
│   ├── [[文件夹/笔记]]
│   └── [[笔记本:笔记]]
├── #tagging             # 标签系统
│   ├── #标签名
│   └── nb list --tags
├── --related            # 关联 URL
└── nb browse            # 浏览链接关系
```

### 6. 📂 笔记本管理

```
笔记本管理
├── nb notebooks         # 列出笔记本
│   ├── add              # 创建笔记本
│   ├── delete           # 删除笔记本
│   ├── rename           # 重命名笔记本
│   ├── use              # 切换笔记本
│   └── archive          # 归档笔记本
├── nb init              # 初始化笔记本
├── nb sync              # 同步到远程仓库
└── nb remote            # 配置远程仓库
```

### 7. 📁 文件夹管理

```
文件夹管理
├── nb folders           # 文件夹操作
│   ├── add              # 创建文件夹
│   ├── delete           # 删除文件夹
│   └── list             # 列出文件夹
└── nb list --folders-first  # 文件夹优先显示
```

### 8. ✅ 待办事项管理

```
待办管理
├── nb todo              # 待办操作
│   ├── add              # 添加待办
│   ├── do               # 标记完成
│   └── undo             # 标记未完成
├── nb todos             # 列出待办
│   ├── open             # 未完成的
│   └── closed           # 已完成的
└── nb tasks             # 列出任务
```

### 9. 🏷️ 标签系统

```
标签功能
├── 添加标签
│   ├── nb add --tags tag1,tag2
│   └── nb edit --tags tag1,tag2
├── 查看标签
│   ├── nb list --tags
│   └── nb show --tags
└── 搜索标签
    ├── nb search --tags tag1
    └── nb search #标签名
```

### 10. 📌 固定功能

```
固定功能
├── nb pin              # 固定笔记
└── nb unpin            # 取消固定
```

### 11. 📤 导入导出

```
导入导出
├── nb import           # 导入
│   ├── bookmarks       # 导入书签
│   ├── copy            # 复制导入
│   ├── download        # 下载导入
│   └── move            # 移动导入
├── nb export           # 导出
│   ├── notebook        # 导出笔记本
│   └── pandoc          # Pandoc 格式导出
└── nb archive          # 归档笔记本
```

### 12. 🔧 系统配置

```
系统配置
├── nb settings         # 设置管理
│   ├── colors          # 颜色主题
│   ├── edit            # 编辑设置
│   └── list            # 列出设置
├── nb env              # 环境信息
├── nb plugins          # 插件管理
│   ├── install         # 安装插件
│   └── uninstall       # 卸载插件
└── nb update           # 更新 nb
```

### 13. 📜 历史与版本

```
版本控制
├── nb history          # 查看历史
├── nb git              # Git 操作
│   ├── checkpoint      # 创建检查点
│   └── dirty           # 检查未提交更改
└── nb undo             # 撤销操作
```

### 14. 🌐 浏览功能

```
浏览功能
├── nb browse           # 浏览笔记
│   ├── --gui           # GUI 浏览器
│   ├── --serve         # 启动服务器
│   └── --daemon        # 后台运行
└── nb shell            # 交互式 shell
```

---

## 常用工作流

### 创建笔记并关联

```bash
# 1. 创建笔记
nb add --title "新笔记" --tags "标签1,标签2"

# 2. 在其他笔记中引用
echo "查看 [[新笔记]] 了解详情" | nb add --title "关联笔记"

# 3. 浏览关联关系
nb browse
```

### 组织笔记结构

```bash
# 1. 创建文件夹
nb folders add "项目A"

# 2. 移动笔记到文件夹
nb move 1 "项目A/"

# 3. 查看文件夹内容
nb list "项目A/"
```

### 标签管理

```bash
# 1. 添加带标签的笔记
nb add --title "笔记" --tags "工作,重要"

# 2. 搜索特定标签
nb search --tags "工作"

# 3. 列出所有标签
nb list --tags
```

---

## 快速参考表

| 功能 | 命令 | 说明 |
|------|------|------|
| 创建笔记 | `nb add` | 添加新笔记 |
| 查看笔记 | `nb show` | 显示笔记详情 |
| 编辑笔记 | `nb edit` | 编辑笔记内容 |
| 删除笔记 | `nb delete` | 删除笔记 |
| 搜索笔记 | `nb search` | 搜索笔记内容 |
| 列出笔记 | `nb list` | 列出所有笔记 |
| 创建笔记本 | `nb notebooks add` | 创建新笔记本 |
| 切换笔记本 | `nb use` | 切换到指定笔记本 |
| 创建文件夹 | `nb folders add` | 创建新文件夹 |
| 添加标签 | `--tags` | 为笔记添加标签 |
| 固定笔记 | `nb pin` | 固定重要笔记 |
| 同步笔记 | `nb sync` | 同步到远程仓库 |

---

*创建时间: 2026-06-19*
