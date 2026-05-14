---
name: ward
description: Ward US Market Data API 调用指南。任何 AI Agent（Hermes、OpenClaw、Claude Code 等）均可使用。Ward 跑在 localhost:8000，支持美股指数、黄金、个股行情、AI 分析报告、盘前盘后数据、异步分析任务、Runtime Trace 和多股对比。
---

# ward — Ward API 调用指南

适用于任何能发 HTTP 请求的 AI Agent（Hermes、OpenClaw、Claude Code 等）。Ward 默认跑在 `localhost:8000`，不暴露公网时通过本机访问。

## 基础信息

| 项目 | 值 |
|------|-----|
| Ward API 基础地址 | `http://localhost:8000` |
| 启动命令 | `screen -dmS ward /root/.venv/bin/ward` |
| 停止命令 | `screen -S ward -X quit` |

## 什么时候用

用户问以下类型问题时，调用 Ward API 获取数据：

- 美股三大指数行情：Nasdaq 综合、道琼斯、标普 500
- 黄金价格或黄金走势
- 个股价格、涨跌、成交量、PE、市值、52 周高低
- 指数、黄金、个股的 K 线和技术走势
- 指数 / 个股 / 市场整体 AI 分析报告
- 多只股票横向比较，例如“NVDA 和 TSLA 哪个更好”
- 盘前 / 盘中 / 盘后交易数据
- 当前市场情绪、科技股表现、某只股票近期走势

## 调用策略

1. **简单事实查询优先用同步数据接口**：价格、涨跌幅、K 线、盘前盘后、搜索。
2. **深度 AI 报告优先创建异步 Job**：个股分析、指数分析、市场报告、多股对比都可能较慢，使用 `/api/analysis-jobs/*` 创建任务，再查询状态或 Trace。
3. **用户已经在对话中给出多只股票并要求比较**：优先调用 `/api/chat/stream`，Ward 会识别多股对比意图并创建 `stock_comparison` Team 任务。
4. **需要排障、解释报告来源或查看执行过程**：用 Runtime Trace 接口查询 `job_id`。
5. **区分分析 prefix 和行情 symbol**：AI 分析使用 `prefix`：`ixic`、`dji`、`spx`、`gold`；K 线 HTTP 接口使用 Yahoo symbol，例如 `^IXIC`、`^DJI`、`^GSPC`、`GC=F`。

## API 速查

### 市场总览（三大指数 + 黄金）

```http
GET http://localhost:8000/api/market-overview
```

返回顶层字段：

- `nasdaq_composite`
- `nasdaq_100`
- `dow_jones`
- `sp500`
- `gold`

每个行情对象通常包含：`close`、`change`、`change_pct`、`open`、`high`、`low`、`volume` 等字段。

### 单项指数 / 黄金行情

```http
GET http://localhost:8000/api/quote
GET http://localhost:8000/api/ndx-quote
GET http://localhost:8000/api/dji-quote
GET http://localhost:8000/api/spx-quote
GET http://localhost:8000/api/gold-quote
```

### 个股搜索

```http
GET http://localhost:8000/api/stock/search?q=关键词
```

用于把用户输入的公司名或模糊代码转成具体美股 symbol。

### 个股行情

```http
GET http://localhost:8000/api/stock/{symbol}/quote
```

例如 `AAPL`、`NVDA`、`TSLA`。返回名称、现价、涨跌、PE、市值、52 周高低、分析师评级等。

### 个股历史 / K 线

```http
GET http://localhost:8000/api/stock/{symbol}/history?days=30
GET http://localhost:8000/api/stock/{symbol}/kline?days=60
```

返回每日 OHLCV 数据。个股使用 `AAPL`、`NVDA` 等 symbol；指数 / 黄金使用 Yahoo symbol：

- Nasdaq 综合：`%5EIXIC`（URL 编码后的 `^IXIC`）
- 道琼斯：`%5EDJI`（URL 编码后的 `^DJI`）
- 标普 500：`%5EGSPC`（URL 编码后的 `^GSPC`）
- 黄金：`GC%3DF`（URL 编码后的 `GC=F`）

不要把 `spx`、`ixic`、`dji` 这类分析 prefix 传给 K 线接口。

### 盘前 / 盘中 / 盘后数据

```http
GET http://localhost:8000/api/stock/{symbol}/extended
```

返回字段：`pre_market`、`regular`、`after_hours`、`previous_close`。盘前 / 盘后无数据时对应字段可能为 `null`。

指数盘前盘后可用 ETF 代理 symbol：

