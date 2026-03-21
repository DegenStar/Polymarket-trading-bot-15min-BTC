# Bitcoin 15min Arbitrage Bot - Polymarket

[English](README.md) | [中文说明](README-CN.md)

Professional arbitrage bot for Bitcoin 15-minute markets on Polymarket.

> 🆕 **Enhanced Version**: This bot has been significantly improved with professional features including statistics tracking, risk management, enhanced logging, and configuration validation. See [CHANGELOG.md](CHANGELOG.md) for details. **100% backward compatible** - all new features are optional.

> 📚 **New to the bot?** Check out the [docs/GETTING_STARTED.md](docs/GETTING_STARTED.md) for a quick start guide!

---

## 🎯 Strategy

**Pure arbitrage**: Buy both sides (UP + DOWN) when total cost < $1.00 to guarantee profit regardless of outcome.

### Example:
```
BTC goes up (UP):     $0.48
BTC goes down (DOWN): $0.51
─────────────────────────
Total:                $0.99  ✅ < $1.00
Profit:               $0.01 per share (1.01%)
```

**Why does it work?**
- At close, ONE of the two sides pays $1.00 per share
- If you paid $0.99 total, you earn $0.01 no matter which side wins
- It's **guaranteed profit** (pure arbitrage)

---

## 🚀 Installation

### 1. Clone the repository and enter the project directory
(Make sure you have `git` installed. If not, see ➡️ [Git installation guide](./安装git教程.md))

```
git clone https://github.com/web3toolshub/Polymarket-trading-bot-15min-BTC

cd Polymarket-trading-bot-15min-BTC
```

### 2. Install dependencies

- 📌 **Linux / macOS / WSL**
```bash
# Automatically check/install missing dependencies and configure env
./install.sh
```

- 📌 **Windows**
```powershell
# Run PowerShell as Administrator, then execute in project root
Set-ExecutionPolicy Bypass -Scope CurrentUser -Force
.\install.ps1
```

### 3. Copy the sample environment file
```
cp .env.example .env
```

### 4. Configure environment variables

#### Required Variables

| Variable | Description | How to Get It |
|----------|-------------|---------------|
| `POLYMARKET_PRIVATE_KEY` | Your wallet's private key (starts with `0x`) | Export from your wallet (MetaMask, etc.) or use the one linked to your Polymarket account |
| `POLYMARKET_API_KEY` | API key for Polymarket CLOB | Run `poetry run python -m src.generate_api_key` |
| `POLYMARKET_API_SECRET` | API secret for Polymarket CLOB | Run `poetry run python -m src.generate_api_key` |
| `POLYMARKET_API_PASSPHRASE` | API passphrase for Polymarket CLOB | Run `poetry run python -m src.generate_api_key` |

#### Wallet Configuration

| Variable | Description | Value |
|----------|-------------|-------|
| `POLYMARKET_SIGNATURE_TYPE` | Type of wallet signature | `0` = EOA (MetaMask, hardware wallet)<br>`1` = Magic.link (email login on Polymarket)<br>`2` = Gnosis Safe |
| `POLYMARKET_FUNDER` | Proxy wallet address (only for Magic.link users) | Leave **empty** for EOA wallets. For Magic.link, see instructions below. |

##### ⚠️ Important: Magic.link users (signature_type=1)

If you use **email login** on Polymarket (Magic.link), you have **two addresses**:

1. **Signer address** (derived from your private key): This is the wallet that signs transactions.
2. **Proxy wallet address** (POLYMARKET_FUNDER): This is where your funds actually live on Polymarket.

**To find your proxy wallet address:**
1. Go to your Polymarket profile: `https://polymarket.com/@YOUR_USERNAME`
2. Click the **"Copy address"** button next to your balance
3. This is your `POLYMARKET_FUNDER` — it should look like `0x...` and is **different** from your signer address

**Common mistake:** Setting `POLYMARKET_FUNDER` to your Polygon wallet address (where you might have USDC on-chain) instead of the Polymarket proxy address. This causes `"invalid signature"` errors.

