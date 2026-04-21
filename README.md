# ward-skill

AI Agent 调用 [Ward](https://github.com/rainj2013/ward-agent) 美股数据 API 的技能说明。适用于 Hermes、OpenClaw、Claude Code 等任何能发 HTTP 请求的 Agent。

用户只需在微信/飞书/Telegram 等平台发消息问美股，Agent 自动调 Ward API 回复。

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

# 智能问答
"现在市场情绪怎么样"
"科技股最近表现如何"
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
