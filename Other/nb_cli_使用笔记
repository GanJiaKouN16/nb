# NB CLI 使用笔记

## 基本命令

- `nb notebooks` - 查看所有笔记本
- `nb add --title "标题"` - 创建新笔记
- `nb show <id>` - 查看笔记
- `nb edit <id>` - 编辑笔记
- `nb list` - 列出所有笔记

## 中文显示问题解决

### 问题
`nb show` 默认使用 glow 渲染器，中文显示乱码且带有 ANSI 颜色代码。

### 解决方案

**方法1：使用 --dump 参数**
```bash
nb show <id> --dump
```

**方法2：去除 ANSI 颜色代码（推荐）**
```bash
nb show <id> --dump | sed 's/\x1b[^a-zA-Z]*[a-zA-Z]//g; s/\x0f//g; s/\x07//g' | tr -d '\000-\010\016\017\020-\037'
```

**方法3：设置别名（已配置到 ~/.bashrc）**
```bash
alias nb-show='nb show --dump | sed "s/\x1b[^a-zA-Z]*[a-zA-Z]//g; s/\x0f//g; s/\x07//g" | tr -d "\000-\010\016\017\020-\037"'
alias nb-list='nb list 2>&1 | sed "s/\x1b[^a-zA-Z]*[a-zA-Z]//g; s/\x0f//g; s/\x07//g" | tr -d "\000-\010\016\017\020-\037"'
```
使用：
- `nb-show <id>` - 查看笔记（无颜色）
- `nb-list` - 列出笔记（无颜色）

## 笔记存储位置

- 默认笔记本：`~/.nb/home/`
- 文件格式：markdown (.md)

## 常用设置

- `nb settings list` - 查看所有设置
- `nb settings set color_primary 0` - 尝试禁用颜色（效果有限）

## 备注

- nb 没有直接禁用 glow 渲染器的设置
- 使用别名可以完全去除 ANSI 颜色代码