**How to verify:** Run `poetry run python -m src.test_balance`:
- "Getting USDC balance" shows the balance via Polymarket API (should show your funds)
- "Balance on-chain" queries Polygon directly (may show $0 if your funds are in the proxy, which is normal)

#### Trading Configuration

| Variable | Description | Default | Recommended |
|----------|-------------|---------|-------------|
| `TARGET_PAIR_COST` | Maximum combined cost to trigger arbitrage | `0.99` | `0.99` - `0.995` |
| `ORDER_SIZE` | Number of shares per trade (minimum is 5) | `50` | Start with `5`, increase after testing |
| `ORDER_TYPE` | Order time-in-force (`FOK`, `FAK`, `GTC`) | `FOK` | Use `FOK` to avoid leaving one leg open |
| `DRY_RUN` | Simulation mode | `false` | Start with `true`, change to `false` for live trading |
| `SIM_BALANCE` | Starting cash used in simulation mode (`DRY_RUN=true`) | `0` | e.g. `100` |
| `COOLDOWN_SECONDS` | Minimum seconds between executions | `10` | Increase if you see repeated triggers |

#### Risk Management (New) ⚡

| Variable | Description | Default | Recommended |
|----------|-------------|---------|-------------|
| `MAX_DAILY_LOSS` | Maximum loss per day in USDC (0 = disabled) | `0` | e.g. `50.0` to limit daily losses |
| `MAX_POSITION_SIZE` | Maximum position size in USDC per trade (0 = disabled) | `0` | e.g. `100.0` to cap trade sizes |
| `MAX_TRADES_PER_DAY` | Maximum number of trades per day (0 = disabled) | `0` | e.g. `20` to limit trading frequency |
| `MIN_BALANCE_REQUIRED` | Minimum balance required to continue trading | `10.0` | Adjust based on your risk tolerance |
| `MAX_BALANCE_UTILIZATION` | Maximum % of balance to use per trade (0.8 = 80%) | `0.8` | Lower = more conservative |

#### Statistics & Logging (New) 📊

| Variable | Description | Default |
|----------|-------------|---------|
| `ENABLE_STATS` | Enable statistics tracking and trade history | `true` |
| `TRADE_LOG_FILE` | Path to trade history JSON file | `trades.json` |
| `USE_RICH_OUTPUT` | Use rich console formatting (requires `rich` package) | `true` |
| `VERBOSE` | Enable verbose (DEBUG) logging | `false` |

#### Optional

| Variable | Description |
|----------|-------------|
| `POLYMARKET_MARKET_SLUG` | Force a specific market slug (leave empty for auto-discovery) |
| `USE_WSS` | Enable Polymarket Market WebSocket feed (`true`/`false`) |
| `POLYMARKET_WS_URL` | Base WSS URL (default: `wss://ws-subscriptions-clob.polymarket.com`) |

---

## 🔑 Generating API Keys

Before running the bot, you need to generate your Polymarket API credentials.

### Step 1: Set your private key

Make sure your `.env` file includes your private key:
```env
POLYMARKET_PRIVATE_KEY=0xYOUR_PRIVATE_KEY_HERE
```

### Step 2: Run the API key generator

```
poetry run python -m src.generate_api_key
```

This will output something like:
```
API Key: abc123...
Secret: xyz789...
Passphrase: mypassphrase
```

### Step 3: Add the credentials to `.env`

```env
POLYMARKET_API_KEY=abc123...
POLYMARKET_API_SECRET=xyz789...
POLYMARKET_API_PASSPHRASE=mypassphrase
```

> ⚠️ **Important**: The API credentials are derived from your private key. If you change the private key, you'll need to regenerate the API credentials.

---

## � Diagnosing Configuration Issues

If you get `"invalid signature"` errors, run the diagnostic tool:

```
poetry run python -m src.diagnose_config
```

This will check:
- Whether your `POLYMARKET_FUNDER` is correctly set (required for Magic.link accounts)
- Whether the signer and funder addresses are different (they should be for Magic.link)
- Whether the bot can detect `neg_risk` for BTC 15min markets
- Your current USDC balance via the Polymarket API

