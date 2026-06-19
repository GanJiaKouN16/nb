#beergaao
# BeerGaao-Web 项目说明

# BeerGaao-Web — 智能股票分析工具

基于 LLM Agent 的智能股票分析工具前端项目。

## 技术栈

- React 19 + TypeScript
- Vite 8
- TailwindCSS v4
- Lightweight Charts (K线图)
- React Router v7
- GitHub OAuth (Vite server middleware)

## 项目结构

```
src/
├── pages/              # 页面组件
│   ├── Home.tsx        # 首页（含内嵌股票详情）
│   ├── Settings.tsx    # 设置页
│   ├── Login.tsx       # 登录页
│   └── OAuthCallback.tsx # GitHub OAuth 回调
├── components/         # 通用组件
│   ├── Nav.tsx         # 顶部导航（暗/亮双变体）
│   ├── Footer.tsx      # 法律免责声明
│   ├── StockDetail.tsx # 股票详情（11 个区块）
│   └── KLineChart.tsx  # Lightweight Charts K 线图组件
├── hooks/
│   └── useAuth.ts      # GitHub 登录状态 hook
├── data/
│   ├── stocks.ts       # 东方财富 API 封装
│   ├── llm.ts          # LLM 客户端
│   ├── analysis.ts     # 5 个 LLM 分析函数
│   ├── indicators.ts   # 8 个技术指标前端纯算
│   ├── moneyflow.ts    # 资金流向数据
│   ├── backtest.ts     # 前端回测引擎
│   ├── fundamentals.ts # 财报/同业/股东数据
│   ├── providers.ts    # Tushare / Wind 付费接口
│   └── provider.ts     # 数据源路由 + LLM 配置管理
├── lib/
│   ├── supabase.ts     # Supabase 客户端初始化
│   ├── storage.ts      # localStorage / Supabase 双模式存储
│   ├── auth.ts         # Supabase Auth（GitHub OAuth）
│   └── crypto.ts       # AES-256-GCM 加密
├── styles/
│   └── tokens.ts       # 共享样式 token
├── App.tsx             # 路由入口
├── main.tsx            # 主入口
└── index.css           # 全局样式
server/
├── main.py             # FastAPI Agent 后端骨架
└── requirements.txt    # Python 依赖
supabase/
└── init.sql            # user_settings 表 + RLS 策略
```

## 快速开始

```bash
npm install
cp .env.example .env
npm run dev
```

访问 http://localhost:5173

## 功能

### 首页
- Hero — 暗色全幅 hero
- 搜索 — 实时下拉建议（300ms 防抖）
- 搜索历史 — 卡片式展示，最多 10 条
- 我的自选 — 卡片式展示
- 股票详情 — 11 个区块垂直排列

### 设置页
- LLM 提供商 — 四选一 + API Key 校验
- 数据接口 — 免费/付费二选一
- 云端同步 — AES-256-GCM 加密

## 数据源

通过 Vite 代理调用东方财富和腾讯公开 API：
- 搜索: searchapi.eastmoney.com
- 行情: push2.eastmoney.com
- K线: web.ifzq.gtimg.cn（腾讯）
- 资金流向: push2.eastmoney.com
- 财报: emweb.securities.eastmoney.com

## 相关项目

- BeerGaao-skill
- buffett-skills
- ai-hedge-fund
