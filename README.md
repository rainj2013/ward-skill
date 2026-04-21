# hermes-ward

Hermes Agent 技能，让 Hermes 能通过微信、飞书、Telegram 等消息平台调用 [Ward](https://github.com/rainj2013/ward-agent) 的美股数据 API。

用户只需在微信/飞书发消息问美股，Hermes 自动调 Ward API 回复，无需暴露 Ward 到公网。

## 前置要求

- [Ward](https://github.com/rainj2013/ward-agent) 已安装并运行在 `localhost:8000`
- Hermes Agent 已安装

## 安装

将 `hermes-ward` 目录放入 `~/.hermes/skills/`：

```bash
cp -r hermes-ward ~/.hermes/skills/
```

然后重启 Hermes Gateway：

```bash
hermes gateway run --replace
```

## 使用方式

安装后，在微信/飞书等已接入 Hermes 的平台直接发消息问美股即可，Hermes 会自动识别并调用 Ward API：

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
用户(微信): 纳斯达克今天怎么样
Hermes:    Nasdaq 综合: 24,485.70 (+0.33%)
           Nasdaq 100: 26,666.89 (+0.29%)
           道琼斯: 49,813.09 (+0.75%)
           标普 500: 7,131.41 (+0.31%)
           三大指数全线小幅收涨...
```

## 安全设计

Ward 默认只绑定 `127.0.0.1:8000`，不暴露公网。所有来自微信/飞书的消息通过 Hermes Gateway 转发到本地 API，无需开放端口。

如需开启公网访问，设置环境变量 `WARD_PUBLIC_MODE=1` 后重启 Ward 即可：

## 目录结构

```
hermes-ward/
├── README.md      # 本文件
└── SKILL.md       # Hermes Skill 定义（Hermes Agent 自动加载）
```