**Common causes of "invalid signature":**
1. `POLYMARKET_FUNDER` is empty for Magic.link accounts
2. `POLYMARKET_FUNDER` is set to your Polygon wallet address instead of your Polymarket proxy wallet
3. API credentials were generated with a different private key or configuration
4. The `neg_risk` flag is incorrectly detected (fixed in latest version - bot now forces `neg_risk=True` for BTC 15min markets)

**About "Balance on-chain" showing $0:**
This is **normal** for Magic.link accounts. Your funds are held in a Polymarket proxy contract, not directly in your Polygon wallet. The "USDC balance" via API should show your correct balance.

---

## �💰 Checking Your Balance

Before trading, verify that your wallet is configured correctly and has funds:

```
poetry run python -m src.test_balance
```

This will show:
```
======================================================================
POLYMARKET BALANCE TEST
======================================================================
Host: https://clob.polymarket.com
Signature Type: 1
Private Key: ✓
API Key: ✓
API Secret: ✓
API Passphrase: ✓
======================================================================

1. Creating ClobClient...
   ✓ Client created

2. Deriving API credentials from private key...
   ✓ Credentials configured

3. Getting wallet address...
   ✓ Address: 0x52e78F6071719C...

4. Getting USDC balance (COLLATERAL)...
   💰 BALANCE USDC: $25.123456

5. Verifying balance directly on Polygon...
   🔗 Balance on-chain: $25.123456

======================================================================
TEST COMPLETED
======================================================================
```

> ⚠️ If balance shows `$0.00` but you have funds on Polymarket, check your `POLYMARKET_SIGNATURE_TYPE` and `POLYMARKET_FUNDER` settings.

---

## 💻 Usage

### Simulation mode (recommended first):

Make sure `DRY_RUN=true` in `.env`, then:
```
poetry run python -m src.simple_arb_bot
```

The bot will scan for opportunities but won't place real orders.

### Optional: WebSocket market data (lower latency)

By default the bot polls the CLOB order book over HTTPS. You can optionally enable
the Polymarket CLOB **Market WebSocket** feed to receive pushed order book updates
and reduce per-scan latency.

Set the following in your `.env`:

```env
USE_WSS=true
POLYMARKET_WS_URL=wss://ws-subscriptions-clob.polymarket.com
```

Notes on WSS mode:
- The Market channel can send either a single JSON object or a JSON array (batched events). The bot handles both.
- If the connection drops or a proxy/firewall blocks WSS, the bot will reconnect and print the error reason.
- Internally, WSS mode maintains an in-memory L2 book using `book` snapshots + `price_change` deltas.

Then run the bot the same way:

```
poetry run python -m src.simple_arb_bot
```

### Live trading mode:

1. Change `DRY_RUN=false` in `.env`
2. Ensure you have USDC in your Polymarket wallet
3. Run:
```
poetry run python -m src.simple_arb_bot
```

### Paired execution safety (avoids “one-leg fills”)

In real trading, it’s possible for **only one leg** (UP or DOWN) to fill if the book moves.
To reduce the risk of ending up with an imbalanced position, the bot now:

- **Submits both legs**, then **verifies** each order by polling `get_order`.
- Only logs **“EXECUTED (BOTH LEGS FILLED)”** and increments `trades_executed` when **both** legs are confirmed filled.
- If only one leg fills, it will **best-effort cancel** the remaining order(s) and attempt to **flatten exposure** by submitting a
   `SELL` on the filled leg at the current `best_bid` using `FAK` (fill-and-kill).

Recommendation:
- Keep `ORDER_TYPE=FOK` for entries (fill-or-kill) to avoid leaving open orders.

Important:
- This is **risk-reduction**, not a perfect guarantee. In fast markets, unwind orders can also fail or partially fill.
- Always monitor your positions on Polymarket, especially if you see a “Partial fill detected” warning.

---

## 📊 Features