- Nasdaq：`QQQ`
- 标普 500：`SPY`
- 道琼斯：`DIA`

### 同步 AI 分析（兼容旧用法）

```http
GET http://localhost:8000/api/stock/{symbol}/analyze
GET http://localhost:8000/api/index/{prefix}/analyze
GET http://localhost:8000/api/report
```

指数 prefix：`ixic`（Nasdaq 综合）、`dji`（道琼斯）、`spx`（标普 500）、`gold`（黄金）。

这些接口会直接等待报告生成。若 Agent 所在平台容易超时，改用异步 Job 接口。

### 流式 AI 分析

```http
GET http://localhost:8000/api/stock/{symbol}/analyze/stream
GET http://localhost:8000/api/index/{prefix}/analyze/stream
GET http://localhost:8000/api/report/stream
```

返回 SSE。适合需要边生成边展示的 Agent。

### 异步分析 Job（推荐）

创建任务：

```http
POST http://localhost:8000/api/analysis-jobs/stock/{symbol}
POST http://localhost:8000/api/analysis-jobs/index/{prefix}
POST http://localhost:8000/api/analysis-jobs/report
```

返回：`{ok, job}`。`job.id` 是后续查询状态和 Trace 的关键。

查询任务状态：

```http
GET http://localhost:8000/api/analysis-jobs/{job_id}
```

订阅任务事件（SSE）：

```http
GET http://localhost:8000/api/analysis-jobs/{job_id}/events
```

任务状态通常是：

- `queued`：已进入队列
- `running`：正在执行
- `succeeded`：已完成
- `failed`：失败，查看 `error`

### Runtime Trace

```http
GET http://localhost:8000/api/analysis-jobs/{job_id}/trace
GET http://localhost:8000/api/runtime/stats?range=1d
```

Trace 可查看任务执行过程、缓存命中、prompt、模型原始返回、token、耗时、Verifier 结果等。`range` 可用 `1d`、`7d`、`30d`。

### 智能问答

非流式：

```http
POST http://localhost:8000/api/chat
Content-Type: application/json

{
  "conversation_id": null,
  "message": "用户问题",
  "context": {
    "indices": [],
    "stocks": [],
    "index_klines": {},
    "stock_klines": {},
    "stock_analyses": {},
    "index_analyses": {},
    "extended_hours": {}
  }
}
```

流式：

```http
POST http://localhost:8000/api/chat/stream
Content-Type: application/json
```

请求体同上，响应为 SSE。事件中可能包含：

- `chunk`：回答文本片段
- `thinking`：模型思考 / 进度信息
- `tool_call` / `tool_result`：工具调用状态
- `job`：后台 Job，例如多股对比 Team
- `assistant_message_id`：持久化消息 ID
- `done`：结束标记

取消进行中的对话：

```http
POST http://localhost:8000/api/chat/{conversation_id}/cancel
```

### 对话历史

```http
GET http://localhost:8000/api/history/{conversation_id}
GET http://localhost:8000/api/history/{conversation_id}/messages?limit=20&before_id={message_id}
```

第二个接口用于分页加载更早的消息。

## 多股对比

当用户问“NVDA、TSLA、AAPL 哪个更好”“比较 MSFT 和 GOOG”这类问题时，优先走智能问答流式接口：

```http
POST http://localhost:8000/api/chat/stream
Content-Type: application/json

{
  "conversation_id": null,
  "message": "比较 NVDA、TSLA、AAPL 的相对机会和风险"
}
```

Ward 会创建 `stock_comparison` 后台任务，返回 `job` 和 `/runtime?job_id=...`。随后可用 Job 查询接口轮询结果。

## 返回回答时的建议

- 报价类问题：直接给当前价、涨跌幅、成交量，并说明数据来源 Ward。
- 分析类问题：如果用了异步 Job，先告诉用户任务已创建和当前状态；完成后再总结报告。
- 多股对比：说明这是相对比较，不构成投资建议。
- Trace / Runtime 信息一般只在用户追问“为什么这么慢”“报告怎么来的”“查一下这个 job”时展示。

## 注意事项

1. 所有默认 API 调用都在 `localhost`。
2. AI 报告有缓存；相同数据、相同模型、短时间内可能复用结果。
3. 指数 / 黄金 AI 分析用 prefix：`ixic`、`dji`、`spx`、`gold`。
4. K 线接口用 symbol：个股用 `AAPL`，指数用 `%5EIXIC` / `%5EDJI` / `%5EGSPC`，黄金用 `GC%3DF`。
5. 盘前盘后数据可能为空。
6. 异步 Job 在服务重启后未完成任务会标记失败。
