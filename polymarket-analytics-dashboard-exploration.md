# Polymarket Analytics Dashboard: Deep Exploration

> A deep-dive into building a professional analytics dashboard for Polymarket prediction markets — what exists, what's missing, what problems it solves, and how it would work.

---

## Table of Contents

1. [The Opportunity](#the-opportunity)
2. [What Already Exists](#what-already-exists) — 170+ tools mapped, competitive landscape
3. [The Gap: What's Missing](#the-gap-whats-missing) — TradingView comparison, trader quotes
4. [What Problems Would This Solve?](#what-problems-would-this-solve) — 6 core problems
5. [Technical Architecture](#technical-architecture) — Full API ecosystem, architecture diagram
6. [Feature Breakdown](#feature-breakdown) — MVP → Intelligence → Advanced phases
7. [Business Model](#business-model) — Nansen/Arkham analysis, pricing tiers
8. [Build Roadmap](#build-roadmap) — 12-week implementation plan
9. [Risks and Open Questions](#risks-and-open-questions)
10. [Why This Is Worth Building](#why-this-is-worth-building)

---

## The Opportunity

Polymarket has crossed **$9B+ in trading volume** (2025), with ICE (NYSE parent) investing $2B at a $9B valuation in October 2025 and Jump Trading taking stakes in both Polymarket and Kalshi in February 2026. Average trade size grew from **$300** (early 2024) to **$4,800** (late 2025), reflecting institutional participation. The platform now attracts dedicated desks from Susquehanna (SIG), Jane Street, Jump Trading, and DRW.

**The thesis**: Polymarket has reached a scale where traders need professional-grade analytics, but the tools haven't kept up with the market's growth. The gap between what traders need and what exists is similar to crypto in 2019-2020 — right before Nansen and Arkham Intelligence emerged and built $75M+ ARR businesses.

The built-in Polymarket UI provides:
- Basic line charts (no candlesticks, no volume overlay)
- Total volume numbers
- Simple position tracking
- Category browsing

It does **not** provide: order book depth visualization, whale tracking, portfolio P&L analytics, cross-market correlation, order flow analysis, alerts, or any of the tools that professional traders expect from a serious trading platform.

---

## What Already Exists

The ecosystem is **far more developed** than it appears at first glance. The [DeFiPrime census (Jan 2026)](https://defiprime.com/definitive-guide-to-the-polymarket-ecosystem) counts **170+ tools** across 19 categories, and [Polymark.et](https://polymark.et/) catalogs 100+ tools with 97K users. Here's the competitive landscape:

### Tier 1: Major Analytics Platforms

| Tool | What It Does | Monetization |
|---|---|---|
| **[Polymarket Analytics](https://polymarketanalytics.com)** | Most comprehensive standalone: trader tracking, portfolio builder (multi-wallet), cross-platform (Polymarket + Kalshi), arbitrage detection, deposit/withdrawal monitoring. Powered by Goldsky, updates every 5 min. | Freemium |
| **[Hashdive](https://hashdive.com)** | "Smart Score" system (-100 to 100) rating traders by historical performance, open bets, consistency. Market screeners by liquidity/volume/behavior. | Freemium |
| **[PredictFolio](https://predictfolio.com)** | Real-time PnL, volume, win rate benchmarking against top wallets, search millions of traders by handle/market/performance. | Free |
| **[Polysights](https://polysights.xyz)** | AI-powered (Vertex AI + Gemini + Perplexity). 30+ custom metrics, up to 20 wallet tracking, Telegram alerts. | Tiered SaaS |
| **[PolymarketDash](https://polymarketdash.com)** | "Smart Money" / whale analysis. Historical PnL leaderboards, wallet win rates, event analytics, free Telegram bot (20 addresses). | Freemium |

### Tier 2: Trading Terminals

| Tool | What It Does | Notable |
|---|---|---|
| **[Oddpool](https://oddpool.com)** | Self-described "Bloomberg for prediction markets". Cross-venue (Polymarket, Kalshi, CME), arb scanner (762+ opportunities/week for Pro), whale tracking, 800+ markets. | Closest competitor |
| **[Verso](https://verso.trading)** | Institutional-grade Bloomberg-style interface. Real-time data + news for Polymarket and Kalshi. | Institutional pricing |
| **[Betmoar](https://betmoar.fun)** | Web-based terminal; ~$110M cumulative volume driven through it. Real-time news integration. | High volume |
| **[Stand](https://stand.trade)** | Multi-market terminal. Trade up to 8 markets simultaneously, copy trading, whale alerts. | Multi-market view |
| **[TradeFox](https://thetradefox.com)** | VC-backed aggregator/prime brokerage. Limit orders, advanced filters, volume rewards. | VC backing |

### Tier 3: Whale Trackers

| Tool | What It Does |
|---|---|
| **[Polywhaler](https://polywhaler.com)** | Leading dedicated whale tracker. $10K+ trade detection, insider activity detection. |
| **[PolyTrack](https://polytrackhq.app)** | 10,000+ traders tracked. Copy trading in 25ms. Free tier. |
| **[EdgeMarket](https://edgemarket.app)** | Whale tracking + Telegram alerts + Auto-Redeem SaaS. |
| **[MobyScreener](https://mobyscreener.com)** | Live feed of top traders' buys/sells in real time. |
| **[Unusual Whales](https://unusualwhales.com/predictions)** | Extension of the stock options flow tracker into prediction markets. |
| **[Polymarket Whale Tracker](https://chromewebstore.google.com/detail/polymarket-whale-tracker/onhhaghaecempnnodenjjlhkobgpkkfj)** | Chrome extension. Customizable thresholds ($10K-$100K). |

### Tier 4: AI Analysis Tools

| Tool | What It Does |
|---|---|
| **[Inside Edge](https://inside.fyi)** | AI market inefficiency detector. Paste any Polymarket URL for quantified edge + probability breakdown. |
| **[Polyfactual](https://polyfactual.com)** | ML-generated sentiment, risk, confidence, and signal analysis. |
| **[PolyRadar](https://polyradar.io)** | Multiple independent AI models for prediction analysis. |
| **[PolyOracle](https://polyoracle.com)** | Multi-LLM analysis of active markets. |
| **[Alphascope](https://alphascope.app)** | AI-driven market intelligence engine. |

### Tier 5: Specialized Tools

| Tool | What It Does |
|---|---|
| **[PolyScan](https://polyscan.bet)** | Etherscan-style on-chain explorer for Polymarket. Wallet lookups with all-time PnL. |
| **[Wethr](https://wethr.net)** | Advanced real-time weather analytics for Polymarket weather markets. |
| **[Polymarket Alerts](https://apps.apple.com/us/app/polymarket-alerts/id6748630806)** | Mobile app (iOS + Android) — price/whale/comment alerts, no account required. |
| **[ICE Polymarket Signals](https://www.marketsmedia.com/ice-launches-polymarket-signals-and-sentiment-tool/)** | Exclusive institutional data provider (Intercontinental Exchange). Enterprise tier. |

### Notable: Parsec (Shut Down)

**[Parsec](https://parsec.fi)** was the most institutional-grade real-time flow analytics tool — live trades, top holders, open interest charts, customizable dashboards. It [wound down operations after 5 years](https://www.theblock.co/post/390562/onchain-analytics-tool-parsec-winding-down-operations), leaving a significant gap for institutional-grade real-time flow analytics.

### Open-Source Projects (GitHub)

| Repo | Tech | What It Does |
|---|---|---|
| [polymarket-dashboard](https://github.com/buddies2705/polymarket-dashboard) | Next.js, React, SQLite, Bitquery | Market data, trades, token holders via Bitquery APIs |
| [polymarket-orderbook-viewer](https://github.com/Xemur/polymarket-orderbook-viewer) | React, TanStack Query, WebSocket | Live orderbook viewer, direct WS from frontend via Cloudflare CORS proxy |
| [polymarket-trade-tracker](https://github.com/leolopez007/polymarket-trade-tracker) | — | PnL tracking, Maker/Taker analysis, on-chain ops (Split, Merge, Redeem) |
| [polymarket-subgraph-analytics](https://github.com/PaulieB14/polymarket-subgraph-analytics) | GraphQL, The Graph | Real-time orderbook depth, spreads, cross-subgraph analytics guide |
| [polymarket-api](https://github.com/ryanschwarting/polymarket-api) | Next.js 14, Tailwind, Framer | Prediction markets explorer integrating Polymarket + Kalshi |
| [polybot](https://github.com/ent0n29/polybot) | Java 21, ClickHouse, Redpanda | Multi-service system: execution, strategy runtime, trade ingestion |
| [polymarket-kit](https://github.com/HuakunShen/polymarket-kit) | TypeScript, Elysia | Fully-typed SDK + CORS proxy server for CLOB and Gamma APIs |
| [polyterm](https://github.com/NYTEMODEONLY/polyterm) | — | Terminal-based monitoring: whale activity, insider patterns, arb opportunities |
| [poly_data](https://github.com/warproxxx/poly_data) | Python | Historical data pipeline with downloadable snapshots |

### 10+ Dune Analytics Dashboards

- [Polymarket Overview](https://dune.com/rchen8/polymarket) by rchen8
- [Activity & Volume](https://dune.com/filarm/polymarket-activity) by filarm
- [Capital & Whales 2025](https://dune.com/thxshogun/polymarket-2025-capital-and-whales) by thxshogun
- [Capital Flows & Users](https://dune.com/dune/polymarket-capital-flows-and-users) by Dune (official)
- [Cross-Platform Prediction Markets](https://dune.com/datadashboards/prediction-markets) by datadashboards
- [CLOB Stats](https://dune.com/lifewillbeokay/polymarket-clob-stats) by lifewillbeokay
- [Accuracy/Brier Score](https://dune.com/alexmccullough/how-accurate-is-polymarket) by alexmccullough

### Assessment: Where's the Actual Gap?

With 170+ tools, the obvious question is: **why build another one?**

The answer is that the ecosystem is **wide but shallow**:
- **Fragmentation is the #1 problem** — traders juggle 5+ tabs across different tools. No single product covers analytics + charts + whale tracking + alerts + portfolio
- **Parsec's shutdown leaves a void** — the closest thing to institutional-grade real-time flow analytics is gone
- **No market microstructure tools** — nobody provides VPIN, order flow toxicity, spread analytics, or maker/taker volume breakdown
- **Resolution risk is unquantified** — UMA governance disputes are a real concern (accusations of manipulation) but no tool quantifies resolution risk
- **99.49% of wallets are unprofitable** — retail traders desperately need analytics to level the playing field, but existing tools don't help them understand *why* they're losing
- **AI tools are surface-level** — current AI analysis tools provide generic summaries, not actionable signals with quantified edge
- **Historical depth is weak** — most tools are real-time only. Backtesting, calibration curves, and resolution accuracy analysis are underserved

**The gap isn't "no tools exist" — it's "no single tool does it well enough to become the default."**

---

## The Gap: What's Missing

### Side-by-Side Comparison: What Traders Expect vs What Exists

| Feature | TradingView / Bloomberg | Current Polymarket Tools |
|---|---|---|
| Candlestick charts | Multiple timeframes, 100+ indicators | Basic line chart only |
| Order book depth visualization | Heatmap + depth chart | Not visible |
| Volume overlay on charts | Session volume, volume profile | Total number only |
| Time & Sales (trade tape) | Real-time individual trades | Not available |
| Technical indicators (RSI, MACD, Bollinger) | 100+ built-in | None |
| Drawing tools (trendlines, Fibonacci) | Full suite | None |
| Market screener/scanner | Filter by any criteria | Category browsing only |
| Alerts (price, volume, pattern) | Price, indicator, webhook alerts | Very basic |
| Watchlists | Customizable multi-column | Basic favorites |
| Multi-chart layout | Up to 8 charts side by side | Single market view |
| Wallet/entity labels | N/A (Nansen/Arkham for crypto) | Crude on some tools |
| Smart money tracking | N/A (Nansen/Arkham) | Basic whale alerts |
| Portfolio P&L with cost basis | Standard on brokerages | Not available |
| Cross-market correlation | Built-in on Bloomberg | Not available |
| News correlation | Bloomberg terminal | Not available |
| Order flow toxicity (VPIN) | Institutional tools | Not available |
| Spread analytics (historical) | Built-in | Not available |
| Tax export | Standard on brokerages | Not available |

### What Traders Are Actually Saying

From Reddit, Twitter/X, and community discussions:

> "Polymarket's charts are useless — no candlestick charts, no volume bars, no order book depth"

> "I can't see who's buying — whale tracking is the most requested feature"

> "No way to compare my performance against the market or other traders"

> "I want TradingView but for prediction markets"

> "I trade on Polymarket AND Kalshi — I need one view"

> "How do I find alpha? Trending markets are just sorted by volume, not by signal"

---

## What Problems Would This Solve?

### Problem 1: "I Can't SEE the Market"

**Pain**: Polymarket shows a basic line chart. No candlesticks. No volume bars. No order book depth. No technical indicators. Traders from TradingView, crypto exchanges, or traditional finance find this unacceptable.

**Solution**: Professional charting — candlestick charts with configurable timeframes, volume overlay, order book depth heatmap, and basic technical indicators (moving averages, RSI, Bollinger bands).

**Who needs this**: Every active Polymarket trader (estimated ~50K monthly active users).

### Problem 2: "I Don't Know WHO Is Trading"

**Pain**: All Polymarket trades settle on Polygon and are fully public, but there's no easy way to identify who's behind large orders. Is this Jane Street accumulating? A whale insider? A new retail wallet?

**Solution**: Whale tracking with wallet labels (known entities like SIG, Jane Street, prominent traders), real-time large-trade alerts, and portfolio composition views for tracked wallets. Think Arkham Intelligence but purpose-built for prediction markets.

**Who needs this**: Traders who want to follow smart money, avoid adverse selection, or understand market dynamics.

### Problem 3: "I Can't FIND Opportunities"

**Pain**: Polymarket's discovery is volume-based sorting. No way to find markets with unusual activity, widening spreads, sudden volume spikes, or cross-platform price discrepancies.

**Solution**: Market scanner with filters for: volume spike %, spread change, price momentum, time-to-resolution, cross-platform arbitrage (Polymarket vs Kalshi price differences), and "unusual activity" alerts.

**Who needs this**: Active traders looking for edge, arbitrageurs, and market makers evaluating where to deploy capital.

### Problem 4: "I Can't MEASURE My Performance"

**Pain**: Polymarket shows current positions but no detailed P&L analytics. No cost basis tracking, no realized vs unrealized breakdown, no historical performance, no benchmarking against the market.

**Solution**: Portfolio analytics with: trade history, cost basis per position, realized/unrealized P&L, win rate, Sharpe ratio, ROI by market category, and CSV/tax export.

**Who needs this**: Anyone trading more than casually — especially for tax reporting.

### Problem 5: "I Can't PROTECT Myself from Informed Traders"

**Pain**: Market makers and active traders have no way to measure whether current order flow is informed (toxic) or noise. This is the core insight from the @gemchange_ltd thread — VPIN and order flow toxicity are critical but unavailable to retail.

**Solution**: VPIN indicator, order flow imbalance metrics, maker vs taker volume breakdown, and spread analytics. Alert when flow turns toxic ("don't trade right now, informed money is active").

**Who needs this**: Market makers, active traders who place limit orders.

### Problem 6: "I Can't COMPARE Across Platforms"

**Pain**: Traders using Polymarket, Kalshi, and Metaculus have no unified view. Prices for the same event can differ across platforms, but there's no tool to spot this.

**Solution**: Unified dashboard showing the same event across platforms with price comparison, spread overlay, and arbitrage alerts.

**Who needs this**: Cross-platform arbitrageurs, traders who want the best price.

---

## Technical Architecture

### The Polymarket API Ecosystem

A key finding: the Polymarket API surface is **remarkably comprehensive** and almost entirely **public, no-auth required**. An analytics dashboard can be built entirely on free, public endpoints.

#### API Services Overview

| Service | URL | Auth? |
|---|---|---|
| CLOB (trading, prices, order books) | `https://clob.polymarket.com` | No (read) / Yes (trade) |
| **Data API (positions, activity)** | `https://data-api.polymarket.com` | **No — fully public** |
| Gamma API (events, metadata) | `https://gamma-api.polymarket.com` | No |
| WebSocket (real-time market data) | `wss://ws-subscriptions-clob.polymarket.com/ws/` | No (market) / Yes (user) |
| RTDS (low-latency market data) | `wss://ws-live-data.polymarket.com` | Unknown |

#### CLOB API — Market Data (No Auth)

**Pricing & Order Book:**
- `GET /price?token_id=X&side=BUY` — Current price
- `GET /midpoint?token_id=X` — Mid-price (best bid + best ask / 2)
- `GET /spread?token_id=X` — Bid-ask spread
- `GET /book?token_id=X` — Full order book (bids/asks with price/size levels)
- `GET /last-trade-price?token_id=X` — Most recent trade
- `POST /books` — Batch: multiple order books in one call
- `POST /spreads` — Batch: multiple spreads by POST

**Historical:**
- `GET /prices-history?market=X&interval=1d&fidelity=100` — Price history per token
  - `interval`: `"max"`, `"1w"`, `"1d"`, `"6h"`, `"1h"`
  - `startTs` / `endTs`: Custom date range (Unix timestamps)
  - `fidelity`: Number of data points returned

**Market Info:**
- `GET /market` — Single market (condition ID, tokens, fees, tick size, neg_risk flag)
- `GET /markets` — All markets, paginated
- `GET /tick-size?token_id=X` — Min tick (changes when price > 0.96 or < 0.04)
- `GET /rewards/markets` — Maker reward rates per market

#### Data API — The Game-Changer (No Auth!)

This is the most underappreciated API. It lets you query **any wallet's positions and full trade history** with just their public address:

- `GET /positions?user=0x...` — Current positions for any wallet
  - Filter by `conditionIds` or `eventId`
  - Sortable by: `TOKENS`, `CURRENT`, `INITIAL`, `CASHPNL`, `PERCENTPNL`, `PRICE`
- `GET /activity?user=0x...` — Full trade/activity history
  - Filter by `type` (`TRADE`, `SPLIT`, `MERGE`, `REDEEM`, `REWARD`, `CONVERSION`)
  - Filter by `side` (`BUY`/`SELL`), `start`/`end` timestamps
- `GET /trades` — Complete trade history (100 per call, paginated)

**This means whale tracking and portfolio analytics can be built without any on-chain indexing** — the Data API already aggregates this data.

#### WebSocket — Real-Time Feeds

**Market Channel (public, no auth):**
- `book` — Full order book snapshot on subscribe + incremental updates
- `price_change` — When orders placed/cancelled (updated best bid/ask)
- `last_trade_price` — When a trade is matched
- `tick_size_change` — When min tick changes

Subscribe format: `{"type": "subscribe", "channel": "market", "assets_id": "<token_id>"}`

Requires ping/pong heartbeat every 30s. Connections drop periodically — reconnection + state reconciliation essential.

#### Gamma API — Market Metadata

Three-tier model: **Event** → **Market** → **Outcomes/Prices**

- `GET /events?active=true&closed=false` — Active, tradable markets
- `GET /events?closed=true` — Resolved markets (for historical analysis)
- `GET /markets?slug=fed-decision-in-october` — Market by slug
- `GET /tags` — All category tags
- Filterable by: `tag_id`, `liquidity`, `volume`, `start_date`, `end_date`

#### On-Chain Data (Polygon)

**Smart Contract Addresses (Polygon Mainnet):**

| Contract | Address |
|---|---|
| USDC.e | `0x2791Bca1f2de4661ED88A30C99A7a9449Aa84174` |
| Conditional Tokens (CTF) | `0x4D97DCd97eC945f40cF65F87097ACe5EA0476045` |
| CTF Exchange | `0x4bFb41d5B3570DeFd03C39a9A4D8dE6Bd8B8982E` |
| Neg Risk CTF Exchange | `0xC5d563A36AE78145C45a50134d48A1215220f80a` |

**Polymarket Subgraph**: [Official on The Graph](https://thegraph.com/explorer/subgraphs/81Dm16JjuFSrqz813HysXoUPvzTwE7fsfPk2RTf66nyC) — GraphQL for all markets, trades, positions. 100K queries/month free.

**Other indexers:** Goldsky (reorg-aware), Bitquery (GraphQL + Kafka streaming), Envio (multi-chain).

**RPC providers:** Alchemy (30M CU/month free), Infura, Ankr, public `polygon-rpc.com`.

#### Rate Limits

| Endpoint | Limit | Enforcement |
|---|---|---|
| Public REST | ~60 req/min | Cloudflare throttling (delayed, not dropped) |
| `/order` | 3,000 per 10-min window | — |
| WebSocket | Virtually unlimited | — |

#### CORS — Critical Constraint

**Polymarket APIs do NOT natively support browser CORS requests.** A backend proxy is required.

Community solution: [polymarket-kit](https://github.com/HuakunShen/polymarket-kit) — typed SDK + Elysia proxy with CORS headers. Or the [polymarket-orderbook-viewer](https://github.com/Xemur/polymarket-orderbook-viewer) approach using Cloudflare Workers.

#### Official Client Libraries

- **Python**: [py-clob-client](https://github.com/Polymarket/py-clob-client)
- **TypeScript**: `@polymarket/clob-client`
- **Rust**: [rs-clob-client](https://github.com/Polymarket/rs-clob-client)
- **Community**: `polymarket-apis` on PyPI (unified CLOB + Gamma + Data + WebSocket)

#### What Must Be Computed (Not Available via API)

| Feature | Available? | How to Build It |
|---|---|---|
| Candlestick (OHLCV) data | Must compute | Aggregate from trade stream into time buckets |
| VPIN (order flow toxicity) | Must compute | Classify trades as buy/sell, compute volume imbalance |
| Historical order book depth | Must capture | Record WebSocket `book` snapshots to your own DB |
| Cross-market correlation | Must compute | Store historical prices, compute rolling coefficients |
| Wallet labels | Must build/source | Community curation + on-chain heuristics + Arkham API |
| Spread history | Must capture | Record spread snapshots over time |
| Market making metrics | Must compute | Track order lifecycle: placed → filled/cancelled → P&L |
| Portfolio P&L | **Available!** | Data API `/positions?user=0x...` provides this directly |

#### Historical Data Sources

| Source | Resolution | Notes |
|---|---|---|
| CLOB `/prices-history` | Configurable interval | Official, supports custom date ranges |
| Data API `/trades`, `/activity` | Trade-level | Full history, paginated (100/call) |
| The Graph subgraph | Block-level (~2s) | On-chain, comprehensive GraphQL |
| Dune Analytics | Query-dependent | SQL-based, 10+ community dashboards |
| [PolymarketData.co](https://polymarketdata.co) | 1-minute | Professional archive, order book snapshots |
| [poly_data](https://github.com/warproxxx/poly_data) | Trade-level | Open-source pipeline, downloadable snapshots |
| [Kaggle dataset](https://www.kaggle.com/datasets/sandeepkumarfromin/full-market-data-from-polymarket) | Static | For offline analysis |
| [FinFeedAPI](https://finfeedapi.com) | OHLCV | Unified API covering Polymarket + Kalshi + others |

### Proposed Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                         FRONTEND (Next.js)                         │
│                                                                    │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────────────┐   │
│  │ Market   │  │ Charts   │  │ Whale    │  │ Portfolio        │   │
│  │ Scanner  │  │ (TradingView│ │ Tracker  │  │ Analytics        │   │
│  │          │  │  Lightweight)│ │          │  │                  │   │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────────────┘   │
│       │              │              │              │                │
│  ┌────┴──────────────┴──────────────┴──────────────┴───────────┐   │
│  │              React Query / TanStack Query                    │   │
│  │              (data fetching, caching, real-time sync)        │   │
│  └──────────────────────┬──────────────────────────────────────┘   │
│                          │                                         │
│  ┌───────────────────────┴─────────────────────────────────────┐   │
│  │              WebSocket Manager (reconnection, heartbeat)     │   │
│  └──────────────────────┬──────────────────────────────────────┘   │
└──────────────────────────┼─────────────────────────────────────────┘
                           │
                    HTTPS / WSS
                           │
┌──────────────────────────┼─────────────────────────────────────────┐
│                      BACKEND (Node.js)                             │
│                                                                    │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐  │
│  │ API Gateway  │  │ WebSocket    │  │ Background Workers       │  │
│  │ (REST proxy  │  │ Relay        │  │                          │  │
│  │  + caching)  │  │ (fan-out to  │  │  • Trade aggregator      │  │
│  │              │  │  clients)    │  │    (builds OHLCV candles) │  │
│  │              │  │              │  │  • VPIN calculator        │  │
│  │              │  │              │  │  • Whale detector         │  │
│  │              │  │              │  │  • Spread recorder        │  │
│  │              │  │              │  │  • Order book snapshotter │  │
│  │              │  │              │  │  • Polygon chain indexer  │  │
│  └──────┬───────┘  └──────┬───────┘  └──────────┬───────────────┘  │
│         │                 │                      │                  │
│  ┌──────┴─────────────────┴──────────────────────┴───────────────┐  │
│  │                     Data Layer                                 │  │
│  │                                                                │  │
│  │  PostgreSQL           Redis              TimescaleDB            │  │
│  │  (markets, wallets,   (real-time cache,  (time-series:          │  │
│  │   labels, users)      pub/sub)           trades, prices,        │  │
│  │                                          OHLCV, VPIN,           │  │
│  │                                          spreads, depth)        │  │
│  └────────────────────────────────────────────────────────────────┘  │
│                                                                    │
│  ┌────────────────────────────────────────────────────────────────┐  │
│  │                  External Data Sources                         │  │
│  │                                                                │  │
│  │  Polymarket CLOB API    Polymarket Gamma API    Polygon RPC    │  │
│  │  (REST + WebSocket)     (market metadata)       (on-chain)     │  │
│  │                                                                │  │
│  │  Dune Analytics API     News APIs               Kalshi API     │  │
│  │  (historical on-chain)  (NewsAPI, GNews)        (cross-platform)│ │
│  └────────────────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────────────┘
```

### Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **Frontend** | Next.js 14+ (App Router) | SSR for SEO, React ecosystem, API routes |
| **Charts** | TradingView Lightweight Charts (open source) | Industry standard, free, candlestick/volume/depth built-in |
| **State/Data** | TanStack Query (React Query) | Caching, real-time sync, stale-while-revalidate |
| **Styling** | Tailwind CSS + shadcn/ui | Fast iteration, dark mode, consistent design |
| **Backend** | Node.js (API routes or separate Express) | Same language as frontend, good WebSocket support |
| **Database** | PostgreSQL + TimescaleDB extension | Time-series for trades/prices, relational for metadata |
| **Cache** | Redis | Real-time pub/sub, WebSocket fan-out, rate limiting |
| **Realtime** | WebSocket (native or Socket.IO) | Sub-second updates for charts and order book |
| **On-chain indexing** | Alchemy SDK + ethers.js | Polygon event subscriptions, wallet tracking |
| **Deployment** | Vercel (frontend) + Railway/Fly.io (backend + DB) | Cost-effective, auto-scaling |

---

## Feature Breakdown

### Phase 1: Core Dashboard (MVP)

#### 1.1 Market Browser
- List active markets from Gamma API with search, filter by category/tag, sort by volume/liquidity/recency
- Show key stats: current price, 24h change, volume, liquidity, spread, time to resolution
- "Trending" and "Unusual Activity" sections

#### 1.2 Professional Charts
Using **TradingView Lightweight Charts** (free, open source):
- Candlestick charts with configurable timeframes (1m, 5m, 15m, 1h, 4h, 1d)
- Volume bars overlay
- Moving averages (SMA, EMA)
- Price history from `/prices-history` endpoint, augmented by computed OHLCV from trade stream

```
Data flow for charts:
  1. Historical: GET /prices-history → seed chart
  2. Real-time: WebSocket "market" channel → append new candles
  3. Compute: Aggregate raw trades into OHLCV buckets (1m, 5m, etc.)
  4. Store: TimescaleDB for historical OHLCV (not available from API)
```

#### 1.3 Order Book Depth Chart
- Real-time order book from WebSocket `book` channel
- Depth chart visualization (cumulative bids/asks)
- Highlight large orders (potential support/resistance)

#### 1.4 Trade Tape (Time & Sales)
- Real-time stream of individual trades
- Color-coded buy vs sell (classify using trade price vs mid-price)
- Aggregate large trades visually

### Phase 2: Intelligence Layer

#### 2.1 Whale Tracker
- Index Polygon on-chain data to identify wallets with large Polymarket positions
- Label known entities (Jane Street, SIG, Jump, prominent community traders)
- Real-time alerts when labeled wallets trade
- Wallet detail page: portfolio composition, historical P&L, win rate

```
Data flow for whale tracking:
  1. Polygon RPC: Subscribe to PolymarketCTFExchange contract events
  2. Filter: Trades above threshold (e.g., >$5K)
  3. Enrich: Look up wallet label from DB
  4. Store: PostgreSQL (wallet profiles, trade history)
  5. Alert: Push via WebSocket to frontend + Telegram bot
```

#### 2.2 Market Scanner
- Filter markets by: volume spike %, price momentum, spread width, time to resolution
- "Unusual Activity" detector: flag markets where volume or price movement significantly exceeds recent baseline
- Watchlist with custom alerts

#### 2.3 Portfolio Analytics
- Connect wallet (read-only via address) or import from on-chain data
- Cost basis per position, realized/unrealized P&L
- Historical performance (equity curve)
- Win rate, average win/loss, Sharpe ratio
- CSV export for tax reporting

### Phase 3: Advanced Analytics

#### 3.1 VPIN (Order Flow Toxicity)
The key insight from the @gemchange_ltd thread — measuring whether order flow is informed:

```
VPIN calculation (rolling window):
  1. Classify each trade as buy or sell (tick rule or quote rule)
  2. Group trades into volume buckets (e.g., 50 buckets of equal volume)
  3. For each bucket: compute |buy_volume - sell_volume| / total_volume
  4. VPIN = mean of bucket imbalances over rolling window

VPIN near 0 = balanced flow (safe to quote)
VPIN near 1 = heavily directional (informed trading — pull quotes)
```

Display as a real-time gauge on the market detail page. Alert when VPIN crosses threshold.

#### 3.2 Spread Analytics
- Historical bid-ask spread chart (must capture from WebSocket)
- Spread percentile vs historical (is current spread unusually wide/tight?)
- Maker vs taker volume breakdown

#### 3.3 Cross-Market Correlation
- For related markets (e.g., "Trump wins" vs "GOP wins Senate"), show rolling correlation coefficient
- Cross-platform comparison (Polymarket vs Kalshi prices for the same event)
- Arbitrage alert when price difference exceeds threshold

#### 3.4 News Impact Analysis
- Monitor news APIs (NewsAPI, GNews, RSS)
- Match headlines to active markets via keyword/NLP matching
- Show news timeline alongside price chart
- Track: which news sources move which markets?

---

## Business Model

### How Successful Crypto Analytics Platforms Monetize

| Platform | Model | Revenue | Key Moat |
|---|---|---|---|
| **Nansen** | SaaS ($150-$2,500/mo) | $75M+ ARR | Wallet labels, smart money tracking |
| **Arkham** | Freemium + ARKM token | Token economics | 50M+ labeled addresses, intel exchange |
| **DeBank** | Freemium + NFT access | Growing | Best multi-chain portfolio view, social |
| **Dune** | Freemium ($349/mo+) | Growing | Community SQL dashboards, network effects |

**Common success patterns:**
1. Free tier to build user base → Premium for power features
2. Community/social features → Network effects, viral growth
3. Unique data that's hard to get elsewhere (labels, classifications, computed metrics)
4. API access for programmatic users (higher-tier plans)
5. Alert systems that create daily engagement and habit formation
6. Content/research that drives organic traffic and SEO

### Proposed Pricing

| Tier | Price | Features |
|---|---|---|
| **Free** | $0 | Market browser, basic line charts, top-10 whale alerts, 3 watchlist markets |
| **Pro** | $19/mo | Candlestick charts, full order book depth, unlimited watchlist, whale tracker with labels, portfolio P&L, custom alerts, spread analytics |
| **Trader** | $49/mo | VPIN indicator, cross-market correlation, market scanner, cross-platform view (Kalshi), news impact overlay, CSV/tax export, API access (1K calls/day) |
| **API** | $99/mo | Full API access (10K calls/day), WebSocket feed, historical data downloads, webhook integrations |

### Revenue Estimate

- Polymarket has ~50K monthly active traders (conservative estimate based on on-chain data)
- If 5% convert to Pro ($19) and 1% to Trader ($49):
  - 2,500 Pro × $19 = $47,500/mo
  - 500 Trader × $49 = $24,500/mo
  - 50 API × $99 = $4,950/mo
  - **Total: ~$77K MRR / ~$920K ARR**
- With market growth and institutional adoption: potential $1-5M ARR
- Not venture-scale on its own, but **viable as a bootstrapped or small-team product**

---

## Build Roadmap

### Week 1-2: Foundation
- [ ] Next.js project setup with Tailwind + shadcn/ui (dark mode by default)
- [ ] Polymarket API client library (CLOB REST + Gamma API)
- [ ] Market browser page (list/search/filter markets from Gamma API)
- [ ] Basic market detail page with price from CLOB API
- [ ] WebSocket connection manager (connect to Polymarket WS, handle reconnection/heartbeat)

### Week 3-4: Charts & Order Book
- [ ] Integrate TradingView Lightweight Charts
- [ ] Build OHLCV aggregator (trade stream → candle buckets)
- [ ] Historical price chart from `/prices-history`
- [ ] Real-time price updates via WebSocket
- [ ] Order book depth chart from WebSocket `book` channel
- [ ] Trade tape (time & sales) component

### Week 5-6: Data Pipeline
- [ ] Set up PostgreSQL + TimescaleDB
- [ ] Background workers: trade aggregator, spread recorder, order book snapshotter
- [ ] Redis for real-time pub/sub and caching
- [ ] Polygon chain indexer for whale detection (large trade threshold)
- [ ] Basic wallet profile storage

### Week 7-8: Intelligence
- [ ] Whale tracker page (large trades, wallet profiles)
- [ ] Market scanner (volume spikes, unusual activity)
- [ ] Watchlist with custom alerts
- [ ] Portfolio analytics (connect wallet, P&L tracking)

### Week 9-10: Advanced
- [ ] VPIN calculator and real-time gauge
- [ ] Spread analytics (historical spread chart)
- [ ] Cross-market correlation view
- [ ] Telegram bot for alerts

### Week 11-12: Polish & Launch
- [ ] Authentication (email magic link or wallet connect)
- [ ] Stripe billing integration (free/pro/trader tiers)
- [ ] Mobile responsive design
- [ ] Landing page, documentation
- [ ] Beta launch to Polymarket community (Twitter, Reddit, Discord)

---

## Risks and Open Questions

### Technical Risks

1. **Rate limiting** — Polymarket API rate limits are not officially documented. Heavy polling could get throttled. Mitigation: aggressive caching, WebSocket-first architecture, backend proxy.

2. **WebSocket reliability** — Polymarket WebSocket connections drop periodically. Need robust reconnection + state reconciliation (re-fetch order book snapshot after reconnect).

3. **OHLCV data gap** — Polymarket does not provide candlestick data. Must be computed from trade stream. If the system goes down, there's a gap in historical candles. Mitigation: backfill from on-chain data via Dune/subgraph.

4. **Order book history** — Only current snapshots available. Historical depth requires continuous capture. Storage costs scale with number of markets monitored × snapshot frequency.

5. **Trade classification** — VPIN requires classifying trades as "buy" or "sell". The standard methods (tick rule, Lee-Ready) may be less reliable on a CLOB vs traditional exchanges.

### Business Risks

1. **Small TAM** — Prediction market traders are a fraction of crypto traders. The addressable market may be too small for a standalone product. Counter: prediction markets are growing rapidly, and the TAM is expanding.

2. **Regulatory uncertainty** — Polymarket's legal status varies by jurisdiction. A regulatory crackdown could shrink the user base. Mitigation: multi-platform support (Kalshi is CFTC-regulated).

3. **Cyclical engagement** — Trading activity spikes around major events (elections) and drops between. User retention and MRR may be volatile. Mitigation: diversify across event types (politics, sports, weather, crypto).

4. **Free alternatives** — Dune dashboards and basic whale trackers may be "good enough" for casual traders. The premium features must provide clear, demonstrable value over free options.

5. **Platform risk** — Dependence on Polymarket's API. If they build these analytics features natively (or change their API), the product loses its value prop. Mitigation: multi-platform from early on.

### Open Questions

1. **CORS in production** — Polymarket APIs do NOT support browser CORS. Backend proxy is required. Solved problem — polymarket-kit and Cloudflare Workers approaches exist.

2. **Wallet labeling** — How to build a useful label database? Options: (a) manual community curation, (b) license from Arkham/Nansen, (c) on-chain heuristics (cluster analysis, known contract interactions). This is the hardest part and potentially the biggest moat.

3. **Cross-platform feasibility** — Kalshi has a documented REST + WebSocket API at `trading-api.kalshi.com`. Metaculus is prediction-focused with a simpler API. FinFeedAPI offers unified coverage.

4. **Historical data depth** — `/prices-history` supports custom date ranges via `startTs`/`endTs`. For deeper history, supplement with Data API `/trades` (paginated), The Graph subgraph, or poly_data snapshots.

5. **Real-time vs batch** — Real-time critical: charts, order book, whale alerts. Acceptable at 1-min+: portfolio P&L, market scanner, VPIN (typically computed over volume buckets, not tick-by-tick).

6. **Differentiation in a 170+ tool ecosystem** — The gap is integration, not features. No single tool combines professional charts + whale tracking + microstructure analytics + portfolio. The opportunity is to be the first "all-in-one" that eliminates tab-hell.

7. **Resolution risk** — UMA governance disputes are a real user concern. No tool quantifies dispute risk. Could this be a differentiating feature? (Track UMA voting patterns, historical dispute outcomes, resolution delay probability.)

---

## Why This Is Worth Building

The Polymarket analytics dashboard sits at the intersection of several trends:

1. **Prediction markets are going mainstream** — ICE's $9B valuation, Jump Trading's investment, and growing retail participation signal this isn't a niche anymore.

2. **The "picks and shovels" strategy works** — In every trading ecosystem, the analytics tooling companies (Bloomberg, TradingView, Nansen) capture value more reliably than the traders themselves. As @gemchange_ltd's own thread argues: building tools is more viable than trying to compete as a market maker against Jane Street.

3. **The gap is real and painful** — Active traders consistently ask for features that don't exist. The current tooling is fragmented, low-quality, and missing critical capabilities.

4. **The tech is accessible** — The Polymarket API is comprehensive, well-structured, and free. The core technologies (Next.js, TradingView Charts, PostgreSQL, WebSocket) are well-understood. No exotic infrastructure required.

5. **The data moat is buildable** — Wallet labels, historical computed metrics (OHLCV, VPIN, spread history), and cross-platform normalization create compounding value over time. The longer you run, the more valuable your historical data becomes.

The honest caveat: this is not a venture-scale opportunity today (~$1M ARR potential). But it's a viable bootstrapped product, a strong portfolio piece, and positioned to grow with the prediction market ecosystem.

---

## Resources

### Official Polymarket Documentation
- [CLOB API Introduction](https://docs.polymarket.com/developers/CLOB/introduction)
- [Public API Methods](https://docs.polymarket.com/developers/CLOB/clients/methods-public)
- [API Reference (Interactive)](https://docs.polymarket.com/api-reference/introduction)
- [WebSocket Overview](https://docs.polymarket.com/developers/CLOB/websocket/wss-overview)
- [Gamma API Overview](https://docs.polymarket.com/developers/gamma-markets-api/overview)
- [How to Fetch Markets](https://docs.polymarket.com/developers/gamma-markets-api/fetch-markets-guide)
- [Data API: Get Positions](https://docs.polymarket.com/developers/misc-endpoints/data-api-get-positions)
- [Data API: Get Activity](https://docs.polymarket.com/developers/misc-endpoints/data-api-activity)
- [Contract Addresses](https://docs.polymarket.com/resources/contract-addresses)
- [Blockchain Data Resources](https://docs.polymarket.com/developers/builders/blockchain-data-resources)
- [Rate Limits](https://docs.polymarket.com/quickstart/introduction/rate-limits)
- [Authentication](https://docs.polymarket.com/developers/CLOB/authentication)

### Client Libraries
- [py-clob-client](https://github.com/Polymarket/py-clob-client) (official Python)
- [@polymarket/clob-client](https://github.com/Polymarket/clob-client) (official TypeScript)
- [rs-clob-client](https://github.com/Polymarket/rs-clob-client) (official Rust)
- [polymarket-kit](https://github.com/HuakunShen/polymarket-kit) (community: typed SDK + CORS proxy)
- `polymarket-apis` on PyPI (community: unified CLOB + Gamma + Data + WS)

### Frontend Libraries
- [TradingView Lightweight Charts](https://tradingview.github.io/lightweight-charts/) (open source, free)
- [TanStack Query](https://tanstack.com/query) (data fetching, caching)
- [shadcn/ui](https://ui.shadcn.com) (UI components)

### Data Sources
- [Polymarket Subgraph on The Graph](https://thegraph.com/explorer/subgraphs/81Dm16JjuFSrqz813HysXoUPvzTwE7fsfPk2RTf66nyC)
- [PolymarketData.co](https://polymarketdata.co) (1-min resolution, order book snapshots)
- [poly_data](https://github.com/warproxxx/poly_data) (open-source historical pipeline)
- [FinFeedAPI](https://finfeedapi.com) (unified API: Polymarket + Kalshi + Manifold)
- [Kaggle Polymarket Dataset](https://www.kaggle.com/datasets/sandeepkumarfromin/full-market-data-from-polymarket)
- [Kalshi API](https://trading-api.kalshi.com/docs)
- [Dune Analytics](https://dune.com/docs/api)

### Competitive Research
- [Polymark.et](https://polymark.et/) — tool directory (100+ tools, 97K users, 18 categories)
- [LaunchPoly](https://launchpoly.com) — community-rated tool discovery
- [DeFiPrime Ecosystem Guide](https://defiprime.com/definitive-guide-to-the-polymarket-ecosystem) — "170+ Tools" census
- [Awesome-Prediction-Market-Tools](https://github.com/aarora4/Awesome-Prediction-Market-Tools) (GitHub)
- [Awesome-Polymarket-Tools](https://github.com/harish-garg/Awesome-Polymarket-Tools) (GitHub)
- [Parsec Shutdown Article](https://www.theblock.co/post/390562/onchain-analytics-tool-parsec-winding-down-operations)
- [Blockworks Polymarket Analytics](https://blockworks.com/analytics/polymarket)

### Open-Source Reference Projects
- [polymarket-dashboard](https://github.com/buddies2705/polymarket-dashboard) (Next.js + Bitquery)
- [polymarket-orderbook-viewer](https://github.com/Xemur/polymarket-orderbook-viewer) (React + WS + CF Workers)
- [polymarket-trade-tracker](https://github.com/leolopez007/polymarket-trade-tracker) (PnL analysis)
- [polymarket-subgraph-analytics](https://github.com/PaulieB14/polymarket-subgraph-analytics) (analytics guide)
- [polymarket-api](https://github.com/ryanschwarting/polymarket-api) (Next.js 14 + Kalshi)
- [polybot](https://github.com/ent0n29/polybot) (Java microservices, ClickHouse, monitoring)
- [Polymarket Agents](https://github.com/Polymarket/agents) (official AI agent framework)

### Background Reading
- [Polymarket Market Making Exploration](./polymarket-market-making-exploration.md) — companion doc with A-S model details
- [@gemchange_ltd's original thread](https://x.com/gemchange_ltd/status/2025908468633268456)
- [Automated Market Making on Polymarket](https://news.polymarket.com/p/automated-market-making-on-polymarket)