### Core Features
✅ **Auto-discovers** active BTC 15min market  
✅ **Detects opportunities** when price_up + price_down < threshold  
✅ **Execution-aware pricing**: uses order book asks (not last trade price)  
✅ **Depth-aware sizing**: walks the ask book to ensure `ORDER_SIZE` can fill (uses a conservative "worst fill" price)  
✅ **Continuous scanning** with no delays (maximum speed)  
✅ **Lower latency polling**: fetches UP/DOWN order books concurrently  
✅ **Auto-switches** to next market when current one closes  
✅ **Final summary** with total investment, profit and market result  
✅ **Simulation mode** for risk-free testing  
✅ **Balance verification** before executing trades  
✅ **Paired execution verification**: confirms both legs filled (otherwise cancels + attempts to unwind)

### Enhanced Features (New) ⚡
✅ **Statistics Tracking**: Comprehensive trade history and performance metrics  
✅ **Risk Management**: Daily loss limits, position size limits, trade frequency controls  
✅ **Configuration Validation**: Validates settings before startup with helpful error messages  
✅ **Enhanced Logging**: Rich console output with colors and better formatting (optional)  
✅ **Graceful Shutdown**: Clean shutdown with statistics saving  
✅ **Trade History Export**: Export trade data to JSON and CSV formats  
✅ **Performance Analytics**: Win rate, average profit, and detailed statistics  

---

## 📈 Example Output

```
🚀 BITCOIN 15MIN ARBITRAGE BOT STARTED
======================================================================
Market: btc-updown-15m-1765301400
Time remaining: 12m 34s
Mode: 🔸 SIMULATION
Cost threshold: $0.99
Order size: 5 shares
======================================================================

[Scan #1] 12:34:56
No arbitrage: UP=$0.48 + DOWN=$0.52 = $1.00 (needs < $0.99)

🎯 ARBITRAGE OPPORTUNITY DETECTED
======================================================================
UP price (goes up):   $0.4800
DOWN price (goes down): $0.5100
Total cost:           $0.9900
Profit per share:     $0.0100
Profit %:             1.01%
----------------------------------------------------------------------
Order size:           5 shares each side
Total investment:     $4.95
Expected payout:      $5.00
EXPECTED PROFIT:      $0.05
======================================================================
✅ ARBITRAGE EXECUTED SUCCESSFULLY

🏁 MARKET CLOSED - FINAL SUMMARY
======================================================================
Market: btc-updown-15m-1765301400
Result: UP (goes up) 📈
Mode: 🔴 REAL TRADING
----------------------------------------------------------------------
Total opportunities detected:  3
Total trades executed:         3
Total shares bought:           30
----------------------------------------------------------------------
Total invested:                $14.85
Expected payout at close:      $15.00
Expected profit:               $0.15 (1.01%)
----------------------------------------------------------------------
📊 OVERALL STATISTICS:
  Total trades:                 3
  Win rate:                     100.0%
  Average profit per trade:     $0.05
  Average profit %:             1.01%
----------------------------------------------------------------------
⚠️ RISK MANAGEMENT:
  Daily trades:                 3
  Daily net P&L:                $0.15
======================================================================
```

---

## 📁 Project Structure

```
Bot/
├── src/
│   ├── simple_arb_bot.py    # Main arbitrage bot
│   ├── config.py            # Configuration loader
│   ├── config_validator.py  # Configuration validation (NEW)
│   ├── lookup.py            # Market ID fetcher
│   ├── trading.py           # Order execution
│   ├── statistics.py        # Statistics tracking (NEW)
│   ├── risk_manager.py      # Risk management (NEW)
│   ├── logger.py            # Enhanced logging (NEW)
│   ├── utils.py             # Utility functions (NEW)
│   ├── wss_market.py        # WebSocket market client
│   ├── generate_api_key.py  # API key generator utility
│   ├── diagnose_config.py   # Configuration diagnostic tool
│   └── test_balance.py      # Balance verification utility
├── tests/
│   └── test_state.py        # Unit tests
├── .env                     # Environment variables (create from .env.example)
├── .env.example             # Environment template (if available)
├── requirements.txt         # Dependencies
├── README.md                # This file
├── CHANGELOG.md             # Detailed changelog
└── docs/                    # Documentation folder
    ├── README.md            # Documentation index
    ├── GETTING_STARTED.md   # Quick start guide
    ├── CONFIGURATION.md     # Configuration guide
    ├── FEATURES.md          # Features guide
    └── TROUBLESHOOTING.md   # Troubleshooting guide
```

