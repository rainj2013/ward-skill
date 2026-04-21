---
name: hermes-ward
description: Ward US Market Data API 调用指南。任何 AI Agent（Hermes、OpenClaw、Claude Code 等）均可使用。Ward 跑在 localhost:8000，支持美股指数、个股行情、AI 分析报告、盘前盘后数据。
---

# hermes-ward — Ward API 调用指南

适用于任何 AI Agent（Hermes、OpenClaw、Claude Code 等）。Ward 跑在 `localhost:8000`，不暴露公网时通过本地代理转发访问。

## 基础信息

| 项目 | 值 |
|------|-----|
| Ward API 基础地址 | `http://localhost:8000` |
| 启动命令 | `screen -dmS ward /root/.venv/bin/ward` |
| 停止命令 | `screen -S ward -X quit` |

## 什么时候用

用户问以下类型问题时，调用 Ward API 获取数据：

- 美股三大指数行情（Nasdaq、道琼斯、标普500）
- 个股价格、涨跌、成交量
- 指数/个股的 AI 分析报告
- 盘前/盘后交易数据
- 用户直接问"帮我看看 NVDA 走势"
- 用户问"现在市场情绪怎么样"
- 任何美股相关的数据查询

## API 速查

### 市场总览（三大指数）

```
GET http://localhost:8000/api/market-overview
```

返回：`{ok, indices: [{name, symbol, close, change, change_pct, volume, high, low, open}]}`

### 个股行情

```
GET http://localhost:8000/api/stock/{symbol}/quote
```

例如 `AAPL`、`NVDA`、`TSLA`。返回：名称、现价、涨跌、PE、52周高低、分析师评级等。

### 个股历史 K 线

```
GET http://localhost:8000/api/stock/{symbol}/kline?days=30
```

返回每日 OHLCV 数据。指数 symbol 用 `^IXIC`（Nasdaq综合）、`^DJI`（道琼斯）、`^GSPC`（标普500）。

### 盘前/盘后数据

```
GET http://localhost:8000/api/stock/{symbol}/extended
```

返回：`{pre, regular, after, previous_close}`，pre/after 可能是 null（盘前/盘后无数据时）。

### AI 分析报告

```
GET http://localhost:8000/api/stock/{symbol}/analyze
GET http://localhost:8000/api/index/{prefix}/analyze
```

指数 prefix：`ixic`（Nasdaq综合）、`dji`（道琼斯）、`spx`（标普500）。
报告有缓存（5分钟 TTL），短时间内不会重复生成。

### 智能问答（带上下文）

```
POST http://localhost:8000/api/chat
Content-Type: application/json

{
  "message": "用户问题",
  "context": {
    "indices": [...],
    "stocks": [...],
    "index_klines": {},
    "stock_klines": {},
    "extended_hours": {}
  }
}
```

流式响应，SSE 格式。`context` 所有字段均为可选。

### 股票搜索

```
GET http://localhost:8000/api/stock/search?q=关键词
```

## 注意事项

1. **所有 API 调用都在 localhost**
2. **AI 报告有缓存**（TTL 5分钟），短时间内相同 symbol 不会重复生成
3. **指数 K 线 symbol 格式**：必须是 `^IXIC`/`^DJI`/`^GSPC`
4. **盘前盘后数据**可能为空
