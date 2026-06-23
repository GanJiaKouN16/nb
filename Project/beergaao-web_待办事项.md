# BeerGaao-Web 待办事项

#beergaao

## 高优先级

### 深层数据区块
- **位置**: `src/components/stock-detail/DeepData.tsx`
- **现状**: 占位按钮，待接入实际数据
- **待办**: 接入财报/同业/股东/技术数据的完整展示

### API 数据加载问题
- **位置**: `src/data/fundamentals.ts`, `vite.config.ts` proxy
- **现状**: 点击"加载对比"、"加载股东"按钮后数据未显示
- **问题**:
  - [ ] **同业对比 (peers)**: `push2.eastmoney.com/api/qt/slist/get` 返回 `rc:102` 错误
  - [ ] **十大股东 (holders)**: 按钮点击后数据未加载
  - [ ] **财报摘要 (f10)**: A 股 API 返回 HTML 错误页面

### LLM 分析模块
- **位置**: `src/data/analysis.ts`, `src/components/stock-detail/InvestorAnalysis.tsx`
- **现状**: 未配置 LLM Key 时显示"分析生成失败"
- **依赖 LLM 的模块**:
  - [ ] 综合信号 / ML 预测 / 市场环境 / 置信度
  - [ ] 目标价 / 折溢价 / 关键风险
  - [ ] DCF 内在价值 / Owner Earnings
  - [ ] 投资大师分析 (13 位大师)
  - [ ] 情绪分析 / 风险分析

### 测试覆盖
- **现状**: 仅 E2E 测试，缺少单元测试
- **待办**:
  - 为 `data/indicators.ts` 添加单元测试
  - 为 `data/analysis.ts` 添加单元测试
  - 为 `data/stocks.ts` 添加单元测试
  - 扩充 E2E 测试覆盖
  模块覆盖: 8/10 (80%)
  函数覆盖: ~36/39 (~92%)
  整体行覆盖估算: ~75-80%

## 中优先级

### 错误处理
- **现状**: 部分 API 调用缺少 try/catch
- **待办**:
  - `src/data/stocks.ts` 补充错误边界
  - `src/data/moneyflow.ts` 补充错误边界
  - 前端统一 toast 错误提示

### 加载状态
- **现状**: 部分组件缺少 loading 态
- **待办**:
  - 资金流向区块添加骨架屏
  - 回测区块添加骨架屏
  - 深层数据区块添加骨架屏

## 低优先级

### 性能优化
- **待办**:
  - K 线图组件懒加载
  - 搜索结果虚拟滚动
  - LLM 分析结果缓存

### 用户体验
- **待办**:
  - 搜索历史云端同步
  - 自选股云端同步
  - 移动端适配优化
