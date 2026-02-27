# TinkClaw API & Integration Guide

**Quant Intelligence for Trading Bots**

TinkClaw delivers fractal regime detection, information flow analysis, and multi-signal confluence scoring via REST. Plug into your existing data pipeline and start making smarter trades.

- **Website**: [tinkclaw.com](https://tinkclaw.com)
- **Base URL**: `https://api.tinkclaw.com/v1`
- **Authentication**: API key via `X-API-Key` header

---

## Table of Contents

1. [Quick Start](#quick-start)
2. [Authentication](#authentication)
3. [Rate Limits](#rate-limits)
4. [Endpoints](#endpoints)
   - [Trading Signals](#get-v1signals)
   - [ML-Enhanced Signals](#get-v1signals-ml)
   - [Market Analysis](#get-v1analysis)
   - [Market Summary](#get-v1market-summary)
   - [Quantitative Analysis](#get-v1quant)
   - [Technical Indicators](#get-v1indicators)
   - [Risk Metrics](#get-v1quantrisk-metrics)
   - [Hurst History](#get-v1quanthurst-history)
   - [Price Charts](#get-v1quantprice-chart)
   - [Correlation Matrix](#get-v1quantcorrelation)
   - [Asset Screener](#get-v1screener)
   - [News Feed](#get-v1news)
   - [Global Indices](#get-v1indices)
   - [Backtesting](#post-v1backtest)
   - [Confluence Score](#get-v1confluence)
   - [Webhook Subscribe](#post-v1webhookssubscribe)
   - [Webhook List](#get-v1webhooks)
   - [Usage Stats](#get-v1usage)
   - [Key Info](#get-v1api-keysinfo)
   - [Key Rotation](#post-v1api-keysrotate)
5. [Edge Caching](#edge-caching)
6. [Integration Examples](#integration-examples)
7. [Error Handling](#error-handling)
8. [SDKs & Libraries](#sdks--libraries)
9. [FAQ](#faq)

---

## Quick Start

Get your first signal in under 60 seconds.

### 1. Get an API Key

Sign up at [tinkclaw.com](https://tinkclaw.com) — the Explorer tier gives you 100 requests/day, no credit card required.

### 2. Make Your First Request

```bash
curl -H "X-API-Key: YOUR_KEY" \
  "https://api.tinkclaw.com/v1/signals?symbols=BTC,ETH"
```

### 3. Parse the Response

```json
{
  "signals": [
    {
      "symbol": "BTC",
      "signal": "BUY",
      "confidence": 78,
      "price": 68087.76,
      "target": 74896.54,
      "stop_loss": 64683.37,
      "rsi": 34.79,
      "timestamp": "2026-02-22T14:30:00Z"
    }
  ],
  "plan": "free",
  "calls_remaining": 499
}
```

That's it. You now have actionable signals flowing into your bot.

---

## Authentication

All API requests require an `X-API-Key` header.

```
X-API-Key: tinkclaw_free_a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6
```

Keys follow the format `tinkclaw_<plan>_<32-char-hex>`. Your key is generated automatically after checkout and can be retrieved from your dashboard.

**Never expose your API key in client-side code.** Always make API calls from your server or bot backend.

---

## Rate Limits

| Plan | Daily Limit | Price | Best For |
|------|------------|-------|----------|
| **Explorer** | 100 | $0 | Testing & prototyping |
| **Builder** | 5,000 | $29/mo | Production bots & single-strategy |
| **Pro** | 50,000 | $99/mo | Multi-strategy & high-volume |
| **Enterprise** | Unlimited | Custom | Institutional & white-label |

Annual billing: 20% off ($79/mo for Pro, $23/mo for Builder). No credit card required for Explorer tier.

Rate limits reset at **midnight UTC** daily. Every response includes headers to track your usage and cache status:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 82
X-RateLimit-Reset: 2026-02-24T00:00:00Z
X-Cache: HIT
```

When you exceed your limit, you'll receive a `429` response:

```json
{
  "error": "Rate limit exceeded",
  "reset_at": "2026-02-23T00:00:00Z"
}
```

---

## Endpoints

### `GET /v1/signals`

Trading signals based on technical indicators (RSI, SMA, price action).

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `symbols` | string | `BTC,ETH` | Comma-separated asset symbols |

**Response**

```json
{
  "signals": [
    {
      "symbol": "BTC",
      "signal": "BUY",
      "confidence": 78,
      "price": 68087.76,
      "target": 74896.54,
      "stop_loss": 64683.37,
      "rsi": 34.79,
      "price_vs_sma20": -1.45,
      "timestamp": "2026-02-22T14:30:00Z",
      "data_source": "tinkclaw-quant"
    }
  ],
  "plan": "free",
  "calls_remaining": 499,
  "disclaimer": "Technical indicators for informational use. Not financial advice."
}
```

**Signal Values**: `BUY` | `SELL` | `HOLD`

**Confidence**: 0-100 (higher = stronger conviction)

---

### `GET /v1/signals-ml`

AI-enhanced signals combining technical indicators with machine learning predictions.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `symbols` | string | `BTC,ETH` | Comma-separated asset symbols |

**Response**

```json
{
  "signals": [
    {
      "symbol": "BTC",
      "recommendation": "BUY",
      "confidence": 82,
      "source": "consensus",
      "baseline": {
        "signal": "BUY",
        "confidence": 78
      },
      "ml": {
        "prediction": "BUY",
        "confidence": 85,
        "probabilities": {
          "BUY": 0.85,
          "SELL": 0.10,
          "HOLD": 0.05
        }
      },
      "price": 68087.76,
      "target": 74896.54,
      "stop_loss": 64683.37,
      "timestamp": "2026-02-22T14:30:00Z"
    }
  ],
  "ml_enabled": true
}
```

**Signal Source**: `consensus` (ML agrees with baseline) | `ml` (ML overrides) | `baseline` (ML unavailable)

---

### `GET /v1/analysis`

Detailed technical analysis including trend, support/resistance, Bollinger Bands, and recommendations. Supports batch queries.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `symbol` | string | `BTC` | Single asset symbol |
| `symbols` | string | — | Comma-separated symbols for batch (up to 10). Overrides `symbol`. |

**Response**

```json
{
  "symbol": "BTC",
  "current_price": 68087.76,
  "trend": "BULLISH",
  "support": 63456.23,
  "resistance": 74892.11,
  "rsi": 34.79,
  "ema_12": 67823.45,
  "ema_26": 68234.56,
  "sma_20": 68423.10,
  "price_vs_sma20_pct": -1.45,
  "bollinger_bands": {
    "upper": 74892.11,
    "middle": 68234.56,
    "lower": 63456.23
  },
  "recommendation": "BUY",
  "timestamp": "2026-02-22T14:30:00Z"
}
```

**Trend**: `BULLISH` (EMA12 > EMA26) | `BEARISH` (EMA12 < EMA26)

**Recommendation**: `STRONG BUY` | `BUY` | `HOLD` | `SELL` | `STRONG SELL`

---

### `GET /v1/market-summary`

Overview of all 34 supported assets across crypto, forex, and commodities with sentiment data.

**Response**

```json
{
  "crypto": [
    { "symbol": "BTC", "price": 68087.76, "change_24h": 2.34, "volume": 28932491232 },
    { "symbol": "ETH", "price": 3200.00, "change_24h": -0.82, "volume": 14521300000 }
  ],
  "forex": [
    { "symbol": "EUR/USD", "price": 1.0842, "change_24h": 0.12 }
  ],
  "commodities": [
    { "symbol": "GOLD", "price": 2034.50, "change_24h": 0.45 }
  ],
  "sentiment": {
    "overall": "neutral",
    "fear_greed_index": 52
  },
  "timestamp": "2026-02-22T14:30:00Z"
}
```

---

### `GET /v1/quant`

Fractal regime detection using MFDFA (Multifractal Detrended Fluctuation Analysis) and Hurst exponent.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `symbol` | string | `BTC` | Asset symbol |

**Response**

```json
{
  "symbol": "BTC",
  "hurst_exponent": 0.62,
  "regime": "trending",
  "mfdfa": {
    "alpha_min": 0.45,
    "alpha_max": 0.78,
    "width": 0.33,
    "asymmetry": 0.12
  },
  "interpretation": "Market is in a trending regime (H > 0.5). Momentum strategies favored.",
  "timestamp": "2026-02-22T14:30:00Z"
}
```

**Hurst Exponent Interpretation**:
- `H < 0.5` — Mean-reverting (range-bound strategies)
- `H = 0.5` — Random walk (no clear edge)
- `H > 0.5` — Trending (momentum strategies)

---

### `GET /v1/indicators`

Pre-calculated technical indicators: RSI, MACD, Bollinger Bands, SMA, EMA, ATR, and more.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `symbol` | string | `BTC` | Asset symbol |
| `range` | string | `30d` | Time range (`7d`, `30d`, `90d`) |

**Response**

```json
{
  "symbol": "BTC",
  "indicators": {
    "rsi_14": 34.79,
    "macd": { "value": 1023.45, "signal": 956.78, "histogram": 66.67 },
    "bollinger": { "upper": 74892.11, "middle": 68234.56, "lower": 63456.23 },
    "sma_20": 68423.10,
    "sma_50": 66234.23,
    "ema_12": 67823.45,
    "ema_26": 68234.56,
    "atr_14": 1234.56,
    "adx": 45.67,
    "volume_sma": 24123012301,
    "obv": 1823412312
  }
}
```

---

### `GET /v1/quant/risk-metrics`

Portfolio risk metrics: Sharpe, Sortino, VaR, CVaR, and maximum drawdown.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `symbol` | string | `BTC` | Asset symbol |

**Response**

```json
{
  "symbol": "BTC",
  "sharpe_ratio": 1.84,
  "sortino_ratio": 2.12,
  "var_95": -0.023,
  "cvar_95": -0.034,
  "max_drawdown": -0.12,
  "volatility_30d": 0.045,
  "beta": 1.0
}
```

---

### `GET /v1/quant/hurst-history`

Rolling Hurst exponent timeseries for regime change detection.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `symbol` | string | `BTC` | Asset symbol |
| `range` | string | `30d` | Time range |

**Response**

```json
{
  "symbol": "BTC",
  "history": [
    { "date": "2026-01-23", "hurst": 0.58, "regime": "trending" },
    { "date": "2026-01-24", "hurst": 0.47, "regime": "mean_reverting" },
    { "date": "2026-01-25", "hurst": 0.51, "regime": "random_walk" }
  ]
}
```

---

### `GET /v1/quant/price-chart`

OHLCV candle data for charting.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `symbol` | string | `BTC` | Asset symbol |
| `interval` | string | `1d` | Candle interval (`1h`, `4h`, `1d`) |
| `range` | string | `30d` | Time range |

**Response**

```json
{
  "symbol": "BTC",
  "candles": [
    {
      "timestamp": "2026-02-22T00:00:00Z",
      "open": 67500.00,
      "high": 68500.00,
      "low": 67200.00,
      "close": 68087.76,
      "volume": 28932491232
    }
  ]
}
```

---

### `GET /v1/quant/correlation`

Cross-asset correlation matrix.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `symbols` | string | `BTC,ETH,SOL` | Comma-separated (2-10 assets) |
| `range` | string | `30d` | Time range |

**Response**

```json
{
  "matrix": {
    "BTC": { "BTC": 1.0, "ETH": 0.87, "SOL": 0.72 },
    "ETH": { "BTC": 0.87, "ETH": 1.0, "SOL": 0.81 },
    "SOL": { "BTC": 0.72, "ETH": 0.81, "SOL": 1.0 }
  },
  "period": "30d"
}
```

---

### `GET /v1/screener`

All 34 assets with key metrics and Hurst exponent at a glance.

**Response**

```json
{
  "assets": [
    {
      "symbol": "BTC",
      "price": 68087.76,
      "change_24h": 2.34,
      "rsi": 34.79,
      "hurst": 0.62,
      "regime": "trending",
      "signal": "BUY",
      "confluence_score": 78
    }
  ],
  "total": 34
}
```

---

### `GET /v1/news`

Multi-source RSS news feed with sentiment analysis.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `category` | string | `all` | `crypto`, `forex`, `commodities`, `macro`, `all` |

**Response**

```json
{
  "articles": [
    {
      "title": "Bitcoin Breaks Key Resistance Level",
      "source": "CoinDesk",
      "url": "https://...",
      "sentiment": "positive",
      "sentiment_score": 0.82,
      "published_at": "2026-02-22T12:00:00Z"
    }
  ]
}
```

---

### `GET /v1/indices`

Major global stock indices.

**Response**

```json
{
  "indices": [
    { "symbol": "SPX", "name": "S&P 500", "value": 5234.12, "change": 0.45 },
    { "symbol": "DJI", "name": "Dow Jones", "value": 39234.56, "change": 0.32 },
    { "symbol": "IXIC", "name": "NASDAQ", "value": 16789.01, "change": 0.67 }
  ]
}
```

---

### `POST /v1/backtest`

Run a backtest with built-in or custom strategies against historical data.

**Request Body**

```json
{
  "symbol": "BTC",
  "strategy": "hurst_momentum",
  "start_date": "2025-01-01",
  "end_date": "2026-02-01",
  "initial_capital": 10000
}
```

**Available Strategies**: `hurst_momentum` | `mean_reversion` | `ma_crossover` | `buy_and_hold`

**Response**

```json
{
  "strategy": "hurst_momentum",
  "symbol": "BTC",
  "period": { "start": "2025-01-01", "end": "2026-02-01" },
  "results": {
    "total_return": 0.43,
    "annual_return": 0.38,
    "sharpe_ratio": 1.84,
    "max_drawdown": -0.12,
    "win_rate": 0.60,
    "total_trades": 47,
    "profit_factor": 1.92
  }
}
```

---

### `GET /v1/confluence`

6-layer weighted confluence score combining technical, sentiment, on-chain, macro, information flow, and quantitative signals. Includes ATR-based volatility bands. Supports batch queries.

**Parameters**

| Name | Type | Default | Description |
|------|------|---------|-------------|
| `symbol` | string | `BTC` | Asset symbol |
| `symbols` | string | — | Comma-separated symbols for batch (up to 10). Overrides `symbol`. |

**Response**

```json
{
  "symbol": "BTC",
  "score": 87,
  "score_band": { "low": 83, "high": 91 },
  "volatility": "moderate",
  "atr_pct": 2.14,
  "layers": {
    "technical": 90,
    "sentiment": 82,
    "on_chain": 88,
    "macro": 85,
    "flow": 91,
    "quant": 86
  },
  "recommendation": "STRONG BUY",
  "timestamp": "2026-02-23T12:00:00.000Z"
}
```

**Score Interpretation**:
- **80-100**: Strong signal (high conviction)
- **60-79**: Moderate signal
- **40-59**: Weak/neutral
- **0-39**: Contrarian territory

**Volatility Regimes**: `low` | `moderate` | `high` | `extreme`

**Score Band**: ATR-adjusted range showing where the score could move based on current volatility.

---

### `POST /v1/webhooks/subscribe`

Subscribe to price, confluence, or RSI alerts. TinkClaw will POST to your callback URL when conditions trigger.

**Request Body**

```json
{
  "url": "https://your-server.com/webhook",
  "event": "confluence_above",
  "symbol": "BTC",
  "threshold": 80
}
```

**Available Events**: `price_above` | `price_below` | `confluence_above` | `confluence_below` | `rsi_above` | `rsi_below`

**Response**

```json
{
  "id": "wh_a1b2c3d4",
  "status": "active",
  "event": "confluence_above",
  "symbol": "BTC",
  "threshold": 80,
  "url": "https://your-server.com/webhook",
  "created_at": "2026-02-23T12:00:00Z"
}
```

---

### `GET /v1/webhooks`

List your active webhook subscriptions.

**Response**

```json
{
  "webhooks": [
    {
      "id": "wh_a1b2c3d4",
      "event": "confluence_above",
      "symbol": "BTC",
      "threshold": 80,
      "url": "https://your-server.com/webhook",
      "status": "active",
      "last_triggered": "2026-02-23T14:30:00Z"
    }
  ]
}
```

---

### `GET /v1/usage`

Your API key usage stats and remaining quota.

**Response**

```json
{
  "plan": "free",
  "daily_limit": 500,
  "calls_today": 18,
  "calls_remaining": 482,
  "reset_at": "2026-02-24T00:00:00Z"
}
```

---

### `GET /v1/api-keys/info`

Get metadata about your API key: plan, status, daily usage, and remaining quota.

**Response**

```json
{
  "key_prefix": "tinkclaw_free_a8",
  "plan": "free",
  "status": "active",
  "created_at": 1771951188186,
  "daily_limit": 500,
  "used_today": 18,
  "remaining": 482,
  "reset_at": "2026-02-28T00:00:00.000Z"
}
```

**Status Values**: `active` | `deprecated` (within 24h rotation grace period)

---

### `POST /v1/api-keys/rotate`

Rotate your API key. Generates a new key with the same plan and metadata. The old key remains valid for **24 hours** to give you time to update your integrations.

**Request**: No body required. Authenticates with your current `X-API-Key`.

**Response**

```json
{
  "success": true,
  "api_key": "tinkclaw_free_de56886c978a4e3096bc2e66fb8a8a25",
  "plan": "free",
  "message": "New API key issued. Old key will remain active for 24 hours."
}
```

**Important**: Save the new `api_key` immediately — it is only returned once. Update your bot configuration within 24 hours.

---

## Edge Caching

All GET endpoints are served via Cloudflare's global edge network. Responses are cached per-region for ultra-low latency.

| Endpoint Type | Cache TTL |
|---------------|-----------|
| Confluence, Indicators | 30 seconds |
| Quant, Risk, Screener, Correlation | 60 seconds |

Every response includes an `X-Cache` header:

- `HIT` — Served from edge cache (~5ms)
- `MISS` — Fetched from backend, now cached (~300-800ms)
- `BYPASS` — Non-cacheable request (POST, etc.)

Edge caching is automatic — no configuration needed. All tiers get the same caching performance.

---

## Integration Examples

### Python

```python
import requests

API_KEY = "tinkclaw_free_YOUR_KEY_HERE"
BASE_URL = "https://api.tinkclaw.com/v1"

headers = {"X-API-Key": API_KEY}

# Get signals for BTC and ETH
response = requests.get(
    f"{BASE_URL}/signals",
    headers=headers,
    params={"symbols": "BTC,ETH"}
)
data = response.json()

for signal in data["signals"]:
    print(f"{signal['symbol']}: {signal['signal']} "
          f"(confidence: {signal['confidence']}%)")

# Check remaining calls
remaining = response.headers.get("X-RateLimit-Remaining")
print(f"Calls remaining today: {remaining}")
```

### Node.js

```javascript
const API_KEY = "tinkclaw_free_YOUR_KEY_HERE";
const BASE_URL = "https://api.tinkclaw.com/v1";

async function getSignals(symbols = "BTC,ETH") {
  const response = await fetch(
    `${BASE_URL}/signals?symbols=${symbols}`,
    { headers: { "X-API-Key": API_KEY } }
  );

  if (!response.ok) {
    if (response.status === 429) {
      const { reset_at } = await response.json();
      console.log(`Rate limited. Resets at ${reset_at}`);
      return null;
    }
    throw new Error(`API error: ${response.status}`);
  }

  return response.json();
}

// Usage
const data = await getSignals("BTC,SOL,ETH");
data.signals.forEach(s =>
  console.log(`${s.symbol}: ${s.signal} (${s.confidence}%)`)
);
```

### Go

```go
package main

import (
    "encoding/json"
    "fmt"
    "net/http"
)

const (
    apiKey  = "tinkclaw_free_YOUR_KEY_HERE"
    baseURL = "https://api.tinkclaw.com/v1"
)

type SignalResponse struct {
    Signals []struct {
        Symbol     string  `json:"symbol"`
        Signal     string  `json:"signal"`
        Confidence int     `json:"confidence"`
        Price      float64 `json:"price"`
    } `json:"signals"`
    CallsRemaining int `json:"calls_remaining"`
}

func getSignals(symbols string) (*SignalResponse, error) {
    req, _ := http.NewRequest("GET",
        fmt.Sprintf("%s/signals?symbols=%s", baseURL, symbols), nil)
    req.Header.Set("X-API-Key", apiKey)

    resp, err := http.DefaultClient.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()

    var data SignalResponse
    json.NewDecoder(resp.Body).Decode(&data)
    return &data, nil
}
```

### Trading Bot Example (Python)

A minimal bot that checks signals every hour and logs trade decisions:

```python
import requests
import time
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("tinkclaw-bot")

API_KEY = "tinkclaw_free_YOUR_KEY_HERE"
BASE = "https://api.tinkclaw.com/v1"
HEADERS = {"X-API-Key": API_KEY}
WATCHLIST = "BTC,ETH,SOL"

def check_signals():
    """Fetch signals and act on high-confidence ones."""
    resp = requests.get(f"{BASE}/signals-ml", headers=HEADERS,
                        params={"symbols": WATCHLIST})
    resp.raise_for_status()
    data = resp.json()

    for signal in data["signals"]:
        sym = signal["symbol"]
        action = signal["recommendation"]
        conf = signal["confidence"]

        if conf >= 75:
            logger.info(f"HIGH CONFIDENCE: {sym} → {action} ({conf}%)")
            # execute_trade(sym, action, signal["price"],
            #               signal["target"], signal["stop_loss"])
        else:
            logger.debug(f"{sym} → {action} ({conf}%) — skipping")

    logger.info(f"Calls remaining: {data['calls_remaining']}")

if __name__ == "__main__":
    while True:
        try:
            check_signals()
        except requests.exceptions.HTTPError as e:
            if e.response.status_code == 429:
                logger.warning("Rate limited. Sleeping until reset.")
                time.sleep(3600)
            else:
                raise
        time.sleep(3600)  # Check every hour
```

---

## Error Handling

| Status | Meaning | Action |
|--------|---------|--------|
| `200` | Success | Parse response body |
| `401` | Invalid or missing API key | Check your `X-API-Key` header |
| `403` | Forbidden | Endpoint requires a higher plan |
| `429` | Rate limit exceeded | Wait until `reset_at` time |
| `500` | Server error | Retry with exponential backoff |
| `503` | Backend unavailable | Retry after 30 seconds |

**Recommended retry strategy:**

```python
import time

def api_call_with_retry(url, headers, max_retries=3):
    for attempt in range(max_retries):
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            return response.json()
        if response.status_code == 429:
            reset = response.json().get("reset_at")
            print(f"Rate limited. Resets at {reset}")
            return None
        if response.status_code >= 500:
            wait = 2 ** attempt  # 1s, 2s, 4s
            time.sleep(wait)
            continue
        response.raise_for_status()
    raise Exception("Max retries exceeded")
```

---

## SDKs & Libraries

### Python SDK (Official)

```bash
pip install tinkclaw
```

```python
from tinkclaw import TinkClawClient

client = TinkClawClient(api_key="tinkclaw_free_YOUR_KEY")

# Get trading signals
signals = client.get_signals(["BTC", "ETH"])
for s in signals:
    print(f"{s['symbol']}: {s['signal']} ({s['confidence']}%)")

# Check key info
info = client.key_info()
print(f"Plan: {info['plan']}, Remaining: {info['remaining']}")

# Rotate key (old key valid for 24h)
result = client.rotate_key()
new_client = TinkClawClient(api_key=result['api_key'])
```

Full docs: [pypi.org/project/tinkclaw](https://pypi.org/project/tinkclaw/) | [GitHub](https://github.com/TinkClaw/tinkclaw-python)

### OpenAPI Specification

The complete API is defined in [`openapi.yaml`](openapi.yaml) (OpenAPI 3.0). Use it to auto-generate a typed client in any language:

```bash
# Python
pip install openapi-python-client
openapi-python-client generate --url https://raw.githubusercontent.com/TinkClaw/docs/main/openapi.yaml

# TypeScript
npx openapi-typescript-codegen --input openapi.yaml --output ./tinkclaw-client

# Go
go install github.com/oapi-codegen/oapi-codegen/v2/cmd/oapi-codegen@latest
oapi-codegen -package tinkclaw openapi.yaml > tinkclaw.go

# Rust, Java, C#, Ruby, etc.
npx @openapitools/openapi-generator-cli generate -i openapi.yaml -g <language> -o ./client
```

You can also import `openapi.yaml` directly into [Postman](https://www.postman.com/), [Insomnia](https://insomnia.rest/), or [Swagger UI](https://swagger.io/tools/swagger-ui/) for interactive testing.

The REST API works with any HTTP client — no SDK required. See the [integration examples](#integration-examples) below.

---

## Supported Assets (34)

### Crypto (20)
BTC, ETH, SOL, ADA, DOT, AVAX, MATIC, LINK, UNI, ATOM, XRP, BNB, DOGE, SHIB, LTC, BCH, XLM, ALGO, FTM, NEAR

### Forex (8)
EUR/USD, GBP/USD, USD/JPY, AUD/USD, USD/CAD, NZD/USD, USD/CHF, EUR/GBP

### Commodities (6)
GOLD, SILVER, OIL, NATGAS, COPPER, WHEAT

---

## FAQ

**Is this financial advice?**
No. TinkClaw provides quantitative analysis and data for informational purposes only. Always do your own research.

**What data sources do you use?**
We use a proprietary multi-source pipeline to compute indicators, sentiment, and on-chain metrics.

**How fresh is the data?**
All endpoints are served via global edge caching. Confluence and indicators are cached for 30 seconds, quant and risk data for 60 seconds. Check the `X-Cache` response header (`HIT` = served from edge, `MISS` = fresh from backend).

**Can I use custom strategies with backtesting?**
Currently 4 built-in strategies are available. Custom strategy support is on the roadmap. Enterprise customers can request early access.

**Do you offer refunds?**
No cash refunds are issued. You may cancel your subscription at any time — your access continues until the end of your current billing period. Any unused portion is converted to API credit, which is automatically applied if you resubscribe within 12 months. See the Cancellation & Billing Policy below for details.

**What happens when I cancel?**
Your API key remains active until the end of the current billing cycle. After that, it downgrades to the free Explorer tier (100 requests/day). No data is deleted.

---

## Cancellation & Billing Policy

**Effective: February 27, 2026**

1. **No Cash Refunds** — All payments are final and non-refundable once processed. By subscribing, you acknowledge that you are purchasing immediate access to a digital service.

2. **Cancel Anytime** — You may cancel your subscription at any time by contacting api@tinkclaw.com. Cancellation takes effect at the end of your current billing period — you retain full access until then.

3. **Prorated API Credit** — When you cancel mid-cycle, the unused portion of your subscription is calculated on a daily pro-rata basis and issued as API credit to your account. Credits are non-transferable and expire 12 months from the date of issuance.

4. **Credit Application** — API credits are automatically applied to your next subscription payment if you resubscribe within the 12-month validity window. Credits cannot be redeemed for cash.

5. **Free Tier Downgrade** — After your paid billing period ends, your API key automatically downgrades to the Explorer tier (100 requests/day). Your usage history and data remain intact.

6. **Billing Cycle** — Subscriptions are billed monthly from the date of activation. There are no annual commitments or long-term contracts.

7. **Disputes** — If you believe you were charged in error, contact api@tinkclaw.com within 30 days of the charge. We will investigate and resolve billing disputes promptly.

---

## Architecture

```
Your Bot
   │
   ▼ HTTPS
┌──────────────────┐
│   Edge Gateway    │  ← Auth, rate limiting, edge caching (sub-50ms)
│  (Cloudflare)     │     X-Cache: HIT/MISS on every response
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│  Quant Engine     │  ← MFDFA, indicators, confluence
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│   ML Service      │  ← Signal enhancement, prediction
└──────────────────┘
```

---

## License

This documentation is provided under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/). The TinkClaw API itself is a commercial product — see [tinkclaw.com](https://tinkclaw.com) for terms.

---

**Questions?** Open an issue in this repo or reach out at [tinkclaw.com](https://tinkclaw.com).
