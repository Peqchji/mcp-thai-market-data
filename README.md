# 🇹🇭 Thai Stock & Fund Analyzer — MCP Server

> An MCP (Model Context Protocol) server that gives Claude real-time access to Thai stock (SET/MAI) and mutual fund data — powered entirely by **free APIs**.

![Python](https://img.shields.io/badge/Python-3.10+-blue)
![MCP](https://img.shields.io/badge/MCP-Compatible-green)
![License](https://img.shields.io/badge/License-MIT-yellow)

---

## 📌 What Is This?

This project connects **Claude Desktop** to live Thai financial data using the [Model Context Protocol (MCP)](https://modelcontextprotocol.io). Instead of Claude guessing or giving outdated info, it calls your local server to fetch real data on demand.

**Example conversation after setup:**
```
You:    "How is PTT doing today? Compare it with CPALL."
Claude: → calls get_stock_price("PTT.BK") and get_stock_price("CPALL.BK")
Claude: "PTT is trading at ฿35.50 (-1.2%). CPALL is at ฿62.00 (+0.8%).
         CPALL is outperforming today. PTT's P/E ratio is..."
```

```
You:    "Which fund in my portfolio has the best 1-year return?"
Claude: → calls analyze_portfolio([...]) and get_fund_nav(...)
Claude: "Your best performer is KFSDIV-A with +12.4% over 12 months..."
```

---

## ✨ Features

| Tool | Description | Data Source |
|------|-------------|-------------|
| `get_stock_price` | Real-time price, change %, volume | Yahoo Finance (free) |
| `get_stock_info` | P/E, market cap, 52w high/low, dividend yield | Yahoo Finance (free) |
| `compare_stocks` | Side-by-side comparison of multiple SET stocks | Yahoo Finance (free) |
| `get_fund_nav` | Latest NAV, daily change | SEC Thailand API (free) |
| `get_fund_info` | Fund type, AMC, risk level, min investment | SEC Thailand API (free) |
| `compare_funds` | Side-by-side NAV + return comparison | SEC Thailand API (free) |
| `analyze_portfolio` | P&L, allocation %, best/worst performers | Yahoo Finance + SEC (free) |
| `search_funds` | Search Thai funds by name or AMC | SEC Thailand API (free) |

---

## 🆓 Free Data Sources

| Source | What It Covers | Rate Limit | Auth Required |
|--------|---------------|------------|---------------|
| [Yahoo Finance](https://finance.yahoo.com) via `yfinance` | SET/MAI stocks using `.BK` suffix (e.g. `PTT.BK`) | Generous, unofficial | ❌ None |
| [SEC Thailand Open API](https://api.sec.or.th/FundFactsheet/fund) | All registered Thai mutual funds, NAV history | Fair use | ❌ None |
| [SET Smart API](https://www.setsmart.com) *(optional)* | Official SET data — limited free tier | 100 req/day | ✅ Free registration |

> 💡 **No paid subscriptions required.** This project is designed to work 100% with free-tier and open APIs.

---

## 🗂️ Project Structure

```
thai-market-mcp/
├── server.py              # MCP server entry point
├── tools/
│   ├── __init__.py
│   ├── stocks.py          # SET stock tools (Yahoo Finance)
│   └── funds.py           # Mutual fund tools (SEC API)
├── utils/
│   ├── __init__.py
│   ├── formatter.py       # Format numbers, currencies (฿)
│   └── cache.py           # Simple in-memory cache (avoid rate limits)
├── tests/
│   ├── test_stocks.py
│   └── test_funds.py
├── requirements.txt
├── .env.example           # Environment variables template
└── README.md
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.10+
- [Claude Desktop](https://claude.ai/download) (free)
- Git

### 1. Clone the repo

```bash
git clone https://github.com/yourusername/thai-market-mcp.git
cd thai-market-mcp
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure Claude Desktop

Add this to your Claude Desktop config file:

**macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`  
**Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "thai-market": {
      "command": "python",
      "args": ["/absolute/path/to/thai-market-mcp/server.py"]
    }
  }
}
```

### 4. Restart Claude Desktop

You should see **thai-market** appear in Claude's tool list (🔧 icon).

---

## 📦 Dependencies

```txt
mcp>=1.0.0           # MCP Python SDK
yfinance>=0.2.36     # Yahoo Finance — Thai stocks via .BK tickers
requests>=2.31.0     # SEC Thailand API calls
python-dotenv>=1.0.0 # Environment variable management
```

Install with:
```bash
pip install -r requirements.txt
```

---

## 💬 Example Prompts

Once set up, try asking Claude:

**Stocks:**
- `"What is the current price of PTT, CPALL, and ADVANC?"`
- `"Compare the top 3 Thai bank stocks: BBL, KBANK, and SCB"`
- `"Show me PTT's P/E ratio and dividend yield"`
- `"Which of my stocks are in the red today?"`

**Funds:**
- `"Find me all Kasikorn Asset Management equity funds"`
- `"What is the latest NAV of KFSDIV-A?"`
- `"Compare these 3 funds: KFSDIV-A, SCBDV, TMBDIV"`
- `"Which fund type has the lowest risk for beginners?"`

**Portfolio:**
- `"I hold 1000 PTT at ฿32, 500 CPALL at ฿58, analyze my portfolio"`
- `"What percentage of my portfolio is in energy stocks?"`
- `"Show my total P&L and best/worst performer"`

---

## 🗺️ Roadmap

- [x] Phase 1: Project setup & Hello World MCP tool
- [x] Phase 2: Stock price tools (Yahoo Finance)
- [x] Phase 3: Fund NAV tools (SEC Thailand API)
- [ ] Phase 4: Portfolio analyzer
- [ ] Phase 5: Historical price charts (text-based sparklines)
- [ ] Phase 6: Fund screener (filter by risk level, return, AMC)
- [ ] Phase 7: SET index overview (SET, SET50, SET100)
- [ ] Phase 8: News sentiment (from free RSS feeds)

---

## 🏗️ Architecture

```
┌──────────────────┐      MCP (stdio)      ┌─────────────────────┐
│   Claude Desktop │ ◄───────────────────► │   server.py (MCP)   │
│   (AI client)    │   tool calls/results  │                     │
└──────────────────┘                       └──────────┬──────────┘
                                                      │
                              ┌───────────────────────┼──────────────────────┐
                              ▼                       ▼                      ▼
                     ┌──────────────┐      ┌──────────────────┐   ┌─────────────────┐
                     │ Yahoo Finance│      │  SEC Thailand    │   │  In-Memory      │
                     │  (yfinance)  │      │  Open API        │   │  Cache Layer    │
                     │  .BK stocks  │      │  /FundFactsheet  │   │  (rate limiting)│
                     └──────────────┘      └──────────────────┘   └─────────────────┘
```

---

## 🤝 Contributing

Pull requests welcome! Areas that need help:
- Adding SET index tracking
- Thai language support for fund names
- More portfolio analytics (Sharpe ratio, beta)
- Historical NAV charting

---

## 📄 License

MIT License — free to use, modify, and distribute.

---

## 👤 Author

Built as a portfolio project demonstrating MCP server development with real-world Thai financial data integration.

---

## ⚠️ Disclaimer

This tool is for **informational purposes only**. It is not financial advice. Always do your own research before making investment decisions. NAV and stock data may be delayed.
