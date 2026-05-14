# ward-skill

AI Agent 调用 [Ward](https://github.com/rainj2013/ward-agent) 美股数据 API 的技能说明。适用于 Hermes、OpenClaw、Claude Code 等任何能发 HTTP 请求的 Agent。

用户只需在微信/飞书/Telegram 等平台发消息问美股，Agent 自动调 Ward API 回复。当前 skill 覆盖行情查询、黄金、盘前盘后、AI 分析报告、异步 Job、Runtime Trace 和多股对比 Team。

## 前置要求

- [Ward](https://github.com/rainj2013/ward-agent) 已安装并运行在 `localhost:8000`
- 任意 AI Agent（Hermes、OpenClaw、Claude Code 等）

## 安装（Hermes）

将 `ward` 目录放入 `~/.hermes/skills/`:

```bash
cp -r ward ~/.hermes/skills/
```

重启 Hermes Gateway：

```bash
hermes gateway run --replace
```

## 安装（OpenClaw）

将 `ward` 目录放入 `~/.openclaw/skills/`:

```bash
cp -r ward ~/.openclaw/skills/
```

## 安装（Claude Code / 其他 Agent）

Skill 本质上是一份 API 调用说明文件（SKILL.md），任何 Agent 只需读取该文件即可获知如何调用 Ward API。直接读取或复制 `SKILL.md` 的内容即可使用。

## 使用方式

安装后，直接在已接入 Agent 的平台发消息问美股即可：

```
# 指数行情
"纳斯达克现在怎么样"
"道琼斯今天跌了多少"
"标普500实时行情"

# 个股查询
"帮我看看 NVDA 的价格"
"苹果股票现在多少"
"特斯拉走势怎么样"

# AI 分析报告
"帮我分析一下 TSLA"
"纳指 AI 报告"
"生成今天的市场报告"

# 智能问答
"现在市场情绪怎么样"
"科技股最近表现如何"

# 多股对比
"比较 NVDA、TSLA、AAPL 的相对机会和风险"
"MSFT 和 GOOG 哪个更适合长期观察"

# Runtime / Trace
"查一下 job_xxx 的执行过程"
"这个分析任务为什么失败了"
```

## 效果

```
用户: 纳斯达克今天怎么样
Agent: Nasdaq 综合: 24,485.70 (+0.33%)
       Nasdaq 100: 26,666.89 (+0.29%)
       道琼斯: 49,813.09 (+0.75%)
       标普 500: 7,131.41 (+0.31%)
       三大指数全线小幅收涨...
```

## 当前支持的 Ward 能力

- 市场总览：Nasdaq 综合、Nasdaq 100、道琼斯、标普 500、黄金
- 个股查询：报价、历史数据、K 线、盘前 / 盘中 / 盘后
- AI 分析：个股、指数、黄金、市场报告
- 异步任务：创建分析 Job，轮询状态，订阅 SSE 事件
- Runtime Trace：查看任务阶段、缓存命中、prompt、模型返回、token 和耗时
- 多股对比：通过聊天接口创建 Leader / Worker / Verifier Team 任务
- 对话历史：读取会话历史和分页加载更早消息

## 推荐调用方式

- 简单价格、涨跌幅、K 线问题：直接调用同步数据接口。
- 深度 AI 报告：优先创建 `/api/analysis-jobs/*` 异步任务，避免 Agent 平台超时。
- 多股对比：优先调用 `/api/chat/stream`，Ward 会自动识别并创建 Team 任务。
- 排障或解释报告来源：用 `/api/analysis-jobs/{job_id}/trace`。

## 公网访问

Ward 默认只绑定 `127.0.0.1:8000`。如需开启公网访问，设置环境变量 `WARD_PUBLIC_MODE=1` 后重启 Ward：

```bash
WARD_PUBLIC_MODE=1 screen -dmS ward /root/.venv/bin/ward
```

## 目录结构

```
ward/
├── README.md   # 本文件
└── SKILL.md    # Agent Skill 定义（Hermes/OpenClaw/Claude Code 等通用）
```
