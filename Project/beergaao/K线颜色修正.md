---
type: Reference
title: K线颜色修正 - 中国市场标准
description: K 线图颜色从国际标准（涨绿跌红）改为中国市场标准（涨红跌绿）的修改记录与验证方法。
tags: [project, beergaao, kline, frontend, stock]
timestamp: 2026-06-18T00:00:00Z
---

# K线颜色修正 - 中国市场标准


**修改日期**: 2026-06-18

## 问题描述
K线图颜色使用国际标准（涨绿跌红），需要修改为中国市场标准（涨红跌绿）。

## 修改内容

### KLineChart.tsx - K线蜡烛图颜色

**修改前（国际标准）：**
- 上涨：绿色 (#05b169)
- 下跌：红色 (#cf202f)

**修改后（中国标准）：**
- 上涨：红色 (#cf202f)
- 下跌：绿色 (#05b169)

**涉及属性：**
- upColor, downColor, borderUpColor, borderDownColor, wickUpColor, wickDownColor

### KLineChart.tsx - 成交量柱状图颜色

**修改前：** close >= open → 绿色半透明, close < open → 红色半透明
**修改后：** close >= open → 红色半透明, close < open → 绿色半透明

## 相关组件状态

CSS变量已正确：
- --color-up: #FF1744 (红色 - 涨)
- --color-down: #00C853 (绿色 - 跌)

使用这些变量的组件：WatchlistItem, PriceSummaryBar, DeepData, SignalSection, InvestorAnalysis, ValuationSection

## 验证方法

1. K线图：上涨日红色蜡烛，下跌日绿色蜡烛
2. 成交量：上涨日红色半透明，下跌日绿色半透明
3. 其他显示：涨幅数字红色，跌幅数字绿色

## 备注

中国市场（A股、港股等）采用涨红跌绿配色，与国际市场（美股等）的涨绿跌红相反。

---

## 相关笔记


- [BeerGaao-Web 项目说明](/Project/beergaao/项目说明.md) — 项目技术栈与架构
- [BeerGaao-Web 待办事项](/Project/beergaao/待办事项.md) — 项目待办清单