---

## ⚠️ Warnings

- ⚠️ **DO NOT use `DRY_RUN=false` without funds** in your Polymarket wallet
- ⚠️ **Spreads** can eliminate profit (verify liquidity)
- ⚠️ Markets close every **15 minutes** (don't accumulate positions)
- ⚠️ Start with **small orders** (ORDER_SIZE=5)
- ⚠️ This software is **educational only** - use at your own risk
- ⚠️ **Never share your private key** with anyone

---

## 🔧 Troubleshooting

### Configuration Validation

The bot now validates your configuration before starting. If you see validation errors:
- Check the error messages for specific issues
- Verify your `.env` file format
- Ensure all required fields are set
- Run `poetry run python -m src.diagnose_config` for detailed diagnostics

### "Invalid signature" error
- Verify `POLYMARKET_SIGNATURE_TYPE` matches your wallet type
- Regenerate API credentials with `poetry run python -m src.generate_api_key`
- For Magic.link users: ensure `POLYMARKET_FUNDER` is set correctly
- Run `poetry run python -m src.diagnose_config` for detailed diagnostics

### Balance shows $0 but I have funds
- Check that your private key corresponds to the wallet with funds
- For Magic.link: the private key is for your EOA, not the proxy wallet
- Run `poetry run python -m src.test_balance` to see your wallet address
- Verify `POLYMARKET_FUNDER` is set for Magic.link accounts

### "No active BTC 15min market found"
- Markets open every 15 minutes; wait for the next one
- Check your internet connection
- Try visiting https://polymarket.com/crypto/15M manually

### Trade blocked by risk management
- Check your risk management settings (MAX_DAILY_LOSS, MAX_POSITION_SIZE, etc.)
- Review the risk management stats in the final summary
- Adjust limits if needed (set to 0 to disable)

### Statistics not showing
- Ensure `ENABLE_STATS=true` in your `.env` file
- Check that `TRADE_LOG_FILE` is writable
- Verify you have write permissions in the bot directory

---

## 📚 Resources & Documentation

### Documentation
- **[docs/README.md](docs/README.md)** - Documentation index and navigation
- **[docs/GETTING_STARTED.md](docs/GETTING_STARTED.md)** - Quick start guide (5 minutes)
- **[docs/CONFIGURATION.md](docs/CONFIGURATION.md)** - Complete configuration guide
- **[docs/FEATURES.md](docs/FEATURES.md)** - Detailed feature explanations
- **[docs/TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md)** - Common issues and solutions
- **[CHANGELOG.md](CHANGELOG.md)** - Detailed changelog of all improvements

### External Resources
- [Polymarket](https://polymarket.com/)
- [BTC 15min Markets](https://polymarket.com/crypto/15M)
- [py-clob-client documentation](https://github.com/Polymarket/py-clob-client)

### Utilities
- `poetry run python -m src.generate_api_key` - Generate API credentials
- `poetry run python -m src.test_balance` - Verify wallet configuration and balance
- `poetry run python -m src.diagnose_config` - Diagnose configuration issues

---


## 🆕 What's New?

This bot has been significantly enhanced with professional features:

- **Statistics Tracking**: Track all trades, performance metrics, and export data
- **Risk Management**: Configure daily limits, position sizes, and trade frequency
- **Enhanced Logging**: Rich console output with better formatting
- **Configuration Validation**: Catch configuration errors before trading
- **Graceful Shutdown**: Clean shutdown with data preservation
- **Better Documentation**: Comprehensive beginner's guide and detailed docs

All new features are **optional** and the bot is **100% backward compatible**. See [CHANGELOG.md](CHANGELOG.md) for details.

## ⚖️ Disclaimer

This software is for educational purposes only. Trading involves risk. I am not responsible for financial losses. Always do your own research and never invest more than you can afford to lose.

**Risk Management Features**: While the bot includes risk management tools, these are not guarantees against losses. Always monitor your trades and set appropriate limits based on your risk tolerance.

☕ **Buy me a coffee (EVM):** `0xd9c5d6111983ea3692f1d29bec4ac7d6f723217a`
