# Polymarket Algorithmic Market Making: Implementation & Use Cases

> Exploration inspired by [@gemchange_ltd's thread](https://x.com/gemchange_ltd/status/2025908468633268456) on Avellaneda-Stoikov market making on Polymarket prediction markets.

---

## Table of Contents

1. [What Is This About?](#what-is-this-about)
2. [How Polymarket Works Under the Hood](#how-polymarket-works-under-the-hood)
3. [The Avellaneda-Stoikov Market Making Model](#the-avellaneda-stoikov-market-making-model)
4. [Adapting for Binary Prediction Markets](#adapting-for-binary-prediction-markets)
5. [The Full "Applied Polymarket Stack"](#the-full-applied-polymarket-stack)
6. [Implementation: How It Works in Code](#implementation-how-it-works-in-code)
7. [Real-World Strategies That Made Money](#real-world-strategies-that-made-money)
8. [The Polymarket Ecosystem: 170+ Tools](#the-polymarket-ecosystem-170-tools)
9. [What Can You Use This For?](#what-can-you-use-this-for)
10. [Risks and Limitations](#risks-and-limitations)
11. [Resources](#resources)

---

## What Is This About?

@gemchange_ltd (founder of @coldvisionXYZ) published a long-form X Article titled **"How Jump Trading, Jane Street, and a Guy With $10K Fighting Over the Same Polymarket Order Book"** (February 21, 2026 — 79.2K views, 722 likes, 1.3K bookmarks).

The article explains that **most retail traders on Polymarket are unknowingly trading against sophisticated algorithmic market makers** running math from the Avellaneda-Stoikov model — a framework developed in 2008 for optimal market making in electronic markets.

The key insight: while retail traders chase narratives and gut feelings, quantitative firms like Jane Street (20-person desk, sub-100ms execution) and Jump Trading deploy bots that mathematically compute optimal bid/ask quotes, manage inventory risk, and capture the spread — consistently extracting value from the market.

The article breaks down into three parts:
- **Part I: The Machine You're Trading Against** — The competitive landscape: Jump Trading, institutional infrastructure, and why the "$10K + Python era is dead"
- **Part II: The Reservation Price** — The single most important number in market making: not the observed mid-price, but the market maker's own adjusted internal valuation
- **Part III: The Bounded Inventory Problem** — How market makers manage inventory risk on binary outcome markets that settle at 0 or 1

Key data points from the article:
- Arbitrage opportunity duration shrunk from **12.3 seconds** (2024) to **2.7 seconds** (2026)
- **73%** of arbitrage profits captured by sub-100ms execution bots
- Median arbitrage spread: just **0.3%** (barely profitable after gas fees)
- Python systems: **250-500 microseconds** per message; Rust/C++ HFT systems: **~12 microseconds** (Python is 20-40x slower)

As [@Kropanchik reacted](https://x.com/Kropanchik/status/2025730323342876944):
> "This article just explained why building a profitable Polymarket bot in 2026 is nearly IMPOSSIBLE. The math checks out — Jump Trading, 20-person desk, sub-100ms execution. Arbitrage windows down from 12 seconds to 2.7 seconds. The $10K + Python era is dead."

---

## How Polymarket Works Under the Hood

### Architecture

Polymarket is a **prediction market** built on Polygon (Ethereum L2) that uses a **Central Limit Order Book (CLOB)** — not an AMM like Uniswap. This is a crucial distinction:

| Component | Description |
|-----------|-------------|
| **CLOB** | A traditional order book matching engine (operated by Polymarket) — limit orders, market orders, bid/ask spread |
| **CTF Tokens** | Conditional Token Framework (Gnosis) — each market has YES and NO outcome tokens that settle at $1 or $0 |
| **USDC Settlement** | All trades are denominated in USDC (stablecoin). You buy YES shares at e.g. $0.65, and if the event happens, they settle at $1.00 |
| **On-chain Settlement** | Token minting/redemption happens on-chain (Polygon), but the order matching happens off-chain for speed |

### Three-Layer Architecture

| Layer | Components |
|-------|-----------|
| **Application Layer** | Web Frontend (React), Mobile, Third-party Apps |
| **Service Layer** | CLOB API, Data API, Gamma API |
| **Protocol Layer** | CTF Exchange, CTF Core, USDC Token |

### How Trading Works

```
1. Market: "Will X happen by date Y?"
2. You can buy YES tokens (betting it will happen) or NO tokens (betting it won't)
3. YES price + NO price ≈ $1.00 (with small spread)
4. At resolution: winning tokens → $1.00, losing tokens → $0.00
5. Profit = $1.00 - purchase_price (if you were right)
```

### Three Core Trade Operations

1. **Direct Match (Token Matching)**: User-to-user trade with no minting or burning
2. **Minting (Split)**: When orders for opposite outcomes match in price (e.g., YES at $0.70 + NO at $0.30 = $1.00), their combined $1.00 USDC is locked as collateral, and a new pair of YES and NO tokens is minted
3. **Merging (Burn)**: Two sell orders for opposite tokens can be matched — the pair is burned, and $1.00 USDC collateral is released

### The CLOB API

Polymarket exposes three APIs and a WebSocket for programmatic trading:

| API | Purpose | Auth Required |
|-----|---------|---------------|
| **Gamma API** | Market metadata and discovery (find markets, descriptions, categories) | None |
| **CLOB API** | Trading operations (order book, placing/canceling orders) | API keys + EIP-712 |
| **Data API** | User-specific data (positions, trade history) | API keys |

```
Base URL: https://clob.polymarket.com

Read Endpoints (No Auth):
  GET  /price?token_id=<id>       — Best bid/ask for a token
  GET  /book?token_id=<id>        — Full order book
  GET  /midpoint?token_id=<id>    — Midpoint price
  GET  /spread?token_id=<id>      — Bid-ask spread

Write Endpoints (Auth Required):
  POST /order                      — Place an order (signed with API key)
  DELETE /order/<id>               — Cancel an order

WebSocket: wss://ws-subscriptions-clob.polymarket.com/ws/market
  — Real-time order book updates, trades, price changes

Rate Limits:
  Public API: 100 requests/minute
  Trading endpoints: 60 orders/minute

Order Types:
  GTC (Good-Til-Cancelled) — stays on book until filled or cancelled
  GTD (Good-Til-Date)      — active until a specified UTC timestamp
  FOK (Fill-or-Kill)       — market order, fills immediately or cancels
```

Orders are signed using an API key + secret derived from your Polygon wallet. The system uses EIP-712 typed data signing. Authentication has two levels:
- **L1 Headers**: For creating or deriving API credentials
- **L2 Headers**: For all trading operations (still requires private key for EIP-712 payload signing)

---

## The Avellaneda-Stoikov Market Making Model

### The Problem

A market maker continuously quotes bid and ask prices, earning the spread between them. But they face two key risks:

1. **Inventory Risk** — As you buy/sell, you accumulate a directional position. If the price moves against you, your inventory loses value.
2. **Adverse Selection** — Informed traders (who know something you don't) will pick off your stale quotes, causing systematic losses.

### Core Mathematical Framework

The 2008 paper by Marco Avellaneda and Sasha Stoikov provides optimal closed-form solutions.

#### 1. The Reservation Price (r)

The market maker's "true" internal price, adjusted for inventory:

```
r(s, q, t) = s - q * γ * σ² * (T - t)
```

Where:
- `s` = current mid-price
- `q` = current inventory (positive = long, negative = short)
- `γ` = risk aversion parameter (how much you hate holding inventory)
- `σ²` = price variance (volatility squared)
- `T - t` = time remaining until end of trading horizon

**Intuition**: If you're long (q > 0), your reservation price drops below the mid-price — you want to sell, so you quote lower to attract sellers. If you're short, the opposite.

#### 2. The Optimal Spread (δ)

```
δ(q, t) = γ * σ² * (T - t) + (2/γ) * ln(1 + γ/k)
```

Where:
- `k` = order arrival intensity parameter (related to market activity)
- Other variables as above

The optimal bid and ask quotes are then:

```
bid = r - δ/2
ask = r + δ/2
```

#### 3. The Key Insight

The model dynamically adjusts quotes based on:
- **Inventory**: The more inventory you hold, the more aggressively you skew quotes to reduce it
- **Volatility**: Higher volatility → wider spreads (more risk per trade)
- **Time**: As the trading horizon approaches, the market maker becomes more aggressive about unwinding inventory
- **Activity**: More active markets allow tighter spreads

---

## Adapting for Binary Prediction Markets

Polymarket outcomes are binary (YES/NO → $1/$0), which creates unique challenges for the standard Avellaneda-Stoikov model:

### Key Differences from Traditional Markets

| Traditional Market | Binary Prediction Market |
|---|---|
| Price can go to any value | Price bounded [0, 1] |
| Continuous price movement | Jumps to 0 or 1 at settlement |
| Volatility is somewhat stable | Volatility increases dramatically near events |
| No inherent terminal value | Terminal value is exactly 0 or 1 |

### Adaptations Required

#### 1. Bounded Price Space
Prices are capped at [0, 1]. The model must respect these bounds — you never bid above $1 or offer below $0. Standard Avellaneda-Stoikov can suggest prices outside valid ranges.

```python
# Clip quotes to valid range
bid = max(0.01, min(0.99, reservation_price - spread/2))
ask = max(0.01, min(0.99, reservation_price + spread/2))
```

#### 2. Modified Volatility Model
In binary markets, volatility is a function of the price itself (prices near 0.5 are most volatile, near 0 or 1 are least):

```python
# Binary market volatility estimate
sigma = price * (1 - price)  # Maximum at 0.5, minimum near boundaries
```

#### 3. Inventory Limits
Since outcomes are binary, holding too much of one side is extremely risky. Maximum position sizes must be enforced.

#### 4. Event-Driven Risk
Unlike traditional markets, prediction markets have "resolution events" where the price jumps to 0 or 1. The model must account for the probability and timing of these events.

---

## The Full "Applied Polymarket Stack"

As described in @gemchange_ltd's thread, professional Polymarket market makers use a layered stack:

### 1. Avellaneda-Stoikov (Core Quoting Engine)
The base quoting model, adapted for binary settlement as described above. Determines reservation price and optimal spread.

### 2. GLFT Inventory Bounds
**Guéant-Lehalle-Fernandez-Tapia (GLFT)** extended the Avellaneda-Stoikov model to handle bounded inventory. Instead of letting inventory grow unboundedly, GLFT adds hard constraints:

```
Key idea: Set maximum inventory bounds [-Q_max, +Q_max]
When inventory approaches bounds → widen spread dramatically on the
dangerous side, narrow on the reducing side
```

This prevents catastrophic losses from over-accumulation.

### 3. Glosten-Milgrom Adverse Selection Model
Handles the problem of informed traders. The model assumes some fraction of traders have private information:

```
When a buy order arrives → update probability upward (buyer may know something)
When a sell order arrives → update probability downward (seller may know something)

Adjust quotes based on estimated probability of trading against informed flow.
```

In Polymarket, this is critical because insider knowledge about real-world events (elections, policy decisions) creates genuine information asymmetry.

### 4. VPIN Kill Switches
**Volume-Synchronized Probability of Informed Trading (VPIN)** measures the toxicity of order flow in real-time:

```python
# VPIN calculation (simplified)
buy_volume = sum(volume where price_change > 0)
sell_volume = sum(volume where price_change < 0)
vpin = abs(buy_volume - sell_volume) / total_volume

# Kill switch
if vpin > threshold:  # e.g., 0.7
    cancel_all_orders()  # Stop quoting — informed traders are active
    wait_for_cooldown()
```

When VPIN spikes, it means order flow is heavily directional (likely informed trading). The bot stops quoting to avoid being picked off.

---

## Implementation: How It Works in Code

### Basic Architecture

```
┌─────────────────────────────────────────────┐
│              Market Making Bot               │
├─────────────────────────────────────────────┤
│                                             │
│  ┌─────────┐  ┌──────────┐  ┌───────────┐  │
│  │  Data    │  │ Strategy │  │ Execution │  │
│  │  Feed    │→ │  Engine  │→ │  Engine   │  │
│  └─────────┘  └──────────┘  └───────────┘  │
│       │            │              │         │
│  WebSocket    Avellaneda-    CLOB API       │
│  price/book   Stoikov +      POST/DELETE    │
│  updates      GLFT + VPIN    orders         │
│                                             │
│  ┌──────────────────────────────────────┐   │
│  │         Risk Management              │   │
│  │  - Max inventory limits              │   │
│  │  - P&L tracking                      │   │
│  │  - VPIN kill switch                  │   │
│  │  - Max drawdown stop                 │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

### Python Pseudocode: Core Market Making Loop

```python
import numpy as np
from polymarket_client import PolymarketCLOB

class AvellanedaStoikovMM:
    def __init__(self, market_id, config):
        self.client = PolymarketCLOB(api_key, api_secret)
        self.market_id = market_id

        # Model parameters
        self.gamma = config['risk_aversion']      # e.g., 0.1
        self.k = config['order_intensity']         # e.g., 1.5
        self.q_max = config['max_inventory']       # e.g., 1000 shares
        self.vpin_threshold = config['vpin_kill']   # e.g., 0.7

        # State
        self.inventory = 0
        self.active_orders = []
        self.trade_history = []

    def compute_reservation_price(self, mid_price, time_remaining):
        """Avellaneda-Stoikov reservation price with binary vol adjustment."""
        # Binary market volatility
        sigma_sq = mid_price * (1 - mid_price)

        # Reservation price: skew based on inventory
        r = mid_price - self.inventory * self.gamma * sigma_sq * time_remaining

        return np.clip(r, 0.01, 0.99)

    def compute_optimal_spread(self, sigma_sq, time_remaining):
        """Optimal spread from A-S model."""
        spread = (self.gamma * sigma_sq * time_remaining +
                  (2 / self.gamma) * np.log(1 + self.gamma / self.k))
        return max(spread, 0.01)  # minimum 1 cent spread

    def compute_vpin(self, recent_trades, window=50):
        """Volume-synchronized probability of informed trading."""
        if len(recent_trades) < window:
            return 0

        buys = sum(t['size'] for t in recent_trades[-window:] if t['side'] == 'buy')
        sells = sum(t['size'] for t in recent_trades[-window:] if t['side'] == 'sell')
        total = buys + sells

        if total == 0:
            return 0
        return abs(buys - sells) / total

    def should_quote(self):
        """Check kill switches before quoting."""
        # VPIN kill switch
        if self.compute_vpin(self.trade_history) > self.vpin_threshold:
            return False

        # Inventory limits (GLFT bounds)
        if abs(self.inventory) >= self.q_max:
            return False

        return True

    def generate_quotes(self):
        """Main quoting logic."""
        if not self.should_quote():
            self.cancel_all_orders()
            return

        # Get current market state
        book = self.client.get_orderbook(self.market_id)
        mid_price = (book['best_bid'] + book['best_ask']) / 2
        time_remaining = self.get_time_to_resolution()
        sigma_sq = mid_price * (1 - mid_price)

        # Compute optimal quotes
        r = self.compute_reservation_price(mid_price, time_remaining)
        spread = self.compute_optimal_spread(sigma_sq, time_remaining)

        bid = np.clip(r - spread / 2, 0.01, 0.99)
        ask = np.clip(r + spread / 2, 0.01, 0.99)

        # GLFT inventory skew — aggressively skew when near limits
        inventory_ratio = self.inventory / self.q_max
        if inventory_ratio > 0.5:  # too long, lower bid size, raise ask
            bid_size = int(100 * (1 - inventory_ratio))
            ask_size = int(100 * (1 + inventory_ratio))
        elif inventory_ratio < -0.5:  # too short, raise bid, lower ask size
            bid_size = int(100 * (1 + abs(inventory_ratio)))
            ask_size = int(100 * (1 - abs(inventory_ratio)))
        else:
            bid_size = ask_size = 100

        # Cancel stale orders and place new ones
        self.cancel_all_orders()

        if bid_size > 0:
            self.place_order('BUY', bid, bid_size)
        if ask_size > 0:
            self.place_order('SELL', ask, ask_size)

    def run(self, interval_seconds=2):
        """Main loop — update quotes every N seconds."""
        while True:
            try:
                self.generate_quotes()
            except Exception as e:
                print(f"Error: {e}")
                self.cancel_all_orders()
            time.sleep(interval_seconds)
```

### Official Polymarket Python Client (py-clob-client)

```python
# pip install py-clob-client
# pin web3==6.14.0 to avoid dependency conflicts

from py_clob_client.client import ClobClient
from py_clob_client.clob_types import OrderArgs, OrderType
from py_clob_client.order_builder.constants import BUY

# Initialize client (signature_type=1 for email/Magic wallet)
client = ClobClient(
    "https://clob.polymarket.com",
    key="<private-key>",
    chain_id=137,
    signature_type=1,
    funder="<funder-address>"
)
client.set_api_creds(client.create_or_derive_api_creds())

# --- Read market data (no auth needed) ---
simple_client = ClobClient("https://clob.polymarket.com")
mid = simple_client.get_midpoint("<token-id>")
price = simple_client.get_price("<token-id>", side="BUY")
book = simple_client.get_order_book("<token-id>")

# --- Place a GTC limit order ---
order = OrderArgs(token_id="<token-id>", price=0.50, size=10.0, side=BUY)
signed = client.create_order(order)
resp = client.post_order(signed, OrderType.GTC)

# --- Place a FOK market order ---
from py_clob_client.clob_types import MarketOrderArgs
mo = MarketOrderArgs(
    token_id="<token-id>", amount=25.0, side=BUY, order_type=OrderType.FOK
)
signed = client.create_market_order(mo)
resp = client.post_order(signed, OrderType.FOK)
```

### TypeScript/Node.js: Connecting to Polymarket CLOB

```typescript
import { ethers } from 'ethers';

// Polymarket CLOB client setup
const CLOB_BASE = 'https://clob.polymarket.com';

interface Order {
  tokenID: string;
  price: number;
  size: number;
  side: 'BUY' | 'SELL';
  expiration: number;
  nonce: number;
  feeRateBps: number;
  signatureType: number;
  signature: string;
}

class PolymarketMMBot {
  private wallet: ethers.Wallet;
  private apiKey: string;
  private apiSecret: string;

  constructor(privateKey: string, apiKey: string, apiSecret: string) {
    this.wallet = new ethers.Wallet(privateKey);
    this.apiKey = apiKey;
    this.apiSecret = apiSecret;
  }

  async getOrderBook(tokenId: string) {
    const res = await fetch(`${CLOB_BASE}/book?token_id=${tokenId}`);
    return res.json();
  }

  async placeOrder(tokenId: string, side: 'BUY' | 'SELL', price: number, size: number) {
    // Sign order using EIP-712
    const order = await this.buildSignedOrder(tokenId, side, price, size);

    const res = await fetch(`${CLOB_BASE}/order`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'POLY_API_KEY': this.apiKey,
        'POLY_SIGNATURE': this.apiSecret,
        'POLY_TIMESTAMP': Date.now().toString(),
      },
      body: JSON.stringify(order),
    });
    return res.json();
  }

  // Main quoting loop using Avellaneda-Stoikov
  async runMMLoop(tokenId: string, config: MMConfig) {
    while (true) {
      const book = await this.getOrderBook(tokenId);
      const mid = (book.bids[0]?.price + book.asks[0]?.price) / 2;

      // Binary vol estimate
      const sigmaSq = mid * (1 - mid);

      // Reservation price with inventory skew
      const r = mid - config.inventory * config.gamma * sigmaSq * config.timeRemaining;
      const clippedR = Math.max(0.01, Math.min(0.99, r));

      // Optimal spread
      const spread = config.gamma * sigmaSq * config.timeRemaining
                     + (2 / config.gamma) * Math.log(1 + config.gamma / config.k);

      const bid = Math.max(0.01, clippedR - spread / 2);
      const ask = Math.min(0.99, clippedR + spread / 2);

      await this.cancelAllOrders();
      await this.placeOrder(tokenId, 'BUY', bid, config.size);
      await this.placeOrder(tokenId, 'SELL', ask, config.size);

      await new Promise(resolve => setTimeout(resolve, 2000));
    }
  }
}
```

---

## Real-World Strategies That Made Money

### 1. Jane Street's 15-Minute Momentum Lag Strategy

Identified by @gemchange_ltd — an address named "JaneStreetIndia" made ~$360,000 trading Polymarket's 15-minute crypto price prediction markets:

- **How it works**: These markets ask "Will BTC/ETH/SOL go up or down in the next 15 minutes?"
- **The edge**: When crypto moves hard in one direction, these short-window markets lag by 30-90 seconds. Order books thin out, prices stick.
- **The bot**: Scans every active micro-window, watching for 3-5% gaps between where the market *should* be priced and where it *is* priced
- **Dual-directional bets**: Often bets on both YES and NO when total cost < $1 (e.g., YES at $0.48 + NO at $0.46 = $0.94, guaranteed $0.06 profit regardless of outcome)
- **Result**: 99.5% algorithm accuracy over extended run

### 2. Weather Market Bots (Most Accessible Edge Right Now)

This is arguably the most accessible and documented profitable strategy:

- **Core mechanism**: Compare official weather forecasts (GFS, ECMWF, ICON models) to Polymarket prices. When multiple models agree but the market is mispriced, bet on the forecast
- **Multi-model consensus**: When 3+ models agree on a temperature range, accuracy is 70-90%, but markets often price these outcomes much lower
- **Information speed edge**: Weather models update every 6 hours; bots that ingest new model runs immediately can trade before the market adjusts
- **Documented profits**:
  - One bot turned $1,000 into $24,000 since April 2025 trading London weather markets
  - Another pulled in $65,000 across New York, London, and Seoul weather
  - Trader "neobrother" accumulated $20,000+ via "temperature laddering"
  - Trader "gopfan2" made $2M+ net profit, mostly from weather
  - Trader "Hans323" earned $1.11M from a single London weather prediction ($92K bet at 8% odds)

### 3. News-Based Speed Trading

- Breaking news hits 5-10 minutes before Polymarket prices react
- Monitoring news APIs, press release feeds, official government channels
- Automated systems that detect keywords and trade before the market adjusts
- @gemchange_ltd claims this window is "enough time to make serious money"

### 4. Market Making for Maker Rewards

- Polymarket incentivizes liquidity provision with maker rewards (paid daily at midnight UTC)
- Rewards use a quadratic spread function that heavily penalizes quotes far from the midpoint; ~3x rewards for two-sided quoting
- **Early returns**: With ~$10K capital, some LPs reported $200-$800/day at peak
- **Current realistic returns**: ~10% annualized for well-designed systems targeting low-volatility, long-dated markets
- Best targets: long-dated markets without imminent catalysts (e.g., 2028 election markets)

### 5. Cross-Platform Arbitrage

- Tools like ArbBets and EventArb automate arbitrage between Polymarket and rivals (Kalshi, Betfair)
- $40M+ in total arbitrage profits from April 2024 to April 2025
- **Critical limitation**: Average arb opportunity duration has dropped to 2.7 seconds (from 12.3s in 2024), with 73% of profits captured by sub-100ms bots

### 6. AI-Driven Probability Trading

- Multiple AI models (GPT-4, Claude, fine-tuned models) analyze headlines and assign probabilities
- Identify edges when the market is mispriced relative to AI assessment
- One AI-driven probabilistic model reportedly generated $2.2M in profits within two months

---

## The Polymarket Ecosystem: 170+ Tools

As of early 2026, Polymarket hit $21.5B in trading volume during 2025, with weekly volumes exceeding $1.5B by January 2026. The ecosystem has grown to 170+ third-party tools across 19 categories.

**Key stat: Only 7.6% of wallets are profitable** (~120,000 making money while 1.5M lose). Only 0.51% of users earned more than $1,000.

| Category | Examples | Description |
|----------|----------|-------------|
| **AI Agents** | PolyBro, Billy Bets, Astron (Raven 1.0), Fraction AI | Autonomous AI-powered trading |
| **Analytics** | Polytrader, PolyRadar, Alphascope, Inside Edge | Market data and intelligence |
| **Copy Trading** | PolyGun, PolyDex, Polycop, Polycool, OkBet | Follow successful traders |
| **Cross-Platform Arb** | ArbBets, EventArb ($40M+ in profits) | Price differences between prediction markets |
| **Trading Terminals** | Betmoar (~$110M volume), Stand.trade | Professional trading interfaces |
| **Aggregators** | Oddpool ("Bloomberg of prediction markets") | Cross-market data aggregation |
| **Fund Management** | PolyFund | Decentralized fund creation, pooled capital |
| **Alerts** | PolyAlertHub, PolySpyBot | Telegram/email notifications |
| **Whale Tracking** | Polyburg, Polywhaler, HashDive | Track large trader movements |
| **Sports Prediction** | Sportstensor (NFL/NBA AI), Rainmaker | Sports-specific models |
| **Dev Infrastructure** | Dome, PolyRouter, py-clob-client | Standardized APIs and clients |
| **Bot Frameworks** | OpenClaw (one bot made $115K in a single week) | Build custom trading bots |

### Notable Open-Source Implementations

| Repo | Description | Status |
|------|-------------|--------|
| [warproxxx/poly-maker](https://github.com/warproxxx/poly-maker) | Market making bot with Google Sheets config | Author warns: "not profitable in today's market" |
| [lorine93s/polymarket-market-maker-bot](https://github.com/lorine93s/polymarket-market-maker-bot) | Production-ready CLOB market maker | Full risk controls, inventory management |
| [fedecaccia/avellaneda-stoikov](https://github.com/fedecaccia/avellaneda-stoikov) | Pure A-S model implementation (Python) | Educational, not Polymarket-specific |
| [elielieli909/polymarket-marketmaking](https://github.com/elielieli909/polymarket-marketmaking) | Automated MM with bands.json config | Configurable via JSON |

---

## What Can You Use This For?

Given your background as a senior web engineer (React, TypeScript, Node.js), here are practical applications ranked by feasibility and usefulness:

### Directly Applicable

#### 1. Build a Polymarket Analytics Dashboard (Web App)
- Use your React/Next.js skills to build a real-time dashboard consuming the Polymarket CLOB WebSocket API
- Visualize order book depth, price movements, volume, whale activity
- Monitor VPIN and other market microstructure signals
- **Stack**: Next.js + WebSocket + D3.js/Recharts + MongoDB for historical data
- **Monetization**: SaaS tool for Polymarket traders

#### 2. Build a Copy Trading / Signal Platform
- Track top Polymarket traders (on-chain data is public on Polygon)
- Build alerts when high-performers enter new positions
- **Stack**: Node.js backend polling Polygon RPCs + React frontend + Telegram bot for alerts

#### 3. Build a News-to-Market Signal System
- Monitor news APIs (NewsAPI, RSS feeds, social media)
- Use NLP/LLM to extract events relevant to active Polymarket markets
- Display matching markets with current prices and estimated "fair" price
- **Stack**: Node.js + OpenAI API + Polymarket API + React dashboard

### Educational / Portfolio Projects

#### 4. Implement an Avellaneda-Stoikov Simulator
- Build an interactive web visualization of the A-S model
- Let users adjust parameters (gamma, sigma, inventory) and see how quotes change in real-time
- Great portfolio piece demonstrating understanding of quant finance
- **Stack**: React + Canvas/WebGL for visualization

#### 5. Paper Trading Bot
- Implement the full market making stack against real Polymarket data, but simulate trades instead of executing
- Track theoretical P&L to validate strategies without risking capital
- **Stack**: TypeScript/Node.js + Polymarket WebSocket API

### Advanced (If Interested in Trading)

#### 6. Automated Market Making Bot
- Implement the full Avellaneda-Stoikov + GLFT + VPIN stack
- Start on low-volume markets where competition is minimal
- Use Polymarket's maker rewards to offset spread losses
- **Warning**: The poly-maker author notes this is "not profitable" due to competition from professional firms

#### 7. Weather/Niche Market Specialist Bot
- Focus on less competitive market categories (weather, sports, niche events)
- Use domain-specific data sources (NOAA weather data, sports APIs) for pricing edge
- Lower competition than crypto/political markets

### Transferable Skills & Concepts

Even if you never trade on Polymarket, understanding these concepts is valuable:

- **Market Microstructure** → Understanding how exchanges, order books, and liquidity work (useful for any fintech work)
- **Real-time Systems** → WebSocket architectures, event-driven programming, low-latency processing
- **Risk Management** → Inventory management, kill switches, position limits (applicable to any system design)
- **Quantitative Modeling** → Statistical modeling, parameter optimization, backtesting
- **On-chain Integration** → Polygon/EVM integration, EIP-712 signing, smart contract interaction

---

## Risks and Limitations

### Financial Risks
- **Most people lose money**: Only 7.6% of wallets are profitable; 80% of participants are net losers
- **Binary loss**: If your prediction is wrong, you lose 100% of what you invested
- **Competition**: Professional quant firms (Jane Street, etc.) have massive advantages in speed, capital, and sophistication
- **Adverse Selection**: You *will* trade against informed traders who know outcomes before you
- **Resolution Risk**: Markets can resolve unexpectedly — a $0.95 YES position can still go to $0
- **Smart Contract Risk**: Bugs in the CTF token contracts or Polymarket's system

### Competitive / Structural Risks
- **Bot dominance**: 73% of arbitrage profits captured by sub-100ms bots. Average arb opportunity duration: 2.7 seconds
- **Spread compression**: Bid-ask spreads compressed from 4.5% (2023) to 1.2% (2025). Professional market makers dominate order books
- **Liquidity reward decay**: Rewards that once yielded 2-3% daily now yield ~10% annualized

### Regulatory Risks
- **U.S. Federal**: Polymarket paid $1.4M CFTC penalty in 2022. Re-entered the U.S. market in late 2025 after acquiring CFTC-licensed QCEX for $112M
- **U.S. State conflicts**: Nevada (gaming license dispute), Tennessee (ordered closure of sports prediction markets), Massachusetts (injunction against sports-related contracts)
- **International bans**: Blocked in Poland, Singapore, Belgium, Hungary, and Portugal
- **Ethical controversies**: Markets on NASA mission explosions, war outcomes, and "death markets" have drawn Congressional scrutiny
- **Insider trading**: In January 2026, a newly created account made $400K+ on Venezuela-related positions under suspicion of insider knowledge, prompting the "Public Integrity in Financial Prediction Markets Act of 2026"

### Technical Risks
- **Latency**: Your bot is competing against colocated systems; retail latency is a disadvantage
- **API Rate Limits**: Polymarket's CLOB API limits: 100 req/min (public), 60 orders/min (trading)
- **Liquidity**: Many markets are thin — your orders may be the only liquidity, making you vulnerable
- **Order Book Manipulation**: A single <$0.10 transaction can wipe market-making orders worth tens of thousands of dollars (documented vulnerability)
- **Security**: Documented cases of fake Polymarket npm packages that steal private keys
- **Wash trading**: Research flagged ~15% of Polymarket wallets as having activity consistent with wash trading

### The Honest Assessment

@gemchange_ltd themselves [wrote a separate thread](https://x.com/gemchange_ltd/status/2003420311983731150) titled **"Devs are making $10k-200k monthly on Polymarket — no they're not"**, debunking easy-profit claims: zero fees = zero friction for HFT shops, and you're competing with Rust bots on dedicated Polygon nodes.

As the author of poly-maker (warproxxx) puts it:

> "In today's market, this bot is not profitable and will lose money. I recommend using it as a reference implementation. Increased competition on Polymarket makes it impractical unless you're willing to dedicate significant time."

The real value for most developers is in **building tools around the ecosystem** (analytics, dashboards, signals) rather than trying to compete as a market maker against institutional quant firms.

---

## Resources

### Academic Papers
- [Avellaneda & Stoikov (2008) — "High-frequency trading in a limit order book"](https://www.math.nyu.edu/~avellane/HighFrequencyTrading.pdf) — The foundational paper
- Guéant, Lehalle & Fernandez-Tapia — "Dealing with inventory risk" (GLFT extension)
- Glosten & Milgrom (1985) — "Bid, ask and transaction prices" (adverse selection model)

### Code Repositories
- [fedecaccia/avellaneda-stoikov](https://github.com/fedecaccia/avellaneda-stoikov) — Pure model implementation
- [warproxxx/poly-maker](https://github.com/warproxxx/poly-maker) — Polymarket market maker
- [lorine93s/polymarket-market-maker-bot](https://github.com/lorine93s/polymarket-market-maker-bot) — Production MM bot
- [Polymarket CLOB Client (Python)](https://github.com/Polymarket/py-clob-client) — Official Python client
- [Polymarket CLOB Client (TypeScript)](https://github.com/Polymarket/clob-client) — Official TS client

### Articles & Threads
- [@gemchange_ltd — "How Jump Trading, Jane Street, and a Guy With $10K..."](https://x.com/gemchange_ltd/status/2025908468633268456) — The main article
- [@gemchange_ltd — "Devs are making $10k-200k monthly — no they're not"](https://x.com/gemchange_ltd/status/2003420311983731150) — Reality check
- [@gemchange_ltd — Jane Street bot analysis](https://x.com/gemchange_ltd/status/1999398039761428615) — JaneStreetIndia $360K HFT
- [@Kropanchik's reaction](https://x.com/Kropanchik/status/2025730323342876944) — "The $10K + Python era is dead"
- [DeFiPrime — Definitive Guide to the Polymarket Ecosystem](https://defiprime.com/definitive-guide-to-the-polymarket-ecosystem)
- [Polymarket — Automated Market Making guide](https://news.polymarket.com/p/automated-market-making-on-polymarket)
- [Avellaneda-Stoikov implementation walkthrough (Medium)](https://medium.com/@degensugarboo/avellaneda-and-stoikov-mm-paper-implementation-b7011b5a7532)
- [Jump Trading takes stakes in Polymarket/Kalshi (CoinDesk)](https://www.coindesk.com/business/2026/02/10/jump-trading-to-take-small-stakes-in-polymarket-kalshi-bloomberg/)

### Polymarket Documentation
- [Polymarket CLOB API Docs](https://docs.polymarket.com/developers/CLOB/introduction)
- [Polymarket Orders Overview](https://docs.polymarket.com/developers/CLOB/orders/orders)
- [Polymarket Liquidity Rewards Docs](https://docs.polymarket.com/polymarket-learn/trading/liquidity-rewards)
- [Polymarket GitHub Organization](https://github.com/Polymarket)
- [Polymarket AI Agents Repo](https://github.com/Polymarket/agents)
- [CTF Exchange Contract](https://github.com/Polymarket/ctf-exchange)

### Tool Directories
- [PolyCatalog](https://www.polycatalog.io/polymarket-tools) — Comprehensive tool directory
- [Polymark.et](https://polymark.et/) — Apps directory
- [Awesome Prediction Market Tools](https://github.com/aarora4/Awesome-Prediction-Market-Tools) — Curated list
